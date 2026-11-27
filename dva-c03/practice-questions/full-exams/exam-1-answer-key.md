# DVA-C03 Full Mock Exam 1 — Answer Key

Correct answer(s), full explanation, and why each distractor is wrong — cross-linked to the relevant service/domain docs. Matches the question numbering in the companion `*-questions.md` file exactly.

---

## Q1 — Domain 1

**Correct Answer: B**

**Explanation:** A `ConditionExpression` makes the write conditional server-side — DynamoDB evaluates it
atomically against the item's current state and rejects the update (`ConditionalCheckFailedException`)
if the condition isn't met, which is exactly how to implement optimistic locking / safe state
transitions without a separate read-then-write race window.

**Why the others are wrong:**
- A: `ProjectionExpression` only controls which attributes are returned by a read; it has no effect on write conditions.
- C: A `GetItem` followed by a separate `UpdateItem` is a classic check-then-act race — another process can change the item between the two calls, which is the exact bug described.
- D: `ConsistentRead` applies to read operations (`GetItem`/`Query`/`Scan`); `UpdateItem` is always strongly consistent on the item it writes, and this option doesn't exist as a parameter on `UpdateItem`.

**Study links:** [`services/database/dynamodb.md`](../../../../services/database/dynamodb.md)

---

## Q2 — Domain 2

**Correct Answer: B**

**Explanation:** IAM's policy evaluation logic is: explicit Deny always wins. Across every policy type
that applies to a request (identity-based, resource-based, permissions boundaries, SCPs), if any
statement explicitly denies the action, the request is denied — full stop — no matter how many
other statements allow it or how "specific" the Allow looks.

**Why the others are wrong:**
- A: Specificity of the resource ARN does not create precedence over an explicit Deny; IAM has no such tie-breaking rule — Deny always wins regardless.
- C: IAM evaluation doesn't depend on attachment order/recency; all applicable policies are evaluated together and any explicit Deny wins.
- D: There is no "last attached wins" rule in IAM policy evaluation — this misunderstands the deny-by-default, explicit-deny-always-wins model entirely.

**Study links:** [`services/security-identity-compliance/iam.md`](../../../../services/security-identity-compliance/iam.md)

---

## Q3 — Domain 3

**Correct Answer: A**

**Explanation:** `sam validate` is exactly Task 3.2's "validate infrastructure templates before deploying
them" capability — it statically checks the SAM/CloudFormation template for syntax and structural
correctness, catching broken references and invalid properties in a CI pipeline stage with zero AWS
side effects, before any real deployment attempt.

**Why the others are wrong:**
- B: This flag doesn't exist on the SAM CLI; `sam validate` is the actual documented command for this.
- C: Manual review doesn't scale, isn't automatable in CI/CD, and misses the exact kind of structural/reference errors a validator catches reliably.
- D: Deleting a stack has nothing to do with validating a template's correctness before deployment — this is an unrelated and destructive operation.

