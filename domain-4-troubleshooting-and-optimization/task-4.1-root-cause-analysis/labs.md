# Task 4.1 labs

These labs run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

The `date -d '1 hour ago'` commands below use GNU `date`. On macOS, use `date -v-1H +%s` for one hour ago and `date -v-10M +%s` for ten minutes ago.

## Lab 1: CloudWatch Logs Insights query across an incident

**Skills:** 4.1.3, 4.1.1

**Why this lab:** Task 4.1 expects you to quantify a failure ("how often is this actually failing")
before chasing a root cause. Writing a real Logs Insights query against generated log data
practices that discipline instead of letting you jump straight to the first error you see.

```bash
# 1. Generate some log activity by invoking a function that logs structured JavaScript Object Notation (JSON)
cat > loghandler.py <<'EOF'
import json, logging, random
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def handler(event, context):
    request_id = context.aws_request_id
    if random.random() < 0.3:
        logger.error(json.dumps({"requestId": request_id, "level": "ERROR", "msg": "payment declined"}))
        return {"statusCode": 500}
    logger.info(json.dumps({"requestId": request_id, "level": "INFO", "msg": "order processed"}))
    return {"statusCode": 200}
EOF
zip logfunction.zip loghandler.py
awslocal lambda create-function \
  --function-name order-processor \
  --runtime python3.12 --handler loghandler.handler \
  --zip-file fileb://logfunction.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

for i in $(seq 1 20); do awslocal lambda invoke --function-name order-processor out.json; done

# 2. Run a Logs Insights-style query to find the error rate
awslocal logs start-query \
  --log-group-name /aws/lambda/order-processor \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | stats count() as errorCount'

# 3. Fetch results (poll until status is Complete)
awslocal logs get-query-results --query-id <id-from-step-2>
```

