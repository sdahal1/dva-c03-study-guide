# Task 3.3 practice questions

## Q1

**Skills:** 3.3.1

A team deploys Lambda functions in two different ways across their account: some as traditional
`.zip` deployment packages, and others as container images pushed to ECR, with runtimes exceeding
the standard zip package size limits and requiring custom OS-level dependencies. Per Task 3.3, which
packaging approach should new functions that need large custom dependencies (e.g. a sizeable native
image-processing library with OS packages) use?

A. Always use `.zip` packages, since Lambda doesn't support container images.

B. Use a container image deployment, since it supports a larger size limit (10 GB) and lets the team fully control the OS-level dependencies/runtime via a Dockerfile, which `.zip` packages with standard runtimes can't offer as flexibly.

C. Split the dependency across 20 separate Lambda layers to stay under the zip size limit.

D. Move the dependency into a DynamoDB item and read it at runtime.

**Correct Answer: B**

**Explanation:** Task 3.3 tests the zip-versus-container-image packaging decision: container images
support up to 10 GB and give full control over the OS and runtime environment through a Dockerfile, making
them the right choice when a function needs large or custom native dependencies that standard zip
packages and layers can't comfortably accommodate. See [3.3.1 Lambda packaging](3.3.1-lambda-packaging.md#the-three-packaging-methods)
and [3.1.2 Container images and ECR](../task-3.1-prepare-deployable-artifacts/3.1.2-container-images-and-ecr.md).

**Why the others are wrong:**
- A: Lambda has supported container image packaging since 2020. This option states a false limitation.
- C: A function can use at most 5 layers, and the function plus all layers must stay within 250 MB unzipped. Splitting a large native dependency across layers doesn't change that combined limit, adds complexity, and doesn't solve the OS-dependency-control problem.
- D: A binary or OS-level dependency cannot be meaningfully "stored in a DynamoDB item and read at runtime" the way application config can. This isn't a coherent packaging strategy for native libraries.

---

## Q2 (Choose TWO)

**Skills:** 3.3.6, 3.3.7, 3.3.4

A production deployment must have near-zero downtime, and if the new version has a problem after
going live, the team needs to be able to revert to the exact previous environment instantly,
without waiting for infrastructure to be re-provisioned. Which TWO characteristics describe why a
blue/green deployment strategy (via CodeDeploy) is the correct choice over a rolling deployment for
this requirement?

A. Blue/green keeps the previous ("blue") environment fully intact and running in parallel while the new ("green") environment receives traffic, so reverting means simply shifting traffic back to blue, an environment that was never torn down.

B. Rolling deployments replace instances/tasks gradually in place, so there's no single point where "the entire previous environment" still exists as a unit to instantly flip back to. A full rollback effectively means a second gradual rollout of the old version.

C. Rolling deployments are always faster than blue/green, which is the only relevant factor for this requirement.

D. Blue/green deployments cannot use health checks, so they roll out blindly and must rely on CloudWatch alarms for rollback.

E. Rolling deployments require manual approval gates while blue/green deployments never do.

**Correct Answer: A and B**

**Explanation:** Blue/green's full environment duplication is what makes an instant, complete
rollback possible (A), while rolling deployments gradually replace instances with no equivalent
single "flip back" point (B). Reverting a rolling deployment is itself a second gradual rollout, not
instant. See [3.3.6 Deployment strategies](3.3.6-deployment-strategies.md),
[3.3.7 Rollback](3.3.7-rollback.md), and [3.3.4 CodeDeploy](3.3.4-codedeploy.md).

**Why the others are wrong:**
- C: Deployment speed isn't "the only relevant factor" for this requirement. The requirement asks about instant, complete rollback capability, which is an architectural property, not a raw-speed comparison, and speed alone doesn't satisfy "revert instantly to the exact previous environment."
- D: Blue/green deployments do support health checks against the green environment before traffic shifts, and CloudWatch alarms can additionally trigger automatic rollback during traffic shifting. The claim that health checks can't be used is false.
- E: Manual approval gates are a CodePipeline/CodeDeploy configuration choice independent of deployment strategy. Neither strategy inherently requires or forbids them.

---

## Q3

**Skills:** 3.3.6, 3.3.4

A team wants to reduce the blast radius of a potentially bad new version by exposing it to only
10% of production traffic initially, automatically monitoring for errors, and only then shifting
the remaining 90% if metrics stay healthy. Which deployment strategy and tool combination correctly
implements this?

A. A rolling deployment via Auto Scaling, replacing instances evenly with no traffic-percentage control.

B. A canary deployment via CodeDeploy, which shifts a small percentage of traffic to the new version first, waits, monitors, and then shifts the rest if healthy.

C. A blue/green deployment with an immediate 100% traffic cutover and no intermediate percentage step.

D. Manually redirecting traffic using Route 53 weighted routing with no automated health monitoring or rollback tied to the deployment.

**Correct Answer: B**

