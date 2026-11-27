# Domain 3 Labs — Testing and Deployment (Floci)

Hands-on exercises against the free local [Floci](../../../labs-floci/README.md) emulator.

## Lab 1 — Build and push a container image to ECR (Task 3.1)

**Why this lab:** ECR lifecycle policies and scan-on-push are easy to nod along to in a README but
easy to forget to actually configure in real pipelines — writing the lifecycle JSON yourself here
means you've seen its exact shape once before an exam question describes it in prose.

```bash
# 1. Create the repository
awslocal ecr create-repository --repository-name demo-app

# 2. Build a trivial image
cat > Dockerfile <<'EOF'
FROM public.ecr.aws/lambda/python:3.12
COPY handler.py ./
CMD ["handler.handler"]
EOF
echo 'def handler(event, context): return {"statusCode": 200, "body": "ok"}' > handler.py
docker build -t demo-app .

# 3. Tag and push to the local registry
REPO_URI=$(awslocal ecr describe-repositories --repository-names demo-app \
  --query 'repositories[0].repositoryUri' --output text)
docker tag demo-app:latest $REPO_URI:v1
docker push $REPO_URI:v1

# 4. Set a lifecycle policy — expire untagged images after 7 days (Task 3.1's ECR lifecycle piece)
cat > lifecycle-policy.json <<'EOF'
{
  "rules": [{
    "rulePriority": 1,
    "description": "Expire untagged images older than 7 days",
    "selection": {"tagStatus": "untagged", "countType": "sinceImagePushed", "countUnit": "days", "countNumber": 7},
    "action": {"type": "expire"}
  }]
}
EOF
awslocal ecr put-lifecycle-policy --repository-name demo-app --lifecycle-policy-text file://lifecycle-policy.json
```

## Lab 2 — Validate and deploy with SAM (Task 3.2 + 3.3)

**Why this lab:** Task 3.2 is specifically about validating *before* anything is deployed — running
`sam validate` and `sam local invoke` back to back shows you two distinct pre-deployment checks
(template correctness, handler logic) that happen with zero live infrastructure involved.

```bash
# 1. Minimal SAM template
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

# 2. Validate BEFORE deploying — this is Task 3.2's "validate infrastructure templates" skill
sam validate --template template.yaml

# 3. Build and deploy against Floci
sam build
sam deploy --guided --resolve-s3 \
  --config-env floci \
  --no-confirm-changeset

# 4. Invoke a test event locally without a live API Gateway in front of it (Task 3.2 mock/test events)
cat > event.json <<'EOF'
{"httpMethod": "GET", "path": "/hello", "headers": {}}
EOF
sam local invoke HelloFunction --event event.json
```

**What to notice:** step 2 (`sam validate`) and step 4 (`sam local invoke` with a test event) are
both things that happen *before* anything touches a real/emulated environment — exactly what
Task 3.2 means by testing before deployment.

## Lab 3 — Blue/Green-style deployment with CodeDeploy concepts (Task 3.3)

**Why this lab:** the exam expects you to recognize named deployment-config strings
(`LambdaCanary10Percent5Minutes`, etc.) by pattern, not operate CodeDeploy from memory — authoring
an AppSpec and a deployment group with one of these configs ties the name to the actual traffic-
shifting behavior it encodes.

Floci's CodeDeploy support is limited, so this lab focuses on authoring the **AppSpec** and
deployment-group configuration correctly — the artifact-shape knowledge the exam actually tests.

```bash
# 1. AppSpec for a Lambda deployment (traffic shifting between function versions/aliases)
cat > appspec.yaml <<'EOF'
version: 0.0
Resources:
  - HelloFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: "hello-function"
        Alias: "live"
        CurrentVersion: "1"
        TargetVersion: "2"
Hooks:
  - BeforeAllowTraffic: "PreTrafficHookFunction"
  - AfterAllowTraffic: "PostTrafficHookFunction"
EOF

# 2. Create a deployment group with a canary traffic-shifting config
awslocal deploy create-application --application-name demo-app --compute-platform Lambda

awslocal deploy create-deployment-group \
  --application-name demo-app \
  --deployment-group-name canary-10-percent-5-minutes \
  --deployment-config-name CodeDeployDefault.LambdaCanary10Percent5Minutes \
  --service-role-arn arn:aws:iam::000000000000:role/CodeDeployServiceRole

# 3. Trigger the deployment using the appspec above
awslocal deploy create-deployment \
  --application-name demo-app \
  --deployment-group-name canary-10-percent-5-minutes \
  --revision '{"revisionType": "AppSpecContent", "appSpecContent": {"content": "'"$(cat appspec.yaml | sed 's/"/\\"/g')"'"}}'
```

