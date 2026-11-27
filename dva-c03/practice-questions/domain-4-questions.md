# Domain 4 Practice Questions — Troubleshooting and Optimization (22%)

11 questions covering Task 4.1 (root cause analysis: CloudWatch, X-Ray, CloudTrail, AI-assisted
error analysis), Task 4.2 (instrumenting code for observability), and Task 4.3 (optimization:
Lambda performance tuning, cost/performance tradeoffs, AI-assisted optimization recommendations).
See [`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md)
for the full domain outline. 1 question uses the **(Choose TWO)** multi-select format, and Q10
covers the AI-assisted development pattern in this domain.

---

## Q1

A request flows through API Gateway → Lambda → DynamoDB → an external payment API, and users report
the overall request is slow, but CloudWatch Logs for the Lambda function show no errors and
reasonable execution duration. The team needs to find exactly *which hop* in this chain is adding
the latency. What should they use?

A. CloudTrail, since it logs every API call made in the account.

B. The X-Ray service map / trace segments, which show per-hop latency across the entire distributed call chain (API Gateway → Lambda → DynamoDB → external call), pinpointing exactly where time is being spent.

C. A CloudWatch Logs Insights query across all four services' log groups, manually correlating timestamps by hand.

D. Increase the Lambda function's memory, since more memory always reduces end-to-end latency regardless of where the bottleneck is.

**Correct Answer: B**

**Explanation:** Task 4.1 distinguishes CloudWatch Logs (what happened inside one service) from
X-Ray (where in a multi-hop distributed chain the time went) — "need to see which specific
microservice in a chain of 5 calls is adding latency → X-Ray service map / trace segments, not just
CloudWatch Logs (logs alone don't show cross-service timing without manual correlation)." See
[`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md)
and [`observability-instrumentation.md`](../domain-4-troubleshooting-optimization/observability-instrumentation.md).

**Why the others are wrong:**
- A: CloudTrail records *who/what AWS API call* happened and when (audit trail), not application-level request latency across a call chain — it wouldn't show "the external payment API call took 2.1 seconds."
- C: Manually correlating timestamps across four separate log groups is slow, error-prone, and exactly the problem X-Ray's unified trace view exists to solve — it's technically possible but far inferior to using the purpose-built tool.
- D: Increasing Lambda memory only helps if the Lambda function itself is the CPU-bound bottleneck — if the slow hop is actually the external payment API or a DynamoDB call, more Lambda memory does nothing, and the scenario explicitly says Lambda's own duration looks reasonable.

---

## Q2

An on-call engineer needs to find the specific stack trace / error message that caused a Lambda
function invocation to fail at 3:14 AM, including the full exception text and any custom log lines
the function emitted right before failing. Where should they look?

A. CloudTrail event history, filtered to the function's ARN.

B. CloudWatch Logs for that function's log group, searching around the 3:14 AM timestamp for the error output and preceding application log lines.

C. AWS Config's resource timeline for the Lambda function.

D. The X-Ray service map's high-level latency view only, with no drill-down into individual segments.

**Correct Answer: B**

**Explanation:** Task 4.1 distinguishes CloudWatch (application/runtime telemetry — logs, metrics,
stack traces, custom log output) from CloudTrail (API call audit trail) — finding "what happened"
inside the function's own execution, including a full stack trace and custom log lines, is
precisely CloudWatch Logs' job. See
[`services/management-governance/cloudwatch.md`](../../../services/management-governance/cloudwatch.md).

**Why the others are wrong:**
- A: CloudTrail records API calls (e.g. "who called `lambda:Invoke`" or "who changed this function's configuration"), not the function's internal runtime output like an exception stack trace.
- C: AWS Config tracks configuration state/changes of resources over time (e.g. "did someone change this function's memory setting"), not application-level error output from a single invocation.
- D: The high-level X-Ray service map shows latency per hop, but drilling into an individual trace segment's annotations/exceptions is a different (valid) X-Ray use, and even then the full custom log text the function emitted is still found in CloudWatch Logs, not the service map view.

---

## Q3

A production incident is being investigated and the team suspects a recent IAM policy change
accidentally revoked a Lambda function's DynamoDB write permission, starting the incident's error
spike. They need to know exactly which IAM principal made that policy change and precisely when.
Which service answers this?

