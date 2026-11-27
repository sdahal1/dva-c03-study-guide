# Services in scope

**Foundations** · [Folder overview](README.md) · Related: [Exam guide](exam-guide.md) · [Glossary](glossary.md) · [AWS service basics](aws-service-basics.md)

The official DVA-C03 in-scope service list, mapped to the skill notes where each service matters. The first link in each row is the note with the fullest treatment. AWS calls the list non-exhaustive and subject to change, and the [exam guide](exam-guide.md#in-scope-aws-services) is the final word on scope.

Abbreviations in the tables are spelled out in the [glossary](glossary.md).

Use this page as a checklist. Before exam day you should be able to say what each service does, when a scenario calls for it, and which alternatives it beats.

## How to read the table

- **Skill notes** lists skill numbers in order of importance for that service. A service that appears in many skills (Lambda, DynamoDB, API Gateway, IAM) gets tested from several angles.
- **Service basics** points to [aws-service-basics.md](aws-service-basics.md), which covers background material for EC2, EBS, EFS, ELB, Auto Scaling, Route 53, and VPC that no single skill owns.
- The exam uses short service names (Amazon SNS, not Amazon Simple Notification Service). The in-exam Help feature lists the full names.

## Analytics

| Service | What it is | Skill notes |
|---|---|---|
| **Amazon Athena** | Serverless SQL queries over data in S3, including CloudTrail, ALB, and VPC Flow Logs | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [4.1.3](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.3-querying-logs.md), [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md) |
| **Amazon Data Firehose** | Managed delivery of streaming data to S3, OpenSearch, and other destinations, with optional Lambda transformation | [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md), [1.1.7](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.7-serialization-and-deserialization.md), [4.1.3](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.3-querying-logs.md) |
| **Amazon Kinesis** | Kinesis Data Streams: sharded, ordered, replayable streams consumed by Lambda or the KCL | [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md), [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md), [1.2.3](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.3-lambda-invocation-lifecycle-and-failures.md), [1.1.7](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.7-serialization-and-deserialization.md) |
| **Amazon OpenSearch Service** | Search and log analytics; also a vector store for RAG | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md), [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md) |

## Application integration

| Service | What it is | Skill notes |
|---|---|---|
| **Amazon EventBridge** | Event bus with content-based rules, Pipes, and Scheduler; receives AWS service events such as CodeDeploy state changes | [1.1.8](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.8-event-driven-applications.md), [1.1.2](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.2-communication-patterns.md), [1.1.1](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.1-coupling-and-decoupling.md), [4.2.3](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.3-alerts-and-notifications.md) |
| **Amazon SNS** | Pub/sub topics, fan-out to SQS and Lambda, filter policies | [1.1.8](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.8-event-driven-applications.md), [4.3.3](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.3-messaging-optimization.md), [1.1.1](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.1-coupling-and-decoupling.md), [4.2.3](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.3-alerts-and-notifications.md) |
| **Amazon SQS** | Standard and FIFO queues, visibility timeout, DLQs, long polling | [1.1.1](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.1-coupling-and-decoupling.md), [1.1.2](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.2-communication-patterns.md), [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md), [4.3.3](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.3-messaging-optimization.md) |
| **AWS Step Functions** | State machine workflows with Retry/Catch, task tokens, Standard and Express types | [1.1.2](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.2-communication-patterns.md), [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md), [2.3.4](../domain-2-security/task-2.3-secure-ai-services/2.3.4-ai-agent-security.md), [4.1.1](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.1-debugging-code-and-integrations.md) |

## Compute

