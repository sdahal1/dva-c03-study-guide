# DVA-C03 Full Mock Exam 2 — Questions

65 questions. Multiple choice (pick ONE) unless a question explicitly says **Choose TWO**. No answers are shown in this file — see the matching `*-answer-key.md` file after you finish (or after each section if you're reviewing, not simulating).

---

## Q1

A DynamoDB table `Inventory` uses partition key `sku` and stores a `stockCount`
attribute. Multiple warehouse scanners increment `stockCount` concurrently as items arrive. Using a
plain `GetItem` followed by an application-side `stockCount + 1` and then `PutItem` occasionally
loses updates when two scanners process items at nearly the same moment. What single-call change
fixes this race condition cleanly?

A. Use `UpdateItem` with an `ADD` (or `SET stockCount = stockCount + :incr`) update expression, which performs the increment atomically inside DynamoDB itself rather than reading, computing, and writing back in application code.

B. Read with `ConsistentRead: true` before writing, which alone prevents the race.

C. Have scanners write to a separate table per scanner and sum the tables nightly.

D. Add a `ProjectionExpression` restricting the write to only the `stockCount` attribute.

---

## Q2

An IAM policy attached to a developer role includes:
```json
{
  "Effect": "Allow",
  "NotAction": "iam:*",
  "Resource": "*"
}
```
with no other statements in the policy. What does this statement actually grant, and why is
`NotAction` combined with `Resource: *` and no other conditions a common Task 2.1 misconfiguration
to watch for?

A. It allows every AWS action **except** the listed `iam:*` actions, on every resource — meaning the role gets broad access across virtually every other service (EC2, S3, DynamoDB, etc.), which is almost certainly far more permissive than intended, even though it looks restrictive at a glance because of the word 'Not'.

B. It denies every action except `iam:*`, which is the opposite of an overly broad grant.

C. `NotAction` has no effect when combined with `Allow`; this statement grants nothing at all.

D. It only allows IAM actions, since `NotAction` is a typo for `Action` and AWS silently corrects it.

---

## Q3

A team defines infrastructure as AWS CDK (TypeScript) constructs. Before every deploy,
they want to catch issues like a security group allowing unrestricted inbound SSH, or an S3 bucket
missing encryption, as part of CI — ideally before `cdk deploy` ever runs, using rules that can be
customized for the organization's own security standards (not just generic CloudFormation syntax
validity). What should they add to the pipeline?

A. A CDK-aware static analysis/linting step (e.g. `cdk-nag`, run against the synthesized CloudFormation template or directly against CDK constructs) as a CI stage after `cdk synth` and before `cdk deploy`, failing the build if a defined security rule (like 'no 0.0.0.0/0 on port 22') is violated.

B. Rely solely on `cdk synth` succeeding, since successful synthesis guarantees the infrastructure is secure.

C. Manually review every CDK diff by eye before every deployment, with no tooling.

D. Wait for AWS Config to flag the problem after the resources are already deployed to production.

---

## Q4

A production incident report needs to answer: "across all Lambda functions in this
application, which specific function and which specific log line first mentioned the string
`DatabaseConnectionTimeout` in the last 6 hours?" The team has 12 different Lambda functions, each
with its own log group. What is the fastest way to answer this without opening each log group one
by one?

A. A single CloudWatch Logs Insights query run across all 12 log groups simultaneously (Logs Insights supports querying multiple log groups in one query), filtering for the exact string and sorting by `@timestamp` ascending to surface the first occurrence.

B. Open each of the 12 log groups' consoles in 12 separate browser tabs and manually scan each one.

C. Export every log group's full contents to a local file and use a text editor's 'Find in Files' across all 12 files.

D. Assume the error always originates in the same function every time, since errors are deterministic.

---

## Q5

A video-processing pipeline ingests a Kinesis Data Stream with 8 shards. Traffic has
grown and the stream now regularly hits `WriteProvisionedThroughputExceededException` during peak
hours, while off-peak traffic fits comfortably within 8 shards. The team wants to increase write
throughput for peak periods without over-provisioning (and over-paying for) capacity during quiet
hours. What should they do?

A. Enable Kinesis Data Streams On-Demand capacity mode, which automatically scales shard capacity up and down based on observed traffic, removing the need to manually reshard for predictable peak/off-peak cycles.

B. Permanently split every shard into two, doubling the shard count forever regardless of time of day.

C. Switch to a Standard SQS queue, which has unlimited throughput with no configuration needed.

D. Disable the stream's retention period to increase write throughput.

---

## Q6

A GitHub Actions CI/CD pipeline needs to deploy to AWS without storing any long-lived
AWS access keys as GitHub repository secrets, since the team wants to eliminate the risk of a
leaked long-lived credential entirely. What AWS mechanism supports this keyless pattern?

A. `AssumeRoleWithWebIdentity` using OIDC federation — configure an IAM OIDC identity provider trusting GitHub's OIDC token issuer, and a role trust policy scoped to the specific repository/branch, so GitHub Actions exchanges its short-lived OIDC token for short-lived AWS credentials on every run with no stored AWS keys at all.

B. Store a long-lived IAM user's access key and secret key as encrypted GitHub secrets, and accept the leak risk as unavoidable.

C. Use the AWS root account's access keys in the pipeline, since root keys never expire and are therefore more reliable for automation.

D. Hardcode the credentials directly in the workflow YAML file for simplicity, since GitHub's repository is private.

---

## Q7

A developer wants a disposable, pre-configured cloud development environment with the
AWS CLI, SDKs, and a code editor already set up, accessible from any browser, to avoid "works on
my machine" setup drift across a team of contractors who each only need the environment for a few
weeks. Which AWS service is designed for this, as opposed to each contractor configuring their own
local machine?

A. AWS Cloud9 — a cloud-based IDE accessible via browser, pre-configured with AWS CLI credentials tied to the launching user's role, requiring no per-contractor local environment setup, and easy to tear down when the engagement ends.

B. Amazon EC2 with no further configuration, since a bare EC2 instance already includes a browser-based IDE by default.

C. AWS CloudShell, which permanently persists all installed custom packages and tools indefinitely with no storage limit.

D. Amazon WorkSpaces, which is a service exclusively for watching streaming video content.

---

## Q8

An X-Ray trace shows a `GetUserProfile` request touching four downstream services. The
team wants to tag specific traces with a business-relevant value (`customerId`) so they can later
filter and search traces in the X-Ray console for a specific customer's requests, in addition to
attaching non-searchable debug context (like the full raw request object) to the same trace
segment for engineers reviewing it by hand. Which two X-Ray concepts map to these two needs?

A. Annotations — key-value pairs that are indexed and can be used to filter/search traces in the X-Ray console (e.g. `customerId`) — and metadata — arbitrary additional data attached to a segment that is *not* indexed/searchable but is still visible when inspecting a specific trace in detail (e.g. the full raw request object).

B. Annotations and metadata are identical in every way, including both being searchable in the console.

C. Subsegments are the only mechanism for attaching any extra data to an X-Ray trace.

D. X-Ray does not support attaching any custom key-value data to a trace beyond the default segment fields.

---

## Q9

