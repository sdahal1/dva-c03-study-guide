# AWS Certified Developer - Associate (DVA-C03) exam guide

> Study summary based on the official [AWS DVA-C03 exam guide](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html), accessed November 26, 2026.

## Exam status

AWS opened registration for DVA-C03 on October 27, 2026. November 30, 2026 is the last day candidates can take DVA-C02. This workspace targets **DVA-C03** for the exam scheduled on **December 30, 2026**.

AWS reviews the guide and publishes revisions at least one month before they affect the exam. Check the official guide again before the exam.

## Purpose

DVA-C03 tests whether a developer can develop, test, deploy, optimize, secure, and debug applications on AWS. The updated exam includes AI-assisted development tools.

AWS expects candidates to know how to:

- Develop and optimize applications on AWS.
- Use AI-assisted development tools.
- Package and deploy applications through CI/CD workflows.
- Secure application code and data.
- Diagnose and resolve application problems.

## Target candidate

AWS recommends at least one year of hands-on experience developing and maintaining applications with AWS services.

### General knowledge

You should understand:

- One or more high-level programming languages.
- Application lifecycle management.
- Cloud application development basics.
- Development tools, including AI-assisted tools and workflows.

### AWS knowledge

You should be able to:

- Develop and secure applications with AWS service APIs, the AWS CLI, and AWS SDKs.
- Use an existing CI/CD pipeline to deploy applications.
- Apply AI tools and services during development, testing, and release.
- Integrate managed AI services into applications.

## Tasks outside the target role

The exam does not expect you to perform these tasks:

- Design distributed systems, microservices, or database schemas.
- Design and create CI/CD pipelines.
- Administer IAM users and groups.
- Administer servers or operating systems.
- Select foundation models, engineer prompts, or design RAG architectures.
- Design AI safety frameworks, guardrails, or governance.
- Design and create ETL pipelines.
- Design AWS network infrastructure.

The exam can still test the developer-level use of related services. For example, you need to use a CI/CD pipeline even though you do not need to design one.

## Question format

| Item | Detail |
| --- | --- |
| Multiple-choice questions | One correct answer and three distractors |
| Multiple-response questions | Two or more correct answers among at least five choices; you must select every correct answer |
| Scored questions | 50 |
| Unscored questions | 15, mixed into the exam and not identified |
| Total questions | 65 |
| Penalty for guessing | None; unanswered questions count as incorrect |

## Scoring

- AWS reports a scaled score between 100 and 1,000.
- The minimum passing score is **720**.
- The exam uses compensatory scoring. You pass based on your overall score, so you do not need to pass each domain on its own.
- The score report can include broad performance classifications by domain. These classifications do not represent exact domain scores.

## Domains and weights

| Domain | Weight of scored content |
| --- | ---: |
| 1. Development with AWS Services | 30% |
| 2. Security | 26% |
| 3. Testing and Deployment | 22% |
| 4. Troubleshooting and Optimization | 22% |

Use these weights to divide study time. The percentages apply to scored content and do not guarantee an exact number of questions from each domain.

## Detailed domain outline

The following sections paraphrase the official skills into a study checklist. Skill numbers match the AWS exam guide.

### Domain 1: Development with AWS Services (30%)

Source: [Official Domain 1 details](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain1.html)

#### Task 1.1: Build application code on AWS

- **1.1.1:** Recognize tight and loose coupling, then select AWS services that decouple components.
- **1.1.2:** Choose synchronous, asynchronous, or event-driven communication. Know common patterns involving API Gateway, SQS, and EventBridge.
- **1.1.3:** Build resilient applications with bounded retries, exponential backoff, failure handling, and graceful degradation.
- **1.1.4:** Create and maintain APIs, including request and response transformations and input validation.
- **1.1.5:** Call AWS services from application code through service APIs and AWS SDKs.
- **1.1.6:** Process and manage streaming data with AWS services.
- **1.1.7:** Serialize and deserialize data exchanged between application code and AWS services.
- **1.1.8:** Implement event-driven applications with services such as EventBridge, S3 Event Notifications, SQS, and SNS.
- **1.1.9:** Use AI-assisted tools for code generation, review, completion, refactoring, optimization, and security scanning.
- **1.1.10:** Integrate managed AWS AI services into applications.

