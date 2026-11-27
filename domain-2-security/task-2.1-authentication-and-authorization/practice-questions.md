# Task 2.1 practice questions

## Q1

**Skills:** 2.1.6

A Lambda function's execution role currently has `"Action": "s3:*", "Resource": "*"` so the
function can read one specific object from one specific bucket at startup. A security review flags
this as overly permissive. What is the correctly scoped replacement?

A. Leave it as-is. Lambda execution roles are not externally callable, so the broad scope carries no real risk.

B. Replace it with a policy granting only `s3:GetObject` on the exact object Amazon Resource Name(s) (ARNs) the function actually reads, following least privilege.

C. Replace `s3:*` with `s3:Get*` on `Resource: "*"`. Broader actions are fine as long as they're read-only.

D. Remove the policy from the role entirely and instead make the bucket public so no permission is required.

**Correct Answer: B**

**Explanation:** Task 2.1's AWS Identity and Access Management (IAM) authorization model expects least-privilege policies: grant the
specific action(s) actually used (`s3:GetObject`, not `s3:*`) scoped to the specific resource
ARN(s) actually needed, not `Resource: "*"`. This is the same principle the exam re-applies across
every resource type (including, in Task 2.3, Bedrock model invocation). See
[2.1.6 Least privilege permissions](2.1.6-least-privilege-permissions.md#least-privilege-in-practice).

**Why the others are wrong:**
- A: An over-privileged execution role is still a risk. If the function (or its code or dependencies) is ever compromised through a vulnerability or supply-chain issue, the blast radius is every S3 action on every bucket, not just the one read the function legitimately needs.
- C: `s3:Get*` on `Resource: "*"` is still far broader than necessary (every bucket's every "Get" action, including ones unrelated to the function's actual job). "Read-only" doesn't make overly broad scope acceptable under least privilege.
- D: Making a bucket public to avoid configuring IAM permissions is a severe security regression, not a fix, and breaks the premise that only this function should access the object.

---

## Q2 (Choose TWO)

**Skills:** 2.1.1

A mobile app needs end users to sign up and sign in with email/password, and after authenticating,
the app needs temporary AWS credentials scoped to that specific user so it can directly call S3 to
upload a profile photo (without routing the upload through a backend). Which TWO Cognito components
are both required to fully satisfy this flow?

A. A Cognito User Pool, to handle user sign-up/sign-in and issue tokens (ID/access/refresh) upon successful authentication.

B. A Cognito Identity Pool, to exchange the User Pool's token for temporary IAM credentials (via AWS Security Token Service, STS) scoped to that user, usable directly against S3.

C. An IAM user created per app user, with long-term access keys embedded in the mobile app.

D. An S3 bucket policy that allows anonymous `PutObject` from any source, removing the need for authentication.

E. AWS Direct Connect, to establish a private network path from the mobile app to S3.

**Correct Answer: A and B**

**Explanation:** This is the canonical Task 2.1 pattern: a **User Pool** handles application-user
authentication (sign-up/sign-in, returns JWTs), and an **Identity Pool** takes the ID token and
exchanges it for temporary, per-user-scoped IAM credentials via STS. That is what lets the mobile
app call S3 directly and securely without a long-lived credential or a backend proxy. The
identity pool's authenticated role can limit each user to their own prefix with the
`${cognito-identity.amazonaws.com:sub}` policy variable. See
[2.1.1 Identity federation](2.1.1-identity-federation.md#cognito-identity-pools-aws-credentials-for-app-users)
and [2.1.7 Assuming IAM roles](2.1.7-assuming-iam-roles.md#sts-operations).

**Why the others are wrong:**
- C: Long-term IAM user access keys embedded in a mobile app are a serious credential-exposure risk (the app binary can be decompiled), which is exactly what Cognito Identity Pools + STS exist to avoid.
- D: Removing authentication and allowing anonymous uploads doesn't satisfy "end users sign up and sign in" at all, and is a data-integrity and abuse risk.
- E: Direct Connect is a private network link between on-premises infrastructure and AWS; it's irrelevant to a public mobile app's authentication flow.

---

## Q3

**Skills:** 2.1.7, 2.1.6

A Lambda function running in Account A (`111111111111`) needs to read objects from an S3 bucket
owned by Account B (`222222222222`). The security team requires that no long-lived credentials or
access keys ever be shared between the two accounts. What is the correct pattern?

A. Create an IAM user in Account B, generate an access key, and store that access key as an environment variable on the Account A Lambda function.

B. Create an IAM role in Account B with a trust policy allowing Account A's Lambda execution role to assume it, and have the Lambda function call `sts:AssumeRole` to obtain short-lived credentials scoped to that role's permissions.

C. Make the S3 bucket public so no cross-account permission is required.

D. Copy the bucket's contents into Account A on a schedule so no cross-account access is ever needed.

**Correct Answer: B**

**Explanation:** Cross-account access without sharing long-lived credentials is exactly what
STS `AssumeRole` is for: Account B defines a role with a trust policy naming Account A's principal,
and Account A's Lambda function assumes that role to get short-lived, automatically expiring
credentials scoped to only what the role permits. Because this is cross-account, the function's
execution role in Account A must also allow `sts:AssumeRole` on the Account B role ARN. See
[2.1.7 Assuming IAM roles](2.1.7-assuming-iam-roles.md#cross-account-access-step-by-step).

**Why the others are wrong:**
- A: This is precisely the "shared long-lived access key between accounts" pattern the requirement forbids, and storing it as an environment variable also risks exposure in logs or the console.
- C: Making the bucket public violates least-exposure principles and isn't a cross-account access pattern at all. It exposes the data to everyone, not just Account A.
- D: Duplicating data on a schedule introduces staleness, cost, and sync complexity, and doesn't solve "access it securely." It avoids the actual requirement.
