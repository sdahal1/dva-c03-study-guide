# Task 2.1 labs

These labs exercise AWS Identity and Access Management (IAM), AWS Security Token Service (STS), and Amazon Cognito. They run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

## Lab 1: IAM least privilege + STS AssumeRole

**Skills:** 2.1.6, 2.1.7

**Why this lab:** nearly every Domain 2 "will this call succeed" question is really asking you to
reason about a trust policy plus a permissions policy together. Writing both from scratch and
then watching a disallowed action get rejected makes the two-document model concrete
instead of a memorized rule.

```bash
# 1. Create a restrictive policy: only read access to one specific S3 bucket
cat > read-only-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:ListBucket"],
    "Resource": [
      "arn:aws:s3:::reports-bucket",
      "arn:aws:s3:::reports-bucket/*"
    ]
  }]
}
EOF
awslocal iam create-policy --policy-name ReadReportsOnly --policy-document file://read-only-policy.json
# 2. Create a role that trusts a Lambda-style principal, attach the policy
cat > trust-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "lambda.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}
EOF
awslocal iam create-role --role-name ReportsReaderRole --assume-role-policy-document file://trust-policy.json
awslocal iam attach-role-policy --role-name ReportsReaderRole \
  --policy-arn arn:aws:iam::000000000000:policy/ReadReportsOnly
# 3. Simulate assuming the role via STS and confirm the temporary credentials
awslocal sts assume-role \
  --role-arn arn:aws:iam::000000000000:role/ReportsReaderRole \
  --role-session-name test-session
# 4. Try (and expect to fail) a write action with those scoped credentials to prove least privilege
awslocal s3api put-object --bucket reports-bucket --key test.txt --body test.txt
# A real AWS account with this exact policy attached would reject s3:PutObject. Floci's IAM
# enforcement is partial, so treat this step as "read the policy and reason about what SHOULD
# happen," which is exactly what the exam asks you to do.
```

**What to notice:** the trust policy (who can assume the role) and the permissions policy (what
the role can do once assumed) are two separate documents, a very common exam confusion point.
In a real AWS account, step 3 would also fail when run as your own user, because this trust policy
names only `lambda.amazonaws.com`. To test assumption yourself, add your user or role Amazon Resource Name (ARN) as an
`AWS` principal in the trust policy. That failure is the "cannot assume the role at all, check the
trust policy" symptom from [2.1.7](2.1.7-assuming-iam-roles.md#how-role-assumption-works).

## Lab 2: Cognito user pool sign-up and sign-in flow

**Skills:** 2.1.1, 2.1.2

**Why this lab:** candidates frequently mix up which Cognito token (ID, access, refresh)
gets validated at an API Gateway authorizer. Running a real sign-in and inspecting all
three tokens in the response removes the ambiguity that a diagram alone can leave.

```bash
# 1. Create a user pool and an app client
POOL_ID=$(awslocal cognito-idp create-user-pool --pool-name demo-pool --query 'UserPool.Id' --output text)
CLIENT_ID=$(awslocal cognito-idp create-user-pool-client \
  --user-pool-id $POOL_ID --client-name demo-client \
  --explicit-auth-flows ALLOW_USER_PASSWORD_AUTH ALLOW_REFRESH_TOKEN_AUTH \
  --query 'UserPoolClient.ClientId' --output text)
# 2. Create a user and set a permanent password (skipping email verification for the lab)
awslocal cognito-idp admin-create-user --user-pool-id $POOL_ID --username shopper1 \
  --user-attributes Name=email,Value=shopper1@example.com --message-action SUPPRESS
awslocal cognito-idp admin-set-user-password --user-pool-id $POOL_ID --username shopper1 \
  --password 'TempPass123!' --permanent
# 3. Authenticate: this is the API path Task 2.1's "application user authentication" tests
awslocal cognito-idp initiate-auth \
  --client-id $CLIENT_ID \
  --auth-flow USER_PASSWORD_AUTH \
  --auth-parameters USERNAME=shopper1,PASSWORD=TempPass123!
```

**What to notice:** the response contains `IdToken`, `AccessToken`, and `RefreshToken`, three
distinct tokens with different purposes (identity claims, API authorization with scopes, and
obtaining new tokens without re-entering credentials). The ID and access tokens are JWTs you can
decode; the refresh token is opaque to your app. An API Gateway Cognito user pool authorizer
accepts a valid **ID token** by default, and requires the **access token** when the method is
configured with OAuth scopes. Because this lab signs in through `InitiateAuth`, the access token
carries only the `aws.cognito.signin.user.admin` scope, so it would not satisfy a method that
requires a custom scope. See [2.1.1](2.1.1-identity-federation.md#tokens) and
[2.1.2](2.1.2-access-controls-tokens-and-mfa.md#api-gateway-authorization-options).

## Lab 3: IAM policy evaluation order, permissions boundary as an unbreakable ceiling

**Skills:** 2.1.6

**Why this lab:** [2.1.6](2.1.6-least-privilege-permissions.md#four-ceilings-compared) explains
permissions boundaries as a ceiling that caps a role's *effective* permissions regardless of how
broad its own identity policy is. This lab makes that concrete by attaching a boundary narrower
than the role's own policy and confirming (by reasoning through Floci's partial enforcement, the
same way Lab 1 does) that the boundary, not the identity policy, determines the effective limit.

```bash
# 1. Create a broad identity-based policy: full S3 access
cat > broad-s3-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{ "Effect": "Allow", "Action": "s3:*", "Resource": "*" }]
}
EOF
awslocal iam create-policy --policy-name BroadS3Access --policy-document file://broad-s3-policy.json
# 2. Create a MUCH narrower permissions boundary: read-only, one bucket
cat > boundary-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:ListBucket"],
    "Resource": ["arn:aws:s3:::team-reports-bucket", "arn:aws:s3:::team-reports-bucket/*"]
  }]
}
EOF
awslocal iam create-policy --policy-name ReadOnlyBoundary --policy-document file://boundary-policy.json
# 3. Create a role with the BROAD policy attached AND the narrow boundary set
cat > trust-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "lambda.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}
EOF
awslocal iam create-role --role-name BoundedRole \
  --assume-role-policy-document file://trust-policy.json \
  --permissions-boundary arn:aws:iam::000000000000:policy/ReadOnlyBoundary
awslocal iam attach-role-policy --role-name BoundedRole \
  --policy-arn arn:aws:iam::000000000000:policy/BroadS3Access
# 4. Confirm the role's attached policy AND its boundary independently
awslocal iam list-attached-role-policies --role-name BoundedRole
awslocal iam get-role --role-name BoundedRole --query 'Role.PermissionsBoundary'
```

**What to notice:** the role's own attached policy grants `s3:*` on everything, but its
permissions boundary only ever allows read access to one bucket. Per the evaluation logic in
[2.1.6](2.1.6-least-privilege-permissions.md#policy-evaluation-logic), the role's *effective*
permission is the intersection of the two, meaning a real AWS account would reject any S3 write
or any access to a different bucket, no matter how broad `BroadS3Access` looks. This is the
single most misread line in a Domain 2 IAM scenario: a role's attached policy is not the same
thing as its effective permission once a boundary is in play.