#### Task 1.2: Develop AWS Lambda functions

- **1.2.1:** Give Lambda functions access to private resources inside a VPC.
- **1.2.2:** Configure environment variables, memory, concurrency, timeouts, runtimes, handlers, layers, and extensions.
- **1.2.3:** Handle invocation lifecycles and failures with destinations, retries, and dead-letter queues.
- **1.2.4:** Integrate Lambda with services such as API Gateway and DynamoDB through code, event source mappings, triggers, and destinations.

#### Task 1.3: Use data stores in application development

- **1.3.1:** Manage data lifecycles with S3 Lifecycle rules, TTL settings, and retention periods.
- **1.3.2:** Select ElastiCache, DynamoDB, S3, RDS, or another suitable store based on access patterns and application requirements.
- **1.3.3:** Choose a caching approach that fits the selected data store and freshness requirements.
- **1.3.4:** Configure DynamoDB partition keys, consistency, queries, scans, attributes, and indexes for application access patterns.

### Domain 2: Security (26%)

Source: [Official Domain 2 details](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain2.html)

#### Task 2.1: Implement authentication and authorization

- **2.1.1:** Implement identity federation for application users or workforce identities.
- **2.1.2:** Apply access controls such as fine-grained permissions, temporary security tokens, and MFA.
- **2.1.3:** Configure secure programmatic access to AWS.
- **2.1.4:** Provide time-limited access with mechanisms such as S3 presigned URLs and CloudFront signed access.
- **2.1.5:** Make authenticated requests to AWS services.
- **2.1.6:** Define permissions for IAM users, roles, services, and resources according to least privilege.
- **2.1.7:** Assume IAM roles and use the resulting temporary credentials.

#### Task 2.2: Encrypt and protect sensitive data

- **2.2.1:** Apply encryption both at rest and in transit.
- **2.2.2:** Manage public and private certificates with ACM and AWS Private CA.
- **2.2.3:** Choose and implement client-side or server-side encryption.
- **2.2.4:** Use encryption keys to protect and recover data, including KMS-based patterns.
- **2.2.5:** Generate certificates and SSH keys for development needs.
- **2.2.6:** Detect, mask, and restrict access to sensitive data such as PII and PHI.
- **2.2.7:** Store and retrieve sensitive values with managed secret services.

#### Task 2.3: Secure the use of AI services

- **2.3.1:** Request, grant, restrict, and manage access to AI services.
- **2.3.2:** Protect data sent to AI services with private connectivity and data-use controls, including VPC endpoints and no-training commitments.
- **2.3.3:** Filter model inputs and outputs for harmful content, sensitive information, denied topics, prompt injection, and manipulation attempts.
- **2.3.4:** Secure AI agents with tool authorization, isolated sessions, and human approval before sensitive actions.
- **2.3.5:** Prevent prompts, responses, PII, PHI, and other sensitive content from leaking through monitoring logs.

### Domain 3: Testing and Deployment (22%)

Source: [Official Domain 3 details](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain3.html)

#### Task 3.1: Prepare deployable application artifacts

- **3.1.1:** Package code dependencies, configuration files, environment variables, and related artifacts correctly.
- **3.1.2:** Build and manage container images with Dockerfiles, ECR repositories, image tags, and lifecycle policies.
- **3.1.3:** Use code repositories as part of deployment workflows.
- **3.1.4:** Keep environment-specific configuration in services such as AppConfig, Parameter Store, Secrets Manager, API Gateway, and Lambda.
- **3.1.5:** Configure API Gateway resources such as stages, endpoints, and stage-specific URLs.

#### Task 3.2: Test applications in AWS environments

- **3.2.1:** Develop and validate tests with tools and templates such as AWS SAM, AWS CDK, and CloudFormation.
- **3.2.2:** Replace external dependencies with mock APIs through services and tools such as API Gateway.
- **3.2.3:** Create representative test events, including JSON payloads for Lambda functions and API Gateway APIs.
- **3.2.4:** Create isolated test environments with Lambda aliases, container tags, Amplify branches, Copilot environments, and API Gateway APIs.
- **3.2.5:** Use AWS AI tools to generate tests, run repetitive test workflows, analyze results, automate regression testing, and find coverage gaps.

