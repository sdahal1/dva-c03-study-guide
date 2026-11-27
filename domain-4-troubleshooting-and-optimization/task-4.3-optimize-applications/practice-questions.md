# Task 4.3 practice questions

## Q1

**Skills:** 4.3.2, 4.3.5

Under load, a Lambda function running image-resizing logic intermittently times out. Profiling data
shows the function is consistently near 100% central processing unit (CPU) utilization for the full duration of each
invocation (not idle-waiting on a network call). The function's memory is currently set to the
minimum (128 MB). What is the most appropriate Task 4.3 optimization?

A. Increase the function's timeout value only, leaving memory unchanged, so the CPU-bound work has more wall-clock time to finish.
B. Increase the function's memory allocation, since Lambda allocates CPU power proportionally to configured memory. More memory directly means more CPU for a CPU-bound workload, likely reducing duration enough to avoid timeouts.
C. Decrease the function's memory allocation to reduce cost, since the timeout is unrelated to memory.
D. Add a DynamoDB Accelerator (DAX) cache in front of the function, since caching fixes CPU-bound workloads.

**Correct Answer: B**

**Explanation:** Task 4.3's canonical Lambda performance-tuning scenario: "a Lambda function times
out intermittently under load, and profiling shows it's CPU-bound, not idle-waiting → increase
memory (which scales CPU proportionally in Lambda), not just the timeout value." Lambda allocates
CPU power proportional to the configured memory, so a CPU-bound workload at minimum memory is
leaving performance on the table. See
[4.3.2 Memory and compute tuning](4.3.2-memory-and-compute-tuning.md#reading-the-evidence).

**Why the others are wrong:**
- A: Raising only the timeout doesn't make the CPU-bound work finish any faster. It just allows the same slow execution to run longer before being killed, potentially masking the problem while increasing cost per invocation (billed duration) without fixing root cause.
- C: Decreasing memory for a CPU-bound workload would make the function even slower (less CPU allocated), worsening the timeout problem, not helping cost in any meaningful way once timeouts and retries are accounted for.
- D: DAX is a caching layer for DynamoDB read latency. It has no relevance to a CPU-bound image-resizing computation happening entirely within the function's own execution.

---

## Q2 (Choose TWO)

**Skills:** 4.3.5, 4.3.4

A DynamoDB table serving a high-traffic application starts returning
`ProvisionedThroughputExceededException` errors under load. Investigating further, the team finds
that one specific partition key value (a popular "featured" item ID) receives a disproportionate
share of all requests, while most other partition keys are barely touched. Which TWO statements
correctly distinguish the diagnosis and the appropriate fix for this specific situation?

A. This is a "hot partition" access-pattern problem: a single partition key is receiving disproportionate traffic relative to the table's other keys, regardless of the table's overall provisioned or on-demand capacity.
B. The correct fix for a hot-partition problem is to redesign the access pattern (for example, add a cache in front of the hot item, or shard the hot key into multiple logical keys), not simply raise the table's overall provisioned capacity, since raising overall capacity doesn't fix uneven distribution across keys.
C. The correct fix is always to switch the table to on-demand capacity mode, which eliminates hot-partition throttling entirely regardless of key design.
D. This is purely a capacity problem unrelated to key design. Simply raising the table's total provisioned throughput will evenly resolve the throttling no matter which key is hot.
E. Since DynamoDB automatically rebalances traffic evenly across all partitions regardless of key design, this symptom could not actually be caused by an uneven key access pattern.

**Correct Answer: A and B**

**Explanation:** Task 4.3 explicitly distinguishes this exact failure mode: "DynamoDB reads are
being throttled under load → check if it's a hot partition key (access pattern issue, fix the key
design) vs. simply under-provisioned capacity (fix: on-demand mode or raise provisioned capacity).
These have different fixes despite the same symptom." A single disproportionately hot key (A) needs
an access-pattern-level fix like caching or key sharding (B). Raising overall table capacity
doesn't address the skew. CloudWatch Contributor Insights for DynamoDB is the tool that shows which
keys are hot. See
[4.3.5 Performance bottleneck detection](4.3.5-performance-bottleneck-detection.md#dynamodb-throttling-hot-key-or-capacity)
and [1.3.4 DynamoDB design and operations](../../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md#primary-keys-and-partition-design).

**Why the others are wrong:**
- C: On-demand mode helps with overall unpredictable traffic *volume*, but it does not eliminate hot-partition throttling caused by one key receiving disproportionate traffic relative to DynamoDB's underlying partition-level throughput limits. A sufficiently hot single key can still be throttled even under on-demand mode.
- D: This contradicts the scenario's own data: most keys are "barely touched" while one key is overloaded. Raising total table throughput mostly adds unused capacity to the cold keys and doesn't proportionally help the one hot key beyond its own partition-level ceiling.
- E: DynamoDB does distribute partitions, but traffic *to* a single partition key is still bounded by that key's own partition throughput limits. Uneven application-level access patterns can and do cause hot-partition throttling; this option denies a real, commonly tested failure mode.

---

## Q3

**Skills:** 4.3.6

After a major marketing campaign, a company's AWS bill increased substantially. Leadership wants to
know whether the current architecture (several large, always-on Elastic Compute Cloud (EC2) instances running a backend
service) could meet the same performance requirements at lower cost, and wants a structured,
AWS-recommended review rather than guesswork. What should the team use per Task 4.3?

A. Immediately terminate the largest EC2 instances without any analysis, since smaller is always cheaper.
B. Use the Well-Architected Tool's cost optimization pillar review alongside Trusted Advisor and Cost Explorer to structurally evaluate cost/performance tradeoffs (for example right-sizing, Graviton, Spot, or moving parts of the workload to serverless) before making architecture changes.
C. Switch the entire workload to the largest available instance type, since bigger instances are always more cost-efficient per request at scale.
D. Ignore the cost increase since campaign-driven traffic spikes are temporary and don't warrant architectural review.

**Correct Answer: B**

**Explanation:** Task 4.3's cost/performance tradeoff guidance points to a structured
review rather than guessing: "cost grew significantly after a traffic increase and the team needs
to know if a cheaper architecture meets the same requirements → the cost/performance tradeoff. Check
Trusted Advisor and Cost Explorer and the Well-Architected Tool's cost pillar review; don't guess." See
[4.3.6 AI-assisted optimization](4.3.6-ai-assisted-optimization.md#resource-and-cost-optimization-tools-outside-the-service-list).
The Well-Architected Tool, Trusted Advisor, and Cost Explorer are not on the DVA-C03 in-scope
service list, so expect this scenario mainly in older or third-party practice material.

**Why the others are wrong:**
- A: Terminating large instances with no analysis risks breaking the performance requirements the architecture was sized for. This is guesswork, exactly what the task guidance warns against.
- C: "Bigger is always more cost-efficient" is not generally true, and this option doesn't even attempt to address the stated goal of *reducing* cost. It increases it further with no justification.
- D: Dismissing a substantial cost increase without review ignores the explicit requirement that leadership wants to know whether a cheaper architecture meets the same needs. This isn't a valid response to the stated ask.
