# Domain 1 Practice Questions — Development with AWS Services (30%)

15 questions covering Task 1.1 (SDKs/APIs, streaming, serialize/deserialize, event-driven apps,
API Gateway stages), Task 1.2 (Lambda: VPC, layers, concurrency, triggers), and Task 1.3 (data
stores: DynamoDB, S3, RDS/Aurora). See [`../domain-1-development/README.md`](../domain-1-development/README.md)
for the full domain outline. 2 questions use the **(Choose TWO)** multi-select format.

---

## Q1

A developer's application calls `DynamoDB:Query` against a table expected to return several
thousand items matching a partition key. The application code calls `Query` once and only
processes the items in the response, then moves on. In production, the application is silently
missing a large percentage of matching items. What is the most likely cause, and the correct fix?

A. DynamoDB silently drops items over 400 KB; the fix is to compress item attributes before writing them.

B. `Query` results are capped at 1 MB per call and the response includes a `LastEvaluatedKey` when more data remains; the fix is to loop on `Query` using `ExclusiveStartKey` until `LastEvaluatedKey` is absent.

C. The table is in eventually-consistent mode, so roughly half of all items are temporarily invisible; the fix is to enable strong consistency on the table.

D. The AWS SDK's default retry policy drops any item that required a retry; the fix is to disable retries in the SDK client configuration.

**Correct Answer: B**

**Explanation:** A single `Query` (or `Scan`) call returns at most 1 MB of data and sets
`LastEvaluatedKey` in the response whenever more matching items exist beyond that page. Application
code must paginate — loop, passing the previous response's `LastEvaluatedKey` as the next call's
`ExclusiveStartKey` — until `LastEvaluatedKey` is no longer returned. This is exactly Task 1.3's
"use the SDK correctly against each [data store]: client config, retries, pagination, batching."
See [`services/database/dynamodb.md`](../../../services/database/dynamodb.md) and the SDK usage
note in [`../domain-1-development/README.md`](../domain-1-development/README.md).

**Why the others are wrong:**
- A: DynamoDB's per-item size limit is 400 KB and exceeding it causes a write-time validation error, not a silent read-time drop — unrelated to this symptom.
- C: Consistency (eventual vs. strong) affects whether a read reflects the *most recent write*, not whether a single `Query` call returns all matching items across multiple pages — it never causes missing items across a whole result set.
- D: The SDK's default retry policy retries failed requests with backoff; it does not drop successfully-returned items, and disabling retries would make transient throttling failures worse, not fix pagination.

---

## Q2

A company ingests clickstream data from millions of devices and needs to process records in the
order they were produced, per device, while allowing multiple consumer applications to read the
same data independently (one for real-time dashboards, one for batch analytics). Which AWS service
is the best fit for the ingestion layer, and why?

A. Amazon SQS Standard queue — any consumer that needs the data can poll the same queue.

B. Amazon Kinesis Data Streams — records are grouped into shards by partition key (preserving per-key order) and multiple independent consumers can each read the full stream at their own pace.

C. Amazon SNS topic — it fans out every message to all subscribed consumers automatically.

D. AWS Step Functions — a state machine can track per-device order as workflow executions.

**Correct Answer: B**

**Explanation:** Kinesis Data Streams is built for streaming data ingestion with ordering
guarantees *within a shard* (records with the same partition key land in the same shard, preserved
in order) and supports multiple independent consumer applications reading the same stream
concurrently via separate shard iterators (or enhanced fan-out) — exactly Task 1.1's "work with
streaming data." See [`services/analytics/kinesis-family.md`](../../../services/analytics/kinesis-family.md).

