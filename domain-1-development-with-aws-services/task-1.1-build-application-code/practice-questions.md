# Task 1.1 practice questions

Seven questions in exam format: single-answer multiple choice unless marked "Choose TWO". Each question lists the skills it covers. Try each question before reading the answer below it.

Abbreviations: Amazon Simple Queue Service (SQS), Amazon Simple Notification Service (SNS), Amazon Simple Storage Service (S3), application programming interface (API), Representational State Transfer (REST), JavaScript Object Notation (JSON), Amazon Resource Name (ARN), dead-letter queue (DLQ), AWS Identity and Access Management (IAM), artificial intelligence (AI), integrated development environment (IDE), continuous integration and continuous delivery (CI/CD).

## Q1

**Skills:** 1.1.6, 1.1.1

A company ingests clickstream data from millions of devices and needs to process records in the
order they were produced, per device, while allowing multiple consumer applications to read the
same data independently (one for real-time dashboards, one for batch analytics). Which AWS service
is the best fit for the ingestion layer, and why?

A. Amazon SQS Standard queue: any consumer that needs the data can poll the same queue.

B. Amazon Kinesis Data Streams: records are grouped into shards by partition key (preserving per-key order) and multiple independent consumers can each read the full stream at their own pace.

C. Amazon SNS topic: it fans out every message to all subscribed consumers automatically.

D. AWS Step Functions: a state machine can track per-device order as workflow executions.

**Correct Answer: B**