**Explanation:** The canary deployment pattern is exactly "only a small percentage of users
should see the new version first, to limit risk before a full rollout." CodeDeploy supports
canary traffic-shifting configurations (for example `CodeDeployDefault.LambdaCanary10Percent5Minutes`:
10%, wait 5 minutes, then 100%) with CloudWatch alarm-based rollback. See
[3.3.6 Deployment strategies](3.3.6-deployment-strategies.md) and
[3.3.4 CodeDeploy](3.3.4-codedeploy.md#compute-platforms-compared).

**Why the others are wrong:**
- A: A standard rolling deployment doesn't offer an explicit traffic-percentage split the way canary does. It gradually replaces capacity rather than deliberately exposing a fixed fraction of live traffic to the new version for a monitored period.
- C: Immediate 100% cutover with no intermediate step is exactly what the scenario wants to avoid ("only 10% initially"). This describes a plain blue/green cutover without the gradual canary ramp.
- D: Manual traffic redirection with no automated monitoring or rollback doesn't meet "automatically monitoring for errors" and reintroduces the manual toil and risk the deployment strategy should automate.

---

## Q4

**Skills:** 3.3.5

A CodePipeline is configured to run unit tests, build a container image, push it to ECR, and
deploy straight to production on every successful build with no gate in between. After a bad
deployment causes an outage, the team wants to add a safeguard without slowing down every single
low-risk change to a crawl. What should be added?

A. A manual approval action in CodePipeline positioned before the production deploy stage, so a human must explicitly approve before the change is promoted, while keeping earlier stages (build, test) fully automated.

B. Remove CodePipeline entirely and have engineers deploy manually from their laptops going forward.

C. Add a second, slower CodeBuild stage that simply repeats the exact same unit tests already run in the first stage.

D. Disable automatic triggering so pipelines must be started by hand every time, but keep the direct-to-prod deploy with no approval gate once started.

**Correct Answer: A**

**Explanation:** Task 3.3 calls for a manual approval gate before production changes. When an AI
tool suggests a deployment is safe but the team still wants a person to confirm before production
changes go live, the answer is a manual approval action in CodePipeline, informed but not replaced
by the AI flag. This adds a safeguard at exactly the right point (before prod) without slowing down
the automated build and test stages that run on every change. See
[3.3.5 CodePipeline release promotion](3.3.5-codepipeline-release-promotion.md#manual-approval-actions).

**Why the others are wrong:**
- B: Abandoning CI/CD automation for manual laptop deploys removes the testing and build consistency CodePipeline provides. It isn't a safeguard; it's a regression to a riskier, less repeatable process.
- C: Repeating the identical test stage doesn't add any new safety check. If the same tests didn't catch the bad deployment the first time, running them again produces the same result.
- D: Requiring a human to manually *start* the pipeline doesn't add any check on *what gets promoted to production* once it's running. The actual gap (no review before prod) remains.

---

## Q5 (Choose TWO)

**Skills:** 3.3.9, 3.3.5

A CodePipeline includes a manual approval stage before deploying to production, and the team adds
an AI tool that reviews each pending change's CDK diff and flags anything it considers risky (e.g.
a diff that deletes a DynamoDB table) for the human approver to see alongside the change. Which TWO
statements correctly describe how this should be used per Task 3.3's AI-assisted deployment
approvals guidance?

A. The AI's risk flag is additional context for the human approver, not a replacement for their judgment. The human still must explicitly approve before the pipeline promotes the change.

B. If the AI tool does not flag a change as risky, the approval stage can be safely skipped automatically for that change, since the AI's silence is equivalent to human sign-off.

C. The AI summarizing/flagging a diff (e.g. noting a destructive resource change) is a legitimate, useful way to give the human approver more context before they decide.

D. Because the AI assists with the decision, CodeDeploy can be configured to proceed automatically without any manual approval action at all, since human review is now redundant.

E. The AI tool should be given IAM permissions to approve or reject the pipeline stage itself, removing the human from the loop for efficiency.

**Correct Answer: A and C**

**Explanation:** The AI-assisted deployment approvals pattern keeps a human in the loop. An
AI tool can usefully surface risk in a diff (C) to inform the decision, but it never replaces the
human approval gate (A): the AI informs the approval gate, it does not replace it. Skipping this gate for
production changes is a common exam wrong-answer trap. See
[3.3.9 AI-assisted deployment](3.3.9-ai-assisted-deployment.md),
[3.3.5 CodePipeline release promotion](3.3.5-codepipeline-release-promotion.md#manual-approval-actions), and
[1.1.9 AI-assisted development tools](../../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md).

**Why the others are wrong:**
- B: Treating the AI's silence as equivalent to a human sign-off removes the human check entirely for "unflagged" changes. That is the kind of unattended, irreversible-without-review action the exam marks wrong, since the AI can miss risks (false negatives) just as it can raise false positives.
- D: Configuring full automatic promotion with no manual approval action directly contradicts the requirement that a human still clicks approve for production changes. This is the wrong-answer pattern the exam warns about.
- E: Granting the AI tool permission to approve or reject the pipeline stage itself removes the human from the loop entirely, which is the opposite of the tested correct pattern (AI informs, human decides).
