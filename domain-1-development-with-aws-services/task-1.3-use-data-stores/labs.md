# Task 1.3 labs

These labs run against Floci, a free local AWS emulator. Start it with `docker run -d -p 4566:4566 --name floci floci/floci` and set up the `awslocal` alias before starting.

## Lab 1: DynamoDB consistency, eventual vs. strong read

**Skills:** 1.3.4

**Why this lab:** "which read guarantees you see your own write" is one of the most frequently tested Task 1.3 distinctions, and it's a one-flag difference in code. Seeing both calls side by side, against the same item, is the fastest way to make the distinction permanent in memory.

```bash
awslocal dynamodb create-table --table-name Counters \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

awslocal dynamodb put-item --table-name Counters \
  --item '{"id": {"S": "visits"}, "count": {"N": "1"}}'

# Eventually consistent read (default): cheaper, may lag on real AWS (Floci returns immediately
# since there's no real multi-AZ replication to simulate, but the API shape/flag is what's tested)
awslocal dynamodb get-item --table-name Counters --key '{"id": {"S": "visits"}}'

# Strongly consistent read: costs double the read capacity on real AWS
awslocal dynamodb get-item --table-name Counters --key '{"id": {"S": "visits"}}' \
  --consistent-read
```

**What to notice:** the only code difference between the two reads is the `--consistent-read` flag (`ConsistentRead=true` in the AWS software development kits, or SDKs). This is exactly the kind of one-line distinction the exam likes to test with a "which read guarantees you see your own write immediately" scenario. Strongly consistent reads work on the base table and local secondary indexes, not on global secondary indexes. See [1.3.4 DynamoDB design and operations](1.3.4-dynamodb-design-and-operations.md#read-consistency).
