# DVA-C03 Full Mock Exam 1 — Questions

65 questions. Multiple choice (pick ONE) unless a question explicitly says **Choose TWO**. No answers are shown in this file — see the matching `*-answer-key.md` file after you finish (or after each section if you're reviewing, not simulating).

---

## Q1

A payments service writes an order item to DynamoDB and later needs to update its `status`
field from `PENDING` to `SHIPPED`, but only if another process hasn't already moved it to
`CANCELLED` in the meantime. The current code does a plain `UpdateItem` call with no extra
parameters, and the team has seen orders incorrectly flip from `CANCELLED` back to `SHIPPED` under
race conditions. What should the developer add to the `UpdateItem` call to fix this?

A. A `ProjectionExpression` limiting the call to only the `status` attribute.

B. A `ConditionExpression` such as `status <> :cancelled` so the update is rejected if the item has already moved to CANCELLED.

C. A second `GetItem` call immediately before `UpdateItem`, performed in the same application thread.

D. Strongly consistent reads (`ConsistentRead: true`) on the `UpdateItem` call.

---

## Q2

An IAM user has a policy attached that grants `s3:*` on `arn:aws:s3:::reports-bucket/*`.
A separate policy, attached to the same user by a different team for a compliance requirement,
explicitly denies `s3:DeleteObject` on all resources. The user calls `DeleteObject` on an object in
`reports-bucket`. What happens, and why?

A. The call succeeds, because the Allow statement is more specific (bucket-scoped) than the Deny statement (account-wide).

B. The call is denied — an explicit Deny in any applicable policy always overrides any Allow, regardless of specificity or how many other policies grant access.

C. The call succeeds only if the Deny policy was attached after the Allow policy.

D. IAM evaluates policies in attachment order and only applies the last one attached.

---

## Q3

A team manages a serverless application with a `template.yaml` written in AWS SAM.
Before every deployment, they want to catch structural errors (e.g. a Lambda function referencing
an IAM role that doesn't exist in the template, invalid resource properties) without actually
deploying anything to AWS. What command accomplishes this as an automated pre-deploy check in
CI/CD?

A. `sam validate` (or `sam validate --lint` for deeper linting), run as a CI step before `sam deploy`, which parses and validates the template's structure and references without creating or changing any AWS resources.

B. `sam deploy --dry-run-only`, which does not exist as a flag but is assumed to work.

C. Manually read through the YAML file for syntax errors before every deployment.

D. `aws cloudformation delete-stack`, run first to confirm the stack doesn't already exist.

---

## Q4

A production Lambda function started throwing intermittent `ProvisionedThroughputExceededException`
errors from DynamoDB about an hour ago, across thousands of log lines spread over dozens of log
streams. The developer needs to find the exact table name and the time window the errors started,
without manually opening each log stream. What is the fastest way to do this?

A. Run a CloudWatch Logs Insights query against the function's log group filtering for `ProvisionedThroughputExceededException`, extracting the table name field and aggregating error counts by time bucket (e.g. `stats count() by bin(5m)`).

B. Download every log stream's raw text file and `grep` through them one at a time locally.

C. Increase the Lambda function's memory, which will make the error messages easier to read.

D. Disable logging temporarily so the error stops appearing.

---

## Q5

A clickstream ingestion pipeline uses a Kinesis Data Stream with 20 shards. Three
separate applications need to independently read every record from the stream at their own pace:
a real-time fraud-detection Lambda, a batch analytics job, and a new dashboard that was just added
and is causing the fraud-detection Lambda's `GetRecords` calls to occasionally hit
`ReadProvisionedThroughputExceeded`. What is the best fix that lets all three consumers read
independently without throttling each other?

A. Increase the number of shards until the throttling stops.

B. Switch all three consumers to enhanced fan-out, giving each consumer its own dedicated 2 MB/sec read throughput per shard instead of sharing the classic 2 MB/sec/shard pool.

C. Have the dashboard read from an SQS queue instead, fed by polling the stream on a cron schedule.

D. Combine all three consumers into a single Lambda function so there is only one reader.

---

## Q6

A SaaS vendor needs to let each of its customers' own AWS accounts assume a role in the
vendor's account to push data, without creating per-customer IAM users and without risking that one
customer could trick another customer's intermediary into assuming the vendor's role on their
behalf (the "confused deputy" problem). What should the trust policy include?

A. A condition requiring a unique `sts:ExternalId` value (assigned per customer) in the `AssumeRole` call, checked against the value the vendor recorded for that specific customer.

B. A condition that only checks the calling principal's AWS account ID, with no external ID.

C. Granting `sts:AssumeRole` with `Resource: *` so any account can assume the role.

D. Requiring MFA on the customer's IAM user calling `AssumeRole`, with no external ID.

---

## Q7

A Lambda function's dependencies (including a large machine-learning library) make
its deployment package exceed the 250 MB unzipped limit for .zip-based Lambda functions. The team
still wants to deploy on Lambda rather than move to a different compute service. What is the
correct fix?

A. Package and deploy the function as a container image (up to 10 GB) stored in Amazon ECR instead of a .zip archive — Lambda supports container image packaging specifically for cases exceeding the zip size limits.

B. Split the single function into 10 smaller functions that each import a tenth of the library.

C. Ask AWS Support to manually raise the 250 MB limit for this one function.

D. Store the library in S3 and have the function download it on every invocation before running.

---

## Q8

An API request traverses API Gateway → Lambda → RDS and takes 3.2 seconds end-to-end,
well above the SLA. AWS X-Ray is enabled with active tracing on the Lambda function. Looking at the
trace's service map and segment timeline, what is the correct way to identify exactly which part of
the call chain caused the slowdown?

A. Open the trace timeline and compare the duration of each segment/subsegment (Lambda initialization, handler execution, and the RDS call captured via the X-Ray SDK's database client instrumentation) — the subsegment with the largest duration is where the time is actually being spent.

B. Assume the slowdown is always in API Gateway, since it's the entry point.

C. Only check CloudWatch Logs timestamps, since X-Ray cannot show per-call timing within a single invocation.

D. Re-run the request from a different Region and see if it's faster there.

---

## Q9

An API Gateway REST API uses Lambda proxy integration. A developer's Python handler
returns:
```python
return {
    "statusCode": 200,
    "headers": {"Content-Type": "application/json"},
    "body": {"orderId": "abc123", "total": 42.50}
}
```
Every client call fails with a generic `Internal Server Error`, even though CloudWatch Logs show
the handler ran to completion with no exceptions. What is wrong?

A. The Lambda execution role is missing `execute-api:Invoke` permission.

B. `body` must be a JSON-encoded string (e.g. `json.dumps({...})`), not a raw Python dict — proxy integration passes `body` through to the client byte-for-byte.

C. `statusCode` must be passed as the string `"200"` rather than the integer `200`.

D. The `headers` dict must include `Access-Control-Allow-Origin` even for non-browser clients, or API Gateway rejects the response.

---

## Q10

A mobile app authenticates end users with email/password, then needs those authenticated
users to upload photos directly to S3 using temporary AWS credentials scoped to only their own
"folder" in the bucket, without routing the upload through a backend server. Which Cognito
construct issues these temporary AWS credentials after sign-in?

A. A Cognito user pool alone — user pools can directly issue IAM credentials for AWS API calls.

B. A Cognito identity pool, federated with the Cognito user pool as an identity provider — the identity pool exchanges the user pool's ID token for temporary IAM credentials via STS, scoped by an IAM role mapped to the authenticated identity.

C. An IAM user created on the fly for each app user at sign-up.

D. API Gateway Lambda authorizer, which issues AWS credentials directly to the client.

---

## Q11

A team deploys a new Lambda function version behind an alias used by API Gateway. They
want traffic to shift gradually — 10% to the new version, then the rest after 5 minutes if no
CloudWatch alarms fire — and to automatically roll back to the previous version if an alarm does
fire during the shift. What AWS feature implements this?

A. AWS CodeDeploy for Lambda, using a canary deployment configuration (e.g. `Canary10Percent5Minutes`) with a CloudWatch alarm configured on the deployment group to trigger automatic rollback.

B. Manually updating the Lambda alias's weighted routing configuration every few minutes by hand and watching CloudWatch dashboards in a separate browser tab.

C. API Gateway stage variables alone, with no CodeDeploy involvement.

D. Lambda reserved concurrency, set to 10% of the function's normal value during deployment.

---

## Q12

A critical S3 bucket policy was unexpectedly modified overnight, removing a restriction
that previously blocked public access, and the security team needs to know exactly which IAM
principal made the change, from what source IP, and at what time, to investigate whether it was
authorized. Which service provides this information?

A. AWS CloudTrail — search the event history (or a configured trail's delivered logs) for the `PutBucketPolicy` management event on that bucket, which records the calling principal's ARN, source IP, user agent, and exact timestamp.

B. Amazon CloudWatch Logs alone, since every AWS API call is written there automatically with no CloudTrail involved.

C. AWS Config, since it can tell you who made a change, not just what changed.

D. S3 Server Access Logs, which record every IAM principal's console actions by default.

---

## Q13

A Lambda function needs to query a private RDS instance that lives in a VPC with no
internet gateway. After deploying the function with VPC configuration attached, invocations now
time out when the function also tries to call `s3.get_object()` to fetch a config file from an S3
bucket, something that worked before the function was attached to the VPC. What is the most likely
cause and fix?

A. Lambda functions cannot access S3 at all once placed in a VPC; move the config into environment variables instead.

B. The subnets have no route to the internet or an S3 gateway endpoint, so outbound HTTPS calls to the public S3 service endpoint have nowhere to go; add an S3 gateway VPC endpoint (or a NAT gateway) to the route table used by the Lambda's subnets.

C. The Lambda execution role is missing `s3:GetObject`; re-attach VPC config has nothing to do with it.

D. Increase the Lambda function's timeout setting so the S3 call has more time to complete.

---

## Q14

A company with 40 AWS accounts organized under AWS Organizations wants developers to be
able to sign in once and switch between accounts they're authorized for, using a consistent set of
permissions (e.g. "Developer", "ReadOnly") defined centrally rather than recreating IAM roles by
hand in every account. What AWS feature is designed for this?

A. AWS IAM Identity Center (successor to AWS SSO) with permission sets assigned to users/groups across accounts — permission sets define the permissions once and are provisioned as IAM roles into each assigned account automatically.

B. Create an identical IAM user in all 40 accounts with the same password.

C. A single IAM role in the Organizations management account that every developer assumes directly, regardless of which member account they need to work in.

D. Amazon Cognito user pools, one per AWS account.

---

## Q15

Five Lambda functions across a serverless application all need the same third-party
HTTP client library and a shared internal logging utility. The developer wants to avoid packaging
the same ~15 MB of dependencies into all five deployment packages and wants a single place to patch
the library version. What AWS feature solves this?

A. AWS Lambda layers — publish the shared dependencies as a layer version and attach it to all five functions; each function's own package stays small.

B. Lambda function aliases pointing at a shared version.

C. A single Lambda function with five different handler entry points configured via environment variables.

D. Amazon EFS mounted read-write by each function, with the dependencies copied in at every cold start.

---

## Q16

A CI/CD pipeline automatically deploys every merge to `main` straight to production with
no human review, and a recent bad deploy caused a customer-facing outage. The team wants to keep
full automation for build/test but require a human to explicitly approve the specific release
before it reaches the production stage, without reverting to manual deployments entirely. What
should be added to the pipeline?

A. A manual approval action in AWS CodePipeline, placed as its own stage immediately before the production deployment stage, which pauses the pipeline until an authorized reviewer approves (or rejects) via the console, CLI, or SNS-notified link.

B. Remove the production stage from the pipeline entirely and deploy production manually outside of CodePipeline forever.

C. Add a `sleep 3600` step to the buildspec so there is a delay before deployment.

D. Require every commit to main to be signed with GPG.

---

## Q17

A Lambda function behind API Gateway has high p99 latency specifically on the first
request after a period of inactivity (cold start), while warm invocations are consistently fast.
The function is memory-constrained at 128 MB and does meaningful CPU-bound initialization work
(parsing a large configuration object) during the INIT phase. The team needs to both reduce cold
start impact for latency-sensitive traffic and speed up the init phase itself. What two changes
address this?

A. Enable Provisioned Concurrency to keep a pool of initialized execution environments ready (eliminating cold starts for that pool), and separately increase the function's memory allocation, which proportionally increases allocated CPU and speeds up the CPU-bound initialization work itself.

B. Decrease the function's timeout setting, which directly reduces cold start duration.

C. Switch the function's trigger from API Gateway to an SQS queue, since SQS-triggered functions never experience cold starts.

D. Add more reserved concurrency, which makes individual cold starts faster.

---

## Q18

A company must ensure that **no account** in its AWS Organization can ever launch
resources outside two approved Regions, even if an account administrator with full IAM
"AdministratorAccess" tries to do so. What is the correct control, and where is it enforced?

A. A Service Control Policy (SCP) attached at the Organization or OU level, denying actions when `aws:RequestedRegion` is not one of the approved Regions — SCPs set the maximum permissions boundary for every principal in the account, which even a full-admin IAM policy cannot override.

B. An IAM policy with the same region condition, attached only in the accounts that need it.

C. A CloudWatch alarm that notifies security if a non-approved-Region resource is launched.

D. Removing the `AdministratorAccess` managed policy from all IAM roles.

---

## Q19

A Lambda function processes orders and is invoked by API Gateway (synchronous). During
a flash sale, the account's overall concurrency limit is being exhausted by a completely unrelated,
lower-priority batch-processing function, causing the order function to be throttled with
`TooManyRequestsException`. What is the correct fix to protect the order function specifically?

A. Set **reserved concurrency** on the order function so it always has a guaranteed, ring-fenced pool of concurrency that other functions in the account cannot consume.

B. Increase the batch function's memory allocation so it finishes faster and frees concurrency sooner.

C. Set the order function's timeout to the maximum (15 minutes) so it never gets throttled.

D. Delete the batch function's event source mapping and recreate it after the sale.

---

## Q20

An ECR repository accumulates thousands of old, untagged container images from CI
builds, most of which are never deployed and are only superseded by newer ones in the same
repository. The team wants these cleaned up automatically rather than manually deleting images,
while keeping the last 10 tagged `release-*` images indefinitely. What should they configure?

A. An ECR lifecycle policy with rules that expire untagged images after a set number of days, and a separate rule that keeps only the most recent N images matching the `release-*` tag prefix.

B. Set the repository to private, which automatically purges old images after 30 days.

C. Manually run `aws ecr batch-delete-image` once a week via a person logging in to check.

D. Disable image tagging entirely so there's nothing to clean up.

---

## Q21

A payments service's error rate alarm fires frequently during normal, expected daily
traffic spikes (e.g. lunchtime), creating alert fatigue, because its static threshold doesn't
account for the workload's predictable daily/weekly seasonality. The team wants an alarm that
adapts its expected range to historical patterns rather than using one fixed number. What CloudWatch
feature should they use?

A. CloudWatch anomaly detection — it models a metric's expected value as a band that adapts to historical trends and seasonality (e.g. daily/weekly cycles), and an anomaly detection alarm fires only when the metric falls **outside** that dynamically computed band, rather than crossing one static number.

B. A static threshold alarm set to the highest value ever observed, so it never fires.

C. Deleting the alarm entirely, since false positives are unavoidable.

D. A CloudWatch dashboard widget, which can itself trigger a page to on-call engineers.

---

## Q22

An Application Load Balancer needs a TLS certificate for `api.example.com` so the
application can terminate HTTPS at the load balancer. The certificate must auto-renew without the
team manually re-uploading a new certificate every year. What is the correct approach?

A. Request a public certificate through AWS Certificate Manager (ACM), validate domain ownership via DNS validation (a CNAME record), and attach the ACM certificate ARN to the ALB's HTTPS listener — ACM manages renewal automatically as long as the validation record remains in place.

B. Generate a self-signed certificate and upload it to IAM's server certificate store, renewing it manually each year.

C. Use AWS KMS to issue a TLS certificate directly for the ALB listener.

D. Store the certificate's private key in Secrets Manager and have the ALB fetch it on every request.

---

## Q23

An image-processing Lambda is triggered by an SQS standard queue (event source
mapping, batch size 10). The team notices the same image is sometimes processed twice. The function
itself is idempotent-unsafe (processing twice corrupts output), and investigation shows the
function occasionally takes longer than the queue's visibility timeout to finish a batch. What
should be changed?

A. Increase the queue's visibility timeout to comfortably exceed the function's maximum expected processing time for a batch (at least 6x the function timeout, per AWS guidance).

B. Switch the queue to SNS so messages are pushed instead of polled.

C. Reduce the Lambda batch size to 1 so visibility timeout no longer matters.

D. Delete messages from the queue manually inside the handler before processing them.

---

## Q24

A media application lets users upload video files that range from a few MB to over
10 GB directly to S3 from a browser using presigned URLs. Uploads larger than ~100 MB fail or are
unreliable on flaky mobile connections. What change improves reliability for large uploads?

A. Use the S3 Multipart Upload API to split large files into parts uploaded independently (and resumably) in parallel, rather than a single `PutObject` call.

B. Switch the bucket's storage class to S3 Glacier Instant Retrieval before upload.

C. Increase the presigned URL's expiration time to 7 days.

D. Enable S3 Transfer Acceleration only, with no change to the upload API calls.

---

## Q25

An application in Account A needs to read objects from an S3 bucket in Account B that
are encrypted with a KMS key owned by Account B. Account A's Lambda execution role already has
`s3:GetObject` granted via the bucket policy. The `GetObject` calls still fail with `AccessDenied`
related to KMS. What else is required?

A. The KMS key policy in Account B must also grant Account A's Lambda execution role (or role ARN) `kms:Decrypt` permission — S3 bucket permissions and the KMS key's own resource policy/grants are evaluated separately, and both must allow the action.

B. Nothing else — S3 bucket policies automatically imply permission to use whatever KMS key encrypted the objects.

C. Account A must create its own copy of the same KMS key.

D. The objects must be re-uploaded unencrypted before Account A can read them.

---

## Q26

A containerized service runs on Amazon ECS with the Fargate launch type behind an
Application Load Balancer. The team wants a blue/green deployment where a new task set is fully
provisioned and validated before any production traffic shifts to it, with the ability to roll back
instantly to the old task set if post-deployment validation fails. What should orchestrate this?

A. AWS CodeDeploy configured for an ECS blue/green deployment, using two ALB target groups (one for the current/blue task set, one for the new/green task set) — CodeDeploy provisions the green set, reroutes the listener once healthy, and can roll back by rerouting back to blue.

B. A manual `aws ecs update-service --force-new-deployment`, which performs an in-place rolling update with no separate green environment.

C. Deleting the ECS service and recreating it from scratch for every release.

D. Switching the launch type from Fargate to EC2 to enable blue/green support, since Fargate doesn't support it.

---

## Q27

A team wants to emit a custom business metric (`OrdersProcessed`, dimensioned by
`region`) from inside a Lambda function's structured JSON logs, without making an extra
`cloudwatch:PutMetricData` API call on every invocation (to avoid added latency and API cost), while
still having the metric show up in CloudWatch Metrics for dashboards and alarms. What should they
use?

A. Embedded Metric Format (EMF) — write a specially structured JSON log line (including a `_aws` metadata block defining metric names/namespaces/dimensions) using the CloudWatch Logs/Lambda Powertools EMF helper; CloudWatch Logs automatically extracts and publishes it as a real CloudWatch metric with no extra API call from the function.

B. Call `cloudwatch:PutMetricData` synchronously inside the handler for every invocation, awaiting the response before returning.

C. Write the metric value to a DynamoDB table and build a separate ETL job to copy it into CloudWatch nightly.

D. Store the metric in an environment variable that updates on every invocation.

---

## Q28

A table `Orders` has partition key `orderId`. A new requirement appears: look up all
orders for a given `customerId`, sorted by order date, without scanning the whole table. The table
is already live in production with existing items and access patterns that must keep working.
What's the correct way to add this access pattern?

A. Add a Global Secondary Index (GSI) with partition key `customerId` and sort key `orderDate` — it can be created on an existing table without disrupting the existing key schema or current access patterns.

B. Change the table's partition key from `orderId` to `customerId`.

C. Add a Local Secondary Index (LSI) with partition key `customerId`.

D. Run a `Scan` with a `FilterExpression` on `customerId` for this access pattern.

---

## Q29

An RDS database's master password needs to rotate automatically every 30 days without
any application downtime or manual password updates, and the connecting Lambda function should
always fetch the current password at runtime. Separately, a feature-flag config value (not
sensitive) needs to be read by the same function. Which combination is correct?

A. AWS Secrets Manager for the RDS credential, using its built-in automatic rotation (with an AWS-provided or custom rotation Lambda) so the function always calls `GetSecretValue` for the current password; AWS Systems Manager Parameter Store (Standard, plaintext) for the non-sensitive feature flag.

B. Systems Manager Parameter Store `SecureString` for the RDS password, since it is cheaper, and Secrets Manager for the feature flag, since it is more fully featured.

C. Hardcode the RDS password as a Lambda environment variable and update it manually every 30 days.

D. Store both values in the same DynamoDB item with no encryption, since Lambda already has an execution role.

---

## Q30

Before applying a CloudFormation stack update that modifies a production RDS instance's
engine version, the team wants to see exactly which resources will be added, modified (and whether
modification is in-place or requires replacement), or deleted — without actually applying the
update yet. What should they use?

A. A CloudFormation change set — create one from the updated template, review the proposed resource-level changes (including replacement vs. in-place indicators), then execute it only if the preview looks correct.

B. `aws cloudformation delete-stack` followed immediately by `create-stack` with the new template.

C. Directly edit the resource in the RDS console, bypassing CloudFormation, then run `update-stack` afterward to "sync" the template.

D. Deploy the change directly to production first, then check the stack's event history afterward to see what happened.

---

## Q31

A compliance requirement states that every EBS volume in the account must be encrypted
at rest, but the team has discovered (after the fact) several unencrypted volumes that were created
by mistake weeks ago. They want continuous, automatic detection going forward — flagging any new
non-compliant volume the moment it's created — rather than periodic manual audits. What should they
configure?

A. An AWS Config managed rule (e.g. `encrypted-volumes`) with the recorder enabled for EBS volumes — Config continuously evaluates resource configuration against the rule and flags any volume (new or existing) that drifts into non-compliance, close to real-time.

B. A CloudFormation stack policy that merely documents the encryption requirement in a README.

C. A one-time AWS CLI script run manually once a quarter to list unencrypted volumes.

D. S3 Object Lambda, since it can intercept and modify EBS volume configurations.

---

## Q32

Choose **TWO** correct statements about building a highly available, read-scaled
relational database for an application with a global user base that also needs disaster recovery
across AWS Regions, using Amazon Aurora.

A. Aurora Replicas within the same Region can serve read traffic and provide fast failover for the writer in case of failure.

B. Aurora Global Database replicates a primary cluster's data to secondary clusters in other Regions with typically sub-second replication lag, enabling fast regional disaster recovery.

C. Aurora requires manually configured binlog-based replication to create cross-Region read replicas.

D. Aurora Replicas can accept both read and write traffic simultaneously for true multi-master writes by default.

E. Cross-Region read replicas are only possible for Aurora PostgreSQL, never Aurora MySQL.

---

## Q33

A company suspects an S3 bucket used for customer document uploads may contain
Social Security Numbers and credit card numbers that were never supposed to land there. They want
an automated, recurring scan across all buckets in the account to detect this kind of sensitive
data without writing custom detection logic. What should they use?

A. Amazon Macie — it uses machine learning and pattern matching to automatically discover and classify sensitive data (PII, financial data, credentials) in S3 and can run scheduled, recurring discovery jobs.

B. Amazon GuardDuty, which scans S3 object contents for PII by default.

C. AWS Config, configured with a rule that reads the content of every S3 object.

D. AWS Trusted Advisor's security category.

---

## Q34

An API Gateway REST API has three stages: `dev`, `test`, and `prod`. Each stage must
invoke a different Lambda function alias (`dev-alias`, `test-alias`, `prod-alias`) without
maintaining three separate API definitions or three separate integration configurations. What
feature accomplishes this cleanly?

A. Stage variables — reference `${stageVariables.lambdaAlias}` in the integration's Lambda function ARN so each stage's own stage variable value resolves to the right alias at invoke time.

B. Create three entirely separate REST APIs, one per environment.

C. Hardcode three different Lambda ARNs in the integration and use a resource policy to pick one based on the caller's IP.

D. Use three different custom domain names that all point at the same single stage.

---

## Q35

An application currently has a feature flag (`enableNewCheckout: true/false`) baked
into its deployment package as a config file, requiring a full redeploy to flip it. The team wants
to toggle the flag instantly in production, target it to a percentage of users for gradual rollout,
and automatically roll back the flag change if an associated CloudWatch alarm fires — all without
redeploying code. What service fits this?

A. AWS AppConfig — define the flag as an externalized configuration profile, deploy configuration changes independently of code deployments, with built-in deployment strategies (gradual percentage rollout) and automatic rollback tied to a CloudWatch alarm.

B. Bake a new config file into the deployment package and redeploy every time the flag needs to change.

C. Store the flag in the Lambda function's code as a hardcoded boolean and redeploy to change it.

D. Use S3 Versioning alone, with the application polling a bucket object every request with no caching.

---

## Q36

A recurring incident requires restarting a stuck application service on a fleet of EC2
instances via SSH, which an on-call engineer currently does manually by logging into each instance.
The team wants this turned into a safe, auditable, one-click (or fully automatic, triggered by a
CloudWatch alarm) remediation that doesn't require SSH key distribution or opening inbound SSH
access. What should they use?

A. AWS Systems Manager Automation, running a runbook (Automation document) that uses SSM Run Command / Agent (already on the instances, reachable without inbound SSH) to restart the service, triggerable manually or automatically from a CloudWatch alarm action.

B. Distribute the same SSH private key to every engineer and document the restart steps in a wiki page.

C. Open inbound SSH (port 22) from 0.0.0.0/0 to make it faster for any engineer to log in during an incident.

D. Have CloudWatch directly SSH into the instance and run the restart command itself.

---

## Q37

Under the AWS Shared Responsibility Model, which of the following is the **customer's**
responsibility when running a web application on AWS Lambda?

A. Patching the underlying operating system and the Lambda execution runtime environment's host infrastructure.

B. Writing IAM policies that follow least privilege for the function's execution role, and configuring encryption for any data the function reads or writes.

C. Physical security of the data center housing the Lambda compute fleet.

D. Patching the hypervisor layer that AWS uses to isolate Lambda execution environments.

---

## Q38

An order-processing system publishes an `OrderPlaced` event to an EventBridge custom
event bus. Three independent consumers exist: a shipping service that only cares about orders with
`region: "US"`, an email service that cares about all orders, and a fraud-review service that
only cares about orders over `$1000`. How should this be implemented so each consumer only receives
the events relevant to it, with no custom filtering logic inside the consumers themselves?

A. Create one EventBridge rule per consumer, each with an event pattern matching the attributes that consumer cares about (e.g. `{"detail": {"region": ["US"]}}`), targeting only that consumer.

B. Publish three separate events (one tailored per consumer) from the order service every time an order is placed.

C. Use a single rule with no event pattern that targets all three consumers, and have each consumer filter in its own code.

D. Use SQS message attributes for filtering instead of EventBridge, since EventBridge does not support content-based filtering.

---

## Q39

A developer wants to test a Lambda function's event-handling logic against a sample
S3 `ObjectCreated` event, entirely on their local machine, without deploying anything to AWS or
paying for any Lambda invocations, and also wants to simulate invoking it behind API Gateway
locally before pushing code. Which SAM CLI commands enable this?

A. `sam local invoke` with a local JSON test event file to directly invoke the function handler locally in a Docker-simulated Lambda runtime, and `sam local start-api` to emulate API Gateway + Lambda proxy integration locally for interactive HTTP testing.

B. `sam deploy --stage local`, which creates a special "local" stage inside the real AWS account.

C. `aws lambda invoke` against a function that doesn't exist yet, since invoking a nonexistent ARN automatically creates it in a sandbox.

D. Manually copy the handler code into a Jupyter notebook and call the function directly, bypassing the Lambda runtime entirely.

---

## Q40

A DynamoDB table provisioned with fixed read/write capacity is throttling
(`ProvisionedThroughputExceededException`) during unpredictable traffic spikes that vary widely day
to day, and the team doesn't want to constantly hand-tune capacity numbers. What is the most
appropriate fix?

A. Switch the table's billing mode to On-Demand (pay-per-request), which automatically scales to accommodate traffic without any capacity planning, or alternatively configure DynamoDB Auto Scaling on the provisioned table to adjust capacity within defined bounds based on utilization.

B. Add a Global Secondary Index, since GSIs automatically absorb all throttling from the base table.

C. Enable DynamoDB Streams, which increases the base table's throughput ceiling automatically.

D. Switch the table to use Strongly Consistent reads, which consume less capacity than eventually consistent reads.

---

## Q41

A public-facing API Gateway endpoint is being hit by a scripted client sending an
abnormally high volume of requests from a small set of IP addresses, degrading service for
legitimate users, but it isn't large enough to be a volumetric DDoS attack. What should be
configured to automatically block this pattern at the edge, before it reaches the API?

A. An AWS WAF web ACL with a rate-based rule, associated with the API Gateway stage, that automatically blocks IP addresses exceeding a configured request-rate threshold within a rolling time window.

B. AWS Shield Advanced alone, with no WAF rule.

C. Increase the API Gateway account-level throttling limit so more requests can be processed.

D. Switch the API from REST to HTTP API type.

---

## Q42

A mobile app needs a GraphQL API backed directly by DynamoDB for simple CRUD
operations (get a user profile, list a user's posts) with minimal backend code to maintain. Which
approach avoids writing and deploying a separate Lambda function for every single resolver?

A. AWS AppSync with a DynamoDB data source and a VTL (or JavaScript) resolver mapping template that talks to DynamoDB directly, with no Lambda in the middle for simple operations.

B. API Gateway with a Lambda proxy integration per GraphQL field.

C. AppSync configured to invoke one giant Lambda function per query, which then calls DynamoDB.

D. Amazon Cognito with a custom authentication Lambda trigger that also queries DynamoDB.

---

## Q43

A `buildspec.yml` for AWS CodeBuild needs to: install exact dependency versions from a
lockfile, run unit tests and fail the build if they don't pass, then zip the Lambda handler code
and dependencies into a deployable artifact for the next pipeline stage. Which `buildspec.yml`
phases map to these three responsibilities, in order?

A. `install` (install dependencies from the lockfile), `build` (run unit tests — a non-zero exit fails the build automatically), `post_build` (package the artifact, e.g. `zip`), with the resulting file listed under `artifacts`.

B. `pre_build`, `pre_build`, `pre_build` — everything happens in a single phase for simplicity.

C. `artifacts`, `install`, `build` — artifacts must be declared first before any other phase runs.

D. There is no concept of ordered phases in CodeBuild; all commands in the file run in parallel.

---

## Q44

A Lambda function's average duration is 2800 ms at 128 MB memory, costing a certain
amount per invocation. The team suspects that increasing memory (and therefore proportionally
allocated CPU) could make the function finish so much faster that total cost per invocation
actually goes **down**, despite the higher per-GB-second price, because duration drops more than
proportionally. What is the correct way to find the actual optimal memory setting rather than
guessing?

A. Benchmark the function at multiple memory settings (e.g. using a tool like AWS Lambda Power Tuning, which runs the function at several memory values and plots duration vs. cost) and pick the memory value that minimizes actual billed cost (or best balances cost and latency) for this specific workload.

B. Always set every Lambda function to the maximum available memory (10,240 MB), since more memory is always cheaper.

C. Reduce the memory setting as low as possible, since memory is the dominant cost driver regardless of duration.

D. Assume CPU-bound functions and I/O-bound functions behave identically with respect to memory/duration tradeoffs.

---

## Q45

Choose **TWO** correct statements about layering AWS Shield Advanced with AWS WAF to
protect a public API Gateway + Lambda application against DDoS attacks, where the team also wants
cost protection against charges incurred from a DDoS-driven usage spike.

A. Shield Advanced provides enhanced protection against larger-scale network and transport layer (Layer 3/4) DDoS events and includes DDoS cost protection credits for scaling charges incurred during a declared attack.

B. WAF web ACLs provide application-layer (Layer 7) protections — rate-based rules, managed rule groups (e.g. SQL injection, known bad inputs) — that Shield Advanced does not provide on its own.

C. Shield Standard and Shield Advanced provide identical protections and pricing; there's no reason to pay for Advanced.

D. Enabling Shield Advanced automatically disables the need for any WAF rules, since Shield covers every OSI layer.

E. Shield Advanced can only be used with EC2, never with API Gateway or CloudFront.

---

## Q46

A Step Functions state machine orchestrates an order workflow: validate payment, reserve
inventory, ship order. The `ReserveInventory` Lambda task occasionally fails due to a transient
downstream throttle, and when it fails permanently the workflow must transition to a
`NotifyCustomerOfFailure` state instead of simply erroring out the whole execution. Which two state
machine fields implement "retry a few times, then gracefully handle the failure"?

A. `Retry` (to retry the task a bounded number of times with backoff) and `Catch` (to route to a fallback state after retries are exhausted).

B. `Parallel` and `Map`.

C. `Choice` and `Wait`.

D. `TimeoutSeconds` alone, with no other fields.

---

## Q47

Choose **TWO** correct statements about using Amazon ElastiCache to hold web session
data for a fleet of stateless EC2 instances behind a load balancer (sessions must survive across
requests landing on different instances, and some session keys need atomic increment operations).

A. ElastiCache for Redis supports atomic operations like `INCR` and richer data structures (lists, hashes, sorted sets), making it a good fit when the app needs more than simple key/value get/set.

B. Because ElastiCache sits outside any individual EC2 instance, any instance can read a session written by another instance — this is what makes it suitable for sharing session state across a stateless fleet.

C. ElastiCache for Memcached supports built-in multi-AZ automatic failover with data replication, identical to Redis.

D. Sessions stored only in each EC2 instance's local memory would work equally well for a stateless, load-balanced fleet.

E. ElastiCache cannot be used for session storage; only DynamoDB supports this pattern.

---

## Q48

Amazon GuardDuty raises a finding: `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`,
indicating an EC2 instance's credentials were used to make API calls from an unusual geographic
location shortly after also being used normally from inside the VPC. What is the most appropriate
immediate response, consistent with Task 4.1's root-cause workflow?

A. Ignore it — GuardDuty findings are frequently false positives and don't require investigation.

B. Immediately rotate/invalidate the instance's credentials (e.g. by replacing the instance profile role's trust or terminating/replacing the instance), then use CloudTrail to review exactly which API calls that role made, from where, and over what time window, to scope the impact.

C. Delete the GuardDuty detector so the finding stops appearing in the console.

D. Wait for a scheduled quarterly security review to look into it.

---

## Q49

A team wants every pull request opened against their main branch to automatically
trigger a build-and-test run in CodeBuild, with the result (pass/fail) reported back as a status
check on the PR, before code review is considered complete — without merging the code first. Which
setup achieves this?

A. Configure the source repository's pull-request webhook/events to trigger CodeBuild (directly, or via a CodePipeline source stage with webhook-based triggers) to run on PR creation/update, reporting build status back to the PR.

B. Only run CodeBuild after merge, on the main branch, and retroactively revert the merge if the build fails.

C. Have developers manually run the test suite locally and paste the output into the PR description.

D. Configure CodeBuild to run once per day on a fixed schedule, regardless of PR activity.

---

## Q50

API Gateway starts returning `503 Service Unavailable` to a subset of clients during a
traffic spike, while CloudWatch shows the Lambda function's `Throttles` metric spiking at the same
time and the function's concurrent executions metric flat-lining at its reserved concurrency limit.
What is the correct interpretation and fix?

A. The Lambda function's reserved/account concurrency is being exhausted by the spike, so new invocation attempts are throttled (`TooManyRequestsException`), which API Gateway surfaces to clients as a 503 — raise the function's reserved concurrency (or account-level limit) or add a buffering layer (e.g. SQS) in front of it to smooth the burst.

B. The issue is unrelated to Lambda; API Gateway custom domain names cause 503s under any load regardless of backend health.

C. The fix is to disable X-Ray tracing, which is known to cause 503 errors under load.

D. The fix is to reduce the Lambda function's timeout to 1 second so it fails faster and frees capacity sooner.

---

## Q51

A company ingests IoT sensor data and wants every record permanently and durably
delivered into S3 as compressed Parquet files for later Athena queries, with automatic batching
and no custom consumer code to write, poll, or operate. Which service is the best fit for the
delivery layer, as opposed to raw Kinesis Data Streams?

A. Amazon Kinesis Data Firehose — a fully managed delivery service that batches, optionally transforms/converts format, and writes records to a destination like S3 automatically, with no consumer application to run.

B. Amazon Kinesis Data Streams alone, with a custom Lambda consumer that buffers records in memory and writes to S3 every 5 minutes.

C. Amazon SQS, polled by an EC2 fleet that batches messages into Parquet files.

D. AWS Step Functions, with a state machine invoked per record to write it to S3.

---

## Q52

An organization needs an immutable, tamper-evident audit trail of every management API
call made across all 15 of its AWS accounts and all Regions, retained centrally for compliance,
with log file integrity validation to prove logs haven't been altered after the fact. What should
be configured?

A. A single multi-Region CloudTrail organization trail (created from the Organizations management account) delivering to a centralized, access-restricted S3 bucket, with log file validation enabled to generate digitally signed digest files.

B. A separate, independently configured CloudTrail trail in each of the 15 accounts, each writing to its own local bucket with no centralization.

C. CloudWatch Logs alone, with no CloudTrail, since CloudWatch already retains all API call history by default.

D. VPC Flow Logs, since they capture all account activity including console and API actions.

---

## Q53

A CloudFormation stack update partially fails midway through (e.g. one resource's
update throws an error after three other resources were already successfully updated). By default,
what does CloudFormation do, and why does this matter for Task 3.3's "configure rollback"
requirement?

A. CloudFormation automatically rolls the entire stack back to its last known-good state, undoing the already-applied changes, so the stack never sits in a half-updated, inconsistent state — this default behavior is why rollback doesn't need to be separately 'configured' for basic stack update failures.

B. CloudFormation leaves the three successfully updated resources in their new state and simply stops, leaving the stack permanently inconsistent until manually fixed.

C. CloudFormation deletes the entire stack and all its resources immediately on any update failure.

D. CloudFormation retries the failed resource indefinitely until it succeeds, blocking forever.

---

## Q54

A monthly AWS bill review shows a large, steadily growing RDS instance that Cost
Explorer shows running at under 5% average CPU utilization for the last 90 days, alongside several
unattached EBS volumes left over from terminated instances. The team wants an automated, ongoing
set of recommendations flagging this kind of waste across the whole account, rather than manually
reviewing Cost Explorer every month. What should they use?

A. AWS Trusted Advisor's cost optimization checks (e.g. low-utilization EC2/RDS instances, idle load balancers, unattached EBS volumes) — these run continuously and surface actionable, recurring recommendations without a manual monthly review.

B. AWS Budgets alerts alone, which only notify when a spending threshold is crossed, not which specific resources are underutilized.

C. Manually re-running the same Cost Explorer report every month and eyeballing the same trends by hand.

D. Deleting the RDS instance immediately with no further investigation, since any utilization under 100% is automatically waste.

---

## Q55

A `Sessions` DynamoDB table stores user session items that should automatically
disappear 24 hours after creation, without any application code running a cleanup job. Which
DynamoDB feature should the developer use, and how?

A. DynamoDB Time to Live (TTL) — set a Unix-epoch-seconds attribute (e.g. `expiresAt`) on each item and enable TTL on that attribute; DynamoDB automatically deletes expired items in the background at no extra write-capacity cost for the deletion.

B. A DynamoDB Stream that triggers a Lambda function to call `DeleteItem` on any item older than 24 hours, polling every minute.

C. A scheduled EventBridge rule that runs a full table `Scan` every hour, deleting items with an old timestamp.

D. Set the table's billing mode to on-demand, which automatically expires old items.

---

## Q56

A chatbot application calls Amazon Bedrock's `InvokeModel` API using a Lambda function's
execution role. The role currently has the managed policy granting `"Action": "bedrock:*",
"Resource": "*"` attached, left over from initial development. A security review flags this under
Task 2.3. What should the policy be changed to?

A. Scope the policy to only `bedrock:InvokeModel` and `bedrock:InvokeModelWithResponseStream` on the specific foundation model ARN(s) the chatbot is approved to use (e.g. one Claude model family ARN), removing management-plane Bedrock actions entirely from the runtime role.

B. Leave it as-is, since Bedrock has no sensitive management-plane actions worth restricting.

C. Replace `bedrock:*` with `s3:*` since the model's underlying weights are stored in S3.

D. Grant the same wildcard policy to every other Lambda function in the account as well, for consistency.

---

## Q57

An order-processing pipeline uses an SQS **FIFO** queue so that updates for the same
order are always processed in the order they were sent, and so that an accidental duplicate
`SendMessage` call (e.g. due to a client-side retry) doesn't create a second entry. Which two
message attributes/settings make this work?

A. `MessageGroupId` set to the order ID (messages sharing a group are delivered in strict FIFO order) and either content-based deduplication or an explicit `MessageDeduplicationId` (to suppress duplicate sends within the 5-minute dedup interval).

B. `DelaySeconds` and `VisibilityTimeout` alone — these are sufficient for both ordering and deduplication on a FIFO queue.

C. Standard queue long polling with `ReceiveMessageWaitTimeSeconds` set to 20.

D. A Lambda function downstream that manually sorts messages by timestamp after receiving them out of order.

---

## Q58

A team is choosing between Amazon EKS and Amazon ECS for a new containerized workload.
The deciding requirement is that the team already has significant in-house Kubernetes tooling,
Helm charts, and `kubectl`-based operational runbooks from workloads running elsewhere (e.g.
on-prem), and wants to reuse that tooling as-is, while still using Fargate so they don't manage EC2
instances. What should they choose?

A. Amazon EKS with Fargate profiles — EKS exposes a standard, compliant Kubernetes API (so existing `kubectl`/Helm/K8s-native tooling works unmodified), and Fargate profiles let specific pods run on Fargate with no EC2 node management.

B. Amazon ECS with the Fargate launch type, since ECS also fully supports Helm charts and raw `kubectl` natively.

C. Amazon ECS with the EC2 launch type, since Fargate cannot be combined with Kubernetes-style tooling under any circumstances.

D. AWS Lambda, since both ECS and EKS are being deprecated in favor of pure serverless functions for containerized workloads.

---

## Q59

A read-heavy product-catalog API hits RDS directly for every request, and RDS CPU and
connection count are both consistently near their limits during peak hours even though most
catalog data changes only a few times a day. The team wants to reduce load on RDS for this
read-heavy, infrequently-changing data without rearchitecting the whole data model. What pattern
should they apply?

A. A cache-aside pattern with Amazon ElastiCache in front of RDS: on a read, check the cache first; on a cache miss, read from RDS and populate the cache with a reasonable TTL; invalidate or update the cache entry when the underlying data changes.

B. Increase the RDS instance's storage size, since storage capacity is what's limiting read throughput.

C. Remove all indexes from the RDS tables to speed up writes, since writes are the real bottleneck.

D. Switch the API to use eventually-consistent DNS resolution, which reduces database load.

---

## Q60

A healthcare company's application calls Amazon Bedrock from Lambda functions running
inside a VPC with no internet gateway or NAT, for compliance reasons requiring no public-internet
egress for any workload handling patient data. The calls currently fail because the subnets have no
route to Bedrock's public endpoint. What is the correct Task 2.3 fix?

A. Create an interface VPC endpoint (AWS PrivateLink) for `bedrock-runtime` (and `bedrock-agent-runtime` if using agents) in the Lambda's VPC, with an endpoint policy restricting which principals/resources can use it, so traffic to Bedrock stays on the AWS private network.

B. Add a NAT gateway and an internet gateway so the subnet can reach Bedrock's public endpoint over the internet.

C. Disable the compliance requirement for this one workload since Bedrock cannot be reached privately.

D. Move the Lambda function out of the VPC entirely, since that is the only way to reach any AWS service.

---

## Q61

A public API Gateway endpoint accepts JSON order submissions. The team wants malformed
requests (missing required fields, wrong data types) rejected with a `400` **before** the request
ever reaches the backend Lambda function, to reduce unnecessary invocations and keep validation
logic out of application code. What should they configure?

A. API Gateway request validation with a JSON Schema model attached to the method, so API Gateway itself checks the payload shape and required fields and rejects non-conforming requests at the gateway.

B. A Lambda authorizer that parses and validates the full request body.

C. VTL mapping templates that silently drop invalid fields and forward the rest to Lambda.

D. Increase the Lambda function's memory so it can validate requests faster.

---

## Q62

Before merging a new Lambda handler that processes refund requests, the team wants to
quickly generate a baseline set of unit tests covering typical inputs, edge cases (empty refund
amount, negative amount), and error paths, to accelerate Task 3.2 test coverage, then have a human
review and refine the generated tests before they're trusted as the real regression suite. Which
practice does this describe?

A. AI-assisted test generation — using an AI coding assistant to draft an initial unit test suite from the handler's code/signature, which a developer then reviews, edits, and validates before it's relied upon as the authoritative regression suite.

B. Skipping unit tests entirely and relying only on manual QA in the `test` stage before promoting to `prod`.

C. Letting the AI-generated tests run in production directly with no review, since AI-generated code is assumed correct.

D. Writing the tests by hand with no tooling assistance, since AI should never be involved in testing.

---

## Q63

After a production incident, an engineer pastes 40,000 lines of CloudWatch Logs and an
X-Ray trace export into an AI assistant and asks it to summarize the likely root cause before the
team's post-incident review. What is the correct way to treat the AI's output, consistent with
Task 4.1's "AI-assisted error analysis" guidance?

A. Treat it as a fast first-pass hypothesis that accelerates triage, then have an engineer verify the suggested root cause against the actual logs/trace/metrics before it's written into the official incident report as the confirmed cause.

B. Copy the AI's summary verbatim into the official incident report as the final, confirmed root cause with no verification.

C. Ignore AI assistance entirely for incident analysis, since it is explicitly out of scope for DVA-C03.

D. Use the AI's summary to automatically close the incident ticket without any human reviewing it.

---

## Q64

Choose **TWO** correct Task 2.3 mitigations for a customer-facing Bedrock-powered
support chatbot that must (a) refuse to output harmful/toxic content and resist prompt-injection
attempts hidden in user messages, and (b) never let an attached Bedrock Agent take actions beyond
what the support use case requires, even if a malicious user tries to manipulate the agent into
calling an unintended tool.

A. Configure Amazon Bedrock Guardrails with content filters and denied-topic/word policies, and apply them to both user input and model output, which also helps detect and block prompt-injection-style attempts to override the system instructions.

B. Scope the Bedrock Agent's own execution role to only the specific action groups/Lambda functions the support use case legitimately needs, following least privilege, rather than granting it a broad set of tools 'just in case.'

C. Trust the foundation model's own built-in judgment entirely and skip Guardrails, since modern models never fall for prompt injection.

D. Grant the Bedrock Agent `AdministratorAccess` so it never fails due to missing permissions during a demo.

E. Rely solely on client-side (browser) input filtering, since server-side filtering is redundant for a chatbot.

---

## Q65

Choose **TWO** correct statements about using Amazon SNS to fan out a single
`PaymentReceived` notification to multiple SQS queues (one per downstream team), where each team's
queue should only receive messages for payments over $500 in their own region.

A. SNS subscription filter policies can be attached per-subscription so each SQS queue only receives messages matching that subscription's filter policy (e.g. matching on message attributes like `amount` and `region`), even though all subscriptions point at the same topic.

B. Filtering happens on message **attributes**, not the message body, so the publisher should set relevant fields (like `amount`, `region`) as SNS message attributes, not bury them only in the JSON body.

C. SNS topics can only fan out to Lambda functions, never to SQS queues.

D. Filter policies are applied by the publisher before `Publish` is even called, not by SNS at delivery time.

E. Every subscriber to an SNS topic always receives every message published to that topic, with no way to narrow delivery per subscriber.

---
