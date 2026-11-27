# Task 4.2 practice questions

## Q1

**Skills:** 4.2.1, 4.2.2

A team repeatedly struggles to debug production incidents because their Lambda functions log only
bare `print("error")` statements with no request ID, no structured fields, and no context about
which input caused the failure. The team wants to fix this proactively rather than scrambling during
the next incident. What is the correct Task 4.2 action?

A. Wait until the next incident and add better logging then, under time pressure, since that's when the specific need will be clearest.

B. Instrument the code now to emit structured logs (e.g. JSON (JavaScript Object Notation) with request ID, relevant input identifiers, and error context) and custom CloudWatch metrics for key operations, so the right signals already exist before the next incident happens.

C. Increase the Lambda function's memory allocation, since more memory produces more detailed log output automatically.

D. Disable logging entirely to reduce CloudWatch costs, since logs didn't help root-cause the last incident anyway.

**Correct Answer: B**

**Explanation:** Task 4.2 is about instrumenting code *before* an incident, not during one. The
right signals must already exist when something breaks: structured logs with context (request or
correlation ID, relevant identifiers, error detail) and custom metrics emitted from application
code. A Lambda function whose errors don't carry enough context to debug has an instrumentation
gap, and the fix belongs before the next incident. See
[4.2.1 Structured logging](4.2.1-structured-logging.md#why-structure-matters) and
[4.2.2 Custom metrics](4.2.2-custom-metrics.md).

**Why the others are wrong:**
- A: Waiting until the next incident to add logging guarantees the *current* incident (and likely the next one too, before the fix ships) remains hard to debug. Task 4.2 is about proactive instrumentation, not reactive scrambling.
- C: Lambda memory allocation controls compute and CPU (central processing unit) resources, not log verbosity or structure. More memory does not automatically produce better log content.
- D: Removing logging entirely makes future root-cause analysis strictly harder, not easier, and isn't a reasonable response to logs that were unstructured (the fix is to improve them, not eliminate them).

---

## Q2

**Skills:** 4.2.4

A development team wants to add distributed tracing to a Lambda function so that each significant
operation inside the handler (e.g. "validate input," "call DynamoDB," "call external API") shows up
as its own timed unit in the end-to-end trace, not just one big opaque Lambda invocation block. What
should they add to the application code?

A. Nothing. X-Ray automatically breaks every Lambda invocation into named subsegments for each internal function call with zero code changes.

B. Use the X-Ray SDK to create custom subsegments around each significant internal operation (e.g. `validate input`, the DynamoDB call, the external API call), giving fine-grained, named timing within the single Lambda invocation's trace segment.

C. Create a separate Lambda function for every internal operation so each shows up as its own top-level X-Ray segment.

D. Increase the Lambda function's reserved concurrency so X-Ray automatically captures more detail per invocation.

**Correct Answer: B**

**Explanation:** Skill 4.2.4 is about instrumenting distributed tracing from application code. That
means using the X-Ray SDK (or OpenTelemetry) to wrap specific internal operations as named
subsegments. AWS SDK calls (e.g. the DynamoDB call) are instrumented automatically once the X-Ray
SDK wraps or patches the client, but custom internal logic (like "validate input") needs an explicit
subsegment added by the developer. In Lambda, annotations and metadata also go on these
subsegments, because the function segment Lambda creates can't be modified from code. See
[4.2.4 Distributed tracing](4.2.4-distributed-tracing.md#instrumenting-code).

**Why the others are wrong:**
- A: X-Ray does automatically instrument supported AWS SDK calls once the SDK is wrapped/patched, but it does not automatically create subsegments for arbitrary custom internal logic like a validation step with zero code changes. That requires explicit instrumentation.
- C: Splitting internal operations into separate Lambda functions changes the application's architecture unnecessarily and adds invocation overhead/latency and IAM (AWS Identity and Access Management) complexity just to get tracing granularity that subsegments provide without restructuring the function.
- D: Reserved concurrency controls capacity guarantees for a function; it has no relationship to X-Ray trace detail or subsegment creation.

---

## Q3

**Skills:** 4.2.2

A service composed of six separate Lambda functions needs a single view showing the overall error
rate and latency trend across all six functions combined, over the last 24 hours, for an on-call
engineer to check at a glance. What is the best way to build this?

A. Open six separate default per-function CloudWatch dashboards and mentally combine the numbers each time.

B. Build a single CloudWatch dashboard using a custom metric math expression that combines the relevant metrics (e.g. summed errors, averaged latency) across all six functions into one chart.

C. Ask each engineer to `tail` the raw CloudWatch Logs of all six functions simultaneously in separate terminal windows.

D. Delete five of the six functions to reduce the number of dashboards needed.

**Correct Answer: B**

**Explanation:** A single chart for overall error rate and latency across a service made of six
Lambda functions is a CloudWatch dashboard with a metric math expression, not six separate default
dashboards. Metric math sums `Errors` and `Invocations` across the functions and divides them to
get a service-wide error rate, with no new data published. See
[4.2.2 Custom metrics](4.2.2-custom-metrics.md#combining-metrics-metric-math-and-dashboards).

**Why the others are wrong:**
- A: Mentally combining six separate dashboards is slow, error-prone, and fails the "at a glance" requirement.
- C: Tailing raw logs across six terminals shows text output, not an aggregated error-rate/latency trend chart, and doesn't scale as a daily monitoring practice.
- D: Deleting functions to reduce dashboard count doesn't preserve the application's actual required functionality. It "solves" the symptom by breaking the system.

---

## Q4

**Skills:** 4.2.2, 4.2.3

A recurring class of incidents (a specific downstream API, or application programming interface, occasionally returning 503s) has a known,
well-understood remediation: restart a specific background worker process. The team wants this
remediation to happen automatically and consistently once the known error pattern is detected in
CloudWatch Logs, without waiting on a human to be paged and manually SSH (Secure Shell) into anything. What AWS
service is designed for this kind of automated runbook/remediation?

A. AWS CloudTrail, since it can automatically restart processes when it detects a logged API call.

B. AWS Systems Manager (e.g. Automation documents / runbooks), triggered by a CloudWatch Logs metric filter or alarm, to execute the known remediation steps automatically and consistently.

C. Amazon GuardDuty, since it can execute arbitrary remediation scripts on any detection.

D. AWS Config, since it can restart processes whenever it detects configuration drift.

**Correct Answer: B**

**Explanation:** Alerting (skill 4.2.3) extends naturally into automated response. A metric filter
turns the 503 log pattern into a metric, a CloudWatch alarm watches that metric, and the alarm's
state change (routed through EventBridge) starts a Systems Manager Automation runbook that runs the
known, pre-tested remediation steps consistently. No human has to intervene manually for a
well-understood, recurring problem, though the same alarm should still notify the team. See
[4.2.3 Alerts and notifications](4.2.3-alerts-and-notifications.md#from-alert-to-automated-remediation)
and [4.2.2 Custom metrics](4.2.2-custom-metrics.md#metric-filters).

**Why the others are wrong:**
- A: CloudTrail is a passive API-call audit log; it has no built-in mechanism to execute remediation actions when a pattern appears. It isn't an automation/runbook execution service.
- C: GuardDuty is a threat-detection service that produces findings; it is not designed as a general-purpose remediation-execution engine for arbitrary application-level incidents like a stuck worker process.
- D: AWS Config detects and records configuration drift over time; it doesn't restart application processes and isn't triggered by application log patterns like a 503 error burst.