| Service | What it is | Skill notes |
|---|---|---|
| **Amazon EC2** | Virtual machines, instance profiles, IMDSv2, key pairs; target for CodeDeploy and Auto Scaling | [Service basics](aws-service-basics.md#amazon-ec2), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md), [2.2.5](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.5-development-certificates-and-ssh-keys.md), [3.3.4](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.4-codedeploy.md), [4.3.1](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.1-concurrency-controls.md) |
| **AWS Elastic Beanstalk** | Managed web platform; deployment policies, `.ebextensions`, environment configuration | [3.3.6](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md), [3.1.1](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.1-packaging-dependencies-and-configuration.md), [3.1.4](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.4-environment-specific-configuration.md), [3.3.7](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.7-rollback.md), [4.1.4](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.4-diagnosing-deployment-failures.md) |
| **AWS Lambda** | Serverless functions: configuration, VPC access, invocation and failure handling, integrations, packaging, tuning | [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md), [1.2.2](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.2-lambda-configuration.md), [1.2.3](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.3-lambda-invocation-lifecycle-and-failures.md), [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md), [3.3.1](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.1-lambda-packaging.md), [4.3.1](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.1-concurrency-controls.md), [4.3.2](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.2-memory-and-compute-tuning.md) |

## Containers

| Service | What it is | Skill notes |
|---|---|---|
| **Amazon ECR** | Container image registry: tags, digests, immutability, scanning, lifecycle policies | [3.1.2](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.2-container-images-and-ecr.md), [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md), [3.3.1](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.1-lambda-packaging.md) |
| **Amazon ECS** | Container orchestration: task definitions, services, rolling and blue/green deployments, health checks | [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md), [3.2.4](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.4-isolated-test-environments.md), [4.2.5](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.5-health-checks-and-probes.md), [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md) |
| **Amazon EKS** | Managed Kubernetes: manifests, rolling updates, liveness and readiness probes, pod IAM roles | [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md), [4.2.5](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.5-health-checks-and-probes.md), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |

## Database

| Service | What it is | Skill notes |
|---|---|---|
| **Amazon Aurora** | AWS-built relational engine (MySQL and PostgreSQL compatible), reader endpoints, Aurora Serverless | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [1.3.3](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.3-caching-strategies.md), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md), [2.2.1](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.1-encryption-at-rest-and-in-transit.md) |
| **Amazon DynamoDB** | Serverless key-value database: keys, indexes, consistency, capacity, TTL, Streams, transactions, DAX | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md), [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md), [1.3.3](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.3-caching-strategies.md), [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md), [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md), [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md) |
| **Amazon ElastiCache** | Managed Valkey, Redis OSS, and Memcached caches | [1.3.3](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.3-caching-strategies.md), [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md), [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md) |
| **Amazon RDS** | Managed relational databases, read replicas, Multi-AZ, RDS Proxy, IAM database authentication | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md), [2.2.7](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.7-secrets-management.md), [4.3.1](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.1-concurrency-controls.md) |

## Developer tools

| Service | What it is | Skill notes |
|---|---|---|
| **AWS Amplify** | Front-end hosting, branch environments and previews, Gen 2 sandboxes, auth libraries | [3.2.4](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.4-isolated-test-environments.md), [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |
| **AWS CloudShell** | Browser shell with the CLI preinstalled and console credentials | [1.1.5](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |
| **AWS CodeArtifact** | Private package repositories and upstream caching of public registries | [3.1.1](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.1-packaging-dependencies-and-configuration.md) |
| **AWS CodeBuild** | Managed builds and tests (`buildspec.yml`), test reports, build logs | [3.3.5](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.5-codepipeline-release-promotion.md), [3.1.3](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.3-code-repositories-in-deployment.md), [3.2.5](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.5-ai-assisted-testing.md), [4.1.4](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.4-diagnosing-deployment-failures.md) |
| **AWS CodeDeploy** | Deployments to EC2/on-premises, Lambda, and ECS with hooks, deployment configurations, and rollback | [3.3.4](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.4-codedeploy.md), [3.3.6](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md), [3.3.7](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.7-rollback.md), [4.1.4](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.4-diagnosing-deployment-failures.md), [4.2.3](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.3-alerts-and-notifications.md) |
| **AWS CodePipeline** | Release pipelines: source, build, approval, deploy stages across environments | [3.3.5](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.5-codepipeline-release-promotion.md), [3.1.3](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.3-code-repositories-in-deployment.md), [3.3.9](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.9-ai-assisted-deployment.md) |
| **AWS X-Ray** | Distributed tracing, service maps, annotations, sampling; OpenTelemetry is now the recommended instrumentation | [4.2.4](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.4-distributed-tracing.md), [4.1.2](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.2-metrics-logs-and-traces.md), [4.3.5](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.5-performance-bottleneck-detection.md) |

## Machine learning and AI

| Service | What it is | Skill notes |
|---|---|---|
| **Amazon Bedrock** | Foundation model APIs (Converse, InvokeModel), Knowledge Bases, Agents, Guardrails, invocation logging | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md), [2.3.1](../domain-2-security/task-2.3-secure-ai-services/2.3.1-ai-service-access-control.md), [2.3.2](../domain-2-security/task-2.3-secure-ai-services/2.3.2-ai-data-privacy-and-private-connectivity.md), [2.3.3](../domain-2-security/task-2.3-secure-ai-services/2.3.3-ai-input-and-output-filtering.md), [2.3.5](../domain-2-security/task-2.3-secure-ai-services/2.3.5-ai-sensitive-data-in-logs.md) |
| **Amazon Bedrock AgentCore** | Run and secure AI agents: Runtime, Gateway, Identity, Memory, Policy, Code Interpreter, Browser, Observability | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md), [2.3.4](../domain-2-security/task-2.3-secure-ai-services/2.3.4-ai-agent-security.md) |
| **Amazon Quick** | AI workspace for business users over company data and apps (not a coding tool; Quick Sight alone is out of scope) | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md), [1.1.9](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md) |
| **Kiro** | AWS agentic IDE and CLI: specs, steering, hooks, MCP; used for AI-assisted coding, testing, deployment, debugging, and optimization | [1.1.9](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md), [3.2.5](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.5-ai-assisted-testing.md), [3.3.9](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.9-ai-assisted-deployment.md), [4.1.5](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.5-ai-assisted-error-analysis.md), [4.3.6](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.6-ai-assisted-optimization.md) |

