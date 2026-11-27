# Glossary

**Foundations** · [Folder overview](README.md) · Related: [Developer basics](developer-basics.md) · [AWS service basics](aws-service-basics.md) · [Services in scope](services-in-scope.md)

Abbreviations and key terms used across the DVA-C03 notes, A to Z. Each entry links to the skill note that covers it in depth, or to the foundations note that explains it when no single skill owns it. Skill notes also spell out each abbreviation the first time it appears.

**Jump to:** [A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [J](#j) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [Q](#q) · [R](#r) · [S](#s) · [T](#t) · [U](#u) · [V](#v) · [W](#w) · [X](#x) · [Y](#y) · [Z](#z)

## A

| Term | Meaning | Where it matters |
|---|---|---|
| **A record / AAAA record** | Route 53 records that map a name to an IPv4 / IPv6 address. Alias records are A or AAAA. | [Service basics](aws-service-basics.md#records-and-hosted-zones) |
| **ABAC** | Attribute-based access control. Permissions decided by matching tags on principals and resources (for example `aws:PrincipalTag/team`). | [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md) |
| **Access key** | Long-term IAM user credential pair (access key ID starting `AKIA` plus secret access key). Avoid in code; prefer roles. Temporary STS keys start `ASIA`. | [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |
| **Access token** | OAuth 2.0 token that says what the caller may do (scopes). Send it to APIs. | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md), [2.1.2](../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md) |
| **ACM** | AWS Certificate Manager. Issues, imports, and renews TLS certificates for ELB, CloudFront, and API Gateway. | [2.2.2](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.2-certificate-management.md) |
| **ADOT** | AWS Distro for OpenTelemetry. AWS-supported OpenTelemetry SDKs and collector for traces and metrics. | [4.2.4](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.4-distributed-tracing.md) |
| **AES / AES-GCM** | Advanced Encryption Standard, a symmetric cipher; AES-GCM is AES in Galois/Counter Mode, which also authenticates the data. | [2.2.1](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.1-encryption-at-rest-and-in-transit.md), [2.2.4](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.4-kms-key-management.md) |
| **AgentCore** | Amazon Bedrock AgentCore. Managed services to deploy and run AI agents: Runtime, Gateway, Identity, Memory, Policy, Code Interpreter, Browser, Observability. | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md), [2.3.4](../domain-2-security/task-2.3-secure-ai-services/2.3.4-ai-agent-security.md) |
| **AI** | Artificial intelligence. | [Developer basics](developer-basics.md#ai-concepts-for-developers) |
| **AIF / AIP / DEA / DOP / SAA** | Other AWS certification codes: AI Practitioner, Generative AI Developer - Professional, Data Engineer - Associate, DevOps Engineer - Professional, Solutions Architect - Associate. | [Exam guide](exam-guide.md) |
| **ALB** | Application Load Balancer. Layer 7 HTTP/HTTPS load balancer with path and host routing; can target Lambda. | [Service basics](aws-service-basics.md#application-load-balancer), [4.2.5](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.5-health-checks-and-probes.md) |
| **Alias (Lambda)** | Named pointer to a Lambda version (`prod`, `staging`); can split traffic between two versions for canary releases. | [3.3.8](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.8-versions-and-releases.md), [3.2.4](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.4-isolated-test-environments.md) |
| **Alias record (Route 53)** | Route 53 extension that points a name, including the zone apex, at an AWS resource; free and health-check aware. | [Service basics](aws-service-basics.md#cname-vs-alias) |
| **AMI** | Amazon Machine Image. Template for launching EC2 instances; Region-specific. | [Service basics](aws-service-basics.md#snapshots-and-amis) |
| **API** | Application programming interface. | [Developer basics](developer-basics.md#api), [1.1.5](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md) |
| **API key** | API Gateway identifier sent in `x-api-key` and tied to usage plans for throttling and quotas. Not an authentication mechanism on its own. | [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md), [3.1.5](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md) |
| **AppConfig** | AWS AppConfig. Managed feature flags and configuration with validators, gradual deployment, and alarm-based rollback. | [3.1.4](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.4-environment-specific-configuration.md) |
| **arm64** | 64-bit Arm architecture (AWS Graviton). Lambda on arm64 usually costs less per GB-second. | [4.3.2](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.2-memory-and-compute-tuning.md), [3.3.1](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.1-lambda-packaging.md) |
| **ARN** | Amazon Resource Name: `arn:partition:service:region:account:resource`. | [Developer basics](developer-basics.md#amazon-resource-names), [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md) |
| **ASG** | Auto Scaling group. Keeps a fleet of EC2 instances between minimum and maximum, replaces unhealthy ones, and scales with policies. | [Service basics](aws-service-basics.md#auto-scaling-groups), [4.3.1](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.1-concurrency-controls.md) |
| **AssumeRole** | STS API that returns temporary credentials for a role; needs a trust policy allowing the caller. | [2.1.7](../domain-2-security/task-2.1-authentication-and-authorization/2.1.7-assuming-iam-roles.md) |
| **Asynchronous invocation** | Lambda queues the event and returns 202; Lambda retries failures twice by default, then sends to a DLQ or on-failure destination. | [1.2.3](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.3-lambda-invocation-lifecycle-and-failures.md) |
| **At-least-once delivery** | A message or event can arrive more than once; consumers must be idempotent. | [Developer basics](developer-basics.md#delivery-guarantees), [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md) |
| **Athena** | Amazon Athena. Serverless SQL over data in S3. | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [4.1.3](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.3-querying-logs.md) |
| **Authorizer** | API Gateway component that checks a request before the backend runs: IAM (SigV4), Cognito user pool, JWT (HTTP APIs), or Lambda authorizer. | [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md), [2.1.2](../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md) |
| **AWS** | Amazon Web Services. | [Exam guide](exam-guide.md) |
| **AZ** | Availability Zone. One or more isolated data centers in a Region. | [Service basics](aws-service-basics.md#scalability-vs-availability) |

## B

| Term | Meaning | Where it matters |
|---|---|---|
| **BAA** | Business Associate Addendum. The HIPAA agreement with AWS that covers PHI on eligible services. | [2.2.6](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.6-sensitive-data-detection-and-masking.md) |
| **Base64** | Text encoding for binary data. Kinesis payloads in Lambda events and API Gateway binary bodies arrive base64-encoded. | [Developer basics](developer-basics.md#base64), [1.1.7](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.7-serialization-and-deserialization.md) |
| **Batch item failures** | Partial batch response: a Lambda function returns the IDs of failed records so only those are retried (SQS, Kinesis, DynamoDB Streams). | [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md), [1.2.3](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.3-lambda-invocation-lifecycle-and-failures.md) |
| **Bedrock** | Amazon Bedrock. Managed access to foundation models through `Converse`, `InvokeModel`, Knowledge Bases, Agents, and Guardrails. | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md), [2.3.1](../domain-2-security/task-2.3-secure-ai-services/2.3.1-ai-service-access-control.md) |
| **BI** | Business intelligence. | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md) |
| **Bisect on function error** | Kinesis and DynamoDB Streams event source mapping option that splits a failing batch in two to isolate a bad record. | [1.2.3](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.3-lambda-invocation-lifecycle-and-failures.md), [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md) |
| **Blue/green deployment** | Run the new version beside the old one and switch traffic; quick rollback by switching back. | [3.3.6](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md) |
| **buildspec.yml** | CodeBuild build instructions: phases, environment variables, artifacts, reports, cache. | [3.3.5](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.5-codepipeline-release-promotion.md), [4.1.4](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.4-diagnosing-deployment-failures.md) |

## C

| Term | Meaning | Where it matters |
|---|---|---|
| **CA** | Certificate authority. Signs certificates; public CAs for internet trust, AWS Private CA for internal trust. | [2.2.2](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.2-certificate-management.md) |
| **Canary** | Send a small share of traffic to the new version first, then the rest (CodeDeploy `Canary10Percent5Minutes`, API Gateway stage canary). | [3.3.6](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md) |
| **Cardinality** | Number of distinct values of a key. High-cardinality partition keys spread load in DynamoDB. | [Developer basics](developer-basics.md#dynamodb-keys-and-indexes), [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **CDK** | AWS Cloud Development Kit. Define infrastructure in a programming language; synthesizes CloudFormation. | [3.2.1](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.1-testing-with-sam-cdk-and-cloudformation.md), [3.3.3](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md) |
| **CDN** | Content delivery network (CloudFront). | [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md) |
| **Change set** | Preview of what a CloudFormation stack update will add, modify, or replace. | [3.3.3](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md), [3.3.7](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.7-rollback.md) |
| **CI/CD** | Continuous integration and continuous delivery or deployment. | [Developer basics](developer-basics.md#cicd), [3.3.5](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.5-codepipeline-release-promotion.md) |
| **CIDR** | Classless Inter-Domain Routing notation for IP ranges (`10.0.0.0/16`). | [Service basics](aws-service-basics.md#building-blocks) |
| **Circuit breaker** | Stop calling a failing dependency for a while; ECS deployment circuit breaker rolls back failed deployments. | [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md), [3.3.7](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.7-rollback.md) |
| **CLB** | Classic Load Balancer. Legacy layer 4/7 load balancer; one certificate only. | [Service basics](aws-service-basics.md#load-balancer-types) |
| **CLI** | AWS Command Line Interface. | [Developer basics](developer-basics.md#sdk-and-cli), [1.1.5](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md) |
| **CloudFront** | Amazon CloudFront. CDN with edge caching, signed URLs and cookies, origin access control. | [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md), [2.1.4](../domain-2-security/task-2.1-authentication-and-authorization/2.1.4-time-limited-access.md) |
| **CloudShell** | AWS CloudShell. Browser shell in the console, pre-authenticated, with the CLI installed. | [1.1.5](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md) |
| **CloudTrail** | AWS CloudTrail. Records API calls (who did what, when, from where); event history 90 days. | [4.1.1](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.1-debugging-code-and-integrations.md), [2.3.5](../domain-2-security/task-2.3-secure-ai-services/2.3.5-ai-sensitive-data-in-logs.md) |
| **CloudWatch** | Amazon CloudWatch. Metrics, alarms, dashboards; CloudWatch Logs for log groups and Logs Insights queries. | [4.1.2](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.2-metrics-logs-and-traces.md), [4.2.2](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.2-custom-metrics.md) |
| **CMK** | Customer master key, the old name for a KMS key; "customer managed key" now means a KMS key you create and control. | [2.2.4](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.4-kms-key-management.md) |
| **CNAME** | Canonical name record. Maps a name to another hostname; not allowed at the zone apex. | [Service basics](aws-service-basics.md#cname-vs-alias) |
| **CodeArtifact** | AWS CodeArtifact. Private package repository (npm, PyPI, Maven) that can proxy public registries. | [3.1.1](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.1-packaging-dependencies-and-configuration.md) |
| **CodeBuild** | AWS CodeBuild. Managed build and test service driven by `buildspec.yml`. | [3.3.5](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.5-codepipeline-release-promotion.md), [4.1.4](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.4-diagnosing-deployment-failures.md) |
| **CodeDeploy** | AWS CodeDeploy. Deploys revisions to EC2/on-premises, Lambda, and ECS with `appspec.yml` hooks and deployment configurations. | [3.3.4](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.4-codedeploy.md) |
| **CodePipeline** | AWS CodePipeline. Orchestrates source, build, test, approval, and deploy stages. | [3.3.5](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.5-codepipeline-release-promotion.md) |
| **Cognito** | Amazon Cognito. User pools (sign-in, JWTs) and identity pools (temporary AWS credentials). | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |
| **Cold start** | Delay while Lambda creates a new execution environment and runs init code. | [Developer basics](developer-basics.md#cold-starts), [4.3.2](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.2-memory-and-compute-tuning.md) |
| **Condition expression** | DynamoDB write condition (`attribute_not_exists`, version check) used for optimistic locking and idempotency. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md), [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md) |
| **Connection draining** | See deregistration delay. | [Service basics](aws-service-basics.md#connection-draining-deregistration-delay) |
| **Converse API** | Bedrock API with one request format across models, including tool use and streaming (`ConverseStream`). | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md) |
| **Copilot** | AWS Copilot CLI. Creates and deploys ECS environments and services. | [3.2.4](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.4-isolated-test-environments.md) |
| **CORS** | Cross-origin resource sharing. Browser rules for calling an API on another origin; with proxy integrations the backend must return the CORS headers. | [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md) |
| **CPU** | Central processing unit. | [4.3.2](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.2-memory-and-compute-tuning.md) |
| **CRL / OCSP** | Certificate revocation list / Online Certificate Status Protocol. Ways to check whether a certificate was revoked. | [2.2.2](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.2-certificate-management.md) |
| **Cross-zone load balancing** | Each load balancer node spreads traffic across targets in all AZs. On by default for ALB, off for NLB. | [Service basics](aws-service-basics.md#cross-zone-load-balancing) |
| **CSR** | Certificate signing request. | [2.2.5](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.5-development-certificates-and-ssh-keys.md) |

## D

| Term | Meaning | Where it matters |
|---|---|---|
| **DAX** | DynamoDB Accelerator. In-memory write-through cache in front of DynamoDB for microsecond reads. | [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md), [1.3.3](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.3-caching-strategies.md) |
| **Dead-letter queue (DLQ)** | Queue that holds messages or events that failed after retries. | [Developer basics](developer-basics.md#dead-letter-queues), [1.2.3](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.3-lambda-invocation-lifecycle-and-failures.md) |
| **Deployment configuration** | CodeDeploy rule for how fast traffic or instances shift (`AllAtOnce`, `OneAtATime`, canary, linear). | [3.3.4](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.4-codedeploy.md) |
| **Deregistration delay** | Time a load balancer lets in-flight requests finish before removing a target (default 300 seconds). | [Service basics](aws-service-basics.md#connection-draining-deregistration-delay) |
| **Destination (Lambda)** | Where Lambda sends the result of an async invocation or a failed event source batch: SQS, SNS, Lambda, EventBridge, S3. | [1.2.3](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.3-lambda-invocation-lifecycle-and-failures.md) |
| **Digest** | Immutable content hash of a container image (`sha256:...`). | [3.1.2](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.2-container-images-and-ecr.md) |
| **DNS** | Domain Name System. | [Service basics](aws-service-basics.md#amazon-route-53) |
| **Drift** | Difference between a CloudFormation stack's template and the actual resource configuration. | [3.3.3](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md) |
| **DSSE-KMS** | Dual-layer server-side encryption with KMS keys for S3. | [2.2.3](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.3-client-side-vs-server-side-encryption.md) |
| **DVA-C03** | AWS Certified Developer - Associate exam, version 3. | [Exam guide](exam-guide.md) |
| **DynamoDB** | Amazon DynamoDB. Serverless key-value and document database. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **DynamoDB Streams** | Ordered change log of item writes, kept 24 hours; triggers Lambda through an event source mapping. | [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md), [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md) |

## E

| Term | Meaning | Where it matters |
|---|---|---|
| **EBS** | Amazon Elastic Block Store. Network block volumes for EC2, locked to one AZ. | [Service basics](aws-service-basics.md#ebs-volumes) |
| **EC2** | Amazon Elastic Compute Cloud. Virtual machines. | [Service basics](aws-service-basics.md#amazon-ec2) |
| **ECC** | Elliptic curve cryptography. | [2.2.5](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.5-development-certificates-and-ssh-keys.md) |
| **ECR** | Amazon Elastic Container Registry. Stores container images; tag immutability, scanning, lifecycle policies. | [3.1.2](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.2-container-images-and-ecr.md) |
| **ECS** | Amazon Elastic Container Service. Runs containers from task definitions on EC2 or Fargate. | [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md) |
| **EFO** | Enhanced fan-out. Dedicated 2 MB/s per shard per consumer on Kinesis Data Streams, pushed over HTTP/2. | [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md) |
| **EFS** | Amazon Elastic File System. Shared NFS file system across AZs; Linux only. | [Service basics](aws-service-basics.md#amazon-efs), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md) |
| **EKS** | Amazon Elastic Kubernetes Service. Managed Kubernetes; deployments use manifests. | [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md) |
| **Elastic Beanstalk** | AWS Elastic Beanstalk. Managed platform for web apps with deployment policies (all at once, rolling, immutable, traffic splitting). | [3.3.6](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md), [3.1.1](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.1-packaging-dependencies-and-configuration.md) |
| **ElastiCache** | Amazon ElastiCache. Managed Valkey, Redis OSS, and Memcached in-memory caches. | [1.3.3](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.3-caching-strategies.md), [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md) |
| **ELB** | Elastic Load Balancing (ALB, NLB, GWLB, CLB). | [Service basics](aws-service-basics.md#high-availability-load-balancing-and-auto-scaling) |
| **Embedding** | Vector of numbers that represents meaning, used for semantic search and RAG. | [Developer basics](developer-basics.md#embeddings-and-rag), [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md) |
| **EMF** | CloudWatch Embedded Metric Format. Structured log lines that CloudWatch turns into metrics asynchronously. | [4.2.2](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.2-custom-metrics.md) |
| **Encryption context** | Non-secret key-value pairs bound to KMS ciphertext; must match on decrypt and appear in CloudTrail. | [2.2.4](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.4-kms-key-management.md) |
| **ENI** | Elastic network interface. Virtual network card; VPC Lambda functions and interface endpoints use ENIs. | [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md) |
| **Envelope encryption** | Encrypt data with a data key, then encrypt the data key with a KMS key. | [Developer basics](developer-basics.md#envelope-encryption), [2.2.4](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.4-kms-key-management.md) |
| **ESM** | Event source mapping. Lambda poller that reads SQS, Kinesis, DynamoDB Streams, MSK, or Amazon MQ and invokes the function with batches. | [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md) |
| **EULA** | End user license agreement, accepted for some third-party Bedrock models. | [2.3.1](../domain-2-security/task-2.3-secure-ai-services/2.3.1-ai-service-access-control.md) |
| **EventBridge** | Amazon EventBridge. Event bus with content-based rules, schema registry, Pipes, and Scheduler. | [1.1.8](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.8-event-driven-applications.md) |
| **Execution role** | IAM role a Lambda function assumes to call AWS services. | [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md), [1.2.2](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.2-lambda-configuration.md) |
| **Exponential backoff** | Wait longer after each failed retry, plus jitter. | [Developer basics](developer-basics.md#retries-exponential-backoff-and-jitter), [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md) |

## F

| Term | Meaning | Where it matters |
|---|---|---|
| **Fan-out** | One message delivered to many consumers, typically SNS to several SQS queues. | [4.3.3](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.3-messaging-optimization.md), [1.1.1](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.1-coupling-and-decoupling.md) |
| **Fargate** | AWS Fargate. Serverless compute for ECS and EKS tasks; no instances to manage. | [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md) |
| **FIFO** | First-in, first-out. SQS FIFO queues order messages per message group and deduplicate within 5 minutes. | [1.1.1](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.1-coupling-and-decoupling.md), [4.3.3](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.3-messaging-optimization.md) |
| **FIPS** | Federal Information Processing Standards. FIPS endpoints use validated cryptography. | [2.2.1](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.1-encryption-at-rest-and-in-transit.md) |
| **Firehose** | Amazon Data Firehose. Managed delivery of streaming data to S3, OpenSearch, Redshift, and HTTP endpoints, with optional Lambda transformation. | [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md) |
| **Foundation model** | Large pre-trained model adapted to many tasks. | [Developer basics](developer-basics.md#foundation-models-llms-and-tokens), [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md) |
| **FQDN** | Fully qualified domain name. | [Service basics](aws-service-basics.md#dns-fundamentals) |

## G

| Term | Meaning | Where it matters |
|---|---|---|
| **GB / GiB** | Gigabyte (10^9 bytes) / gibibyte (2^30 bytes). | [Developer basics](developer-basics.md) |
| **Glacier IR** | S3 Glacier Instant Retrieval storage class: archive pricing with millisecond access. | [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md) |
| **GraphQL** | Graph Query Language. Clients request exactly the fields they need. | [Developer basics](developer-basics.md#graphql-and-websocket) |
| **GSI** | Global secondary index. DynamoDB index with a different partition key; eventually consistent. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **Guardrails** | Amazon Bedrock Guardrails. Filters for harmful content, denied topics, PII, words, prompt attacks, and grounding. | [2.3.3](../domain-2-security/task-2.3-secure-ai-services/2.3.3-ai-input-and-output-filtering.md) |
| **GWLB** | Gateway Load Balancer. Layer 3 load balancer for inline network appliances (GENEVE, port 6081). | [Service basics](aws-service-basics.md#gateway-load-balancer) |

## H

| Term | Meaning | Where it matters |
|---|---|---|
| **HA** | High availability. Run across at least two AZs. | [Service basics](aws-service-basics.md#scalability-vs-availability) |
| **Health check** | Probe that decides whether a target gets traffic (ELB, Route 53, ECS, Kubernetes). | [4.2.5](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.5-health-checks-and-probes.md), [Service basics](aws-service-basics.md#health-checks) |
| **HIPAA** | Health Insurance Portability and Accountability Act (US health data law). | [2.2.6](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.6-sensitive-data-detection-and-masking.md) |
| **HMAC** | Hash-based message authentication code. Keyed hash used in SigV4 and webhooks. | [Developer basics](developer-basics.md#hashing-and-hmac), [2.1.5](../domain-2-security/task-2.1-authentication-and-authorization/2.1.5-authenticated-requests.md) |
| **Hosted zone** | Route 53 container for a domain's records; public or private. | [Service basics](aws-service-basics.md#records-and-hosted-zones) |
| **Hot partition** | DynamoDB partition receiving a disproportionate share of traffic, which throttles even with spare table capacity. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **HTTP / HTTPS** | Hypertext Transfer Protocol / HTTP Secure (over TLS). | [Developer basics](developer-basics.md#rest-and-http-methods) |
| **Human-in-the-loop** | A person approves a sensitive action before an agent or workflow performs it. | [2.3.4](../domain-2-security/task-2.3-secure-ai-services/2.3.4-ai-agent-security.md), [3.3.9](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.9-ai-assisted-deployment.md) |

## I

| Term | Meaning | Where it matters |
|---|---|---|
| **IA** | Infrequent Access storage class (S3 Standard-IA, EFS IA). | [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md) |
| **IaC** | Infrastructure as code. | [Developer basics](developer-basics.md#infrastructure-as-code), [3.3.3](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md) |
| **IAM** | AWS Identity and Access Management. | [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md) |
| **ID token** | OIDC token describing the signed-in user. | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |
| **IDE** | Integrated development environment. | [1.1.9](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md) |
| **Idempotency** | Doing an operation twice has the same effect as once. | [Developer basics](developer-basics.md#idempotency), [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md) |
| **Identity Center** | AWS IAM Identity Center. Workforce single sign-on to accounts and the CLI (`aws configure sso`). | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |
| **Identity pool** | Cognito identity pool. Exchanges a user's token for temporary AWS credentials through an IAM role. | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |
| **IdP** | Identity provider. | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |
| **IGW** | Internet gateway. Gives a VPC's public subnets internet access. | [Service basics](aws-service-basics.md#building-blocks) |
| **IMDSv2** | Instance Metadata Service version 2. Token-based metadata access on EC2. | [Service basics](aws-service-basics.md#user-data-and-instance-metadata), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |
| **Instance profile** | Container that attaches an IAM role to an EC2 instance. | [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |
| **Instance store** | Ephemeral local disk on the EC2 host. | [Service basics](aws-service-basics.md#ec2-instance-store) |
| **Interface endpoint** | VPC endpoint powered by AWS PrivateLink; an ENI with a private IP for most AWS services. | [Service basics](aws-service-basics.md#vpc-endpoints-and-aws-privatelink), [2.3.2](../domain-2-security/task-2.3-secure-ai-services/2.3.2-ai-data-privacy-and-private-connectivity.md) |
| **IOPS** | Input/output operations per second. | [Service basics](aws-service-basics.md#ebs-volume-types) |
| **IP / IPv6** | Internet Protocol / Internet Protocol version 6. | [Service basics](aws-service-basics.md#amazon-vpc-basics) |
| **IPsec** | Internet Protocol Security. Encrypts Site-to-Site VPN tunnels. | [Service basics](aws-service-basics.md#site-to-site-vpn-vs-direct-connect) |
| **IRSA / EKS Pod Identity** | IAM roles for service accounts and EKS Pod Identity. Give Kubernetes pods their own IAM role. | [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md), [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md) |

## J

| Term | Meaning | Where it matters |
|---|---|---|
| **Jitter** | Randomness added to backoff delays so clients don't retry in lockstep. | [Developer basics](developer-basics.md#retries-exponential-backoff-and-jitter), [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md) |
| **JMESPath** | JSON query language used by the CLI `--query` option. | [1.1.5](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md) |
| **JSON** | JavaScript Object Notation. | [Developer basics](developer-basics.md#serialization), [1.1.7](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.7-serialization-and-deserialization.md) |
| **JSONata** | Query and transformation language available in Step Functions states. | [1.1.2](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.2-communication-patterns.md) |
| **JWKS** | JSON Web Key Set. Public keys an issuer publishes for verifying JWT signatures. | [Developer basics](developer-basics.md#jwt) |
| **JWT** | JSON Web Token: `header.payload.signature`. | [Developer basics](developer-basics.md#jwt), [2.1.2](../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md) |

## K

| Term | Meaning | Where it matters |
|---|---|---|
| **KB / KiB** | Kilobyte / kibibyte. | [Developer basics](developer-basics.md) |
| **KCL / KPL** | Kinesis Client Library (consumer with checkpointing in DynamoDB) / Kinesis Producer Library (batching producer). | [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md) |
| **KDS** | Kinesis Data Streams. Sharded, ordered, replayable stream. | [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md) |
| **Kiro** | AWS agentic development environment (IDE and CLI) with specs, steering, hooks, and MCP. | [1.1.9](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md), [3.2.5](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.5-ai-assisted-testing.md) |
| **KMS** | AWS Key Management Service. | [2.2.4](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.4-kms-key-management.md) |
| **Knowledge Bases** | Amazon Bedrock Knowledge Bases. Managed RAG: ingest documents, embed, retrieve, cite. | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md) |

## L

| Term | Meaning | Where it matters |
|---|---|---|
| **Lambda** | AWS Lambda. Serverless functions. | [1.2.2](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.2-lambda-configuration.md) |
| **Launch template** | EC2 launch settings (AMI, type, user data, SGs, role) used by ASGs. | [Service basics](aws-service-basics.md#auto-scaling-groups) |
| **Layer** | Lambda ZIP archive of shared libraries or data mounted at `/opt`; up to 5 per function. | [1.2.2](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.2-lambda-configuration.md), [3.3.1](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.1-lambda-packaging.md) |
| **Least privilege** | Grant only the actions and resources a task needs. | [Developer basics](developer-basics.md#principal-policy-and-least-privilege), [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md) |
| **Lifecycle policy** | Rules that transition or expire data as it ages (S3, EFS, ECR). | [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md), [3.1.2](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.2-container-images-and-ecr.md) |
| **LLM** | Large language model. | [Developer basics](developer-basics.md#foundation-models-llms-and-tokens) |
| **Long polling** | SQS `ReceiveMessage` waits up to 20 seconds for messages, cutting empty responses and cost. | [4.3.3](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.3-messaging-optimization.md) |
| **LSI** | Local secondary index. Same partition key, different sort key; created with the table. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |

## M

| Term | Meaning | Where it matters |
|---|---|---|
| **Mapping template** | API Gateway REST transformation written in Velocity Template Language (VTL). | [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md) |
| **MB / MiB** | Megabyte / mebibyte. | [Developer basics](developer-basics.md) |
| **MCP** | Model Context Protocol. Open standard for connecting AI agents to tools and data. | [Developer basics](developer-basics.md#mcp), [1.1.9](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md) |
| **MFA** | Multi-factor authentication. | [Developer basics](developer-basics.md#mfa), [2.1.2](../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md) |
| **ML** | Machine learning. | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md) |
| **Mock integration** | API Gateway integration that returns a response without calling a backend. | [3.2.2](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.2-mock-apis.md) |
| **MSK** | Amazon Managed Streaming for Apache Kafka. | [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md) |
| **mTLS** | Mutual TLS. Both client and server present certificates. | [2.2.2](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.2-certificate-management.md) |
| **Multi-AZ** | Standby or replicas in another AZ for availability (RDS, Aurora, ElastiCache). | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md) |

## N

| Term | Meaning | Where it matters |
|---|---|---|
| **NACL** | Network access control list. Stateless subnet firewall with ordered allow and deny rules. | [Service basics](aws-service-basics.md#security-groups-vs-network-acls) |
| **NAT gateway** | Network address translation gateway. Outbound internet for private subnets (VPC Lambda functions need one for internet access). | [Service basics](aws-service-basics.md#building-blocks), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md) |
| **NLB** | Network Load Balancer. Layer 4 TCP/UDP/TLS, static IP per AZ. | [Service basics](aws-service-basics.md#network-load-balancer) |
| **NoSQL** | "Not only SQL" databases such as DynamoDB. | [Developer basics](developer-basics.md#relational-vs-nosql) |
| **NS record** | Name server record for a hosted zone. | [Service basics](aws-service-basics.md#registrar-vs-dns-service) |

## O

| Term | Meaning | Where it matters |
|---|---|---|
| **OAC** | Origin access control. Lets CloudFront read a private S3 bucket (replaces origin access identity, OAI). | [2.1.4](../domain-2-security/task-2.1-authentication-and-authorization/2.1.4-time-limited-access.md), [4.3.4](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md) |
| **OAuth 2.0** | Open Authorization. Framework for issuing access tokens with scopes. | [Developer basics](developer-basics.md#federation-oauth-20-oidc-saml), [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |
| **OIDC** | OpenID Connect. Identity layer on OAuth 2.0; also how GitHub Actions gets AWS credentials without keys. | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md), [2.1.3](../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md) |
| **On-demand capacity** | DynamoDB pay-per-request mode. No capacity planning, but can still throttle on hot partitions, sudden spikes beyond the previous peak, or table quotas. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **OpenSearch Service** | Amazon OpenSearch Service. Search and log analytics; also a vector store. | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [1.1.6](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md) |
| **OpenTelemetry** | Open standard for traces, metrics, and logs; AWS recommends it over the X-Ray SDKs. | [4.2.4](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.4-distributed-tracing.md) |
| **ORC / Parquet** | Optimized Row Columnar / Apache Parquet. Columnar file formats for analytics. | [1.1.7](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.7-serialization-and-deserialization.md) |
| **OS** | Operating system. | [Service basics](aws-service-basics.md#launch-time-choices) |

## P

| Term | Meaning | Where it matters |
|---|---|---|
| **p99** | 99th percentile. | [Developer basics](developer-basics.md#percentiles), [4.3.5](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.5-performance-bottleneck-detection.md) |
| **Paginator** | SDK helper that follows continuation tokens through every page. | [Developer basics](developer-basics.md#pagination-and-waiters), [1.1.5](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md) |
| **Parameter Store** | AWS Systems Manager Parameter Store. Hierarchical configuration and `SecureString` secrets. | [2.2.7](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.7-secrets-management.md), [3.1.4](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.4-environment-specific-configuration.md) |
| **PartiQL** | SQL-compatible query language for DynamoDB. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **Partition key** | DynamoDB key hashed to choose the storage partition. | [Developer basics](developer-basics.md#dynamodb-keys-and-indexes), [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **PassRole** | `iam:PassRole` permission needed to hand a role to a service (for example when creating a Lambda function). | [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md) |
| **PHI** | Protected health information. | [Developer basics](developer-basics.md#pii-and-phi-masking-and-redaction), [2.2.6](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.6-sensitive-data-detection-and-masking.md) |
| **PII** | Personally identifiable information. | [Developer basics](developer-basics.md#pii-and-phi-masking-and-redaction), [2.2.6](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.6-sensitive-data-detection-and-masking.md) |
| **PITR** | Point-in-time recovery (DynamoDB continuous backups, up to 35 days). | [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md) |
| **PKCE** | Proof Key for Code Exchange. Protects the OAuth authorization code flow for public clients. | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |
| **Powertools for AWS Lambda** | Open-source library for structured logging, metrics (EMF), tracing, idempotency, and batch processing in Lambda. | [4.2.1](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.1-structured-logging.md), [1.1.3](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md) |
| **PPL** | Piped Processing Language, one of the CloudWatch Logs Insights query languages (with SQL and the Logs Insights language). | [4.1.3](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.3-querying-logs.md) |
| **PR** | Pull request. | [3.3.8](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.8-versions-and-releases.md) |
| **Presigned URL** | S3 URL signed with SigV4 in the query string; grants time-limited access with the signer's permissions. | [2.1.4](../domain-2-security/task-2.1-authentication-and-authorization/2.1.4-time-limited-access.md) |
| **Private CA** | AWS Private Certificate Authority. Issues private certificates for internal TLS and mTLS. | [2.2.2](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.2-certificate-management.md) |
| **PrivateLink** | AWS PrivateLink. Private connectivity to services through interface endpoints. | [2.3.2](../domain-2-security/task-2.3-secure-ai-services/2.3.2-ai-data-privacy-and-private-connectivity.md), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md) |
| **Prompt injection** | Input text that tries to override a model's instructions. | [Developer basics](developer-basics.md#prompt-injection), [2.3.3](../domain-2-security/task-2.3-secure-ai-services/2.3.3-ai-input-and-output-filtering.md) |
| **Provisioned concurrency** | Pre-initialized Lambda execution environments that remove cold starts for a version or alias. | [4.3.1](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.1-concurrency-controls.md), [1.2.2](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.2-lambda-configuration.md) |

## Q

| Term | Meaning | Where it matters |
|---|---|---|
| **Query (DynamoDB)** | Read items with one partition key value, optionally filtered by sort key condition; 1 MB per page. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **Quick** | Amazon Quick. AI workspace for business users over company data and apps. Quick Sight alone is out of scope. | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md), [1.1.9](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md) |

## R

| Term | Meaning | Where it matters |
|---|---|---|
| **RAG** | Retrieval-augmented generation. | [Developer basics](developer-basics.md#embeddings-and-rag), [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md) |
| **RAM** | Random access memory; also AWS Resource Access Manager (share resources across accounts). | [4.3.2](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.2-memory-and-compute-tuning.md) |
| **RCU / WCU** | Read / write capacity unit. 1 RCU = one strongly consistent (or two eventually consistent) 4 KB reads per second; 1 WCU = one 1 KB write per second. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **RDS** | Amazon Relational Database Service. Managed MySQL, PostgreSQL, MariaDB, Oracle, SQL Server; Aurora is the AWS-built engine. | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md) |
| **RDS Proxy** | Managed connection pool in front of RDS and Aurora; protects the database from Lambda connection storms and supports IAM authentication. | [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md), [4.3.1](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.1-concurrency-controls.md) |
| **Reserved concurrency** | Lambda setting that both guarantees and caps a function's concurrent executions. | [4.3.1](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.1-concurrency-controls.md), [1.2.2](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.2-lambda-configuration.md) |
| **REST** | Representational State Transfer. | [Developer basics](developer-basics.md#rest-and-http-methods) |
| **Rolling deployment** | Replace instances or tasks in batches. | [3.3.6](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md), [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md) |
| **Route 53** | Amazon Route 53. Authoritative DNS, registrar, and health checks. | [Service basics](aws-service-basics.md#amazon-route-53) |
| **rps / TPS** | Requests per second / transactions per second. | [Developer basics](developer-basics.md) |
| **RSA** | Rivest-Shamir-Adleman asymmetric algorithm. | [2.2.5](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.5-development-certificates-and-ssh-keys.md) |
| **RUM** | CloudWatch Real User Monitoring (browser performance from real users). | [4.1.2](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.2-metrics-logs-and-traces.md) |

## S

| Term | Meaning | Where it matters |
|---|---|---|
| **S3** | Amazon Simple Storage Service. Object storage. | [1.3.2](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md), [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md) |
| **SaaS** | Software as a service. | [1.1.8](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.8-event-driven-applications.md) |
| **SAM** | AWS Serverless Application Model. CloudFormation transform and CLI for serverless apps (`sam build`, `sam local invoke`, `sam deploy`). | [3.3.3](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md), [3.2.1](../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.1-testing-with-sam-cdk-and-cloudformation.md) |
| **SAML** | Security Assertion Markup Language. XML-based enterprise federation. | [Developer basics](developer-basics.md#federation-oauth-20-oidc-saml), [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |
| **Scan (DynamoDB)** | Read every item in a table or index, then filter; costly at scale. | [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **SCP** | Service control policy. AWS Organizations guardrail that limits maximum permissions in member accounts. | [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md), [2.3.1](../domain-2-security/task-2.3-secure-ai-services/2.3.1-ai-service-access-control.md) |
| **SDK** | Software development kit. | [Developer basics](developer-basics.md#sdk-and-cli), [1.1.5](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md) |
| **Secrets Manager** | AWS Secrets Manager. Stores, rotates, and shares secrets. | [2.2.7](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.7-secrets-management.md) |
| **Security group (SG)** | Stateful allow-only firewall at the ENI level. | [Service basics](aws-service-basics.md#security-groups), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md) |
| **SES** | Amazon Simple Email Service. | [1.1.8](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.8-event-driven-applications.md) |
| **Session token** | Third part of temporary credentials; must accompany the key pair. | [2.1.7](../domain-2-security/task-2.1-authentication-and-authorization/2.1.7-assuming-iam-roles.md) |
| **SHA / SHA-256** | Secure Hash Algorithm (256-bit). A Git commit SHA is the commit's hash ID. | [Developer basics](developer-basics.md#hashing-and-hmac) |
| **SigV4** | Signature Version 4 request signing. | [Developer basics](developer-basics.md#request-signing-sigv4), [2.1.5](../domain-2-security/task-2.1-authentication-and-authorization/2.1.5-authenticated-requests.md) |
| **SLO** | Service level objective. | [4.2.3](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.3-alerts-and-notifications.md) |
| **SnapStart** | Lambda feature that snapshots an initialized environment to cut cold starts (Java, Python, .NET). | [4.3.2](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.2-memory-and-compute-tuning.md), [1.2.2](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.2-lambda-configuration.md) |
| **SNI** | Server Name Indication. Lets one TLS listener serve many certificates. | [Service basics](aws-service-basics.md#tls-certificates-and-sni) |
| **SNS** | Amazon Simple Notification Service. Pub/sub topics with fan-out and filter policies. | [1.1.8](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.8-event-driven-applications.md), [4.3.3](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.3-messaging-optimization.md) |
| **Sort key** | Second part of a DynamoDB composite primary key; supports range queries. | [Developer basics](developer-basics.md#dynamodb-keys-and-indexes), [1.3.4](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md) |
| **Spec-driven development** | Generate requirements, design, and tasks before code (Kiro specs). | [1.1.9](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md) |
| **SQL** | Structured Query Language. | [Developer basics](developer-basics.md#relational-vs-nosql) |
| **SQS** | Amazon Simple Queue Service. Standard (at-least-once) and FIFO queues. | [1.1.1](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.1-coupling-and-decoupling.md), [4.3.3](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.3-messaging-optimization.md) |
| **SSE-C / SSE-KMS / SSE-S3** | S3 server-side encryption with customer-provided keys / KMS keys / S3-managed keys. | [2.2.3](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.3-client-side-vs-server-side-encryption.md) |
| **SSE-SQS** | SQS server-side encryption with SQS-owned keys. | [2.2.1](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.1-encryption-at-rest-and-in-transit.md) |
| **SSH / SSL / TLS** | Secure Shell / Secure Sockets Layer (old name) / Transport Layer Security. | [2.2.5](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.5-development-certificates-and-ssh-keys.md), [2.2.1](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.1-encryption-at-rest-and-in-transit.md) |
| **SSN** | Social Security number (US), a common PII type in masking questions. | [2.2.6](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.6-sensitive-data-detection-and-masking.md) |
| **SSO** | Single sign-on (IAM Identity Center). | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |
| **Stage (API Gateway)** | Named deployment of an API (`dev`, `prod`) with its own URL, settings, and stage variables. | [3.1.5](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md) |
| **Standard-IA** | S3 Standard-Infrequent Access storage class. | [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md) |
| **Step Functions** | AWS Step Functions. State machine workflows; Standard (exactly-once workflow execution, up to 1 year) or Express (up to 5 minutes; async at-least-once, sync at-most-once). | [1.1.2](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.2-communication-patterns.md) |
| **Sticky sessions** | Load balancer routes a client to the same target using a cookie. | [Service basics](aws-service-basics.md#sticky-sessions) |
| **STS** | AWS Security Token Service. Issues temporary credentials. | [2.1.7](../domain-2-security/task-2.1-authentication-and-authorization/2.1.7-assuming-iam-roles.md) |
| **Systems Manager** | AWS Systems Manager. Parameter Store, Session Manager, Run Command, and more. | [3.1.4](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.4-environment-specific-configuration.md), [2.2.5](../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.5-development-certificates-and-ssh-keys.md) |

## T

| Term | Meaning | Where it matters |
|---|---|---|
| **Task definition** | ECS blueprint for containers: image, CPU, memory, ports, task role, execution role. | [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md) |
| **Task role / task execution role** | ECS task role = permissions for your app code; task execution role = permissions for ECS to pull images and read secrets. | [2.1.6](../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md), [3.3.2](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md) |
| **Task token** | Step Functions callback pattern: the workflow pauses until something calls `SendTaskSuccess` with the token (human approval). | [1.1.2](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.2-communication-patterns.md), [2.3.4](../domain-2-security/task-2.3-secure-ai-services/2.3.4-ai-agent-security.md) |
| **TCP / UDP** | Transmission Control Protocol / User Datagram Protocol. | [Service basics](aws-service-basics.md#network-load-balancer) |
| **TOTP** | Time-based one-time password (authenticator app codes). | [2.1.2](../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md) |
| **Trust policy** | Resource policy on an IAM role that says who may assume it. | [2.1.7](../domain-2-security/task-2.1-authentication-and-authorization/2.1.7-assuming-iam-roles.md) |
| **TTL** | Time to live: automatic expiry (DynamoDB items, cache entries, DNS records). | [Developer basics](developer-basics.md#ttl-and-data-lifecycle), [1.3.1](../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md) |

## U

| Term | Meaning | Where it matters |
|---|---|---|
| **UI** | User interface. | [1.1.10](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md) |
| **URL** | Uniform resource locator. | [Developer basics](developer-basics.md#rest-and-http-methods) |
| **Usage plan** | API Gateway throttling and quota limits tied to API keys. | [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md), [3.1.5](../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md) |
| **User data** | EC2 bootstrap script that runs once at first boot as root. | [Service basics](aws-service-basics.md#user-data-and-instance-metadata) |
| **User pool** | Cognito user directory that signs users in and issues JWTs. | [2.1.1](../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md) |

## V

| Term | Meaning | Where it matters |
|---|---|---|
| **vCPU** | Virtual CPU. | [Service basics](aws-service-basics.md#launch-time-choices) |
| **Versions (Lambda)** | Immutable snapshots of function code and configuration (`$LATEST` is mutable). | [3.3.8](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.8-versions-and-releases.md) |
| **Visibility timeout** | Time an SQS message stays hidden after a consumer receives it (default 30 seconds); set above processing time to avoid duplicates. | [4.3.3](../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.3-messaging-optimization.md), [1.2.4](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md) |
| **VPC** | Amazon Virtual Private Cloud. | [Service basics](aws-service-basics.md#amazon-vpc-basics), [1.2.1](../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md) |
| **VPC endpoint** | Private access to AWS services from a VPC: gateway (S3, DynamoDB) or interface (PrivateLink). | [Service basics](aws-service-basics.md#vpc-endpoints-and-aws-privatelink), [2.3.2](../domain-2-security/task-2.3-secure-ai-services/2.3.2-ai-data-privacy-and-private-connectivity.md) |
| **VPN** | Virtual private network. | [Service basics](aws-service-basics.md#site-to-site-vpn-vs-direct-connect) |
| **VTL** | Velocity Template Language, used in API Gateway REST mapping templates. | [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md) |

## W

| Term | Meaning | Where it matters |
|---|---|---|
| **WAF** | AWS WAF (web application firewall). Rules on CloudFront, ALB, API Gateway, Cognito, AppSync. | [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md), [4.1.1](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.1-debugging-code-and-integrations.md) |
| **Waiter** | SDK helper that polls until a resource reaches a state. | [Developer basics](developer-basics.md#pagination-and-waiters), [1.1.5](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md) |
| **WebSocket** | Persistent two-way connection; API Gateway WebSocket APIs. | [Developer basics](developer-basics.md#graphql-and-websocket), [1.1.4](../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md) |

## X

| Term | Meaning | Where it matters |
|---|---|---|
| **X-Forwarded-For** | Header carrying the original client IP through a load balancer or proxy. | [Service basics](aws-service-basics.md#application-load-balancer) |
| **X-Ray** | AWS X-Ray. Distributed tracing with service maps, segments, subsegments, annotations, and sampling. | [4.2.4](../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.4-distributed-tracing.md), [4.1.2](../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.2-metrics-logs-and-traces.md) |

## Y

| Term | Meaning | Where it matters |
|---|---|---|
| **YAML** | YAML Ain't Markup Language. Format for CloudFormation, SAM, buildspec, and appspec files. | [3.3.3](../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md) |

## Z

| Term | Meaning | Where it matters |
|---|---|---|
| **Zone apex** | The root of a domain (`example.com`). Use an Alias record, not a CNAME. | [Service basics](aws-service-basics.md#cname-vs-alias) |

## Sources

- Merged from: `archive/dva-c03-study-notes/00-abbreviations-glossary.md` (every abbreviation), `archive/dva-c03-study-notes/00-developer-basics.md`, `archive/dva-c03-study-notes/05-cheat-sheet.md`, and the service-note chapters in `archive/aws-dva-notes/Markdown Files/` (chapters 2, 3, 4, 6, 7, 9 for infrastructure terms).
- Definitions for limits and behaviors follow the linked skill notes, which cite current AWS documentation.
