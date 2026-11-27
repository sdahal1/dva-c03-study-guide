# Task 3.2 practice questions

## Q1

**Skills:** 3.2.1

Before a CloudFormation/SAM template is applied to any real AWS account, the team wants to catch
syntax errors and obvious misconfigurations automatically as part of the CI pipeline, without
spending time or cost provisioning real resources to find out something is broken. What should run
in the CodeBuild stage before the deploy stage, per Task 3.2?

A. Nothing. CloudFormation only reports errors after attempting to create resources, so pre-deploy validation isn't possible.

B. Run template validation/linting tools (e.g. `sam validate`, `cfn-lint`, or `cdk diff` for CDK) against the template as a build step, failing the pipeline early if the template is malformed.

C. Deploy the template to production directly and roll back immediately if it fails.

D. Manually read through the YAML/JSON template line by line before every deployment.

**Correct Answer: B**

**Explanation:** Task 3.2 expects automated IaC (infrastructure as code) validation before
deployment. Running `sam validate`/`cfn-lint`/`cdk diff` style checks as a CodeBuild step catches
syntax errors and obvious misconfigurations before anything touches a live environment, which is
cheaper and faster than discovering problems mid-deploy. See
[3.2.1 Testing with SAM, CDK, and CloudFormation](3.2.1-testing-with-sam-cdk-and-cloudformation.md#cloudformation-validate-templates-before-and-after-deployment).

**Why the others are wrong:**
- A: CloudFormation, SAM, and CDK all provide pre-apply validation tooling specifically to catch errors before real resources are touched, and CloudFormation itself now runs pre-deployment validation on create, update, and change set operations. This option describes an old, manual workflow, not current practice.
- C: Deploying straight to production and relying on rollback-after-failure is the opposite of validating early. It risks real customer impact for an error that validation tooling could have caught for free before deploy.
- D: Manual line-by-line review doesn't scale, is error-prone, and is exactly the kind of repetitive check automated linting/validation tools exist to replace.

---

## Q2

**Skills:** 3.2.3

A Lambda function processes S3 upload events, but the team wants to run it through its unit/
integration test suite in CI without actually uploading a real file to a real S3 bucket for every
test run. What is the correct Task 3.2 approach?

A. Provision a dedicated S3 bucket solely for CI and perform a real upload before every single test run.

B. Use a mock/test event that simulates the shape of an S3 `PutObject` event notification (e.g. the console's built-in S3 Put test event template) and invoke the function's handler directly with that payload.

C. Skip testing the S3-triggered path entirely since it's "too hard" to test without live infrastructure.

D. Only test this function manually in the production environment after every deployment.

**Correct Answer: B**

**Explanation:** Task 3.2 explicitly calls for test events that exercise Lambda functions without
live dependencies. Using a template S3 event payload (from the console template or
`sam local generate-event s3 put`) lets the handler's actual processing logic be tested directly,
in isolation, without provisioning or touching real S3 infrastructure. See
[3.2.3 Test events and payloads](3.2.3-test-events-and-payloads.md#generate-and-use-events-with-aws-sam).

**Why the others are wrong:**
- A: Provisioning real infrastructure and performing real uploads for every CI run is slower, costs more, and introduces external dependencies and flakiness into what should be a fast, isolated unit test.
- C: Skipping test coverage for a significant code path (the actual trigger logic) leaves a real correctness gap. Task 3.2 exists specifically to make this kind of path testable, not to excuse skipping it.
- D: Testing only in production after deployment means defects are caught after they're already live and potentially affecting real users, the opposite of "test before deploy."

---

## Q3

**Skills:** 3.2.5

A team wants to speed up writing regression tests for a newly refactored payment-processing Lambda
function. A developer uses an AI coding assistant to generate a first batch of unit tests covering
the function's main code paths and several edge cases the developer hadn't initially considered.
Per Task 3.2's AI-assisted testing guidance, what should happen next?

A. Merge the AI-generated tests immediately with no review, since generating tests is the slow part and the AI has already done the hard work.

B. A human reviews the AI-generated tests for correctness and coverage gaps (the AI may have missed domain-specific edge cases or asserted the wrong expected behavior), adds any missing cases, and only then merges them into the suite, using the AI output as an accelerant, not a final deliverable.

C. Discard all AI-generated tests and write every test by hand, since AI assistance is never appropriate for test code.

D. Use the AI-generated tests as the only tests ever needed for this function going forward, skipping future manual test additions.

**Correct Answer: B**

**Explanation:** This is Domain 3's instance of the cross-domain AI-assisted development pattern:
an assistant can propose unit and regression tests (Task 3.2), but a human still reviews for
correctness and coverage before merging. The AI accelerates the first draft; it doesn't replace
review. See [3.2.5 AI-assisted testing](3.2.5-ai-assisted-testing.md#review-ai-generated-tests)
and [1.1.9 AI-assisted development tools](../../domain-1-development-with-aws-services/task-1.1-build-application-code/1.1.9-ai-assisted-development-tools.md).

**Why the others are wrong:**
- A: Merging without review risks shipping tests that assert incorrect expected behavior (a bad test can be worse than no test, by giving false confidence). The exam explicitly tests recognizing that AI output still needs human verification.
- C: Blanket rejection of AI assistance for test generation isn't the tested correct pattern. The exam tests *safe usage with review*, not avoidance.
- D: Treating AI-generated tests as a permanent, complete, never-updated suite ignores that test coverage should evolve with the code. The AI-assisted pattern being tested is an accelerant within an ongoing review process.
