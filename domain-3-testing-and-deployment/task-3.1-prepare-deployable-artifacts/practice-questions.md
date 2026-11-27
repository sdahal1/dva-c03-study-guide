# Task 3.1 practice questions

## Q1

**Skills:** 3.1.2

A team pushes a new container image to Amazon Elastic Container Registry (ECR) on every commit to `main`, tagging each image
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
[3.1.2 Container images and ECR](3.1.2-container-images-and-ecr.md#lifecycle-policies).

**Why the others are wrong:**
- A: Manual weekly review doesn't scale and is exactly the toil lifecycle policies exist to automate. It's also error-prone (risk of deleting an image still referenced by a running deployment).
- C: ECR has no Glacier storage class, and storage class is set per image, not per repository. ECR does have an archive storage class (since November 2025) that lifecycle rules can transition images into, but archived images still appear in the repository, cannot be pulled, and carry a 90-day minimum storage charge. Archiving is for images you must keep, not a way to clean up thousands of abandoned ones.
- D: Overwriting a single `latest` tag removes any ability to roll back to a specific prior commit's image, which breaks traceability and defeats the whole point of commit-SHA tagging. It doesn't solve the "cleanup" problem either: each overwritten image stays in the repository as an untagged image.


---

## Q2

**Skills:** 3.1.4, 3.1.1

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
Store, Secrets Manager) rather than baking it into the deployment artifact. This decouples
configuration changes from code deployments entirely, letting ops update a flag or endpoint without
touching the Lambda package. See [3.1.4 Environment-specific configuration](3.1.4-environment-specific-configuration.md#choosing-the-mechanism)
and [3.1.1 Packaging dependencies and configuration](3.1.1-packaging-dependencies-and-configuration.md#what-goes-into-an-artifact).

**Why the others are wrong:**
- A: Comments don't change the fact that changing a value still requires a code change and redeploy. This doesn't solve the stated problem at all.
- C: Environment variables set once and never updated reproduce the same problem in a different location (still requires a function update/redeploy to change). The fix is specifically to externalize to a service designed for *runtime* lookups, not a static config baked at deploy time.
- D: API Gateway resource policies control who can invoke the API. They have no mechanism for storing arbitrary application configuration values.


---

## Q3

**Skills:** 3.1.5, 3.1.4

An API has `dev`, `test`, and `prod` stages, each needing different throttle limits and a different
backend Lambda alias, and the team wants this stage-specific configuration to be part of the
versioned deployment artifact rather than manually clicked into the console after each deploy. How
should this be achieved for Task 3.1?

A. Define the API Gateway stage configuration (stage variables, throttling settings) in the infrastructure as code (IaC) template (SAM/CDK/CloudFormation) alongside the rest of the deployment, so it's version-controlled and reproducible.

B. Manually configure each stage via the console immediately after every deployment, documenting the steps in a wiki.

C. Use a single shared stage for all three environments, relying on separate AWS accounts to distinguish them instead.

D. Store stage configuration in a spreadsheet maintained by the operations team outside of version control.

**Correct Answer: A**

**Explanation:** Task 3.1 treats API Gateway stage configuration as part of the deployable
artifact. Defining it in IaC (SAM/CDK/CloudFormation) makes it reproducible, versioned, and
automatically applied by the pipeline rather than a manual, error-prone post-deploy step. See
[3.1.5 API Gateway stages and endpoints](3.1.5-api-gateway-stages-and-endpoints.md#stages-as-code).

**Why the others are wrong:**
- B: Manual console clicks after every deployment are exactly the toil and configuration-drift risk IaC exists to eliminate, and documentation in a wiki doesn't enforce consistency the way code does.
- C: Using one shared stage for three environments defeats the purpose of having distinct dev/test/prod stages with separate throttle limits and aliases, and risks dev/test traffic affecting prod configuration.
- D: A spreadsheet outside version control has no enforcement mechanism and can drift silently from what's actually deployed. It's not a deployment artifact at all.


---
