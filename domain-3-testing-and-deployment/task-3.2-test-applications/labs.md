# Task 3.2 labs

These labs run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

## Lab 1: Validate and invoke with SAM before deploying

**Skills:** 3.2.1, 3.2.3

**Why this lab:** Task 3.2 is about validating *before* anything is deployed. Running
`sam validate` and `sam local invoke` back to back shows you two distinct pre-deployment checks
(template correctness, handler logic) that happen with zero live infrastructure involved.
Background: [3.2.1 Testing with SAM, CDK, and CloudFormation](3.2.1-testing-with-sam-cdk-and-cloudformation.md#aws-sam-test-locally-and-in-the-cloud) and [3.2.3 Test events and payloads](3.2.3-test-events-and-payloads.md#generate-and-use-events-with-aws-sam).

```bash
# 1. Minimal SAM template and the handler it references
cat > template.yaml <<'EOF'
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  HelloFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: handler.handler
      Runtime: python3.12
      CodeUri: .
      Events:
        Api:
          Type: Api
          Properties:
            Path: /hello
            Method: get
EOF

cat > handler.py <<'EOF'
import json
def handler(event, context):
    return {"statusCode": 200, "body": json.dumps({"path": event.get("path")})}
EOF

# 2. Validate BEFORE deploying: this is Task 3.2's "validate infrastructure templates" skill
sam validate --template template.yaml

# 3. Build and deploy against Floci
sam build
sam deploy --guided --resolve-s3 \
  --config-env floci \
  --no-confirm-changeset

# 4. Invoke a test event locally without a live API Gateway in front of it (Task 3.2 test events)
cat > event.json <<'EOF'
{"httpMethod": "GET", "path": "/hello", "headers": {}}
EOF
sam local invoke HelloFunction --event event.json
```

**What to notice:** step 2 (`sam validate`) and step 4 (`sam local invoke` with a test event) are
both things that happen *before* anything touches a real or emulated environment, exactly what
Task 3.2 means by testing before deployment. `sam local invoke` needs Docker (or another container
runtime) because it runs the handler in a local Lambda-like container.

## Lab 2: Drift simulation and remediation

**Skills:** 3.2.1

**Why this lab:** drift detection is easy to understand in the abstract and easy to miss in
practice. Manually breaking a stack's resource out of band, then running drift detection yourself,
builds the muscle memory for recognizing *why* a stack shows `DRIFTED` before the exam describes it
in prose. Background: [3.2.1 Drift detection](3.2.1-testing-with-sam-cdk-and-cloudformation.md#drift-detection-validation-after-deployment).

```bash
# 1. Deploy a simple stack with a security group via CloudFormation
cat > drift-template.yaml << 'EOF'
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  DemoSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Demo SG for drift lab"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
EOF
awslocal cloudformation deploy --template-file drift-template.yaml --stack-name drift-lab-stack

# 2. Simulate an out-of-band console change: someone opens port 22 "just for now"
SG_ID=$(awslocal cloudformation describe-stack-resources --stack-name drift-lab-stack \
  --query "StackResources[?LogicalResourceId=='DemoSecurityGroup'].PhysicalResourceId" --output text)
awslocal ec2 authorize-security-group-ingress --group-id $SG_ID \
  --protocol tcp --port 22 --cidr 0.0.0.0/0

# 3. Detect drift: this stack should now report DRIFTED
awslocal cloudformation detect-stack-drift --stack-name drift-lab-stack
awslocal cloudformation describe-stack-resource-drifts --stack-name drift-lab-stack

# 4. Try re-deploying the unchanged template: CloudFormation reports no changes,
#    because it compares the template with the previous template, not with live state
awslocal cloudformation deploy --template-file drift-template.yaml --stack-name drift-lab-stack

# 5. Revert the drift with a drift-aware change set
awslocal cloudformation create-change-set --stack-name drift-lab-stack \
  --change-set-name revert-drift --template-body file://drift-template.yaml \
  --deployment-mode REVERT_DRIFT
awslocal cloudformation describe-change-set --stack-name drift-lab-stack --change-set-name revert-drift
awslocal cloudformation execute-change-set --stack-name drift-lab-stack --change-set-name revert-drift

# 6. Confirm the stack is back IN_SYNC
awslocal cloudformation detect-stack-drift --stack-name drift-lab-stack
awslocal cloudformation describe-stack-resource-drifts --stack-name drift-lab-stack
```

**What to notice:** step 4 does not remove the port 22 rule. A normal deployment of an unchanged
template is a no-op, so drift survives it. Step 5 uses a drift-aware change set, which compares the
live state, the previous deployment, and the template, and brings the security group back to the
template's single port 443 rule. The other remediation path is to adopt the change: if port 22 was
an intended, correct change, edit the template to include it and deploy that. Either way, fix drift
through IaC (infrastructure as code), not by editing the drifted resource by hand a second time,
which would just be more untracked drift.

If Floci does not support drift detection or `--deployment-mode REVERT_DRIFT`, run steps 5 and 6 in
a sandbox AWS account with the regular `aws` CLI (verify).

## Lab 3: Unit and integration tests for a Lambda handler with mocked AWS services

**Skills:** 3.2.1, 3.2.2, 3.2.3

**Why this lab:** the exam expects you to recognize the difference between a unit test (no AWS
calls at all) and an integration test (talks to a real or emulated AWS service). Writing one of
each against the same handler makes the boundary concrete instead of definitional. Background:
[3.2.1 The testing pyramid](3.2.1-testing-with-sam-cdk-and-cloudformation.md#the-testing-pyramid-for-serverless-apps) and [3.2.2 Mock AWS SDK calls](3.2.2-mock-apis.md#mock-aws-sdk-calls-in-unit-tests).

```python
# handler.py
import boto3, os
def handler(event, context):
    table = boto3.resource("dynamodb").Table(os.environ["TABLE_NAME"])
    item_id = event["pathParameters"]["id"]
    resp = table.get_item(Key={"id": item_id})
    if "Item" not in resp:
        return {"statusCode": 404, "body": "not found"}
    return {"statusCode": 200, "body": str(resp["Item"])}
```

```python
# test_unit.py: UNIT test, no AWS call at all, pure logic in isolation
from unittest.mock import MagicMock
import handler

def test_returns_404_when_item_missing(monkeypatch):
    monkeypatch.setenv("TABLE_NAME", "orders-test")
    fake_table = MagicMock()
    fake_table.get_item.return_value = {}  # no "Item" key
    fake_resource = MagicMock()
    fake_resource.Table.return_value = fake_table
    monkeypatch.setattr("boto3.resource", lambda *_: fake_resource)
    event = {"pathParameters": {"id": "missing-id"}}
    result = handler.handler(event, None)
    assert result["statusCode"] == 404
```

```python
# test_integration.py: INTEGRATION test, real DynamoDB API calls against Floci
import boto3, os, pytest
import handler

@pytest.fixture(scope="module")
def dynamodb_table():
    ddb = boto3.client("dynamodb", endpoint_url="http://localhost:4566", region_name="us-east-1")
    ddb.create_table(
        TableName="orders-test", KeySchema=[{"AttributeName": "id", "KeyType": "HASH"}],
        AttributeDefinitions=[{"AttributeName": "id", "AttributeType": "S"}],
        BillingMode="PAY_PER_REQUEST")
    ddb.put_item(TableName="orders-test", Item={"id": {"S": "abc"}, "item": {"S": "widget"}})
    os.environ["TABLE_NAME"] = "orders-test"
    yield
    ddb.delete_table(TableName="orders-test")

def test_returns_200_for_real_item(dynamodb_table):
    event = {"pathParameters": {"id": "abc"}}
    result = handler.handler(event, None)
    assert result["statusCode"] == 200
```

```bash
# Run both layers separately: this separation is itself the testable skill
pytest test_unit.py -v          # milliseconds, no network, no Floci needed

# AWS_ENDPOINT_URL points every boto3 client (including the one inside handler.py) at Floci
AWS_ENDPOINT_URL=http://localhost:4566 AWS_DEFAULT_REGION=us-east-1 \
AWS_ACCESS_KEY_ID=test AWS_SECRET_ACCESS_KEY=test \
  pytest test_integration.py -v   # seconds, needs Floci running
```

**What to notice:** `test_unit.py` has no fixture that needs a live endpoint and runs in
milliseconds even with Floci stopped. This is what "many, fast, isolated" at the base of the
testing pyramid means concretely. `test_integration.py` requires Floci running and is
meaningfully slower, which is exactly why you want fewer of these and many more of the first kind.
Without `AWS_ENDPOINT_URL`, the handler's own `boto3.resource("dynamodb")` call would go to real
AWS instead of Floci.
