# DVA-C03 cheat sheet: clue to answer

One-page review for the last week and exam morning. Each row pairs a scenario clue with the answer the exam expects and links to the skill note that explains it. Rows follow the official outline, domain by domain and task by task. Read it top to bottom in about 30 minutes; follow a skill link only when a row surprises you.

For pacing, question-reading tactics, and the final-week plan, see [exam strategy](exam-strategy.md).

## Abbreviations used

| Short form | Meaning |
|---|---|
| ABAC | Attribute-based access control |
| ACM | AWS Certificate Manager |
| ALB / ELB | Application Load Balancer / Elastic Load Balancing |
| AMI | Amazon Machine Image |
| API | Application programming interface |
| ARN | Amazon Resource Name |
| CA | Certificate authority |
| CDK | AWS Cloud Development Kit |
| CI/CD | Continuous integration and continuous delivery |
| CLI | AWS Command Line Interface |
| CNAME | Canonical name (DNS record) |
| CORS | Cross-origin resource sharing |
| DAX | DynamoDB Accelerator |
| DLQ | Dead-letter queue |
| EC2 | Amazon Elastic Compute Cloud |
| ECR / ECS / EKS | Amazon Elastic Container Registry / Elastic Container Service / Elastic Kubernetes Service |
| EMF | CloudWatch Embedded Metric Format |
| ENI | Elastic network interface |
| ESM | Lambda event source mapping |
| FIFO | First-in, first-out |
| GSI / LSI | Global secondary index / local secondary index |
| IAM | AWS Identity and Access Management |
| JWT | JSON Web Token (JSON is JavaScript Object Notation) |
| KMS | AWS Key Management Service |
| MFA | Multi-factor authentication |
| mTLS | Mutual Transport Layer Security |
| NAT | Network address translation |
| OAC | CloudFront origin access control |
| OIDC / SAML | OpenID Connect / Security Assertion Markup Language |
| PII / PHI | Personally identifiable information / protected health information |
| RCU / WCU | Read capacity unit / write capacity unit |
| RDS | Amazon Relational Database Service |
| SAM | AWS Serverless Application Model |
| SCP | Service control policy (AWS Organizations) |
| SDK | AWS software development kit |
| SG | Security group |
| SigV4 | AWS Signature Version 4 request signing |
| SNS / SQS | Amazon Simple Notification Service / Simple Queue Service |
| SSE-S3 / SSE-KMS / SSE-C | S3 server-side encryption with S3-managed keys / KMS keys / customer-provided keys |
| SSH | Secure Shell |
| STS | AWS Security Token Service |
| TTL | Time to live |
| VPC | Amazon Virtual Private Cloud |
| VTL | Apache Velocity Template Language (API Gateway mapping templates) |
| WAF | AWS WAF (web application firewall) |

## Domain 1: Development with AWS Services (30%)

### Task 1.1: Build application code on AWS

| Clue | Answer | Skill |
|---|---|---|
| Decouple a producer from a slow consumer, buffer bursts | **SQS** | [1.1.1] |
| One event, many independent consumers | **SNS to multiple SQS queues (fan-out)**, or an EventBridge rule with several targets | [1.1.1] |
| Long tasks make an Elastic Beanstalk web app slow | **Beanstalk worker environment** (pulls from SQS) | [1.1.1] |
| Choreography vs orchestration | Services react to events on their own (**EventBridge, SNS**) vs a central coordinator (**Step Functions**) | [1.1.1] |
| Multi-step workflow with retries and human approval | **Step Functions** with a task token | [1.1.2] |
| Pause a workflow until a human or external system answers | **`.waitForTaskToken`** (Standard workflows only) | [1.1.2] |
| Step Functions: payments, non-idempotent steps, runs up to a year | **Standard**: exactly-once workflow execution, up to 1 year | [1.1.2] |
| Step Functions: high volume, short, idempotent steps | **Express**: up to 5 minutes; asynchronous is at-least-once, synchronous is at-most-once | [1.1.2] |
| Task state vs Pass state | **Task** does the work; **Pass** only passes or shapes input | [1.1.2] |
| Server pushes messages to browsers | **API Gateway WebSocket API** plus the `@connections` callback | [1.1.2] |
| Throttling or 5xx errors from AWS APIs | **Exponential backoff with jitter** (the SDK retries for you) | [1.1.3] |
| Third-party API is flaky | **Bounded retries with backoff and jitter, timeouts, circuit breaker** (stop calling after N failures, try again later) | [1.1.3] |
| Same request processed twice must not double-charge | **Idempotency**: idempotency key or conditional write | [1.1.3] |
| Step Functions error handling | **`Retry`** first, then **`Catch`**; keep the error with `ResultPath: "$.error"` | [1.1.3] |
| API Gateway request flow | Method request, integration request, backend, integration response, method response | [1.1.4] |
| Client must send a parameter; reject bad requests before Lambda | **Method request + request validator + model** | [1.1.4] |
| Lambda receives the whole HTTP request | **`AWS_PROXY`** (Lambda proxy integration) | [1.1.4] |
| Any URL backend (Beanstalk, ALB, on premises), passed through unchanged | **`HTTP_PROXY`**. `AWS_PROXY` is for Lambda only | [1.1.4] |
| Reshape request or response, SOAP XML backend | **`AWS` or `HTTP` non-proxy integration + mapping templates (VTL)** | [1.1.4] |
| Put API data straight into SQS, Kinesis, or DynamoDB | **API Gateway AWS service integration** (no Lambda) | [1.1.4] |
| Charge customers, per-client quotas | **Usage plan + API key** (`x-api-key` header). API keys identify callers; they are not authorization | [1.1.4] |
| New API key gets 403 | Key is not in a usage plan: **`CreateUsagePlanKey`** | [1.1.4] |
| CORS error with Lambda proxy integration | **Lambda returns `Access-Control-Allow-Origin`** in its response headers | [1.1.4] |
| Cheapest API, JWT authorizer, no caching needed | **HTTP API**. Caching, API keys, request validation, and mapping templates need a **REST API** | [1.1.4] |
| SQL injection or cross-site scripting against an API | **AWS WAF** web ACL on API Gateway, ALB, or CloudFront | [1.1.4] |
| Wait until a resource is ready | **SDK waiter** | [1.1.5] |
| Large list results, CLI output too long or timing out | **Paginator**; CLI `--page-size` and `--max-items` | [1.1.5] |
| Upload large objects | **Multipart upload**: recommended above 100 MB, required above 5 GB | [1.1.5] |
| Faster long-distance uploads to S3 | **S3 Transfer Acceleration** | [1.1.5] |
| Download parts of a big object in parallel | **Byte-range fetches** | [1.1.5] |
| Browser on another domain calls S3 | **CORS configuration on the bucket being called** | [1.1.5] |
| Browser shell with the CLI already signed in | **AWS CloudShell** | [1.1.5] |
| Real-time ordered stream, replay, several consumers, order per key | **Kinesis Data Streams** (partition key) | [1.1.6] |
| Load a stream into S3, Redshift, or OpenSearch with no code | **Amazon Data Firehose** (near real time, buffered) | [1.1.6] |
| Kinesis `ProvisionedThroughputExceededException` | More shards, a higher-cardinality partition key, backoff | [1.1.6] |
| Strict order for one producer's Kinesis records | **`SequenceNumberForOrdering`** | [1.1.6] |
| Lambda event `body` won't parse | It is a **JSON string**; parse it first | [1.1.7] |
| Kinesis record data unreadable in Lambda | **Base64-decode** it | [1.1.7] |
| Route events by content, SaaS partner events | **EventBridge rule** (event pattern) on an event bus | [1.1.8] |
| Replay past events to test or recover | **EventBridge archive and replay** | [1.1.8] |
| Know the shape of events, generate code for them | **EventBridge schema registry** (code bindings) | [1.1.8] |
| Point-to-point source, filter, enrich, target, no code | **EventBridge Pipes** | [1.1.8] |
| Send events to another account | **Cross-account event bus** (resource policy on the target bus) | [1.1.8] |
| Run code on a schedule or react to AWS events | **EventBridge** rules or EventBridge Scheduler | [1.1.8] |
| Run code when an object is uploaded | **S3 Event Notifications** to Lambda, SQS, SNS, or EventBridge | [1.1.8] |
| Thumbnails on upload | **S3 event to Lambda**, writing output to a different prefix or bucket to avoid a loop | [1.1.8] |
| AI coding assistant: requirements, then design, then tasks | **Kiro specs** | [1.1.9] |
| Run an AI action automatically on file save | **Kiro hooks**; project-wide rules go in **steering** files | [1.1.9] |
| AI help to write code, generate unit tests, review or scan code, explain errors | **Kiro** (the named AI development tool for DVA-C03) | [1.1.9] |
| Call a foundation model from code | **Amazon Bedrock** `Converse` or `InvokeModel` on the `bedrock-runtime` client | [1.1.10] |
| Same code for several models, plus tool use | **Bedrock Converse API** | [1.1.10] |
| Question answering over company documents with citations | **Bedrock Knowledge Bases** | [1.1.10] |
| Agent calls your API, with user confirmation | **Bedrock Agents action group** with user confirmation or return of control | [1.1.10] |
| Deploy and run agents with session isolation and managed tools | **Amazon Bedrock AgentCore** | [1.1.10] |
| Cheap, large, offline inference | **Bedrock batch inference** | [1.1.10] |

