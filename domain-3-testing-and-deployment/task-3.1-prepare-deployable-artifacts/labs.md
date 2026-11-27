# Task 3.1 labs

These labs run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

## Lab 1: Build and push a container image to ECR

**Skills:** 3.1.2

**Why this lab:** Amazon Elastic Container Registry (ECR) lifecycle policies and scan-on-push are easy to nod along to in a README but
easy to forget to actually configure in real pipelines. Writing the lifecycle JSON yourself here
means you've seen its exact shape once before an exam question describes it in prose.

Builds the workflow from [3.1.2 Container images and ECR](3.1.2-container-images-and-ecr.md#build-tag-and-push-to-ecr).

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

# 4. Set a lifecycle policy: expire untagged images after 7 days (Task 3.1's ECR lifecycle piece)
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

## Lab 2: Externalized config read at runtime

**Skills:** 3.1.4, 3.1.1

**Why this lab:** "the same artifact must be promotable across environments without rebuilding" is
a recurring Task 3.1 scenario phrase. Deploying one literal zip twice with only an environment
variable differing proves the artifact-immutability discipline instead of just stating it as a
rule to follow.

Builds the "same artifact, different environments" discipline from
[3.1.4 Environment-specific configuration](3.1.4-environment-specific-configuration.md#the-pattern-one-artifact-configuration-per-environment)
and [3.1.1 Build once, deploy many](3.1.1-packaging-dependencies-and-configuration.md#build-once-deploy-many).

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

**What to notice:** `config_function.zip` is byte-for-byte identical in both deployments. Only the
environment variable and the parameter it then looks up differ. This is the concrete mechanism
behind Task 3.1's "externalize configuration" bullet, and it's exactly what a question describing
"the same build artifact must be promotable across environments without rebuilding" is testing.
