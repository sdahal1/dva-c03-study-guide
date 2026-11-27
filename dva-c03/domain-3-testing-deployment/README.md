# Domain 3: Testing and Deployment (22%)

![AWS CodeDeploy blue/green workflow for AWS Lambda: traffic shifts from the current function version to a new version according to a deployment configuration.](https://docs.aws.amazon.com/wellarchitected/latest/framework/images/deployment-process-lambda.png)
*Source: [AWS Well-Architected Framework — OPS04-BP01 Employ and test safe deployment strategies](https://docs.aws.amazon.com/wellarchitected/latest/framework/ops04-bp01.html). This is the official shape of the exact Lambda traffic-shifting flow this domain's "how it fits together" diagram below describes in detail.*

Once code exists (Domain 1) and is reasonably secure (Domain 2), Domain 3 is about shipping it
reliably: packaging artifacts correctly, proving they work before they go live, and getting them
into production through an automated, reversible pipeline. If Domain 1 is "does it work" and
Domain 2 is "is it safe," Domain 3 is "can you get it to production without betting the business
on it going right the first time" — every control in this domain exists specifically to make a bad
deployment cheap to detect and cheap to undo.

## Task statements (verbatim from the exam guide)

**Task 3.1 — Prepare application artifacts for deployment**
- Dependency management (bundling, layers, lockfiles) — see also
  [CodeArtifact](../../../services/developer-tools/codeartifact.md) for where the packages
  themselves are resolved from in the first place
- Build container images for ECR (Dockerfiles, tagging, lifecycle policies)
- Externalize configuration (AppConfig, Parameter Store, Secrets Manager — don't bake config into
  the artifact)
- Configure API Gateway stages as part of the deployment artifact (stage-specific variables)

**Task 3.2 — Implement and automate application testing**
- Validate infrastructure templates (SAM, CDK, CloudFormation) before deploying them
- Use mock APIs and test events to exercise Lambda functions without live dependencies
- AI-assisted test generation and regression-test automation

**Task 3.3 — Deploy code via CI/CD**
- Package and deploy Lambda functions (zip vs. container image)
- Deploy to ECS/EKS/Fargate (new container-deployment emphasis in DVA-C03)
- Deploy infrastructure via IaC templates (CloudFormation/CDK/SAM)
- Use CodeDeploy for compute deployment, CodePipeline for orchestration
- Choose a deployment strategy: blue/green, canary, rolling — and configure rollback
- AI-assisted deployment approvals (human-in-the-loop gating informed by AI risk flags)

## Every task, mapped to a doc

| Task | What to read | Where |
|---|---|---|
| 3.1 — Dependency bundling, Lambda layers | [`services/compute/lambda.md`](../../../services/compute/lambda.md) |
| 3.1 — Dependency management for code modules (private npm/PyPI/Maven/NuGet packages, lockfiles resolving against a controlled, cached source instead of the public internet) | [`services/developer-tools/codeartifact.md`](../../../services/developer-tools/codeartifact.md) |
| 3.1 — Container images, ECR build/tag/lifecycle | [`services/containers/ecr.md`](../../../services/containers/ecr.md) |
| 3.1 — Container-vs-VM-vs-serverless packaging decision | [`services/containers/containers-vs-vms-vs-serverless.md`](../../../services/containers/containers-vs-vms-vs-serverless.md) |
| 3.1 — Externalized config, including "manage environment-specific application configuration" | [`services/management-governance/appconfig.md`](../../../services/management-governance/appconfig.md) for the gradual-rollout/auto-rollback config-deployment case specifically, [`services/security-identity-compliance/secrets-manager-vs-parameter-store.md`](../../../services/security-identity-compliance/secrets-manager-vs-parameter-store.md) for plain value/secret storage with no rollout concept |
| 3.1 — API Gateway stage config as deployable artifact | [`services/networking-content-delivery/api-gateway.md`](../../../services/networking-content-delivery/api-gateway.md) |
| 3.2 — IaC template authoring/validation | [`services/developer-tools/sam.md`](../../../services/developer-tools/sam.md), [`services/management-governance/cloudformation.md`](../../../services/management-governance/cloudformation.md), [`services/management-governance/cdk.md`](../../../services/management-governance/cdk.md) |
| 3.2 — Build-time test execution | [`services/developer-tools/codebuild.md`](../../../services/developer-tools/codebuild.md) |
| 3.2 — Source control triggering the pipeline | [`services/developer-tools/codecommit.md`](../../../services/developer-tools/codecommit.md) |
| 3.2 — AI-assisted test generation | Cross-link — see below |
| 3.3 — Lambda packaging & versioning | [`services/compute/lambda.md`](../../../services/compute/lambda.md) |
| 3.3 — ECS/EKS/Fargate deployment (new DVA-C03 emphasis) | [`services/containers/ecs.md`](../../../services/containers/ecs.md), [`services/containers/eks.md`](../../../services/containers/eks.md), [`services/containers/fargate.md`](../../../services/containers/fargate.md), [`services/compute/fargate.md`](../../../services/compute/fargate.md) |
| 3.3 — IaC deployment | [`services/management-governance/cloudformation.md`](../../../services/management-governance/cloudformation.md), [`services/management-governance/cdk.md`](../../../services/management-governance/cdk.md), [`services/developer-tools/sam.md`](../../../services/developer-tools/sam.md) |
| 3.3 — Compute deployment strategies (blue/green, canary, rolling) | [`services/developer-tools/codedeploy.md`](../../../services/developer-tools/codedeploy.md) |
| 3.3 — Pipeline orchestration end-to-end | [`services/developer-tools/codepipeline.md`](../../../services/developer-tools/codepipeline.md) |
| 3.3 — Low-friction dev environments to test deployment scripts from | [`services/developer-tools/cloud9-and-cloudshell.md`](../../../services/developer-tools/cloud9-and-cloudshell.md) |
| 3.3 — AI-assisted deployment approvals | Cross-link — see below |
| Supporting — the IAM identity the pipeline itself runs as | [`services/security-identity-compliance/iam.md`](../../../services/security-identity-compliance/iam.md), [`services/security-identity-compliance/sts-and-role-switching.md`](../../../services/security-identity-compliance/sts-and-role-switching.md) |
| Supporting — the metric data a rollback decision is based on | [`services/management-governance/cloudwatch.md`](../../../services/management-governance/cloudwatch.md) |

### AI-assisted development cross-link

Task 3.2's test generation and Task 3.3's deployment approvals are two of the four places the
AI-assisted development pattern shows up on this exam. Read the canonical explanation once in
Domain 1, then apply it here:

➡️ **[`../domain-1-development/ai-assisted-development.md`](../domain-1-development/ai-assisted-development.md#how-it-works--the-pattern-across-the-sdlc)**
— see the "Domain 3" section of that doc's diagram for how AI-generated tests and AI-flagged
deployment risk fit into a CodePipeline run.

## How it fits together: a full CI/CD pipeline with blue/green deployment

```mermaid
flowchart TD
    DEV[Developer pushes code] --> CC[CodeCommit / Git repo]
    CC -->|trigger| CP[CodePipeline]
    CP --> CB[CodeBuild: install deps, run unit tests, build artifact/container image]
    CB -->|test failure| FAIL[Pipeline stops, developer notified]
    CB -->|pass| ECR[Push image to ECR / package Lambda zip]
    ECR --> VALIDATE[CloudFormation/SAM/CDK: validate + package IaC template]
    VALIDATE --> APPROVE{Manual approval gate}
    APPROVE -->|AI flags risky diff, human reviews| APPROVE
    APPROVE -->|approved| CD[CodeDeploy: Blue/Green deployment]
    CD --> GREEN[Deploy new version to GREEN environment]
    GREEN --> HEALTH{Health checks pass?}
    HEALTH -->|yes| SHIFT[Shift traffic: 10% canary, then 100%]
    HEALTH -->|no| ROLLBACK[Automatic rollback to BLUE]
    SHIFT --> MONITOR[CloudWatch alarms watch error rate]
    MONITOR -->|alarm breached| ROLLBACK
    MONITOR -->|healthy| DONE[GREEN is now production; BLUE kept warm or terminated]
```

Step by step:
1. **A developer pushes code** to the source repo, which triggers the pipeline automatically —
   no manual kickoff needed (Task 3.3's CI/CD automation).
2. **CodeBuild compiles, runs unit/integration tests, and produces a deployable artifact** — either
   a container image or a Lambda deployment package. If tests fail here, the pipeline stops before
   anything touches a live environment — this is where Task 3.2's automated testing actually runs.
3. **The artifact is pushed to ECR** (containers) or packaged as a Lambda zip, and any IaC template
   (CloudFormation/SAM/CDK) describing the infrastructure change is validated — catching a broken
   template before it's ever applied.
4. **A manual approval gate** pauses the pipeline. This is where Task 3.3's "AI-assisted deployment
   approvals" lives — an AI tool can summarize the diff and flag risk, but a human still clicks
   approve. Skipping this gate for production changes is a common exam wrong-answer trap.
5. **CodeDeploy performs a blue/green deployment**: the new version goes live in a parallel
   ("green") environment while the current ("blue") environment keeps serving all traffic.
6. **Health checks run against green** before any real traffic reaches it — if they fail, the
   deployment is aborted and blue is untouched (zero customer impact).
7. **Traffic shifts gradually** (a canary-style ramp: e.g. 10% → monitor → 100%) rather than all at
   once — this limits the blast radius of a problem that health checks didn't catch.
8. **CloudWatch alarms watch the shifted traffic** (error rate, latency) throughout the shift — if
   an alarm breaches its threshold, CodeDeploy automatically rolls back to blue. This automatic,
   metric-driven rollback is exactly what Task 3.3 means by "configure rollback," and it's also the
   bridge into Domain 4 (the same CloudWatch alarms get used for troubleshooting).
9. **Once healthy, green becomes the new production** — blue is either terminated or kept idle
   briefly as an instant rollback target.

## Task 3.1 in one picture: what actually goes into a deployable artifact

```mermaid
flowchart TD
    CODE[Application source code] --> ARTIFACT{Artifact type?}
    ARTIFACT -->|Lambda zip| ZIP[Zip: code + dependencies\nNOT config, NOT secrets]
    ARTIFACT -->|Container image| IMG[Docker image: code + runtime + OS deps\npushed to ECR with a tag]
    ZIP --> LAYER[Shared deps split into a Lambda layer\nso every function's own zip stays small]
    IMG --> LIFECYCLE[ECR lifecycle policy: expire old/untagged images\nscan on push for vulnerabilities]
    CONFIG[Environment-specific config] -.NEVER baked into the artifact.-> EXTERNAL[AppConfig / Parameter Store / Secrets Manager\nread at runtime, per environment]
    ZIP -.resolves build-time dependencies from.-> ARTREPO[CodeArtifact: private npm/PyPI/Maven repo]
    IMG -.resolves build-time dependencies from.-> ARTREPO
    ZIP -.at deploy time, reads.-> EXTERNAL
    IMG -.at deploy time, reads.-> EXTERNAL
```

The exam-relevant discipline here: **the same built artifact should be promotable unchanged from
dev to staging to prod** — if you have to rebuild the zip/image to change which database it points
at, you've baked environment-specific config into the artifact, which is exactly what Task 3.1 is
testing you to avoid. The artifact is immutable; only the externalized config it reads at startup
changes per environment.

## Deployment strategies compared

```mermaid
flowchart LR
    subgraph BG["Blue/Green"]
        B1[100% Blue] -->|instant cutover or gradual shift| B2[100% Green]
        B2 -.instant rollback: flip back.-> B1
    end
    subgraph CAN["Canary"]
        C1[90% old / 10% new] -->|monitor, then| C2[0% old / 100% new]
    end
    subgraph ROLL["Rolling"]
        R1[Instance 1: updated] --> R2[Instance 2: updated] --> R3[Instance 3: updated]
    end
```

| Strategy | Rollback speed | Extra capacity needed | Best for |
|---|---|---|---|
| **Blue/Green** | Instant (flip traffic back to the untouched blue environment) | Full duplicate environment during the cutover | "Zero-downtime, must be able to undo instantly" requirements |
| **Canary** | Fast (stop shifting, route remaining traffic back) | A small amount of extra capacity for the canary slice | "Limit the blast radius to a small % of users first" requirements |
| **Rolling** | Slow (must roll each instance back one at a time, no single flip point) | None beyond normal capacity | Cost-sensitive deployments where some mixed-version window is acceptable |

A classic trap: a scenario needing "instant, full rollback" gets answered with rolling, because
rolling *sounds* safe and gradual — but rolling has no single point where you can flip back
immediately; you're undoing it instance by instance, which takes real time. Blue/green is the only
one of the three with a true instant full-rollback property.

## Full worked case study: CI/CD for a containerized checkout microservice

**Requirement.** A checkout microservice runs on ECS Fargate. The team wants every merge to `main`
to deploy automatically, but a production deployment must never go live without unit tests
passing, an infrastructure-template validation step, a human approval informed by an AI-generated
risk summary of the change, and an automatic rollback if error rate spikes within 5 minutes of a
new version receiving traffic. Config (DB connection string, feature flags) must differ between
staging and production without rebuilding the container image.

**Architecture.**

```mermaid
flowchart TD
    GIT[git push to main] --> CP[CodePipeline]
    CP --> CB[CodeBuild]
    CB -->|npm test / pytest| TESTRESULT{Unit tests pass?}
    TESTRESULT -->|no| STOP[Pipeline stops]
    TESTRESULT -->|yes| BUILDIMG[docker build, tag with commit SHA]
    BUILDIMG --> SCAN[ECR image scan on push]
    SCAN --> CDKDIFF[cdk diff — validates the ECS task definition / service change]
    CDKDIFF --> AIRISK[AI step summarizes the diff's blast radius]
    AIRISK --> APPROVAL{Human approval}
    APPROVAL -->|approved| DEPLOY[CodeDeploy: ECS blue/green via a second target group]
    DEPLOY --> GREEN_TASKS[New task set behind GREEN target group]
    GREEN_TASKS --> ALB_SHIFT[ALB listener shifts traffic to GREEN]
    ALB_SHIFT --> ALARM[CloudWatch alarm: 5xx rate over 5 minutes]
    ALARM -->|breached| AUTOROLLBACK[CodeDeploy auto-rollback to BLUE target group]
    ALARM -->|healthy| PROMOTE[GREEN becomes production; BLUE task set drained]
    DEPLOY -.reads at container startup.-> PARAMSTORE[Parameter Store / Secrets Manager\nper-environment config]
```

**Why each choice:**
- **ECR image tagged with the commit SHA, not `latest`.** This is Task 3.1's artifact-immutability
  discipline in practice — every deployment is traceable to an exact commit, and a rollback can
  reference an exact prior image tag instead of guessing which "latest" was actually good.
- **CodeBuild runs unit tests before the image is even pushed to ECR** — Task 3.2's "automate
  application testing," positioned as early as possible so a failing test never results in a
  built, scannable, deployable artifact at all.
- **`cdk diff` validates the infrastructure change before deployment** — catching, for example, an
  accidental task definition change that would remove a required environment variable, before it
  ever reaches a live environment. This is Task 3.2's infrastructure-template-validation bullet.
- **An AI step summarizes the diff for the human approver**, but the approval itself is still a
  manual CodePipeline approval action — the AI informs, a human decides, exactly the pattern from
  [`../domain-1-development/ai-assisted-development.md`](../domain-1-development/ai-assisted-development.md).
- **ECS blue/green via CodeDeploy, using a second target group behind the ALB.** This is the
  container-specific version of the blue/green pattern Task 3.3 now emphasizes — functionally
  identical in spirit to the Lambda alias-based blue/green shown in the main pipeline diagram
  above, just applied to ECS task sets instead of Lambda versions.
- **A CloudWatch alarm on 5xx rate over a 5-minute window drives automatic rollback** — satisfying
  the "must automatically roll back on error spike" requirement without anyone needing to be
  actively watching a dashboard during the deployment window.
- **Config is read from Parameter Store/Secrets Manager at container startup**, keyed by
  environment — the exact same container image runs in staging and production, with the
  environment-appropriate config injected at runtime rather than baked into two different images.

## Use Cases — Real Scenarios

Concrete developer/DevOps situations where Task 3.1/3.2/3.3 decisions actually get made, with app
type / requirement / implementation decision spelled out:

1. **App type:** Multi-environment SaaS API (dev/staging/prod on the same codebase).
   **Requirement:** the exact same tested build must be promotable across all three environments
   without a rebuild, and a config mistake in one environment must not be able to leak into
   another. **Decision:** build one immutable container image tagged with the commit SHA, and
   read all environment-specific values (DB host, feature flags) from Parameter Store/Secrets
   Manager keyed by environment at container startup — never bake config into the image.
2. **App type:** Payments microservice on ECS Fargate. **Requirement:** a bad deployment must be
   undoable within seconds, even if that means briefly running double capacity.
   **Decision:** blue/green deployment via CodeDeploy using a second ALB target group — the only
   one of the three strategies with a true instant full-rollback flip point, worth the temporary
   duplicate-capacity cost for a payments-critical path.
3. **App type:** High-traffic content API serving millions of daily reads. **Requirement:** limit
   the blast radius of a bad deploy to a small percentage of real users before committing to a
   full rollout, without provisioning a complete duplicate environment. **Decision:** canary
   deployment (e.g. `LambdaCanary10Percent5Minutes`), accepting a smaller resource overhead than
   blue/green in exchange for a built-in automatic "pause and evaluate" step.
4. **App type:** Internal cost-sensitive batch-processing service. **Requirement:** deploy safely
   but without justifying the expense of a duplicate environment for every release.
   **Decision:** rolling deployment — slower to roll back (instance by instance, no single flip
   point) but no extra capacity cost, an acceptable tradeoff for a non-customer-facing workload
   that can tolerate a brief mixed-version window.
5. **App type:** Regulated fintech deployment pipeline. **Requirement:** no infrastructure change
   can reach production without both an automated template-validation check and a documented human
   sign-off, even when an AI tool says the diff is low-risk. **Decision:** `cdk diff`/`sam validate`
   runs in the CodeBuild stage as a hard gate, and a CodePipeline manual approval action (informed
   by, but not replaced by, an AI risk summary) gates the deploy stage.
6. **App type:** Team migrating a monolith's first feature to serverless. **Requirement:** write
   automated tests for a new Lambda handler without needing a live API Gateway, DynamoDB table, or
   deployed stack just to exercise the handler's logic. **Decision:** hand-crafted/console-template
   test events plus `sam local invoke`, so the handler's parsing/response-shaping logic is verified
   entirely locally before anything is deployed.
7. **App type:** Container-based checkout service (see the full case study above).
   **Requirement:** an automatic rollback must trigger on a genuine production error spike, not
   just when a deployment script itself fails to run. **Decision:** associate a CloudWatch alarm
   (5xx rate over a 5-minute window) with the CodeDeploy deployment group specifically — the
   deployment-failure trigger alone wouldn't catch "deployed fine, now erroring in production."
8. **App type:** Platform team standardizing Lambda packaging across 40 functions.
   **Requirement:** stop every team from committing different versions of the same third-party
   SDK, bloating deployment packages and causing version drift. **Decision:** a shared Lambda
   layer for the common dependency, versioned centrally, so updating it once propagates to every
   function on their next deploy rather than 40 separate dependency bumps.
9. **App type:** Image-processing pipeline moving from Lambda zip packaging to containers.
   **Requirement:** old, untagged, or vulnerability-flagged container images must be purged
   automatically instead of accumulating in ECR indefinitely. **Decision:** an ECR lifecycle
   policy expiring untagged images after N days, combined with scan-on-push, rather than a
   manual cleanup script someone has to remember to run.

## Task 3.2 in practice: validating before you ever deploy

Three concrete validation techniques the exam expects you to recognize by behavior, not just name:

**Template validation** catches syntax/schema errors before any API call to create/update a stack:
```bash
sam validate --template template.yaml        # SAM-specific linting
aws cloudformation validate-template --template-body file://template.yaml  # generic CFN syntax check
cdk diff                                      # shows exactly what would change, before deploying
```
`cdk diff` is the one worth understanding conceptually even if you never type the others: it
compares your synthesized CloudFormation template against what's currently deployed and shows a
human-readable diff — the modern equivalent of "review the plan before you apply it," borrowed
from the same idea Terraform popularized, applied inside the AWS-native IaC toolchain.

**Mock APIs and test events** let you exercise a Lambda function's logic without any live
dependency:
```bash
# A hand-crafted test event simulating an API Gateway proxy request, with no real API Gateway involved
cat > test-event.json <<'EOF'
{"httpMethod": "POST", "path": "/orders", "body": "{\"item\":\"widget\"}", "headers": {}}
EOF
sam local invoke CreateOrderFunction --event test-event.json
```
The console's Lambda test feature ships built-in templates (S3 Put, SNS notification, DynamoDB
Stream, etc.) for exactly this reason — you don't need a real S3 bucket wired up to validate that
your handler parses an S3 event correctly.

**AI-assisted test generation** (the Domain 1 cross-link) proposes unit tests from existing code —
useful for quickly covering the obvious cases, but still requires a human to add the
business-specific edge cases the AI has no way to infer; see
[`../domain-1-development/ai-assisted-development.md`](../domain-1-development/ai-assisted-development.md#feature-by-feature-what-each-ai-capability-maps-to-on-the-exam)
for the full breakdown of this capability.

## SAM vs. CDK vs. raw CloudFormation, briefly

Task 3.2/3.3 assumes you can tell these apart without memorizing syntax for all three:

| | Raw CloudFormation | AWS SAM | AWS CDK |
|---|---|---|---|
| Authoring format | YAML/JSON template directly | YAML/JSON with serverless-specific shorthand (`AWS::Serverless::Function`) | Real programming language (TypeScript, Python, etc.) that synthesizes to CloudFormation |
| Best for | Full control, no abstraction | Serverless-first apps (Lambda, API Gateway, DynamoDB) wanting less boilerplate | Complex infra needing loops/conditionals/reusable constructs beyond what a template DSL comfortably expresses |
| Local testing | None built-in | `sam local invoke`, `sam local start-api` | Relies on the same `cdk synth`/`cdk diff`, no Lambda-specific local runner built in |
| Validation command | `validate-template` | `sam validate` | `cdk diff` / `cdk synth` |

A scenario describing "needs to loop over a list of regions and generate a repeated resource block
per region with conditional logic" is pointing at CDK (real code, real control flow) over SAM or
raw CloudFormation (template DSLs that make this awkward). A scenario about "fast local testing of
a serverless app's Lambda functions and API Gateway locally before any deploy" is SAM specifically,
because of `sam local`.

## Common exam scenarios



- *"A deployment must have near-zero downtime and an instant, full rollback option if anything goes
  wrong"* → blue/green (full environment swap), not rolling (rolling has no single "flip back"
  point — it's gradually replacing instances).
- *"Only a small percentage of users should see the new version first, to limit risk before a full
  rollout"* → canary deployment.
- *"A monitoring alarm should automatically trigger a rollback during a deployment, not just alert
  a human"* → configure CodeDeploy rollback to trigger on CloudWatch alarm state, not just on
  deployment failure.
- *"A Lambda function's test events need to simulate an S3 upload without actually uploading a
  file"* → use a mock/test event (Task 3.2), e.g. the console's built-in S3 Put test event template.
- *"Infrastructure changes must be validated for syntax/drift before they're ever applied"* → run
  `cfn-lint`/`sam validate`/`cdk diff` in the CodeBuild stage, before the deploy stage.
- *"Container images must be scanned and only tagged images older than N days should be purged"* →
  ECR lifecycle policies + image scanning, not manual cleanup.
- *"An AI tool suggests a deployment is safe, but the team still wants a person to confirm before
  production changes go live"* → manual approval action in CodePipeline, informed but not replaced
  by the AI flag.
- *"A new container-based microservice must scale automatically without managing EC2 instances"* →
  ECS/EKS on Fargate, not self-managed EC2-backed ECS/EKS.
- *"The same build artifact must be deployable unchanged to dev, staging, and production"* →
  externalize config via Parameter Store/Secrets Manager/AppConfig, read at runtime — never bake
  environment-specific values into the artifact itself.
- *"A feature flag/config change must roll out gradually to a percentage of traffic and
  automatically roll back if a CloudWatch alarm trips"* → AWS AppConfig's deployment strategy +
  alarm integration, not a plain Parameter Store value flipped instantly for 100% of traffic — see
  [`services/management-governance/appconfig.md`](../../../services/management-governance/appconfig.md).
- *"A build keeps failing because a public npm/PyPI package was yanked or a public registry had an
  outage"* → the team isn't using a private, cached dependency repository — CodeArtifact resolves
  and permanently retains previously-fetched package versions so a public-registry outage or
  removal doesn't break the build; see
  [`services/developer-tools/codeartifact.md`](../../../services/developer-tools/codeartifact.md).
- *"An attacker published a malicious package on the public registry using the same name as an
  internal, never-publicly-published package"* → that's a dependency confusion attack — CodeArtifact
  package origin controls (block upstream fetches for internally-namespaced packages) are the fix,
  not a naming convention alone.
- *"A rollback must happen within seconds, even at the cost of running double capacity briefly"* →
  blue/green, accepting the temporary cost of a duplicate environment for the fastest possible undo.
- *"A cost-sensitive team wants gradual deployment but can't justify running a duplicate full
  environment"* → rolling deployment — cheaper, slower to roll back, acceptable when a short mixed-
  version window during the rollout is tolerable.
- *"A CodeBuild stage needs to pull a private dependency and also run as part of a pipeline with
  least-privilege access"* → scope the CodeBuild service role narrowly (read access to just the
  needed artifact/source locations), the same least-privilege discipline from Domain 2 applied to
  a pipeline's own execution identity.
- *"A pipeline deploying a Lambda function needs to warm up the new version's concurrency before
  it receives real traffic during a canary shift"* → combine CodeDeploy's Lambda traffic-shifting
  configs (e.g. `LambdaCanary10Percent5Minutes`) with provisioned concurrency on the new version,
  so the canary slice isn't hit with cold starts during the very window you're trying to validate.
- *"A deployment needs a pre-traffic and post-traffic validation hook before/after the shift"* →
  CodeDeploy AppSpec `BeforeAllowTraffic`/`AfterAllowTraffic` lifecycle hooks, which can run a
  Lambda function to run smoke tests before any real traffic reaches the new version.
- *"ECS blue/green deployments need to be managed declaratively alongside the rest of the
  infrastructure"* → CodeDeploy-via-CloudFormation support for ECS blue/green, so the deployment
  strategy itself is version-controlled IaC, not a manually-clicked console action.

## Domain 3 study checklist (core)

- [ ] Explain the difference between blue/green, canary, and rolling deployments, including which
      one has a true instant-rollback property.
- [ ] Explain why config must be externalized rather than baked into a build artifact.
- [ ] Name at least two things Task 3.2 expects you to validate *before* deployment (template
      syntax, mock test events) and why each matters.
- [ ] Describe where in a CI/CD pipeline an AI-assisted test-generation or deployment-approval step
      would realistically sit, and what a human still must do at that step.
- [ ] Explain ECR lifecycle policies and image scanning, and why "just delete old images manually"
      is the wrong exam answer.
- [ ] Pick the right CodeDeploy compute platform (EC2/on-prem, Lambda, ECS) for a given scenario.

## Rollback mechanics: how "automatic" rollback actually triggers

A frequently-misunderstood detail: CodeDeploy rollback doesn't happen just because "something went
wrong" in the abstract — it's wired to one of two concrete triggers, and the exam tests whether you
know which trigger a given scenario is describing.

```mermaid
flowchart TD
    DEPLOY[Deployment in progress] --> T1{Deployment lifecycle event fails?\ne.g. BeforeInstall hook script errors}
    DEPLOY --> T2{CloudWatch alarm associated\nwith the deployment group breaches?}
    T1 -->|yes| RB1[Rollback: deployment-failure trigger]
    T2 -->|yes| RB2[Rollback: alarm-based trigger]
    T1 -->|no| CONTINUE[Deployment continues]
    T2 -->|no breach| CONTINUE
```

- **Deployment-failure rollback** fires when a lifecycle hook itself errors (a script in the
  AppSpec's `BeforeInstall`/`AfterInstall`/etc. returns non-zero) — this catches "the deployment
  process itself broke," not "the new version is live but behaving badly."
- **Alarm-based rollback** fires when a CloudWatch alarm *you explicitly associate with the
  deployment group* enters `ALARM` state during/after the shift — this is the one that catches
  "the new version deployed successfully but is producing errors/high latency in production,"
  which is the scenario most exam questions about "automatic rollback" are actually describing.

Both must be explicitly configured — neither is on by default, and a scenario describing "the
rollback didn't happen even though the new version was clearly broken" is almost always testing
whether you know an alarm wasn't associated with the deployment group at all.

## Plain-English analogies for the whole domain

Task statements are abstract until you can picture them. These four analogies map directly onto
Task 3.1/3.2/3.3 and get referenced throughout the rest of this document — read them once up
front.

- **The pipeline is a restaurant kitchen, not a vending machine.** A vending machine just dispenses
  whatever's loaded into it (manual, un-reviewed deployment). A kitchen has stations (source →
  build → test → deploy), a head chef who tastes before anything leaves the pass (the test stage),
  and an expediter who holds a dish back if it isn't right (a manual approval gate or a failed
  CloudWatch alarm). CodePipeline is the kitchen's workflow; CodeBuild is the station that actually
  cooks; CodeDeploy is the server who carries the plate out — and can take it back to the kitchen
  (rollback) if a customer sends it back.
- **Blue/green is a relay race with two full tracks.** Both the old runner (blue) and the new
  runner (green) are already standing at the starting line, fully warmed up, before the race ever
  starts. The baton (traffic) is handed to green only when green proves ready; if green trips, the
  baton instantly goes back to blue, who never stopped being ready. This is why blue/green's
  rollback is "instant" — the old runner was never put away.
- **Canary deployments are a canary in a coal mine, literally.** Miners sent a canary ahead of
  themselves into unknown tunnel air — if the canary was fine, the air was breathable, if it
  dropped, the miners knew to turn back before anyone was in danger. A canary deployment sends 10%
  of real traffic ahead of the other 90% for the same reason: if the small slice is unhealthy, you
  find out before the other 90% of your "miners" (users) ever breathe the bad air.
- **Externalized config is a hotel room key card, not a key cut into the door lock.** Rebuilding a
  Lambda zip or Docker image for every environment is like re-carving a new lock into the door
  every time a new guest checks in. A key card system (Parameter Store/Secrets Manager/AppConfig)
  issues a different card (config value) to the same unchanged lock (the immutable artifact) — the
  door itself is never rebuilt, only the card presented to it changes.

## Zooming out: where Domain 3 sits in the full SDLC

```mermaid
flowchart LR
    D1["Domain 1: Development\n(write the code)"] --> D3prep["Task 3.1\nPrepare artifacts"]
    D3prep --> D3test["Task 3.2\nAutomate testing"]
    D3test --> D3deploy["Task 3.3\nDeploy via CI/CD"]
    D3deploy --> D4["Domain 4: Troubleshooting\n& Optimization\n(watch it run, fix it, tune it)"]
    D2["Domain 2: Security\n(IAM, encryption, secrets)"] -.->|"governs every stage"| D3prep
    D2 -.-> D3test
    D2 -.-> D3deploy
```

Walkthrough: Domain 3 is the connective tissue between "code exists" (Domain 1) and "code is live
and being watched" (Domain 4) — and Domain 2's security controls (least-privilege IAM for the
pipeline's own identity, encrypted secrets, scoped roles) apply to every single stage of Domain 3,
not as a separate concern bolted on afterward. A pipeline that builds, tests, and deploys
flawlessly but runs under an over-permissioned IAM role has failed Domain 2 even while succeeding
at Domain 3 — the exam tests both simultaneously in scenario questions far more often than it
tests either domain in isolation.

## CodeBuild's own "how it works" diagrams, in the CI/CD context

![Diagram showing how CodeBuild works: a build project with source, build environment, buildspec, S3 output, and CloudWatch Logs](https://docs.aws.amazon.com/codebuild/latest/userguide/images/arch.png)

*Source: [AWS CodeBuild User Guide — "AWS CodeBuild concepts"](https://docs.aws.amazon.com/codebuild/latest/userguide/concepts.html), © Amazon Web Services.*

![Diagram showing how CodeBuild integrates as a build or test action inside an AWS CodePipeline stage](https://docs.aws.amazon.com/codebuild/latest/userguide/images/pipeline.png)

*Source: [AWS CodeBuild User Guide — "What is AWS CodeBuild?"](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html), © Amazon Web Services.*

The first diagram is CodeBuild viewed in isolation: a project reads a buildspec, builds in an
ephemeral container, and uploads an artifact while streaming logs to CloudWatch. The second
diagram is the same CodeBuild project viewed **from CodePipeline's perspective** — just one action
inside one stage of a larger, multi-stage pipeline. The exam-relevant point these two images make
together: CodeBuild never orchestrates *itself* into a sequence with other stages — that's always
CodePipeline's job. A standalone CodeBuild project with no CodePipeline in front of it is a valid,
supported thing (you can trigger it from a webhook or the console directly), but Task 3.3's
"deploy code via CI/CD" almost always implies the second picture, not the first.

## Full multi-environment promotion pipeline, step by step

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Src as Source (GitHub/CodeCommit)
    participant CP as CodePipeline
    participant CB as CodeBuild
    participant ECRsvc as ECR
    participant Stg as Staging (ECS Fargate)
    participant Appr as Manual Approval
    participant Prod as Production (ECS Fargate)
    participant CW as CloudWatch Alarms

    Dev->>Src: git push main
    Src->>CP: Triggers pipeline execution
    CP->>CB: Source stage artifact -> Build stage
    CB->>CB: install / pre_build / build / post_build
    CB->>ECRsvc: docker push <commit-sha>
    CB-->>CP: imagedefinitions.json artifact
    CP->>Stg: Deploy stage: update ECS service (staging)
    Stg-->>CP: Smoke tests pass
    CP->>Appr: Manual approval action (AI risk summary attached)
    Appr-->>CP: Approved by human reviewer
    CP->>Prod: Deploy stage: CodeDeploy blue/green (production)
    Prod->>CW: Alarm watches 5xx rate during traffic shift
    CW-->>Prod: Healthy - complete shift to 100%
    CW-->>Prod: Unhealthy - automatic rollback to BLUE
```

Walkthrough: notice that **the exact same commit-SHA-tagged image** flows from the build stage all
the way to production — staging and production never get separately rebuilt artifacts, satisfying
Task 3.1's artifact-immutability discipline end to end. The manual approval sits **between**
staging (where it already passed automated smoke tests) and production (where a human now signs
off on the diff) — this ordering matters: approving before staging validation would mean approving
blind, which defeats the purpose of the gate.

## Worked example: a complete SAM-based pipeline — buildspec.yml + template.yaml + appspec equivalent together

A common exam-adjacent task is recognizing how three different artifact types cooperate in one
release. Here is a genuinely complete, consistent trio for a Lambda-backed orders API.

**`template.yaml`** (SAM — defines the function, its API route, and its canary deployment
preference in one place):
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Globals:
  Function:
    Runtime: python3.12
    Timeout: 10
    AutoPublishAlias: live
Resources:
  OrdersApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: prod
  GetOrderFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/get_order/
      Handler: app.handler
      Events:
        GetOrder:
          Type: Api
          Properties:
            RestApiId: !Ref OrdersApi
            Path: /orders/{id}
            Method: get
      DeploymentPreference:
        Type: Canary10Percent5Minutes
        Alarms:
          - !Ref OrdersErrorAlarm
  OrdersErrorAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      MetricName: Errors
      Namespace: AWS/Lambda
      Statistic: Sum
      Period: 60
      EvaluationPeriods: 1
      Threshold: 1
      ComparisonOperator: GreaterThanOrEqualToThreshold
      Dimensions:
        - Name: FunctionName
          Value: !Ref GetOrderFunction
```

**`buildspec.yml`** (CodeBuild — validates and packages the SAM template as a pipeline stage):
```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      python: 3.12
    commands:
      - pip install aws-sam-cli
  build:
    commands:
      - sam validate --template template.yaml
      - sam build
      - sam package --s3-bucket $ARTIFACT_BUCKET --output-template-file packaged.yaml
artifacts:
  files:
    - packaged.yaml
```

What matters here: **`sam validate` runs inside the build stage, before `sam package` ever
touches a real deploy** — this is Task 3.2's "validate infrastructure templates" bullet happening
literally in the pipeline, not as a separate manual step someone remembers to run. The packaged
template (not the raw `template.yaml`) is what CodePipeline's downstream CloudFormation deploy
action actually applies.

**No separate raw AppSpec file is needed here** — this is a detail worth understanding precisely:
because `DeploymentPreference` is declared directly inside `template.yaml`, SAM's transform
generates the equivalent `AWS::Lambda::Alias` and `AWS::CodeDeploy::DeploymentGroup` resources
(and the AppSpec-equivalent configuration) automatically at deploy time — you only hand-write a
standalone AppSpec YAML file when you're calling CodeDeploy's raw API directly (as in the EC2/ECS
worked examples elsewhere in this doc and in `labs.md`), not when going through SAM's
`DeploymentPreference` shorthand. A question contrasting "I see an AppSpec file in this pipeline"
versus "I see a `DeploymentPreference` block in a SAM template" is testing whether you recognize
these as two different authoring paths to the *same* underlying CodeDeploy mechanism.

## Container CI/CD deep dive: ECS blue/green with a hand-written AppSpec

For a *non*-SAM, raw-CodeDeploy container pipeline (common when CloudFormation/CDK define the ECS
service directly rather than through SAM), the AppSpec is a separate, explicit file:

```yaml
# appspec.yaml — ECS blue/green via CodeDeploy
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "<TASK_DEFINITION>"   # CodeDeploy substitutes this at deploy time
        LoadBalancerInfo:
          ContainerName: "orders-api"
          ContainerPort: 8080
Hooks:
  - BeforeInstall: "arn:aws:lambda:us-east-1:123456789012:function:pre-install-checks"
  - AfterAllowTestTraffic: "arn:aws:lambda:us-east-1:123456789012:function:run-smoke-tests"
  - AfterAllowTraffic: "arn:aws:lambda:us-east-1:123456789012:function:post-traffic-checks"
```

```yaml
# buildspec.yml — builds the image, registers a new task definition revision,
# and produces the two files CodePipeline's ECS-blue/green deploy action needs
version: 0.2
phases:
  pre_build:
    commands:
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
  build:
    commands:
      - docker build -t orders-api:$COMMIT_HASH .
      - docker tag orders-api:$COMMIT_HASH $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/orders-api:$COMMIT_HASH
  post_build:
    commands:
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/orders-api:$COMMIT_HASH
      - printf '{"ImageURI":"%s"}' $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/orders-api:$COMMIT_HASH > imageDetail.json
artifacts:
  files:
    - imageDetail.json
    - appspec.yaml
    - taskdef.json
```

```json
// taskdef.json — the task definition template CodePipeline's ECS (blue/green) deploy action
// fills in the IMAGE1_NAME placeholder for, using imageDetail.json's ImageURI
{
  "family": "orders-api",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "orders-api",
      "image": "<IMAGE1_NAME>",
      "portMappings": [{"containerPort": 8080, "protocol": "tcp"}]
    }
  ]
}
```

Three files travel together as the build artifact — `appspec.yaml`, `taskdef.json`, and
`imageDetail.json` — and CodePipeline's **ECS (blue/green)** deploy action (distinct from its
plain **ECS** deploy action, which does a rolling update with no CodeDeploy involvement at all)
consumes all three together. This exact three-file bundle, by name, is a specific, testable detail:
a question describing "the ECS blue/green deploy action in CodePipeline isn't finding the image
reference" is almost always pointing at a missing or malformed `imageDetail.json`.

## Testing pyramid for serverless and container apps

```mermaid
flowchart TB
    subgraph Pyramid["Testing pyramid - bottom is cheapest and fastest"]
        direction TB
        Unit["Unit tests\nmany, fast, no AWS calls\npytest/jest against pure functions"]
        Integration["Integration tests\nfewer, slower\nreal or emulated AWS services (moto/Floci/LocalStack)"]
        E2E["End-to-end / load tests\nfewest, slowest, most expensive\nagainst a real deployed staging environment"]
    end
    Unit --> Integration --> E2E
```

Walkthrough: this pyramid shape is the single most important testing-strategy fact for Task 3.2 —
you want **many** fast, isolated unit tests, **some** integration tests against real or emulated
AWS services, and only a **small number** of expensive end-to-end/load tests against a real
environment. A question describing a test suite that's mostly end-to-end tests against a live
staging stack, running for 40 minutes per pipeline execution, is describing an **inverted pyramid**
— a specific anti-pattern the exam expects you to recognize and recommend fixing by pushing more
coverage down into unit/integration tests. The full depth of this topic (including worked
pytest/moto examples, load-testing with Locust, and exact tool selection per layer) has its own
dedicated doc:

➡️ **[`testing-strategies-for-serverless-apps.md`](testing-strategies-for-serverless-apps.md)**

## IaC drift: the validation story doesn't end at deploy time

Task 3.2's "validate infrastructure templates" is usually read as a pre-deploy check
(`sam validate`, `cdk diff`) — but a template that was valid at deploy time can still **drift**
away from what's actually running, if someone makes an out-of-band change directly in the console.

![The Detect drift for current stack command selected on the Stack actions menu for the selected stack.](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/console-stacks-actions-detect-drift-1.png)

*Source: [AWS CloudFormation User Guide — "Detect drift on an entire CloudFormation stack"](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/detect-drift-stack.html), © Amazon Web Services.*

A stack that shows `DRIFTED` means at least one resource's live configuration no longer matches
what the template says it should be — a classic cause is "someone manually changed a security
group rule in the console to fix an urgent issue, and never updated the template to match." The
full drift-detection and remediation workflow (detection cadence, automated multi-account
detection via AWS Config, and the remediation decision tree) is covered in depth here:

➡️ **[`iac-drift-detection-and-remediation.md`](iac-drift-detection-and-remediation.md)**

## Deployment strategy choice, with real CodeDeploy configuration names

The "Deployment strategies compared" table above gives the conceptual tradeoffs; the exam also
expects you to recognize CodeDeploy's actual **named deployment configurations** by pattern:

| Named config | Compute platform | Shape |
|---|---|---|
| `CodeDeployDefault.LambdaAllAtOnce` | Lambda | 100% traffic shifted immediately, no gradual ramp |
| `CodeDeployDefault.LambdaCanary10Percent5Minutes` | Lambda | 10% shifted, wait 5 min, then 100% |
| `CodeDeployDefault.LambdaLinear10PercentEvery1Minute` | Lambda | +10% every minute until 100% |
| `CodeDeployDefault.ECSAllAtOnce` | ECS | 100% traffic shifted to the replacement task set immediately |
| `CodeDeployDefault.ECSCanary10Percent5Minutes` | ECS | 10% shifted, wait 5 min, then 100% |
| `CodeDeployDefault.ECSLinear10PercentEvery1Minute` | ECS | +10% every minute until 100% |
| `CodeDeployDefault.OneAtATime` | EC2/on-premises | One instance at a time (rolling) |
| `CodeDeployDefault.HalfAtATime` | EC2/on-premises | Half the fleet, then the other half |
| `CodeDeployDefault.AllAtOnce` | EC2/on-premises | Every instance simultaneously — fastest, least safe |

A full, dedicated comparison of blue/green vs. canary vs. linear — including the complete worked
AppSpec/CodeDeploy YAML for all three against both Lambda and ECS, the exact bake-time tradeoffs,
and a decision flowchart — lives in its own topic doc:

➡️ **[`blue-green-canary-linear-deployment-strategies.md`](blue-green-canary-linear-deployment-strategies.md)**

## The pipeline's own identity: IAM and STS in CI/CD

Every stage of a pipeline runs **as some IAM identity**, and Task 3.3 expects you to reason about
least privilege for the pipeline itself, not just for the application it deploys.

```mermaid
flowchart TD
    CPRole["CodePipeline service role"] -->|assumes, scoped to this pipeline only| CBRole["CodeBuild project role"]
    CBRole -->|"ecr:GetAuthorizationToken,\necr:PutImage (scoped to ONE repo)"| ECRsvc[ECR]
    CBRole -->|"ssm:GetParameter,\nsecretsmanager:GetSecretValue\n(scoped to specific ARNs)"| Config[Parameter Store / Secrets Manager]
    CPRole -->|"sts:AssumeRole into a\ncross-account deploy role"| ProdRole["Cross-account production\ndeploy role (separate AWS account)"]
    ProdRole -->|"cloudformation:*,\nscoped to this stack's resources"| Prod[Production stack]
```

Walkthrough: a pipeline deploying into a **separate production AWS account** (a common
multi-account pattern — see `glossary`/Organizations content) does so via **STS `AssumeRole`**
into a role that exists *in* the production account, trusting only the pipeline's own account as
principal. This is exactly the `sts-and-role-switching` pattern applied to CI/CD: the pipeline's
own service role never holds standing production credentials — it assumes a temporary, scoped role
only for the duration of the deploy action. A scenario describing "the pipeline account should
never have persistent IAM users with direct production access" is testing this exact cross-account
AssumeRole pattern, not a different cross-account sharing mechanism.

## Secrets vs. config vs. feature flags in the pipeline — the three-way distinction, concretely

| Need | Service | Why not the other two |
|---|---|---|
| A database password that must rotate automatically | Secrets Manager | Parameter Store has no built-in rotation engine; AppConfig has no concept of a rotating credential at all |
| 2,000 non-secret environment settings at effectively zero cost | Parameter Store (Standard) | Secrets Manager's per-secret fee at that volume is a real, avoidable cost; AppConfig is overkill for values with no gradual-rollout requirement |
| A feature flag that must roll out to 10% of users with automatic rollback on an alarm | AppConfig | Parameter Store/Secrets Manager have no deployment strategy or rollback concept at all — flipping a plain parameter is instant and all-or-nothing |

This table is the fast, testable version of the full comparison in
[`services/security-identity-compliance/secrets-manager-vs-parameter-store.md`](../../../services/security-identity-compliance/secrets-manager-vs-parameter-store.md)
and [`services/management-governance/appconfig.md`](../../../services/management-governance/appconfig.md)
— a scenario naming "gradual rollout" or "automatic rollback on alarm" for a *configuration* change
(not a deployment) is the single clearest AppConfig signal, distinguishing it from both of its
"just store a value" siblings.

## Additional worked exam scenarios

- *"A pipeline builds a container image, but the ECS blue/green deploy action in CodePipeline
  can't determine which image to deploy."* → the build stage's artifacts are missing
  `imageDetail.json` (or it's malformed) — this is the specific file the ECS blue/green deploy
  action consumes, distinct from `imagedefinitions.json` used by the plain (non-blue/green) ECS
  deploy action.
- *"A canary Lambda deployment should run an automated smoke test against the new version before
  any of the remaining 90% of traffic shifts."* → an `AfterAllowTestTraffic`/`PostTraffic` AppSpec
  hook (or SAM's `DeploymentPreference.Hooks.PostTraffic`) running a validation Lambda that calls
  `PutLifecycleEventHookExecutionStatus` — not a separate, disconnected test suite run after the
  deployment is already considered complete.
- *"A development team wants to validate a CloudFormation template's syntax and see exactly what
  would change before applying an update, without actually deploying anything."* → `cdk diff` (CDK
  apps) or a CloudFormation **change set** (raw CloudFormation/SAM) — both show the pending diff
  without mutating any live resource.
- *"A security team discovered a resource was manually modified in the console after its stack was
  deployed, and wants to detect this automatically going forward, across hundreds of accounts."* →
  CloudFormation **drift detection**, automated via AWS Config rules and EventBridge across
  accounts — not a one-off manual console check; see the drift-detection topic doc.
- *"A CI/CD pipeline must deploy into a separate AWS account than the one the pipeline itself runs
  in, without the pipeline account holding any standing production credentials."* → cross-account
  **STS `AssumeRole`** into a role defined in the target account, trusting only the pipeline
  account as principal.
- *"A test suite takes 40 minutes per pipeline run because most of the coverage is full
  end-to-end tests against a live staging environment."* → the test suite has an **inverted testing
  pyramid** — the fix is pushing more coverage down into fast unit tests and a moderate layer of
  integration tests, reserving end-to-end/load tests for a small number of critical paths.
- *"A load test must simulate realistic, bursty user traffic against a staging API before a major
  release, without hand-rolling a custom load generator."* → a dedicated load-testing tool (e.g.,
  Locust, Artillery, or AWS's own Distributed Load Testing solution) run against a staging
  environment, not folded into the same CodeBuild unit-test phase that runs on every commit — load
  tests are expensive and slow, and belong at the top of the testing pyramid, run less frequently.
- *"A rotating database credential must be referenced by an ECS task definition without writing
  any code to fetch it."* → reference the Secrets Manager ARN directly in the task definition's
  `secrets` block — both ECS and Lambda support native secret/parameter references with zero SDK
  code needed in the application itself.
- *"A Lambda canary deployment needs to warm up provisioned concurrency on the new version before
  the canary traffic shift begins, so the first real users don't experience cold starts during the
  validation window."* → configure **provisioned concurrency** on the new published version ahead
  of the CodeDeploy canary shift — provisioned concurrency and traffic-shifting are independent
  features that must be deliberately combined, not something CodeDeploy does automatically.
- *"A CDK app needs to be checked for unintended infrastructure changes before every deployment, as
  a hard pipeline gate, not just a developer's manual habit."* → `cdk diff` run as an explicit
  CodeBuild phase that fails the build (or requires approval) on any change touching
  security-sensitive resource types — making template validation part of the pipeline's enforced
  behavior rather than a convention someone might skip under deadline pressure.

## Domain 3 study checklist (expanded)

- [ ] Explain the difference between blue/green, canary, and rolling deployments, including which
      one has a true instant-rollback property.
- [ ] Name the actual CodeDeploy deployment-config strings (`LambdaCanary10Percent5Minutes`,
      `ECSLinear10PercentEvery1Minute`, `OneAtATime`, etc.) and match them to compute platform.
- [ ] Explain why config must be externalized rather than baked into a build artifact, and when
      that externalized config should be Parameter Store vs. Secrets Manager vs. AppConfig.
- [ ] Name at least two things Task 3.2 expects you to validate *before* deployment (template
      syntax, mock test events) and why each matters — plus what drift detection catches *after*.
- [ ] Describe where in a CI/CD pipeline an AI-assisted test-generation or deployment-approval step
      would realistically sit, and what a human still must do at that step.
- [ ] Explain ECR lifecycle policies and image scanning, and why "just delete old images manually"
      is the wrong exam answer.
- [ ] Pick the right CodeDeploy compute platform (EC2/on-prem, Lambda, ECS) for a given scenario.
- [ ] Describe the shape of the testing pyramid and recognize an inverted-pyramid anti-pattern.
- [ ] Explain cross-account pipeline deployment via STS AssumeRole, and why a pipeline shouldn't
      hold standing credentials in the account it deploys into.
- [ ] Identify the three artifacts an ECS blue/green CodePipeline deploy action needs
      (`appspec.yaml`, `taskdef.json`, `imageDetail.json`) versus the single file
      (`imagedefinitions.json`) the plain ECS deploy action needs.

## Labs

See [`labs.md`](labs.md) for hands-on Floci exercises covering this domain.