### Task 1.2: Develop AWS Lambda functions

| Clue | Answer | Skill |
|---|---|---|
| Lambda must reach a private RDS database | **Attach the function to the VPC** (subnets + SG; the role needs ENI permissions) | [1.2.1] |
| VPC-attached Lambda lost internet access | **NAT gateway** in a public subnet, route from the private subnets, SG outbound allowed | [1.2.1] |
| Private subnet reaches S3 or DynamoDB without internet | **Gateway VPC endpoint** | [1.2.1] |
| Private subnet reaches other AWS services | **Interface VPC endpoint** (AWS PrivateLink) | [1.2.1] |
| Shared libraries across functions | **Layers** (mounted at `/opt`, up to 5 per function) | [1.2.2] |
| Reuse database connections and SDK clients | Create them **outside the handler** | [1.2.2] |
| Large temporary files or a scratch cache | **`/tmp`** (512 MB to 10 GB) | [1.2.2] |
| Language without a managed runtime | **Custom runtime** (`provided` runtimes). Python, Node.js, Java, .NET, and Ruby have managed runtimes | [1.2.2] |
| Find a function's log stream from code | **`context.log_stream_name`** | [1.2.2] |
| Invoke asynchronously | `Invoke` with **`InvocationType=Event`** | [1.2.3] |
| Keep failed asynchronous events | **On-failure destination** (carries error context) or a **DLQ** on SQS or SNS (event only) | [1.2.3] |
| Notify another service when an async invocation succeeds | **On-success destination** | [1.2.3] |
| Poison record blocks a Kinesis or DynamoDB stream | **Bisect batch on error + maximum retry attempts or record age + on-failure destination** | [1.2.3] |
| Retry only the failed SQS messages in a batch | **`ReportBatchItemFailures`** (partial batch response) | [1.2.3] |
| Lambda is throttled | **429 `TooManyRequestsException`** | [1.2.3] |
| S3, SNS, or EventBridge can't invoke the function | Add a **resource-based policy** statement (`lambda:InvokeFunction`) | [1.2.4] |
| HTTPS endpoint for one function, no API Gateway | **Lambda function URL** | [1.2.4] |
| Kinesis or DynamoDB stream concurrency | **One concurrent batch per shard**, more with the parallelization factor | [1.2.4] |
| Process only some stream or queue records | **ESM filter criteria** | [1.2.4] |
| Lambda floods the database with connections | **RDS Proxy** | [1.2.4] |

### Task 1.3: Use data stores in application development

| Clue | Answer | Skill |
|---|---|---|
| Delete old DynamoDB items at no cost | **DynamoDB TTL** (attribute holds epoch seconds) | [1.3.1] |
| Move objects to cheaper storage or delete after N days | **S3 Lifecycle rules** (Standard-IA, S3 Glacier storage classes, expiration) | [1.3.1] |
| Serverless SQL on S3 | **Amazon Athena** (faster and cheaper with Parquet, compression, partitions) | [1.3.2] |
| Search any field, partial match, on DynamoDB data | **DynamoDB Streams to Lambda to Amazon OpenSearch Service** | [1.3.2] |
| RDS failover vs read scaling | **Multi-AZ**: synchronous standby for failover. **Read replica**: asynchronous copy for reads | [1.3.2] |
| Read-after-write on S3 | S3 is **strongly consistent** | [1.3.2] |
| Session store shared by many instances | **ElastiCache for Redis** (or DynamoDB with TTL) | [1.3.2] |
| Multithreaded, simple key-value cache | **ElastiCache for Memcached** | [1.3.2] |
| Cache only on read; stale data acceptable | **Lazy loading** (cache-aside) | [1.3.3] |
| Keep the cache closer to the database | **Write-through + TTL**. It reduces staleness, but failed or reordered writes and invalidation bugs can still leave stale entries | [1.3.3] |
| Cache SQL query results | **ElastiCache** with lazy loading | [1.3.3] |
| Query by customer and date range | **Partition key `customerId`, sort key date, `Query` with `BETWEEN`** | [1.3.4] |
| Query by a different attribute, added later | **GSI** (eventually consistent only, own capacity) | [1.3.4] |
| Same partition key, different sort key, strongly consistent reads | **LSI** (only at table creation) | [1.3.4] |
| GSI is throttled | **Base-table writes are throttled too** | [1.3.4] |
| Hot partition | **Higher-cardinality partition key** | [1.3.4] |
| Leaderboard per game | **GSI**: partition key game, sort key score | [1.3.4] |
| Prevent overwrite, optimistic locking | **Condition expression** (`attribute_not_exists` or a version attribute) | [1.3.4] |
| All-or-nothing writes across items | **`TransactWriteItems`** (up to 100 actions, 4 MB) | [1.3.4] |
| Microsecond DynamoDB reads | **DAX** | [1.3.4] |
| React to item changes | **DynamoDB Streams + Lambda** | [1.3.4] |
| Old and new item in the stream record | **`NEW_AND_OLD_IMAGES`** (previous value only: `OLD_IMAGE`) | [1.3.4] |
| Return only some attributes | **`ProjectionExpression`** | [1.3.4] |
| Filter after read | **`FilterExpression`** (still consumes RCU for everything read) | [1.3.4] |
| Scan gets slow as the table grows | **`Query` instead**, smaller page size | [1.3.4] |
| Faster big scan without hurting other traffic | **Rate-limited parallel scan** | [1.3.4] |
| `UnprocessedKeys` or `UnprocessedItems` | **Retry with exponential backoff** (the SDK helps) | [1.3.4] |
| Newest items first | **`ScanIndexForward=false`** | [1.3.4] |
| Capacity used per table and index | **`ReturnConsumedCapacity=INDEXES`** | [1.3.4] |
| On-demand capacity mode | Removes capacity planning, but can still throttle on hot partitions, sudden spikes far beyond the previous peak, or table quotas | [1.3.4] |

