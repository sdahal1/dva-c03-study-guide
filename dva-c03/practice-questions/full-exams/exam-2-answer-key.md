# DVA-C03 Full Mock Exam 2 — Answer Key

Correct answer(s), full explanation, and why each distractor is wrong — cross-linked to the relevant service/domain docs. Matches the question numbering in the companion `*-questions.md` file exactly.

---

## Q1 — Domain 1

**Correct Answer: A**

**Explanation:** DynamoDB's `UpdateItem` with an atomic counter update (`ADD` or an arithmetic `SET`
expression) performs the read-modify-write entirely server-side as a single atomic operation, so
two concurrent increments from different scanners both land correctly instead of one overwriting
the other — this removes the check-then-act window inherent in a separate GetItem/PutItem pair.

**Why the others are wrong:**
- B: Consistent reads guarantee you see the latest written value at read time, but the race is in the *gap* between read and write in application code — a consistent read doesn't close that gap by itself.
- C: Splitting state across per-scanner tables adds significant complexity and a batch-reconciliation delay, when a single atomic update expression solves it immediately and simply.
- D: `ProjectionExpression` only applies to reads, limiting which attributes are returned; it has no effect on write semantics or atomicity.

**Study links:** [`services/database/dynamodb.md`](../../../../services/database/dynamodb.md)

---

## Q2 — Domain 2

**Correct Answer: A**

**Explanation:** `NotAction` inverts the action list — an `Allow` with `NotAction: iam:*` and `Resource:
*` grants every action in every other service, which is a dramatically broader grant than it
appears at first glance. This is a real, documented exam trap: `NotAction` is easy to misread as
restrictive when paired with `Allow`, when it actually produces a near-administrator-level grant
unless scoped down with additional conditions or resource restrictions.

**Why the others are wrong:**
- B: This misreads the semantics entirely — with `Effect: Allow`, `NotAction` grants everything *except* the listed actions, not the reverse.
- C: `NotAction` is a valid, documented IAM policy element with well-defined (if often misunderstood) semantics; it is not a no-op.
- D: `NotAction` and `Action` are two distinct, valid policy elements, not a typo of one another — there is no silent auto-correction in IAM.

**Study links:** [`services/security-identity-compliance/iam.md`](../../../../services/security-identity-compliance/iam.md)

---

## Q3 — Domain 3

**Correct Answer: A**

**Explanation:** This is Task 3.2's "validate infrastructure templates before deploying" extended to
custom organizational security rules: a CDK-aware static analysis tool (synthesizing the template
and checking it, or checking constructs directly) run in CI catches issues like overly permissive
security groups or missing encryption *before* `cdk deploy` ever executes — exactly matching
"customizable rules, pre-deploy, in CI" rather than generic syntax checking or after-the-fact
detection.

**Why the others are wrong:**
- B: `cdk synth` only confirms the CDK app can successfully produce valid CloudFormation syntax — it says nothing about whether the resulting resources follow security best practices.
- C: Manual review doesn't scale, is inconsistent, and is exactly the kind of check that should be automated and enforced in CI rather than left to individual diligence.
- D: Catching the misconfiguration only after it's already live in production (via AWS Config) is reactive, not preventive — the question specifically asks for a pre-deploy check.

