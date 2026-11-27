# Task 4.1 practice questions

## Q1

**Skills:** 4.1.2

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
X-Ray (where in a multi-hop distributed chain the time went). When you need to see which specific
service in a chain of calls is adding latency, use the X-Ray service map and trace segments, not
just CloudWatch Logs: logs alone don't show cross-service timing without manual correlation. See
[4.1.2 Metrics, logs, and traces](4.1.2-metrics-logs-and-traces.md#reading-x-ray-traces).

**Why the others are wrong:**
- A: CloudTrail records *who/what AWS API call* happened and when (an audit trail), not application-level request latency across a call chain. It wouldn't show "the external payment API call took 2.1 seconds."
- C: Manually correlating timestamps across four separate log groups is slow and error-prone, and it is the problem X-Ray's unified trace view exists to solve. It is possible but far inferior to the purpose-built tool.
- D: Increasing Lambda memory only helps if the function itself is the CPU-bound bottleneck. If the slow hop is the external payment API or a DynamoDB call, more Lambda memory does nothing, and the scenario says Lambda's own duration looks reasonable.

---

## Q2

**Skills:** 4.1.2, 4.1.3

An on-call engineer needs to find the specific stack trace / error message that caused a Lambda
function invocation to fail at 3:14 AM, including the full exception text and any custom log lines
the function emitted right before failing. Where should they look?

A. CloudTrail event history, filtered to the function's ARN.

B. CloudWatch Logs for that function's log group, searching around the 3:14 AM timestamp for the error output and preceding application log lines.

C. AWS Config's resource timeline for the Lambda function.

D. The X-Ray service map's high-level latency view only, with no drill-down into individual segments.

**Correct Answer: B**

**Explanation:** Task 4.1 distinguishes CloudWatch (application and runtime telemetry: logs,
metrics, stack traces, custom log output) from CloudTrail (the API call audit trail). Finding what
happened inside the function's own execution, including a full stack trace and custom log lines,
is CloudWatch Logs' job. A Logs Insights query filtered to that time window and the failing
request ID isolates the lines quickly. See
[4.1.3 Querying logs](4.1.3-querying-logs.md#query-patterns-by-goal).

**Why the others are wrong:**
- A: CloudTrail records API calls (for example, who called `lambda:Invoke` or who changed this function's configuration), not the function's internal runtime output such as an exception stack trace.
- C: AWS Config tracks configuration state and changes of resources over time (for example, whether someone changed this function's memory setting), not application-level error output from a single invocation. Config is also not on the DVA-C03 service list.
- D: The high-level X-Ray service map shows latency per hop. Drilling into a trace segment's exceptions is a different (valid) X-Ray use, and even then the full custom log text the function emitted is in CloudWatch Logs, not the service map view.

---

## Q3

**Skills:** 4.1.1

A production incident is being investigated and the team suspects a recent IAM policy change
accidentally revoked a Lambda function's DynamoDB write permission, starting the incident's error
spike. They need to know exactly which IAM principal made that policy change and precisely when.
Which service answers this?

A. CloudWatch Metrics, since it tracks DynamoDB throttling automatically.

B. CloudTrail event history, which logs every AWS API call (including IAM policy modifications) along with the calling principal and timestamp.

C. X-Ray, since it traces every call a Lambda function makes to DynamoDB.

D. Amazon Macie, since it monitors for sensitive configuration changes.

**Correct Answer: B**

**Explanation:** Task 4.1 includes using CloudTrail to find who or what made the API call that
caused a problem. When you need to know which IAM principal changed or deleted a resource at a
specific time, use CloudTrail event history, not CloudWatch Logs. CloudTrail is the audit trail of
*API calls*; CloudWatch is *application and runtime* telemetry. Event history keeps 90 days of
management events, which include IAM policy changes. See
[4.1.1 Debugging code and integrations](4.1.1-debugging-code-and-integrations.md#finding-what-changed-with-cloudtrail).

**Why the others are wrong:**
- A: CloudWatch Metrics would show the *symptom* (for example, a spike in errors from `AccessDeniedException`) but not *who* made the IAM policy change that caused it.
- C: X-Ray traces the application's own call chain (Lambda → DynamoDB) for latency and errors within requests. It has no visibility into IAM policy changes made outside the traced request flow.
- D: Macie discovers and classifies sensitive data at rest in S3. It does not audit IAM policy change history.

---

## Q4

**Skills:** 4.1.5

During an incident, an AI assistant ingests the relevant CloudWatch Logs and X-Ray trace data and
proposes: "the root cause is a misconfigured retry policy causing duplicate writes." The on-call
engineer is under pressure to resolve the incident quickly. Per Task 4.1's AI-assisted error
analysis guidance, what is the correct next step?

A. Immediately apply a fix for the AI's stated root cause and close the incident, since the AI already analyzed the actual CloudWatch/X-Ray data.

B. Treat the AI's proposed root cause as a hypothesis to verify against the actual CloudWatch/X-Ray data before acting, since the assistant can produce a plausible-sounding but incorrect explanation (a hallucination), rather than treating it as a confirmed diagnosis.

C. Ignore the AI's output entirely and refuse to use AI-assisted error analysis under any circumstances during incidents.

D. Escalate the incident only because an AI tool was involved in the analysis, regardless of what the actual data shows.

**Correct Answer: B**

**Explanation:** When an AI tool suggests a specific root cause for a production incident from the
logs it was given, treat it as a hypothesis to verify against the actual CloudWatch and X-Ray data,
not a confirmed diagnosis. The assistant can produce a plausible-sounding but wrong explanation,
for example by latching onto the most frequent error rather than the first, causal one. See
[4.1.5 AI-assisted error analysis](4.1.5-ai-assisted-error-analysis.md#verify-before-acting) and
[1.1.9 AI-assisted development tools](../../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md).

**Why the others are wrong:**
- A: Acting on an unverified AI root cause risks applying the wrong fix (and possibly introducing a new problem) while the real cause remains. Do not treat AI-generated root-cause explanations as ground truth.
- C: Refusing AI-assisted analysis altogether isn't the expected behavior. The tools are useful for summarizing logs and traces quickly during triage; the correct discipline is verification, not avoidance.
- D: Escalation should follow the incident's actual severity and data, not whether an AI tool helped generate a hypothesis.
