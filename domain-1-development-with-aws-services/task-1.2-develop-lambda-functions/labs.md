# Task 1.2 labs

These labs run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

## Lab 1: Lambda + API Gateway + DynamoDB (the Domain 1 request flow)

**Skills:** 1.2.4, 1.2.2

**Why this lab:** this is the single most common request shape on the real exam: a REST API in
front of a serverless function in front of a NoSQL table. Building it once by hand turns four
abstract Task 1.1/1.3 bullet points (SDK calls, serialization, API Gateway stages, DynamoDB writes)
into one concrete, debuggable thing you've actually run.

Builds the exact flow diagrammed in [1.2.4 Lambda service integrations](1.2.4-lambda-service-integrations.md#amazon-api-gateway).

```bash
# 1. Create the DynamoDB table
awslocal dynamodb create-table \
  --table-name Orders \
  --attribute-definitions AttributeName=orderId,AttributeType=S \
  --key-schema AttributeName=orderId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# 2. Write a minimal handler (save as handler.py)
cat > handler.py <<'EOF'
import json, boto3, uuid
ddb = boto3.client("dynamodb", endpoint_url="http://host.docker.internal:4566", region_name="us-east-1")

def handler(event, context):
    body = json.loads(event.get("body") or "{}")
    order_id = str(uuid.uuid4())
    ddb.put_item(TableName="Orders", Item={
        "orderId": {"S": order_id},
        "item": {"S": body.get("item", "unknown")},
    })
    return {"statusCode": 200, "body": json.dumps({"orderId": order_id})}
EOF
zip function.zip handler.py

# 3. Create the function
awslocal lambda create-function \
  --function-name create-order \
  --runtime python3.12 \
  --handler handler.handler \
  --zip-file fileb://function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

# 4. Invoke it directly first to prove the DynamoDB write works
awslocal lambda invoke --function-name create-order \
  --payload '{"body": "{\"item\":\"widget\"}"}' response.json
cat response.json

# 5. Confirm the item landed in DynamoDB
awslocal dynamodb scan --table-name Orders

# 6. Wire up API Gateway in front of it (REST API, proxy integration)
API_ID=$(awslocal apigateway create-rest-api --name orders-api --query 'id' --output text)
ROOT_ID=$(awslocal apigateway get-resources --rest-api-id $API_ID --query 'items[0].id' --output text)
awslocal apigateway put-method --rest-api-id $API_ID --resource-id $ROOT_ID \
  --http-method POST --authorization-type NONE
awslocal apigateway put-integration --rest-api-id $API_ID --resource-id $ROOT_ID \
  --http-method POST --type AWS_PROXY --integration-http-method POST \
  --uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:000000000000:function:create-order/invocations
awslocal apigateway create-deployment --rest-api-id $API_ID --stage-name prod

# 7. Call it through the stage, like a real client would
curl -X POST "http://localhost:4566/restapis/$API_ID/prod/_user_request_/" \
  -d '{"item":"gadget"}'
```

**What to notice:** the stage name (`prod`) is part of the URL. This is API Gateway stages
([3.1.5 API Gateway stages and endpoints](../../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md)) made concrete. Try creating a second `dev` stage pointing at a different
Lambda alias and compare.

## Lab 2: Lambda layers, sharing a dependency across two functions

**Skills:** 1.2.2

**Why this lab:** layers are easy to describe ("shared code across functions") but the exam tests
whether you understand what's actually *not* in each function's own package. Building two
functions that both import the same layer-only module makes that boundary concrete instead of
theoretical.

```bash
# 1. Build a layer containing a shared "helpers" module
mkdir -p python
cat > python/order_helpers.py <<'EOF'
def format_order_id(raw_id: str) -> str:
    return f"ORD-{raw_id.upper()}"
EOF
cd python && zip -r ../layer.zip . && cd ..

LAYER_ARN=$(awslocal lambda publish-layer-version \
  --layer-name order-helpers \
  --zip-file fileb://layer.zip \
  --compatible-runtimes python3.12 \
  --query 'LayerVersionArn' --output text)
echo "Layer ARN: $LAYER_ARN"

# 2. Two separate functions, both importing the same layer
cat > func_a.py <<'EOF'
from order_helpers import format_order_id
def handler(event, context):
    return {"statusCode": 200, "body": format_order_id("abc123")}
EOF
zip func_a.zip func_a.py

cat > func_b.py <<'EOF'
from order_helpers import format_order_id
def handler(event, context):
    return {"statusCode": 200, "body": format_order_id("xyz789")}
EOF
zip func_b.zip func_b.py

for FN in func_a func_b; do
  awslocal lambda create-function \
    --function-name $FN \
    --runtime python3.12 \
    --handler $FN.handler \
    --zip-file fileb://$FN.zip \
    --role arn:aws:iam::000000000000:role/lambda-role \
    --layers $LAYER_ARN
done

# 3. Invoke both. Both successfully import the SAME layer code without bundling it twice
awslocal lambda invoke --function-name func_a out_a.json; cat out_a.json
awslocal lambda invoke --function-name func_b out_b.json; cat out_b.json
```

**What to notice:** neither function's own zip contains `order_helpers.py`; it only exists in the
layer. Update the layer version once and both functions pick up the change on their next cold
start (after you point them at the new layer version), instead of redeploying every function that
uses it.

## Lab 3: Reserved concurrency as a blast-radius limiter

**Skills:** 1.2.2

**Why this lab:** reserved concurrency is one of the exam's most common "which knob fixes this"
wrong-answer traps (it doesn't pre-warm anything, provisioned concurrency does). Setting a cap
and confirming the function genuinely can't exceed it builds the correct mental model directly,
instead of relying on memorizing the distinction from a table.

```bash
# 1. A function that simulates a slow downstream call (e.g. a legacy DB with few connections)
cat > slow_handler.py <<'EOF'
import time
def handler(event, context):
    time.sleep(2)
    return {"statusCode": 200, "body": "done"}
EOF
zip slow_function.zip slow_handler.py

awslocal lambda create-function \
  --function-name slow-downstream-caller \
  --runtime python3.12 \
  --handler slow_handler.handler \
  --zip-file fileb://slow_function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

# 2. Cap its concurrency so it can never open more than 5 connections to the imaginary downstream DB
awslocal lambda put-function-concurrency \
  --function-name slow-downstream-caller \
  --reserved-concurrent-executions 5

# 3. Confirm the setting
awslocal lambda get-function-concurrency --function-name slow-downstream-caller
```

**What to notice:** this is reserved concurrency used defensively: not to guarantee capacity for
this function, but to guarantee it *never exceeds* a downstream limit. A real AWS account would
reject the 6th concurrent invocation with a `TooManyRequestsException` once the cap is hit, which
is the exact behavior a "protect a legacy database from being overwhelmed by a serverless caller"
exam scenario is describing.

## Lab 4: Event source mapping with partial batch response (`ReportBatchItemFailures`)

**Skills:** 1.2.3, 1.2.4

**Why this lab:** this is the single most tested poll-based-trigger knob on the exam (see
[1.2.3 partial batch response](1.2.3-lambda-invocation-lifecycle-and-failures.md#partial-batch-response)
and [1.2.4 event source mappings](1.2.4-lambda-service-integrations.md#event-source-mappings-in-depth)). Running a
batch with one deliberately bad record, first without and then with the feature enabled, makes the
"whole batch retries" vs. "only the bad item retries" difference undeniable instead of abstract.

```bash
# 1. A queue and a handler that fails on exactly one "poison pill" message
awslocal sqs create-queue --queue-name batch-demo-queue

cat > batch_handler.py <<'EOF'
import json

def handler(event, context):
    batch_item_failures = []
    for record in event["Records"]:
        body = json.loads(record["body"])
        try:
            if body.get("item") == "poison-pill":
                raise ValueError("Simulated processing failure")
            print("Processed:", body["item"])
        except ValueError:
            batch_item_failures.append({"itemIdentifier": record["messageId"]})
    return {"batchItemFailures": batch_item_failures}
EOF
zip batch_function.zip batch_handler.py

awslocal lambda create-function \
  --function-name batch-processor \
  --runtime python3.12 \
  --handler batch_handler.handler \
  --zip-file fileb://batch_function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

# 2. Wire the event source mapping WITH partial batch response enabled
QUEUE_ARN=$(awslocal sqs get-queue-attributes \
  --queue-url http://localhost:4566/000000000000/batch-demo-queue \
  --attribute-names QueueArn --query 'Attributes.QueueArn' --output text)

awslocal lambda create-event-source-mapping \
  --function-name batch-processor \
  --event-source-arn "$QUEUE_ARN" \
  --batch-size 5 \
  --function-response-types ReportBatchItemFailures

# 3. Send a batch: 4 good messages, 1 poison pill
for ITEM in widget gadget poison-pill thingamajig doohickey; do
  awslocal sqs send-message --queue-url http://localhost:4566/000000000000/batch-demo-queue \
    --message-body "{\"item\":\"$ITEM\"}"
done

# 4. Tail the logs. Expect to see only "poison-pill" retried, the other 4 processed once
awslocal logs tail /aws/lambda/batch-processor --follow
```

**What to notice:** only the `messageId` of the poison-pill record ever reappears in the logs as
reprocessed. The four good records are never seen by the handler a second time. Remove the
`--function-response-types ReportBatchItemFailures` flag and repeat the same test (fresh queue) to
see the contrasting whole-batch-retries behavior described in
[1.2.3](1.2.3-lambda-invocation-lifecycle-and-failures.md#sqs-failures). The difference is
worth seeing side by side, not just reading about. On real AWS, a standard queue can still deliver
a message twice even without errors, so production handlers must be idempotent.