**Why the others are wrong:**
- A: SQS Standard does not guarantee per-key ordering and a message is only visible to one consumer group at a time (it's a queue, consumed once), not multiple independent full-stream readers.
- C: SNS fans out to subscribers, but it has no concept of ordered, replayable streaming data per partition key — once delivered, a message isn't re-readable from an offset the way a Kinesis shard is.
- D: Step Functions orchestrates workflows/state transitions; it isn't an ingestion mechanism for high-volume streaming records at all.

---

## Q3

A Lambda function is integrated with API Gateway using the Lambda proxy integration. The function
runs without errors and CloudWatch Logs show the handler completing successfully and returning a
Python dict like `{"statusCode": 200, "body": {"orderId": "123"}}`. However, every client call
receives a generic `Internal Server Error` from API Gateway. What is the bug?

A. The Lambda execution role is missing `apigateway:*` permissions.

B. `body` must be a JSON-encoded **string**, not a raw dict/object — API Gateway's proxy integration expects `body` to already be serialized text.

C. `statusCode` must be a string ("200"), not an integer, in proxy integration responses.

D. API Gateway proxy integration requires the response to include a `headers` key even if empty, or it rejects the whole payload.

**Correct Answer: B**

**Explanation:** This is a classic serialize/deserialize trap under Task 1.1. The Lambda proxy
integration contract requires the return object's `body` field to be a JSON **string**
(`json.dumps({"orderId": "123"})`), because API Gateway passes that string through to the client
byte-for-byte rather than re-serializing it. Returning a raw object there breaks the integration
contract and API Gateway surfaces it as a 500. See the sequence walkthrough in
[`../domain-1-development/README.md`](../domain-1-development/README.md) (step 7) and
[`services/networking-content-delivery/api-gateway.md`](../../../services/networking-content-delivery/api-gateway.md).

**Why the others are wrong:**
- A: API Gateway invoking Lambda needs `lambda:InvokeFunction` permission (typically granted via resource policy, not an IAM role on the function), and that's unrelated to a 500 caused by a malformed response body — a permissions failure would show as a 403, not a generic 500 after the handler already ran successfully.
- C: `statusCode` is expected as an integer in the proxy response contract; sending a string there is not the documented requirement and isn't what breaks this scenario.
- D: `headers` is optional in the proxy response shape; its absence does not cause a 500.

---

## Q4 (Choose TWO)

An application must process an image immediately after it's uploaded to an S3 bucket. The
processing Lambda function occasionally experiences brief periods where downstream capacity is
limited, and the team wants uploads during a traffic spike to not be dropped or fail outright even
if Lambda can't immediately keep up. Which TWO design choices satisfy both requirements (trigger on
upload, and tolerate processing bursts without dropping work)?

A. Configure an S3 event notification that invokes the Lambda function directly and synchronously for every `PutObject`.

B. Configure an S3 event notification targeting an SQS queue, then have Lambda poll the queue as an event source — the queue buffers bursts and Lambda scales consumption at its own pace.

C. Configure an S3 event notification targeting EventBridge, with a rule that can apply additional filtering/fan-out before invoking Lambda.

D. Disable all S3 event notifications and instead have Lambda run on a fixed CloudWatch Events schedule to scan the bucket for new objects every minute.

E. Increase the S3 bucket's request rate limit so uploads never need buffering.

**Correct Answer: B and C**

**Explanation:** Both B and C satisfy "trigger immediately on upload" (S3 event notifications are
near-real-time) while adding a buffer/decoupling layer between the upload event and Lambda
execution. SQS (B) explicitly buffers bursts — Lambda's poller scales up consumers against queue
depth rather than being invoked once per object synchronously. EventBridge (C) adds routing/filtering
and can itself be paired with a queue downstream, and decouples the S3 event from a single rigid
invocation path. This is Task 1.1's event-driven application pattern — see
[`services/application-integration/eventbridge.md`](../../../services/application-integration/eventbridge.md),
[`services/application-integration/sqs.md`](../../../services/application-integration/sqs.md), and
[`services/storage/s3.md`](../../../services/storage/s3.md).

**Why the others are wrong:**
- A: Direct S3-to-Lambda invocation is asynchronous, not synchronous, but more importantly it has no buffer — if Lambda's concurrency is exhausted during a spike, events queue internally for a limited retry window and can eventually be dropped (sent to a DLQ if configured, or lost) rather than being durably buffered the way an SQS queue buffers them.
- D: Polling on a schedule defeats "process immediately after upload" — it introduces up to a full polling-interval delay and is a worse design than event notifications for this requirement.
- E: S3 request rate isn't the bottleneck described (Lambda's downstream capacity is) — raising an S3 limit does nothing to fix Lambda-side burst handling.

