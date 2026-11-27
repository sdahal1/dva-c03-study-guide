# Domain 3 Practice Questions — Testing and Deployment (22%)

11 questions covering Task 3.1 (preparing deployment artifacts: ECR, config externalization, API
Gateway stages), Task 3.2 (automated testing: IaC validation, mock events, AI-assisted test
generation), and Task 3.3 (CI/CD deployment: Lambda packaging, ECS/EKS/Fargate, CodeDeploy/
CodePipeline, deployment strategies, AI-assisted approvals). See
[`../domain-3-testing-deployment/README.md`](../domain-3-testing-deployment/README.md) for the
full domain outline. 2 questions use the **(Choose TWO)** multi-select format, and 2 questions
(Q7, Q11) cover AI-assisted development patterns in this domain.

---

## Q1

A team pushes a new container image to Amazon ECR on every commit to `main`, tagging each image
with the Git commit SHA. After six months, the repository has thousands of old images, most from
abandoned feature branches, consuming storage and making the console hard to navigate. What is the
correct way to automatically clean this up as part of Task 3.1's artifact preparation?

A. Manually review and delete old images from the ECR console once a week.

B. Configure an ECR lifecycle policy that automatically expires images matching a rule (e.g. untagged images older than 14 days, or keep only the most recent N tagged images per repository).

C. Set the ECR repository's storage class to Glacier to reduce cost without needing to delete anything.

D. Disable image tagging entirely so every push overwrites the same `latest` tag.

**Correct Answer: B**