### Domain 1 numbers

| Item | Value | Skill |
|---|---|---|
| API Gateway integration timeout | **29 s** default for REST APIs; Regional and private REST APIs can request more; HTTP APIs fixed at 30 s | [1.1.4] |
| API Gateway payload | **10 MB** | [1.1.4] |
| API Gateway account throttle | **10,000 requests per second**, 5,000 burst (429 when exceeded) | [1.1.4] |
| Kinesis shard | **1 MB/s or 1,000 records/s in**, **2 MB/s out**, 5 read calls/s; `PutRecords` up to 500 records | [1.1.6] |
| Kinesis retention | **24 hours** default, up to **365 days** | [1.1.6] |
| EventBridge `PutEvents` | **10 entries** per call | [1.1.8] |
| Step Functions duration | Standard **1 year**; Express **5 minutes** | [1.1.2] |
| Step Functions input or output per state | **256 KiB** | [1.1.2] |
| Lambda memory | **128 MB to 10,240 MB**; CPU scales with memory | [1.2.2] |
| Lambda timeout | **30 minutes (1,800 s)** per the current quotas page. Older material and many practice questions say 15 minutes | [1.2.2] |
| Lambda `/tmp` | **512 MB to 10,240 MB** | [1.2.2] |
| Lambda environment variables | **4 KB** total | [1.2.2] |
| Lambda package | **50 MB** zipped direct upload, **250 MB** unzipped including layers, **10 GB** container image, **5 layers** | [1.2.2] |
| Lambda code storage | **300 GB** per Region for .zip functions and layers (older material says 75 GB) | [1.2.2] |
| Lambda payload | Synchronous **6 MB** request and response; asynchronous **1 MB** (formerly 256 KB) | [1.2.3] |
| Lambda asynchronous retries | **2 retries**, event age up to **6 hours** | [1.2.3] |
| Lambda concurrency | **1,000** per Region by default; each function scales by 1,000 execution environments every 10 seconds | [1.2.2] |
| DynamoDB item | **400 KB** | [1.3.4] |
| RCU / WCU | 1 RCU = **1 strongly consistent read/s of 4 KB** (or 2 eventually consistent); 1 WCU = **1 write/s of 1 KB**; transactions cost **2x** | [1.3.4] |
| DynamoDB batches | `Query`/`Scan` pages up to **1 MB**; `BatchGetItem` **100 items / 16 MB**; `BatchWriteItem` **25 put or delete requests** | [1.3.4] |
| DynamoDB transactions | Up to **100 actions**, **4 MB** aggregate | [1.3.4] |
| DynamoDB indexes | **20 GSIs** (default), **5 LSIs** | [1.3.4] |
| DynamoDB partition | **3,000 RCU / 1,000 WCU** | [1.3.4] |
| DynamoDB Streams retention | **24 hours** | [1.3.4] |
| S3 object | Up to **50 TB** (older material says 5 TB); single `PUT` up to 5 GB | [1.1.5] |

## Domain 2: Security (26%)

### Task 2.1: Implement authentication and authorization