**Study links:** [`services/management-governance/cdk.md`](../../../../services/management-governance/cdk.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q4 — Domain 4

**Correct Answer: A**

**Explanation:** Logs Insights explicitly supports selecting multiple log groups for a single query, so
one query — filtering for the exact error string and sorting by timestamp — searches all 12
functions' logs simultaneously and surfaces the earliest matching line directly, which is both the
fastest and most reliable approach for a cross-function root-cause timeline question like this.

**Why the others are wrong:**
- B: Manually scanning 12 separate consoles one at a time is slow, error-prone, and doesn't easily let you compare exact timestamps across functions to determine which came first.
- C: Exporting and searching locally works but is slower and more manual than a single Insights query that already runs server-side across all the specified log groups at once.
- D: Assuming deterministic origin without evidence is exactly the kind of unverified guess root-cause analysis should avoid — the question specifically requires finding the actual first occurrence.

**Study links:** [`services/management-governance/cloudwatch.md`](../../../../services/management-governance/cloudwatch.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q5 — Domain 1

**Correct Answer: A**

**Explanation:** On-Demand mode for Kinesis Data Streams automatically scales the stream's capacity based
on observed throughput over the preceding 30 days, without the developer having to manually reshard
(split/merge shards) for predictable but varying peak/off-peak patterns — directly addressing
"handle peak throughput without permanently over-provisioning for off-peak" more simply than manual
provisioned-mode resharding.

**Why the others are wrong:**
- B: Permanently doubling shard count fixes peak throughput but keeps paying for that doubled capacity during off-peak hours too — exactly the over-provisioning the team wants to avoid.
- C: SQS is a different service with a different delivery model (queue, not an ordered replayable stream) and migrating to it is a far larger change than adjusting Kinesis capacity mode, and SQS Standard queues are also not literally unlimited in a way that requires zero configuration.
- D: Retention period controls how long records remain available to replay; it has no relationship to write throughput capacity.

**Study links:** [`services/analytics/kinesis-family.md`](../../../../services/analytics/kinesis-family.md)

---

## Q6 — Domain 2

**Correct Answer: A**

**Explanation:** `AssumeRoleWithWebIdentity` with an OIDC identity provider is exactly AWS's
documented keyless CI/CD pattern: GitHub Actions presents a short-lived, workflow-scoped OIDC
token, IAM verifies it against a configured trust relationship (which can be scoped to specific
repos/branches/environments), and STS returns short-lived AWS credentials for that run only — no
long-lived access key ever needs to be created, stored, or rotated.

**Why the others are wrong:**
- B: Storing a long-lived access key, even encrypted as a GitHub secret, is exactly the risk the OIDC pattern is designed to eliminate — it remains a standing credential that could leak or be misused.
- C: Root account keys are a severe security anti-pattern under any circumstance — root credentials should never be used for routine automation, and never expiring is a liability here, not a benefit.
- D: Hardcoding credentials in a workflow file, private repo or not, is a well-known credential-exposure risk (history, forks, logs, misconfigured visibility) and should never be done regardless of repo privacy settings.

**Study links:** [`services/security-identity-compliance/sts-and-role-switching.md`](../../../../services/security-identity-compliance/sts-and-role-switching.md)

---

## Q7 — Domain 3

**Correct Answer: A**

**Explanation:** Cloud9 is specifically a managed, browser-accessible cloud IDE that comes pre-configured
with the AWS CLI/SDKs and credentials scoped to the user's role — exactly the "no local setup,
consistent environment, easy to provision/tear down per contractor" requirement, standing in
contrast to each person configuring their own machine from scratch.

**Why the others are wrong:**
- B: A bare EC2 instance has no IDE pre-installed by default; that capability specifically comes from a service like Cloud9 layered on top.
- C: CloudShell is a browser-based shell with the AWS CLI pre-authenticated, but its persistent storage is limited (1 GB home directory) and it's meant for quick CLI tasks, not a full persistent development environment with an IDE — this statement also mischaracterizes it as having no storage limit.
- D: Amazon WorkSpaces provides managed virtual desktops; it is not a video-streaming service — this statement is factually wrong about what WorkSpaces does.

**Study links:** [`services/developer-tools/cloud9-and-cloudshell.md`](../../../../services/developer-tools/cloud9-and-cloudshell.md)

---

## Q8 — Domain 4

**Correct Answer: A**

**Explanation:** This is exactly the documented distinction: annotations are indexed and searchable
(the right choice for `customerId`-based trace filtering), while metadata holds richer, non-indexed
context visible when drilling into one specific trace but not usable for search/filter queries —
together they map precisely to "searchable business key" + "rich debug context" in this scenario.

**Why the others are wrong:**
- B: This is factually wrong — the key documented difference between annotations and metadata is specifically that annotations are indexed/searchable and metadata is not; conflating them loses that distinction.
- C: Subsegments represent timed sub-operations within a segment (e.g. a downstream call); they aren't the mechanism for attaching arbitrary custom key-value data — annotations and metadata are.
- D: This is factually incorrect — X-Ray explicitly supports both annotations and metadata as customizable fields on segments/subsegments.

**Study links:** [`domain-4-troubleshooting-optimization/observability-instrumentation.md`](../../domain-4-troubleshooting-optimization/observability-instrumentation.md)

---

## Q9 — Domain 1

**Correct Answer: A**

**Explanation:** This is precisely what non-proxy (custom) integration is for: a mapping template
(written in VTL) sits between the Lambda's raw return value and the client response, transforming
the shape as needed per-method — without touching the shared Lambda function's code or its other
consumers' expected contract.

**Why the others are wrong:**
- B: Proxy integration passes the Lambda's response straight through unmodified (aside from the proxy response contract) — it does not reshape JSON; that transformation capability belongs to non-proxy mapping templates, which this scenario is already using.
- C: Pushing the transformation requirement onto every client contradicts the stated goal of a single clean reshaping done once, and increases client-side complexity unnecessarily.
- D: Adding a second Lambda and a Step Functions execution for a simple field rename is far more operational overhead and latency than a mapping template evaluated inline by API Gateway.

**Study links:** [`services/networking-content-delivery/api-gateway.md`](../../../../services/networking-content-delivery/api-gateway.md)

---

## Q10 — Domain 2

**Correct Answer: A**

**Explanation:** The Pre Sign-up Lambda trigger is exactly the documented Cognito extension point for
custom sign-up business logic: it fires synchronously as part of the `SignUp` flow, before the
user is created, letting custom code (like a domain allow-list check) reject or auto-confirm the
sign-up — directly implementing "only approved domains may sign up" without any after-the-fact
cleanup.

**Why the others are wrong:**
- B: IAM policies control which AWS principals can call AWS APIs; they have no visibility into or control over end-user-submitted sign-up form fields like an email domain.
- C: Identity pools issue temporary AWS credentials after authentication; they have no role in the user pool's sign-up validation flow at all.
- D: Manual post-hoc review and deletion lets unapproved accounts exist (even briefly) and doesn't scale, unlike a synchronous pre-sign-up check that rejects them immediately.

**Study links:** [`services/security-identity-compliance/cognito.md`](../../../../services/security-identity-compliance/cognito.md)

---

## Q11 — Domain 3

**Correct Answer: A**

**Explanation:** Change sets expose exactly this information — per-resource action (`Add`/`Modify`/
`Remove`) and, critically, whether a `Modify` requires replacement — so a pipeline stage that
inspects (or gates manual approval on) the change set's replacement indicators before executing it
catches exactly this class of destructive-update-by-accident, directly addressing Task 3.2's
"validate infrastructure templates before deploying them" for a stateful-resource safety case.

**Why the others are wrong:**
- B: This is false — many property changes on resources like RDS instances (e.g. changing certain parameters) implicitly require CloudFormation to replace the resource; this isn't a separate explicit action the developer must request, which is exactly what caused the incident.
- C: Abandoning CloudFormation for manual console management reintroduces all the drift/consistency/audit problems IaC solves, as an overcorrection to a solvable pipeline gap.
- D: Backup retention period affects how far back restorable backups exist; it has no bearing on whether a CloudFormation update triggers resource replacement.

**Study links:** [`services/management-governance/cloudformation.md`](../../../../services/management-governance/cloudformation.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q12 — Domain 4

**Correct Answer: A**

**Explanation:** CloudTrail Insights is specifically the feature built for this: it continuously
analyzes a trail's management event activity against an automatically learned baseline and flags
statistically unusual patterns (like an abnormal spike in a particular API call's volume from a
given actor) as Insights events — exactly "automated volume-anomaly detection" rather than a
manual historical comparison.

**Why the others are wrong:**
- B: Data events add object/resource-level logging granularity (e.g. S3 `GetObject`); they are not themselves a statistical anomaly detection feature — that's specifically Insights.
- C: AWS Config evaluates resource configuration compliance; it has no concept of API call volume/statistical anomaly detection — that's outside its scope entirely.
- D: Manual spreadsheet analysis is exactly the non-automated, easily-missed process the team wants replaced with a standing automated capability.

**Study links:** [`services/management-governance/cloudtrail.md`](../../../../services/management-governance/cloudtrail.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q13 — Domain 1

**Correct Answer: A**

**Explanation:** Bedrock (like many AWS services) is reachable via an interface VPC endpoint
(PrivateLink), which creates ENIs inside the specified subnets that resolve the service's private
DNS name — this gives the Lambda function a private path to Bedrock without needing any internet
gateway/NAT at all, keeping the subnet fully private while still being functional, which is more
scoped than opening general internet egress just for one service call.

**Why the others are wrong:**
- B: A NAT gateway solves connectivity but grants broad internet egress to resources that otherwise only needed to talk to RDS and one specific AWS API — more exposure than necessary when an interface endpoint solves it precisely.
- C: Splitting the workload across a private function and a separate public-subnet function adds architectural complexity to work around a solvable connectivity gap.
- D: There is no route to the destination at all without an endpoint or NAT; raising the timeout doesn't create a network path, so the call will still never succeed.

**Study links:** [`services/networking-content-delivery/vpc-endpoints-privatelink.md`](../../../../services/networking-content-delivery/vpc-endpoints-privatelink.md), [`services/machine-learning-ai/bedrock.md`](../../../../services/machine-learning-ai/bedrock.md)

---

## Q14 — Domain 2

**Correct Answer: A**

**Explanation:** Permission sets in IAM Identity Center are explicitly designed to be assigned per
user/group and per account — a `ReadOnly` permission set assigned to exactly one account gives
exactly the scoped, time-boxed access needed, with centralized visibility into every assignment and
one-click revocation by removing the account assignment when the engagement ends, which is both
more auditable and more easily revocable than a standalone IAM user.

**Why the others are wrong:**
- B: A standalone IAM user with an emailed password is a far weaker, harder-to-audit, harder-to-revoke-cleanly pattern than a centrally managed Identity Center assignment, and email is not a secure credential-sharing channel.
- C: Permission sets are explicitly scoped per-account during assignment; granting broad admin access across all 30 accounts when only one account and read-only access is needed is a severe over-grant.
- D: Root user credentials should never be shared with anyone, let alone a short-term contractor — this is one of the most severe possible security anti-patterns.

**Study links:** [`services/security-identity-compliance/iam-identity-center-sso.md`](../../../../services/security-identity-compliance/iam-identity-center-sso.md)

---

## Q15 — Domain 1

**Correct Answer: A**

**Explanation:** Cross-account layer sharing uses the layer version's own resource-based policy
(distinct from the function's execution role's identity-based IAM policy): the layer owner grants
a specific account (or principal) `lambda:GetLayerVersion` permission on a specific layer version
ARN, after which a function in the granted account can reference that exact ARN when attaching
layers — each layer version's permission must be granted individually.

**Why the others are wrong:**
- B: Layers are private to the owning account by default; cross-account use requires an explicit resource-based permission grant, not an automatic public default.
- C: Cross-account layer sharing is explicitly supported via resource-based permissions on the layer version; re-publishing a duplicate copy is unnecessary extra work and loses version parity with the source.
- D: `lambda:*` IAM permissions in Account A don't grant Account B's principals anything — IAM policies in Account A only govern principals that can be evaluated against Account A's own policies, not resource-based cross-account layer access, which needs the layer's own resource policy.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q16 — Domain 3

**Correct Answer: A**

**Explanation:** ECR repository policies are resource-based policies specifically for granting
cross-account pull (or push) access to a named external account/principal, scoped to exactly the
pull-related actions needed — satisfying "shared with one partner account, not the whole internet"
precisely, without resorting to public visibility or over-granting broad account access.

**Why the others are wrong:**
- B: Making a repository public exposes the image to anyone on the internet, which is explicitly the opposite of the stated requirement to share with just one partner.
- C: Emailing a tarball is manual, doesn't scale, loses version/tag integrity, and isn't how a CI/CD pipeline should consume container images.
- D: Granting `AdministratorAccess` to an external account is a massive, unnecessary over-grant just to let them pull one image — repository-scoped pull permissions are both sufficient and far safer.

**Study links:** [`services/containers/ecr.md`](../../../../services/containers/ecr.md)

---

## Q17 — Domain 4

**Correct Answer: A**

**Explanation:** The account-level concurrent executions limit is a documented, adjustable service
quota — the correct, supported path when the real constraint is total account capacity (not an
individual function's own reserved concurrency) is to request a quota increase through Service
Quotas, which raises the actual shared ceiling rather than cannibalizing capacity from other
legitimate workloads.

**Why the others are wrong:**
- B: Setting other functions' reserved concurrency to zero would block those functions from executing at all, trading one team's outage for another's — a destructive workaround rather than the supported fix.
- C: Permanently deleting unrelated functions to free up concurrency for one workload is a drastic, almost certainly unacceptable operational response to a capacity problem that has a proper request-based solution.
- D: This is factually wrong — AWS account-level service quotas, including concurrent Lambda executions, are explicitly adjustable via a quota increase request; they are not hard, permanent limits.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q18 — Domain 2

**Correct Answer: A**

**Explanation:** Because SCPs bound the maximum permissions available to every principal in an account
— including that account's own administrators — an SCP explicitly denying the trail-stopping and
log-bucket-deletion actions is the only one of these options that actually prevents even a
full-admin account owner from tampering with the audit trail, which is exactly the guarantee a
compliance requirement like this needs.

**Why the others are wrong:**
- B: No such named feature exists, and even if a toggle like this existed, something an admin can simply toggle would not meet a hard compliance guarantee against that same admin.
- C: An IAM policy scoped to a service role doesn't constrain what an account's own administrators (who can edit IAM policies) are able to do — only an SCP, enforced outside the account's own IAM control, can bind the admin themselves.
- D: Relying on trust with no enforced technical control doesn't satisfy a compliance requirement for a guaranteed, unbroken audit trail.

**Study links:** [`services/security-identity-compliance/organizations-and-scps.md`](../../../../services/security-identity-compliance/organizations-and-scps.md), [`services/management-governance/cloudtrail.md`](../../../../services/management-governance/cloudtrail.md)

---

## Q19 — Domain 1

**Correct Answer: A**

**Explanation:** Provisioned concurrency is explicitly configured per function version (directly, or via
an alias pointing to that version) — it does not automatically "follow" a new version. The
documented pattern for safe deploys is to provision concurrency on the new version ahead of the
traffic shift (e.g. CodeDeploy supports pre-provisioning during a canary/linear deployment) so
warmed capacity exists before real traffic arrives at the new version.

**Why the others are wrong:**
- B: This is false — provisioned concurrency must be explicitly configured on a version or alias; it does not automatically migrate to new versions on publish.
- C: Provisioned concurrency can be configured on `$LATEST` (unqualified ARN) or on a published version/alias; aliases are a standard, commonly used target for it, especially for blue/green-style promotions.
- D: Reserved concurrency (an account-level allocation cap/floor) and provisioned concurrency (pre-initialized, warmed execution environments) are distinct settings that serve different purposes, despite the similar names.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q20 — Domain 3

**Correct Answer: A**

**Explanation:** Moving test execution into a pipeline stage (CodeBuild within CodePipeline) makes
passing tests a structural precondition for reaching the deploy stage — the pipeline itself
enforces the gate for the exact commit being deployed, which is fundamentally different from (and
far more reliable than) relying on a developer's local habit or an easily-skipped manual step.

**Why the others are wrong:**
- B: A reminder is not an enforced gate — it relies entirely on developer discipline and can be silently skipped, which is exactly the unreliability the team wants to eliminate.
- C: Testing only after production deployment doesn't prevent a broken build from reaching production in the first place — it's detection after the fact, not a gate.
- D: A screenshot is unverifiable, easy to fake or be stale, and isn't tied programmatically to the specific commit being deployed — it's not a real automated gate.

**Study links:** [`services/developer-tools/codepipeline.md`](../../../../services/developer-tools/codepipeline.md), [`services/developer-tools/codebuild.md`](../../../../services/developer-tools/codebuild.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q21 — Domain 4

**Correct Answer: A**

**Explanation:** CloudWatch's cross-account observability feature (built on Observability Access Manager
account linking) is specifically designed for exactly this: a central monitoring account's
dashboard can pull in and display metrics/logs/traces from multiple linked source accounts in one
unified view, which is the documented solution for "one dashboard, several accounts" rather than
maintaining duplicate per-account dashboards or forcing account-switching.

**Why the others are wrong:**
- B: This is factually wrong — CloudWatch explicitly supports cross-account dashboards via account linking; dashboards are not strictly single-account only.
- C: Manual copy-paste is slow, error-prone, stale almost immediately, and exactly what a native cross-account feature replaces.
- D: AWS Budgets is a cost-management/alerting tool; it has no role in cross-account CloudWatch metric dashboard aggregation.

**Study links:** [`services/management-governance/cloudwatch.md`](../../../../services/management-governance/cloudwatch.md)

---

## Q22 — Domain 2

**Correct Answer: A**

**Explanation:** Private CA is exactly the service for internal-only PKI: it issues certificates from a
CA hierarchy that's trusted only by systems explicitly configured to trust it (e.g. internal
service meshes doing mTLS), which is the right tool when certificates are for internal,
never-public-facing use — unlike public ACM certificates, which are meant for publicly trusted
TLS endpoints.

**Why the others are wrong:**
- B: Public ACM certificates are issued from a publicly trusted CA chain and are meant for internet-facing endpoints; using that mechanism doesn't make sense (and isn't the documented approach) for purely internal, never-public service-to-service mTLS.
- C: Cognito is an identity/authentication service for end users and applications; it does not issue X.509 certificates for mTLS between backend services.
- D: KMS manages symmetric/asymmetric encryption keys; a raw KMS key is not itself a TLS certificate and cannot be used as one without an entirely different issuance process (which is what Private CA provides).

**Study links:** [`services/security-identity-compliance/acm-and-private-ca.md`](../../../../services/security-identity-compliance/acm-and-private-ca.md)

---

## Q23 — Domain 1

**Correct Answer: A**

**Explanation:** A DLQ with `maxReceiveCount` is the documented poison-message handling pattern for
SQS-triggered Lambda: once a message has failed processing the configured number of times, SQS
redirects it to the DLQ automatically instead of endlessly recycling it back onto the main queue —
unblocking batches (since, depending on batch item failure reporting, a persistently failing
message no longer keeps forcing retries of the whole batch) and giving the team a place to inspect
the bad message later.

**Why the others are wrong:**
- B: Memory affects compute resources, not whether a specific message's content is malformed and causes a parsing exception — it won't fix a genuinely bad payload.
- C: Deleting the whole queue destroys every currently-queued valid message along with the one bad one — a wildly disproportionate and destructive fix.
- D: A visibility timeout of 0 would make messages immediately visible again to other consumers while still being processed, causing even more duplicate concurrent processing — the opposite of the desired effect.

**Study links:** [`services/application-integration/sqs.md`](../../../../services/application-integration/sqs.md), [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q24 — Domain 1

**Correct Answer: A**

**Explanation:** Transfer Acceleration is specifically built to speed up transfers (including uploads)
over long geographic distances by using CloudFront's global edge network as an on-ramp onto AWS's
backbone, rather than having the client's bytes traverse the public internet all the way to the
bucket's Region — exactly the documented fix for "far-away uploaders, slow long-haul network path."

**Why the others are wrong:**
- B: Intelligent-Tiering optimizes storage cost based on access frequency after objects exist; it has no effect on upload transfer speed.
- C: Object Lock enforces retention/immutability policies; it is unrelated to transfer performance.
- D: CRR only begins after an object is already fully uploaded to the source bucket — it doesn't address the upload leg's latency at all, just post-upload replication to another Region.

**Study links:** [`services/storage/s3.md`](../../../../services/storage/s3.md)

---

## Q25 — Domain 2

**Correct Answer: A**

**Explanation:** This is the textbook envelope encryption pattern and exactly why `GenerateDataKey`
exists: KMS generates and returns both a plaintext and an encrypted copy of a data key, the
plaintext copy is used locally for fast bulk symmetric encryption of the large payload (never
sent to KMS), and only the small encrypted data key (not the large file) needs a KMS `Decrypt`
call later — this is precisely Task 2.2's "envelope encryption" pattern for large data.

**Why the others are wrong:**
- B: `kms:Encrypt` is explicitly documented with a small payload size limit (4 KB for symmetric keys) — it is not designed or usable for directly encrypting a 50 MB file; this option is factually incorrect.
- C: KMS key material is deliberately never exposed outside KMS for customer-managed keys backed by KMS's own HSMs — there is no supported way to extract raw key material to derive keys manually, and doing so would undermine the whole security model.
- D: Encrypting only the filename leaves the actual file content (the sensitive 50 MB payload) completely unencrypted, failing any real encryption-at-rest requirement.

**Study links:** [`services/security-identity-compliance/kms.md`](../../../../services/security-identity-compliance/kms.md)

---

## Q26 — Domain 3

**Correct Answer: A**

**Explanation:** CodeDeploy's `Linear` deployment type for Lambda shifts traffic in equal, scheduled
increments (e.g. 10% every minute) rather than all at once, and wiring a CloudWatch alarm to the
deployment group means a detected regression automatically halts the shift and rolls back — exactly
turning "100% blast radius immediately" into "small, bounded exposure with automatic protection,"
addressing both the gradual-shift and automatic-rollback requirements in one configuration.

**Why the others are wrong:**
- B: Manual traffic adjustment with eyeballed metrics reintroduces the exact unreliable, slow-to-react manual rollback process that caused the incident described.
- C: Memory size has no relationship to deployment traffic-shifting strategy or blast radius during a bad release.
- D: Repeating the same deployment doesn't change its traffic-shifting behavior at all; it's an irrelevant action.

**Study links:** [`services/developer-tools/codedeploy.md`](../../../../services/developer-tools/codedeploy.md), [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q27 — Domain 4

**Correct Answer: A**

**Explanation:** X-Ray's automatic SDK instrumentation specifically covers calls made through the AWS
SDK; capturing a non-AWS HTTP call as its own visible subsegment requires explicitly wrapping that
call (via the SDK's HTTP capture helpers or manual subsegment begin/end calls) — this is the
documented way to make a third-party dependency's latency visible and attributable in the trace
timeline rather than invisible or folded into the parent segment's total duration.

**Why the others are wrong:**
- B: Automatic instrumentation specifically covers the AWS SDK clients it wraps; a third-party HTTP call to a non-AWS domain needs to be explicitly captured to show up as its own subsegment — this isn't zero-configuration for arbitrary HTTP calls.
- C: A third-party dependency generally can't be meaningfully switched to use the AWS SDK (it's not an AWS service), and this doesn't match the actual mechanism X-Ray provides for capturing arbitrary HTTP calls as subsegments.
- D: X-Ray explicitly supports capturing calls to non-AWS HTTP endpoints as subsegments via its HTTP capture helpers; this statement is factually wrong about X-Ray's capability.

**Study links:** [`domain-4-troubleshooting-optimization/observability-instrumentation.md`](../../domain-4-troubleshooting-optimization/observability-instrumentation.md)

---

## Q28 — Domain 1

**Correct Answer: A**

**Explanation:** `TransactWriteItems` is exactly DynamoDB's native ACID-transaction API for this: up to
100 actions across one or more tables are committed as a single atomic unit — all succeed or all
are rolled back — which is the documented mechanism for "these two writes must happen together or
not at all," satisfying the cross-item/cross-table consistency guarantee this scenario requires.

**Why the others are wrong:**
- B: DynamoDB does not automatically group unrelated sequential API calls into a transaction just because they're in the same Lambda invocation — without `TransactWriteItems`, the two writes are fully independent and either could succeed while the other fails.
- C: DynamoDB Streams is a change-capture mechanism for reacting to writes after they happen; it has no transactional rollback capability for a separate, already-committed write.
- D: Consistent reads affect read freshness, not whether two separate writes commit together atomically — they don't provide any transactional guarantee.

**Study links:** [`services/database/dynamodb.md`](../../../../services/database/dynamodb.md)

---

## Q29 — Domain 2

**Correct Answer: A**

**Explanation:** Caching with a bounded TTL (via the purpose-built Parameters and Secrets Lambda
Extension, or a simple in-handler cache keyed by execution environment reuse) is the documented
way to reduce repeated Parameter Store calls while still allowing config changes to take effect
after the cache expires — directly balancing "fewer calls/less latency" against "still updatable
without redeploying code," which permanently baking the value in would sacrifice.

**Why the others are wrong:**
- B: Baking the value in at build time means any future change again requires a full redeploy, which is exactly the flexibility the team wants to keep.
- C: Memory configuration has no relationship to Parameter Store client-side caching; this is not a real feature of Lambda memory settings.
- D: Secrets Manager is also a remote API call with its own latency characteristics; switching services doesn't eliminate network/API call latency, and this statement about zero latency is not accurate.

**Study links:** [`services/security-identity-compliance/secrets-manager-vs-parameter-store.md`](../../../../services/security-identity-compliance/secrets-manager-vs-parameter-store.md), [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q30 — Domain 3

**Correct Answer: A**

**Explanation:** Dynamic references to Secrets Manager (or SSM Parameter Store) are exactly
CloudFormation's documented mechanism for this: the template contains only a reference string, not
the secret, and CloudFormation resolves the actual value at deploy time without persisting the
plaintext into the template, change set, or certain parts of the stack's stored configuration —
directly satisfying "never in source control or template" for the actual secret value.

**Why the others are wrong:**
- B: A comment is not a technical control and does nothing to prevent the plaintext key from being stored in the template and version control history, where it already is.
- C: Base64 is an encoding, not encryption — it is trivially reversible and provides no actual confidentiality; this is a common, important exam distinction.
- D: `NoEcho` masks a parameter value from being *displayed* in the console/CLI output, but the underlying template would still need to not contain the plaintext secret as a literal `Default`; `NoEcho` alone, without also removing the literal secret from the template, is an incomplete fix and is commonly over-trusted.

**Study links:** [`services/security-identity-compliance/secrets-manager-vs-parameter-store.md`](../../../../services/security-identity-compliance/secrets-manager-vs-parameter-store.md), [`services/management-governance/cloudformation.md`](../../../../services/management-governance/cloudformation.md)

---

## Q31 — Domain 4

**Correct Answer: A**

**Explanation:** Config's continuous evaluation model (which conformance packs bundle as a reusable set
of rules) is specifically designed to flag new or changed resources as their configuration is
recorded, not just resources that existed at deployment time — this is the core distinction from
a periodic manual audit, which could leave a non-compliant resource undetected for up to the full
audit interval (here, up to a quarter) rather than minutes to hours.

**Why the others are wrong:**
- B: This is factually wrong — Config rules (including those in a conformance pack) evaluate newly created and changed resources continuously, not only resources that existed at deployment time.
- C: This mischaracterizes the actual, meaningfully different detection latency between continuous automated evaluation and an infrequent manual audit — they are not functionally identical in practice.
- D: Config evaluation runs automatically based on configuration-change events (and periodically); it does not require a developer to manually trigger an evaluation after every resource creation.

**Study links:** [`services/management-governance/aws-config.md`](../../../../services/management-governance/aws-config.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q32 — Domain 1

**Correct Answer: A and B**

**Explanation:** Usage plans define the actual throttle/quota numbers (and can differ per tier), while
API keys are the per-customer identity that's associated with a usage plan so API Gateway can meter
and enforce that specific customer's consumption against their assigned plan's limits — this
two-part model (plan = limits, key = identity/association) is exactly how tiered public API access
control works on API Gateway.

**Why the others are wrong:**
- C: An API key with no usage plan attached does not enforce any throttling/quota by itself — the usage plan is what actually carries the limits; the key is just the identifier associated with a plan.
- D: Usage plans work with any API Gateway-published API (public or private); this statement about a VPC-only restriction is incorrect.
- E: A customer can be moved to a different usage plan by re-associating their *existing* API key with the new plan — a new key is not required just to change tiers.

**Study links:** [`services/networking-content-delivery/api-gateway.md`](../../../../services/networking-content-delivery/api-gateway.md)

---

## Q33 — Domain 2

**Correct Answer: A**

**Explanation:** Macie and Inspector are complementary, purpose-specific services: Macie classifies and
discovers sensitive data at rest in S3, while Inspector automatically and continuously scans EC2
instances, running container images in ECR, and Lambda functions for known software
vulnerabilities (CVEs) and network reachability issues — together covering both of the distinct
needs described, neither of which the other fully replaces.

**Why the others are wrong:**
- B: GuardDuty is a threat-detection service analyzing activity logs for malicious behavior; it does not perform content-level sensitive-data discovery (Macie's job) or systematic CVE vulnerability scanning (Inspector's job).
- C: AWS Config evaluates resource *configuration* against rules; it has no sensitive-data content scanning capability and no CVE vulnerability database scanning capability.
- D: Trusted Advisor provides general best-practice checks (cost, fault tolerance, some security configuration checks); it is not a CVE/vulnerability scanning engine for running workloads — that is specifically Inspector's purpose.

**Study links:** [`services/security-identity-compliance/macie.md`](../../../../services/security-identity-compliance/macie.md), [`services/security-identity-compliance/security-hub-and-inspector.md`](../../../../services/security-identity-compliance/security-hub-and-inspector.md)

---

## Q34 — Domain 1

**Correct Answer: A and B**

**Explanation:** Standard Workflows are the correct fit here specifically because of the stated
requirements — up to a year of duration, exactly-once semantics, and full execution history in the
console — none of which Express Workflows provide, since Express targets high-throughput,
short-duration (≤5 minute), at-least-once workloads instead. (Note: the fifth statement about
per-transition pricing is also factually true of Standard Workflows, but only two options are
required for this question — the two listed above are the ones that directly justify the choice.)

**Why the others are wrong:**
- C: Express Workflows are capped at 5 minutes of execution duration, not one year — this statement is factually wrong and is the opposite of Standard's duration limit.
- D: Standard Workflows provide exactly-once execution semantics, not at-least-once — this statement incorrectly swaps the two workflow types' characteristics.
- E: This statement is actually true of Standard Workflows (state-transition-based pricing) but is not one of the two required answers for this specific question about duration/semantics/history — it's included as a plausible-sounding but non-selected true fact to test careful reading of 'choose TWO'.

**Study links:** [`services/application-integration/step-functions.md`](../../../../services/application-integration/step-functions.md)

---

## Q35 — Domain 3

**Correct Answer: A**

**Explanation:** `sam local invoke --event <file>` is exactly built for feeding a crafted or
template-based test event (SAM ships sample event payload templates per trigger type, including
SNS) into a locally-run handler, letting a developer validate parsing/handling logic against a
realistic payload shape without deploying anything — directly matching Task 3.2's "use mock APIs
and test events to exercise Lambda functions without live dependencies."

**Why the others are wrong:**
- B: The event payload is explicitly developer-supplied via the `--event` flag (or stdin); it is not randomly generated, which is what makes this command useful for targeted, repeatable testing.
- C: Deploying to production just to test event-handling logic is unnecessary, slow, and risky compared to local testing with a representative test event — exactly what SAM's local tooling avoids.
- D: `sam package` prepares and uploads a deployment artifact for an actual deployment; it has nothing to do with locally simulating event invocations.

**Study links:** [`services/developer-tools/sam.md`](../../../../services/developer-tools/sam.md)

---

## Q36 — Domain 4

**Correct Answer: A**

**Explanation:** DAX is specifically purpose-built as a DynamoDB-native caching layer: it's
API-compatible with the existing DynamoDB SDK calls (so the application code barely changes),
delivers microsecond read latency, and manages cache population and invalidation internally —
directly matching "no hand-rolled cache-aside logic, no API rewrite, DynamoDB-specific" far more
precisely than a general-purpose cache like ElastiCache would without custom application code.

**Why the others are wrong:**
- B: ElastiCache for Redis is a valid general-purpose cache, but using it would require the application to switch to a separate client and hand-write cache-aside/invalidation logic — exactly what the requirement explicitly wants to avoid, unlike DAX's drop-in API compatibility.
- C: CloudFront caches HTTP(S) content at edge locations for content delivery; it has no native integration with or awareness of DynamoDB API calls.
- D: Global Accelerator improves network routing/availability for application endpoints; it has no database query caching capability at all.

**Study links:** [`services/database/dynamodb.md`](../../../../services/database/dynamodb.md), [`services/database/elasticache.md`](../../../../services/database/elasticache.md)

---

## Q37 — Domain 2

**Correct Answer: A**

**Explanation:** RDS is explicitly a managed database service — AWS takes on patching the underlying
engine software and host OS as part of "security OF the cloud," which is the key differentiator
from self-managed EC2-hosted databases where the customer would be responsible for all of that
patching themselves. The customer remains responsible for everything else listed (network access
config, IAM, credential hygiene) as "security IN the cloud."

**Why the others are wrong:**
- B: Security group configuration is explicitly the customer's responsibility under the shared responsibility model for any VPC-attached resource, including RDS.
- C: IAM policy design for application roles is always the customer's responsibility — AWS never decides an application's own permission scoping for them.
- D: Credential strength/rotation practices for the database's own master user are the customer's operational responsibility, even though RDS can assist (e.g. integration with Secrets Manager rotation).

**Study links:** [`services/security-identity-compliance/shared-responsibility-model-deep-dive.md`](../../../../services/security-identity-compliance/shared-responsibility-model-deep-dive.md)

---

## Q38 — Domain 1

**Correct Answer: A**

**Explanation:** Subscriptions are AppSync's dedicated real-time feature: a client issues a GraphQL
subscription, AppSync maintains a managed WebSocket connection, and whenever a mutation that
matches the subscription fires, AppSync pushes the updated data to every subscribed client
immediately — exactly the "instant push, no client polling" requirement.

**Why the others are wrong:**
- B: Sub-second client polling wastes bandwidth/battery and still introduces polling-interval latency; it's the pattern subscriptions exist to replace.
- C: A 5-minute scheduled email is nowhere near 'the instant it changes' and is a batch, not real-time, notification mechanism.
- D: API Gateway WebSocket APIs are a valid way to build custom real-time features, but this statement is factually wrong that AppSync has no native equivalent — Subscriptions is exactly that equivalent, and is the documented Task 1.1 AppSync real-time pattern.

**Study links:** [`services/application-integration/appsync.md`](../../../../services/application-integration/appsync.md)

---

## Q39 — Domain 3

**Correct Answer: A**

**Explanation:** Privileged mode is specifically documented as required when a CodeBuild buildspec
needs to interact with a Docker daemon (building/pushing images) — it's an explicit opt-in project
setting precisely because running a nested Docker daemon needs elevated container privileges that
aren't granted by default.

**Why the others are wrong:**
- B: This is incorrect — building/pushing Docker images from CodeBuild specifically fails with daemon-connection errors unless privileged mode is enabled; it is not a default-on capability.
- C: Docker build support is unrelated to which source provider (CodeCommit, GitHub, S3, etc.) triggers the build; this is a red herring not tied to the actual requirement.
- D: Compute type (general1.small/medium/large, etc.) affects CPU/memory available to the build, not whether a Docker daemon is running inside the container — that's specifically privileged mode's job.

**Study links:** [`services/developer-tools/codebuild.md`](../../../../services/developer-tools/codebuild.md), [`services/containers/ecr.md`](../../../../services/containers/ecr.md)

---

## Q40 — Domain 4

**Correct Answer: A**

**Explanation:** Exactly like every other AI-assisted pattern tested on DVA-C03 (test generation, deployment
approvals, error analysis), AI-assisted optimization accelerates the *analysis* step — surfacing
candidate changes faster than manual review of every resource — but a human still verifies and
approves before changes hit production, since the AI output can miss context (burst needs, SLA
margins) that only someone with full workload knowledge would catch.

**Why the others are wrong:**
- B: Blind, no-review auto-application risks introducing real production regressions (e.g. undersizing a Lambda that actually needs burst memory for occasional spikes) — the documented pattern always keeps a human verification step before production changes.
- C: Dismissing AI-assisted analysis entirely contradicts Task 4.3's explicit inclusion of 'AI-assisted optimization recommendations' as in-scope exam content — the pattern is to use AI assistance *with* review, not to reject it outright.
- D: Skipping review of most recommendations just because one was applied doesn't constitute verification of the ones actually being changed, and 'assuming the AI prioritized correctly' is exactly the unverified trust the documented pattern says to avoid.

**Study links:** [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md), [`domain-1-development/ai-assisted-development.md`](../../domain-1-development/ai-assisted-development.md), [`services/management-governance/cost-management-tools.md`](../../../../services/management-governance/cost-management-tools.md)

---

## Q41 — Domain 2

**Correct Answer: A**

**Explanation:** WAF's AWS Managed Rules are exactly built for this: pre-built, continuously updated
rule groups (including a SQL-injection-focused rule group) maintained by AWS, attached to a web ACL
on CloudFront/API Gateway/ALB — giving "blocked automatically with no custom regex to maintain,"
which is the documented alternative to hand-rolling detection logic.

**Why the others are wrong:**
- B: Writing and maintaining custom SQL injection detection logic by hand duplicates exactly what AWS Managed Rules already provides, with ongoing maintenance burden the question explicitly wants to avoid.
- C: Shield Standard protects against common network/transport-layer DDoS patterns automatically; it has no application-layer rule groups for SQL injection — that's WAF's domain.
- D: GuardDuty is a detection service analyzing logs/activity for threats; it does not sit inline blocking HTTP requests at the edge the way a WAF web ACL does.

**Study links:** [`services/security-identity-compliance/waf.md`](../../../../services/security-identity-compliance/waf.md)

---

## Q42 — Domain 1

**Correct Answer: A and B**

**Explanation:** Aurora's storage architecture (distributed, auto-scaling, replicated across 3 AZs by
default) and its replica-failover design (replicas share the same underlying storage, so promotion
doesn't require replaying a replication stream) are the two headline differentiators that make it
both easier to scale and typically faster to fail over than standard RDS Multi-AZ — directly
matching the stated "scale significantly" and "fast failover" requirements.

**Why the others are wrong:**
- C: This is factually false — Aurora's storage auto-scales up to very large sizes with no manual resizing, and failover is generally faster due to the shared-storage replica design; they are not identical.
- D: Aurora clusters are inherently multi-AZ by design (storage replicated across 3 AZs); this statement inverts the actual resilience characteristics.
- E: There is no such automatic conversion feature between RDS and Aurora based on usage thresholds — migrating requires an explicit, deliberate action (e.g. snapshot migration).

**Study links:** [`services/database/aurora.md`](../../../../services/database/aurora.md), [`services/database/rds.md`](../../../../services/database/rds.md), [`services/database/database-selection-guide.md`](../../../../services/database/database-selection-guide.md)

---

## Q43 — Domain 3

**Correct Answer: A**

**Explanation:** A genuinely separate pipeline (or independently triggered CodeBuild project) scoped to
feature branches, containing only build/test stages with no deployment capability at all, cleanly
separates "lower-stakes CI for in-progress work" from "the pipeline that can reach production" —
avoiding any risk that a feature-branch trigger could accidentally exercise a production deployment
stage, which a single shared pipeline with conditional logic risks if the condition is ever
misconfigured.

**Why the others are wrong:**
- B: A single pipeline with a silent conditional no-op for deployment still includes the deploy stage's permissions/configuration in a pipeline that feature-branch pushes can trigger — a misconfigured condition could accidentally deploy from a feature branch, which a fully separate pipeline avoids structurally.
- C: Renaming branches as a workaround is operationally absurd and doesn't actually create the described separate, lower-stakes CI process.
- D: Disabling the entire production pipeline whenever any feature branch exists would block legitimate production deployments constantly, since feature branches typically exist most of the time in active development.

**Study links:** [`services/developer-tools/codepipeline.md`](../../../../services/developer-tools/codepipeline.md), [`services/developer-tools/codecommit.md`](../../../../services/developer-tools/codecommit.md)

---

## Q44 — Domain 4

**Correct Answer: A**

**Explanation:** A function missing from a trace despite being invoked is a classic trace-context-propagation
or tracing-not-enabled problem: X-Ray only emits a segment for a function if active tracing is
turned on for it and the trace ID context is actually propagated to it (which can silently break
across certain async invocation patterns) — CloudWatch Logs confirming invocation doesn't imply
tracing is configured, since logging and tracing are separate, independently configured features.

**Why the others are wrong:**
- B: There is no such fixed 3-function-per-trace limit in X-Ray; traces can include many more segments than that.
- C: Logging and tracing are independent systems — appearing in CloudWatch Logs says nothing about whether X-Ray active tracing is enabled for that function or whether trace context reached it.
- D: X-Ray explicitly supports Lambda tracing (it's one of the most common ways X-Ray is used) in addition to EC2/ECS/other compute; this statement is factually wrong.

**Study links:** [`domain-4-troubleshooting-optimization/observability-instrumentation.md`](../../domain-4-troubleshooting-optimization/observability-instrumentation.md), [`services/compute/lambda.md`](../../../../services/compute/lambda.md)

---

## Q45 — Domain 2

**Correct Answer: A and B**

**Explanation:** Automated remediation requires both pieces working together: Security Hub gives
centralized visibility/aggregation (useful for reporting and prioritization) but does not itself
execute remediation actions, while an EventBridge rule matching on the GuardDuty finding event is
the actual trigger that invokes a remediation target (Lambda, SSM Automation, etc.) — Security Hub
aggregation and the EventBridge-driven trigger are the two distinct, necessary pieces for this
no-human-in-the-loop workflow.

**Why the others are wrong:**
- C: Security Hub centralizes and prioritizes findings for human/dashboard consumption; it does not independently execute remediation actions without an EventBridge rule (or similar) wired to an actual target.
- D: GuardDuty findings are explicitly designed to be consumed as EventBridge events — this is the standard, documented integration path for automated response, so this statement is factually wrong.
- E: Security Hub is a findings-aggregation and posture-management service; it does not generate GuardDuty's own threat-detection analytics and does not replace GuardDuty — disabling GuardDuty would remove the actual detection capability entirely.

**Study links:** [`services/security-identity-compliance/guardduty.md`](../../../../services/security-identity-compliance/guardduty.md), [`services/security-identity-compliance/security-hub-and-inspector.md`](../../../../services/security-identity-compliance/security-hub-and-inspector.md)

---

## Q46 — Domain 1

**Correct Answer: A**

**Explanation:** Step Functions' `waitForTaskToken` pattern is exactly built for this: it pauses the
state machine at that step (consuming no compute while waiting, unlike a sleeping Lambda) until an
external process calls `SendTaskSuccess`/`SendTaskFailure` with the matching token, and the whole
sequence — including long external waits — is visually represented and auditable per execution in
the Step Functions console, directly satisfying both the long-wait and auditability requirements.

**Why the others are wrong:**
- B: Lambda has a hard 15-minute maximum execution duration — it physically cannot sleep for 48 hours in a single invocation, and even if it could, it would be needlessly billed for idle wait time.
- C: SQS has no concept of a visual, auditable multi-step state machine with long external waits; it's a message queue, not a workflow orchestrator.
- D: Polling every second for 48 hours (over 172,000 calls) is extremely wasteful and unnecessary compared to an event-driven callback via task tokens.

**Study links:** [`services/application-integration/step-functions.md`](../../../../services/application-integration/step-functions.md)

---

## Q47 — Domain 1

**Correct Answer: A**

**Explanation:** Cluster Mode Enabled is specifically Redis's horizontal-scaling mechanism in
ElastiCache: the keyspace is partitioned into hash slots distributed across multiple shards (node
groups), letting total cluster capacity scale well beyond any single node's memory while
maintaining Redis's characteristic low-latency operations — directly solving "dataset bigger than
one node can hold."

**Why the others are wrong:**
- B: This both overstates Memcached (which does support client-side/consistent-hash-based sharding, not something fundamentally beyond Redis) and understates Redis, which explicitly supports cluster-mode sharding — the comparison as stated is incorrect.
- C: Vertically scaling a single node has a hard ceiling (the largest available instance type) and doesn't provide horizontal scale-out; Redis absolutely does support multi-node sharding via Cluster Mode.
- D: Multi-AZ (replication group failover) provides high availability for a given shard's data, not horizontal partitioning of the dataset across more capacity — Cluster Mode is the distinct feature needed for that.

**Study links:** [`services/database/elasticache.md`](../../../../services/database/elasticache.md)

---

## Q48 — Domain 2

**Correct Answer: A**

**Explanation:** Management events (enabled by default on a trail) capture control-plane operations
like creating/deleting a bucket or changing its policy — they do **not** capture per-object
data-plane calls like `GetObject`. Data events are the specific, opt-in CloudTrail feature for
object-level S3 (and Lambda invoke, DynamoDB item-level, etc.) logging, which is exactly what's
needed to audit individual object reads by principal and time.

**Why the others are wrong:**
- B: This is factually incorrect — management events do not include individual object-level `GetObject`/`PutObject` calls; that is specifically what data events add.
- C: VPC Flow Logs capture IP-layer traffic metadata (source/destination IP, port, bytes) at the network interface level, not which S3 API action was called or by which IAM principal.
- D: S3 Versioning tracks object version history for overwrites/deletes; it has no concept of logging *read* access by principal — that is a CloudTrail data events / S3 server access logging concern, not Versioning.

**Study links:** [`services/management-governance/cloudtrail.md`](../../../../services/management-governance/cloudtrail.md), [`services/storage/s3.md`](../../../../services/storage/s3.md)

---

## Q49 — Domain 3

**Correct Answer: A**

**Explanation:** ECS with Fargate is exactly the combination for "container orchestration, zero EC2/
control-plane management, no Kubernetes tooling required": ECS has its own task-definition model
entirely separate from Kubernetes, and the Fargate launch type removes all EC2 instance management
for either ECS or EKS — but since no Kubernetes API/tooling is wanted here, plain ECS (not EKS) is
the right orchestrator.

**Why the others are wrong:**
- B: Fargate is available as a launch type for *both* ECS and EKS — it does not require EKS specifically, and choosing EKS here would needlessly introduce the Kubernetes API/tooling the requirement explicitly wants to avoid.
- C: This is backwards — Fargate's entire purpose is to remove the need for any self-managed EC2 instances; the EC2 launch type (not Fargate) is the one that requires the team to manage EC2 instances themselves.
- D: Lambda has its own packaging/invocation model (ZIP or Lambda-specific container image contract) and does not directly run standard ECS task definitions — this statement is factually incorrect.

**Study links:** [`services/containers/ecs.md`](../../../../services/containers/ecs.md), [`services/containers/fargate.md`](../../../../services/containers/fargate.md), [`services/containers/containers-vs-vms-vs-serverless.md`](../../../../services/containers/containers-vs-vms-vs-serverless.md)

---

## Q50 — Domain 4

**Correct Answer: A**

**Explanation:** Because Lambda scales allocated CPU/vCPU proportionally with configured memory, a
CPU-bound workload (as this profiling shows — most time in actual computation, not waiting on I/O)
is exactly the profile where raising memory tends to meaningfully cut wall-clock duration, directly
addressing both "timeout risk" and "shorten the run" in one lever — a much more targeted fix than
adjusting timeout or trigger type, neither of which changes available compute.

**Why the others are wrong:**
- B: Timeout only controls how long the function is allowed to run before being killed; it does not add CPU or make computation faster — raising it alone doesn't shrink the run window, it just permits a longer one.
- C: Trigger type (schedule vs. event) determines *when/how* a function is invoked, not how much CPU/memory is allocated to a given invocation — this is unrelated to the performance profile described.
- D: Blindly minimizing memory ignores the stated CPU-bound profile; for this workload, lower memory would likely make duration (and often even total cost, given the duration/cost tradeoff) worse, not better.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q51 — Domain 1

**Correct Answer: A**

**Explanation:** EventBridge scheduled rules (cron or rate expressions) are the documented, fully
managed way to invoke a target like Lambda on a recurring schedule, with zero servers to run or
maintain — exactly matching "no EC2 instance just to fire a nightly job," which is a Task 1.1
event-driven-application pattern applied to scheduling.

**Why the others are wrong:**
- B: Lambda has a 15-minute maximum execution duration — it cannot sleep in a loop waiting for the next day; this also wastes billed compute time sitting idle.
- C: Keeping an EC2 instance running 24/7 just to fire one daily job defeats the serverless, no-idle-infrastructure goal entirely.
- D: API Gateway usage plan quotas limit *how many* requests a client can make; they don't actively trigger an invocation on a schedule — this conflates two unrelated concepts.

**Study links:** [`services/application-integration/eventbridge.md`](../../../../services/application-integration/eventbridge.md)

---

## Q52 — Domain 2

**Correct Answer: A**

**Explanation:** Cognito identity pools support two distinct IAM roles — one for unauthenticated,
one for authenticated identities — and the least-privilege design here scopes the unauthenticated
role to only read-only catalog access (guests have no identity-specific data to protect/write),
while the authenticated role adds narrowly scoped write access to only the signed-in user's own
cart data (e.g. via a `dynamodb:LeadingKeys` condition tying access to their Cognito identity ID),
directly matching the stated requirements for each user class.

**Why the others are wrong:**
- B: Blanket `dynamodb:*` access for both roles is a severe least-privilege violation, letting a guest read or write arbitrary data far beyond the public catalog.
- C: Identity pools explicitly support two separate roles (unauthenticated and authenticated) specifically so each class of user can have different permissions — this statement is factually wrong.
- D: Allowing unauthenticated guests to write to a cart table they have no durable identity tied to creates orphaned, unattributable data and exceeds what an anonymous guest role should be trusted with.

**Study links:** [`services/security-identity-compliance/cognito.md`](../../../../services/security-identity-compliance/cognito.md)

---

## Q53 — Domain 3

**Correct Answer: A**

**Explanation:** This matches the documented AI-assisted development pattern across every domain it
appears in (code review, test generation, deployment approvals, error analysis): AI accelerates
and surfaces useful signal, but a human remains the actual decision-maker/gate — here, that means
AI comments inform the reviewer rather than AI output automatically merging code or replacing the
human approval requirement.

**Why the others are wrong:**
- B: Auto-merging based solely on an AI tool's silence removes the human-in-the-loop safeguard entirely, risking merged code that an AI tool simply failed to flag an issue in.
- C: Fully removing human review assumes AI review is a complete substitute for human judgment, which is explicitly not the documented pattern — AI assists, it doesn't replace the human reviewer.
- D: Letting an AI tool silently commit its own fixes without author/reviewer involvement bypasses the entire review process and accountability chain for a code change.

**Study links:** [`domain-1-development/ai-assisted-development.md`](../../domain-1-development/ai-assisted-development.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q54 — Domain 4

**Correct Answer: A**

**Explanation:** Cost allocation tags are exactly the documented mechanism for this: consistently
tagging every resource involved in a given feature/project (regardless of resource type), then
activating that tag key for cost allocation in Billing, lets Cost Explorer (and Cost and Usage
Reports) break down and filter spend by that tag — enabling exactly the "cost of this one feature
across Lambda + S3 + Step Functions" attribution the team wants.

**Why the others are wrong:**
- B: Budgets has no automatic knowledge of which resources belong to a feature; it (like Cost Explorer) relies on cost allocation tags or filters to scope spend meaningfully — it doesn't infer feature ownership on its own.
- C: This is factually wrong — AWS explicitly supports fine-grained cost attribution via tags (and Cost and Usage Reports), specifically for cases like this where spend spans multiple resource types for one logical feature.
- D: Log volume is not a reliable or direct proxy for dollar cost across heterogeneous resource types (Lambda duration/memory, S3 storage/requests, Step Functions transitions each price differently) — this isn't a valid cost-attribution method.

**Study links:** [`services/management-governance/cost-management-tools.md`](../../../../services/management-governance/cost-management-tools.md)

---

## Q55 — Domain 1

**Correct Answer: A**

**Explanation:** `INCLUDE` lets a GSI project only the specific non-key attributes actually needed by
queries against it, minimizing both the extra storage consumed by the index and the write-capacity
cost of keeping it in sync on every base-table write — the right choice here since exactly three
non-key attributes are needed and nothing more, which is cheaper than `ALL` and avoids `KEYS_ONLY`'s
need for a second round-trip per result.

**Why the others are wrong:**
- B: `ALL` costs more in storage and per-write overhead for attributes the query never uses; it's not universally 'better' when a narrower set of attributes is known to be sufficient.
- C: `KEYS_ONLY` would require a follow-up `GetItem` per item to fetch `name` and `price`, adding latency and extra read capacity consumption that `INCLUDE` avoids entirely by projecting those specific attributes directly into the index.
- D: This is factually incorrect — GSIs explicitly support `KEYS_ONLY`, `INCLUDE`, and `ALL` projection types specifically to let developers control exactly this tradeoff.

**Study links:** [`services/database/dynamodb.md`](../../../../services/database/dynamodb.md)

---

## Q56 — Domain 2

**Correct Answer: A**

**Explanation:** Task 2.3's "keeping your inputs/outputs out of a provider's model training data" bullet
maps directly to Bedrock's actual documented data-handling model: AWS states that Amazon Bedrock
does not use customer content submitted through the API to train the underlying foundation models
or share it with model providers for their own improvement — this is the core fact to know for
this exam topic, not a feature flag that needs separate opt-out configuration per request.

**Why the others are wrong:**
- B: This option states the opposite of Bedrock's actual documented data-handling behavior and recommends an unnecessary (and impractical) workaround to a non-existent default-training-use problem.
- C: Bedrock's data-handling commitments specifically address this training-data concern for AWS-hosted use of the service; a self-managed model isn't required to get this assurance.
- D: `InvokeModel` requires the model to receive processable plaintext input to generate a summary — encrypting the document before sending it would make it unusable to the model, and this isn't actually how the training-data protection works (it's a service-level data-handling commitment, not something achieved by client-side encryption of the payload).

**Study links:** [`domain-2-security/ai-service-security-risks.md`](../../domain-2-security/ai-service-security-risks.md), [`services/machine-learning-ai/bedrock.md`](../../../../services/machine-learning-ai/bedrock.md)

---

## Q57 — Domain 1

**Correct Answer: A**

**Explanation:** S3 Lifecycle rules are exactly the native, no-code mechanism for this: transition
actions move objects between storage classes at defined ages, and expiration actions delete
objects at a defined age — both evaluated and executed by S3 itself in the background, directly
matching "90 days standard → archival for 9 months → delete at 1 year" with zero custom code.

**Why the others are wrong:**
- B: Hand-rolling a daily scan-and-migrate Lambda duplicates Lifecycle's built-in functionality with far more code, cost (listing/copying millions of objects), and operational risk.
- C: S3 Versioning has no automatic 90-day deletion behavior by default; versions persist until an explicit lifecycle rule or deletion removes them — this option misstates Versioning's behavior.
- D: Building a custom retention-tracking table and gating every read on it reinvents what Lifecycle already does natively, with added latency and a new system to maintain.

**Study links:** [`services/storage/s3.md`](../../../../services/storage/s3.md)

---

## Q58 — Domain 3

**Correct Answer: A**

**Explanation:** This is exactly Task 3.1's "dependency management (bundling, layers, lockfiles)"
scenario: a layer is the correct zip-based mechanism for sharing dependencies across multiple
functions, shrinking each function's own package to just its unique code, and centralizing
version bumps to one layer publish rather than five separate full re-uploads.

**Why the others are wrong:**
- B: This claim is false — layers are precisely Lambda's built-in zip-based code-sharing mechanism, which does not require switching to container image packaging at all.
- C: Merging five independent functions into one package with manual dispatch logic conflates unrelated responsibilities, loses independent scaling/permissions/versioning per function, and doesn't actually solve the shared-dependency-versioning problem described.
- D: Re-downloading 50 MB from S3 on every single invocation adds significant latency to every cold *and* warm invocation and is far worse than a layer, which is attached once per execution environment lifecycle, not re-fetched per call.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q59 — Domain 4

**Correct Answer: A**

**Explanation:** RDS Proxy is purpose-built for exactly this Lambda-to-RDS connection-management
problem: it maintains a pooled set of actual database connections and multiplexes many concurrent
Lambda executions' logical connections over that smaller pool, directly decoupling "how many Lambda
invocations are running" from "how many real database connections exist" — solving the exhaustion
problem without artificially throttling the function's overall concurrency.

**Why the others are wrong:**
- B: Capping reserved concurrency to 1 would solve the connection problem by destroying the function's ability to scale at all for its entire workload — exactly the tradeoff the question says to avoid.
- C: Storage size and maximum connection count are different, largely independent RDS parameters; increasing storage does not directly raise the connection ceiling.
- D: Opening a brand-new connection on every single invocation (no reuse at all) would make the connection-exhaustion problem worse, not better, since even warm execution environments would stop reusing an existing connection.

**Study links:** [`services/database/rds.md`](../../../../services/database/rds.md), [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q60 — Domain 2

**Correct Answer: A**

**Explanation:** Task 2.3's logging concern is addressed the same way any other sensitive-data-in-logs
problem is: redact/mask known sensitive patterns before they're persisted, and apply standard
least-privilege access controls to whatever log data remains — AI service call logs aren't exempt
from the general Task 2.2/2.3 principle of not persisting PII in plaintext where it isn't needed and
isn't tightly access-controlled.

**Why the others are wrong:**
- B: Logging is valuable for debugging and support quality; the documented fix is redaction/access control, not eliminating all observability for the feature.
- C: CloudWatch Logs is a general-purpose log store with no built-in PII exemption or automatic redaction — storing PII there in plaintext carries the same compliance exposure as storing it in any other unprotected store.
- D: User input is exactly where users are described as pasting PII (account numbers) in this scenario — restricting redaction to only the model's output would miss the actual source of the sensitive data described.

**Study links:** [`domain-2-security/ai-service-security-risks.md`](../../domain-2-security/ai-service-security-risks.md), [`services/management-governance/cloudwatch.md`](../../../../services/management-governance/cloudwatch.md)

---

## Q61 — Domain 1

**Correct Answer: A**

**Explanation:** WebSocket APIs are API Gateway's purpose-built feature for persistent, bidirectional
client connections: the `$connect` route event includes the new `connectionId`, which the app
typically persists in DynamoDB; later, the backend can push a message to any specific stored
`connectionId` via the API Gateway Management API's `PostToConnection` action — exactly the
documented "push to a specific connected client" chat pattern.

**Why the others are wrong:**
- B: REST APIs are request/response over HTTP; they have no concept of a persistent, server-push-capable connection — that's specifically what the WebSocket API type adds.
- C: SNS delivers to subscribed endpoints (SQS, Lambda, HTTP(S), etc.) per publish; it does not maintain a persistent connection per end-user client the way a WebSocket API does, and isn't designed for per-connection targeted pushes like this.
- D: Sub-second polling from every client is far less efficient and far higher latency than a true push-based WebSocket connection, and is exactly the pattern WebSocket APIs exist to replace.

**Study links:** [`services/networking-content-delivery/api-gateway.md`](../../../../services/networking-content-delivery/api-gateway.md)

---

## Q62 — Domain 3

**Correct Answer: A**

**Explanation:** CDK explicitly supports multiple Stack instances within one app, parameterized by
environment-specific props/context and each targeting an explicit `env` (account/Region) —
instantiating one Stack subclass per environment with clear, explicit names (rather than branching
on ambient state inside a single shared stack) is the documented pattern for making "which
environment am I about to deploy to" unambiguous and reduces the risk of accidentally deploying
dev-shaped resources to prod or vice versa.

**Why the others are wrong:**
- B: CDK apps commonly define many Stack instances; this statement is factually wrong about CDK's capabilities.
- C: Defaulting to production and requiring manual override for every non-prod deploy is backwards and dangerous — the safer default is the opposite, or (better) explicit, unambiguous stack identifiers per environment as described in the correct answer.
- D: Manually editing resources in the console for production specifically reintroduces drift between CDK's view of the stack and reality, undermining IaC entirely for the most important environment.

**Study links:** [`services/management-governance/cdk.md`](../../../../services/management-governance/cdk.md), [`domain-3-testing-deployment/README.md`](../../domain-3-testing-deployment/README.md)

---

## Q63 — Domain 4

**Correct Answer: A**

**Explanation:** Only an actual empirical test against a representative payload (which is exactly what
Lambda Power Tuning automates — invoking at each candidate memory value and reporting real
duration/cost) gives workload-specific evidence; generic advice or blog-post defaults can be wrong
for a given function's specific CPU/I/O profile, which is precisely why the exam's documented
Task 4.3 guidance is to benchmark rather than assume.

**Why the others are wrong:**
- B: A generic 'common default' ignores this specific function's actual behavior and may be far from its real optimal point — exactly the untested guess the question wants to avoid.
- C: Duration vs. memory is frequently non-linear and workload-dependent (I/O-bound workloads may see little to no improvement); assuming strict linearity is an unverified and often incorrect assumption.
- D: More memory does not strictly guarantee faster performance for every workload profile, and it always increases per-GB-second cost — 'always max, never measure' is not a sound strategy and can raise cost without any performance benefit for some workloads.

**Study links:** [`services/compute/lambda.md`](../../../../services/compute/lambda.md), [`domain-4-troubleshooting-optimization/README.md`](../../domain-4-troubleshooting-optimization/README.md)

---

## Q64 — Domain 2

**Correct Answer: A**

**Explanation:** This is exactly Task 2.3's "AI agent security" principle in action: an agent should
never be *able* to attempt actions beyond its intended scope, and relying on an incidental,
undocumented permission gap to prevent real damage is not a security control — it's luck. The
correct fix is deliberate least-privilege scoping of the agent's execution role (so sensitive
actions like refunds simply aren't reachable from a status-inquiry use case) combined with
guardrails/input handling so manipulated prompts are also caught before an unauthorized tool call
is even attempted.

**Why the others are wrong:**
- B: Treating an accidental permission gap as 'working as intended' ignores that the agent was successfully manipulated into attempting an unauthorized action — the control that happened to stop it wasn't deliberate, and a differently configured role (or a different exploited action group) could easily have succeeded.
- C: Granting full permissions on both action groups removes the only thing that currently prevents the successful exploit from actually completing — this is the opposite of the correct fix.
- D: IAM scoping is precisely the control that limited the blast radius here; removing it entirely would make a successful prompt-injection exploit fully effective rather than blocked.

**Study links:** [`domain-2-security/ai-service-security-risks.md`](../../domain-2-security/ai-service-security-risks.md), [`services/machine-learning-ai/bedrock.md`](../../../../services/machine-learning-ai/bedrock.md)

---

## Q65 — Domain 1

**Correct Answer: A and B**

**Explanation:** This is the canonical SNS fan-out pattern precisely because it (a) decouples the
publisher entirely from how many or which services ultimately consume the event, and (b) gives each
consumer its own independent queue, so one slow or failing consumer's backlog has zero effect on
the others' throughput — exactly the stated requirement of independent per-microservice pacing.
(Statement 5 is also factually true of this architecture but isn't one of the two required answers
for what makes it the right choice here.)

**Why the others are wrong:**
- C: Each subscribed queue receives its own independent copy of the message; processing in one queue has no effect on the copies in the other two queues — there is no cross-queue "consumed once" behavior in a fan-out topology.
- D: SNS natively fans a single published message out to every subscribed endpoint (including multiple SQS queues) — the publisher sends exactly one message to the topic, not one per destination; this statement misstates how SNS fan-out works.
- E: True but not one of the two selected answers — it's a correct supporting detail about independent queue configuration, included to test careful 'choose exactly TWO' selection rather than picking every true-sounding option.

**Study links:** [`services/application-integration/sns.md`](../../../../services/application-integration/sns.md), [`services/application-integration/messaging-patterns-overview.md`](../../../../services/application-integration/messaging-patterns-overview.md)

---
