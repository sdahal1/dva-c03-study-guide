# Task 2.3 labs

These labs exercise the parts of Task 2.3 that an emulator can run: AWS PrivateLink interface virtual private cloud (VPC) endpoints for Amazon Bedrock, and an AWS Step Functions human-approval gate for AI agent actions. Floci does not emulate Bedrock model inference, Guardrails, or Bedrock Agents. They run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

## Lab 1: VPC endpoint for private AI-service connectivity

**Skills:** 2.3.2, 2.3.1

**Why this lab:** Task 2.3's "VPC endpoints for data privacy" is new on this exam version and easy
to treat as abstract policy language. Creating the actual endpoint resource against a real
service name (`bedrock-runtime`) shows it's the same PrivateLink mechanism you already use for
other AWS services, just pointed at a different service.

```bash
# 1. Create a VPC (Floci gives you a default VPC, but let's be explicit)
VPC_ID=$(awslocal ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)

# 2. Create an interface VPC endpoint for Bedrock runtime, the private path described
#    in 2.3.2
VPCE_ID=$(awslocal ec2 create-vpc-endpoint \
  --vpc-id $VPC_ID \
  --service-name com.amazonaws.us-east-1.bedrock-runtime \
  --vpc-endpoint-type Interface \
  --query 'VpcEndpoint.VpcEndpointId' --output text)

# 3. Attach a restrictive endpoint policy so only a specific role can use this private path
cat > endpoint-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::000000000000:role/BedrockCallerRole"},
    "Action": "bedrock:InvokeModel",
    "Resource": "*"
  }]
}
EOF
awslocal ec2 modify-vpc-endpoint --vpc-endpoint-id $VPCE_ID \
  --policy-document file://endpoint-policy.json
awslocal ec2 describe-vpc-endpoints --vpc-endpoint-ids $VPCE_ID
```

**What to notice:** this is the same `create-vpc-endpoint` API you'd use for any other interface
endpoint, just pointed at `bedrock-runtime`. Task 2.3's "VPC endpoints for data privacy" isn't a
new mechanism; it's the existing PrivateLink pattern applied to a new service category. Without
the custom policy, the endpoint's default policy allows full access, so step 3 is what actually
restricts the private path. In a real account you would also attach a security group to the
endpoint that allows HTTPS (443) only from the calling function's security group, and add
`bedrock-agent-runtime` if the application uses agents or knowledge bases. See
[2.3.2 AI data privacy and private connectivity](2.3.2-ai-data-privacy-and-private-connectivity.md#private-connectivity-with-privatelink).

## Lab 2: Simulating the AI-agent human-approval gate

**Skills:** 2.3.4

**Why this lab:** "an AI agent proposes, a human approves before an irreversible action executes"
is the single rule that answers most Task 2.3 agent-security questions. This lab makes it
concrete by building a state machine that cannot proceed past the approval step
without an external signal, instead of just asserting the rule from memory.

Floci doesn't emulate Bedrock Agents, but the **approval-gate pattern** underneath "require human
approval for irreversible agent actions" is a Step Functions wait-for-callback task. It is fully
exercisable locally, and it is the part of this topic that's actually testable hands-on.

```bash
# 1. A state machine: "propose action" -> wait for human approval token -> "execute action"
cat > agent-approval.json <<'EOF'
{
  "Comment": "Simulates an AI agent proposing an irreversible action and waiting for a human",
  "StartAt": "ProposeAction",
  "States": {
    "ProposeAction": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke.waitForTaskToken",
      "Parameters": {
        "FunctionName": "notify-approver",
        "Payload": {
          "action": "cancel_subscription_and_refund",
          "taskToken.$": "$$.Task.Token"
        }
      },
      "Next": "ExecuteAction"
    },
    "ExecuteAction": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {"FunctionName": "execute-refund"},
      "End": true
    }
  }
}
EOF

# 2. The "notify-approver" function would normally page a human; for the lab, just log the token
cat > notify_approver.py <<'EOF'
def handler(event, context):
    print("APPROVAL NEEDED. Action:", event["action"])
    print("Task token (a human/system calls SendTaskSuccess with this):", event["taskToken"])
    return {"status": "pending"}
EOF
zip notify_approver.zip notify_approver.py
awslocal lambda create-function --function-name notify-approver \
  --runtime python3.12 --handler notify_approver.handler \
  --zip-file fileb://notify_approver.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

cat > execute_refund.py <<'EOF'
def handler(event, context):
    print("Refund executed.")
    return {"status": "done"}
EOF
zip execute_refund.zip execute_refund.py
awslocal lambda create-function --function-name execute-refund \
  --runtime python3.12 --handler execute_refund.handler \
  --zip-file fileb://execute_refund.zip \
  --role arn:aws:iam::000000000000:role/lambda-role

# 3. Create and start the state machine
awslocal stepfunctions create-state-machine \
  --name agent-approval-gate \
  --definition file://agent-approval.json \
  --role-arn arn:aws:iam::000000000000:role/sfn-role

EXEC_ARN=$(awslocal stepfunctions start-execution \
  --state-machine-arn arn:aws:states:us-east-1:000000000000:stateMachine:agent-approval-gate \
  --query 'executionArn' --output text)

# 4. The execution is now PAUSED waiting on a human. Grab the task token from the Lambda logs,
#    then "approve" it by sending task success (this is the human-in-the-loop step)
awslocal logs tail /aws/lambda/notify-approver --since 1m
# awslocal stepfunctions send-task-success --task-token <TOKEN-FROM-LOGS> --task-output '{}'

# 5. Confirm it only completes AFTER the approval, never before
awslocal stepfunctions describe-execution --execution-arn $EXEC_ARN
```

**What to notice:** the execution pauses at `ProposeAction` until something external
calls `SendTaskSuccess` with the matching token. There is no code path where `ExecuteAction` runs
without that external signal. A reviewer who rejects the action would call `SendTaskFailure`
instead. This is the concrete, testable mechanism behind "AI agent proposes, human approves before
an irreversible action executes" from
[2.3.4 AI agent security](2.3.4-ai-agent-security.md#human-in-the-loop-approval).