---

## Q5

A retail application needs several independent subsystems (inventory, shipping, notifications) to
each react to an `OrderPlaced` event, but only the subsystems that care about a specific order
`region` attribute should actually be invoked for that event. Which approach correctly implements
this in an event-driven architecture on AWS?

A. Publish the event to an SNS topic and have every subscriber inspect the `region` field in its own code before deciding whether to act.

B. Publish the event to an EventBridge event bus and create a separate rule per subsystem with an event pattern that matches on the `region` attribute, so EventBridge itself filters which targets get invoked.

C. Publish the event directly to each subsystem's Lambda function via separate synchronous `Invoke` calls from the order service.

D. Write the event to a DynamoDB table and have each subsystem poll the table every few seconds looking for new rows.

**Correct Answer: B**

**Explanation:** EventBridge is the central event bus built exactly for Task 1.1's "build
event-driven applications" with content-based filtering: rules use event patterns to match on
specific attributes (like `region`) and only invoke matching targets, so routing logic lives in the
bus's rule configuration rather than being duplicated in every consumer. See
[`services/application-integration/eventbridge.md`](../../../services/application-integration/eventbridge.md).

**Why the others are wrong:**
- A: SNS can filter at the subscription level via filter policies, but without a filter policy, every subscriber receives every message and must self-filter — this works but is a worse fit than EventBridge's richer pattern matching, and the option as described (inspecting in code) means the bus does no filtering, increasing invocation cost and coupling unnecessarily.
- C: Direct synchronous invokes tightly couple the order service to every subsystem's availability and address — it is the opposite of an event-driven, decoupled design, and adding a new subsystem requires changing the order service's code.
- D: Polling a table introduces latency and unnecessary read cost/load, and isn't an event-driven pattern at all — it's a workaround for not having a real event bus.

---

## Q6

A team has a single API Gateway REST API with two stages, `test` and `prod`. Each stage must
invoke a different Lambda alias (`test` → alias `TEST`, `prod` → alias `LIVE`) using the same API
definition and without maintaining two separate API Gateway resources. How should this be
configured?

A. Create two entirely separate API Gateway REST APIs, one per environment, each hardcoding its own Lambda ARN.

B. Use a stage variable (e.g. `${stageVariables.lambdaAlias}`) in the integration request's Lambda ARN, and set that stage variable to `TEST` on the `test` stage and `LIVE` on the `prod` stage.

C. Set the Lambda function's reserved concurrency differently per stage to control which alias is used.

D. Use a different API key for each stage and have the Lambda function branch its logic based on which API key was presented.

**Correct Answer: B**

**Explanation:** Stage variables are API Gateway's mechanism for parameterizing a single API
definition across environments — a stage variable can be referenced inside an integration's target
ARN, so the same deployed API routes `test` traffic to one Lambda alias and `prod` traffic to
another without duplicating resources. This is precisely Task 1.1's "use API Gateway stages
correctly (dev/test/prod promotion, stage variables)." See
[`services/networking-content-delivery/api-gateway.md`](../../../services/networking-content-delivery/api-gateway.md).

**Why the others are wrong:**
- A: Duplicating the whole API defeats the stated requirement ("without maintaining two separate API Gateway resources") and doubles maintenance overhead for every future change.
- C: Reserved concurrency controls capacity/throttling guarantees for a function, not which alias or ARN an API Gateway stage integrates with — it has no bearing on environment routing.
- D: API keys are for usage-plan throttling/metering of callers, not for routing a stage's backend integration target — this doesn't configure which Lambda alias gets invoked.

---

## Q7

A Lambda function needs to query a MySQL RDS instance that lives in private subnets with no public
endpoint, and the same function also needs to call an external third-party HTTPS API on the public
internet. What must be configured for this to work correctly?

A. Nothing extra — Lambda can always reach any RDS instance and any internet endpoint by default.

B. Attach the Lambda function to the VPC (so it gets an ENI in a private subnet with a security group allowing the DB port to RDS), and route outbound internet traffic through a NAT gateway in a public subnet.