**Explanation:** Kinesis Data Streams is built for streaming data ingestion with ordering
guarantees *within a shard* (records with the same partition key land in the same shard, in order)
and supports multiple independent consumer applications reading the same stream concurrently through
separate shard iterators or enhanced fan-out. This is skill 1.1.6, processing and managing streaming
data. See [1.1.6 Streaming data](1.1.6-streaming-data.md) and the service comparison in
[1.1.1 Coupling and decoupling](1.1.1-coupling-and-decoupling.md#the-decoupling-toolbox).

**Why the others are wrong:**
- A: SQS Standard does not guarantee per-key ordering, and a message is consumed once by one consumer (it is a queue), not read in full by multiple independent readers.
- C: SNS fans out to subscribers, but it has no concept of ordered, replayable streaming data per partition key. Once delivered, a message cannot be re-read from an offset the way a Kinesis shard can.
- D: Step Functions orchestrates workflows and state transitions; it is not an ingestion mechanism for high-volume streaming records.

---

## Q2

**Skills:** 1.1.7, 1.1.4

A Lambda function is integrated with API Gateway using the Lambda proxy integration. The function
runs without errors and CloudWatch Logs show the handler completing successfully and returning a
Python dict like `{"statusCode": 200, "body": {"orderId": "123"}}`. However, every client call
receives a generic `Internal Server Error` from API Gateway. What is the bug?

A. The Lambda execution role is missing `apigateway:*` permissions.

B. `body` must be a JSON-encoded **string**, not a raw dict/object. API Gateway's proxy integration expects `body` to already be serialized text.

C. `statusCode` must be a string ("200"), not an integer, in proxy integration responses.

D. API Gateway proxy integration requires the response to include a `headers` key even if empty, or it rejects the whole payload.

**Correct Answer: B**

**Explanation:** This is a classic serialize/deserialize trap. The Lambda proxy integration contract
requires the return object's `body` field to be a JSON **string** (`json.dumps({"orderId": "123"})`),
because API Gateway passes that string through to the client rather than re-serializing it.
Returning a raw object breaks the integration contract, and API Gateway responds with
`502 Bad Gateway` and the message "Internal server error". See the Lambda proxy contract and the
502 walkthrough in [1.1.4](1.1.4-apis-transformation-and-validation.md#lambda-proxy-integration-contract),
and [1.1.7 Serialization and deserialization](1.1.7-serialization-and-deserialization.md).

**Why the others are wrong:**
- A: API Gateway invoking Lambda needs `lambda:InvokeFunction`, granted through the function's resource-based policy, not `apigateway:*` on the execution role. A missing invoke permission would stop API Gateway from invoking the function at all, so it cannot explain a handler that ran and logged success.
- C: `statusCode` is expected as an integer in the proxy response contract; a string is not the documented requirement and is not what breaks this scenario.
- D: `headers` is optional in the proxy response shape; its absence does not cause an error.

---

## Q3 (Choose TWO)

**Skills:** 1.1.8, 1.1.1

An application must process an image immediately after it's uploaded to an S3 bucket. The
processing Lambda function occasionally experiences brief periods where downstream capacity is
limited, and the team wants uploads during a traffic spike to not be dropped or fail outright even
if Lambda can't immediately keep up. Which TWO design choices satisfy both requirements (trigger on
upload, and tolerate processing bursts without dropping work)?

A. Configure an S3 event notification that invokes the Lambda function directly and synchronously for every `PutObject`.

B. Configure an S3 event notification targeting an SQS queue, then have Lambda poll the queue as an event source. The queue buffers bursts and Lambda scales consumption at its own pace.

C. Configure an S3 event notification targeting EventBridge, with a rule that can apply additional filtering/fan-out before invoking Lambda.

D. Disable all S3 event notifications and instead have Lambda run on a fixed CloudWatch Events schedule to scan the bucket for new objects every minute.

E. Increase the S3 bucket's request rate limit so uploads never need buffering.

**Correct Answer: B and C**

**Explanation:** Both B and C satisfy "trigger immediately on upload" (S3 event notifications are
near-real-time) while adding a buffering or decoupling layer between the upload event and Lambda
execution. SQS (B) explicitly buffers bursts: Lambda's poller scales consumers against queue depth
rather than being invoked once per object. EventBridge (C) adds routing and filtering, retries
delivery to its targets, can be paired with a queue downstream, and decouples the S3 event from a
single rigid invocation path. This is the event-driven application pattern of skill 1.1.8. See
[1.1.8 Event-driven applications](1.1.8-event-driven-applications.md) and
[1.1.1 SQS as a buffer](1.1.1-coupling-and-decoupling.md#amazon-sqs-the-default-buffer).

**Why the others are wrong:**
- A: Direct S3-to-Lambda invocation is asynchronous, not synchronous. More importantly, it has no durable buffer: if Lambda's concurrency is exhausted during a spike, events wait in Lambda's internal queue for a limited retry window (up to 6 hours by default) and can then be dropped (sent to a DLQ or on-failure destination if configured, otherwise lost), rather than being durably buffered the way an SQS queue holds them.
- D: Polling on a schedule defeats "process immediately after upload". It introduces up to a full polling-interval delay and is a worse design than event notifications for this requirement.
- E: S3 request rate is not the bottleneck described (Lambda's downstream capacity is). Raising an S3 limit does nothing for Lambda-side burst handling.

---

## Q4

**Skills:** 1.1.8, 1.1.2

A retail application needs several independent subsystems (inventory, shipping, notifications) to
each react to an `OrderPlaced` event, but only the subsystems that care about a specific order
`region` attribute should actually be invoked for that event. Which approach correctly implements
this in an event-driven architecture on AWS?

A. Publish the event to an SNS topic and have every subscriber inspect the `region` field in its own code before deciding whether to act.

B. Publish the event to an EventBridge event bus and create a separate rule per subsystem with an event pattern that matches on the `region` attribute, so EventBridge itself filters which targets get invoked.

C. Publish the event directly to each subsystem's Lambda function via separate synchronous `Invoke` calls from the order service.

D. Write the event to a DynamoDB table and have each subsystem poll the table every few seconds looking for new rows.

**Correct Answer: B**

**Explanation:** EventBridge is the central event bus for building event-driven applications with
content-based filtering. Rules use event patterns to match on specific attributes (like `region`)
and invoke only matching targets, so routing logic lives in the bus's rule configuration rather
than being duplicated in every consumer. See [1.1.8 Event-driven applications](1.1.8-event-driven-applications.md)
and [1.1.2 event-driven choreography](1.1.2-communication-patterns.md#pattern-6-event-driven-choreography-with-eventbridge).

**Why the others are wrong:**
- A: SNS can filter at the subscription level with filter policies, but without one, every subscriber receives every message and must filter itself. That works, but it is a worse fit than EventBridge's richer pattern matching, and as described (inspecting in code) the bus does no filtering, which increases invocation cost and coupling.
- C: Direct synchronous invokes tightly couple the order service to every subsystem's availability and address. It is the opposite of an event-driven, decoupled design, and adding a new subsystem requires changing the order service's code.
- D: Polling a table adds latency and unnecessary read cost and load, and is not an event-driven pattern at all.

---

## Q5

**Skills:** 1.1.4, 3.1.5

A team has a single API Gateway REST API with two stages, `test` and `prod`. Each stage must
invoke a different Lambda alias (`test` → alias `TEST`, `prod` → alias `LIVE`) using the same API
definition and without maintaining two separate API Gateway resources. How should this be
configured?

A. Create two entirely separate API Gateway REST APIs, one per environment, each hardcoding its own Lambda ARN.

B. Use a stage variable (e.g. `${stageVariables.lambdaAlias}`) in the integration request's Lambda ARN, and set that stage variable to `TEST` on the `test` stage and `LIVE` on the `prod` stage.

C. Set the Lambda function's reserved concurrency differently per stage to control which alias is used.

D. Use a different API key for each stage and have the Lambda function branch its logic based on which API key was presented.

**Correct Answer: B**

**Explanation:** Stage variables parameterize a single API definition across environments. A stage
variable can be referenced inside an integration's target ARN, so the same deployed API routes
`test` traffic to one Lambda alias and `prod` traffic to another without duplicating resources.
API Gateway also needs permission to invoke each alias. Stage configuration is skill 3.1.5, applied
to an API you build and maintain under 1.1.4. See
[3.1.5 API Gateway stages and endpoints](../../domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/3.1.5-api-gateway-stages-and-endpoints.md)
and [1.1.4 Maintain an API](1.1.4-apis-transformation-and-validation.md#maintain-an-api).

**Why the others are wrong:**
- A: Duplicating the whole API defeats the stated requirement ("without maintaining two separate API Gateway resources") and doubles maintenance for every future change.
- C: Reserved concurrency controls capacity and throttling for a function, not which alias or ARN an API Gateway stage integrates with.
- D: API keys are for usage-plan throttling and metering of callers, not for routing a stage's backend integration target.

---

## Q6

**Skills:** 1.1.1, 1.1.2

A checkout service synchronously calls an inventory-reservation service for every order. During a
flash sale, inventory-reservation's latency spikes under load, which in turn makes checkout time
out and fail for users, even though checkout itself has spare capacity. What architectural change
addresses this without checkout needing to know inventory-reservation's internal capacity?

A. Increase checkout's Lambda timeout so it waits longer for inventory-reservation.

B. Introduce an SQS queue between checkout and inventory-reservation so checkout enqueues reservation requests and returns immediately, decoupling checkout's availability from inventory-reservation's momentary capacity, with inventory-reservation consuming from the queue at its own pace.

C. Scale inventory-reservation's database up to the largest available instance size.

D. Have checkout call inventory-reservation twice in parallel to increase the odds one call succeeds in time.

**Correct Answer: B**

**Explanation:** This is the core decoupling pattern: a producer and consumer are decoupled so a
traffic spike on one side does not overwhelm the other. An SQS queue acts as a buffer with
backpressure, replacing a direct synchronous call whose failure mode is tied to the downstream
service's instantaneous capacity. See the checkout and inventory scenario in
[1.1.1](1.1.1-coupling-and-decoupling.md#worked-scenario-checkout-and-inventory).

**Why the others are wrong:**
- A: A longer timeout makes checkout wait longer while still blocked on inventory-reservation's degraded performance. It does not fix the coupling; it delays the failure and worsens user-perceived latency.
- C: Scaling the database is a capacity fix that might help but does not address the architectural coupling. The next unexpected spike causes the same cascading failure, and it does not meet the "without checkout needing to know its internal capacity" requirement.
- D: Calling twice in parallel increases load on an already struggling downstream service, making the problem worse.

---

## Q7

**Skills:** 1.1.9, 2.1.6

> **Not on the DVA-C03 service list.** Amazon Q Developer, named in this question, is not in the DVA-C03 in-scope services. The exam names Kiro as its AI-assisted development tool. The principle tested here (review and tighten AI-generated code before shipping) applies to any AI coding assistant, including Kiro.

A developer uses Amazon Q Developer inline in their IDE to generate a first draft of a new Lambda
handler that writes to DynamoDB. The suggestion includes a complete handler, an IAM policy JSON
snippet granting `dynamodb:*` on `Resource: "*"`, and no input validation on the incoming event
body. What is the correct way to use this AI-generated output per DVA-C03's AI-assisted development
expectations?

A. Accept and deploy the suggestion as-is. AI-generated code from Amazon Q Developer is already reviewed for security and correctness before being suggested.

B. Reject AI-assisted development entirely for production code; only hand-written code should ever be deployed.

C. Treat the suggestion as a fast first draft: the developer reviews and tightens the IAM policy to least privilege (specific table ARN and actions), adds input validation, and applies the same code-review rigor as hand-written code before merging.

D. Deploy the handler as-is but ask Amazon Q Developer to automatically fix the IAM policy at deploy time via a CI/CD approval gate.

**Correct Answer: C**

**Explanation:** DVA-C03 tests the AI-assisted development pattern across the software development
lifecycle: an assistant speeds up producing a first draft, but the developer remains responsible
for correctness and security, reviewing and tightening anything the assistant got wrong (here, an
overly broad `dynamodb:*` on `Resource: "*"` policy and missing validation) before it ships.
Generated code needs the same review rigor as human-written code. See
[1.1.9 AI-assisted development tools](1.1.9-ai-assisted-development-tools.md) and
[2.1.6 Least-privilege permissions](../../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md).

**Why the others are wrong:**
- A: AI coding assistants are not a substitute for security and code review. The exam calls out over-trusting AI output (here, an overly broad IAM policy) as a failure mode to recognize.
- B: The exam does not treat "never use AI assistance" as correct. It tests safe use with human review, not blanket rejection of the tool.
- D: There is no mechanism where an AI assistant "automatically fixes" a policy unattended at deploy time as a substitute for human review. The AI-assisted deployment approval pattern in [3.3.9](../../domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/3.3.9-ai-assisted-deployment.md) still requires a human to approve.