## Management and governance

| Service | What it is | Skill notes |
|---|---|---|
| **AWS AppConfig** | Feature flags and dynamic configuration with validators, gradual rollout, and alarm-based rollback | [3.1.4](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.4-environment-specific-configuration.md), [3.3.6](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md), [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md) |
| **AWS CDK** | Infrastructure in a programming language, synthesized to CloudFormation; assertion and snapshot tests | [3.2.1](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.1-testing-with-sam-cdk-and-cloudformation.md), [3.3.3](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md) |
| **AWS CloudFormation** | Templates, stacks, change sets, drift, deletion policies, rollback | [3.3.3](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md), [3.3.7](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.7-rollback.md), [3.2.1](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.1-testing-with-sam-cdk-and-cloudformation.md), [4.1.4](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.4-diagnosing-deployment-failures.md) |
| **AWS CloudTrail** | API activity history for auditing and root-cause analysis | [4.1.1](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.1-debugging-code-and-integrations.md), [2.3.5](../domain-2-security/task-2.3-secure-ai-services/2.3.5-ai-sensitive-data-in-logs.md), [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md) |
| **Amazon CloudWatch** | Metrics, alarms, dashboards, Application Signals, anomaly detection | [4.1.2](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.2-metrics-logs-and-traces.md), [4.2.2](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.2-custom-metrics.md), [4.2.3](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.3-alerts-and-notifications.md), [4.3.5](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.5-performance-bottleneck-detection.md) |
| **Amazon CloudWatch Logs** | Log groups, Logs Insights queries, retention, metric filters, data protection masking | [4.1.3](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.3-querying-logs.md), [4.2.1](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.1-structured-logging.md), [2.3.5](../domain-2-security/task-2.3-secure-ai-services/2.3.5-ai-sensitive-data-in-logs.md), [2.2.6](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.6-sensitive-data-detection-and-masking.md) |
| **AWS CLI** | Command-line access to every AWS API; profiles, SSO, `--query`, pagination | [1.1.5](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |
| **AWS SAM** | Serverless templates and CLI: `sam build`, `sam local invoke`, `sam deploy`, gradual Lambda deployments | [3.3.3](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md), [3.2.1](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.1-testing-with-sam-cdk-and-cloudformation.md), [3.2.3](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.3-test-events-and-payloads.md), [3.3.1](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.1-lambda-packaging.md) |
| **AWS Systems Manager** | Parameter Store, Session Manager, Run Command | [3.1.4](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.4-environment-specific-configuration.md), [2.2.7](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.7-secrets-management.md), [2.2.5](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.5-development-certificates-and-ssh-keys.md) |

## Networking and content delivery

| Service | What it is | Skill notes |
|---|---|---|
| **Amazon API Gateway** | REST, HTTP, and WebSocket APIs: integrations, mapping templates, validation, authorizers, stages, mocks | [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md), [3.1.5](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md), [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md), [3.2.2](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.2-mock-apis.md), [2.1.2](../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md) |
| **Amazon CloudFront** | CDN: edge caching, signed URLs and cookies, origin access control, Lambda@Edge and CloudFront Functions | [2.1.4](../domain-2-security/task-2.1-authentication-and-authorization/2.1.4-time-limited-access.md), [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md), [1.3.3](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.3-caching-strategies.md), [2.2.1](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.1-encryption-at-rest-and-in-transit.md) |
| **Elastic Load Balancing (ELB)** | ALB, NLB, GWLB: target groups, health checks, TLS listeners, deregistration delay | [Service basics](aws-service-basics.md#high-availability-load-balancing-and-auto-scaling), [4.2.5](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.5-health-checks-and-probes.md), [3.3.6](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md), [3.3.4](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.4-codedeploy.md), [2.2.2](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.2-certificate-management.md) |
| **AWS PrivateLink** | Interface VPC endpoints for private access to AWS services such as Bedrock and Secrets Manager | [2.3.2](../domain-2-security/task-2.3-secure-ai-services/2.3.2-ai-data-privacy-and-private-connectivity.md), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md), [3.1.5](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md) |
| **Amazon Route 53** | DNS records, Alias, routing policies, health checks | [Service basics](aws-service-basics.md#amazon-route-53), [3.1.5](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md), [3.3.6](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md) |
| **Amazon VPC** | Subnets, route tables, security groups, NACLs, NAT gateways, endpoints, Flow Logs | [Service basics](aws-service-basics.md#amazon-vpc-basics), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md), [2.3.2](../domain-2-security/task-2.3-secure-ai-services/2.3.2-ai-data-privacy-and-private-connectivity.md) |

## Security, identity, and compliance

| Service | What it is | Skill notes |
|---|---|---|
| **AWS Certificate Manager (ACM)** | Public and imported TLS certificates, DNS validation, renewal, Region rules | [2.2.2](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.2-certificate-management.md), [3.1.5](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md) |
| **Amazon Cognito** | User pools (sign-in, JWTs, triggers) and identity pools (temporary AWS credentials) | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md), [2.1.2](../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md), [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md) |
| **AWS IAM** | Policies, roles, policy evaluation, least privilege, `iam:PassRole` | [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md), [2.1.2](../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md), [2.3.1](../domain-2-security/task-2.3-secure-ai-services/2.3.1-ai-service-access-control.md) |
| **AWS KMS** | Keys, key policies, grants, envelope encryption, encryption context, rotation | [2.2.4](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.4-kms-key-management.md), [2.2.1](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.1-encryption-at-rest-and-in-transit.md), [2.2.3](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.3-client-side-vs-server-side-encryption.md) |
| **AWS Private CA** | Private certificates for internal TLS and mTLS | [2.2.2](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.2-certificate-management.md), [2.2.5](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.5-development-certificates-and-ssh-keys.md) |
| **AWS Secrets Manager** | Secret storage, rotation, caching, cross-account sharing | [2.2.7](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.7-secrets-management.md), [3.1.4](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.4-environment-specific-configuration.md) |
| **AWS STS** | Temporary credentials: AssumeRole, web identity, SAML, session tokens | [2.1.7](../domain-2-security/task-2.1-authentication-and-authorization/2.1.7-assuming-iam-roles.md), [2.1.5](../domain-2-security/task-2.1-authentication-and-authorization/2.1.5-authenticated-requests.md), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |
| **AWS WAF** | Web ACL rules on API Gateway, CloudFront, ALB, and Cognito: rate limits, managed rule groups | [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md), [3.1.5](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md) |

## Storage

| Service | What it is | Skill notes |
|---|---|---|
| **Amazon EBS** | Block volumes for EC2, snapshots, volume types, encryption | [Service basics](aws-service-basics.md#ec2-storage-ebs-instance-store-and-efs), [2.2.1](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.1-encryption-at-rest-and-in-transit.md) |
| **Amazon EFS** | Shared NFS file system; Lambda and container mounts | [Service basics](aws-service-basics.md#amazon-efs), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md), [1.2.2](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.2-lambda-configuration.md) |
| **Amazon S3** | Object storage: event notifications, lifecycle, storage classes, presigned URLs, encryption, performance | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md), [1.1.8](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.8-event-driven-applications.md), [2.1.4](../domain-2-security/task-2.1-authentication-and-authorization/2.1.4-time-limited-access.md), [2.2.3](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.3-client-side-vs-server-side-encryption.md) |
| **Amazon S3 Glacier** | Archive storage classes (Instant, Flexible, Deep Archive) and restore times | [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md) |

## Services named in skills but not on the list

Some skills name services or features that the in-scope list omits. Learn them at the depth the skill note gives, and no deeper.

| Service or feature | Why it comes up | Skill notes |
|---|---|---|
| AWS Fargate | Named in skill 3.3.2 as an ECS and EKS launch type; treat it as part of ECS and EKS | [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md) |
| AWS Copilot | Named in skill 3.2.4 for isolated ECS environments | [3.2.4](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.4-isolated-test-environments.md) |
| Lambda@Edge, CloudFront Functions | Features of CloudFront | [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md), [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md) |
| DynamoDB Accelerator (DAX) | Feature of DynamoDB, named in skill 4.3.4 | [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md), [1.3.3](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.3-caching-strategies.md) |
| RDS Proxy | Feature of RDS and Aurora for connection pooling | [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md), [4.3.1](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.1-concurrency-controls.md) |
| IAM Identity Center | Workforce federation and CLI single sign-on; part of the IAM family | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |
| AWS Organizations (SCPs) | Service control policies cap permissions, including access to AI models | [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md), [2.3.1](../domain-2-security/task-2.3-secure-ai-services/2.3.1-ai-service-access-control.md) |
| Amazon Q Developer | DVA-C02 era AI coding assistant; DVA-C03 lists Kiro instead | [1.1.9](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md) |
| AWS AppSync | Managed GraphQL; was on the DVA-C02 list | [1.1.2](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.2-communication-patterns.md) |
| AWS CodeCommit | Git hosting; back in general availability since November 2025 but not listed | [3.1.3](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.3-code-repositories-in-deployment.md) |
| Amazon Macie, Amazon Comprehend | PII discovery in S3 and in text | [2.2.6](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.6-sensitive-data-detection-and-masking.md) |
| Amazon MSK, Amazon MQ | Streaming and broker alternatives to Kinesis and SQS | [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md) |
| Amazon MemoryDB | Durable Redis-compatible database | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [1.3.3](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.3-caching-strategies.md) |
| AWS CloudHSM | Single-tenant hardware security modules | [2.2.4](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.4-kms-key-management.md) |
| Amazon Verified Permissions | Fine-grained application authorization | [2.1.2](../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md) |
| Site-to-Site VPN, Direct Connect, Global Accelerator | Networking you only need to eliminate | [Service basics](aws-service-basics.md#site-to-site-vpn-vs-direct-connect) |

> **Not on the DVA-C03 service list.** Every row in the table above is outside the official in-scope list. Skill notes mark them with this same label. When an answer choice relies on one of them and an in-scope service does the same job, prefer the in-scope service.

## Explicitly out of scope

AWS publishes a non-exhaustive [out-of-scope list](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-out-of-scope-services.html). The entries most likely to show up as distractors:

| Category | Out of scope |
|---|---|
| Analytics | Amazon Quick Sight |
| Business applications | Amazon Connect, Amazon WorkMail |
| End user computing | Amazon AppStream 2.0, Amazon WorkSpaces |
| Game tech | Amazon GameLift |
| Machine learning and AI | Amazon Polly, Amazon Rekognition, Amazon SageMaker AI, Strands Agents |
| Management and governance | AWS Managed Services (AMS) |
| Media | Amazon Elastic Transcoder |
| Migration and transfer | AWS Application Discovery Service, AWS Application Migration Service, AWS Transfer Family |
| Security, identity, and compliance | AWS Shield Standard, AWS Shield Advanced |
| Storage | AWS Snow Family, AWS Storage Gateway |

**Trap:** older DVA-C02 study lists include AWS Shield. DVA-C03 lists both Shield tiers as out of scope; AWS WAF stays in scope.

## DVA-C02 vs DVA-C03

> **DVA-C02 vs DVA-C03.** The C03 list adds Amazon Bedrock, Amazon Bedrock AgentCore, Kiro, Amazon Quick, AWS PrivateLink, ACM, AWS Private CA, Amazon Data Firehose, and Amazon S3 Glacier, and lists AWS SAM and CloudWatch Logs as separate entries. AWS AppSync and Amazon Q Developer are no longer listed. SageMaker AI and Strands Agents are explicitly out of scope. Containers (ECR, ECS, EKS) are now tested directly through skills 3.1.2 and 3.3.2.

## Where to spend time

Lambda, DynamoDB, API Gateway, IAM and STS, KMS, SQS and SNS, CloudFormation and SAM, CodeDeploy, and CloudWatch and X-Ray carry the most questions. Each appears in four or more skills above. The AI services (Bedrock, AgentCore, Kiro) are new in DVA-C03 and span Tasks 1.1, 2.3, 3.2, 3.3, 4.1, and 4.3.

## Sources

- Merged from: `archive/dva-c03/services-used-in-this-exam.md` (starting map, links replaced with skill notes), `archive/dva-c03/README.md` (new and out-of-scope services), `archive/aws-dva-notes/Markdown Files/29. Exam Review & Tips.md` (service list changes), `archive/aws-dva-notes/Markdown Files/0. AWS DVA Index.md` (heaviest-tested services).
- AWS documentation:
  - [DVA-C03 in-scope AWS services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-in-scope-services.html)
  - [DVA-C03 out-of-scope AWS services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-out-of-scope-services.html)
  - [Mentions of AWS services on the exam](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-service-mentions.html) (short service names, Help feature)