C. Make the RDS instance publicly accessible so Lambda can reach it without VPC configuration, and allow the third-party API call to go through unmodified.

D. Attach the Lambda function to the VPC only; no additional configuration is needed for the internet-bound call since Lambda always retains direct internet access even when VPC-attached.

**Correct Answer: B**

**Explanation:** To reach a resource inside a VPC's private subnets (the RDS instance here), a
Lambda function must itself be configured with that VPC/subnets/security groups, giving it an ENI
inside the VPC — this is Task 1.2's "configure Lambda to access resources inside a VPC." Once
VPC-attached, a function only has the network routing of that VPC, so reaching the public internet
(the third-party API) additionally requires a NAT gateway (in a public subnet, reachable via the
private subnet's route table) unless a VPC endpoint exists for that specific destination. See
[`services/compute/lambda.md`](../../../services/compute/lambda.md) and
[`services/networking-content-delivery/vpc.md`](../../../services/networking-content-delivery/vpc.md).

**Why the others are wrong:**
- A: Lambda outside a VPC cannot reach resources inside private subnets at all — "always reach any RDS instance" is false for a private RDS instance.
- C: Making a production database publicly accessible to avoid VPC configuration is a serious security anti-pattern (directly contradicts Domain 2's least-exposure principle) and isn't the correct fix for this requirement.
- D: Once Lambda is attached to a VPC, it loses the default internet path and only has whatever routing the VPC's subnets provide — it does not "always retain direct internet access"; that's exactly why a NAT gateway (or VPC endpoint) is needed.

---

## Q8

Five different Lambda functions across a company's account all depend on the same internal
third-party SDK and a shared set of utility modules. Repackaging that SDK into every function's
deployment artifact bloats deploy times and causes version drift when one function updates the SDK
but others don't. What is the correct Task 1.2 solution?

A. Merge all five functions into one large Lambda function with internal routing logic.

B. Publish the shared SDK and utilities as a Lambda layer, and attach that layer version to each of the five functions.

C. Store the SDK code in an S3 bucket and have each function download it from S3 on every cold start.

D. Increase each function's deployment package size limit via a service quota increase request.

**Correct Answer: B**

**Explanation:** Lambda layers exist exactly for this: shared code/dependencies packaged once and
attached to multiple functions, keeping each function's own deployment package smaller and letting
you manage the shared dependency's version centrally (a new layer version can be rolled out
function-by-function). This is Task 1.2's "use Lambda layers to share code/dependencies across
functions." See [`services/compute/lambda.md`](../../../services/compute/lambda.md).

**Why the others are wrong:**
- A: Merging unrelated functions into one increases blast radius, couples unrelated failure domains, and doesn't solve the actual dependency-duplication problem — it's an architectural downgrade, not a fix.
- C: Downloading the SDK from S3 at every cold start adds latency to every cold start and introduces a runtime dependency on S3 availability that layers don't have (layers are already present in the execution environment at invoke time).
- D: There's no "deployment package size limit" quota increase that fixes dependency duplication/version drift — this doesn't address the stated problem at all.

---

## Q9 (Choose TWO)

An API backed by Lambda has a strict p99 latency SLA. During sudden morning traffic spikes, cold
starts cause enough functions to miss the SLA. The team has already confirmed the function's actual
execution time (once warm) is well within budget. Which TWO actions directly reduce the cold-start
latency impact for this workload?

A. Enable provisioned concurrency on the function (or a published version/alias) so a pool of execution environments is pre-initialized and ready before traffic arrives.

B. Configure reserved concurrency equal to the expected peak traffic, since reserved concurrency pre-warms execution environments.

C. Use Application Auto Scaling to schedule a provisioned concurrency increase shortly before the known morning traffic spike.

D. Increase the Lambda function's timeout setting to give cold starts more time to complete.

E. Switch the function's trigger from API Gateway (synchronous) to S3 event notifications (asynchronous) to avoid cold starts entirely.

**Correct Answer: A and C**

**Explanation:** Provisioned concurrency (A) keeps a specified number of execution environments
pre-initialized, eliminating cold-start latency for invocations that land on those pre-warmed
environments. Scheduling an increase in provisioned concurrency ahead of a known traffic pattern
(C) via Application Auto Scaling is the standard pattern for predictable spikes (e.g. "every weekday
at 9am"). This is Task 1.2's concurrency management and directly addresses the documented exam
scenario in [`../domain-1-development/README.md`](../domain-1-development/README.md) ("a burst of
traffic is causing cold starts to hurt p99 latency... provisioned concurrency, not just reserved
concurrency"). See [`services/compute/lambda.md`](../../../services/compute/lambda.md).

**Why the others are wrong:**
- B: Reserved concurrency only caps/guarantees a maximum slice of account concurrency for the function — it does not pre-initialize any execution environments, so it has no effect on cold-start latency by itself.
- D: A longer timeout lets a slow invocation run longer before being killed, but it doesn't make the cold start itself faster — this doesn't fix a latency SLA miss, it just delays when the SLA failure would even be detected.
- E: Changing the trigger/integration type doesn't eliminate cold starts (a fresh execution environment still needs initializing regardless of sync/async), and swapping the trigger would also change the application's actual request model, which isn't an option that preserves the API's behavior.

---

## Q10

A Lambda function is configured to process messages from a standard SQS queue. Separately, another
Lambda function is invoked directly by API Gateway for a REST endpoint. A developer is confused
about why the SQS-triggered function's concurrency behaves differently from the API-Gateway-triggered
one under load. What is the key distinction in how these two trigger types invoke Lambda?

A. There is no real difference — both are synchronous invocations from the caller's perspective.

B. The SQS trigger is poll-based: Lambda's internal poller reads batches from the queue and invokes the function, scaling the number of pollers/concurrent invocations based on queue depth; the API Gateway trigger is synchronous: the caller is blocked waiting for Lambda's response within the integration timeout.

C. The SQS trigger always invokes Lambda exactly once per message with no batching, while API Gateway always batches multiple HTTP requests into a single Lambda invocation.

D. Both trigger types are asynchronous and queue invocations internally for up to 6 hours regardless of source.

**Correct Answer: B**

**Explanation:** Task 1.2 distinguishes three Lambda invocation models: synchronous (the caller
waits — API Gateway proxy integration), asynchronous (fire-and-forget with internal retry/DLQ — e.g.
direct S3 event notifications), and poll-based (an AWS-managed poller reads from a source like SQS,
Kinesis, or DynamoDB Streams and invokes Lambda on the application's behalf, scaling pollers to
queue/stream activity). Understanding which model a given trigger uses determines how to reason
about concurrency, retries, and failure handling. See
[`services/compute/lambda.md`](../../../services/compute/lambda.md) and
[`services/application-integration/sqs.md`](../../../services/application-integration/sqs.md).

**Why the others are wrong:**
- A: API Gateway's Lambda proxy integration is indeed synchronous, but the SQS trigger is not — the queue's producer isn't waiting on Lambda at all, so framing both as "synchronous from the caller's perspective" is incorrect for SQS.
- C: SQS-triggered Lambda explicitly supports batching (multiple messages per invocation, configurable batch size) — "exactly one message per invocation with no batching" is false; API Gateway does not batch multiple distinct HTTP requests into one invocation.
- D: API Gateway's proxy integration is synchronous, not asynchronous, and does not have a 6-hour internal queue — that retry window describes Lambda's asynchronous invocation model (e.g. S3 events), not API Gateway's.

---

## Q11

A banking application writes a transaction to a DynamoDB table and must display the updated balance
to the same user on the very next screen, which issues a `GetItem` immediately after the write
completes. The displayed value must always reflect that write, every time, with no chance of
showing a stale value. How should the `GetItem` call be configured?

A. Leave the default read settings — DynamoDB always returns the most recent write by default.

B. Set `ConsistentRead=true` on the `GetItem` call, accepting roughly double the read-capacity cost compared to an eventually consistent read.

C. Add a manual `sleep(2)` before the `GetItem` call to give replication time to catch up, then use the default eventually consistent read.

D. Switch the table to use Global Tables, since multi-region replication guarantees the latest value is always returned.

**Correct Answer: B**

**Explanation:** DynamoDB's default read is eventually consistent (lower cost, might briefly
reflect a slightly stale value right after a write). For a "must read your own write immediately,
every time" requirement, Task 1.3 expects you to know to request a strongly consistent read via
`ConsistentRead=true` on `GetItem`/`Query`, which costs roughly double the read capacity units of an
eventually consistent read of the same size. See
[`services/database/dynamodb.md`](../../../services/database/dynamodb.md) and the exam-scenario
note in [`../domain-1-development/README.md`](../domain-1-development/README.md).

**Why the others are wrong:**
- A: DynamoDB's default read is explicitly eventually consistent, not strongly consistent — this is the misconception the question is testing against.
- C: A fixed sleep is not a guarantee against eventual consistency's variable propagation delay and is not a real engineering solution AWS exams accept — it's an anti-pattern, not a fix.
- D: Global Tables is about multi-region replication/disaster-recovery and active-active patterns; it does not provide same-region strong consistency guarantees and actually introduces its own (longer) cross-region eventual-consistency window, the opposite of what's needed here.

---

## Q12

A web application lets users upload video files up to 5 GB directly from the browser to S3. Users
on unreliable connections report that large uploads frequently fail entirely and must restart from
0%. Which S3 feature/pattern should the application use to fix this?

A. Reduce the maximum allowed file size to 100 MB so uploads complete faster and failures matter less.

B. Use S3 multipart upload, splitting the file into parts that can be uploaded independently (and retried individually) and only assembled into the final object once all parts succeed.

C. Switch the bucket's storage class to S3 Glacier so uploads are billed at a lower rate.

D. Enable S3 Transfer Acceleration only, with no change to how the file is uploaded as a single PUT.

**Correct Answer: B**

**Explanation:** S3 multipart upload is the standard pattern (Task 1.3's "use data stores... from
application code") for large objects: the client splits the object into parts, uploads them
independently (in parallel or sequentially), and only a failed part needs to be retried — not the
whole object. This directly fixes "uploads frequently fail entirely and restart from 0%." See
[`services/storage/s3.md`](../../../services/storage/s3.md).

**Why the others are wrong:**
- A: Shrinking the allowed file size doesn't meet the stated requirement (users need to upload up to 5 GB files) and doesn't fix the underlying reliability problem, it just avoids it for smaller files.
- C: Glacier is for cold, infrequently accessed storage with retrieval delays — it has nothing to do with upload reliability and isn't even a valid direct-PUT target for client browser uploads in the way described.
- D: Transfer Acceleration speeds up transfer over AWS's edge network but, without multipart upload, a single-PUT failure still restarts the entire object from scratch — it doesn't solve the "restart from 0%" problem by itself.

---

## Q13

An application needs to debit one account and credit another in a single operation that must be
fully atomic (both succeed or both roll back) and also needs complex multi-table SQL joins for
reporting across normalized schemas. Which data store(s) correctly satisfy these requirements, and
why?

A. DynamoDB, using DynamoDB Transactions, which support unlimited items per transaction and native SQL joins.

B. RDS or Aurora, because they provide full ACID transactions and native SQL joins across normalized relational tables — the workload described is fundamentally relational.

C. S3, using versioning to simulate atomic multi-object updates.

D. DynamoDB, because its eventually consistent reads make cross-table joins safe by default.

**Correct Answer: B**

**Explanation:** Task 1.3 expects recognizing when a scenario describes a genuinely relational
workload: ACID guarantees across related rows and multi-table joins. RDS/Aurora provide full SQL
ACID transactions and native joins; DynamoDB transactions exist but are capped at 25 items per
transaction and have no SQL join capability. See
[`services/database/rds.md`](../../../services/database/rds.md),
[`services/database/aurora.md`](../../../services/database/aurora.md), and the decision framework
in [`services/database/database-selection-guide.md`](../../../services/database/database-selection-guide.md).

**Why the others are wrong:**
- A: DynamoDB Transactions are real but limited to 25 unique items per transaction, and DynamoDB has no SQL join engine at all — "unlimited items" and "native SQL joins" are both false.
- C: S3 has no multi-object atomic transaction mechanism; versioning only preserves historical object versions, it doesn't make a multi-object update atomic.
- D: Consistency model (eventual vs. strong) is unrelated to whether joins are possible — DynamoDB simply has no join operator regardless of consistency settings.

---

## Q14

A checkout service synchronously calls an inventory-reservation service for every order. During a
flash sale, inventory-reservation's latency spikes under load, which in turn makes checkout time
out and fail for users, even though checkout itself has spare capacity. What architectural change
addresses this without checkout needing to know inventory-reservation's internal capacity?

A. Increase checkout's Lambda timeout so it waits longer for inventory-reservation.

B. Introduce an SQS queue between checkout and inventory-reservation so checkout enqueues reservation requests and returns immediately, decoupling checkout's availability from inventory-reservation's momentary capacity, with inventory-reservation consuming from the queue at its own pace.

C. Scale inventory-reservation's database up to the largest available instance size.

D. Have checkout call inventory-reservation twice in parallel to increase the odds one call succeeds in time.

**Correct Answer: B**

**Explanation:** This is Task 1.1's core decoupling pattern: "decouple a producer and consumer so a
traffic spike on one side doesn't overwhelm the other" via an SQS queue acting as a buffer with
backpressure, rather than a direct synchronous call whose failure mode is tightly coupled to the
downstream service's instantaneous capacity. See
[`services/application-integration/sqs.md`](../../../services/application-integration/sqs.md) and
[`services/application-integration/messaging-patterns-overview.md`](../../../services/application-integration/messaging-patterns-overview.md).

**Why the others are wrong:**
- A: A longer timeout just makes checkout wait longer while still being blocked on inventory-reservation's degraded performance — it doesn't fix the coupling, it just delays the failure and worsens user-perceived latency.
- C: Scaling the database is a capacity fix that might help but doesn't address the architectural coupling — the next unexpected spike will cause the same cascading failure regardless of size, and it doesn't answer the "without checkout needing to know its internal capacity" requirement.
- D: Calling twice in parallel increases load on an already-struggling downstream service, making the problem worse, not better.

---

## Q15

A developer uses Amazon Q Developer inline in their IDE to generate a first draft of a new Lambda
handler that writes to DynamoDB. The suggestion includes a complete handler, an IAM policy JSON
snippet granting `dynamodb:*` on `Resource: "*"`, and no input validation on the incoming event
body. What is the correct way to use this AI-generated output per DVA-C03's AI-assisted development
expectations?

A. Accept and deploy the suggestion as-is — AI-generated code from Amazon Q Developer is already reviewed for security and correctness before being suggested.

B. Reject AI-assisted development entirely for production code; only hand-written code should ever be deployed.

C. Treat the suggestion as a fast first draft: the developer reviews and tightens the IAM policy to least privilege (specific table ARN and actions), adds input validation, and applies the same code-review rigor as hand-written code before merging.

D. Deploy the handler as-is but ask Amazon Q Developer to automatically fix the IAM policy at deploy time via a CI/CD approval gate.

**Correct Answer: C**

**Explanation:** DVA-C03 explicitly tests the AI-assisted development *pattern* across the SDLC:
an assistant like Amazon Q Developer speeds up producing a first draft, but the developer remains
responsible for correctness and security — reviewing and tightening anything the assistant got
wrong (here: an overly broad `dynamodb:*`/`Resource:*` policy and missing validation) before it
ships. See [`../domain-1-development/ai-assisted-development.md`](../domain-1-development/ai-assisted-development.md)
("Generated code still needs the same code review rigor as human-written code").

**Why the others are wrong:**
- A: AI coding assistants are not a substitute for security/code review — the exam explicitly calls out over-trusting AI output (here, an overly broad IAM policy) as a failure mode to recognize, not a non-issue.
- B: The exam doesn't test "never use AI assistance" as correct — it tests safe usage with human review, not blanket rejection of the tool.
- D: There is no mechanism where the AI assistant "automatically fixes" a policy unattended at deploy time as a substitute for human review — Domain 3's AI-assisted deployment approvals pattern still requires a human to approve, not an unattended auto-fix.