**Explanation:** ECR lifecycle policies are purpose-built for exactly this: rule-based automatic
expiration of old/untagged images (e.g. "expire untagged images after 14 days" or "keep only the
last 10 tagged images"), removing the need for manual cleanup. This is Task 3.1's "build container
images for ECR... tagging, lifecycle policies." See
[`services/containers/ecr.md`](../../../services/containers/ecr.md).

**Why the others are wrong:**
- A: Manual weekly review doesn't scale and is exactly the toil lifecycle policies exist to automate — it's also error-prone (risk of deleting an image still referenced by a running deployment).
- C: ECR doesn't have a "Glacier storage class" option for container images — this isn't a real ECR feature.
- D: Overwriting a single `latest` tag removes any ability to roll back to a specific prior commit's image, which breaks traceability and defeats the whole point of commit-SHA tagging; it doesn't solve the "cleanup" problem, it just removes version history.

---

## Q2

A Lambda function's configuration currently has its database hostname, feature-flag values, and
third-party API base URL hardcoded directly into the deployed source code. Every time one of these
values changes, the team must rebuild and redeploy the function. Per Task 3.1, what is the correct
fix?

A. Keep the values in code but add more comments explaining what each one does.

B. Externalize the configuration into AWS AppConfig or Systems Manager Parameter Store, and have the function read these values at runtime (or via a Lambda extension/cache), so configuration changes don't require a code redeploy.

C. Store the values as Lambda environment variables set once at function creation and never update them again.

D. Move the values into the API Gateway resource policy.

**Correct Answer: B**

**Explanation:** Task 3.1 explicitly calls for externalizing configuration (AppConfig, Parameter
Store, Secrets Manager) rather than baking it into the deployment artifact — this decouples
configuration changes from code deployments entirely, letting ops update a flag or endpoint without
touching the Lambda package. See [`services/management-governance/systems-manager.md`](../../../services/management-governance/systems-manager.md)
and [`services/security-identity-compliance/secrets-manager-vs-parameter-store.md`](../../../services/security-identity-compliance/secrets-manager-vs-parameter-store.md).

**Why the others are wrong:**
- A: Comments don't change the fact that changing a value still requires a code change and redeploy — this doesn't solve the stated problem at all.
- C: Environment variables set once and never updated reproduce the same problem in a different location (still requires a function update/redeploy to change) — the fix is specifically to externalize to a service designed for *runtime* lookups, not a static config baked at deploy time.
- D: API Gateway resource policies control who can invoke the API — they have no mechanism for storing arbitrary application configuration values.

---

## Q3

An API has `dev`, `test`, and `prod` stages, each needing different throttle limits and a different
backend Lambda alias, and the team wants this stage-specific configuration to be part of the
versioned deployment artifact rather than manually clicked into the console after each deploy. How
should this be achieved for Task 3.1?

A. Define the API Gateway stage configuration (stage variables, throttling settings) in the IaC template (SAM/CDK/CloudFormation) alongside the rest of the deployment, so it's version-controlled and reproducible.

B. Manually configure each stage via the console immediately after every deployment, documenting the steps in a wiki.

C. Use a single shared stage for all three environments, relying on separate AWS accounts to distinguish them instead.

D. Store stage configuration in a spreadsheet maintained by the operations team outside of version control.

**Correct Answer: A**

**Explanation:** Task 3.1 treats API Gateway stage configuration as part of the deployable
artifact — defining it in IaC (SAM/CDK/CloudFormation) makes it reproducible, versioned, and
automatically applied by the pipeline rather than a manual, error-prone post-deploy step. See
[`services/networking-content-delivery/api-gateway.md`](../../../services/networking-content-delivery/api-gateway.md).

**Why the others are wrong:**
- B: Manual console clicks after every deployment are exactly the toil and configuration-drift risk IaC exists to eliminate, and documentation in a wiki doesn't enforce consistency the way code does.
- C: Using one shared stage for three environments defeats the purpose of having distinct dev/test/prod stages with separate throttle limits and aliases, and risks dev/test traffic affecting prod configuration.
- D: A spreadsheet outside version control has no enforcement mechanism and can drift silently from what's actually deployed — it's not a deployment artifact at all.

---

## Q4

Before a CloudFormation/SAM template is applied to any real AWS account, the team wants to catch
syntax errors and obvious misconfigurations automatically as part of the CI pipeline, without
spending time/cost provisioning real resources to find out something is broken. What should run in
the CodeBuild stage before the deploy stage, per Task 3.2?

A. Nothing — CloudFormation only reports errors after attempting to create resources, so pre-deploy validation isn't possible.

B. Run template validation/linting tools (e.g. `sam validate`, `cfn-lint`, or `cdk diff` for CDK) against the template as a build step, failing the pipeline early if the template is malformed.

C. Deploy the template to production directly and roll back immediately if it fails.

D. Manually read through the YAML/JSON template line by line before every deployment.

**Correct Answer: B**

**Explanation:** Task 3.2 expects automated IaC validation before deployment — running
`sam validate`/`cfn-lint`/`cdk diff` style checks as a CodeBuild step catches syntax errors and
obvious misconfigurations before anything touches a live environment, which is cheaper and faster
than discovering problems mid-deploy. See [`services/developer-tools/sam.md`](../../../services/developer-tools/sam.md),
[`services/management-governance/cloudformation.md`](../../../services/management-governance/cloudformation.md),
and [`services/management-governance/cdk.md`](../../../services/management-governance/cdk.md).

**Why the others are wrong:**
- A: CloudFormation/SAM/CDK all provide pre-apply validation tooling specifically to catch errors before real resources are touched — this option describes a real limitation of a much older/manual workflow, not current best practice.
- C: Deploying straight to production and relying on rollback-after-failure is the opposite of validating early — it risks real customer impact for an error that validation tooling could have caught for free before deploy.
- D: Manual line-by-line review doesn't scale, is error-prone, and is exactly the kind of repetitive check automated linting/validation tools exist to replace.

---

## Q5

A Lambda function processes S3 upload events, but the team wants to run it through its unit/
integration test suite in CI without actually uploading a real file to a real S3 bucket for every
test run. What is the correct Task 3.2 approach?

A. Provision a dedicated S3 bucket solely for CI and perform a real upload before every single test run.

B. Use a mock/test event that simulates the shape of an S3 `PutObject` event notification (e.g. the console's built-in S3 Put test event template) and invoke the function's handler directly with that payload.

C. Skip testing the S3-triggered path entirely since it's "too hard" to test without live infrastructure.

D. Only test this function manually in the production environment after every deployment.

**Correct Answer: B**

**Explanation:** Task 3.2 explicitly calls for mock APIs and test events to exercise Lambda
functions without live dependencies — using a template S3 event payload lets the handler's actual
processing logic be tested directly, in isolation, without provisioning or touching real S3
infrastructure. See [`../domain-3-testing-deployment/README.md`](../domain-3-testing-deployment/README.md)
("A Lambda function's test events need to simulate an S3 upload without actually uploading a file").

**Why the others are wrong:**
- A: Provisioning real infrastructure and performing real uploads for every CI run is slower, costs more, and introduces external dependencies/flakiness into what should be a fast, isolated unit test.
- C: Skipping test coverage for a significant code path (the actual trigger logic) leaves a real correctness gap — Task 3.2 exists specifically to make this kind of path testable, not to excuse skipping it.
- D: Testing only in production after deployment means defects are caught after they're already live and potentially affecting real users — the opposite of "test before deploy."

---

## Q6

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

**Explanation:** Task 3.3 tests the zip-vs-container-image packaging decision: container images
support up to 10 GB and give full control over the OS/runtime environment via a Dockerfile, making
them the right choice when a function needs large or custom native dependencies that standard zip
packages/layers can't comfortably accommodate. See [`services/compute/lambda.md`](../../../services/compute/lambda.md)
and [`services/containers/ecr.md`](../../../services/containers/ecr.md).

**Why the others are wrong:**
- A: Lambda has supported container image packaging since 2020 — this option states a false limitation.
- C: Splitting a single large native dependency across many layers doesn't change the combined size limits (zip + layers has its own aggregate cap) and adds unnecessary complexity rather than solving the actual OS-dependency-control problem.
- D: A binary/OS-level dependency cannot be meaningfully "stored in a DynamoDB item and read at runtime" the way application config can — this isn't a coherent packaging strategy for native libraries.

---

## Q7

A team wants to speed up writing regression tests for a newly refactored payment-processing Lambda
function. A developer uses an AI coding assistant to generate a first batch of unit tests covering
the function's main code paths and several edge cases the developer hadn't initially considered.
Per Task 3.2's AI-assisted testing guidance, what should happen next?

A. Merge the AI-generated tests immediately with no review, since generating tests is the slow part and the AI has already done the hard work.

B. A human reviews the AI-generated tests for correctness and coverage gaps (the AI may have missed domain-specific edge cases or asserted the wrong expected behavior), adds any missing cases, and only then merges them into the suite — using the AI output as an accelerant, not a final deliverable.

C. Discard all AI-generated tests and write every test by hand, since AI assistance is never appropriate for test code.

D. Use the AI-generated tests as the only tests ever needed for this function going forward, skipping future manual test additions.

**Correct Answer: B**

**Explanation:** This is Domain 3's instance of the cross-domain AI-assisted development pattern:
an assistant can propose unit/regression tests (Task 3.2), but a human still reviews for
correctness and coverage before merging — the AI accelerates the first draft, it doesn't replace
review. See [`../domain-1-development/ai-assisted-development.md`](../domain-1-development/ai-assisted-development.md#how-it-works--the-pattern-across-the-sdlc)
("Domain 3 — test generation & deployment approvals").

**Why the others are wrong:**
- A: Merging without review risks shipping tests that assert incorrect expected behavior (a bad test can be worse than no test, by giving false confidence) — the exam explicitly tests recognizing that AI output still needs human verification.
- C: Blanket rejection of AI assistance for test generation isn't the tested correct pattern — the exam tests *safe usage with review*, not avoidance.
- D: Treating AI-generated tests as a permanent, complete, never-updated suite ignores that test coverage should evolve with the code — this isn't the AI-assisted pattern being tested, which is about an accelerant within an ongoing review process.

---

## Q8 (Choose TWO)

A production deployment must have near-zero downtime, and if the new version has a problem after
going live, the team needs to be able to revert to the exact previous environment instantly,
without waiting for infrastructure to be re-provisioned. Which TWO characteristics describe why a
blue/green deployment strategy (via CodeDeploy) is the correct choice over a rolling deployment for
this requirement?

A. Blue/green keeps the previous ("blue") environment fully intact and running in parallel while the new ("green") environment receives traffic, so reverting means simply shifting traffic back to blue — an environment that was never torn down.

B. Rolling deployments replace instances/tasks gradually in place, so there's no single point where "the entire previous environment" still exists as a unit to instantly flip back to — a full rollback effectively means a second gradual rollout of the old version.

C. Rolling deployments are always faster than blue/green, which is the only relevant factor for this requirement.

D. Blue/green deployments cannot use health checks, so they roll out blindly and must rely on CloudWatch alarms for rollback.

E. Rolling deployments require manual approval gates while blue/green deployments never do.

**Correct Answer: A and B**

**Explanation:** This matches the domain doc's exam-scenario explanation directly: blue/green's
full environment duplication is what makes an instant, complete rollback possible (A), while
rolling deployments gradually replace instances with no equivalent single "flip back" point (B) —
reverting a rolling deployment is itself a second gradual rollout, not instant. See
[`../domain-3-testing-deployment/README.md`](../domain-3-testing-deployment/README.md) and
[`services/developer-tools/codedeploy.md`](../../../services/developer-tools/codedeploy.md).

**Why the others are wrong:**
- C: Deployment speed isn't "the only relevant factor" for this requirement — the requirement specifically asks about instant, complete rollback capability, which is an architectural property, not a raw-speed comparison, and speed alone doesn't satisfy "revert instantly to the exact previous environment."
- D: Blue/green deployments do support health checks against the green environment before traffic shifts, and CloudWatch alarms can additionally trigger automatic rollback during traffic shifting — the option's claim that health checks can't be used is false.
- E: Manual approval gates are a CodePipeline/CodeDeploy configuration choice independent of deployment strategy — neither strategy inherently requires or forbids them.

---

## Q9

A team wants to reduce the blast radius of a potentially bad new version by exposing it to only
10% of production traffic initially, automatically monitoring for errors, and only then shifting
the remaining 90% if metrics stay healthy. Which deployment strategy and tool combination correctly
implements this?

A. A rolling deployment via Auto Scaling, replacing instances evenly with no traffic-percentage control.

B. A canary deployment via CodeDeploy, which shifts a small percentage of traffic to the new version first, waits, monitors, and then shifts the rest if healthy.

C. A blue/green deployment with an immediate 100% traffic cutover and no intermediate percentage step.

D. Manually redirecting traffic using Route 53 weighted routing with no automated health monitoring or rollback tied to the deployment.

**Correct Answer: B**

**Explanation:** Task 3.3's canary deployment pattern is exactly "only a small percentage of users
should see the new version first, to limit risk before a full rollout" — CodeDeploy supports
canary traffic-shifting configurations (e.g. 10% then wait, then 100%) with CloudWatch alarm-based
rollback. See [`../domain-3-testing-deployment/README.md`](../domain-3-testing-deployment/README.md)
and [`services/developer-tools/codedeploy.md`](../../../services/developer-tools/codedeploy.md).

**Why the others are wrong:**
- A: A standard rolling deployment doesn't offer an explicit traffic-percentage split stage the way canary does — it gradually replaces capacity rather than deliberately exposing a fixed fraction of live traffic to the new version for a monitored period.
- C: Immediate 100% cutover with no intermediate step is exactly what the scenario wants to avoid ("only 10% initially") — this describes a plain blue/green cutover without the gradual canary ramp.
- D: Manual traffic redirection with no automated monitoring/rollback doesn't meet "automatically monitoring for errors" and reintroduces manual toil and risk the deployment strategy should automate.

---

## Q10

A CodePipeline is configured to run unit tests, build a container image, push it to ECR, and
deploy straight to production on every successful build with no gate in between. After a bad
deployment causes an outage, the team wants to add a safeguard without slowing down every single
low-risk change to a crawl. What should be added?

A. A manual approval action in CodePipeline positioned before the production deploy stage, so a human must explicitly approve before the change is promoted — while keeping earlier stages (build, test) fully automated.

B. Remove CodePipeline entirely and have engineers deploy manually from their laptops going forward.

C. Add a second, slower CodeBuild stage that simply repeats the exact same unit tests already run in the first stage.

D. Disable automatic triggering so pipelines must be started by hand every time, but keep the direct-to-prod deploy with no approval gate once started.

**Correct Answer: A**

**Explanation:** Task 3.3 explicitly calls for a manual approval gate before production changes
— "an AI tool suggests a deployment is safe, but the team still wants a person to confirm before
production changes go live → manual approval action in CodePipeline, informed but not replaced by
the AI flag" — this adds a safeguard at exactly the right point (before prod) without slowing down
the automated build/test stages that run on every change. See
[`../domain-3-testing-deployment/README.md`](../domain-3-testing-deployment/README.md) and
[`services/developer-tools/codepipeline.md`](../../../services/developer-tools/codepipeline.md).

**Why the others are wrong:**
- B: Abandoning CI/CD automation entirely for manual laptop deploys removes the testing/build consistency CodePipeline provides and isn't a safeguard, it's a regression to a riskier, less repeatable process.
- C: Repeating the identical test stage doesn't add any new safety check — if the same tests didn't catch the bad deployment the first time, running them again produces the same result.
- D: Requiring a human to manually *start* the pipeline doesn't add any check on *what gets promoted to production* once it's running — the actual gap (no review before prod) remains unaddressed.

---

## Q11 (Choose TWO)

A CodePipeline includes a manual approval stage before deploying to production, and the team adds
an AI tool that reviews each pending change's CDK diff and flags anything it considers risky (e.g.
a diff that deletes a DynamoDB table) for the human approver to see alongside the change. Which TWO
statements correctly describe how this should be used per Task 3.3's AI-assisted deployment
approvals guidance?

A. The AI's risk flag is additional context for the human approver, not a replacement for their judgment — the human still must explicitly approve before the pipeline promotes the change.

B. If the AI tool does not flag a change as risky, the approval stage can be safely skipped automatically for that change, since the AI's silence is equivalent to human sign-off.

C. The AI summarizing/flagging a diff (e.g. noting a destructive resource change) is a legitimate, useful way to give the human approver more context before they decide.

D. Because the AI assists with the decision, CodeDeploy can be configured to proceed automatically without any manual approval action at all, since human review is now redundant.

E. The AI tool should be given IAM permissions to approve or reject the pipeline stage itself, removing the human from the loop for efficiency.

**Correct Answer: A and C**

**Explanation:** Task 3.3's AI-assisted deployment approvals pattern keeps a human in the loop: an
AI tool can usefully surface risk in a diff (C) to inform the decision, but it never replaces the
human approval gate (A) — "the AI never *replaces* the approval gate — it informs it." See
[`../domain-1-development/ai-assisted-development.md`](../domain-1-development/ai-assisted-development.md#how-it-works--the-pattern-across-the-sdlc)
and [`../domain-3-testing-deployment/README.md`](../domain-3-testing-deployment/README.md)
("Skipping this gate for production changes is a common exam wrong-answer trap").

**Why the others are wrong:**
- B: Treating the AI's silence as equivalent to a human sign-off removes the human check entirely for "unflagged" changes — exactly the kind of unattended, irreversible-without-review action the exam marks wrong, since the AI can miss risks (false negatives) just as it can raise false positives.
- D: Configuring full automatic promotion with no manual approval action directly contradicts the requirement that a human still clicks approve for production changes — this is explicitly the wrong-answer pattern the domain doc warns about.
- E: Granting the AI tool permission to itself approve/reject the pipeline stage removes the human from the loop entirely, which is the opposite of the tested correct pattern (AI informs, human decides).