#### Task 3.3: Deploy through AWS CI/CD services

- **3.3.1:** Select efficient Lambda packaging methods, including ZIP archives, container images, and layers.
- **3.3.2:** Deploy containers through ECS task definitions, EKS manifests, Fargate launch types, and rolling updates.
- **3.3.3:** Define and deploy infrastructure with CloudFormation and AWS SAM templates.
- **3.3.4:** Deploy application revisions to target environments with CodeDeploy.
- **3.3.5:** Use CodePipeline to connect source repositories and promote releases through multiple environments.
- **3.3.6:** Choose and implement blue/green, canary, or rolling deployment strategies.
- **3.3.7:** Configure automatic or manual rollback through services such as CloudFormation and CodeDeploy.
- **3.3.8:** Manage versions and releases with source-control branches, labels, and related features.
- **3.3.9:** Use AWS AI tools for deployment approvals, environment provisioning, and post-deployment validation.

### Domain 4: Troubleshooting and Optimization (22%)

Source: [Official Domain 4 details](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain4.html)

#### Task 4.1: Assist with root-cause analysis

- **4.1.1:** Debug application code and service integrations to identify defects.
- **4.1.2:** Interpret metrics, logs, and distributed traces.
- **4.1.3:** Query logs to isolate relevant events and evidence.
- **4.1.4:** Diagnose deployment failures from build and deployment output logs.
- **4.1.5:** Use AWS AI tools to analyze errors and propose troubleshooting steps.

#### Task 4.2: Apply observability practices

- **4.2.1:** Implement structured logging that records application behavior, state, events, and user actions.
- **4.2.2:** Write application code that emits useful custom metrics.
- **4.2.3:** Send alerts for conditions such as quota thresholds and completed deployments.
- **4.2.4:** Instrument distributed tracing with AWS services and tools.
- **4.2.5:** Monitor workload health through application health checks and container liveness and readiness probes.

#### Task 4.3: Optimize applications

- **4.3.1:** Apply concurrency controls such as Lambda provisioned concurrency, EC2 Auto Scaling, thread pools, and connection pools.
- **4.3.2:** Tune application memory and compute capacity.
- **4.3.3:** Optimize messaging with SQS queues, SNS fan-out, and suitable delivery patterns.
- **4.3.4:** Implement caching with ElastiCache, DynamoDB Accelerator (DAX), and CloudFront.
- **4.3.5:** Detect performance bottlenecks with services such as CloudWatch.
- **4.3.6:** Use AWS AI tools to find bottlenecks, inefficient code, and resource optimization opportunities.

## Technologies and concepts

Source: [Official technologies and concepts](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-technologies-concepts.html)

AWS says these categories can appear on the exam. The list is non-exhaustive, and its order does not indicate exam weight.

- Analytics
- Application integration
- Compute
- Containers
- Database
- Developer tools
- Machine learning and AI
- Management and governance
- Networking and content delivery
- Security, identity, and compliance
- Storage

## In-scope AWS services

Source: [Official in-scope services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-in-scope-services.html)

AWS describes this list as non-exhaustive and subject to change.

### Analytics

- Amazon Athena
- Amazon Data Firehose
- Amazon Kinesis
- Amazon OpenSearch Service

### Application integration

- Amazon EventBridge
- Amazon SNS
- Amazon SQS
- AWS Step Functions

### Compute

- Amazon EC2
- AWS Elastic Beanstalk
- AWS Lambda

### Containers

- Amazon ECR
- Amazon ECS
- Amazon EKS

### Database

- Amazon Aurora
- Amazon DynamoDB
- Amazon ElastiCache
- Amazon RDS

### Developer tools

- AWS Amplify
- AWS CloudShell
- AWS CodeArtifact
- AWS CodeBuild
- AWS CodeDeploy
- AWS CodePipeline
- AWS X-Ray

### Machine learning and AI

- Amazon Bedrock
- Amazon Bedrock AgentCore
- Amazon Quick
- Kiro

> The official list names **Amazon Quick**, not Amazon Q Developer or Amazon Quick Sight.

### Management and governance