| Clue | Answer | Skill |
|---|---|---|
| App sign-up and sign-in, JWTs, hosted UI, social or SAML sign-in | **Cognito user pool** (federation through the user pool) | [2.1.1] |
| Users need AWS credentials (upload to S3), including guests | **Cognito identity pool** | [2.1.1] |
| In-house login, same identity across devices | **Developer-authenticated identities** (identity pool) | [2.1.1] |
| Only company email addresses may sign up | **Pre sign-up** Lambda trigger | [2.1.1] |
| Move users from a legacy system as they sign in | **Migrate user** trigger | [2.1.1] |
| Add claims to tokens | **Pre token generation** trigger | [2.1.1] |
| Sign-in on an ALB without code changes | ALB **`authenticate-cognito`** action on an **HTTPS** listener | [2.1.1] |
| CI from GitHub without stored keys | **IAM OIDC identity provider + role** | [2.1.1] |
| MFA only for risky sign-ins | **Cognito adaptive authentication** | [2.1.2] |
| Require MFA for CLI actions | `aws:MultiFactorAuthPresent` condition + **`GetSessionToken`** with MFA | [2.1.2] |
| Custom or third-party tokens authorize an API | **Lambda authorizer**: `TOKEN` reads one header; `REQUEST` reads headers, query strings, and more | [2.1.2] |
| Restrict an API to accounts, IP ranges, or a VPC | **API Gateway resource policy** (REST APIs) | [2.1.2] |
| Users see only their own S3 folder or DynamoDB items | Identity pool + **`${cognito-identity.amazonaws.com:sub}`** in the S3 prefix or **`dynamodb:LeadingKeys`** | [2.1.2] |
| Multi-tenant: each tenant sees only its data | **Tenant ID in the partition key** + `dynamodb:LeadingKeys`, per-tenant S3 prefix, **ABAC** (`aws:PrincipalTag`, session tags) | [2.1.2] |
| Fine-grained authorization inside the app | Read **JWT claims** (`sub`, groups, scopes) in the backend | [2.1.2] |
| No access keys on EC2 | **Instance profile** role | [2.1.3] |
| CLI without long-term keys | **IAM Identity Center** profile (`aws sso login`) | [2.1.3] |
| CLI credential lookup order | Command-line options, environment variables, config and credentials files, container role, instance profile | [2.1.3] |
| Which identity am I using? | **`aws sts get-caller-identity`** | [2.1.3] |
| Temporary download or upload link to a private object | **S3 presigned URL** (expires; uses the signer's permissions; dies when the signer's temporary credentials expire) | [2.1.4] |
| One private file through CloudFront | **CloudFront signed URL**; many files: **signed cookies** | [2.1.4] |
| S3 reachable only through CloudFront | **OAC** | [2.1.4] |
| Call an IAM-protected API from code | **SigV4 signing** (the SDK signs for you) | [2.1.5] |
| Identity pool users call API Gateway | **IAM authorization (SigV4)**. There is no "identity pool authorizer" | [2.1.5] |
| Service-to-service authentication | **IAM roles + SigV4**, or the **Cognito client credentials grant** (OAuth scopes) | [2.1.5] |
| Explicit deny vs allow | **Explicit deny always wins** | [2.1.6] |
| Same account vs cross-account access | Same account: identity **or** resource policy can allow. Cross-account: **both** must allow | [2.1.6] |
| AccessDenied when attaching a role to Lambda or EC2 | Missing **`iam:PassRole`** | [2.1.6] |
| Test permissions without running anything | **IAM policy simulator** or `--dry-run` | [2.1.6] |
| Per-user S3 folder for IAM users | **`${aws:username}`** policy variable | [2.1.6] |
| Read an encoded authorization failure | **`aws sts decode-authorization-message`** | [2.1.6] |
| Lambda AccessDenied calling another service | Fix the **execution role**. Who may invoke the function is the **resource policy** | [2.1.6] |
| ECS app permissions vs image pull and logs | App calls: **task role**. Pull from ECR, write logs, read secrets at start: **task execution role** | [2.1.6] |
| Access another account's resources | **Role in the account that owns the resources**; its trust policy names the caller; the caller needs `sts:AssumeRole` | [2.1.7] |
| Third-party access to your account | **Role + external ID** | [2.1.7] |
| `InvalidParameterValueException` on `CreateFunction` | The role's **trust policy** doesn't allow `lambda.amazonaws.com` | [2.1.7] |

### Task 2.2: Encrypt and protect sensitive data