A. CloudWatch Metrics, since it tracks DynamoDB throttling automatically.

B. CloudTrail event history, which logs every AWS API call (including IAM policy modifications) along with the calling principal and timestamp.

C. X-Ray, since it traces every call a Lambda function makes to DynamoDB.

D. Amazon Macie, since it monitors for sensitive configuration changes.

**Correct Answer: B**

**Explanation:** Task 4.1's "use CloudTrail to find who/what made an API call that caused it" is
exactly this scenario — "need to know exactly which IAM principal deleted/changed a resource at a
specific time → CloudTrail event history, not CloudWatch Logs" (CloudTrail is the audit trail of
*API calls*; CloudWatch is *application/runtime* telemetry). See
[`services/management-governance/cloudtrail.md`](../../../services/management-governance/cloudtrail.md)
and [`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md).

**Why the others are wrong:**
- A: CloudWatch Metrics would show the *symptom* (e.g. a spike in `AccessDeniedException`-driven errors or throttling metrics) but not *who* made the IAM policy change that caused it.
- C: X-Ray traces the application's own call chain (Lambda → DynamoDB) for latency/error visibility within requests; it has no visibility into IAM policy management-plane changes made outside the traced request flow.
- D: Macie is for discovering/classifying sensitive data at rest, not for auditing IAM policy change history.

---

## Q4

A team repeatedly struggles to debug production incidents because their Lambda functions log only
bare `print("error")` statements with no request ID, no structured fields, and no context about
which input caused the failure. The team wants to fix this proactively rather than scrambling during
the next incident. What is the correct Task 4.2 action?

A. Wait until the next incident and add better logging then, under time pressure, since that's when the specific need will be clearest.

B. Instrument the code now to emit structured logs (e.g. JSON with request ID, relevant input identifiers, and error context) and custom CloudWatch metrics for key operations, so the right signals already exist before the next incident happens.

C. Increase the Lambda function's memory allocation, since more memory produces more detailed log output automatically.

D. Disable logging entirely to reduce CloudWatch costs, since logs didn't help root-cause the last incident anyway.

**Correct Answer: B**

**Explanation:** Task 4.2 is explicitly about instrumenting code *before* an incident, not during
one: "ensure the right signals exist before an incident, not scrambled together during one" — this
means structured logs with context (request ID, relevant identifiers) and custom metrics emitted
proactively from application code. See
[`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md)
("A Lambda function's errors aren't showing enough context to debug → the function isn't logging
with structured, sufficiently detailed output — this is an instrumentation gap, fix before the next
incident").

**Why the others are wrong:**
- A: Waiting until the next incident to add logging guarantees the *current* incident (and likely the next one too, before the fix ships) remains hard to debug — Task 4.2 is specifically about proactive instrumentation, not reactive scrambling.
- C: Lambda memory allocation controls compute/CPU resources, not log verbosity or structure — more memory does not automatically produce better log content.
- D: Removing logging entirely makes future root-cause analysis strictly harder, not easier, and isn't a reasonable response to logs that were unstructured (the fix is to improve them, not eliminate them).

---

## Q5

A development team wants to add distributed tracing to a Lambda function so that each significant
operation inside the handler (e.g. "validate input," "call DynamoDB," "call external API") shows up
as its own timed unit in the end-to-end trace, not just one big opaque Lambda invocation block. What
should they add to the application code?

A. Nothing — X-Ray automatically breaks every Lambda invocation into named subsegments for each internal function call with zero code changes.

B. Use the X-Ray SDK to create custom subsegments around each significant internal operation (e.g. `validate input`, the DynamoDB call, the external API call), giving fine-grained, named timing within the single Lambda invocation's trace segment.

C. Create a separate Lambda function for every internal operation so each shows up as its own top-level X-Ray segment.

D. Increase the Lambda function's reserved concurrency so X-Ray automatically captures more detail per invocation.

**Correct Answer: B**

**Explanation:** Task 4.2's "emit... trace segments/subsegments from application code" means using
the X-Ray SDK explicitly to wrap specific internal operations as named subsegments — AWS SDK calls
(e.g. the DynamoDB call) are automatically instrumented when the X-Ray SDK wraps the client, but
custom internal logic (like "validate input") needs an explicit subsegment added by the developer.
See [`observability-instrumentation.md`](../domain-4-troubleshooting-optimization/observability-instrumentation.md).

**Why the others are wrong:**
- A: X-Ray does automatically instrument supported AWS SDK calls once the SDK is wrapped/patched, but it does not automatically create subsegments for arbitrary custom internal logic like a validation step with zero code changes — that requires explicit instrumentation.
- C: Splitting internal operations into separate Lambda functions changes the application's architecture unnecessarily and adds invocation overhead/latency and IAM/complexity just to get tracing granularity that subsegments provide without restructuring the function.
- D: Reserved concurrency controls capacity guarantees for a function; it has no relationship to X-Ray trace detail or subsegment creation.

---

## Q6

Under load, a Lambda function running image-resizing logic intermittently times out. Profiling data
shows the function is consistently near 100% CPU utilization for the full duration of each
invocation (not idle-waiting on a network call). The function's memory is currently set to the
minimum (128 MB). What is the most appropriate Task 4.3 optimization?

A. Increase the function's timeout value only, leaving memory unchanged, so the CPU-bound work has more wall-clock time to finish.

B. Increase the function's memory allocation, since Lambda allocates CPU power proportionally to configured memory — more memory directly means more CPU for a CPU-bound workload, likely reducing duration enough to avoid timeouts.

C. Decrease the function's memory allocation to reduce cost, since the timeout is unrelated to memory.

D. Add a DAX cache in front of the function, since caching fixes CPU-bound workloads.

**Correct Answer: B**

**Explanation:** Task 4.3's canonical Lambda performance-tuning scenario: "a Lambda function times
out intermittently under load, and profiling shows it's CPU-bound, not idle-waiting → increase
memory (which scales CPU proportionally in Lambda), not just the timeout value." Lambda allocates
CPU power proportional to the configured memory, so a CPU-bound workload at minimum memory is
leaving performance on the table. See
[`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md)
and [`services/compute/lambda.md`](../../../services/compute/lambda.md).

**Why the others are wrong:**
- A: Raising only the timeout doesn't make the CPU-bound work finish any faster — it just allows the same slow execution to run longer before being killed, potentially masking the problem while increasing cost per invocation (billed duration) without fixing root cause.
- C: Decreasing memory for a CPU-bound workload would make the function even slower (less CPU allocated), worsening the timeout problem, not helping cost in any meaningful way once timeouts/retries are accounted for.
- D: DAX is a caching layer for DynamoDB read latency — it has no relevance to a CPU-bound image-resizing computation happening entirely within the function's own execution.

---

## Q7 (Choose TWO)

A DynamoDB table serving a high-traffic application starts returning
`ProvisionedThroughputExceededException` errors under load. Investigating further, the team finds
that one specific partition key value (a popular "featured" item ID) receives a disproportionate
share of all requests, while most other partition keys are barely touched. Which TWO statements
correctly distinguish the diagnosis and the appropriate fix for this specific situation?

A. This is a "hot partition" access-pattern problem: a single partition key is receiving disproportionate traffic relative to the table's other keys, regardless of the table's overall provisioned/on-demand capacity.

B. The correct fix for a hot-partition problem is to redesign the access pattern (e.g. add a cache in front of the hot item, or shard the hot key into multiple logical keys), not simply raise the table's overall provisioned capacity, since raising overall capacity doesn't fix uneven distribution across keys.

C. The correct fix is always to switch the table to on-demand capacity mode, which eliminates hot-partition throttling entirely regardless of key design.

D. This is purely a capacity problem unrelated to key design — simply raising the table's total provisioned throughput will evenly resolve the throttling no matter which key is hot.

E. Since DynamoDB automatically rebalances traffic evenly across all partitions regardless of key design, this symptom could not actually be caused by an uneven key access pattern.

**Correct Answer: A and B**

**Explanation:** Task 4.3 explicitly distinguishes this exact failure mode: "DynamoDB reads are
being throttled under load → check if it's a hot partition key (access pattern issue, fix the key
design) vs. simply under-provisioned capacity (fix: on-demand mode or raise provisioned capacity) —
these have different fixes despite the same symptom." A single disproportionately hot key (A) needs
an access-pattern-level fix like caching or key sharding (B) — raising overall table capacity
doesn't address the skew. See
[`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md)
and [`services/database/dynamodb.md`](../../../services/database/dynamodb.md).

**Why the others are wrong:**
- C: On-demand mode helps with overall unpredictable traffic *volume*, but it does not eliminate hot-partition throttling caused by one key receiving disproportionate traffic relative to DynamoDB's underlying partition-level throughput limits — a sufficiently hot single key can still be throttled even under on-demand mode.
- D: This contradicts the scenario's own data: most keys are "barely touched" while one key is overloaded — raising total table throughput mostly adds unused capacity to the cold keys and doesn't proportionally help the one hot key beyond its own partition-level ceiling.
- E: DynamoDB does distribute partitions, but traffic *to* a single partition key is still bounded by that key's own partition throughput limits — uneven application-level access patterns absolutely can and do cause hot-partition throttling; this option denies a real, commonly-tested failure mode.

---

## Q8

After a major marketing campaign, a company's AWS bill increased substantially. Leadership wants to
know whether the current architecture (several large, always-on EC2 instances running a backend
service) could meet the same performance requirements at lower cost, and wants a structured,
AWS-recommended review rather than guesswork. What should the team use per Task 4.3?

A. Immediately terminate the largest EC2 instances without any analysis, since smaller is always cheaper.

B. Use the Well-Architected Tool's cost optimization pillar review alongside Trusted Advisor/Cost Explorer to structurally evaluate cost/performance tradeoffs (e.g. right-sizing, Graviton, Spot, or moving parts of the workload to serverless) before making architecture changes.

C. Switch the entire workload to the largest available instance type, since bigger instances are always more cost-efficient per request at scale.

D. Ignore the cost increase since campaign-driven traffic spikes are temporary and don't warrant architectural review.

**Correct Answer: B**

**Explanation:** Task 4.3's cost/performance tradeoff guidance explicitly points to a structured
review rather than guessing: "cost grew significantly after a traffic increase and the team needs
to know if a cheaper architecture meets the same requirements → the cost/performance tradeoff — check
Trusted Advisor/Cost Explorer and the Well-Architected Tool's cost pillar review, don't guess." See
[`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md),
[`services/management-governance/well-architected-tool.md`](../../../services/management-governance/well-architected-tool.md),
and [`services/management-governance/cost-management-tools.md`](../../../services/management-governance/cost-management-tools.md).

**Why the others are wrong:**
- A: Terminating large instances with no analysis risks breaking the performance requirements the architecture was sized for — this is guesswork, exactly what the task guidance warns against.
- C: "Bigger is always more cost-efficient" is not generally true, and this option doesn't even attempt to address the stated goal of *reducing* cost — it increases it further with no justification.
- D: Dismissing a substantial cost increase without review ignores the explicit requirement that leadership wants to know whether a cheaper architecture meets the same needs — this isn't a valid response to the stated ask.

---

## Q9

A service composed of six separate Lambda functions needs a single view showing the overall error
rate and latency trend across all six functions combined, over the last 24 hours, for an on-call
engineer to check at a glance. What is the best way to build this per Task 4.2/4.3?

A. Open six separate default per-function CloudWatch dashboards and mentally combine the numbers each time.

B. Build a single CloudWatch dashboard using a custom metric math expression that combines the relevant metrics (e.g. summed errors, averaged latency) across all six functions into one chart.

C. Ask each engineer to `tail` the raw CloudWatch Logs of all six functions simultaneously in separate terminal windows.

D. Delete five of the six functions to reduce the number of dashboards needed.

**Correct Answer: B**

**Explanation:** Task 4.2/4.3's observability guidance calls for exactly this: "need a single
chart to understand overall error rate and latency trend over the last 24 hours for a service made
of 6 different Lambda functions → a CloudWatch dashboard with a custom metric math expression, not
six separate default dashboards." See
[`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md)
and [`services/management-governance/cloudwatch.md`](../../../services/management-governance/cloudwatch.md).

**Why the others are wrong:**
- A: Mentally combining six separate dashboards is slow, error-prone, and exactly the "at a glance" requirement this approach fails to deliver — it's the explicitly-called-out wrong answer in the domain doc.
- C: Tailing raw logs across six terminals shows text output, not an aggregated error-rate/latency trend chart, and doesn't scale as a daily monitoring practice.
- D: Deleting functions to reduce dashboard count doesn't preserve the application's actual required functionality — it "solves" the symptom by breaking the system.

---

## Q10

During an incident, an AI assistant ingests the relevant CloudWatch Logs and X-Ray trace data and
proposes: "the root cause is a misconfigured retry policy causing duplicate writes." The on-call
engineer is under pressure to resolve the incident quickly. Per Task 4.1's AI-assisted error
analysis guidance, what is the correct next step?

A. Immediately apply a fix for the AI's stated root cause and close the incident, since the AI already analyzed the actual CloudWatch/X-Ray data.

B. Treat the AI's proposed root cause as a hypothesis to verify against the actual CloudWatch/X-Ray data before acting, since the assistant can produce a plausible-sounding but incorrect explanation (a hallucination), rather than treating it as a confirmed diagnosis.

C. Ignore the AI's output entirely and refuse to use AI-assisted error analysis under any circumstances during incidents.

D. Escalate the incident only because an AI tool was involved in the analysis, regardless of what the actual data shows.

**Correct Answer: B**

**Explanation:** Task 4.1 and the cross-domain AI-assisted development doc both explicitly warn:
"an AI tool suggests a specific root cause for a production incident from the logs it was given →
treat it as a hypothesis to verify against the actual CloudWatch/X-Ray data, not a confirmed
diagnosis" — the assistant can hallucinate a plausible-sounding but wrong explanation. See
[`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md)
and [`../domain-1-development/ai-assisted-development.md`](../domain-1-development/ai-assisted-development.md#when-not-to-use-it--limits).

**Why the others are wrong:**
- A: Acting on an unverified AI-proposed root cause risks applying the wrong fix (and potentially introducing a new problem) while the real cause remains unaddressed — exactly the "don't trust AI-generated root-cause explanations as ground truth" warning in the canonical doc.
- C: Blanket refusal to use AI-assisted analysis isn't the tested correct behavior — the tool is explicitly useful for summarizing logs/traces quickly during triage; the correct discipline is verification, not avoidance.
- D: Escalation decisions should be driven by the actual severity/data of the incident, not merely by whether an AI tool was involved in generating a hypothesis — this doesn't reflect any real guidance in the domain content.

---

## Q11

A recurring class of incidents (a specific downstream API occasionally returning 503s) has a known,
well-understood remediation: restart a specific background worker process. The team wants this
remediation to happen automatically and consistently once the known error pattern is detected in
CloudWatch Logs, without waiting on a human to be paged and manually SSH into anything. What AWS
service is designed for this kind of automated runbook/remediation?

A. AWS CloudTrail, since it can automatically restart processes when it detects a logged API call.

B. AWS Systems Manager (e.g. Automation documents / runbooks), triggered by a CloudWatch Logs metric filter or alarm, to execute the known remediation steps automatically and consistently.

C. Amazon GuardDuty, since it can execute arbitrary remediation scripts on any detection.

D. AWS Config, since it can restart processes whenever it detects configuration drift.

**Correct Answer: B**

**Explanation:** Task 4.2 explicitly covers "automating remediation/runbooks once an issue is
found" via Systems Manager — a CloudWatch alarm/metric filter can trigger an SSM Automation
document that runs the known, pre-tested remediation steps consistently, removing the need for a
human to manually intervene for a well-understood, recurring problem. See
[`../domain-4-troubleshooting-optimization/README.md`](../domain-4-troubleshooting-optimization/README.md)
and [`services/management-governance/systems-manager.md`](../../../services/management-governance/systems-manager.md).

**Why the others are wrong:**
- A: CloudTrail is a passive API-call audit log; it has no built-in mechanism to execute remediation actions when a pattern appears — it isn't an automation/runbook execution service.
- C: GuardDuty is a threat-detection service that produces findings; it is not designed as a general-purpose remediation-execution engine for arbitrary application-level incidents like a stuck worker process.
- D: AWS Config detects and records configuration drift over time; it doesn't restart application processes and isn't triggered by application log patterns like a 503 error burst.
