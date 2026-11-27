# Task 1.1 labs

These labs run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

| Lab | Skills | What you build |
|---|---|---|
| [Lab 1](#lab-1-event-driven-s3-upload-triggers-a-lambda-function) | 1.1.8, 1.1.2 | An Amazon Simple Storage Service (S3) upload that triggers an AWS Lambda function |
| [Lab 2](#lab-2-idempotent-writes-with-a-conditional-expression) | 1.1.3, 1.3.4 | A Lambda function that makes retried requests safe with a DynamoDB conditional write |
| [Lab 3](#lab-3-aws-sdk-credential-resolution-made-visible) | 1.1.5, 2.1.3 | A script that shows which credential source the AWS Software Development Kit (SDK) used |

## Lab 1: Event-driven: S3 upload triggers a Lambda function

**Skills:** 1.1.8, 1.1.2

**Why this lab:** the exam constantly contrasts synchronous request/response with asynchronous, event-driven processing. Building the event-driven side makes the difference between "the caller waits" and "the event source hands off and moves on" concrete instead of abstract.

```bash
# 1. Create bucket and function (the function just logs the event; see 1.1.8 for the event-driven flow)
awslocal s3 mb s3://uploads-bucket

cat > s3handler.py <<'EOF'
def handler(event, context):
    for record in event["Records"]:
        print("New object:", record["s3"]["object"]["key"])
    return {"statusCode": 200}
EOF
zip s3function.zip s3handler.py

awslocal lambda create-function \
  --function-name process-upload \
  --runtime python3.12 \
  --handler s3handler.handler \
  --zip-file fileb://s3function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

# 2. Wire the S3 event notification to the Lambda function
awslocal lambda add-permission --function-name process-upload \
  --statement-id s3invoke --action lambda:InvokeFunction \
  --principal s3.amazonaws.com --source-arn arn:aws:s3:::uploads-bucket

awslocal s3api put-bucket-notification-configuration --bucket uploads-bucket \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [{
      "LambdaFunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:process-upload",
      "Events": ["s3:ObjectCreated:*"]
    }]
  }'

# 3. Trigger it
echo "test content" > test.txt
awslocal s3 cp test.txt s3://uploads-bucket/

# 4. Check the Lambda logs to confirm the event fired
awslocal logs tail /aws/lambda/process-upload --follow
```

**What to notice:** `aws s3 cp` returns as soon as the object is stored. The function runs afterward, invoked asynchronously by S3, and its output appears only in the logs. Nothing reports a failure back to the uploader, which is why asynchronous invocations need a dead-letter queue or on-failure destination. See [1.1.2 Communication patterns](1.1.2-communication-patterns.md#lambda-invocation-models) and [1.1.8 Event-driven applications](1.1.8-event-driven-applications.md).

## Lab 2: Idempotent writes with a conditional expression

**Skills:** 1.1.3, 1.3.4

**Why this lab:** "the client may retry the same request, and it must not double-charge" is one of the highest-value patterns on the exam because it appears in many different scenario wordings. Running it, and seeing the *second* invocation succeed without creating a second item, fixes the mechanism (conditional write plus idempotency key) in memory better than reading about it.

This lab builds the "never double-charge on retry" piece of the [flash-sale case study in 1.1.3](1.1.3-resilience-retries-and-backoff.md#worked-case-study-a-flash-sale-order-api).

```bash
# 1. Orders table, keyed by orderId
awslocal dynamodb create-table --table-name IdempotentOrders \
  --attribute-definitions AttributeName=orderId,AttributeType=S \
  --key-schema AttributeName=orderId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

cat > idempotent_handler.py <<'EOF'
import json, boto3
from botocore.exceptions import ClientError
ddb = boto3.client("dynamodb", endpoint_url="http://host.docker.internal:4566", region_name="us-east-1")

def handler(event, context):
    body = json.loads(event.get("body") or "{}")
    idempotency_key = body["idempotencyKey"]  # client-supplied, same value on every retry of the SAME logical request
    try:
        ddb.put_item(
            TableName="IdempotentOrders",
            Item={"orderId": {"S": idempotency_key}, "item": {"S": body.get("item", "unknown")}},
            ConditionExpression="attribute_not_exists(orderId)",
        )
        return {"statusCode": 201, "body": json.dumps({"status": "created"})}
    except ClientError as e:
        if e.response["Error"]["Code"] == "ConditionalCheckFailedException":
            # Same idempotency key already exists: this IS the retry, return success without re-charging
            return {"statusCode": 200, "body": json.dumps({"status": "already processed"})}
        raise
EOF
zip idempotent_function.zip idempotent_handler.py

awslocal lambda create-function \
  --function-name create-order-idempotent \
  --runtime python3.12 \
  --handler idempotent_handler.handler \
  --zip-file fileb://idempotent_function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

# 2. Invoke it TWICE with the same idempotency key, simulating a client network retry
PAYLOAD='{"body": "{\"idempotencyKey\":\"order-abc-123\",\"item\":\"flash-sale-widget\"}"}'
awslocal lambda invoke --function-name create-order-idempotent --payload "$PAYLOAD" out1.json
cat out1.json   # expect 201 "created"
awslocal lambda invoke --function-name create-order-idempotent --payload "$PAYLOAD" out2.json
cat out2.json   # expect 200 "already processed", NOT a duplicate order

# 3. Confirm only ONE item exists in the table despite two invocations
awslocal dynamodb scan --table-name IdempotentOrders
```

If your AWS Command Line Interface (CLI) version 2 rejects the raw JavaScript Object Notation (JSON) payload, add `--cli-binary-format raw-in-base64-out` to each `lambda invoke` command.

**What to notice:** the second invocation does not error out or create a duplicate. The `ConditionExpression` makes the check and the write one atomic operation, and the handler treats the resulting `ConditionalCheckFailedException` as "already done", not as a real failure. This pattern is worth more exam points than memorizing any one service's feature list, because it shows up under many different scenario wordings. See [1.1.3 Idempotency](1.1.3-resilience-retries-and-backoff.md#idempotency-making-retries-safe).

## Lab 3: AWS SDK credential resolution, made visible

**Skills:** 1.1.5, 2.1.3

**Why this lab:** "where did these credentials actually come from" is invisible by default. This lab makes it concrete by deliberately changing which link in the chain resolves and observing the effect, instead of memorizing the chain order from a diagram. See [1.1.5 Credentials and Region](1.1.5-aws-sdks-and-service-apis.md#credentials-and-region) for the mechanics this lab exercises.

```bash
# 1. Baseline: see which source resolves with NOTHING overridden
python3 - <<'EOF'
import boto3
session = boto3.Session()
creds = session.get_credentials()
print("Resolved via:", creds.method if creds else "NONE: chain exhausted")
EOF

# 2. Force environment variables to the front of the chain
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
python3 - <<'EOF'
import boto3
session = boto3.Session()
creds = session.get_credentials()
print("Resolved via:", creds.method)   # now reports "env"
EOF

# 3. Remove the env vars and force a named profile instead (requires ~/.aws/credentials to exist)
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
mkdir -p ~/.aws
cat >> ~/.aws/credentials <<'EOF'
[floci-demo]
aws_access_key_id = test
aws_secret_access_key = test
EOF
python3 - <<'EOF'
import boto3
session = boto3.Session(profile_name="floci-demo")
creds = session.get_credentials()
print("Resolved via:", creds.method)   # now reports "shared-credentials-file"
EOF
```

**What to notice:** identical application code (`boto3.Session().get_credentials()`) reports a *different* `method` based only on what is available in the environment. No code change, only an environment change, flips which link in the chain wins. This is the mechanism behind "works on my machine, fails in Lambda or Amazon Elastic Container Service (ECS)" bugs, reproduced on purpose instead of debugged after the fact. When you finish, remove the `[floci-demo]` section from `~/.aws/credentials`.
