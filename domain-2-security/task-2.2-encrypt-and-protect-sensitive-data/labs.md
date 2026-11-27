# Task 2.2 labs

These labs exercise AWS Secrets Manager, Systems Manager Parameter Store, AWS Key Management Service (AWS KMS), and Lambda. They run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting. Floci's IAM and KMS enforcement is partial, so a command that succeeds locally only confirms the configuration is structured correctly; reason through what a real account would enforce at each step.

## Lab 1: Secrets Manager rotation vs Parameter Store

**Skills:** 2.2.7

**Why this lab:** the exam tests this as a scenario ("the secret rotates automatically...") rather
than naming the service. Storing one value in each service side by side and calling
`rotate-secret` makes the capability gap (rotation vs no native rotation) something you've
actually exercised, not just read in a comparison table.

```bash
# 1. Store a "database password" as a secret
awslocal secretsmanager create-secret --name prod/db/password --secret-string 'S3cretP@ss'

# 2. Store a non-secret config value as a parameter instead
awslocal ssm put-parameter --name /prod/app/log-level --value "INFO" --type String

# 3. Retrieve both from "application code"
awslocal secretsmanager get-secret-value --secret-id prod/db/password --query 'SecretString'
awslocal ssm get-parameter --name /prod/app/log-level --query 'Parameter.Value'

# 4. Rotate the secret (illustrates the capability Parameter Store lacks natively)
awslocal secretsmanager rotate-secret --secret-id prod/db/password
```

In a real account, `rotate-secret` also needs a rotation Lambda function (or managed rotation) configured on the secret.

