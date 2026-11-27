# Task 3.3 labs

These labs run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

## Lab 1: Blue/green-style deployment with CodeDeploy concepts

**Skills:** 3.3.4, 3.3.6

**Why this lab:** the exam expects you to recognize named deployment configuration strings
(`LambdaCanary10Percent5Minutes`, etc.) by pattern, not operate CodeDeploy from memory. Authoring
an AppSpec and a deployment group with one of these configurations ties the name to the actual
traffic-shifting behavior it encodes.

Floci's CodeDeploy support is limited, so this lab focuses on authoring the **AppSpec** and
deployment group configuration correctly, which is the artifact-shape knowledge the exam actually tests.
Background: [3.3.4 CodeDeploy](3.3.4-codedeploy.md#lambda-compute-platform).

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
deployment configuration. The exam expects you to recognize these naming patterns
(`LambdaCanary10Percent5Minutes`, `LambdaLinear10PercentEvery1Minute`,
`LambdaAllAtOnce`) and pick the one matching a scenario's described risk tolerance, not to
memorize every parameter from scratch.

## Lab 2: CloudWatch alarm driving a deployment rollback decision

**Skills:** 3.3.7, 3.3.4

**Why this lab:** a common exam trap is assuming "automatic rollback" happens just because
something went wrong in the abstract. Building the metric filter and alarm from scratch shows
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

# 2. Create an alarm on that metric. This is the alarm CodeDeploy would associate with a deployment group
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

# 4. Check the alarm state. In a real deployment, ALARM state here is what triggers CodeDeploy's
#    automatic rollback, if the alarm is attached to the deployment group and rollback on alarm is enabled
awslocal cloudwatch describe-alarms --alarm-names checkout-high-error-rate
```

**What to notice:** nothing about this alarm is Lambda-specific or CodeDeploy-specific. It's a
plain CloudWatch metric filter and alarm. CodeDeploy's "alarm-based rollback" is a deployment
group configuration that *watches* an alarm like this one (up to 10 alarms per deployment group);
the alarm itself doesn't know or care that a deployment is in progress. That separation is
what [3.3.4 CodeDeploy](3.3.4-codedeploy.md#alarms-and-rollback-summary) and
[3.3.7 Rollback](3.3.7-rollback.md) test.

## Lab 3: ECS task definition and Fargate service

**Skills:** 3.3.2

**Why this lab:** container deployment is a new direct-test area on DVA-C03. Registering a task
definition revision and updating a service to it shows that the immutable-revision model is the ECS
equivalent of the "promote the exact tested artifact, never rebuild" discipline from Task 3.1,
expressed through a different API shape. Background:
[3.3.2 Container deployments](3.3.2-container-deployments.md#deploying-a-new-version-to-ecs).

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

**What to notice:** `register-task-definition` never overwrites the old revision. Each call
creates a new numbered revision of the same family, and `update-service` is what actually shifts
the running service to the new revision. This immutable-revision model is the ECS
equivalent of the "never rebuild the artifact, promote the exact tested one" discipline from
Task 3.1. A bad deploy is fixed by pointing the service back at the previous task definition
revision, not by re-running a build with the old code. The `latest` tag keeps the lab simple; in a
real pipeline, tag images with a version or commit SHA so each revision maps to one exact image.

## Lab 4: Comparing CodeDeploy deployment configurations side by side

**Skills:** 3.3.4, 3.3.6

**Why this lab:** reading the same deployment group's configuration swapped between all-at-once,
canary, and linear makes the pacing difference concrete rather than memorized vocabulary. Full
concept background: [3.3.6 Deployment strategies](3.3.6-deployment-strategies.md) and the
configuration tables in [3.3.4 CodeDeploy](3.3.4-codedeploy.md#compute-platforms-compared).

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
`--deployment-config-name` string. Everything else about the application and deployment group shape
is identical. This is why the exam tests recognition of the *named configuration strings*
rather than deep configuration mechanics: picking the right strategy is a one-line decision once
you know which named configuration matches the scenario's described risk tolerance.
