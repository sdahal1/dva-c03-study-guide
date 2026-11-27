# Task 1.3 practice questions

## Q1

**Skills:** 1.3.4

A developer's application calls `DynamoDB:Query` against a table expected to return several
thousand items matching a partition key. The application code calls `Query` once and only
processes the items in the response, then moves on. In production, the application is silently
missing a large percentage of matching items. What is the most likely cause, and the correct fix?

A. DynamoDB silently drops items over 400 KB; the fix is to compress item attributes before writing them.

B. `Query` results are capped at 1 MB per call and the response includes a `LastEvaluatedKey` when more data remains; the fix is to loop on `Query` using `ExclusiveStartKey` until `LastEvaluatedKey` is absent.

C. The table is in eventually-consistent mode, so roughly half of all items are temporarily invisible; the fix is to enable strong consistency on the table.

D. The AWS SDK's default retry policy drops any item that required a retry; the fix is to disable retries in the SDK client configuration.

**Correct Answer: B**

**Explanation:** A single `Query` (or `Scan`) call returns at most 1 MB of data and sets
`LastEvaluatedKey` in the response whenever more matching items exist beyond that page. Application
code must paginate: loop, passing the previous response's `LastEvaluatedKey` as the next call's
`ExclusiveStartKey`, until `LastEvaluatedKey` is no longer returned. This is the "use the SDK
correctly against the data store" behavior tested across skills 1.1.5 and 1.3.4.
See [1.3.4 DynamoDB design and operations](1.3.4-dynamodb-design-and-operations.md#pagination) and
[1.1.5 AWS SDKs and service APIs](../task-1.1-build-application-code/1.1.5-aws-sdks-and-service-apis.md).

**Why the others are wrong:**
- A: DynamoDB's per-item size limit is 400 KB and exceeding it causes a write-time validation error, not a silent read-time drop. It is unrelated to this symptom.
- C: Consistency (eventual vs. strong) affects whether a read reflects the *most recent write*, not whether a single `Query` call returns all matching items across multiple pages. It never causes missing items across a whole result set.
- D: The SDK's default retry policy retries failed requests with backoff; it does not drop successfully-returned items, and disabling retries would make transient throttling failures worse, not fix pagination.


---

## Q2

**Skills:** 1.3.4

A banking application writes a transaction to a DynamoDB table and must display the updated balance
to the same user on the very next screen, which issues a `GetItem` immediately after the write
completes. The displayed value must always reflect that write, every time, with no chance of
showing a stale value. How should the `GetItem` call be configured?

A. Leave the default read settings, because DynamoDB always returns the most recent write by default.

B. Set `ConsistentRead=true` on the `GetItem` call, accepting roughly double the read-capacity cost compared to an eventually consistent read.

C. Add a manual `sleep(2)` before the `GetItem` call to give replication time to catch up, then use the default eventually consistent read.

D. Switch the table to use Global Tables, since multi-region replication guarantees the latest value is always returned.

**Correct Answer: B**

**Explanation:** DynamoDB's default read is eventually consistent (lower cost, might briefly
reflect a slightly stale value right after a write). For a "must read your own write immediately,
every time" requirement, Task 1.3 expects you to request a strongly consistent read with
`ConsistentRead=true` on `GetItem`/`Query`, which costs double the read capacity units of an
eventually consistent read of the same size. See
[1.3.4 DynamoDB design and operations](1.3.4-dynamodb-design-and-operations.md#read-consistency).

**Why the others are wrong:**
- A: DynamoDB's default read is explicitly eventually consistent, not strongly consistent. This is the misconception the question is testing against.
- C: A fixed sleep is not a guarantee against eventual consistency's variable propagation delay and is not a real engineering solution AWS exams accept. It is an anti-pattern, not a fix.
- D: Global tables solve multi-Region replication, disaster recovery, and active-active patterns, not same-Region read-your-write. In the default mode (multi-Region eventual consistency) they add a cross-Region replication window. The optional multi-Region strong consistency mode must be chosen at table creation, runs in exactly three Regions, and adds write latency; it still relies on `ConsistentRead=true` for the read, so it is not the fix here.


---

## Q3

**Skills:** 1.3.2

A web application lets users upload video files up to 5 GB directly from the browser to S3. Users
on unreliable connections report that large uploads frequently fail entirely and must restart from
0%. Which S3 feature/pattern should the application use to fix this?

A. Reduce the maximum allowed file size to 100 MB so uploads complete faster and failures matter less.

B. Use S3 multipart upload, splitting the file into parts that can be uploaded independently (and retried individually) and only assembled into the final object once all parts succeed.

C. Switch the bucket's storage class to S3 Glacier so uploads are billed at a lower rate.

D. Enable S3 Transfer Acceleration only, with no change to how the file is uploaded as a single PUT.

**Correct Answer: B**

**Explanation:** S3 multipart upload is the standard pattern for large objects written from
application code: the client splits the object into parts, uploads them independently (in
parallel or sequentially), and only a failed part needs to be retried, not the whole object. This
directly fixes "uploads frequently fail entirely and restart from 0%". AWS recommends multipart
upload above 100 MB and requires it above 5 GB. See
[1.3.2 Choosing data stores](1.3.2-choosing-data-stores.md#amazon-s3-as-a-data-store).

**Why the others are wrong:**
- A: Shrinking the allowed file size doesn't meet the stated requirement (users need to upload files up to 5 GB) and doesn't fix the underlying reliability problem; it just avoids it for smaller files.
- C: Glacier storage classes are for cold, rarely accessed data with retrieval delays. Choosing a storage class changes storage cost and retrieval time, not upload reliability.
- D: Transfer Acceleration speeds up transfer over AWS's edge network but, without multipart upload, a single-PUT failure still restarts the entire object from scratch. It doesn't solve the "restart from 0%" problem by itself.


---

## Q4

**Skills:** 1.3.2, 1.3.4

An application needs to debit one account and credit another in a single operation that must be
fully atomic (both succeed or both roll back) and also needs complex multi-table SQL joins for
reporting across normalized schemas. Which data store(s) correctly satisfy these requirements, and
why?

A. DynamoDB, using DynamoDB Transactions, which support unlimited items per transaction and native SQL joins.

B. RDS or Aurora, because they provide full ACID transactions and native SQL joins across normalized relational tables. The workload described is fundamentally relational.

C. S3, using versioning to simulate atomic multi-object updates.

D. DynamoDB, because its eventually consistent reads make cross-table joins safe by default.

**Correct Answer: B**

**Explanation:** Task 1.3 expects you to recognize when a scenario describes a genuinely relational
workload: ACID guarantees across related rows and multi-table joins. RDS and Aurora provide full SQL
ACID transactions and native joins. DynamoDB transactions exist but are limited to 100 actions and
4 MB per transaction, and DynamoDB has no SQL join capability. See
[1.3.2 Choosing data stores](1.3.2-choosing-data-stores.md#decision-tree) and
[1.3.4 DynamoDB design and operations](1.3.4-dynamodb-design-and-operations.md#transactions).

**Why the others are wrong:**
- A: DynamoDB Transactions are real but limited to 100 actions (4 MB aggregate) per transaction, and DynamoDB has no SQL join engine at all. "Unlimited items" and "native SQL joins" are both false.
- C: S3 has no multi-object atomic transaction mechanism; versioning only preserves historical object versions, it doesn't make a multi-object update atomic.
- D: Consistency model (eventual vs. strong) is unrelated to whether joins are possible. DynamoDB has no join operator regardless of consistency settings.
