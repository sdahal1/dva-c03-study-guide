# Task 1.2 practice questions

## Q1

**Skills:** 1.2.1

A Lambda function needs to query a MySQL RDS instance that lives in private subnets with no public
endpoint, and the same function also needs to call an external third-party HTTPS API on the public
internet. What must be configured for this to work correctly?

A. Nothing extra. Lambda can always reach any RDS instance and any internet endpoint by default.

B. Attach the Lambda function to the VPC (so it gets an ENI in a private subnet with a security group allowing the DB port to RDS), and route outbound internet traffic through a NAT gateway in a public subnet.

C. Make the RDS instance publicly accessible so Lambda can reach it without VPC configuration, and allow the third-party API call to go through unmodified.

D. Attach the Lambda function to the VPC only; no additional configuration is needed for the internet-bound call since Lambda always retains direct internet access even when VPC-attached.

**Correct Answer: B**

**Explanation:** To reach a resource inside a VPC's private subnets (the RDS instance here), a
Lambda function must itself be configured with that VPC's subnets and security groups, giving it an
ENI inside the VPC. This is Task 1.2's "configure Lambda to access resources inside a VPC." Once
VPC-attached, a function only has the network routing of that VPC, so reaching the public internet
(the third-party API) additionally requires a NAT gateway (in a public subnet, reachable via the
private subnet's route table) unless a VPC endpoint exists for that specific destination. See
[1.2.1 Lambda VPC access](1.2.1-lambda-vpc-access.md#reaching-private-resources-and-the-outside-world).

**Why the others are wrong:**
- A: Lambda outside a VPC cannot reach resources inside private subnets at all. "Always reach any RDS instance" is false for a private RDS instance.
- C: Making a production database publicly accessible to avoid VPC configuration is a serious security anti-pattern (it directly contradicts Domain 2's least-exposure principle) and isn't the correct fix for this requirement.
- D: Once Lambda is attached to a VPC, it loses the default internet path and only has whatever routing the VPC's subnets provide. It does not "always retain direct internet access"; that's exactly why a NAT gateway (or VPC endpoint) is needed.

---

## Q2

**Skills:** 1.2.2

Five different Lambda functions across a company's account all depend on the same internal
third-party SDK and a shared set of utility modules. Repackaging that SDK into every function's
deployment artifact bloats deploy times and causes version drift when one function updates the SDK
but others don't. What is the correct Task 1.2 solution?

A. Merge all five functions into one large Lambda function with internal routing logic.

B. Publish the shared SDK and utilities as a Lambda layer, and attach that layer version to each of the five functions.

C. Store the SDK code in an S3 bucket and have each function download it from S3 on every cold start.

D. Increase each function's deployment package size limit via a service quota increase request.

**Correct Answer: B**

**Explanation:** Lambda layers exist exactly for this: shared code and dependencies packaged once and
attached to multiple functions, keeping each function's own deployment package smaller and letting
you manage the shared dependency's version centrally (a new layer version can be rolled out
function by function). This is Task 1.2's "use Lambda layers to share code and dependencies across
functions." See [1.2.2 Lambda configuration](1.2.2-lambda-configuration.md#layers).

**Why the others are wrong:**
- A: Merging unrelated functions into one increases blast radius, couples unrelated failure domains, and doesn't solve the actual dependency-duplication problem. It's an architectural downgrade, not a fix.
- C: Downloading the SDK from S3 at every cold start adds latency to every cold start and introduces a runtime dependency on S3 availability that layers don't have (layers are already present in the execution environment at invoke time).
- D: There's no "deployment package size limit" quota increase that fixes dependency duplication or version drift. This doesn't address the stated problem at all.

---

## Q3 (Choose TWO)

**Skills:** 1.2.2

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
environments. It is configured on a published version or alias, not on `$LATEST`. Scheduling an
increase in provisioned concurrency ahead of a known traffic pattern (C) via Application Auto
Scaling is the standard pattern for predictable spikes (for example, "every weekday at 9am"). This
is Task 1.2's concurrency management and the classic exam scenario "a burst of traffic is causing
cold starts to hurt p99 latency": the answer is provisioned concurrency, not reserved concurrency.
See [1.2.2 Lambda configuration](1.2.2-lambda-configuration.md#concurrency).

**Why the others are wrong:**
- B: Reserved concurrency only caps and guarantees a maximum slice of account concurrency for the function. It does not pre-initialize any execution environments, so it has no effect on cold-start latency by itself.
- D: A longer timeout lets a slow invocation run longer before being stopped, but it doesn't make the cold start itself faster. This doesn't fix a latency SLA miss; it only delays when the SLA failure would be detected.
- E: Changing the trigger or integration type doesn't eliminate cold starts (a fresh execution environment still needs initializing regardless of sync or async), and swapping the trigger would also change the application's request model, which isn't an option that preserves the API's behavior.

---

## Q4

**Skills:** 1.2.3, 1.2.4

A Lambda function is configured to process messages from a standard SQS queue. Separately, another
Lambda function is invoked directly by API Gateway for a REST endpoint. A developer is confused
about why the SQS-triggered function's concurrency behaves differently from the API-Gateway-triggered
one under load. What is the key distinction in how these two trigger types invoke Lambda?

A. There is no real difference. Both are synchronous invocations from the caller's perspective.

B. The SQS trigger is poll-based: Lambda's internal poller reads batches from the queue and invokes the function, scaling the number of pollers/concurrent invocations based on queue depth; the API Gateway trigger is synchronous: the caller is blocked waiting for Lambda's response within the integration timeout.

C. The SQS trigger always invokes Lambda exactly once per message with no batching, while API Gateway always batches multiple HTTP requests into a single Lambda invocation.

D. Both trigger types are asynchronous and queue invocations internally for up to 6 hours regardless of source.

**Correct Answer: B**

**Explanation:** Task 1.2 distinguishes three Lambda invocation models: synchronous (the caller
waits, as with an API Gateway proxy integration), asynchronous (fire-and-forget with internal retry
and a DLQ or destination, as with direct S3 event notifications), and poll-based (an AWS-managed
poller reads from a source like SQS, Kinesis, or DynamoDB Streams and invokes Lambda on the
application's behalf, scaling pollers to queue or stream activity). Understanding which model a
given trigger uses determines how to reason about concurrency, retries, and failure handling. See
[1.2.3 Invocation lifecycle and failures](1.2.3-lambda-invocation-lifecycle-and-failures.md#three-invocation-models)
and [1.2.4 event source mappings](1.2.4-lambda-service-integrations.md#scaling-and-ordering-by-source).

**Why the others are wrong:**
- A: API Gateway's Lambda proxy integration is indeed synchronous, but the SQS trigger is not synchronous from the producer's side. The queue's producer isn't waiting on Lambda at all, so framing both as "synchronous from the caller's perspective" is incorrect for SQS.
- C: SQS-triggered Lambda explicitly supports batching (multiple messages per invocation, configurable batch size), so "exactly one message per invocation with no batching" is false. API Gateway does not batch multiple distinct HTTP requests into one invocation.
- D: API Gateway's proxy integration is synchronous, not asynchronous, and does not have a 6-hour internal queue. That retry window describes Lambda's asynchronous invocation model (for example, S3 events), not API Gateway's.
