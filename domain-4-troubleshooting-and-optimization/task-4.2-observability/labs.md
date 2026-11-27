# Task 4.2 labs

These labs run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

The `date -d '10 minutes ago' +%s` commands below use GNU `date`. On macOS, use `date -v-10M +%s` instead.

## Lab 1: Enable X-Ray tracing and read a trace

**Skills:** 4.2.4

**Why this lab:** "active tracing isn't on by default" is a frequent exam distractor and a real
operational dead end. Enabling it, invoking the function, then pulling trace data makes
the `Mode=Active` requirement (and what happens if you forget it) something you've verified
yourself.

```bash
# 1. Create a function with active tracing enabled
cat > handler.py <<'EOF'
import boto3
from aws_xray_sdk.core import xray_recorder, patch_all
patch_all()  # auto-instruments boto3 clients as X-Ray subsegments

ddb = boto3.client("dynamodb", endpoint_url="http://host.docker.internal:4566", region_name="us-east-1")

def handler(event, context):
    ddb.get_item(TableName="Orders", Key={"orderId": {"S": "abc"}})
    return {"statusCode": 200, "body": "ok"}
EOF
zip function.zip handler.py

awslocal lambda create-function \
  --function-name traced-function \
  --runtime python3.12 \
  --handler handler.handler \
  --zip-file fileb://function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --tracing-config Mode=Active

# 2. Invoke it a few times to generate trace data
for i in 1 2 3; do awslocal lambda invoke --function-name traced-function out.json; done

# 3. Pull the trace summaries
awslocal xray get-trace-summaries \
  --start-time $(date -d '10 minutes ago' +%s) \
  --end-time $(date +%s)

# 4. Pull the full trace detail for one trace ID from the summary output above
awslocal xray batch-get-traces --trace-ids <trace-id-from-step-3>
```

**What to notice:** the `--tracing-config Mode=Active` flag is the setting that turns
tracing on. Forgetting it is why a function sometimes shows up with zero data in a real X-Ray
console, a common first debugging dead end. In a real account the execution role also needs
`xray:PutTraceSegments` and `xray:PutTelemetryRecords` (the `AWSXRayDaemonWriteAccess` managed
policy); the emulator does not enforce this, so remember it separately. The function also needs the
`aws-xray-sdk` package in its deployment package or a layer for the import to succeed. See
[4.2.4 Distributed tracing](4.2.4-distributed-tracing.md#enabling-tracing-per-service).

## Lab 2: X-Ray annotations for filterable traces

**Skills:** 4.2.4

**Why this lab:** confusing annotations with metadata is a named exam trap. Tagging the same trace
with both, then proving only the annotation is filterable, removes any ambiguity about which one
to reach for when a scenario needs traces searchable by a business field.

Builds the annotation-versus-metadata distinction from
[4.2.4 Distributed tracing](4.2.4-distributed-tracing.md#annotations-versus-metadata).

```bash
cat > annotated_handler.py <<'EOF'
from aws_xray_sdk.core import xray_recorder, patch_all
patch_all()
import boto3, json
ddb = boto3.client("dynamodb", endpoint_url="http://host.docker.internal:4566", region_name="us-east-1")

def handler(event, context):
    body = json.loads(event.get("body") or "{}")
    customer_id = body.get("customerId", "unknown")

    # In Lambda, the function segment is read-only; record annotations on a subsegment
    with xray_recorder.in_subsegment("lookup_order") as subsegment:
        subsegment.put_annotation("customerId", customer_id)    # searchable
        subsegment.put_metadata("rawRequestBody", body)          # NOT searchable, just attached context
        ddb.get_item(TableName="Orders", Key={"orderId": {"S": "abc"}})

    return {"statusCode": 200, "body": json.dumps({"customerId": customer_id})}
EOF
zip annotated_function.zip annotated_handler.py

awslocal lambda create-function --function-name annotated-lookup \
  --runtime python3.12 --handler annotated_handler.handler \
  --zip-file fileb://annotated_function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --tracing-config Mode=Active

# Invoke with two different customer IDs
awslocal lambda invoke --function-name annotated-lookup \
  --payload '{"body": "{\"customerId\":\"cust-A\"}"}' out_a.json
awslocal lambda invoke --function-name annotated-lookup \
  --payload '{"body": "{\"customerId\":\"cust-B\"}"}' out_b.json

# Filter trace summaries by the annotation: this is the capability metadata does NOT give you
awslocal xray get-trace-summaries \
  --start-time $(date -d '5 minutes ago' +%s) \
  --end-time $(date +%s) \
  --filter-expression 'annotation.customerId = "cust-A"'
```

**What to notice:** the filter expression only works because `customerId` was stored via
`put_annotation`, not `put_metadata`. Try changing the filter to reference `rawRequestBody` and
confirm it can't be filtered the same way. This is the hands-on version of the
annotation-versus-metadata exam trap. The annotation sits on a subsegment because Lambda creates
the function segment itself and your code can't modify it.