| Clue | Answer | Skill |
|---|---|---|
| TLS or HTTPS | Encryption **in transit** only, never "at rest" | [2.2.1] |
| End-to-end HTTPS through CloudFront | **Viewer and origin** protocol policies both HTTPS | [2.2.1] |
| SQL Server or Oracle engine-level encryption | **Transparent Data Encryption (TDE)** | [2.2.1] |
| Certificate for CloudFront, an edge-optimized API, or a Cognito custom domain | **ACM certificate in us-east-1** | [2.2.2] |
| Free public certificate for ALB, CloudFront, or API Gateway | **ACM** (you can't install ACM public certificates directly on EC2) | [2.2.2] |
| Internal mTLS certificates | **AWS Private CA** | [2.2.2] |
| "Don't want to manage keys" | **SSE-S3** | [2.2.3] |
| Audit key use, control rotation | **SSE-KMS** | [2.2.3] |
| Company supplies the key, S3 encrypts | **SSE-C** (HTTPS + 3 headers; lose the key, lose the object) | [2.2.3] |
| AWS must never see plaintext | **Client-side encryption**. SSE-C still sends the key and plaintext to S3 | [2.2.3] |
| SSE-KMS with the default key, least configuration | Header **`x-amz-server-side-encryption: aws:kms`** only | [2.2.3] |
| Encrypt more than 4 KB with KMS | **`GenerateDataKey`** (envelope encryption) or the AWS Encryption SDK | [2.2.4] |
| Get a data key now, encrypt later | **`GenerateDataKeyWithoutPlaintext`** | [2.2.4] |
| Multipart upload with SSE-KMS gets AccessDenied | Missing **`kms:Decrypt`** (and `kms:GenerateDataKey`) | [2.2.4] |
| SSE-KMS throttled or costly on S3 | **S3 Bucket Keys** | [2.2.4] |
| KMS throttling from the Encryption SDK | **Data key caching** | [2.2.4] |
| Bind ciphertext to a tenant | **Encryption context** | [2.2.4] |
| Share an encrypted snapshot across accounts | **Customer managed key** + key policy grant (not the `aws/ebs` managed key) | [2.2.4] |
| Encrypt a CloudWatch Logs log group | **`aws logs associate-kms-key`** (CLI or API, not the console) | [2.2.4] |
| Keys or certificates for development | **EC2 key pairs** or `ssh-keygen`; private certificates from **AWS Private CA** | [2.2.5] |
| Local HTTPS development certificate | **Self-signed certificate** (OpenSSL) | [2.2.5] |
| Avoid distributing SSH keys | **Systems Manager Session Manager** or EC2 Instance Connect | [2.2.5] |
| PII in CloudWatch Logs | **CloudWatch Logs data protection policy** (masks; `logs:Unmask` to see originals) | [2.2.6] |
| Mask data per caller without keeping copies | **S3 Object Lambda** or masking in application code | [2.2.6] |
| Rotate a database password automatically | **Secrets Manager** | [2.2.7] |
| Cheap configuration and simple secrets in a hierarchy | **Parameter Store** (`SecureString`, `--with-decryption`) | [2.2.7] |
| Password in a CloudFormation template | **Dynamic reference** `{{resolve:secretsmanager:...}}` or `ManageMasterUserPassword` | [2.2.7] |
| Secrets in CodeBuild | buildspec **`env.secrets-manager`** or **`env.parameter-store`** | [2.2.7] |
| Secrets for Lambda | **Secrets Manager or Parameter Store with caching** (for example, the Parameters and Secrets Lambda extension). Encrypted environment variables are for non-secret configuration | [2.2.7] |

### Task 2.3: Secure the use of AI services

| Clue | Answer | Skill |
|---|---|---|
| Allow only approved models | IAM **`bedrock:InvokeModel`** scoped to model ARNs | [2.3.1] |
| Block a model organization-wide | **SCP deny** on the model ARN (denying `aws-marketplace:Subscribe` alone doesn't block use) | [2.3.1] |
| Require a guardrail on every call | **`bedrock:GuardrailIdentifier`** condition key | [2.3.1] |
| Reach Bedrock privately | **Interface VPC endpoint (PrivateLink)** + endpoint policy + **`aws:SourceVpce`** (`aws:SourceIp` is absent through endpoints) | [2.3.2] |
| Are prompts used to train models? | **No.** Bedrock doesn't use prompts or outputs to train models, and model providers can't see them | [2.3.2] |
| Redact PII, block harmful content, denied topics, prompt injection | **Bedrock Guardrails** (sensitive information filters with mask, content filters, prompt attack filter, denied topics) | [2.3.3] |
| Apply guardrails without calling a model, or to a non-Bedrock model | **`ApplyGuardrail`** API | [2.3.3] |
| Catch hallucinations against source documents | **Contextual grounding check** | [2.3.3] |
| Isolate each agent session | **AgentCore Runtime** (a microVM per session) | [2.3.4] |
| Control which tools an agent may call | **AgentCore Gateway + policy**, least-privilege tool roles | [2.3.4] |
| Agent acts only with the user's permissions | **AgentCore Identity**, OAuth token propagation | [2.3.4] |
| Human approval before a risky agent action | **User confirmation or return of control**, or Step Functions **`.waitForTaskToken`** | [2.3.4] |
| Log every prompt and response | **Model invocation logging** (off by default) to CloudWatch Logs or S3 | [2.3.5] |
| Invocation logs contain PII | **Encrypt with KMS, mask with a data protection policy, restrict access, set retention** | [2.3.5] |

### Domain 2 numbers

| Item | Value | Skill |
|---|---|---|
| Cognito tokens | ID and access tokens **1 hour** default; refresh token **30 days** default | [2.1.1] |
| STS `AssumeRole` session | **15 minutes to 12 hours** (1 hour default); role chaining caps at **1 hour** | [2.1.7] |
| S3 presigned URL | Up to **7 days** (SigV4 with IAM user credentials); shorter with temporary credentials | [2.1.4] |
| KMS `Encrypt` | **4 KB** of plaintext | [2.2.4] |
| Parameter Store value | Standard **4 KB**, Advanced **8 KB** | [2.2.7] |
| Secrets Manager secret | **64 KB** | [2.2.7] |

## Domain 3: Testing and Deployment (22%)

### Task 3.1: Prepare deployable application artifacts

| Clue | Answer | Skill |
|---|---|---|
| Share internal packages, cache public ones (npm, pip, Maven) | **AWS CodeArtifact** | [3.1.1] |
| CodeBuild configuration | **`buildspec.yml`** at the source root; phases install, pre_build, build, post_build; cache in S3 | [3.1.1] |
| Build reaches a private database | **CodeBuild VPC configuration** | [3.1.1] |
| Native dependencies fail in Lambda | Build them for the **Lambda architecture and OS** (for example, in a matching container) | [3.1.1] |
| Beanstalk configuration files | **`.ebextensions/*.config`** (YAML or JSON) at the bundle root; `env.yaml` for environment settings; `cron.yaml` for worker periodic tasks | [3.1.1] |
| Docker build fails in CodeBuild | Enable **privileged mode** | [3.1.2] |
| Old images pile up in ECR | **ECR lifecycle policy** | [3.1.2] |
| An image tag must never change | **ECR tag immutability** | [3.1.2] |
| Log Docker in to ECR | **`aws ecr get-login-password`** piped to `docker login` | [3.1.2] |
| Feature flags with gradual rollout and rollback | **AWS AppConfig** | [3.1.4] |
| Latest AMI ID in CloudFormation | **Systems Manager public parameter** `/aws/service/...` | [3.1.4] |
| API changes not visible | **Deploy the API to a stage** | [3.1.5] |
| Different Lambda alias per API stage | **Stage variable** `${stageVariables.alias}` + a Lambda permission per alias | [3.1.5] |
| API reachable only inside the VPC | **Private API endpoint + resource policy** | [3.1.5] |

### Task 3.2: Test applications in AWS environments

| Clue | Answer | Skill |
|---|---|---|
| Test CDK output | **CDK assertions** (fine-grained or snapshot tests) | [3.2.1] |
| Detect manual changes to stack resources | **CloudFormation drift detection** | [3.2.1] |
| API before the backend exists | **API Gateway mock integration** (`MOCK`) | [3.2.2] |
| Run Lambda locally with a sample event | **`sam local generate-event` + `sam local invoke`** (or `sam local start-api`) | [3.2.3] |
| Shareable test events | Lambda console **shareable test events** | [3.2.3] |
| `sam local` gets AccessDenied | Pass **`--profile`** for the right account | [3.2.3] |
| Branch preview environments | **Amplify branch deployments** and pull request previews | [3.2.4] |
| Simple multi-environment ECS deployments | **AWS Copilot** environments | [3.2.4] |
| AI-generated tests in the pipeline | **Kiro** (specs, hooks, headless runs) + **CodeBuild test reports** | [3.2.5] |

### Task 3.3: Deploy through AWS CI/CD services

| Clue | Answer | Skill |
|---|---|---|
| Package larger than 250 MB unzipped | **Container image** (up to 10 GB) | [3.3.1] |
| Inline Lambda code in a template | **`Code: ZipFile`** | [3.3.1] |
| ECS zero-downtime update | New **task definition revision** + rolling update (minimum and maximum healthy percent) | [3.3.2] |
| ECS placement: fewest instances / high availability / least configuration | **binpack** / **spread** / **random** | [3.3.2] |
| Group container instances by attribute | **Cluster query language** (`memberOf`) | [3.3.2] |
| Containers must share data | **One task definition with a shared volume** | [3.3.2] |
| No servers to manage for containers | **Fargate** | [3.3.2] |
| Many tasks on one EC2 host behind an ALB | **Dynamic host port mapping** | [3.3.2] |
| Easiest container hosting vs fine-grained control | Easiest: **Elastic Beanstalk**. Fine-grained: **ECS** | [3.3.2] |
| Value from another stack | **Outputs `Export`** + **`Fn::ImportValue`** | [3.3.3] |
| Reference in the same template | **`!Ref`** or **`!GetAtt`** | [3.3.3] |
| Template section that can't use Conditions | **Parameters** | [3.3.3] |
| Preview changes before an update | **Change sets** | [3.3.3] |
| Keep data when the stack or resource is deleted | **`DeletionPolicy: Retain`** or **`Snapshot`** | [3.3.3] |
| Deploy to many accounts and Regions | **StackSets** | [3.3.3] |
| Wait for EC2 setup to finish | **`cfn-signal` + `CreationPolicy`** | [3.3.3] |
| EC2 user data | Runs **once at first launch, as root** | [3.3.3] |
| Template is SAM | **`Transform: AWS::Serverless-2016-10-31`** | [3.3.3] |
| One SAM command to package and deploy | **`sam deploy`** (order: build, package, deploy) | [3.3.3] |
| SAM nested application from the Serverless Application Repository or S3 | **`AWS::Serverless::Application`** | [3.3.3] |
| Infrastructure in Python, TypeScript, or Java | **CDK** | [3.3.3] |
| First CDK deploy in a new account or Region | **`cdk bootstrap`** | [3.3.3] |
| CDK workflow | init, write code, build, **synth**, deploy | [3.3.3] |
| CodeDeploy hooks, EC2 or on premises | ApplicationStop, **DownloadBundle**, BeforeInstall, Install, AfterInstall, ApplicationStart, **ValidateService** | [3.3.4] |
| CodeDeploy hooks, Lambda | **`BeforeAllowTraffic`**, **`AfterAllowTraffic`** only | [3.3.4] |
| CodeDeploy hooks, ECS | BeforeInstall, AfterInstall, AfterAllowTestTraffic, **BeforeAllowTraffic**, AfterAllowTraffic | [3.3.4] |
| CodeDeploy deployment types | On premises: **in-place only**. ECS and Lambda: **blue/green (traffic shifting) only** | [3.3.4] |
| AppSpec required fields | ECS: **TaskDefinition, ContainerName, ContainerPort**. Lambda: **Name, Alias, CurrentVersion, TargetVersion** | [3.3.4] |
| CodeDeploy agent | Needed on **EC2 and on-premises** instances only | [3.3.4] |
| `appspec.yml` vs `buildspec.yml` | **appspec** is CodeDeploy; **buildspec** is CodeBuild | [3.3.4] |
| Deploy EC2 one instance at a time | **`CodeDeployDefault.OneAtATime`** | [3.3.4] |
| Manager approval before production | **CodePipeline manual approval action** | [3.3.5] |
| Pipeline artifacts | Stored in an **S3 artifact bucket** | [3.3.5] |
| Shift 10% of traffic to a new Lambda version | **One alias with weighted routing** (aliases can't point to other aliases) | [3.3.6] |
| Lambda or ECS traffic shifting options | **Canary** (10%, then the rest), **Linear** (10% every N minutes), **AllAtOnce** | [3.3.6] |
| Gradual Lambda deploy in SAM | **`AutoPublishAlias` + `DeploymentPreference`** (uses CodeDeploy) | [3.3.6] |
| Test a new API version on a percentage of traffic | **API Gateway stage canary** | [3.3.6] |
| Run tests before Lambda canary traffic | **`BeforeAllowTraffic`** hook | [3.3.6] |
| Beanstalk deployment policies | All at once: fastest, **downtime**. Rolling: reduced capacity. Rolling with additional batch: **full capacity**. **Immutable**: new Auto Scaling group, fastest rollback. Traffic splitting: canary percentage. Blue/green: new environment + **CNAME swap** | [3.3.6] |
| Beanstalk and RDS | **Create RDS outside the environment**; an RDS instance inside it is deleted with the environment | [3.3.6] |
| Roll back on an alarm | **CodeDeploy alarm-based rollback** or **CloudFormation rollback triggers** | [3.3.7] |
| What a CodeDeploy rollback is | A **new deployment with a new ID** that redeploys the last good revision | [3.3.7] |
| Instant Lambda rollback | **Point the alias back** to the previous version | [3.3.7] |
| ECS rollout keeps failing | **Deployment circuit breaker** with rollback | [3.3.7] |
| Deploy when a release tag is pushed | **CodePipeline V2 trigger on Git tags** | [3.3.8] |
| AI review before an approval | **Bedrock or Kiro step + manual approval** | [3.3.9] |

### Domain 3 numbers

| Item | Value | Skill |
|---|---|---|
| CodeBuild timeout | **60 minutes** default, up to **36 hours** | [3.1.1] |
| API Gateway stage cache TTL | **300 s** default, maximum **3,600 s** | [3.1.5] |

## Domain 4: Troubleshooting and Optimization (22%)

### Task 4.1: Assist with root-cause analysis

| Clue | Answer | Skill |
|---|---|---|
| API Gateway 502 | **Lambda proxy response in the wrong format** (needs `statusCode` and a string `body`) | [4.1.1] |
| API Gateway 504 | **Integration timeout** reached (29 s default on REST APIs): make the work asynchronous, stream, or optimize | [4.1.1] |
| ALB 503 / 504 / 502 | 503: **no registered targets**. 504: **target timeout**. 502: **bad response from the target** | [4.1.1] |
| Uneven load across targets | **Sticky sessions** or unbalanced Availability Zones | [4.1.1] |
| ECS `CannotPullContainerError` | **Task execution role**, ECR VPC endpoints, or a NAT gateway | [4.1.1] |
| Where is API latency spent? | **`Latency`** is end to end; **`IntegrationLatency`** is the backend | [4.1.2] |
| Who made this API call? | **CloudTrail** | [4.1.2] |
| IP traffic to and from ENIs | **VPC Flow Logs** | [4.1.2] |
| One-minute EC2 metrics | **Detailed monitoring** (basic is 5 minutes) | [4.1.2] |
| Query logs | **CloudWatch Logs Insights** | [4.1.3] |
| p99 latency from Lambda logs | **`stats pct(@duration, 99)`** | [4.1.3] |
| Follow one request across services | **Logs Insights + a correlation ID** | [4.1.3] |
| Stream logs to another destination | **Subscription filter** (Lambda, Firehose, or Kinesis) | [4.1.3] |
| Stack failed | Read **stack events** from the first failure | [4.1.4] |
| CodeDeploy `DownloadBundle` fails | The **instance profile can't read the S3 bucket** | [4.1.4] |
| Debug a build before pushing | **CodeBuild local agent** | [4.1.4] |
| AI-assisted root-cause analysis | **Kiro** or **CloudWatch investigations** | [4.1.5] |

### Task 4.2: Apply observability practices

| Clue | Answer | Skill |
|---|---|---|
| Logs that are easy to query and filter | **Structured JSON logging**, queried with Logs Insights | [4.2.1] |
| Lambda log format and level without code changes | Lambda **advanced logging controls** (JSON format, log level) | [4.2.1] |
| Logs, metrics, traces, and idempotency helpers for Lambda | **Powertools for AWS Lambda** | [4.2.1] |
| Publish your own metric | **`PutMetricData`** (one custom namespace per app) | [4.2.2] |
| Metric every 1 to 10 seconds | **High-resolution metric** (`StorageResolution=1`) | [4.2.2] |
| Business metrics from Lambda without extra API calls | **EMF** in log output | [4.2.2] |
| Metric from log text | **Metric filter** | [4.2.2] |
| Memory or disk space on EC2 | **CloudWatch agent** | [4.2.2] |
| Alarm only when 3 of the last 3 periods breach | **Evaluation periods = 3, datapoints to alarm = 3** | [4.2.3] |
| Alert near the Lambda concurrency quota | **CloudWatch alarm on the Service Quotas usage metric** | [4.2.3] |
| Notify on pipeline failure or deployment completion | **CodePipeline notification rule**, or an EventBridge rule on state changes to SNS | [4.2.3] |
| Find a slow downstream call | **X-Ray subsegments** (`namespace` is `aws` or `remote`) | [4.2.4] |
| Search, group, or filter traces by a value such as customer | **Annotations** (indexed) | [4.2.4] |
| Extra trace data that isn't searched | **Metadata** (not indexed) | [4.2.4] |
| Query traces by annotation | **`GetTraceSummaries`** with filter expressions | [4.2.4] |
| Control trace volume and cost | **Sampling rules** (default: first request each second + 5% of the rest) | [4.2.4] |
| X-Ray data path | App + SDK send segments over **UDP port 2000** to the daemon, which batches them to the X-Ray API | [4.2.4] |
| X-Ray on EC2 / ECS / Lambda / Beanstalk | EC2: **user data installs the daemon** + `AWSXRayDaemonWriteAccess`. ECS: daemon **sidecar**. Lambda: **active tracing**. Beanstalk: `.ebextensions` with `XRayEnabled: true` | [4.2.4] |
| X-Ray variables in Lambda | **`_X_AMZN_TRACE_ID`**, **`AWS_XRAY_CONTEXT_MISSING`**, `AWS_XRAY_DAEMON_ADDRESS` | [4.2.4] |
| No traces appear | **IAM permissions** missing; for Lambda, **active tracing** off | [4.2.4] |
| Health check and readiness | **ALB target group health check**, **ECS container `healthCheck`**, **Route 53 health checks**, EKS **readiness probe** | [4.2.5] |
| Restart a stuck container / stop traffic until ready | **Liveness** probe / **readiness** probe | [4.2.5] |
| Route by latency, weight, failover, or location | **Route 53** latency, weighted, failover (uses health checks), or geolocation routing | [4.2.5] |

### Task 4.3: Optimize applications

| Clue | Answer | Skill |
|---|---|---|
| Will it throttle? | **Concurrency = requests per second x duration in seconds**; above 1,000, request an increase | [4.3.1] |
| One function starves the others, or protect a downstream database | **Reserved concurrency** (or ESM maximum concurrency) | [4.3.1] |
| Remove cold starts | **Provisioned concurrency** | [4.3.1] |
| Java cold starts | **SnapStart** or provisioned concurrency | [4.3.1] |
| Cold start visible in logs | **`Init Duration`** in the `REPORT` line | [4.3.1] |
| Database connections exhausted by Lambda | **RDS Proxy** | [4.3.1] |
| Out of memory or slow CPU in Lambda | **Raise memory** (CPU scales with it) | [4.3.2] |
| Cost-optimal Lambda memory | **Lambda Power Tuning** | [4.3.2] |
| Messages processed twice | **Raise the visibility timeout** above processing time; make consumers idempotent | [4.3.3] |
| Processing takes longer than the visibility timeout | **`ChangeMessageVisibility`** | [4.3.3] |
| Fewer empty receives, lower cost | **Long polling** (`WaitTimeSeconds` up to 20 s) | [4.3.3] |
| Delay delivery | **Delay queue** or `DelaySeconds` | [4.3.3] |
| Payload larger than the SQS limit | **Store in S3, send a pointer** (Extended Client Library) | [4.3.3] |
| Ordered processing per entity, suppress duplicate sends | **SQS FIFO** (`MessageGroupId` + deduplication ID). Deduplication covers a 5-minute window; consumers still need idempotency | [4.3.3] |
| DLQ type | Must **match** the source queue (FIFO to FIFO) | [4.3.3] |
| Remove the queue vs remove its messages | **`DeleteQueue`** vs **`PurgeQueue`** | [4.3.3] |
| A subscriber needs only some messages | **SNS subscription filter policy** | [4.3.3] |
| Light header or JWT check at the edge, cheap | **CloudFront Functions** on viewer request | [4.3.4] |
| Network calls, request body, heavier edge logic | **Lambda@Edge** | [4.3.4] |
| Static S3 content + dynamic ALB content | **CloudFront with multiple origins and cache behaviors** | [4.3.4] |
| Fail over between origins | **CloudFront origin group** | [4.3.4] |
| Microsecond DynamoDB reads, hot items | **DAX** | [4.3.4] |
| Lambda CPU, memory, and cold starts per function | **CloudWatch Lambda Insights** | [4.3.5] |
| Hot DynamoDB keys | **CloudWatch Contributor Insights** | [4.3.5] |
| RDS per-process CPU and memory | **RDS Enhanced Monitoring** | [4.3.5] |
| Slow SQL queries | **RDS Performance Insights** | [4.3.5] |

### Domain 4 numbers

| Item | Value | Skill |
|---|---|---|
| SQS message size | **1 MiB** (raised from 256 KiB in 2025); larger payloads use the Extended Client Library with S3 (up to 2 GB) | [4.3.3] |
| SQS retention | **4 days** default, **1 minute to 14 days** | [4.3.3] |
| SQS visibility timeout | **30 s** default, **12 hours** maximum | [4.3.3] |
| SQS long polling / delay / batch | Up to **20 s** / up to **15 minutes** / **10 messages** | [4.3.3] |
| SQS FIFO throughput | **300 API calls/s** per action, **3,000 messages/s** with batching (higher in high-throughput mode); deduplication window **5 minutes** | [4.3.3] |
| SNS message size | **256 KiB** default; a topic's `MaximumMessageSize` can raise it to **1 MiB**; `PublishBatch` up to **10** messages | [4.3.3] |
| CloudWatch high-resolution metric | Down to **1 second** | [4.2.2] |
| EC2 monitoring | Basic **5 minutes**, detailed **1 minute** | [4.1.2] |

## Not on the DVA-C03 service list

> **Not on the DVA-C03 service list.** These services appear in older material and practice questions. DVA-C03 dropped AWS AppSync and Amazon Q Developer from the service list, and lists AWS Shield as out of scope. The others are absent from the in-scope list. Know them well enough to rule them out as distractors.
>
> | Clue | Answer | Related skill |
> |---|---|---|
> | Real-time GraphQL, offline sync, many users | **AWS AppSync** (Cognito Sync is legacy and per-user) | [1.1.2] |
> | AI code help, unit tests, code review and scans (DVA-C02 wording) | **Amazon Q Developer**, which replaced CodeWhisperer; CodeGuru left the exam scope. For DVA-C03 the named tool is **Kiro** | [1.1.9] |
> | Already uses Apache Kafka | **Amazon MSK** | [1.1.6] |
> | Find PII in S3 buckets | **Amazon Macie** | [2.2.6] |
> | Dedicated hardware security module you control | **AWS CloudHSM** | [2.2.4] |
> | DDoS protection at layers 3 and 4 | **AWS Shield** Standard or Advanced (out of scope; AWS WAF is in scope) | [1.1.4] |
> | Send application email | **Amazon SES** | [1.1.8] |
> | Pre-built serverless applications | **AWS Serverless Application Repository** | [3.3.3] |
> | Right-size Lambda or EC2 from usage history | **AWS Compute Optimizer** (Lambda Power Tuning is an open-source tool you run yourself) | [4.3.6] |
> | AI agent for incident root cause | **AWS DevOps Agent** | [4.1.5] |

## Sources

- Merged from: `archive/dva-c03-study-notes/05-cheat-sheet.md` and `archive/aws-dva-notes/Markdown Files/31. Exam Day Cheat Sheet.md` (plain text, not links). Exam technique lines moved to [exam strategy](exam-strategy.md).
- [Amazon SQS message quotas](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html): 1 MiB maximum, FIFO throughput, retention, visibility timeout.
- [Amazon API Gateway quotas for REST APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-execution-service-limits-table.html): 29-second default integration timeout, increase for Regional and private APIs, cache TTL, payload size.
- [Quotas for HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-quotas.html): fixed 30-second integration timeout.
- [Choosing workflow type in Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html): Standard exactly-once, Express asynchronous at-least-once, Express synchronous at-most-once.
- [Step Functions service quotas](https://docs.aws.amazon.com/step-functions/latest/dg/service-quotas.html): 256 KiB input and output, 1-year and 5-minute durations.
- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html) and [Configure Lambda function timeout](https://docs.aws.amazon.com/lambda/latest/dg/configuration-timeout.html): 30-minute timeout, 300 GB code storage, payload sizes, scaling rate.
- [Amazon SNS endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/sns.html): 256 KiB default message size, configurable to 1 MiB.
- [Amazon S3 objects overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingObjects.html): objects up to 50 TB.
- [DVA-C03 in-scope services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-in-scope-services.html) and [out-of-scope services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-out-of-scope-services.html).

[1.1.1]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.1-coupling-and-decoupling.md
[1.1.2]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.2-communication-patterns.md
[1.1.3]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.3-resilience-retries-and-backoff.md
[1.1.4]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.4-apis-transformation-and-validation.md
[1.1.5]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md
[1.1.6]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.6-streaming-data.md
[1.1.7]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.7-serialization-and-deserialization.md
[1.1.8]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.8-event-driven-applications.md
[1.1.9]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md
[1.1.10]: ../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.10-managed-ai-service-integration.md
[1.2.1]: ../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.1-lambda-vpc-access.md
[1.2.2]: ../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.2-lambda-configuration.md
[1.2.3]: ../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.3-lambda-invocation-lifecycle-and-failures.md
[1.2.4]: ../domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/1.2.4-lambda-service-integrations.md
[1.3.1]: ../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.1-data-lifecycle-management.md
[1.3.2]: ../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.2-choosing-data-stores.md
[1.3.3]: ../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.3-caching-strategies.md
[1.3.4]: ../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md
[2.1.1]: ../domain-2-security/task-2.1-authentication-and-authorization/2.1.1-identity-federation.md
[2.1.2]: ../domain-2-security/task-2.1-authentication-and-authorization/2.1.2-access-controls-tokens-and-mfa.md
[2.1.3]: ../domain-2-security/task-2.1-authentication-and-authorization/2.1.3-secure-programmatic-access.md
[2.1.4]: ../domain-2-security/task-2.1-authentication-and-authorization/2.1.4-time-limited-access.md
[2.1.5]: ../domain-2-security/task-2.1-authentication-and-authorization/2.1.5-authenticated-requests.md
[2.1.6]: ../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md
[2.1.7]: ../domain-2-security/task-2.1-authentication-and-authorization/2.1.7-assuming-iam-roles.md
[2.2.1]: ../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.1-encryption-at-rest-and-in-transit.md
[2.2.2]: ../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.2-certificate-management.md
[2.2.3]: ../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.3-client-side-vs-server-side-encryption.md
[2.2.4]: ../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.4-kms-key-management.md
[2.2.5]: ../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.5-development-certificates-and-ssh-keys.md
[2.2.6]: ../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.6-sensitive-data-detection-and-masking.md
[2.2.7]: ../domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/2.2.7-secrets-management.md
[2.3.1]: ../domain-2-security/task-2.3-secure-ai-services/2.3.1-ai-service-access-control.md
[2.3.2]: ../domain-2-security/task-2.3-secure-ai-services/2.3.2-ai-data-privacy-and-private-connectivity.md
[2.3.3]: ../domain-2-security/task-2.3-secure-ai-services/2.3.3-ai-input-and-output-filtering.md
[2.3.4]: ../domain-2-security/task-2.3-secure-ai-services/2.3.4-ai-agent-security.md
[2.3.5]: ../domain-2-security/task-2.3-secure-ai-services/2.3.5-ai-sensitive-data-in-logs.md
[3.1.1]: ../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.1-packaging-dependencies-and-configuration.md
[3.1.2]: ../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.2-container-images-and-ecr.md
[3.1.4]: ../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.4-environment-specific-configuration.md
[3.1.5]: ../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md
[3.2.1]: ../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.1-testing-with-sam-cdk-and-cloudformation.md
[3.2.2]: ../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.2-mock-apis.md
[3.2.3]: ../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.3-test-events-and-payloads.md
[3.2.4]: ../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.4-isolated-test-environments.md
[3.2.5]: ../domain-3-testing-and-deployment/task-3.2-test-applications/3.2.5-ai-assisted-testing.md
[3.3.1]: ../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.1-lambda-packaging.md
[3.3.2]: ../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.2-container-deployments.md
[3.3.3]: ../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.3-infrastructure-as-code-deployment.md
[3.3.4]: ../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.4-codedeploy.md
[3.3.5]: ../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.5-codepipeline-release-promotion.md
[3.3.6]: ../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.6-deployment-strategies.md
[3.3.7]: ../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.7-rollback.md
[3.3.8]: ../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.8-versions-and-releases.md
[3.3.9]: ../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.9-ai-assisted-deployment.md
[4.1.1]: ../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.1-debugging-code-and-integrations.md
[4.1.2]: ../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.2-metrics-logs-and-traces.md
[4.1.3]: ../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.3-querying-logs.md
[4.1.4]: ../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.4-diagnosing-deployment-failures.md
[4.1.5]: ../domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/4.1.5-ai-assisted-error-analysis.md
[4.2.1]: ../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.1-structured-logging.md
[4.2.2]: ../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.2-custom-metrics.md
[4.2.3]: ../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.3-alerts-and-notifications.md
[4.2.4]: ../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.4-distributed-tracing.md
[4.2.5]: ../domain-4-troubleshooting-and-optimization/task-4.2-observability/4.2.5-health-checks-and-probes.md
[4.3.1]: ../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.1-concurrency-controls.md
[4.3.2]: ../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.2-memory-and-compute-tuning.md
[4.3.3]: ../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.3-messaging-optimization.md
[4.3.4]: ../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.4-caching-for-performance.md
[4.3.5]: ../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.5-performance-bottleneck-detection.md
[4.3.6]: ../domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/4.3.6-ai-assisted-optimization.md