An API Gateway REST API uses a **non-proxy (custom)** Lambda integration. The backend
Lambda returns `{"order": {"id": "123", "total": 42.5}}`, but the client needs the response shaped
as `{"orderId": "123", "amount": 42.5}` without modifying the Lambda function's code (it's shared
by other consumers that expect the original shape). What should the developer configure?

A. An integration response mapping template (VTL) on the method, which transforms the Lambda's raw output into the client-facing shape before API Gateway returns it — exactly the use case for non-proxy integration's response mapping.

B. Switch to Lambda proxy integration, which automatically reshapes JSON for every client without any mapping template.

C. Ask every consuming client to parse the original shape themselves and ignore the requirement.

D. Add a second Lambda function that does nothing but rename two JSON fields, and chain it after the first via a Step Functions workflow for every single request.

---

## Q10

A company's mobile app allows anyone to sign up with an email address, but the team
only wants to allow sign-ups from company-approved partner domains (e.g. `@partnerco.com`), rejecting
all others at sign-up time, before an account is ever created. Which Cognito feature implements
this custom business rule?

A. A Cognito user pool **Pre Sign-up** Lambda trigger — it runs synchronously during the sign-up flow and can inspect the submitted email domain, returning an error to reject the sign-up (or auto-confirm it) before the user record is created.

B. An IAM policy attached to the user pool restricting which email domains can call `SignUp`.

C. A Cognito identity pool role mapping rule, since identity pools handle sign-up validation.

D. Manually reviewing every new sign-up in the console and deleting unapproved ones after the fact.

---

## Q11

A team's CodePipeline deploys a CloudFormation stack. A recent incident revealed that
a stack update unintentionally replaced (destroyed and recreated) a stateful RDS instance because
a property change triggered implicit resource replacement, causing data loss. The team wants this
caught automatically in the pipeline before any such destructive change reaches production. What
should be added?

A. A pipeline stage that creates a CloudFormation change set from the proposed template and inspects its output programmatically (or via a manual approval gate shown the change set) specifically checking for any resource marked for `Replacement: True`/`Conditional` on stateful resources, failing/pausing the pipeline if found.

B. Nothing — CloudFormation never replaces a resource unless the developer explicitly requests it via a separate, distinct API call.

C. Remove CloudFormation entirely and manage all infrastructure via the console to avoid replacement risk.

D. Increase the RDS instance's backup retention period, which prevents CloudFormation from ever replacing the instance.

---

## Q12

A security review of CloudTrail logs over the past week shows an unusually high
volume of `AssumeRole` calls from a single IAM user, far above that user's historical baseline,
though none of the individual calls look obviously malicious on their own. The team wants
automated detection of this kind of statistical/volume anomaly going forward, rather than manually
eyeballing CloudTrail event counts. What CloudTrail feature is designed for this?