**What to notice:** both services store string values and both can be read with the SDK. The
exam-relevant difference is rotation support and per-secret pricing (Secrets Manager) vs
no-charge simple config storage (Parameter Store standard tier), covered in
[2.2.7 Secrets management](2.2.7-secrets-management.md#secrets-manager-vs-parameter-store).

## Lab 2: KMS envelope encryption end to end

**Skills:** 2.2.4, 2.2.3

**Why this lab:** the exam likes to ask why you can't just "encrypt a 10 GB file with KMS directly".
Walking through `GenerateDataKey` and seeing that only the small data key, never the bulk
payload, touches the KMS API makes the 4 KB-per-call limit and the envelope encryption pattern
click as a mechanism, not trivia.

```bash
# 1. Create a customer managed KMS key
KEY_ID=$(awslocal kms create-key --description "app-data-key" --query 'KeyMetadata.KeyId' --output text)
awslocal kms create-alias --alias-name alias/app-data-key --target-key-id $KEY_ID

# 2. Generate a data key (returns both plaintext and the KMS-encrypted version)
awslocal kms generate-data-key --key-id alias/app-data-key --key-spec AES_256 > datakey.json
cat datakey.json
PLAINTEXT_KEY=$(python3 -c "import json; print(json.load(open('datakey.json'))['Plaintext'])")

# 3. "Encrypt" a file locally using the plaintext data key (simplified; real code uses an AES library)
echo "This is sensitive application data" > plaintext.txt
# (In real application code: AES-encrypt plaintext.txt with PLAINTEXT_KEY here, then discard the key)

# 4. Store only the CIPHERTEXT data key alongside your encrypted data, never the plaintext key
python3 -c "import json; print(json.load(open('datakey.json'))['CiphertextBlob'])" > encrypted_datakey.txt
cat encrypted_datakey.txt

# 5. To decrypt later: call Decrypt on the stored ciphertext key to get the plaintext key back.
#    The CLI returns the blob base64-encoded, so decode it to binary before passing it with fileb://
base64 -d encrypted_datakey.txt > encrypted_datakey.bin
awslocal kms decrypt --ciphertext-blob fileb://encrypted_datakey.bin --query 'Plaintext' --output text
```

**What to notice:** KMS is only ever called with the small data key, never the actual application
payload. This is envelope encryption in practice, and it's why KMS can protect arbitrarily large
datasets even though its own `Encrypt` and `Decrypt` APIs are capped at 4 KB per call. For the
local AES step written out in Python, see
[2.2.4 KMS key management](2.2.4-kms-key-management.md#envelope-encryption).

## Lab 3: Securing a Lambda secret end to end: environment variable reference plus IAM and KMS scoping

**Skills:** 2.2.7, 2.2.4

**Why this lab:** a secret's value shouldn't live in a plain environment variable, only its
reference should. This lab builds that pattern, including the two separate IAM grants
(`GetSecretValue` and `kms:Decrypt`) a function's execution role needs, so the two-grant
requirement is something you've configured, not just read about.

```bash
# 1. Create a customer managed KMS key and a secret encrypted with it
KEY_ID=$(awslocal kms create-key --description "secrets-cmk" --query 'KeyMetadata.KeyId' --output text)
awslocal kms create-alias --alias-name alias/secrets-cmk --target-key-id $KEY_ID
awslocal secretsmanager create-secret --name prod/payments/db-password \
  --secret-string 'S3cretP@ss' --kms-key-id alias/secrets-cmk
SECRET_ARN=$(awslocal secretsmanager describe-secret --secret-id prod/payments/db-password \
  --query 'ARN' --output text)

# 2. Create an execution role with BOTH required grants; omit one on purpose first
cat > lambda-trust.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "lambda.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}
EOF
awslocal iam create-role --role-name PaymentsFnRole --assume-role-policy-document file://lambda-trust.json

cat > secret-read-only.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "secretsmanager:GetSecretValue",
    "Resource": "$SECRET_ARN"
  }]
}
EOF
awslocal iam put-role-policy --role-name PaymentsFnRole \
  --policy-name ReadThisSecretOnly --policy-document file://secret-read-only.json

# 3. Deploy the function with ONLY the secretsmanager grant (no kms:Decrypt yet) and
#    set the env var to the ARN, never the password itself
cat > handler.py <<'EOF'
import os, json, boto3
def handler(event, context):
    sm = boto3.client("secretsmanager", endpoint_url="http://localhost:4566")
    secret = sm.get_secret_value(SecretId=os.environ["DB_SECRET_ARN"])
    return {"status": "retrieved", "length": len(secret["SecretString"])}
EOF
zip handler.zip handler.py
awslocal lambda create-function --function-name payments-fn \
  --runtime python3.12 --handler handler.handler \
  --zip-file fileb://handler.zip \
  --role arn:aws:iam::000000000000:role/PaymentsFnRole \
  --environment "Variables={DB_SECRET_ARN=$SECRET_ARN}"

# 4. Invoke and reason about the expected real-AWS behavior: GetSecretValue is allowed at the
#    Secrets Manager layer, but decrypting a secret encrypted with a customer managed key ALSO
#    requires kms:Decrypt on that specific key. Add the missing grant and re-invoke
cat > kms-decrypt.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "kms:Decrypt",
    "Resource": "arn:aws:kms:us-east-1:000000000000:key/$KEY_ID"
  }]
}
EOF
awslocal iam put-role-policy --role-name PaymentsFnRole \
  --policy-name DecryptSecretsCMK --policy-document file://kms-decrypt.json

awslocal lambda invoke --function-name payments-fn /tmp/out.json && cat /tmp/out.json
```

**What to notice:** the environment variable (`DB_SECRET_ARN`) never holds the password itself,
only a pointer to it. Getting the real value requires two separate IAM grants
(`secretsmanager:GetSecretValue` plus `kms:Decrypt` on the specific customer managed key), and in a
real account the key policy must also allow the role. See
[2.2.7 Secrets management](2.2.7-secrets-management.md#the-execution-role-needs-two-grants).
Because Floci's IAM and KMS enforcement is partial, treat a successful invoke here as confirmation
the configuration is structured correctly, not as proof a real AWS account would behave the same
without the KMS grant.