**What to notice:** `CodeDeployDefault.LambdaCanary10Percent5Minutes` is a *named, predefined*
deployment config — the exam expects you to recognize these naming patterns
(`LambdaCanary10Percent5Minutes`, `LambdaLinear10PercentEvery1Minute`,
`LambdaAllAtOnce`) and pick the one matching a scenario's described risk tolerance, not to
memorize every parameter from scratch.

## Lab 4 — Externalized config read at runtime (Task 3.1)

**Why this lab:** "the same artifact must be promotable across environments without rebuilding" is
a recurring Task 3.1 scenario phrase — deploying one literal zip twice with only an environment
variable differing proves the artifact-immutability discipline instead of just stating it as a
rule to follow.

Builds the "same artifact, different environments" discipline from the
[README's Task 3.1 diagram](README.md#task-31-in-one-picture-what-actually-goes-into-a-deployable-artifact).

```bash
# 1. Store different config per environment, same parameter path pattern
awslocal ssm put-parameter --name /staging/app/db-host --value "staging-db.local" --type String
awslocal ssm put-parameter --name /prod/app/db-host --value "prod-db.local" --type String

# 2. ONE Lambda function, reads its environment from an env var, then reads the matching parameter
cat > config_handler.py <<'EOF'
import os, boto3
ssm = boto3.client("ssm", endpoint_url="http://host.docker.internal:4566", region_name="us-east-1")

def handler(event, context):
    env = os.environ.get("APP_ENV", "staging")  # set per-deployment, NOT baked into the code
    param = ssm.get_parameter(Name=f"/{env}/app/db-host")
    return {"statusCode": 200, "body": f"Connecting to: {param['Parameter']['Value']}"}
EOF
zip config_function.zip config_handler.py

# 3. Deploy the SAME zip twice, differing only by the APP_ENV environment variable
awslocal lambda create-function --function-name app-staging \
  --runtime python3.12 --handler config_handler.handler \
  --zip-file fileb://config_function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --environment "Variables={APP_ENV=staging}"

awslocal lambda create-function --function-name app-prod \
  --runtime python3.12 --handler config_handler.handler \
  --zip-file fileb://config_function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --environment "Variables={APP_ENV=prod}"

# 4. Confirm each resolves to its own environment's config, despite being the SAME deployed code
awslocal lambda invoke --function-name app-staging out_staging.json; cat out_staging.json
awslocal lambda invoke --function-name app-prod out_prod.json; cat out_prod.json
```

**What to notice:** `config_function.zip` is byte-for-byte identical in both deployments — only the
environment variable and the parameter it then looks up differ. This is the concrete mechanism
behind Task 3.1's "externalize configuration" bullet, and it's exactly what a question describing
"the same build artifact must be promotable across environments without rebuilding" is testing.

## Lab 5 — CloudWatch alarm driving a deployment rollback decision (Task 3.3)

**Why this lab:** a common exam trap is assuming "automatic rollback" happens just because
something went wrong in the abstract — building the metric filter and alarm from scratch shows
it's a specific, must-be-configured CloudWatch resource that CodeDeploy merely watches, not a
built-in deployment behavior.

```bash
# 1. Create a metric filter that counts ERROR log lines as a custom metric
awslocal logs put-metric-filter \
  --log-group-name /aws/lambda/checkout-service \
  --filter-name error-count \
  --filter-pattern "ERROR" \
  --metric-transformations \
    metricName=CheckoutErrors,metricNamespace=CheckoutApp,metricValue=1,defaultValue=0

# 2. Create an alarm on that metric — this is the alarm CodeDeploy would associate with a deployment group
awslocal cloudwatch put-metric-alarm \
  --alarm-name checkout-high-error-rate \
  --namespace CheckoutApp \
  --metric-name CheckoutErrors \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold

# 3. Simulate error log lines that would breach the alarm
cat > error_handler.py <<'EOF'
import logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)
def handler(event, context):
    logger.error("ERROR: payment gateway timeout")
    return {"statusCode": 500}
EOF
zip error_function.zip error_handler.py
awslocal lambda create-function --function-name checkout-service \
  --runtime python3.12 --handler error_handler.handler \
  --zip-file fileb://error_function.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

for i in $(seq 1 6); do awslocal lambda invoke --function-name checkout-service out.json; done

# 4. Check the alarm state — in a real deployment, ALARM state here is what triggers CodeDeploy's
#    automatic rollback, per the README's "Rollback mechanics" section
awslocal cloudwatch describe-alarms --alarm-names checkout-high-error-rate
```

**What to notice:** nothing about this alarm is Lambda-specific or CodeDeploy-specific — it's a
plain CloudWatch metric filter + alarm. CodeDeploy's "alarm-based rollback" is just a deployment
group configuration that *watches* an alarm like this one; the alarm itself doesn't know or care
that a deployment is in progress. Understanding that separation is exactly what the README's
[Rollback mechanics](README.md#rollback-mechanics-how-automatic-rollback-actually-triggers)
section is testing.

## Lab 6 — ECS task definition and Fargate service (Task 3.3's container emphasis)

**Why this lab:** container deployment is a new direct-test area on DVA-C03 — registering a task
definition revision and updating a service to it shows the immutable-revision model is the ECS
equivalent of the "promote the exact tested artifact, never rebuild" discipline from Task 3.1,
just expressed through a different API shape.

```bash
# 1. Register a task definition for the checkout microservice from the case study
cat > task-def.json <<'EOF'
{
  "family": "checkout-service",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [{
    "name": "checkout",
    "image": "public.ecr.aws/docker/library/httpd:latest",
    "portMappings": [{"containerPort": 80, "protocol": "tcp"}],
    "environment": [{"name": "APP_ENV", "value": "staging"}]
  }]
}
EOF
awslocal ecs register-task-definition --cli-input-json file://task-def.json

# 2. Create a cluster and a Fargate service running that task definition
awslocal ecs create-cluster --cluster-name demo-cluster

awslocal ecs create-service \
  --cluster demo-cluster \
  --service-name checkout-service \
  --task-definition checkout-service \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-12345],securityGroups=[sg-12345],assignPublicIp=ENABLED}"

# 3. Confirm the service is running the expected number of tasks
awslocal ecs describe-services --cluster demo-cluster --services checkout-service

# 4. Register a NEW task definition revision (simulating a deploy) and update the service to it
sed 's/staging/prod/' task-def.json > task-def-v2.json
awslocal ecs register-task-definition --cli-input-json file://task-def-v2.json
awslocal ecs update-service --cluster demo-cluster --service checkout-service \
  --task-definition checkout-service
```

**What to notice:** `register-task-definition` never overwrites the old revision — each call
creates a new numbered revision of the same family, and `update-service` is what actually shifts
the running service to the new revision. This immutable-revision model is exactly the ECS
equivalent of the "never rebuild the artifact, promote the exact tested one" discipline from
Task 3.1 — a bad deploy is "point the service back at the previous task definition revision," not
"re-run a build with the old code."

## Lab 7 — `cdk diff`-style drift simulation and remediation (Task 3.2)

**Why this lab:** drift detection is easy to understand in the abstract and easy to miss in
practice — manually breaking a stack's resource out-of-band, then running drift detection yourself,
builds the muscle memory for recognizing *why* a stack shows `DRIFTED` before the exam describes it
in prose. Full concept background: [`iac-drift-detection-and-remediation.md`](iac-drift-detection-and-remediation.md).

```bash
# 1. Deploy a simple stack with a security group via SAM/CloudFormation
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

# 3. Detect drift — this stack should now report DRIFTED
awslocal cloudformation detect-stack-drift --stack-name drift-lab-stack
awslocal cloudformation describe-stack-resource-drifts --stack-name drift-lab-stack

# 4. Remediate by re-applying the template (CloudFormation reasserts the declared state,
#    removing the out-of-band port 22 rule it doesn't know about)
awslocal cloudformation deploy --template-file drift-template.yaml --stack-name drift-lab-stack
awslocal cloudformation detect-stack-drift --stack-name drift-lab-stack
```

**What to notice:** step 4 doesn't require you to manually find and revert the port 22 rule —
simply re-deploying the unchanged template is enough, because CloudFormation treats the template
as the single source of truth and reconciles any resource back to match it. This is the core
remediation principle: **fix drift by re-applying IaC, never by manually editing the drifted
resource a second time** (which would just be more untracked drift).

## Lab 8 — Unit and integration tests for a Lambda handler with mocked AWS services (Task 3.2)

**Why this lab:** the exam expects you to recognize the difference between a unit test (no AWS
calls at all) and an integration test (talks to a real or emulated AWS service) — writing one of
each against the same handler makes the boundary concrete instead of definitional. Full concept
background: [`testing-strategies-for-serverless-apps.md`](testing-strategies-for-serverless-apps.md).

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
# test_unit.py — UNIT test: no AWS call at all, pure logic in isolation
from unittest.mock import MagicMock
import handler

def test_returns_404_when_item_missing(monkeypatch):
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
# test_integration.py — INTEGRATION test: real DynamoDB API calls against Floci
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
# Run both layers separately — this separation is itself the testable skill
pytest test_unit.py -v          # milliseconds, no network, no Floci needed
awslocal_endpoint=http://localhost:4566 pytest test_integration.py -v   # seconds, needs Floci running
```

**What to notice:** `test_unit.py` never imports `pytest.fixture` requiring a live endpoint and
runs in milliseconds even with Floci stopped — this is what "many, fast, isolated" at the base of
the testing pyramid means concretely. `test_integration.py` requires Floci running and is
meaningfully slower, which is exactly why you want fewer of these and many more of the first kind.

## Lab 9 — Comparing CodeDeploy deployment configs side by side (Task 3.3)

**Why this lab:** reading the same deployment group's config swapped between all-at-once, canary,
and linear makes the actual pacing difference concrete rather than memorized vocabulary. Full
concept background: [`blue-green-canary-linear-deployment-strategies.md`](blue-green-canary-linear-deployment-strategies.md).

```bash
# Create the same Lambda application three times, each with a different deployment config,
# and compare the deployment-group JSON shape side by side
for CONFIG in LambdaAllAtOnce LambdaCanary10Percent5Minutes LambdaLinear10PercentEvery1Minute; do
  awslocal deploy create-application --application-name "demo-$CONFIG" --compute-platform Lambda
  awslocal deploy create-deployment-group \
    --application-name "demo-$CONFIG" \
    --deployment-group-name "dg-$CONFIG" \
    --deployment-config-name "CodeDeployDefault.$CONFIG" \
    --service-role-arn arn:aws:iam::000000000000:role/CodeDeployServiceRole
  echo "--- $CONFIG ---"
  awslocal deploy get-deployment-group --application-name "demo-$CONFIG" --deployment-group-name "dg-$CONFIG" \
    --query 'deploymentGroupInfo.deploymentConfigName'
done
```

**What to notice:** the only thing that differs across all three calls is the single
`--deployment-config-name` string — everything else about the application/deployment-group shape
is identical. This is precisely why the exam tests recognition of the *named config strings*
rather than deep configuration mechanics: picking the right strategy is a one-line decision once
you know which named config matches the scenario's described risk tolerance.

## Lab wrap-up and cross-links

| Lab | Task(s) reinforced | README section |
|---|---|---|
| 1 | 3.1 (ECR images, lifecycle policies) | [Task 3.1 in one picture](README.md#task-31-in-one-picture-what-actually-goes-into-a-deployable-artifact) |
| 2 | 3.2 (template validation, mock events), 3.3 (deployment) | [Task 3.2 in practice](README.md#task-32-in-practice-validating-before-you-ever-deploy) |
| 3 | 3.3 (CodeDeploy configs, canary) | [Deployment strategies compared](README.md#deployment-strategies-compared) |
| 4 | 3.1 (externalized config) | [Task 3.1 in one picture](README.md#task-31-in-one-picture-what-actually-goes-into-a-deployable-artifact) |
| 5 | 3.3 (alarm-based rollback) | [Rollback mechanics](README.md#rollback-mechanics-how-automatic-rollback-actually-triggers) |
| 6 | 3.3 (ECS/Fargate deployment) | [Full worked case study](README.md#full-worked-case-study-cicd-for-a-containerized-checkout-microservice) |
| 7 | 3.2 (drift detection and remediation) | [IaC drift](README.md#iac-drift-the-validation-story-doesnt-end-at-deploy-time) |
| 8 | 3.2 (unit vs. integration testing pyramid) | [Testing pyramid](README.md#testing-pyramid-for-serverless-and-container-apps) |
| 9 | 3.3 (CodeDeploy named deployment configs) | [Deployment strategy choice](README.md#deployment-strategy-choice-with-real-codedeploy-configuration-names) |

For the fullest picture of how all nine labs connect into one pipeline, re-read the
[case study](README.md#full-worked-case-study-cicd-for-a-containerized-checkout-microservice)
after finishing Lab 9 — every piece of that architecture diagram has now been exercised locally at
least once, just not wired together into a single literal CodePipeline (which Floci doesn't fully
emulate end to end).