A. CloudTrail Insights — it automatically analyzes management event patterns (like API call volume) against a learned baseline and surfaces unusual activity (e.g. spikes in a specific API call's volume) as its own type of finding, without requiring a custom query or manual historical comparison.

B. CloudTrail data events, since they are specifically designed for statistical anomaly detection on API call volume.

C. AWS Config, since it tracks volumetric trends in API call counts by default.

D. Manually exporting CloudTrail logs to a spreadsheet every week and eyeballing the row counts.

---

## Q13

A Lambda function needs to call a private RDS instance inside a VPC. After attaching
the function to the VPC's private subnets with no NAT gateway, the function can reach RDS fine, but
now fails when it also needs to call the Bedrock API, which has no VPC gateway endpoint equivalent
(interface endpoints only). What fixes the Bedrock connectivity without exposing the RDS-facing
subnet to the broader internet unnecessarily?

A. Create an interface VPC endpoint for `bedrock-runtime` in the same subnets/route tables the Lambda function uses, so Bedrock traffic routes privately through the endpoint's ENI rather than needing internet access at all.

B. Add a NAT gateway and route 0.0.0.0/0 through it, granting the subnet full internet access just to reach Bedrock.

C. Give up on using Bedrock from this function and only call it from a different, public-subnet function instead.

D. Increase the Lambda function's timeout so the call to Bedrock eventually succeeds despite having no route.

---

## Q14

A company uses AWS IAM Identity Center with permission sets assigned per-account.
A contractor needs `ReadOnlyAccess`-level permissions in exactly one of 30 member accounts for a
3-month engagement, and the team wants this scoped as tightly and auditable as possible, with easy
revocation at the end of the engagement. What is the correct approach, as opposed to creating a
standalone IAM user directly in that one account?

A. Create (or reuse) a `ReadOnly` permission set in IAM Identity Center and assign it to the contractor's Identity Center user for only that one specific account — granting access through the centrally managed assignment, which can be revoked instantly by removing the assignment, with Identity Center's own audit trail of who has what.

B. Create a standalone IAM user in that one account with `ReadOnlyAccess` attached directly, and share its password with the contractor over email.

C. Grant the contractor `AdministratorAccess` in all 30 accounts temporarily, since permission sets cannot be scoped to a single account.

D. Add the contractor to the AWS Organizations management account's root user credentials.

---

## Q15

A Lambda layer published in Account A (`arn:aws:lambda:us-east-1:111122223333:layer:shared-utils:3`)
needs to be used by Lambda functions in Account B. The functions in Account B currently fail to
attach the layer with a permissions error. What must be configured in Account A to allow this?

A. Add a layer-version resource-based permission (via `lambda:AddLayerVersionPermission`) granting Account B (or a specific principal in it) `lambda:GetLayerVersion` on that exact layer version ARN.

B. Nothing — Lambda layers are public by default across all AWS accounts once published.

C. Copy the entire layer's .zip contents manually into Account B's own S3 bucket and re-publish it as an identical layer there, since cross-account layer sharing isn't supported at all.

D. Grant Account B's functions `lambda:*` in Account A's IAM policies.

---

## Q16

A containerized microservice built in CI needs to be shared with a partner
organization's separate AWS account so their pipeline can pull and deploy the exact same image,
without making the image publicly accessible to the entire internet. What should be configured on
the ECR repository?

A. An ECR repository policy granting the partner's specific AWS account (or IAM principal ARN) `ecr:GetDownloadUrlForLayer`, `ecr:BatchGetImage`, and `ecr:BatchCheckLayerAvailability` permissions on that repository, scoping cross-account pull access to exactly that account.

B. Make the ECR repository fully public, since that is the only way to share an image across AWS accounts.

C. Export the image as a `.tar` file and email it to the partner organization's team.

D. Grant the partner account `AdministratorAccess` in the account that owns the ECR repository.

---

## Q17

A Lambda function is consistently throttled with `TooManyRequestsException` during a
sustained traffic spike, and investigation shows the function's reserved concurrency is set
appropriately high already — the actual constraint is the **account-level** concurrent execution
limit being shared across dozens of other unrelated functions in the same account/Region. What is
the correct, officially supported path to resolve this, as opposed to arbitrarily lowering other
functions' concurrency?

A. Request a service quota increase for the account's total concurrent executions limit in that Region via the Service Quotas console/API, which raises the shared ceiling rather than redistributing an already-insufficient pool among competing functions.

B. Manually lower every other function's reserved concurrency to zero so this one function gets effectively unlimited concurrency by default.

C. Delete the other, unrelated functions permanently to free up account-level concurrency.

D. Nothing can be done — AWS account concurrency limits are immutable and can never be increased under any circumstances.

---

## Q18

A company wants every account in a specific Organizational Unit (OU) to be prevented
from ever disabling AWS CloudTrail or deleting its S3 log bucket, even by an account's own
administrator, to guarantee an unbroken audit trail for compliance. What is the correct control?

A. An SCP attached to that OU that explicitly denies `cloudtrail:StopLogging`, `cloudtrail:DeleteTrail`, and the relevant `s3:DeleteBucket`/`s3:DeleteObject` actions on the logging bucket, for every principal in accounts under that OU — including the account's own root/admin user.

B. A CloudTrail configuration setting called 'TamperProof Mode' that any account admin can toggle on.

C. An IAM policy attached only to the specific CloudTrail service role, with no SCP involved.

D. Trusting that account administrators will simply not do this, with no technical control.

---

## Q19

A Lambda function configured with provisioned concurrency of 20 is deployed with a new
version during a release. Traffic is currently routed to the alias pointing at the previous
version. What happens to provisioned concurrency during this deployment, and what should the team
configure to avoid a gap in pre-warmed capacity?

A. Provisioned concurrency is tied to a specific published version (or an alias pointing at one); the team should configure provisioned concurrency on the alias itself (or use CodeDeploy's traffic-shifting with pre-provisioning) so the new version has its own warmed pool ready before traffic shifts, avoiding a gap where the new version briefly serves only cold-started invocations.

B. Provisioned concurrency automatically transfers itself to whatever the newest published version is, with no configuration needed.

C. Provisioned concurrency only applies to the `$LATEST` version and is irrelevant to aliases entirely.

D. Nothing needs to change — reserved concurrency and provisioned concurrency are the same setting and move together automatically.

---

## Q20

A Lambda-based API's test suite currently only runs locally on developers' machines
before they manually deploy, with no consistency guarantee that the same tests ran before a given
production deploy. The team wants every deployment to `prod` to be provably gated on that
specific commit's automated test suite passing, as an unskippable step, not a developer's manual
habit. What should be restructured in the pipeline?

A. Move the test execution into a CodeBuild stage inside CodePipeline, positioned before the production deployment stage, so the pipeline itself (not a developer's local habit) runs the tests for that exact commit and the pipeline cannot proceed to the deploy stage unless that CodeBuild stage reports success.

B. Keep tests local-only but add a Slack reminder asking developers to 'please run tests before deploying.'

C. Move testing to occur only after the production deployment completes, as a post-deploy smoke test with no gating effect.

D. Require developers to attach a screenshot of their local test run output to the pull request description as proof.

---

## Q21

A team wants a single CloudWatch dashboard giving leadership visibility into key
metrics (API error rate, Lambda duration, DynamoDB throttles) across **three separate AWS accounts**
(dev, staging, prod) without requiring leadership to switch accounts in the console or maintain
three separate dashboards. What CloudWatch capability supports this?

A. CloudWatch cross-account, cross-Region dashboards (using CloudWatch's account linking / Observability Access Manager) — a dashboard in a designated monitoring account can display metrics sourced from multiple linked member accounts side by side in one view.

B. CloudWatch dashboards are strictly single-account; the team must have leadership log into three separate accounts and view three separate dashboards.

C. Copy and paste metric values manually into a shared spreadsheet from each account's console once a day.

D. Use AWS Budgets, since it natively aggregates CloudWatch metrics across accounts for dashboard purposes.

---

## Q22

Two internal microservices communicate over mutual TLS (mTLS) within a private VPC,
and the team wants certificates issued by an internal certificate authority that's trusted only
within the organization (not a publicly trusted CA, since these services are never exposed to the
public internet). What AWS service should issue these certificates?

A. AWS Private Certificate Authority (Private CA) — it operates a managed internal CA hierarchy that issues certificates trusted by anything configured to trust that private root/subordinate CA, appropriate for internal mTLS between services that should never need a publicly trusted certificate.

B. AWS Certificate Manager's standard public certificate issuance flow, since all ACM certificates are automatically trusted by every client worldwide, public or internal.

C. Amazon Cognito, since it can issue arbitrary X.509 certificates for any use case.

D. AWS KMS, since KMS keys can be directly used as TLS certificates with no conversion needed.

---

## Q23

A document-processing Lambda is triggered by an SQS queue with event source mapping. A
specific malformed message consistently causes the handler to throw an exception on every delivery
attempt, and because the queue's redrive policy has no dead-letter queue configured, the same
message keeps cycling back after each visibility timeout, consuming Lambda invocations indefinitely
and blocking the processing of other, valid messages in the same batch. What should be configured?

A. Configure a redrive policy on the source queue pointing at a dead-letter queue (DLQ), with `maxReceiveCount` set so that after N failed delivery attempts, SQS automatically moves the poison message to the DLQ instead of redelivering it forever — freeing the main queue (and batches) from being blocked by it.

B. Increase the Lambda function's memory, which will make the malformed message parse correctly.

C. Delete the entire queue and recreate it, which will discard the poison message along with every other valid message currently queued.

D. Set the queue's visibility timeout to 0 so the message is never considered in-flight.

---

## Q24

A media company serves large video files from S3 to a globally distributed user base
uploading from mobile devices on inconsistent networks, and wants to minimize the time it takes for
uploads to actually start being received and reduce the impact of long-distance network latency to
the bucket's Region. Which S3 feature is designed specifically for this upload-side latency
problem?

A. S3 Transfer Acceleration — it routes uploads through the nearest Amazon CloudFront edge location onto Amazon's optimized backbone network to the bucket's Region, reducing the effect of long-distance public internet latency on upload speed.

B. S3 Intelligent-Tiering, which automatically moves objects between storage classes based on access patterns.

C. S3 Object Lock, which enforces WORM (write-once-read-many) retention.

D. Cross-Region Replication (CRR), configured to replicate objects after they're already fully uploaded.

---

## Q25

An application needs to encrypt a large (50 MB) file client-side before uploading it
to S3, using envelope encryption with a KMS customer-managed key, without sending the entire 50 MB
payload to KMS itself (which has request size and throughput limits unsuited to bulk data
encryption). What is the correct envelope encryption flow?

A. Call `kms:GenerateDataKey` to get a plaintext data key and its KMS-encrypted counterpart; use the plaintext data key locally (client-side) to encrypt the 50 MB file with a fast symmetric cipher (e.g. AES-256-GCM); discard the plaintext data key from memory; store the encrypted data key alongside the encrypted file (e.g. as S3 object metadata), so decryption later calls `kms:Decrypt` on just the small data key, not the large file.

B. Send the full 50 MB file directly to `kms:Encrypt`, since KMS has no practical size limit on the data it can encrypt directly.

C. Use the KMS key's key material to manually derive an AES key using custom application code instead of calling KMS APIs at all.

D. Store the file unencrypted and instead encrypt only its filename using KMS, which is sufficient for compliance.

---

## Q26

A Lambda function's deployment currently replaces 100% of traffic on the alias
immediately (`AllAtOnce`), and a recent bad deploy caused every request to fail for several
minutes before anyone noticed and manually rolled back. The team wants new deployments to shift
traffic gradually in linear increments over 10 minutes, with CloudWatch-alarm-triggered automatic
rollback so a bad deploy affects only a small percentage of traffic briefly rather than everyone.
What CodeDeploy configuration achieves this?

A. A CodeDeploy `Linear` deployment configuration for Lambda (e.g. `Linear10PercentEvery1Minute`), which shifts an additional fixed percentage of traffic to the new version at each defined interval, combined with a CloudWatch alarm attached to the deployment group so a triggered alarm automatically halts and rolls back the shift mid-way.

B. Switching the alias's traffic-shifting configuration manually to a new fixed number every few minutes, watching metrics by eye, with no CodeDeploy involvement.

C. Keeping `AllAtOnce` but simply reducing the Lambda function's memory, since smaller functions deploy more safely.

D. Running the exact same deployment five times in a row.

---

## Q27

An application instruments its Lambda functions with the X-Ray SDK, wrapping all
outbound AWS SDK calls for automatic tracing. The team additionally wants X-Ray to capture a
subsegment specifically around a call to a *third-party* (non-AWS) HTTP API, so that a slow
third-party dependency shows up distinctly in the trace timeline rather than being invisible or
lumped into the handler's own segment duration. What should they add?

A. Wrap the third-party HTTP call in an explicit X-Ray subsegment (e.g. using the SDK's `captureHTTPsGlobal`/manual subsegment API, or a custom `beginSubsegment`/`endSubsegment` pair around the call) so that call's duration is recorded and displayed as its own distinct, labeled block in the trace timeline.

B. Do nothing additional — X-Ray automatically instruments every outbound HTTP call to any domain, AWS or third-party, with zero configuration.

C. Switch the third-party API call to use the AWS SDK instead, since only AWS SDK calls can ever appear in an X-Ray trace.

D. Disable X-Ray entirely, since it can only trace calls between AWS services and never third-party dependencies.

---

## Q28

A `Flights` table stores items keyed by `flightId` (partition key). A new requirement
needs a strict guarantee: when booking the last available seat, deduct from `seatsAvailable` on the
`Flights` item **and** insert a new item into a `Bookings` table **atomically** — either both
succeed or neither does, with no possibility of a booking existing without a matching seat
deduction (or vice versa). Which DynamoDB capability provides this cross-item, cross-table
guarantee?

A. `TransactWriteItems` — it groups multiple `Put`/`Update`/`Delete`/`ConditionCheck` operations across one or more tables into a single all-or-nothing atomic transaction.

B. Two separate `UpdateItem`/`PutItem` calls made back-to-back in application code, since DynamoDB automatically treats sequential calls from the same Lambda invocation as one transaction.

C. DynamoDB Streams, configured to automatically roll back the first write if the second write fails.

D. Strongly consistent reads on both tables before writing.

---

## Q29

An application reads a non-sensitive application configuration value (`maxRetries: 3`)
from AWS Systems Manager Parameter Store on every Lambda cold start. The team notices this adds
noticeable latency to every cold start and wants to reduce the number of `GetParameter` API calls
without hardcoding the value or losing the ability to update it without a redeploy. What should
they add?

A. The AWS Parameters and Secrets Lambda Extension (or an in-memory cache with a TTL inside the handler) to cache the parameter value for a configurable duration within the execution environment, avoiding a fresh `GetParameter` call on every single invocation while still allowing updates to propagate after the cache TTL expires.

B. Call `GetParameter` once at build time and bake the resulting value permanently into the deployment package.

C. Increase the Lambda function's memory, which caches Parameter Store values automatically.

D. Switch from Parameter Store to Secrets Manager, which has zero API latency by design.

---

## Q30

A team's CloudFormation template currently stores a third-party API key as a plain
string `Default` value in a template `Parameter`, which means it's visible in the CloudFormation
console's parameter history and in the template source committed to version control. They need to
reference this secret from a resource's configuration without ever exposing the plaintext value in
the template, console history, or source control. What should they use instead?

A. A CloudFormation dynamic reference to Secrets Manager (e.g. `{{resolve:secretsmanager:MySecretId:SecretString:apiKey}}`) inside the resource property, which CloudFormation resolves at deploy/update time without ever writing the resolved plaintext value into the template, parameter history, or any stored stack configuration that a casual viewer could see.

B. Keep the plaintext default value in the template but add a code comment saying 'DO NOT SHARE THIS FILE.'

C. Base64-encode the API key before putting it in the template's `Default` value, since Base64 is a form of encryption.

D. Store the key in a `NoEcho` parameter with no other change, since `NoEcho` fully removes the value from all storage and history everywhere.

---

## Q31

An AWS Config conformance pack is deployed across an Organization to continuously
check that every account's resources comply with a defined baseline (encryption enabled, no
public S3 buckets, etc.). A new account is provisioned and, 10 minutes later, a developer creates
a non-compliant public S3 bucket. What is the expected behavior, and how does this differ from a
one-time compliance audit script run manually once a quarter?

A. The conformance pack's underlying Config rules evaluate the new bucket's configuration automatically and flag it as non-compliant close to real-time (as the resource's configuration is recorded), rather than waiting for the next scheduled manual audit — meaning drift is caught within minutes/hours of occurring instead of up to a full quarter later.

B. Conformance packs only evaluate resources that existed at the moment the pack was deployed; resources created afterward are never evaluated.

C. A quarterly manual script and a Config conformance pack are functionally identical in detection latency, since both ultimately just check the same resource properties.

D. Conformance packs require the developer to manually trigger an evaluation via the console after creating each resource.

---

## Q32

Choose **TWO** correct statements about using API Gateway usage plans and API keys to
manage a public API offered to external third-party developers, where different subscription tiers
(Free, Pro) should get different request-rate limits and usage quotas, and the team needs to
measure/limit usage per customer.

A. A usage plan can define throttling (steady-state rate and burst) and a quota (e.g. 10,000 requests/month) that applies to every API key associated with that plan — different plans (Free vs. Pro) can have different throttle/quota values.

B. An API key uniquely identifies a calling customer for metering purposes and is associated with one or more usage plans, letting API Gateway track and enforce that specific customer's consumption against the plan's limits.

C. API keys alone, with no usage plan attached, enforce rate limiting and quotas automatically.

D. Usage plans can only be applied to private (VPC-only) APIs, never to public APIs.

E. Changing a customer's usage plan always requires generating them a brand-new API key.

---

## Q33

A company runs automated, recurring Amazon Macie sensitive-data discovery jobs across
all its S3 buckets, and separately wants ongoing vulnerability scanning of its EC2 instances and
container images for known CVEs. Which two services, together, cover these two distinct needs?

A. Amazon Macie for sensitive-data discovery in S3 (already in place) and Amazon Inspector for automated vulnerability scanning of EC2 instances and container images against known CVE databases.

B. Amazon GuardDuty alone covers both sensitive-data discovery and vulnerability scanning, making a second service unnecessary.

C. AWS Config alone covers both needs, since it can scan both object content and CVE databases.

D. AWS Trusted Advisor is the correct tool for CVE-level vulnerability scanning of running workloads.

---

## Q34

Choose **TWO** correct statements about choosing between AWS Step Functions Standard
and Express workflows for a new order-processing state machine that calls several Lambda functions
and needs to run for up to 2 hours end-to-end with exactly-once execution semantics and a full
execution history visible in the console for auditing.

A. Standard Workflows support executions lasting up to one year, provide exactly-once workflow execution semantics, and retain a full, visualized execution history for every run in the console.

B. Express Workflows are optimized for high-volume, short-duration workloads (up to 5 minutes), using at-least-once execution semantics, making them a poor fit for a 2-hour exactly-once requirement.

C. Express Workflows can run for up to one year, identical to Standard Workflows.

D. Standard Workflows provide only at-least-once execution semantics, identical to Express.

E. Pricing for Standard Workflows is based on the number of state transitions, which matters for a long-running workflow with many steps.

---

## Q35

A developer wants to simulate how their Lambda function responds to an SNS-triggered
event, entirely offline, feeding it a realistic sample SNS notification payload shaped exactly like
what SNS would actually deliver, to catch a parsing bug before ever deploying to AWS. What SAM CLI
feature/workflow supports this specific scenario?

A. `sam local invoke` with a generated or hand-crafted SNS-shaped test event JSON file (SAM/AWS provides sample event templates for common triggers like SNS, S3, and SQS) passed via `--event`, invoking the handler locally against that exact payload shape.

B. `sam local invoke` always generates a random, unpredictable test event automatically, with no way to control its shape.

C. Deploy the function to production first and trigger a real SNS notification, since local testing cannot replicate SNS's exact payload shape.

D. `sam package`, which is specifically designed for local event simulation rather than packaging.

---

## Q36

A read-heavy application queries DynamoDB directly for individual item lookups by
primary key, and the team wants to add a cache layer specifically optimized for microsecond-level
read latency in front of DynamoDB, without re-architecting the application's DynamoDB API calls
(`GetItem`/`Query`) or introducing a general-purpose caching service that would need its own
separate cache-invalidation logic written by hand. What AWS service is purpose-built for this exact
case?

A. Amazon DynamoDB Accelerator (DAX) — a DynamoDB-specific, in-memory caching layer that is API-compatible with the DynamoDB SDK calls the application already makes, automatically handling cache population/invalidation for `GetItem`/`Query`/`Scan` without the application writing custom cache logic.

B. Amazon ElastiCache for Redis, requiring the application to be rewritten to call a separate Redis client and manually implement cache-aside logic including invalidation.

C. Amazon CloudFront, since it caches DynamoDB API responses automatically by default.

D. AWS Global Accelerator, which caches database query results at the network layer.

---

## Q37

Under the AWS Shared Responsibility Model, for an application using Amazon RDS
(a managed database service, as opposed to self-managed EC2-hosted MySQL), which of the following
is AWS's responsibility rather than the customer's?

A. Patching the underlying database engine's software, OS, and host infrastructure for a standard RDS-managed deployment.

B. Configuring the database's security group rules to restrict network access appropriately.

C. Designing a least-privilege IAM policy for the application's role that connects to the database.

D. Choosing strong master credentials and rotating them on an appropriate schedule.

---

## Q38

A ride-sharing app's AppSync GraphQL API needs to push live driver-location updates to
a rider's app the instant the driver's position changes, without the rider's client polling on an
interval. Which AppSync capability implements this push-based, real-time delivery over a
persistent connection?

A. AppSync Subscriptions — clients subscribe to a GraphQL subscription operation, and AppSync pushes matching data to connected clients in real time over a managed WebSocket connection whenever a corresponding mutation occurs.

B. Clients polling a `getDriverLocation` query every 500 milliseconds.

C. A scheduled EventBridge rule that emails the rider a PDF of the driver's location every 5 minutes.

D. API Gateway WebSocket APIs are the only AWS mechanism that can do this; AppSync has no equivalent.

---

## Q39

A `buildspec.yml` needs to build and push a Docker image to ECR as part of a
CodeBuild stage. The build environment requires access to the Docker daemon (to run `docker build`
and `docker push`), which is not available by default in a standard CodeBuild environment. What
setting must be enabled on the CodeBuild project for this to work?

A. Enable privileged mode on the CodeBuild project, which starts a Docker daemon inside the build container so `docker build`/`docker push` commands can run; without it, Docker-in-Docker operations fail.

B. No special configuration is needed — every CodeBuild environment has a running Docker daemon by default with no flags required.

C. Switch the source provider from CodeCommit to GitHub, since Docker builds only work with GitHub-sourced CodeBuild projects.

D. Increase the CodeBuild project's compute type to the largest available size, which automatically enables Docker support.

---

## Q40

A team wants to use an AI assistant to analyze AWS Compute Optimizer and Lambda Power
Tuning output and propose specific rightsizing changes (e.g. "reduce this Lambda's memory from
1024 MB to 512 MB," "resize this RDS instance down one size") across dozens of resources, to save
engineering time versus manually reviewing every recommendation. Consistent with the documented
Task 4.3 "AI-assisted optimization recommendations" pattern, what should happen before any of these
changes are actually applied to production resources?

A. An engineer reviews the AI-synthesized recommendations against the actual workload's real requirements (including less-obvious factors like burst capacity needs or SLA headroom) and explicitly approves which specific changes to apply, rather than letting the recommendations auto-apply to production untouched.

B. Apply every AI-generated recommendation automatically and immediately to every production resource with no review, since the underlying tools (Compute Optimizer, Power Tuning) are AWS-provided and therefore inherently production-safe to blindly trust.

C. Ignore the AI-assisted analysis entirely and only ever consider optimization recommendations typed by a human from scratch.

D. Apply the recommendations only to the single most expensive resource and skip reviewing the rest entirely, assuming the AI prioritized correctly with no need to check.

---

## Q41

A web application behind CloudFront and API Gateway is being probed with crafted
requests attempting SQL injection in query string parameters. The team wants these blocked
automatically at the edge using AWS-maintained, continuously updated rule sets, without writing and
maintaining custom regex rules themselves. What should they use?

A. An AWS WAF web ACL with the AWS Managed Rules groups (e.g. the SQL database rule group and the Core Rule Set), associated with the CloudFront distribution/API Gateway stage, so known SQL injection patterns are blocked using rules AWS maintains and updates on an ongoing basis.

B. A custom Lambda@Edge function that the team writes and maintains itself to pattern-match every known SQL injection signature manually.

C. AWS Shield Standard, which includes SQL injection-specific managed rule groups.

D. Amazon GuardDuty, configured to block requests matching SQL injection signatures in real time.

---

## Q42

Choose **TWO** correct statements about choosing Amazon Aurora over standard Amazon
RDS (e.g. RDS for MySQL) for a new relational workload expected to scale significantly and require
fast failover.

A. Aurora separates compute from a distributed, auto-scaling storage layer (replicated across 3 Availability Zones by design), which lets it scale storage automatically without the manual storage-resizing operations needed on standard RDS.

B. Aurora Replicas typically fail over faster than RDS Multi-AZ standby promotion, because replicas share the writer's storage volume and don't need to replay a separate replication stream from scratch to catch up.

C. Aurora and RDS for MySQL have identical maximum storage limits and failover times, with no meaningful difference between them.

D. Aurora only supports a single Availability Zone per cluster, making it less resilient than RDS Multi-AZ.

E. Standard RDS automatically converts to Aurora at midnight UTC if usage exceeds a certain threshold.

---

## Q43

A team's CodePipeline source stage currently only triggers on pushes directly to
`main`. They've adopted a trunk-based workflow where short-lived feature branches are merged via
pull request, and want a **separate**, lower-stakes pipeline to run build and unit tests (but not
deploy anywhere) on every push to any feature branch, distinct from the main pipeline that builds,
tests, and deploys to production from `main`. What is the correct way to structure this?

A. Create a second, separate CodePipeline (or CodeBuild project triggered independently) whose source stage filters on feature branch pushes/pull requests, running only build+test stages with no deployment stages at all — kept entirely distinct from the main-branch pipeline that includes production deployment stages.

B. Add a `main`-only conditional inside the existing production pipeline's deploy stage that silently no-ops when triggered from a feature branch, while keeping everything else about the single pipeline identical.

C. Rename the feature branches to `main` temporarily whenever they need to be tested, then rename them back.

D. Disable the production pipeline entirely whenever any feature branch exists in the repository.

---

## Q44

A team suspects a specific Lambda function is the root cause of elevated latency in a
distributed checkout flow, but that function doesn't appear anywhere in the X-Ray service map for
these requests, even though CloudWatch Logs confirm the function is definitely being invoked during
checkout. What is the most likely reason the function is invisible in the trace, and the fix?

A. Active tracing is not enabled on that specific Lambda function (or its upstream caller isn't passing the trace context/header along), so no segment is being emitted for it — enable active tracing (and ensure trace context propagation, e.g. via the `_X_AMZN_TRACE_ID` env var /SDK, especially across an async invocation boundary) so the function's segment appears in the trace.

B. X-Ray has an undocumented hard limit of 3 functions per trace, and this is simply the 4th function in the chain.

C. The function's logs appearing in CloudWatch means it is automatically included in every X-Ray trace with no separate configuration needed.

D. X-Ray cannot trace Lambda functions at all; it can only trace EC2 and ECS workloads.

---

## Q45

Choose **TWO** correct actions for integrating Amazon GuardDuty and AWS Security Hub
so that a GuardDuty finding indicating a likely-compromised EC2 instance automatically triggers an
isolation workflow (e.g. quarantining the instance's security group) without a human needing to
manually notice and act on the finding in the console.

A. Enable GuardDuty findings to flow into Security Hub for centralized, aggregated visibility across multiple detection sources, and separately create an EventBridge rule matching the specific GuardDuty finding type/severity that targets a Lambda function (or Systems Manager Automation document) to perform the isolation action automatically.

B. Configure the EventBridge rule to match on the GuardDuty finding event pattern (e.g. detail-type 'GuardDuty Finding' with a severity threshold), since EventBridge is the mechanism that actually triggers the automated remediation target, not Security Hub aggregation alone.

C. Security Hub aggregation alone automatically performs remediation actions with no EventBridge rule or target needed.

D. GuardDuty findings cannot be consumed by EventBridge under any circumstances; only manual console review is supported.

E. Disable GuardDuty once Security Hub is enabled, since Security Hub replaces GuardDuty's detection capability entirely.

---

## Q46

A logistics company needs to track a package's state through a strict sequence:
`Created` → `PickedUp` → `InTransit` → `Delivered`, where each transition must call a different
Lambda function, some steps must wait for an external callback (e.g. a customs clearance system
taking up to 48 hours to respond) before continuing, and the whole workflow must be visually
auditable per package. What should orchestrate this instead of hand-rolled state-tracking logic
inside one monolithic Lambda function?

A. AWS Step Functions, using a state machine with a `Task` state per transition and the `waitForTaskToken` service integration pattern for the long-running customs callback step, pausing the execution (without consuming compute or holding a Lambda invocation open) until the external system calls back with the token.

B. A single long-running Lambda function that sleeps in a loop for up to 48 hours waiting for the customs system's callback.

C. Amazon SQS alone, with the package state stored as the queue's position in line.

D. CloudWatch Events scheduled to poll the customs system every second for 48 hours.

---

## Q47

A gaming leaderboard application uses ElastiCache for Redis to store real-time
player scores, sorted sets, and session data. As the player base grows past what a single Redis
node's memory can hold, the team needs to scale the dataset horizontally across multiple nodes
while keeping sub-millisecond read/write latency. What ElastiCache for Redis feature enables this?

A. Redis Cluster Mode Enabled — it shards data across multiple node groups (each with a primary and optional replicas), automatically distributing keys by hash slot so the total dataset can exceed a single node's memory capacity while each shard independently serves its portion of the keyspace.

B. Switching to ElastiCache for Memcached, since Memcached automatically shards data across nodes with zero configuration and Redis cannot do this at all.

C. Increasing the single node's instance type indefinitely, since Redis has no concept of sharding across multiple nodes.

D. Enabling Multi-AZ alone, with Cluster Mode left disabled, since Multi-AZ itself distributes the dataset across nodes for horizontal scale.

---

## Q48

A security team wants to know, for compliance audit purposes, exactly which S3
objects in a sensitive bucket were read (not just which API calls were made to the bucket overall)
over the last 90 days, including by which IAM principal and at what time, down to the individual
`GetObject` call. What CloudTrail feature is required, since it is not enabled by default?

A. CloudTrail data events for S3 — unlike default management events, data events (which log object-level operations like `GetObject`/`PutObject`) must be explicitly enabled on a trail (often selectively, by bucket/prefix, since they can be high-volume) and record exactly this kind of per-object access detail.

B. CloudTrail management events alone, since they already log every individual `GetObject` call by default.

C. VPC Flow Logs, since they capture a log line for every S3 `GetObject` API call automatically.

D. S3 Versioning, which tracks who read each object version by default.

---

## Q49

A legacy application packaged as a container needs to run on AWS with the team
explicitly wanting to avoid managing any EC2 instances or Kubernetes control plane at all, while
still using standard ECS task definitions and standard Docker images with no Kubernetes-specific
tooling required. Which combination satisfies this?

A. Amazon ECS with the Fargate launch type — ECS's own (non-Kubernetes) task definition model, with Fargate eliminating any EC2 instance or control-plane management for the team.

B. Amazon EKS with the Fargate launch type, since EKS is required whenever Fargate is used.

C. Amazon ECS with the EC2 launch type, since Fargate requires at least one self-managed EC2 instance as a control node.

D. AWS Lambda, since Lambda supports standard ECS task definitions directly with no changes needed.

---

## Q50

A team's nightly batch Lambda function processes a large S3 export and currently runs
at 256 MB memory, taking 9 minutes and frequently coming close to timing out. Profiling shows the
function is almost entirely CPU-bound (JSON parsing and transformation of a large payload) with
very little I/O wait. The team wants to both reduce the risk of timeout and shorten the nightly
run window. What is the most directly applicable lever, given this specific profile?

A. Increase the function's memory allocation significantly — since Lambda allocates CPU proportionally to configured memory, and this workload is CPU-bound, more memory should meaningfully reduce wall-clock duration for the CPU-heavy parsing/transformation work, shrinking both the timeout risk and the run window.

B. Increase only the function's timeout setting to 15 minutes, with memory left at 256 MB, since timeout alone determines how much CPU the function gets.

C. Switch the trigger from a scheduled EventBridge rule to an S3 event notification, since trigger type determines available CPU.

D. Reduce the function's memory further to 128 MB to save cost, since cost should always be minimized regardless of workload profile.

---

## Q51

A nightly batch job needs to invoke a Lambda function at exactly 2:00 AM UTC every
day to generate a sales report, with no manual trigger and no EC2 instance to keep running just to
fire the job. What is the correct, fully managed way to schedule this?

A. An Amazon EventBridge scheduled rule using a cron expression (e.g. `cron(0 2 * * ? *)`) targeting the Lambda function directly — EventBridge invokes the function on the defined schedule with no infrastructure to manage.

B. A `while True: time.sleep(86400)` loop inside the Lambda function itself, checking the current time on every wakeup.

C. An EC2 instance left running 24/7 with a local cron job that calls the Lambda function's URL once a day.

D. API Gateway configured with a `rate()` limit of one request per day, since usage plans substitute for schedules.

---

## Q52

A mobile app uses a Cognito identity pool to let unauthenticated ("guest") users browse
a public product catalog stored in DynamoDB, without requiring sign-in, while authenticated
(signed-in) users should additionally be able to write to their own cart data. What is the correct,
least-privilege way to configure the two Cognito identity pool roles for this?

A. Configure the unauthenticated role with read-only IAM permissions scoped to just the public catalog table/items, and configure the authenticated role with the catalog read permissions **plus** write permissions scoped (e.g. via `dynamodb:LeadingKeys` conditions) to only that specific signed-in user's own cart items.

B. Grant both the unauthenticated and authenticated roles full `dynamodb:*` access on every table, since simplicity outweighs scoping for a mobile app.

C. Use a single shared role for both unauthenticated and authenticated users, since Cognito identity pools only support one role total.

D. Give the unauthenticated role write access to the cart table as well, since guests might want to add items before signing in later.

---

## Q53

A team wants AI assistance reviewing every pull request's code changes for common
issues (missing error handling, potential null dereferences, inconsistent naming) before a human
reviewer looks at it, to surface likely problems faster — but they explicitly do not want AI review
comments to auto-merge a PR or override a human reviewer's final approval decision. How should this
be integrated into the PR workflow, consistent with the documented AI-assisted development pattern?

A. Add the AI review as an automated comment/check on the PR (e.g. triggered by the same CI event that runs tests) that surfaces suggestions for the human reviewer's attention, while keeping required human approval as the actual merge gate — AI augments and accelerates the review, but a human still makes the final call.

B. Configure the AI review tool to automatically approve and merge any PR it doesn't flag issues on, removing human reviewers from the loop entirely.

C. Disable human code review entirely now that an AI reviewer exists, since the AI covers everything a human would have checked.

D. Have the AI review silently rewrite the PR's code to fix issues it finds, merging its own changes without the original author's or reviewer's involvement.

---

## Q54

A team wants to know, across their entire multi-service application, the actual
dollar cost attributable specifically to one feature ("video transcoding") that spans Lambda, S3,
and a Step Functions workflow, so they can decide whether to keep or deprecate the feature based on
its real infrastructure cost — not the application's total bill, which also includes unrelated
features. What AWS capability enables this kind of feature-level cost attribution?

A. Cost allocation tags — apply a consistent tag (e.g. `feature: video-transcoding`) across every resource (Lambda functions, S3 buckets/prefixes where supported, Step Functions state machines) involved in that feature, activate the tag as a cost allocation tag in Billing, then filter Cost Explorer reports by that tag to see attributed spend.

B. AWS Budgets alone, since budgets automatically know which feature each resource belongs to without any tagging.

C. There is no way to attribute cost to a specific feature spanning multiple resource types; AWS only reports total account-level spend.

D. Amazon CloudWatch Logs, since log volume is a direct, built-in substitute for dollar cost with no further configuration.

---

## Q55

A `Products` table has partition key `productId`. A new GSI is added with partition
key `category` to support "list products in a category" queries. The application only ever needs
`productId`, `name`, and `price` from that query — never the other 15 attributes on each product
item. Which GSI projection type minimizes the GSI's storage cost and write amplification while
still satisfying the query?

A. `INCLUDE`, projecting only `productId`, `name`, and `price` (plus the table's keys, which are always projected) into the index — storing exactly the attributes the query needs and nothing more.

B. `ALL`, projecting every attribute from the base table into the index, since more projected data always performs better regardless of what's actually queried.

C. `KEYS_ONLY`, since the application can always make a second `GetItem` call per result if needed.

D. GSIs cannot have a restricted projection; they always copy every attribute from the base table.

---

## Q56

A team's Bedrock-based document-summarization feature processes internal company
documents, some containing confidential business data. The compliance team requires written
assurance that none of this data will be used by the underlying foundation model provider to
improve or train their models. What Task 2.3 control addresses this specific concern?

A. Confirm Bedrock's documented data handling posture: Amazon Bedrock does not use customer input/output data sent through the Bedrock API to train the underlying foundation models, and does not share that data with the model providers for their own model improvement — this is a standing data-handling guarantee of the Bedrock service itself, not a per-request opt-out the application needs to configure.

B. Manually strip every document of sensitive content before calling Bedrock, since Bedrock always uses all submitted data to retrain all available models by default.

C. Switch to a self-managed, on-premises ML model, since no AWS AI service can ever guarantee data isn't used for training.

D. Encrypt the documents with an S3 bucket key before sending them to Bedrock's `InvokeModel` API, which prevents the model from using the plaintext content for training.

---

## Q57

A document archive stores millions of files in S3. Compliance requires keeping files
accessible for 90 days at standard retrieval speed, then moving them to much cheaper archival
storage for 9 more months (still occasionally retrievable within hours if needed), and permanently
deleting them after one year total — all without any application code managing the transitions.
What should be configured?

A. An S3 Lifecycle configuration with a transition rule moving objects to S3 Glacier Flexible Retrieval (or Glacier Instant Retrieval) at 90 days, and an expiration rule deleting objects at 365 days — S3 applies both actions automatically with no application involvement.

B. A Lambda function scheduled via EventBridge to run daily, listing every object in the bucket and manually calling `CopyObject`/`DeleteObject` based on each object's age.

C. S3 Versioning alone, since old versions are automatically deleted after 90 days by default.

D. Store the retention rules in a DynamoDB table and have every application read request check the table before deciding whether to serve the file.

---

## Q58

A team's Lambda functions are currently packaged with every dependency bundled
directly into each function's own deployment .zip, even though five functions share the exact same
50 MB set of third-party libraries. Deployment packages are slow to upload and the team wants to
both shrink each function's own package and have a single place to patch a shared library version
across all five functions at once, without switching to container images. What should they do?

A. Extract the shared dependencies into a Lambda layer, publish it once, and attach that layer version to all five functions — each function's own deployment package then only contains its unique handler code, and bumping the shared library means publishing one new layer version instead of re-uploading five full packages.

B. Duplicate the 50 MB of dependencies into each of the five functions' packages, since Lambda has no mechanism for sharing code across functions without switching to containers.

C. Zip all five functions together into a single deployment package with one shared `handler.py` that dispatches by Lambda context, to avoid the problem entirely.

D. Store the dependencies in an S3 bucket and require each function to re-download and reinstall all 50 MB from S3 on every single invocation.

---

## Q59

A team notices a Lambda function occasionally fails with a connection error against
an RDS instance, correlating with times when the RDS instance's `DatabaseConnections` CloudWatch
metric is near its maximum, caused by Lambda scaling out to many concurrent executions each opening
its own new database connection. The team wants to reduce the number of simultaneous database
connections without artificially capping Lambda's ability to scale for the rest of its workload.
What should they add?

A. Amazon RDS Proxy — it sits between the Lambda functions and the RDS instance, pooling and multiplexing a smaller number of actual database connections across many concurrent Lambda executions, so a scaling burst doesn't translate 1:1 into a burst of new database connections.

B. Set the Lambda function's reserved concurrency to 1, so only one invocation can ever run at a time across the entire application.

C. Increase the RDS instance's storage size, since more storage directly increases the maximum connection limit.

D. Disable connection reuse entirely in the Lambda handler, opening and closing a brand-new connection on every single invocation.

---

## Q60

A Bedrock-powered customer support chatbot logs each conversation turn (including the
full user message and the model's full response) to CloudWatch Logs for debugging purposes. A
security review flags this under Task 2.3's "sensitive data in logs from AI service calls" item,
since users sometimes paste account numbers or other PII into their chat messages. What is the
correct remediation?

A. Apply redaction/masking to the logged conversation content before it's written (e.g. using a log-processing step or Macie-style pattern detection to strip/mask PII patterns like account numbers), and restrict access to any logs that might still retain sensitive context via IAM and log group access controls — treating AI service call logs with the same sensitive-data handling rigor as any other application log that might contain PII.

B. Stop logging entirely, since any logging of AI interactions is inherently non-compliant and cannot be fixed.

C. Log the conversations in full, unredacted, since CloudWatch Logs is itself a PII-safe, specially exempted storage location.

D. Only redact the model's responses, since user input can never contain sensitive data.

---

## Q61

A real-time chat application needs clients to maintain a persistent, bidirectional
connection to the backend so either side can push messages at any time, with each connected
client's unique connection identifier stored so the backend can target a message to a specific
user later. Which API Gateway capability, combined with what storage, implements this?

A. An API Gateway WebSocket API with `$connect`/`$disconnect`/custom routes, storing each client's `connectionId` (available in the `$connect` route's Lambda event) in a DynamoDB table, then using the API Gateway Management API's `PostToConnection` call to push a message to a specific stored `connectionId` later.

B. An API Gateway REST API, since REST APIs natively support persistent bidirectional connections with no additional configuration.

C. Amazon SNS alone, since every SNS subscriber maintains a reusable open HTTP connection to the publisher.

D. Polling a REST endpoint every 200 milliseconds from each client to check for new messages.

---

## Q62

A CDK app currently defines all resources for `dev`, `staging`, and `prod`
environments in a single, monolithic stack with hardcoded environment-specific values scattered
throughout the code via `if` statements, making it error-prone to tell which environment a given
`cdk deploy` will actually target. The team wants a cleaner, safer pattern for managing multiple,
clearly distinguished environment deployments from the same CDK codebase. What should they use?

A. Separate CDK Stack instances per environment (e.g. a reusable `Stack` subclass instantiated once per environment with environment-specific props/context), each explicitly deployed with its own `env` (account/Region) and distinguishing context values (e.g. via `cdk deploy MyStack-dev` or `cdk deploy MyStack-prod`), rather than branching logic inside one shared stack based on ambient state.

B. Keep the single monolithic stack exactly as-is, since CDK has no concept of multiple stacks in one app.

C. Hardcode the production AWS account ID as the default for every deploy, and require developers to remember to override it manually for dev/staging every single time.

D. Deploy to production only by directly editing resources in the CloudFormation console after a dev deployment, bypassing CDK for prod entirely.

---

## Q63

A team wants to validate, before committing to a production Lambda memory setting,
whether a specific function's duration (and therefore cost) actually improves meaningfully between
512 MB, 1024 MB, and 1536 MB for their real production-like payload — rather than guessing based on
general advice about CPU-bound workloads. What approach gives them concrete, workload-specific
evidence rather than a generic rule of thumb?

A. Run the function at each candidate memory setting against a representative test payload (e.g. using the open-source AWS Lambda Power Tuning tool, which automates invoking the function at multiple memory values and reports actual duration/cost for each) and compare the real measured results for this specific function.

B. Pick 1024 MB because it's a commonly cited 'good default' in blog posts, with no testing against this function's actual payload.

C. Assume duration scales exactly linearly with memory for every Lambda function, so testing more than one memory value is unnecessary.

D. Set memory to the account-wide maximum for every function without measuring, since more memory can never make a function slower.

---

## Q64

A company deploys a Bedrock Agent that can call two action groups: one that looks up
order status (read-only) and one that issues refunds (a real financial action). A penetration test
successfully tricks the agent, via a crafted user message, into attempting to call the refund
action group when the user only asked a status question. The refund call fails only because the
agent's own execution role happens to lack `lambda:InvokeFunction` on that specific action group's
function. What does this near-miss indicate about the deployment, and what is the correct fix,
per Task 2.3's "AI agent security" guidance?

A. The agent's IAM execution role is currently under-scoped in a way that accidentally saved the day, but the deployment should not rely on accidental permission gaps — explicitly and deliberately scope the agent's role to only the action groups a given use case legitimately needs (here, likely read-only order status only, with refunds handled through a separate, more tightly gated flow), and add Guardrails/input validation so manipulated prompts are also rejected before ever reaching the point of attempting an unauthorized tool call.

B. Nothing needs to change, since the call ultimately failed and no refund was issued.

C. Grant the agent's role full permissions on both action groups, since restricting permissions caused this near-miss and more permissions would have avoided the confusing error.

D. Disable IAM entirely for Bedrock Agents, since IAM evidently interfered with the agent's ability to do its job.

---

## Q65

Choose **TWO** correct statements about fanning out a single `UserSignedUp` event from
an SNS topic to three independent SQS queues (one per downstream microservice: welcome-email,
analytics, and fraud-check), where each microservice should process the event at its own pace
without affecting the others.

A. The SNS-to-multiple-SQS fan-out pattern decouples the publisher from the number and identity of consumers — the publishing service only needs to know about the topic, not about any specific downstream queue or service.

B. Because each microservice has its own SQS queue, one microservice falling behind or failing to process messages does not block or slow down the other two microservices' independent processing of their own queue.

C. SNS guarantees that a message delivered to one subscribed SQS queue is automatically removed from the other two queues once any one of them processes it.

D. This pattern requires the publisher to send three separate, individually addressed messages — one per downstream queue — since SNS cannot deliver one message to multiple queues.

E. Each SQS queue in this fan-out pattern can independently configure its own visibility timeout and DLQ redrive policy, appropriate to that specific microservice's processing characteristics.

---