**Study links:** [`services/developer-tools/sam.md`](../../../../services/developer-tools/sam.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q4 — Domain 4

**Correct Answer: A**

**Explanation:** CloudWatch Logs Insights is exactly the Task 4.1 tool for this: a purpose-built query
language that searches across every log stream in a log group at once, lets you filter on the
exact exception string, extract structured fields, and aggregate by time — turning "search
thousands of scattered log lines by hand" into a single query that returns the table name and the
exact time the error pattern started.

**Why the others are wrong:**
- B: Manually downloading and grepping every stream doesn't scale across dozens of streams and is far slower than a single Logs Insights query over the whole log group.
- C: Memory affects compute speed, not log readability, and has nothing to do with locating the error's source or timing.
- D: Disabling logging removes visibility into the very problem being investigated — the opposite of useful troubleshooting.

**Study links:** [`services/management-governance/cloudwatch.md`](../../../../services/management-governance/cloudwatch.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q5 — Domain 1

**Correct Answer: B**

**Explanation:** Classic (shared-throughput) consumers all compete for the same 2 MB/sec-per-shard /
5 transactions-per-second read budget, so adding consumers increases contention. Enhanced fan-out
gives each registered consumer a dedicated 2 MB/sec pipe per shard pushed over HTTP/2, which is
precisely the feature for "multiple independent consumers reading the same stream without
throttling each other."

**Why the others are wrong:**
- A: More shards raises overall throughput but doesn't fix shared-throughput contention between consumers on the shards that already exist; it also costs more and doesn't address the root cause.
- C: Polling a stream on a schedule to feed a queue adds latency and defeats near-real-time consumption; it doesn't solve the throttling problem for the other two readers either.
- D: Merging unrelated workloads (fraud detection vs. batch analytics) into one function violates separation of concerns and doesn't change the number of GetRecords calls needed overall.

**Study links:** [`services/analytics/kinesis-family.md`](../../../../services/analytics/kinesis-family.md)

---

## Q6 — Domain 2

**Correct Answer: A**

**Explanation:** The `sts:ExternalId` condition is AWS's documented mitigation for the confused-deputy
problem in third-party cross-account role assumption: the vendor hands each customer a unique,
pre-agreed external ID, and the trust policy requires it to match on every `AssumeRole` call —
so even if a malicious third party convinces the vendor to call `AssumeRole` against a victim's
resource, it can't produce the victim's private external ID.

**Why the others are wrong:**
- B: Account-ID-only trust narrows who can attempt the assumption but doesn't address the confused-deputy scenario where the *vendor itself* is tricked into acting on behalf of the wrong customer context — external ID is the documented fix for that specific pattern.
- C: A wildcard resource/account trust defeats the purpose of restricting role assumption at all and is the opposite of a secure cross-account pattern.
- D: MFA strengthens the customer's own authentication but does not solve the confused-deputy problem the external ID condition specifically addresses.

**Study links:** [`services/security-identity-compliance/sts-and-role-switching.md`](../../../../services/security-identity-compliance/sts-and-role-switching.md)

---

## Q7 — Domain 3

**Correct Answer: A**

**Explanation:** Lambda supports two packaging formats: .zip (capped at 250 MB unzipped) and container
images (up to 10 GB, built with the Lambda runtime interface client, stored in ECR) — exactly the
documented Task 3.1 escape hatch for large dependencies like ML libraries that blow past the zip
size limit, with no change needed to the invocation model or trigger configuration.

**Why the others are wrong:**
- B: Arbitrarily splitting a function by which part of a library it imports usually isn't feasible (the library is typically used as a whole) and doesn't address the actual constraint cleanly.
- C: The 250 MB unzipped limit for .zip deployment packages is a hard platform limit, not a per-function quota that can be raised via a support ticket.
- D: Downloading a large library on every cold start adds significant latency and reliability risk compared to simply using the container image packaging format built for this case.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`services/containers/ecr.md`](../../../../services/containers/ecr.md)

---

## Q8 — Domain 4

**Correct Answer: A**

**Explanation:** X-Ray's value is exactly per-segment timing breakdown within a single request's trace:
with the SDK instrumenting outbound calls (e.g. wrapping the database client), the trace shows a
subsegment for the RDS call with its own duration, separate from Lambda init/handler overhead — so
comparing subsegment durations directly localizes the slow part of the chain (Task 4.1: "use X-Ray
to find *where* in a distributed call chain it happened").

**Why the others are wrong:**
- B: Guessing based on position in the chain isn't root-cause analysis and is frequently wrong — the entry point is rarely the actual source of backend latency.
- C: This is factually wrong — X-Ray's entire purpose is per-call, per-segment timing within a distributed request, which plain log timestamps don't cleanly reconstruct across services.
- D: Testing from a different Region doesn't diagnose which component in *this* specific request's call chain is slow; it's an unrelated variable.

**Study links:** [`domain-4-troubleshooting-optimization/observability-instrumentation.md`](../../domain-4-troubleshooting-optimization/observability-instrumentation.md), [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q9 — Domain 1

**Correct Answer: B**

**Explanation:** In Lambda proxy integration, the response contract requires `body` to already be a
serialized JSON string — API Gateway does not re-serialize it, it forwards the string as-is. A raw
dict there breaks the integration response contract and API Gateway surfaces that as a 500, even
though the Lambda invocation itself succeeded.

**Why the others are wrong:**
- A: `execute-api:Invoke` governs who can call the API Gateway endpoint (resource-policy / IAM auth), not the Lambda function's own execution role, and a permissions problem would surface as 403, not a 500 after a successful Lambda run.
- C: `statusCode` is expected as an integer in the proxy response shape; sending it as a string is not the documented contract and is not the bug here.
- D: CORS headers are optional and only matter for browser-origin requests; their absence does not cause a 500.

**Study links:** [`services/networking-content-delivery/api-gateway.md`](../../../../services/networking-content-delivery/api-gateway.md), [`domain-1-development/README.md`](../../domain-1-development/README.md)

---

## Q10 — Domain 2

**Correct Answer: B**

**Explanation:** User pools handle authentication (who the user is) and issue JWTs (ID/access/refresh
tokens) — they do not themselves grant AWS credentials. Identity pools are the piece that takes a
trusted identity (e.g. a user pool token) and calls STS to vend temporary, role-scoped IAM
credentials the client can use directly against AWS APIs like S3, which is exactly the
"authenticate, then get scoped temp AWS creds for direct S3 access" pattern.

**Why the others are wrong:**
- A: User pools issue identity/authentication tokens (JWTs), not IAM credentials — they are not interchangeable with identity pools for this purpose.
- C: Creating per-user IAM users is both an anti-pattern (IAM users aren't meant for this) and operationally unworkable at mobile-app scale with no automatic token exchange.
- D: A Lambda authorizer evaluates whether to allow an API Gateway request; it does not vend AWS credentials to a mobile client for direct S3 access.

**Study links:** [`services/security-identity-compliance/cognito.md`](../../../../services/security-identity-compliance/cognito.md)

---

## Q11 — Domain 3

**Correct Answer: A**

**Explanation:** CodeDeploy natively supports Lambda deployments with built-in canary (and linear)
deployment configurations that automatically shift alias traffic in defined percentage/time
increments, and — critically — can be wired to CloudWatch alarms so that if an alarm goes into
ALARM state during the shift, CodeDeploy automatically rolls the alias back to the prior version
with no manual intervention.

**Why the others are wrong:**
- B: Manual weight adjustment is exactly the error-prone, non-automated process CodeDeploy exists to replace, and provides no automatic rollback on alarm.
- C: Stage variables route to an alias but have no concept of gradual percentage-based traffic shifting over time or alarm-triggered rollback — that orchestration is CodeDeploy's job.
- D: Reserved concurrency caps how many concurrent executions a function can have; it does not implement weighted alias traffic shifting between two versions.

**Study links:** [`services/developer-tools/codedeploy.md`](../../../../services/developer-tools/codedeploy.md), [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q12 — Domain 4

**Correct Answer: A**

**Explanation:** CloudTrail is specifically the record of "who/what called which AWS API, from where, and
when" — exactly Task 4.1's "use CloudTrail to find who/what made an API call that caused it." A
`PutBucketPolicy` event in CloudTrail's event history includes the caller identity (user/role ARN),
source IP address, and timestamp needed to attribute and investigate the change.

**Why the others are wrong:**
- B: CloudWatch Logs doesn't automatically capture AWS management API calls on its own; CloudTrail is the service that records API activity, and can optionally also stream into CloudWatch Logs.
- C: AWS Config tracks *configuration state changes* and can show a resource's configuration timeline/drift, but the authoritative record of *who* made the API call (identity, source IP) is CloudTrail's job, and Config itself relies on CloudTrail-recorded events for some of this context.
- D: S3 server access logs record *requests made to the bucket's objects* (data-plane access like GetObject), not management-plane IAM calls like changing the bucket policy — that's a CloudTrail management event.

**Study links:** [`services/management-governance/cloudtrail.md`](../../../../services/management-governance/cloudtrail.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q13 — Domain 1

**Correct Answer: B**

**Explanation:** Once a Lambda function is attached to a VPC, its network traffic — including calls to
AWS public-endpoint services like S3 — goes through that VPC's networking, not Lambda's own managed
network. If the private subnets have no NAT gateway and no S3 gateway VPC endpoint, outbound calls
to S3's public endpoint simply have no route and hang until timeout. Adding an S3 gateway endpoint
(free, and the standard pattern for this) or a NAT gateway restores connectivity.

**Why the others are wrong:**
- A: Lambda in a VPC can reach S3 fine — it just needs a route, via a gateway endpoint or NAT — this isn't a hard limitation.
- C: A missing IAM permission produces an `AccessDenied` exception immediately, not a network-level timeout; the symptom described is networking, not authorization.
- D: Raising the timeout doesn't create a network path; the call will still never succeed because there is nowhere for the request to route.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`services/networking-content-delivery/vpc.md`](../../../../services/networking-content-delivery/vpc.md)

---

## Q14 — Domain 2

**Correct Answer: A**

**Explanation:** IAM Identity Center is purpose-built for exactly this: centralized workforce identity,
single sign-on across the AWS Organization, and **permission sets** — a template of permissions
defined once and automatically provisioned as an IAM role in every account a user/group is assigned
to — avoiding hand-maintained, duplicated IAM roles per account.

**Why the others are wrong:**
- B: Duplicating IAM users per account is exactly the manual, unscalable, insecure pattern IAM Identity Center exists to replace.
- C: A single centrally-assumed role doesn't provide differentiated, per-account permission sets (Developer vs. ReadOnly) the way Identity Center's permission sets do, and doesn't match "consistent but centrally defined" per-account access.
- D: Cognito user pools are for application end-user authentication, not workforce access to the AWS Management Console/CLI across an Organization.

**Study links:** [`services/security-identity-compliance/iam-identity-center-sso.md`](../../../../services/security-identity-compliance/iam-identity-center-sso.md)

---

## Q15 — Domain 1

**Correct Answer: A**

**Explanation:** Lambda layers are built exactly for this: a .zip of shared code/dependencies, versioned
independently, extracted to `/opt` in every attached function's execution environment. Updating the
shared library means publishing a new layer version and pointing functions at it — no need to touch
or re-upload each function's own deployment package.

**Why the others are wrong:**
- B: Aliases point at specific **versions of a single function**, they don't share code between different functions.
- C: Collapsing five functions into one trades a packaging problem for a different design smell (loses independent scaling/permissions/memory tuning per responsibility) and isn't what the services offer natively for shared dependencies.
- D: EFS can be used to share large assets, but copying dependencies in at every cold start adds latency and is not the purpose-built mechanism — layers solve this more directly and without a filesystem round-trip.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`domain-1-development/README.md`](../../domain-1-development/README.md)

---

## Q16 — Domain 3

**Correct Answer: A**

**Explanation:** CodePipeline's manual approval action is designed for precisely this: it pauses pipeline
execution at a defined point and waits for an explicit approve/reject decision from an authorized
principal before continuing — keeping build/test automated end-to-end while inserting exactly one
deliberate human gate before production, directly addressing Task 3.3's "AI-assisted deployment
approvals / human-in-the-loop gating" pattern in its general (human-gated) form.

**Why the others are wrong:**
- B: Removing pipeline automation for production entirely throws away CI/CD's reliability and auditability benefits — overcorrecting far past what's needed.
- C: A fixed sleep delay doesn't require or capture any human decision — it only adds latency, not a gate.
- D: Commit signing verifies author identity, not that a human reviewed and approved this specific release going to production.

**Study links:** [`services/developer-tools/codepipeline.md`](../../../../services/developer-tools/codepipeline.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q17 — Domain 4

**Correct Answer: A**

**Explanation:** These are two distinct, well-documented Task 4.3 levers: Provisioned Concurrency
pre-initializes a pool of execution environments so requests routed to that pool skip the cold
init phase entirely, while raising memory (which scales CPU/network proportionally in Lambda)
directly speeds up CPU-bound work like parsing — including the init-phase work itself, independent
of whether an invocation is cold or warm.

**Why the others are wrong:**
- B: Timeout only bounds how long an invocation is allowed to run before being killed; it has no effect on how fast initialization or execution actually happens.
- C: Cold starts happen with any trigger type when a new execution environment must be created; switching triggers doesn't eliminate the underlying initialization cost.
- D: Reserved concurrency guarantees/caps available concurrency; it does not change the speed of any individual invocation's cold start.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q18 — Domain 2

**Correct Answer: A**

**Explanation:** SCPs are an Organizations-level guardrail that define the *maximum available*
permissions for every principal in affected accounts — they don't grant permissions, they cap them,
and they apply account-wide regardless of what IAM policies (including AdministratorAccess) say
inside the account. This is the only one of these options that actually constrains an
account admin, since IAM policies within an account can be bypassed/changed by that same admin.

**Why the others are wrong:**
- B: An IAM policy can be edited, deleted, or bypassed by an account administrator with sufficient permissions — it cannot act as a guardrail that binds the administrator themselves.
- C: A CloudWatch alarm is detective (notifies after the fact), not preventive — it doesn't stop the action from happening.
- D: Removing a managed policy doesn't prevent an administrator from attaching permissions again, and doesn't scale as a durable, org-wide control the way an SCP does.

**Study links:** [`services/security-identity-compliance/organizations-and-scps.md`](../../../../services/security-identity-compliance/organizations-and-scps.md)

---

## Q19 — Domain 1

**Correct Answer: A**

**Explanation:** Reserved concurrency carves out (and caps) a specific slice of the account's total
concurrency pool exclusively for one function — both guaranteeing it a minimum available concurrency
and capping its maximum, so a burst elsewhere in the account can no longer starve it. This is the
standard Task 1.2 control for isolating one function's concurrency from the rest of the account.

**Why the others are wrong:**
- B: Faster execution reduces concurrent invocations somewhat but doesn't guarantee capacity is available at the moment of the flash-sale spike — it's a mitigation, not a reservation.
- C: Timeout controls how long a single invocation is allowed to run; it has no relationship to whether a request is throttled due to concurrency limits.
- D: Removing the batch function's trigger is an operational sledgehammer that stops unrelated legitimate work rather than correctly partitioning concurrency.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q20 — Domain 3

**Correct Answer: A**

**Explanation:** ECR lifecycle policies are exactly the automated-cleanup mechanism for this: JSON rule
sets that match images by tag status/prefix/age/count and automatically expire ones that match,
letting the team both prune untagged build noise and preserve a bounded, named set of tagged
release images with zero manual intervention.

**Why the others are wrong:**
- B: Repository visibility (private vs. public) is unrelated to image retention/expiration; there is no such automatic 30-day purge tied to privacy setting.
- C: A weekly manual process is exactly the toil the question says to avoid and doesn't scale or stay consistent.
- D: Disabling tagging would break the ability to reference specific release versions in deployments, which is the opposite of what's needed.

**Study links:** [`services/containers/ecr.md`](../../../../services/containers/ecr.md)

---

## Q21 — Domain 4

**Correct Answer: A**

**Explanation:** Anomaly detection is exactly built for this: CloudWatch learns a metric's normal
seasonal/trend pattern over time and computes an expected range band, and an anomaly-detection
alarm triggers only on deviation from that adaptive band — directly solving "normal daily spikes
shouldn't alert, but a genuinely abnormal spike should."

**Why the others are wrong:**
- B: Setting the threshold to the historical max defeats the point of alerting at all — it would also fail to catch real problems below that ceiling.
- C: Removing the alarm loses all detection capability rather than fixing the false-positive rate — not a real fix.
- D: Dashboards are visualization only; they don't evaluate conditions or trigger notifications on their own without an underlying alarm.

**Study links:** [`services/management-governance/cloudwatch.md`](../../../../services/management-governance/cloudwatch.md)

---

## Q22 — Domain 2

**Correct Answer: A**

**Explanation:** ACM-issued public certificates used with integrated services (ALB, CloudFront, API
Gateway) are automatically renewed by ACM before expiry as long as the domain validation (DNS
CNAME, typically) stays valid — this is the whole point of using ACM instead of manually managed
certificates, and it directly fixes the "don't want to manually re-upload yearly" requirement.

**Why the others are wrong:**
- B: Self-signed certs trigger browser trust warnings for a public API and still require fully manual renewal — the opposite of the automated goal.
- C: KMS manages encryption keys, not TLS/X.509 certificate issuance for load balancer listeners — this is the wrong service.
- D: ALB HTTPS listeners attach to a certificate (via ACM or IAM), they don't fetch raw private key material from Secrets Manager per request — this isn't how ALB TLS termination works.

**Study links:** [`services/security-identity-compliance/acm-and-private-ca.md`](../../../../services/security-identity-compliance/acm-and-private-ca.md)

---

## Q23 — Domain 1

**Correct Answer: A**

**Explanation:** SQS standard queues are at-least-once delivery: if the consumer doesn't delete a message
before its visibility timeout expires, the message becomes visible again and can be redelivered —
including to a second concurrent Lambda invocation. AWS's documented guidance is to set the queue's
visibility timeout to at least 6x the function's timeout for event-source-mapped Lambda consumers,
so an in-flight batch isn't prematurely considered abandoned.

**Why the others are wrong:**
- B: SNS doesn't buffer/queue messages for pull-based polling the way SQS does, and doesn't resolve a visibility-timeout/duplicate-processing problem — it's not a drop-in substitute here.
- C: A batch size of 1 reduces blast radius per message but does not fix an undersized visibility timeout; the same message can still be redelivered if processing exceeds the timeout.
- D: Lambda's SQS event source mapping manages deletion automatically on successful return; manually deleting mid-handler bypasses the error-handling/retry contract and isn't the documented fix for this symptom.

**Study links:** [`services/application-integration/sqs.md`](../../../../services/application-integration/sqs.md), [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q24 — Domain 1

**Correct Answer: A**

**Explanation:** Multipart upload is the documented mechanism for large objects: it splits the object
into independently-uploaded parts (AWS recommends considering it above 100 MB), lets failed parts
be retried individually instead of restarting the whole transfer, and can upload parts in parallel
for throughput — directly fixing reliability on unstable connections for large files.

**Why the others are wrong:**
- B: Storage class governs retrieval cost/latency after the object exists; it has no bearing on upload reliability or size handling.
- C: A longer URL expiration affects how long the link is valid, not how resiliently the bytes are transferred — it doesn't fix partial-failure retries.
- D: Transfer Acceleration speeds up transfer over Amazon's edge network but doesn't by itself add the resumable, per-part retry behavior that fixes reliability for very large single-stream uploads; it pairs with multipart upload rather than replacing it.

**Study links:** [`services/storage/s3.md`](../../../../services/storage/s3.md)

---

## Q25 — Domain 2

**Correct Answer: A**

**Explanation:** Accessing an SSE-KMS encrypted object requires permission on *two* independent resources:
the S3 object/bucket (via bucket policy or IAM) **and** the KMS key used to encrypt it (via the
key's resource-based key policy, or a grant) — cross-account access needs the external principal
explicitly allowed in the key policy too, since IAM permissions in Account A alone cannot reach
into Account B's KMS key policy.

**Why the others are wrong:**
- B: S3 permissions and KMS key permissions are enforced independently; having one does not imply the other, which is exactly why this `AccessDenied` is happening despite the bucket policy already allowing `GetObject`.
- C: Creating a separate key in Account A would encrypt different data under a different key — it does nothing to grant decrypt rights to the *existing* objects encrypted under Account B's key.
- D: Re-uploading unencrypted defeats the encryption-at-rest requirement and isn't a reasonable or correct fix for a permissions problem.

**Study links:** [`services/security-identity-compliance/kms.md`](../../../../services/security-identity-compliance/kms.md), [`services/storage/s3.md`](../../../../services/storage/s3.md)

---

## Q26 — Domain 3

**Correct Answer: A**

**Explanation:** CodeDeploy's ECS blue/green deployment type is built exactly for this: it stands up a
parallel "green" task set, validates it (optionally with lifecycle hooks running smoke tests),
then shifts the ALB listener's traffic from blue to green — with the old blue task set kept running
briefly so an instant rollback (rerouting the listener back) is possible if something's wrong.

**Why the others are wrong:**
- B: A forced new rolling deployment replaces tasks in place without a separate parallel environment to validate first or instantly roll back to — it's a different (rolling) strategy, not blue/green.
- C: Recreating the whole service from scratch causes downtime and discards the ability to keep the old version running for comparison/rollback — far more disruptive than necessary.
- D: Fargate fully supports ECS blue/green deployments via CodeDeploy; this statement is factually incorrect.

**Study links:** [`services/containers/ecs.md`](../../../../services/containers/ecs.md), [`services/developer-tools/codedeploy.md`](../../../../services/developer-tools/codedeploy.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q27 — Domain 4

**Correct Answer: A**

**Explanation:** EMF is specifically designed to solve this Task 4.2 "instrument code for observability"
problem: the application just logs a structured JSON line (which it was likely already doing
anyway), and CloudWatch Logs' EMF processing pipeline extracts the described metric straight into
CloudWatch Metrics — avoiding a dedicated synchronous `PutMetricData` call (and its added latency)
on the hot path entirely.

**Why the others are wrong:**
- B: A synchronous `PutMetricData` call on every invocation adds latency and an extra billed API call per invocation — exactly what the requirement says to avoid.
- C: Building a custom nightly ETL pipeline to backfill CloudWatch Metrics is far more complex and far less real-time than EMF's built-in log-to-metric extraction.
- D: Environment variables are static per deployment and are not a mechanism for emitting a time-series metric value at all.

**Study links:** [`services/management-governance/cloudwatch.md`](../../../../services/management-governance/cloudwatch.md), [`domain-4-troubleshooting-optimization/observability-instrumentation.md`](../../domain-4-troubleshooting-optimization/observability-instrumentation.md)

---

## Q28 — Domain 1

**Correct Answer: A**

**Explanation:** A GSI has its own partition/sort key independent of the base table's key schema, can be
added at any time to an existing table, and is queried like a separate table-shaped index — exactly
the tool for adding a new query pattern after the table is already live.

**Why the others are wrong:**
- B: A table's primary key schema is fixed at creation time and can't be changed afterward without recreating the table, which would break every existing access pattern.
- C: An LSI must share the base table's partition key and can only be defined **at table creation time** — it cannot use a different partition key (`customerId` here) and cannot be added to an existing table.
- D: A `Scan` with a filter reads the entire table and discards non-matching items, which is exactly the inefficient full-table-scan behavior the question is trying to avoid.

**Study links:** [`services/database/dynamodb.md`](../../../../services/database/dynamodb.md)

---

## Q29 — Domain 2

**Correct Answer: A**

**Explanation:** Secrets Manager's native strength over Parameter Store is built-in automatic *rotation*
with direct database integrations (RDS, Aurora, Redshift, DocumentDB) — exactly what's needed for a
rotating database password with zero app changes. Parameter Store is the right (and cheaper) choice
for ordinary, non-rotating configuration like a feature flag, which doesn't need rotation at all.

**Why the others are wrong:**
- B: Parameter Store does not provide the managed automatic-rotation workflow Secrets Manager has for databases; using it for a rotating RDS credential loses that capability, and using Secrets Manager for a simple flag adds unnecessary cost/complexity.
- C: Hardcoded environment variables require a function re-deploy on every rotation (causing exactly the "manual update" and downtime risk the requirement rules out) and leaks the secret into deployment artifacts/console views.
- D: Storing unencrypted secrets in a general-purpose data store with no secret-specific access logging or rotation is not an acceptable secrets-management pattern.

**Study links:** [`services/security-identity-compliance/secrets-manager-vs-parameter-store.md`](../../../../services/security-identity-compliance/secrets-manager-vs-parameter-store.md)

---

## Q30 — Domain 3

**Correct Answer: A**

**Explanation:** Change sets are exactly CloudFormation's preview mechanism: they compute and display the
full diff of what an update would do — including whether each change is an in-place update or a
disruptive replacement (critical to know before touching a stateful resource like an RDS instance)
— without making any actual changes until the change set is explicitly executed.

**Why the others are wrong:**
- B: Deleting and recreating a stack destroys the existing RDS instance and its data unless explicitly protected — a destructive, unacceptable approach to a routine engine version update.
- C: Manually drifting a resource outside CloudFormation breaks the stack's state tracking ("drift") and isn't a preview mechanism at all — it actively creates an inconsistency to later resolve.
- D: Checking after the fact is not a preview — the question asks to see the impact *before* applying the change, which only a change set provides.

**Study links:** [`services/management-governance/cloudformation.md`](../../../../services/management-governance/cloudformation.md)

---

## Q31 — Domain 4

**Correct Answer: A**

**Explanation:** AWS Config is purpose-built for exactly this Task 4.2 "audit config drift" need:
managed or custom Config rules continuously evaluate live resource configuration (not periodic
exports) against a defined compliance rule and surface non-compliant resources essentially as soon
as they drift, which is the documented mechanism for ongoing automated compliance detection (and
can be paired with Systems Manager Automation to auto-remediate).

**Why the others are wrong:**
- B: A stack policy documented in a README is not an enforced, automated detection mechanism — it's just documentation that's easy to ignore.
- C: A quarterly manual script is periodic, not continuous, and reintroduces the exact 'weeks before anyone noticed' lag the requirement says to eliminate.
- D: S3 Object Lambda intercepts S3 GET requests to transform object data; it has no relationship to EBS volume configuration at all.

**Study links:** [`services/management-governance/aws-config.md`](../../../../services/management-governance/aws-config.md)

---

## Q32 — Domain 1

**Correct Answer: A and B**

**Explanation:** Aurora Replicas (up to 15) share the same underlying storage volume as the writer within
a Region, so they serve reads with minimal lag and can be promoted quickly if the writer fails.
Aurora Global Database extends this across Regions by replicating storage-layer changes to
secondary Region clusters with typically sub-second lag, specifically built for low-RPO/RTO
disaster recovery — the correct combination for "read scaling in-Region + DR across Regions."

**Why the others are wrong:**
- C: Aurora's native cross-Region replication (Global Database) is a managed feature, not something that requires the developer to hand-build binlog replication.
- D: A regular Aurora cluster has exactly one writer at a time (plus up to 15 read replicas); true multi-writer is a separate, opt-in Aurora capability, not the default replica behavior.
- E: Aurora Global Database is supported for both Aurora MySQL-compatible and Aurora PostgreSQL-compatible editions, not PostgreSQL only.

**Study links:** [`services/database/aurora.md`](../../../../services/database/aurora.md), [`services/database/database-selection-guide.md`](../../../../services/database/database-selection-guide.md)

---

## Q33 — Domain 2

**Correct Answer: A**

**Explanation:** Macie is purpose-built for exactly this: automated discovery and classification of
sensitive data in S3 (PII, credentials, financial data) using managed data identifiers, with the
ability to run one-off or recurring scheduled discovery jobs across one or many buckets — no custom
detection logic required.

**Why the others are wrong:**
- B: GuardDuty is a threat-detection service analyzing account/network/API activity (e.g. CloudTrail, VPC Flow Logs, DNS logs) for malicious behavior — it does not scan S3 object content for PII; that's Macie's job.
- C: AWS Config evaluates resource *configuration* against rules (e.g. "is encryption enabled"); it does not inspect the content of objects stored inside them.
- D: Trusted Advisor gives best-practice checks (cost, security config, fault tolerance) — it does not perform content-level PII discovery inside objects.

**Study links:** [`services/security-identity-compliance/macie.md`](../../../../services/security-identity-compliance/macie.md)

---

## Q34 — Domain 1

**Correct Answer: A**

**Explanation:** Stage variables are precisely this feature in API Gateway: a key/value pair scoped to a
stage that can be referenced inside the integration configuration (e.g. in the Lambda ARN used for
the integration), letting one API definition route each stage to a different backend — the
documented Task 1.1 pattern for dev/test/prod promotion without duplicating the API.

**Why the others are wrong:**
- B: Separate APIs per environment duplicates the whole definition and configuration, exactly what the question says to avoid, and loses the single-source-of-truth benefit of stages.
- C: Hardcoding ARNs defeats the purpose of having multiple stages share one integration, and resource policies filter by caller identity/IP, not by which stage was invoked.
- D: A custom domain maps to a single stage (via a base path mapping); it doesn't change which Lambda alias a given stage's integration invokes.

**Study links:** [`services/networking-content-delivery/api-gateway.md`](../../../../services/networking-content-delivery/api-gateway.md), [`domain-1-development/README.md`](../../domain-1-development/README.md)

---

## Q35 — Domain 3

**Correct Answer: A**

**Explanation:** AppConfig is purpose-built for exactly this Task 3.1 "externalize configuration" pattern
taken further: it separates configuration deployment from code deployment entirely, supports
gradual percentage-based rollout strategies, and integrates with CloudWatch alarms to automatically
roll back a configuration deployment if the monitored alarm trips — all without touching the
application's deployment package.

**Why the others are wrong:**
- B: Baking the flag into the deployable artifact is exactly the anti-pattern the requirement says to avoid (full redeploy needed just to flip a flag).
- C: A hardcoded boolean in code is the least flexible option and still requires a full code redeploy to change — the opposite of "instant toggle."
- D: Raw S3 polling with no caching or deployment-strategy tooling reinvents (poorly) what AppConfig already provides managed, including rollout percentage and alarm-based rollback.

**Study links:** [`services/management-governance/systems-manager.md`](../../../../services/management-governance/systems-manager.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q36 — Domain 4

**Correct Answer: A**

**Explanation:** Systems Manager Automation runbooks are exactly the documented Task 4.2 "automating
remediation/runbooks" mechanism: using the SSM Agent (no SSH, no key management, no inbound ports
needed) to execute a defined, auditable, repeatable set of actions — and Automation documents can
be invoked directly from a CloudWatch alarm action for fully automatic remediation, removing the
manual SSH step entirely.

**Why the others are wrong:**
- B: Shared SSH keys are a major credential-management and audit-trail anti-pattern — no per-engineer attribution and a huge blast radius if the key leaks.
- C: Opening SSH to the entire internet is a serious security regression that trades a minor convenience for a large new attack surface.
- D: CloudWatch does not itself have an SSH client/remote-execution capability — alarm actions can invoke SSM Automation, Lambda, or SNS, not raw SSH.

**Study links:** [`services/management-governance/systems-manager.md`](../../../../services/management-governance/systems-manager.md)

---

## Q37 — Domain 2

**Correct Answer: B**

**Explanation:** With Lambda, AWS manages "security OF the cloud" — the underlying OS, host patching,
hypervisor, physical facilities, and the managed runtime environment itself. The customer is
responsible for "security IN the cloud": what their code does, how it's permissioned (IAM), what
data it handles and how that data is protected (encryption, secrets handling) — exactly Task 2.2's
scope for a serverless workload.

**Why the others are wrong:**
- A: This is explicitly AWS's responsibility for managed/serverless compute like Lambda — the customer never patches the underlying host OS or runtime infrastructure.
- C: Physical data center security is always AWS's responsibility across every service, never the customer's.
- D: Hypervisor-layer isolation and patching is part of AWS's 'security OF the cloud' infrastructure management, not something a Lambda customer can or needs to touch.

**Study links:** [`services/security-identity-compliance/shared-responsibility-model-deep-dive.md`](../../../../services/security-identity-compliance/shared-responsibility-model-deep-dive.md)

---

## Q38 — Domain 1

**Correct Answer: A**

**Explanation:** EventBridge rules support event patterns that match on the structure/values inside an
event's `detail` (and other fields), and each rule can target a different consumer — this is
exactly content-based routing/filtering done by the bus itself, so consumers never need to inspect
fields they don't care about or receive events irrelevant to them.

**Why the others are wrong:**
- B: Making the producer aware of every consumer's filtering needs tightly couples the order service to downstream consumers, which event-driven architecture is meant to avoid.
- C: A single unfiltered rule pushes every event to every target, contradicting "only relevant events" and pushing filtering logic back into consumer code — the opposite of what's asked.
- D: EventBridge explicitly supports rich content-based event pattern matching on event `detail` fields; this option is factually wrong about EventBridge's capability.

**Study links:** [`services/application-integration/eventbridge.md`](../../../../services/application-integration/eventbridge.md), [`services/application-integration/messaging-patterns-overview.md`](../../../../services/application-integration/messaging-patterns-overview.md)

---

## Q39 — Domain 3

**Correct Answer: A**

**Explanation:** `sam local invoke` and `sam local start-api` are exactly SAM's offline/local-testing
commands under Task 3.2's "use mock APIs and test events to exercise Lambda functions without live
dependencies": they run the function in a Docker container modeling the real Lambda execution
environment, accept local JSON test events, and can emulate the full API Gateway proxy-integration
HTTP surface for interactive testing — with zero real AWS resources touched or billed.

**Why the others are wrong:**
- B: There's no "local" deployment stage that runs inside a real AWS account without creating real resources — `sam deploy` always provisions actual cloud resources.
- C: Invoking a nonexistent Lambda ARN returns a `ResourceNotFoundException`; AWS does not auto-create functions on invoke.
- D: Running handler code directly in a notebook skips the actual Lambda runtime environment/event contract entirely, so it doesn't validate the real integration behavior the way SAM's local emulation does.

**Study links:** [`services/developer-tools/sam.md`](../../../../services/developer-tools/sam.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q40 — Domain 4

**Correct Answer: A**

**Explanation:** On-Demand capacity mode (or Auto Scaling on a provisioned table) is exactly the
documented Task 4.3 cost/performance tradeoff answer for unpredictable, spiky traffic: On-Demand
removes capacity planning entirely at a higher per-request cost, while Auto Scaling adjusts
provisioned capacity automatically within configured min/max bounds — either removes the need for
manual capacity tuning that's currently causing throttling.

**Why the others are wrong:**
- B: A GSI is a separate index with its own capacity and can itself throttle; it doesn't absorb or fix base-table throttling, and this option misstates how GSIs work.
- C: DynamoDB Streams captures a change log of table activity; enabling it does not increase the table's provisioned throughput ceiling.
- D: This is backwards — strongly consistent reads consume *more* read capacity than eventually consistent reads, not less, so this would make throttling worse, not better.

**Study links:** [`services/database/dynamodb.md`](../../../../services/database/dynamodb.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q41 — Domain 2

**Correct Answer: A**

**Explanation:** A WAF rate-based rule is exactly the documented mechanism for this: it tracks request
rate per originating IP over a rolling window and automatically blocks (or challenges) IPs that
exceed the configured threshold — ideal for application-layer abuse/scripted-client floods that
don't rise to the scale of a true volumetric DDoS.

**Why the others are wrong:**
- B: Shield Advanced focuses on larger-scale network/transport-layer DDoS protection and cost protection; the specific 'block abusive IPs by request rate' control at the application layer is WAF's rate-based rule, which Shield Advanced is typically paired with rather than substituted for.
- C: Raising the throttle limit lets more of the abusive traffic through to the backend rather than blocking it — the opposite of the desired outcome.
- D: The REST vs. HTTP API Gateway type choice is about feature set/cost/latency, not a defense against abusive traffic volume.

**Study links:** [`services/security-identity-compliance/waf.md`](../../../../services/security-identity-compliance/waf.md)

---

## Q42 — Domain 1

**Correct Answer: A**

**Explanation:** AppSync can attach a DynamoDB table as a direct data source and use a resolver mapping
template (VTL or APPSYNC_JS) to translate a GraphQL query straight into DynamoDB API calls
(`GetItem`, `Query`, `PutItem`, etc.) with zero custom compute to deploy or maintain for
straightforward CRUD resolvers — the documented "direct Lambda-less integration" pattern.

**Why the others are wrong:**
- B: API Gateway doesn't natively speak GraphQL field resolution; this would require significant custom routing and still needs a Lambda per field, which is what the question wants to avoid.
- C: Routing everything through one Lambda still requires writing and deploying that function and loses the no-code-resolver benefit AppSync's direct DynamoDB integration offers for simple operations.
- D: Cognito handles authentication, not GraphQL query resolution — unrelated to this requirement.

**Study links:** [`services/application-integration/appsync.md`](../../../../services/application-integration/appsync.md)

---

## Q43 — Domain 3

**Correct Answer: A**

**Explanation:** CodeBuild's buildspec defines an ordered sequence of phases (`install`, `pre_build`,
`build`, `post_build`), each with a `commands` list run in sequence, and CodeBuild fails the overall
build if any command in a phase returns a non-zero exit code (which is exactly how a failing test
suite in the `build` phase stops the pipeline); the `artifacts` section then declares which output
files/paths get zipped up and handed to the next pipeline stage.

**Why the others are wrong:**
- B: Collapsing everything into `pre_build` loses the documented separation of concerns and the implicit 'stop the build if build-phase commands fail' semantics associated with the `build` phase specifically for test runs.
- C: `artifacts` is a top-level section describing output files, not a phase that executes commands, and it doesn't need to be declared before other phases run — order doesn't work that way.
- D: Phases execute in a defined sequential order (`install` → `pre_build` → `build` → `post_build`), not in parallel — this option misstates how buildspec phases work.

**Study links:** [`services/developer-tools/codebuild.md`](../../../../services/developer-tools/codebuild.md)

---

## Q44 — Domain 4

**Correct Answer: A**

**Explanation:** Because Lambda cost is a function of both memory *and* duration, and duration doesn't
scale linearly with memory for every workload, the only reliable way to find the true
cost-optimal setting is to actually benchmark multiple memory values for the specific function
(exactly what the Power Tuning approach does) rather than assuming a universal rule — a CPU-bound
function may see large duration drops from more memory/CPU, while an I/O-bound function may not.

**Why the others are wrong:**
- B: Maximum memory is not universally cheapest — for many functions duration improvements plateau well before 10,240 MB, so cost per invocation can rise again past the actual optimal point.
- C: This ignores that duration is the other half of the cost equation; a workload whose duration drops sharply with more memory can end up cheaper overall at a higher memory setting, not lower.
- D: CPU-bound and I/O-bound workloads respond very differently to added memory/CPU (I/O-bound code often sees little duration improvement) — this is precisely why benchmarking per-function matters instead of assuming one universal behavior.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q45 — Domain 2

**Correct Answer: A and B**

**Explanation:** Shield Advanced and WAF are complementary, not redundant: Shield Advanced strengthens
network/transport-layer DDoS resilience and adds cost protection for scaling charges during an
attack, while WAF handles application-layer (L7) rules like rate limiting and malicious-pattern
blocking that Shield doesn't itself implement — using both together is the standard DVA-C03
"defense in depth against DDoS" pattern.

**Why the others are wrong:**
- C: Shield Standard (free, automatic) covers common, most-frequent network/transport DDoS patterns; Shield Advanced is a paid tier with additional detection, response team access (DRT), and cost protection — they are not identical.
- D: Shield protects against volumetric/protocol attacks; it does not replace WAF's application-layer rule evaluation (e.g. SQL injection patterns, rate-based IP blocking) — both are typically used together.
- E: Shield Advanced explicitly supports API Gateway, CloudFront, Global Accelerator, and ALB/NLB/EC2 with Elastic IPs — not EC2 only.

**Study links:** [`services/security-identity-compliance/shield.md`](../../../../services/security-identity-compliance/shield.md), [`services/security-identity-compliance/waf.md`](../../../../services/security-identity-compliance/waf.md)

---

## Q46 — Domain 1

**Correct Answer: A**

**Explanation:** `Retry` lets a state (or a specific error type) retry automatically with configurable
`IntervalSeconds`/`BackoffRate`/`MaxAttempts` before giving up, and `Catch` lets the state machine
route execution to a different state (e.g. a failure-notification branch) once retries are
exhausted or for errors `Retry` doesn't cover — together they implement exactly the "retry
transient errors, then gracefully handle a permanent one" pattern.

**Why the others are wrong:**
- B: `Parallel` runs branches concurrently and `Map` iterates over a collection — neither relates to error handling/retry semantics.
- C: `Choice` branches on data values and `Wait` pauses execution; neither implements retry-then-fallback error handling.
- D: `TimeoutSeconds` only bounds how long a task is allowed to run before being treated as failed — it doesn't retry or route to a fallback state on its own.

**Study links:** [`services/application-integration/step-functions.md`](../../../../services/application-integration/step-functions.md)

---

## Q47 — Domain 1

**Correct Answer: A and B**

**Explanation:** Redis's richer data model (and atomic ops like `INCR`) handles counters and structured
session data cleanly, and because the cache is a separate, shared service reachable from every
instance, any instance behind the load balancer can read a session another instance wrote — which
is the core requirement for a stateless fleet (no sticky-session dependency on which instance
served a prior request).

**Why the others are wrong:**
- C: Memcached does not provide built-in replication or automatic failover the way Redis does (via ElastiCache for Redis replication groups/Multi-AZ); this is a real, exam-relevant differentiator and this statement is false.
- D: Local in-instance memory is private to that instance — a request landing on a different instance behind the load balancer wouldn't see it, which breaks the stateless-fleet requirement entirely.
- E: DynamoDB can also store sessions, but ElastiCache is a fully valid (and lower-latency) option for this pattern too — this statement is simply false.

**Study links:** [`services/database/elasticache.md`](../../../../services/database/elasticache.md)

---

## Q48 — Domain 2

**Correct Answer: B**

**Explanation:** This finding specifically indicates likely-stolen instance credentials being used outside
the expected context — the correct immediate action is containment (rotate/invalidate the exposed
credentials) paired with CloudTrail-driven root cause analysis (Task 4.1: "use CloudTrail to find
who/what made an API call") to determine exactly what the compromised credentials were used to do,
so the blast radius can be assessed and remediated.

**Why the others are wrong:**
- A: This is a specific, named GuardDuty finding type for a serious credential-exfiltration pattern, not generic noise — treating it as a false positive by default risks missing an active compromise.
- C: Deleting the detector hides the symptom rather than addressing the underlying compromised-credential incident, and loses future detection capability entirely.
- D: Credential exfiltration is an active, time-sensitive security event — waiting for a scheduled review lets a live compromise continue unaddressed.

**Study links:** [`services/security-identity-compliance/guardduty.md`](../../../../services/security-identity-compliance/guardduty.md), [`services/management-governance/cloudtrail.md`](../../../../services/management-governance/cloudtrail.md)

---

## Q49 — Domain 3

**Correct Answer: A**

**Explanation:** Both CodeBuild (via its own webhook-based triggers) and CodePipeline (via a source stage
with webhook filters for PR events) support triggering automatically on pull-request creation/update
and reporting the pass/fail result back to the PR as a status check — enforcing "tested before
merge" rather than "tested, and reverted if broken, after merge."

**Why the others are wrong:**
- B: Testing only after merge means broken code was already merged to main before anyone found out — the opposite of a pre-merge quality gate.
- C: Manual local runs pasted into a PR description are unverifiable, inconsistent, and not an automated CI gate.
- D: A fixed daily schedule is decoupled from actual PR activity — a PR opened right after the daily run wouldn't get tested until the next day, failing the 'every PR' requirement.

**Study links:** [`services/developer-tools/codecommit.md`](../../../../services/developer-tools/codecommit.md), [`services/developer-tools/codebuild.md`](../../../../services/developer-tools/codebuild.md)

---

## Q50 — Domain 4

**Correct Answer: A**

**Explanation:** The CloudWatch evidence directly ties the symptom to Lambda concurrency exhaustion:
flat-lined concurrent executions at the reserved limit plus a `Throttles` spike means new
invocations are being rejected, and API Gateway's Lambda proxy integration surfaces that backend
throttling as a 503 to the caller — the fix is raising the concurrency ceiling (reserved or account
limit) or decoupling the burst with a buffer (SQS) so Lambda scales consumption at its own pace.

**Why the others are wrong:**
- B: Custom domain configuration is unrelated to backend throttling and doesn't explain the correlated Lambda `Throttles` metric spike shown in the evidence.
- C: X-Ray tracing overhead is minor and not a documented cause of 503s; this doesn't match the concurrency-exhaustion evidence presented.
- D: A shorter timeout makes each invocation fail faster but does not increase the number of concurrent invocations allowed — it doesn't address the actual concurrency ceiling being hit.

**Study links:** [`services/networking-content-delivery/api-gateway.md`](../../../../services/networking-content-delivery/api-gateway.md), [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q51 — Domain 1

**Correct Answer: A**

**Explanation:** Firehose is purpose-built as a managed delivery pipe: point it at a destination (S3,
Redshift, OpenSearch, etc.), optionally enable format conversion to Parquet/ORC and compression,
and it handles batching, buffering, and retries with zero consumer code to operate — exactly "no
custom consumer, just deliver it" versus Kinesis Data Streams, which requires you to write and run
a consumer yourself.

**Why the others are wrong:**
- B: This recreates Firehose's job by hand (custom buffering/consumer code to operate and keep highly available) — more operational burden for no benefit over using Firehose directly.
- C: SQS has no native batching-to-S3-with-format-conversion capability; this would require significant custom code and infrastructure to operate.
- D: Per-record Step Functions executions for high-volume IoT ingestion is both costly and the wrong tool — Step Functions orchestrates workflows, it isn't a streaming delivery pipe.

**Study links:** [`services/analytics/kinesis-family.md`](../../../../services/analytics/kinesis-family.md)

---

## Q52 — Domain 2

**Correct Answer: A**

**Explanation:** An AWS Organizations trail, enabled once from the management account and applied
org-wide, automatically covers every member account and (when configured multi-Region) every
Region, delivering to one centralized S3 destination — exactly the "single pane" compliance
requirement. Enabling log file validation adds SHA-256/RSA digest files AWS signs, so any
tampering with delivered log files becomes provably detectable, satisfying "tamper-evident."

**Why the others are wrong:**
- B: 15 independently managed trails create 15 places to audit, configure retention/permissions on, and reconcile — the opposite of a centralized compliance trail, and still needs validation enabled on each separately.
- C: CloudWatch Logs do not capture AWS management/API-call history by default — that is specifically what CloudTrail records; CloudWatch is a destination CloudTrail can optionally stream *into*, not a replacement for it.
- D: VPC Flow Logs capture IP traffic metadata at the network interface level — they do not record AWS API calls at all.

**Study links:** [`services/management-governance/cloudtrail.md`](../../../../services/management-governance/cloudtrail.md)

---

## Q53 — Domain 3

**Correct Answer: A**

**Explanation:** Automatic rollback on failure is CloudFormation's default stack update behavior
(`ROLLBACK_COMPLETE`/`UPDATE_ROLLBACK_COMPLETE`): if any resource fails to update, CloudFormation
reverts every resource it already changed back to the prior state, keeping the stack internally
consistent — which is why "rollback" for ordinary stack updates is a built-in safety net rather
than something the developer must explicitly wire up (unlike CodeDeploy's alarm-based rollback,
which is explicit configuration).

**Why the others are wrong:**
- B: This describes the opposite of CloudFormation's actual default behavior — it does *not* leave partial changes in place; it rolls them back automatically.
- C: A failed update does not delete the whole stack; rollback reverts to the previous good state, it doesn't tear down the stack itself.
- D: CloudFormation does not retry indefinitely by default; it fails the update and initiates rollback rather than hanging forever.

**Study links:** [`services/management-governance/cloudformation.md`](../../../../services/management-governance/cloudformation.md)

---

## Q54 — Domain 4

**Correct Answer: A**

**Explanation:** Trusted Advisor's cost optimization category is exactly built for this recurring,
automated waste-detection need — flagging specific underutilized or idle resources (low-CPU RDS/EC2
instances, unattached EBS volumes, idle load balancers) on an ongoing basis, which is more targeted
and continuous than a manual monthly spend-trend review.

**Why the others are wrong:**
- B: Budgets tells you *that* spend crossed a threshold, not *which specific resources* are the source of the waste — it's a complementary but different tool.
- C: Manual monthly review is exactly the non-automated, easy-to-skip process the requirement wants replaced with a standing, automatic check.
- D: Deleting a production database with no investigation into whether that CPU level is actually sufficient for its workload (or whether it's a read-replica serving a lighter but real purpose) is reckless; the right first step is review/right-sizing, not blind deletion.

**Study links:** [`services/management-governance/trusted-advisor.md`](../../../../services/management-governance/trusted-advisor.md), [`services/management-governance/cost-management-tools.md`](../../../../services/management-governance/cost-management-tools.md)

---

## Q55 — Domain 1

**Correct Answer: A**

**Explanation:** TTL is DynamoDB's native, no-code expiration mechanism: once an item's designated
TTL attribute (an epoch timestamp) is in the past, DynamoDB marks it for deletion and removes it in
the background, typically within 48 hours of expiration, without consuming write capacity for the
delete and without any application/cron logic required.

**Why the others are wrong:**
- B: This reinvents TTL with custom polling code, added cost (RCU/WCU for scanning and explicit deletes), and operational burden, when a native zero-code feature already does exactly this.
- C: A full-table `Scan` every hour is expensive (consumes read capacity proportional to table size) and exactly the pattern TTL exists to avoid.
- D: Billing mode (provisioned vs. on-demand) controls capacity management, not item expiration — this option is factually incorrect.

**Study links:** [`services/database/dynamodb.md`](../../../../services/database/dynamodb.md)

---

## Q56 — Domain 2

**Correct Answer: A**

**Explanation:** Task 2.3's access-control guidance for AI services is least-privilege exactly like any
other AWS API: a runtime application role should only get the specific invoke actions on the
specific model ARNs it's approved to call, never a `bedrock:*`/`Resource: *` wildcard — which
would also allow management-plane actions (creating custom models, modifying guardrails, deleting
resources) that a chatbot's runtime role has no legitimate need for.

**Why the others are wrong:**
- B: Bedrock absolutely has sensitive management-plane actions (e.g. managing custom models, guardrails, provisioned throughput); a wildcard grants all of them to a role that should only ever invoke a model.
- C: The model weights' own storage is managed internally by the Bedrock service, not something the caller's IAM policy needs S3 permissions for — this conflates two unrelated things.
- D: Spreading an overly broad policy to more roles multiplies the security exposure rather than fixing it.

**Study links:** [`domain-2-security/ai-service-security-risks.md`](../../domain-2-security/ai-service-security-risks.md), [`services/machine-learning-ai/bedrock.md`](../../../../services/machine-learning-ai/bedrock.md)

---

## Q57 — Domain 1

**Correct Answer: A**

**Explanation:** On a FIFO queue, `MessageGroupId` scopes strict ordering to messages sharing the same
group (e.g. all messages for one `orderId`), while deduplication — either automatic
content-based deduplication or a caller-supplied `MessageDeduplicationId` — causes SQS to silently
discard a second identical send within its 5-minute deduplication interval, satisfying both
"in order" and "no duplicates" requirements natively.

**Why the others are wrong:**
- B: `DelaySeconds` postpones initial visibility and `VisibilityTimeout` governs in-flight redelivery; neither provides ordering guarantees or deduplication — those are FIFO-specific features, not these settings.
- C: Long polling reduces empty-receive API calls/cost; it has no effect on message ordering or duplicate suppression, and standard queues don't guarantee strict order regardless.
- D: Manually re-sorting after the fact works around a standard queue's lack of ordering rather than using the queue type built to guarantee it, and does nothing about duplicate messages.

**Study links:** [`services/application-integration/sqs.md`](../../../../services/application-integration/sqs.md)

---

## Q58 — Domain 3

**Correct Answer: A**

**Explanation:** EKS is the right choice specifically when existing investment is in the Kubernetes API
and its ecosystem (kubectl, Helm, K8s-native CI/CD and observability tooling) — that tooling only
works against a real Kubernetes control plane, which EKS provides, whereas ECS has its own
proprietary API/task definition model. Fargate profiles then let the team avoid managing EC2 nodes
for the specified pods/namespaces while staying on native Kubernetes.

**Why the others are wrong:**
- B: ECS does not expose a Kubernetes API — Helm charts and `kubectl` do not work against ECS; it has an entirely separate task-definition/service model, so this statement is factually wrong.
- C: This conflates two independent choices (launch type vs. orchestrator) and is also factually wrong — Fargate is fully compatible with EKS via Fargate profiles.
- D: Neither ECS nor EKS is being deprecated, and Lambda is not a drop-in replacement for long-running, Kubernetes-API-dependent containerized workloads.

**Study links:** [`services/containers/eks.md`](../../../../services/containers/eks.md), [`services/containers/containers-vs-vms-vs-serverless.md`](../../../../services/containers/containers-vs-vms-vs-serverless.md)

---

## Q59 — Domain 4

**Correct Answer: A**

**Explanation:** Cache-aside with ElastiCache is the textbook Task 4.3 optimization for exactly this
profile — read-heavy, infrequently-changing data causing a hot, saturated database: most reads get
served from cache (sub-millisecond, no DB round trip) and only cache misses or changed data touch
RDS, directly reducing CPU/connection pressure on the database for the majority of traffic.

**Why the others are wrong:**
- B: Storage size affects capacity for data volume, not CPU/connection saturation from read query volume — it doesn't address the described bottleneck.
- C: Removing indexes would typically make reads (and often even writes with complex queries) slower, not faster, and doesn't relate to the stated read-heavy bottleneck at all.
- D: DNS resolution has no relationship to database read load; this option is a non-sequitur relative to the problem described.

**Study links:** [`services/database/elasticache.md`](../../../../services/database/elasticache.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q60 — Domain 2

**Correct Answer: A**

**Explanation:** This is precisely Task 2.3's "data privacy via VPC endpoints" bullet: a `bedrock-runtime`
interface VPC endpoint keeps Bedrock API traffic on AWS's private network path instead of transiting
the public internet, satisfying a no-public-egress compliance requirement while still allowing the
workload to call Bedrock — paired with an endpoint policy to further scope which roles/resources can
use that private path.

**Why the others are wrong:**
- B: Adding internet access is the opposite of the no-public-egress compliance requirement; it reintroduces the exact exposure the control is meant to prevent.
- C: Bedrock explicitly supports private connectivity via VPC endpoints — the compliance requirement does not need to be waived; this is a solvable, documented configuration.
- D: Lambda functions can run outside a VPC and still reach public AWS endpoints by default, but that reintroduces public-internet egress, which fails the stated compliance requirement — the VPC endpoint is the fix that keeps the function private *and* compliant.

**Study links:** [`domain-2-security/ai-service-security-risks.md`](../../domain-2-security/ai-service-security-risks.md), [`services/networking-content-delivery/vpc-endpoints-privatelink.md`](../../../../services/networking-content-delivery/vpc-endpoints-privatelink.md)

---

## Q61 — Domain 1

**Correct Answer: A**

**Explanation:** API Gateway supports attaching a JSON Schema request model to a method and enabling
request validation (`validateRequestBody` / `validateRequestParameters`), which checks the incoming
payload against the schema and returns a `400 Bad Request` automatically — without invoking the
backend integration at all — exactly "reject bad input before the Lambda runs."

**Why the others are wrong:**
- B: A Lambda authorizer's job is authentication/authorization (identity), not payload schema validation, and still involves invoking a Lambda function for every request.
- C: Silently stripping fields changes the payload rather than rejecting it, and doesn't satisfy "reject malformed requests with a 400."
- D: Memory affects compute speed, not whether invalid requests are filtered before reaching the function at all.

**Study links:** [`services/networking-content-delivery/api-gateway.md`](../../../../services/networking-content-delivery/api-gateway.md)

---

## Q62 — Domain 3

**Correct Answer: A**

**Explanation:** This is explicitly Task 3.2's "AI-assisted test generation" item: an AI assistant
accelerates writing the first draft of a test suite (including edge cases a developer might not
think of immediately), but the documented pattern still requires human review before the generated
tests are trusted as the real regression baseline — AI drafts, a person verifies.

**Why the others are wrong:**
- B: Skipping automated unit tests in favor of only manual QA is a regression in test coverage and automation, not what's being described or recommended.
- C: Trusting AI-generated tests with zero human review risks subtly wrong assertions (tests that pass but don't actually validate correct behavior) — the documented practice always includes a review step.
- D: The scenario is specifically about using AI assistance for a first draft, which this option incorrectly rules out entirely.

**Study links:** [`domain-1-development/ai-assisted-development.md`](../../domain-1-development/ai-assisted-development.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q63 — Domain 4

**Correct Answer: A**

**Explanation:** Exactly like AI-assisted test generation, Task 4.1's AI-assisted error analysis pattern
is "AI accelerates the first pass, a human verifies before it's trusted" — summarizing a large
volume of logs/traces into a likely cause narrows where to look fast, but the documented practice
still requires an engineer to confirm that hypothesis against the real evidence before it becomes
the record of what actually happened.

**Why the others are wrong:**
- B: Treating unverified AI output as the final confirmed cause risks enshrining a plausible-sounding but wrong conclusion in the official record — verification is the documented step this skips.
- C: AI-assisted error analysis is explicitly called out as in-scope content for DVA-C03's Task 4.1 — this option is factually wrong about exam scope.
- D: Auto-closing an incident based on unverified AI output skips human confirmation entirely, which is the opposite of the documented safe pattern.

**Study links:** [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md), [`domain-1-development/ai-assisted-development.md`](../../domain-1-development/ai-assisted-development.md)

---

## Q64 — Domain 2

**Correct Answer: A and B**

**Explanation:** Guardrails are Bedrock's managed control for content filtering and prompt-injection
mitigation, applied at the service layer to both input and output; and scoping the Agent's own IAM
execution role to least privilege ensures that even a successfully manipulated agent still can't
exceed its intended toolset — these two controls together address both halves of Task 2.3's
"content filtering/prompt injection" and "AI agent security" bullets.

**Why the others are wrong:**
- C: Foundation models remain susceptible to crafted prompt-injection inputs; skipping a dedicated guardrail layer is explicitly against documented best practice.
- D: Granting a chatbot's agent `AdministratorAccess` is the textbook violation of least privilege for agent tool use and massively increases blast radius if the agent is manipulated.
- E: Client-side filtering is trivially bypassed by any user calling the API directly; server-side enforcement (Guardrails) is required, not optional or redundant.

**Study links:** [`domain-2-security/ai-service-security-risks.md`](../../domain-2-security/ai-service-security-risks.md), [`services/machine-learning-ai/bedrock.md`](../../../../services/machine-learning-ai/bedrock.md)

---

## Q65 — Domain 1

**Correct Answer: A and B**

**Explanation:** SNS subscription filter policies let each subscription declare which messages it wants
based on message attributes, and SNS evaluates that at delivery time per subscription — so the
same topic can fan out to many SQS queues (or other protocols) while each only gets the subset it
asked for, as long as the relevant data is published as **message attributes** (not just inside the
opaque message body) so SNS can evaluate it.

**Why the others are wrong:**
- C: SNS fans out to many protocols including SQS, Lambda, HTTP(S), email, and mobile push — this statement is factually wrong.
- D: Filter policies are evaluated by SNS at delivery time against each subscription, not something the publisher applies before calling `Publish`.
- E: This describes default behavior with **no filter policy** attached; once a subscription has a filter policy, SNS narrows delivery to only matching messages for that subscription — so "always every message, no way to narrow" is false once filtering is used.

**Study links:** [`services/application-integration/sns.md`](../../../../services/application-integration/sns.md), [`services/application-integration/messaging-patterns-overview.md`](../../../../services/application-integration/messaging-patterns-overview.md)

---
