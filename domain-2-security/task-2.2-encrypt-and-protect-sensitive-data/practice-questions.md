# Task 2.2 practice questions

## Q1

**Skills:** 2.2.2, 2.2.1

An internal service-to-service API call between two microservices must be encrypted in transit, and
the certificate must be trusted automatically by internal clients without manually distributing a
custom root certificate, while using internal-only hostnames (e.g. `billing.internal.corp`) that
a public certificate authority won't issue for. What should be used?

A. AWS Certificate Manager (ACM) issuing a public certificate for the internal hostname.

B. AWS Private CA, issuing a private certificate for the internal hostname from a private certificate hierarchy that's automatically trusted within the environments configured to trust that private CA.

C. A self-signed certificate generated manually on each service and copied by hand to every client.

D. Disable TLS for internal traffic since it's "inside the VPC" and therefore considered safe by default.

**Correct Answer: B**

**Explanation:** Task 2.2 distinguishes public ACM certificates (for internet-facing, publicly
resolvable domains) from **AWS Private CA** (for internal-only hostnames and private trust
hierarchies), which is exactly the scenario described. See
[2.2.2 Certificate management](2.2.2-certificate-management.md#aws-private-ca).

**Why the others are wrong:**
- A: Public certificate authorities (CAs), including ACM's public certificate issuance, require domain validation against a publicly resolvable domain you own and won't issue for purely internal hostnames like `.internal.corp`. Public certificates are also written to public Certificate Transparency logs.
- C: Manually distributing self-signed certificates doesn't scale, has no automated renewal, and is exactly the operational burden Private CA eliminates.
- D: Disabling TLS "because it's inside the VPC" ignores encryption-in-transit requirements. A network boundary isn't equivalent to an encrypted channel, and Task 2.2 expects encryption in transit regardless of network location.

---

## Q2

**Skills:** 2.2.4, 2.2.1

A company stores customer records in DynamoDB and a compliance requirement states that a specific
internal security team must be able to audit and control exactly which principals are allowed to
decrypt the data, with that control independent of any other AWS account's keys. Which encryption
configuration satisfies this?

A. The default AWS owned key, since it requires no configuration.

B. An AWS managed key (aws/dynamodb), since AWS manages rotation for you.

C. A customer managed AWS Key Management Service (AWS KMS) key with a key policy explicitly scoping which principals can use `kms:Decrypt`, giving the security team direct, auditable control over access.

D. Client-side encryption using a hardcoded AES key stored in the application's source code.

**Correct Answer: C**

**Explanation:** Task 2.2 distinguishes encryption-at-rest options by who controls the key: AWS
owned keys and AWS managed keys don't give the customer key-policy control, while a
**customer managed key** lets the security team write an explicit key policy governing
exactly which principals can decrypt, and every use is logged in CloudTrail. That satisfies "must
control and audit exactly who can decrypt it." See
[2.2.4 KMS key management](2.2.4-kms-key-management.md#access-control-key-policies-iam-and-grants).

**Why the others are wrong:**
- A: AWS owned keys are fully managed by AWS with no customer visibility into or control over the key policy. They can't satisfy an "our security team controls exactly who can decrypt" requirement.
- B: AWS managed keys appear in the customer's account, but their key policies are created and controlled by AWS and can't be edited, so the security team can't scope which principals may decrypt beyond what the service allows.
- D: Hardcoding an encryption key in source code is a severe security anti-pattern (the key is now wherever the source is: version control, build artifacts) and is a Task 2.2 "what not to do" example, not a valid encryption design.

---

## Q3

**Skills:** 2.2.7

An application currently stores its RDS database password as a plaintext Lambda environment
variable, baked in at deploy time. The team now needs the password to rotate automatically on a
schedule without any code redeploy, and must never appear in plaintext in the function's
configuration. What should replace the current approach?

A. Move the password into AWS Systems Manager Parameter Store as a `String` type parameter, read at runtime.

B. Store the password in AWS Secrets Manager and have the application fetch it at runtime via the Secrets Manager SDK call, enabling Secrets Manager's built-in automatic rotation.

C. Base64-encode the password and keep it as a Lambda environment variable, since encoding hides it from casual viewing.

D. Store the password in a `.env` file committed to the application's Git repository for version-controlled consistency.

**Correct Answer: B**

**Explanation:** Task 2.2 distinguishes Secrets Manager (built-in automatic rotation,
designed for credentials) from Parameter Store (no native rotation). "An application must not
hardcode a database password anywhere, and the password rotates automatically" points to Secrets
Manager. In Lambda, fetch it through the Parameters and Secrets Lambda Extension or an SDK cache so
warm invocations don't call Secrets Manager every time. See
[2.2.7 Secrets management](2.2.7-secrets-management.md#secrets-manager-vs-parameter-store).

**Why the others are wrong:**
- A: Parameter Store's `String` and `SecureString` types don't include Secrets Manager's native automatic rotation for database credentials. A `String` parameter isn't even encrypted. Parameter Store can hold a secret but doesn't rotate it on a schedule the way Secrets Manager does.
- C: Base64 is an encoding, not encryption. It's trivially reversible and provides no confidentiality, and changing it still requires a redeploy, failing the "rotate without redeploy" requirement.
- D: Committing a secret to source control is one of the most common real-world credential-leak causes and is the anti-pattern Task 2.2 tests against.

---

## Q4

**Skills:** 2.2.6

A security team runs a scan across an S3 data lake used by multiple application teams and needs to
automatically discover which objects contain sensitive data such as credit card numbers or national
ID numbers, without manually reviewing every file. Which AWS service is purpose-built for this?

A. AWS Config, since it tracks resource configuration changes.

B. Amazon Macie, which uses machine learning and pattern matching to discover and classify sensitive data (PII, financial data, credentials) stored in S3.

C. AWS CloudTrail, since it logs every S3 API call.

D. Amazon GuardDuty, since it detects malicious network activity.

**Correct Answer: B**

**Explanation:** Amazon Macie is purpose-built for discovering and classifying sensitive data
(personally identifiable information (PII), financial identifiers, credentials) already stored in
S3. It is a detective control: it reports findings and doesn't encrypt or block anything. Macie
isn't on the DVA-C03 in-scope service list, but it's the standard answer when a question names it.
See [2.2.6 Sensitive data detection and masking](2.2.6-sensitive-data-detection-and-masking.md#detective-plus-preventive).

**Why the others are wrong:**
- A: AWS Config tracks resource *configuration* state and drift over time (for example, "is encryption enabled on this bucket"), not the *content* of objects stored inside it.
- C: CloudTrail logs API activity (who called what, when). It has no visibility into the actual content of S3 objects to classify sensitive data within them.
- D: GuardDuty is a threat-detection service focused on anomalous network and API behavior and compromised credentials or resources, not content classification of stored data.