- AWS AppConfig
- AWS CDK
- AWS CloudFormation
- AWS CloudTrail
- Amazon CloudWatch
- Amazon CloudWatch Logs
- AWS CLI
- AWS SAM
- AWS Systems Manager

### Networking and content delivery

- Amazon API Gateway
- Amazon CloudFront
- Elastic Load Balancing (ELB)
- AWS PrivateLink
- Amazon Route 53
- Amazon VPC

### Security, identity, and compliance

- AWS Certificate Manager (ACM)
- Amazon Cognito
- AWS IAM
- AWS KMS
- AWS Private CA
- AWS Secrets Manager
- AWS STS
- AWS WAF

### Storage

- Amazon EBS
- Amazon EFS
- Amazon S3
- Amazon S3 Glacier

## DVA-C02 and DVA-C03 comparison

Source: [Official DVA-C02 and DVA-C03 comparison](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-comparison.html)

DVA-C03 applies to exams delivered from December 1, 2026. DVA-C02 remains available through November 30, 2026.

### Weight changes

| Domain area | DVA-C02 | DVA-C03 | Change |
| --- | ---: | ---: | ---: |
| Development with AWS Services | 32% | 30% | -2 points |
| Security | 26% | 26% | No change |
| Deployment / Testing and Deployment | 24% | 22% | -2 points |
| Troubleshooting and Optimization | 18% | 22% | +4 points |

### Content added in DVA-C03

- **Task 1.1:** AI-assisted code generation, review, completion, refactoring, optimization, security scanning, and integration of managed AWS AI services.
- **Task 2.1:** Secure, time-limited access through mechanisms such as S3 presigned URLs and CloudFront signed access.
- **Task 2.3:** A new task covering AI service access, private connectivity, model data-use controls, input and output filtering, prompt-injection protection, agent authorization, session isolation, human approval, and sensitive AI logs.
- **Task 3.1:** Building and managing container images with Dockerfiles, ECR, tags, and lifecycle policies.
- **Task 3.2:** AI-generated tests, automated execution, result analysis, regression automation, and coverage checks.
- **Task 3.3:** ECS, EKS, and Fargate deployments, rolling updates, and AI-supported CI/CD workflows.
- **Task 4.1:** AI-assisted error analysis and troubleshooting suggestions.
- **Task 4.3:** AI-assisted detection of bottlenecks, resource waste, and inefficient code.

### Content moved within the outline

| Topic | DVA-C02 location | DVA-C03 location |
| --- | --- | --- |
| Unit testing with AWS SAM | Domain 1, Task 1.1 | Domain 3, Task 3.2 |
| Lambda performance tuning | Domain 1, Task 1.2 | Domain 4, Task 4.3 |
| Data serialization and deserialization | Domain 1, Task 1.3 | Domain 1, Task 1.1 |
| API Gateway stages and custom domains | Domain 3, Task 3.4 | Domain 3, Task 3.1 |
| CDK, SAM, and CloudFormation templates | Domain 3, Task 3.3 | Domain 3, Task 3.3 |

These moves change where AWS classifies the material, not whether you need to study it.

### Structural consolidation

AWS combined overlapping skills to reduce repetition. A shorter list does not mean the underlying material disappeared.

- Four separate DynamoDB skills became one comprehensive DynamoDB skill.
- Encryption and sensitive-data coverage moved into one seven-skill Task 2.2. The new Task 2.3 now covers AI security.
- Two testing tasks containing 11 skills became one five-skill testing task that also includes AI-assisted testing.
- Eight observability skills became five broader skills.
- Nine optimization skills became six broader skills.

## Official detail pages

- [Domain 1: Development with AWS Services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain1.html)
- [Domain 2: Security](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain2.html)
- [Domain 3: Testing and Deployment](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain3.html)
- [Domain 4: Troubleshooting and Optimization](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain4.html)
- [Technologies and concepts](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-technologies-concepts.html)
- [AWS services mentioned on the exam](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-service-mentions.html)
- [In-scope AWS services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-in-scope-services.html)
- [Out-of-scope AWS services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-out-of-scope-services.html)
- [DVA-C02 and DVA-C03 comparison](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-comparison.html)
