# Services Used in DVA-C03

A fast-reference index of every AWS service this repo documents that actually shows up in **DVA-C03**, grouped by the domain that tests it. This is derived directly from the cross-links in this cert's own domain docs — if a service is listed here, a task statement in this exam covers it. Official source of truth: [DVA-C03 Exam Guide](https://docs.aws.amazon.com/pdfs/aws-certification/latest/developer-associate-03/developer-associate-03.pdf).

> Use this page as your checklist: before exam day, you should be able to open every linked doc from memory and explain what it does, when to use it, and how it shows up in a scenario question.

**Total services/topics mapped to this cert: 85** (a service can appear in more than one domain if it's tested from multiple angles).

## Domain 1: Development with AWS Services (30%)

[Full domain guide →](../../certifications/dva-c03/domain-1-development/README.md)

| Service / Topic | Doc |
|---|---|
| AWS Amplify | [amplify.md](../../services/front-end-web-mobile/amplify.md) |
| AWS AppSync | [appsync.md](../../services/application-integration/appsync.md) |
| AWS CodeDeploy | [codedeploy.md](../../services/developer-tools/codedeploy.md) |
| AWS CodePipeline | [codepipeline.md](../../services/developer-tools/codepipeline.md) |
| AWS Identity and Access Management (IAM) | [iam.md](../../services/security-identity-compliance/iam.md) |
| AWS Lambda | [lambda.md](../../services/compute/lambda.md) |
| AWS STS & Role Switching (AssumeRole) | [sts-and-role-switching.md](../../services/security-identity-compliance/sts-and-role-switching.md) |
| AWS Step Functions | [step-functions.md](../../services/application-integration/step-functions.md) |
| Amazon API Gateway | [api-gateway.md](../../services/networking-content-delivery/api-gateway.md) |
| Amazon Aurora | [aurora.md](../../services/database/aurora.md) |
| Amazon Bedrock | [bedrock.md](../../services/machine-learning-ai/bedrock.md) |
| Amazon CloudWatch | [cloudwatch.md](../../services/management-governance/cloudwatch.md) |
| Amazon Data Firehose | [data-firehose.md](../../services/analytics/data-firehose.md) |
| Amazon DynamoDB | [dynamodb.md](../../services/database/dynamodb.md) |
| Amazon ElastiCache | [elasticache.md](../../services/database/elasticache.md) |
| Amazon Elastic Container Registry (ECR) | [ecr.md](../../services/containers/ecr.md) |
| Amazon EventBridge | [eventbridge.md](../../services/application-integration/eventbridge.md) |
| Amazon Kinesis Family (Data Streams, Data Firehose, Data Analytics) | [kinesis-family.md](../../services/analytics/kinesis-family.md) |
| Amazon Q | [amazon-q.md](../../services/machine-learning-ai/amazon-q.md) |
| Amazon RDS (Relational Database Service) | [rds.md](../../services/database/rds.md) |
| Amazon S3 (Simple Storage Service) | [s3.md](../../services/storage/s3.md) |
| Amazon SNS (Simple Notification Service) | [sns.md](../../services/application-integration/sns.md) |
| Amazon SQS (Simple Queue Service) | [sqs.md](../../services/application-integration/sqs.md) |
| Amazon VPC (Virtual Private Cloud) | [vpc.md](../../services/networking-content-delivery/vpc.md) |
| Database Selection Guide — Choosing the Right AWS Database | [database-selection-guide.md](../../services/database/database-selection-guide.md) |
| Messaging Patterns Overview — SQS, SNS, EventBridge & Step Functions Together | [messaging-patterns-overview.md](../../services/application-integration/messaging-patterns-overview.md) |

## Domain 2: Security (26%)

[Full domain guide →](../../certifications/dva-c03/domain-2-security/README.md)

| Service / Topic | Doc |
|---|---|
| AWS Certificate Manager (ACM) & Private CA | [acm-and-private-ca.md](../../services/security-identity-compliance/acm-and-private-ca.md) |
| AWS CloudTrail | [cloudtrail.md](../../services/management-governance/cloudtrail.md) |
| AWS IAM Identity Center (SSO) | [iam-identity-center-sso.md](../../services/security-identity-compliance/iam-identity-center-sso.md) |
| AWS Identity and Access Management (IAM) | [iam.md](../../services/security-identity-compliance/iam.md) |
| AWS Key Management Service (KMS) | [kms.md](../../services/security-identity-compliance/kms.md) |
| AWS Lambda | [lambda.md](../../services/compute/lambda.md) |
| AWS Organizations & Service Control Policies (SCPs) | [organizations-and-scps.md](../../services/security-identity-compliance/organizations-and-scps.md) |
| AWS Pre-Built AI Services — Overview | [ai-services-overview.md](../../services/machine-learning-ai/ai-services-overview.md) |
| AWS STS & Role Switching (AssumeRole) | [sts-and-role-switching.md](../../services/security-identity-compliance/sts-and-role-switching.md) |
| AWS Secrets Manager vs. Systems Manager Parameter Store | [secrets-manager-vs-parameter-store.md](../../services/security-identity-compliance/secrets-manager-vs-parameter-store.md) |
| AWS Security Hub & Amazon Inspector | [security-hub-and-inspector.md](../../services/security-identity-compliance/security-hub-and-inspector.md) |
| AWS Shield (Standard & Advanced) | [shield.md](../../services/security-identity-compliance/shield.md) |
| AWS WAF (Web Application Firewall) | [waf.md](../../services/security-identity-compliance/waf.md) |
| Amazon Bedrock | [bedrock.md](../../services/machine-learning-ai/bedrock.md) |
| Amazon CloudWatch | [cloudwatch.md](../../services/management-governance/cloudwatch.md) |
| Amazon Cognito | [cognito.md](../../services/security-identity-compliance/cognito.md) |
| Amazon DynamoDB | [dynamodb.md](../../services/database/dynamodb.md) |
| Amazon GuardDuty | [guardduty.md](../../services/security-identity-compliance/guardduty.md) |
| Amazon Macie | [macie.md](../../services/security-identity-compliance/macie.md) |
| Amazon Q | [amazon-q.md](../../services/machine-learning-ai/amazon-q.md) |
| Amazon S3 (Simple Storage Service) | [s3.md](../../services/storage/s3.md) |
| The AWS Shared Responsibility Model — Deep Dive | [shared-responsibility-model-deep-dive.md](../../services/security-identity-compliance/shared-responsibility-model-deep-dive.md) |
| VPC Endpoints & AWS PrivateLink | [vpc-endpoints-privatelink.md](../../services/networking-content-delivery/vpc-endpoints-privatelink.md) |

## Domain 3: Testing and Deployment (22%)

[Full domain guide →](../../certifications/dva-c03/domain-3-testing-deployment/README.md)

| Service / Topic | Doc |
|---|---|
| AWS AppConfig | [appconfig.md](../../services/management-governance/appconfig.md) |
| AWS CDK (Cloud Development Kit) | [cdk.md](../../services/management-governance/cdk.md) |
| AWS Cloud9 & AWS CloudShell | [cloud9-and-cloudshell.md](../../services/developer-tools/cloud9-and-cloudshell.md) |
| AWS CloudFormation | [cloudformation.md](../../services/management-governance/cloudformation.md) |
| AWS CodeArtifact | [codeartifact.md](../../services/developer-tools/codeartifact.md) |
| AWS CodeBuild | [codebuild.md](../../services/developer-tools/codebuild.md) |
| AWS CodeCommit | [codecommit.md](../../services/developer-tools/codecommit.md) |
| AWS CodeDeploy | [codedeploy.md](../../services/developer-tools/codedeploy.md) |
| AWS CodePipeline | [codepipeline.md](../../services/developer-tools/codepipeline.md) |
| AWS Fargate | [fargate.md](../../services/compute/fargate.md) |
| AWS Fargate | [fargate.md](../../services/containers/fargate.md) |
| AWS Identity and Access Management (IAM) | [iam.md](../../services/security-identity-compliance/iam.md) |
| AWS Lambda | [lambda.md](../../services/compute/lambda.md) |
| AWS STS & Role Switching (AssumeRole) | [sts-and-role-switching.md](../../services/security-identity-compliance/sts-and-role-switching.md) |
| AWS Secrets Manager vs. Systems Manager Parameter Store | [secrets-manager-vs-parameter-store.md](../../services/security-identity-compliance/secrets-manager-vs-parameter-store.md) |
| AWS Serverless Application Model (SAM) | [sam.md](../../services/developer-tools/sam.md) |
| Amazon API Gateway | [api-gateway.md](../../services/networking-content-delivery/api-gateway.md) |
| Amazon CloudWatch | [cloudwatch.md](../../services/management-governance/cloudwatch.md) |
| Amazon Elastic Container Registry (ECR) | [ecr.md](../../services/containers/ecr.md) |
| Amazon Elastic Container Service (ECS) | [ecs.md](../../services/containers/ecs.md) |
| Amazon Elastic Kubernetes Service (EKS) | [eks.md](../../services/containers/eks.md) |
| Containers vs. VMs vs. Serverless — A Conceptual Comparison | [containers-vs-vms-vs-serverless.md](../../services/containers/containers-vs-vms-vs-serverless.md) |

## Domain 4: Troubleshooting and Optimization (22%)

[Full domain guide →](../../certifications/dva-c03/domain-4-troubleshooting-optimization/README.md)

| Service / Topic | Doc |
|---|---|
| AWS CloudTrail | [cloudtrail.md](../../services/management-governance/cloudtrail.md) |
| AWS Config | [aws-config.md](../../services/management-governance/aws-config.md) |
| AWS Cost Management Tools | [cost-management-tools.md](../../services/management-governance/cost-management-tools.md) |
| AWS Identity and Access Management (IAM) | [iam.md](../../services/security-identity-compliance/iam.md) |
| AWS Lambda | [lambda.md](../../services/compute/lambda.md) |
| AWS Systems Manager | [systems-manager.md](../../services/management-governance/systems-manager.md) |
| AWS Trusted Advisor | [trusted-advisor.md](../../services/management-governance/trusted-advisor.md) |
| AWS Well-Architected Tool | [well-architected-tool.md](../../services/management-governance/well-architected-tool.md) |
| AWS X-Ray | [x-ray.md](../../services/developer-tools/x-ray.md) |
| Amazon API Gateway | [api-gateway.md](../../services/networking-content-delivery/api-gateway.md) |
| Amazon CloudWatch | [cloudwatch.md](../../services/management-governance/cloudwatch.md) |
| Amazon DynamoDB | [dynamodb.md](../../services/database/dynamodb.md) |
| Amazon ElastiCache | [elasticache.md](../../services/database/elasticache.md) |
| Database Selection Guide — Choosing the Right AWS Database | [database-selection-guide.md](../../services/database/database-selection-guide.md) |

## How to use this page

1. Go domain by domain (they're ordered by weight — Domain with the highest % first).
2. For each service, read its full doc once, run its Floci lab, then come back here and check it off mentally.
3. Cross-reference against the [official DVA-C03 in-scope services list](https://docs.aws.amazon.com/pdfs/aws-certification/latest/developer-associate-03/developer-associate-03.pdf) periodically — AWS updates exam guides, and a service appearing in the official list but not yet linked here is a gap to flag.
4. This page only lists services this REPO has already cross-linked from DVA-C03's domain docs — it is not a substitute for the official exam guide's appendix, which is the final authority on what's in/out of scope.
