# Task 4.3 labs

These labs run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

## Lab 1: Lambda memory and performance tuning

**Skills:** 4.3.2

**Why this lab:** "should we raise or lower memory" is one of Domain 4's most commonly
mis-answered question types because cost isn't monotonic with memory. Timing the same CPU-bound function (one limited by its central processing unit, or CPU)
at two memory settings turns the memory × duration cost relationship into something you've
measured, not just read as a formula.

```bash
# 1. Deploy the same CPU-bound function at two memory settings and compare duration
cat > cpuhandler.py <<'EOF'
def handler(event, context):
    total = 0
    for i in range(10_000_000):
        total += i
    return {"statusCode": 200, "body": str(total)}
EOF
zip cpufunction.zip cpuhandler.py

awslocal lambda create-function --function-name cpu-128mb \
  --runtime python3.12 --handler cpuhandler.handler \
  --zip-file fileb://cpufunction.zip \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --memory-size 128

awslocal lambda create-function --function-name cpu-1024mb \
  --runtime python3.12 --handler cpuhandler.handler \
  --zip-file fileb://cpufunction.zip \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --memory-size 1024

# 2. Invoke both and compare the reported duration
time awslocal lambda invoke --function-name cpu-128mb out128.json
time awslocal lambda invoke --function-name cpu-1024mb out1024.json
```

**What to notice:** Lambda allocates CPU proportionally to memory, so the 1024 MB function should
finish the same CPU-bound loop noticeably faster than the 128 MB one. That is the exact tradeoff
Task 4.3 tests: raising memory can *reduce* total cost even though the per-GB-second rate is
higher, because duration drops enough to offset it. Always compute cost = (memory × duration), not
just eyeball the memory setting. See [4.3.2 Memory and compute tuning](4.3.2-memory-and-compute-tuning.md#reading-the-evidence).

## Lab 2: Cost math, proving the memory/duration/cost relationship

**Skills:** 4.3.2

**Why this lab:** the cost table in 4.3.2 is illustrative. Running the same workload at five real
memory sizes and computing memory × duration yourself confirms the relationship holds (and shows
you exactly where diminishing returns kick in) instead of asking you to trust a static table.

Builds on the [Lambda cost math section of 4.3.2](4.3.2-memory-and-compute-tuning.md#cost-math)
with real numbers instead of an illustrative table.

```bash
# 1. Reuse the cpuhandler.py from Lab 1, but test FIVE memory sizes and record actual durations
for MEM in 128 512 1024 1769 3008; do
  awslocal lambda create-function --function-name cpu-test-$MEM \
    --runtime python3.12 --handler cpuhandler.handler \
    --zip-file fileb://cpufunction.zip \
    --role arn:aws:iam::000000000000:role/lambda-role \
    --memory-size $MEM 2>/dev/null || true
  echo "=== ${MEM}MB ==="
  awslocal lambda invoke --function-name cpu-test-$MEM --log-type Tail out.json \
    --query 'LogResult' --output text | base64 --decode | grep "Duration\|Billed Duration\|Memory"
done
```

**What to notice:** plot (memory in GB) × (billed duration in seconds) for each row. The product
should be roughly flat (or even dip) through the CPU-bound range and then start climbing once the
function stops being CPU-bound, confirming the shape of the 4.3.2 cost table with real
(emulated) numbers instead of taking it on faith. The Python loop is single-threaded, so it cannot
use more than one virtual CPU (vCPU); past 1,769 MB the extra memory buys nothing for this code.

## Lab 3: Simulating a cost-anomaly investigation workflow

**Skills:** 4.3.6

**Why this lab:** the whole point of the [cost anomaly investigation workflow](4.3.6-ai-assisted-optimization.md#cost-anomaly-investigation-workflow)
is that a cost anomaly alert never tells you *who* caused it. This lab reproduces the
"forgotten weekend instance" scenario using Floci's synthesized cost data and
CloudTrail together, so the two-tool handoff (Cost Explorer for *what*, CloudTrail for *who*) is
something you've actually executed, not just read about. Cost Explorer and Cost Anomaly Detection
are not on the DVA-C03 service list; CloudTrail is.

```bash
# 1. Tag a resource for cost allocation, simulating a shared multi-team account
awslocal ec2 run-instances --image-id ami-00000000 --instance-type m5.4xlarge \
  --count 1 --tag-specifications 'ResourceType=instance,Tags=[{Key=Team,Value=ML}]'
INSTANCE_ID=$(awslocal ec2 describe-instances --query \
  'Reservations[0].Instances[0].InstanceId' --output text)

# 2. Pull a Cost Explorer breakdown by usage type: the "WHAT and HOW MUCH" half
awslocal ce get-cost-and-usage \
  --time-period Start=2026-11-01,End=2026-11-30 \
  --granularity MONTHLY \
  --filter '{"Dimensions":{"Key":"SERVICE","Values":["Amazon Elastic Compute Cloud - Compute"]}}' \
  --group-by Type=DIMENSION,Key=USAGE_TYPE \
  --metrics "UnblendedCost"

# 3. Find WHO launched it: the half Cost Explorer can never answer on its own
awslocal cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=RunInstances

# 4. Attribute to a team via the cost allocation tag, closing the loop
awslocal ce get-cost-and-usage \
  --time-period Start=2026-11-01,End=2026-11-30 \
  --granularity MONTHLY \
  --group-by Type=TAG,Key=Team \
  --metrics "UnblendedCost"
```

**What to notice:** step 2 alone never names a responsible team or person. Only step 3
(CloudTrail) and step 4 (tag-based attribution) close that gap. This mirrors the structural
pairing used throughout Domain 4's cost and governance content: a detection or quantification tool
paired with CloudTrail for "who", never answered by the detection tool alone. In a real account,
the Team tag must also be activated as a cost allocation tag in the Billing console before step 4
returns anything.