**What to notice:** this is the Task 4.1 skill in miniature. You quantify "how often is this
failing" from logs before you chase a root cause, instead of debugging the first error you
happen to spot. The query patterns are in [4.1.3 Querying logs](4.1.3-querying-logs.md#query-patterns-by-goal).

## Lab 2: Correlating one failing request across multiple log lines

**Skills:** 4.1.3, 4.1.2

**Why this lab:** [4.1.3 Querying logs](4.1.3-querying-logs.md#query-patterns-by-goal) and
[4.2.1 Structured logging](../task-4.2-observability/4.2.1-structured-logging.md) explain the
mechanism. This lab makes the query pattern something you have run against real generated
data, including the failure mode of a function that *doesn't* propagate a correlation ID, so the
difference is concrete rather than theoretical.

```bash
# 1. Deploy a handler that DOES propagate a correlation ID through every log line
cat > correlated_handler.py <<'EOF'
import json, logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def handler(event, context):
    correlation_id = event.get("correlationId", context.aws_request_id)
    logger.info(json.dumps({"correlationId": correlation_id, "event": "order_received"}))
    logger.info(json.dumps({"correlationId": correlation_id, "event": "inventory_checked", "inStock": True}))
    logger.info(json.dumps({"correlationId": correlation_id, "event": "payment_attempted"}))
    logger.error(json.dumps({"correlationId": correlation_id, "event": "payment_failed", "error": "card_declined"}))
    return {"statusCode": 500}
EOF
zip correlated_function.zip correlated_handler.py
awslocal lambda create-function --function-name correlated-order \
  --runtime python3.12 --handler correlated_handler.handler \
  --zip-file fileb://correlated_function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

# 2. Invoke it with a known correlation ID, simulating one customer's one request
awslocal lambda invoke --function-name correlated-order \
  --payload '{"correlationId": "order-9981-trace"}' out.json

# 3. Reconstruct that ONE request's full story, in order, from the log group
awslocal logs start-query \
  --log-group-name /aws/lambda/correlated-order \
  --start-time $(date -d '10 minutes ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter correlationId = "order-9981-trace" | sort @timestamp asc'

awslocal logs get-query-results --query-id <id-from-step-3>
```

On AWS Command Line Interface (AWS CLI) version 2, add `--cli-binary-format raw-in-base64-out` to the `invoke` command in step 2 so the JSON payload is sent as text.

**What to notice:** the query in step 3 only works *because* every log line in step 1 explicitly
included `correlationId`. Delete that field from even one `logger` call and that line silently
drops out of the reconstructed story with no error raised. This quiet, easy-to-introduce
regression is why [Task 4.2](../task-4.2-observability/README.md) asks you to instrument before the incident.

## Lab 3: Reading the `Init Duration` field to prove a cold start

**Skills:** 4.1.2

**Why this lab:** [4.1.2 Metrics, logs, and traces](4.1.2-metrics-logs-and-traces.md#cold-start-or-slow-code)
explains that `Init Duration` in a `REPORT` log line is the definitive signal of a cold start.
This lab produces that log line and compares it with a warm invocation's `REPORT` line, so the
distinction becomes something you have read yourself.

```bash
# 1. Create a fresh function (its very first invocation is guaranteed cold)
cat > coldhandler.py <<'EOF'
import time
# This sleep simulates expensive static initialization work, such as loading a large
# config file or ML model, that only runs once per NEW execution environment.
time.sleep(0.5)

def handler(event, context):
    return {"statusCode": 200, "body": "warm now"}
EOF
zip coldfunction.zip coldhandler.py
awslocal lambda create-function --function-name cold-start-demo \
  --runtime python3.12 --handler coldhandler.handler \
  --zip-file fileb://coldfunction.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

# 2. First invocation: this is cold. Pull its logs and look for Init Duration.
awslocal lambda invoke --function-name cold-start-demo --log-type Tail out1.json \
  --query 'LogResult' --output text | base64 --decode

# 3. Immediately invoke again: this should reuse the already-initialized environment (warm)
awslocal lambda invoke --function-name cold-start-demo --log-type Tail out2.json \
  --query 'LogResult' --output text | base64 --decode
```

**What to notice:** compare the two `REPORT` lines side by side. The first should include an
`Init Duration` field of roughly 500 ms or more; the second should have **no** `Init Duration`
field at all. That is the diagnostic signal described in
[4.1.2](4.1.2-metrics-logs-and-traces.md#reading-the-lambda-report-line), now something you have
produced and read yourself.

## Lab 4: AWS Config drift detection paired with CloudTrail

**Skills:** 4.1.1

> **Not on the DVA-C03 service list.** AWS Config is used here only to show the contrast with CloudTrail. The exam-relevant half of this lab is the CloudTrail lookup in step 4.

**Why this lab:** Config and CloudTrail answer different questions about the same change, and
that is easy to blur in the abstract. Drifting a security group and then querying both services
for the *same* event shows which tool answers "what changed" and which answers "who changed it."

Builds the Config and CloudTrail pairing described in [4.1.1 Debugging code and integrations](4.1.1-debugging-code-and-integrations.md#finding-what-changed-with-cloudtrail).

```bash
# 1. Create a security group in a known-good state
SG_ID=$(awslocal ec2 create-security-group --group-name demo-sg --description "demo" \
  --query 'GroupId' --output text)

# 2. Start recording its configuration with AWS Config
awslocal configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::000000000000:role/config-role
awslocal configservice start-configuration-recorder --configuration-recorder-name default

# 3. "Drift" the security group: open a port to the world (the kind of change that should get flagged)
awslocal ec2 authorize-security-group-ingress --group-id $SG_ID \
  --protocol tcp --port 22 --cidr 0.0.0.0/0

# 4. Confirm CloudTrail has the discrete event for WHO/WHICH call made the change
awslocal cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AuthorizeSecurityGroupIngress

# 5. Confirm Config has the resulting configuration state change in its timeline
awslocal configservice get-resource-config-history \
  --resource-type AWS::EC2::SecurityGroup --resource-id $SG_ID
```

**What to notice:** steps 4 and 5 answer two different questions about the same change.
CloudTrail's event is the discrete "this API call happened at this time, by this principal."
Config's history is the resulting "here is how this resource's configuration looked over time."
A real investigation often needs both: Config to find when the resource drifted, and CloudTrail
to find who made the call.
