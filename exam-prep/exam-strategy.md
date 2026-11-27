# DVA-C03 exam strategy

How the AWS Certified Developer - Associate (DVA-C03) exam works, how to spend the 130 minutes, how to read scenario questions, how to keep a mistakes log, and a plan for the final weeks before the exam on **Wednesday, December 30, 2026**.

## Exam format

| Item | Detail |
|---|---|
| Questions | **65**: 50 scored + 15 unscored. The unscored questions are mixed in and not identified, so treat every question as scored |
| Duration | **130 minutes**, about 2 minutes per question |
| Question types | **Multiple choice**: one correct answer and three distractors. **Multiple response**: two or more correct answers among five or more options; you must select every correct answer to get credit |
| Scoring | Scaled score from **100 to 1,000**; **720** to pass |
| Scoring model | **Compensatory**: you pass on the overall score and do not need to pass each domain |
| Guessing | **No penalty.** Unanswered questions count as wrong, so answer everything |
| Score report | Can show broad performance by domain; those classifications are not exact domain scores |
| Cost | **150 USD** |
| Results | Within **5 business days** (often sooner). You do not see which answers were right or wrong |
| Retake | Wait **14 calendar days** after a fail and pay the full fee for each attempt. After a pass, you can't retake the same exam for **2 years** |

During the exam you have no notes, no pen, and no talking. Use the **Flag** feature to mark questions to revisit, and the review screen at the end to jump back to them. The exam's Help feature lists the full names behind short service names such as Amazon SNS.

### Domains and weights

| Domain | Weight of scored content |
|---|---:|
| 1. Development with AWS Services | 30% |
| 2. Security | 26% |
| 3. Testing and Deployment | 22% |
| 4. Troubleshooting and Optimization | 22% |

Domains 1 and 2 together are more than half of the scored content. When time is short, weight review toward them and toward Task 2.3 (AI service security), the newest task.

### DVA-C02 vs DVA-C03

DVA-C02 was available through November 30, 2026. DVA-C03 applies from December 1, 2026. The changes that affect strategy:

- **AI topics are scored.** On DVA-C02, AI security and AI developer-tool topics appeared only as unscored pretest questions. On DVA-C03 they are scored: AI-assisted development (skills 1.1.9, 3.2.5, 3.3.9, 4.1.5, 4.3.6), managed AI integration (1.1.10), and all of Task 2.3.
- **Weights moved.** Development dropped from 32% to 30%, Testing and Deployment from 24% to 22%, and Troubleshooting and Optimization rose from 18% to 22%.
- **Service list changed.** DVA-C03 adds Amazon Bedrock, Bedrock AgentCore, Kiro, Amazon Quick, AWS PrivateLink, ACM, AWS Private CA, Amazon Data Firehose, and S3 Glacier. AWS AppSync and Amazon Q Developer are no longer listed, and AWS Shield is listed as out of scope. Questions in older material that center on those services carry a "Not on the DVA-C03 service list" label in this folder.
- **Material moved, not removed.** SAM unit testing moved to Domain 3, Lambda performance tuning to Domain 4, and API Gateway stages and custom domains to Task 3.1.

### What the exam does not expect

The target candidate has about a year of hands-on development on AWS. The exam does not ask you to design distributed systems or CI/CD pipelines from scratch, administer IAM users and groups, administer servers, design VPC networks, select foundation models, engineer prompts, or design retrieval-augmented generation (RAG) architectures. It does test the developer-level use of those services, for example deploying through an existing pipeline. If an answer requires an architecture redesign, it is rarely the intended one.

## Time management

130 minutes for 65 questions is 2 minutes per question. Plan two passes.

1. **First pass, about 95 minutes.** Answer every question. If a question takes more than about 2 minutes, choose your best answer, flag it, and move on. Never leave a question blank, even when you flag it.
2. **Checkpoints.** Aim for question 20 at about 30 minutes, question 40 at about 60 minutes, and question 65 at about 95 minutes. If you are behind at a checkpoint, flag faster.
3. **Second pass, about 30 minutes.** Return to flagged questions only. Change an answer only when you find a specific reason (a word you misread, a fact you now recall), not on a vague feeling.
4. **Last 5 minutes.** Confirm that no question is unanswered and that every multiple-response question has the required number of selections.

Long scenario questions often hide a short question. Reading the last sentence first keeps them from eating the clock.

## Reading a question

- **Read the last line first.** Find the real ask and its qualifier: "LEAST operational effort", "MOST cost-effective", "MOST secure", "without code changes", "cannot", "Choose TWO".
- **Least effort or easiest** usually means the most managed option: a built-in feature or configuration change instead of custom code.
- **Every word must match.** "At rest" rules out TLS. "Without code changes" rules out refactoring. "Real time" rules out batch. "Must never see plaintext" rules out every server-side option.
- **Eliminate first.** Most questions are scenarios. Cross out answers you know are wrong, then pick the best of what is left. For Choose TWO or THREE, elimination matters even more because partial selections score as wrong.
- **Feasible but complicated is probably wrong.** If an option works only after building something the managed service already does, it is a distractor.
- **Don't overthink.** There are very few trick questions. The intended answer is usually the documented pattern for the clue in the stem.
- **Watch for keywords that point to one service or feature.** "Serverless SQL on S3" points to Athena; "rotate RDS credentials" points to Secrets Manager. The [cheat sheet](cheat-sheet.md) is built around these clue-to-answer pairs.
- **Watch for stale numbers in answer options.** Some limits changed recently (SQS message size, API Gateway timeout increases, the Lambda timeout). The cheat sheet's numbers tables list current values.

## Mistakes log

A mistakes log stops the same traps from catching you twice. Keep one entry per wrong answer and per lucky guess, from every practice source.

| Date | Source | Skill | Clue I missed | Right answer | Why my choice was wrong | Type | Status |
|---|---|---|---|---|---|---|---|
| Dec 12 | Bank 2, Q60 | 4.1.1 | Synchronous call longer than 29 s returns 504 | Make the work asynchronous | Thought raising the Lambda timeout fixes it | Knowledge gap | Open |

How to fill it in:

- **Source and skill.** Note the file and question number, and take the skill number from the question's study link. Every answer in the [full mock exams](full-exams/index.md) and the [exam banks](exam-banks/README.md) links to a skill note.
- **Clue.** Write the words in the stem that should have pointed you to the right answer.
- **Why my choice was wrong.** Name the distractor's flaw in one line. The "why the others are wrong" bullets in each explanation are usually where the misconception lives.
- **Type.** Use one of four labels so patterns show up: *knowledge gap* (didn't know the fact), *misread qualifier* (missed LEAST, MOST, or "cannot"), *distractor* (knew the fact but chose a plausible wrong option), or *time* (rushed).
- **Status.** Re-answer the question after at least one day. Close the entry once you get it right on two different days.

How to use it:

- **After every practice set,** sort the log by skill. Any skill with three or more open entries gets a reread of its skill note and, if it has one, a run of the task's lab.
- **Weekly,** count entries by type. Many *misread qualifier* entries mean you should slow down on the last line; many *distractor* entries mean you should read every explanation, not only those for questions you missed.
- **Turn repeated misses into flashcards** in the [flashcards](../flashcards/README.md) deck so spaced repetition keeps them fresh.
- **In the final days,** read only the "Clue" and "Right answer" columns. That is your personal cheat sheet.

The root [study guide](../exam-study-guide.md) has a progress dashboard; record weak skills there as well. Its mastery standard is scoring at least 80% twice, on different days, on a task's practice questions.

## Practice material, in order

| Stage | Material | Mode |
|---|---|---|
| While studying each task | The task folder's `practice-questions.md`, for example [Task 2.3 practice questions](../domain-2-security/task-2.3-secure-ai-services/practice-questions.md) | Study the task first, then answer before looking |
| After all four domains | [Exam banks 1 to 3](exam-banks/README.md), 195 questions | Study mode: answer, then read every explanation |
| Final weeks | [Full mock exams 1 and 2](full-exams/index.md), 130 questions | Timed, 130 minutes, answer key only at the end |
| Night before and exam morning | [Cheat sheet](cheat-sheet.md) and your mistakes log | Read once, top to bottom |

Tips that apply to every stage:

- **Answer before you look.** Cover the explanation, commit to an answer, then check.
- **Read every explanation,** including for questions you got right. The wrong-option bullets often contain a second fact, such as a limit or a common distractor.
- **Follow the skill link on a miss** instead of rereading a whole domain.
- **Treat multiple-response questions strictly.** One of two correct selections is a wrong answer.
- **Use about 72% (47 of 65) on a mock exam as a readiness proxy.** AWS does not publish how raw scores map to the scaled 720, so this is an approximation.
- **Verify disputed explanations** against current AWS documentation. Practice questions are prompts, not authority.

### Hands-on practice

Recognition questions go faster when you have built the pattern yourself. If you have time before the final weeks, take a small application and deploy it on EC2, then on Elastic Beanstalk with scaling, then through a CI/CD pipeline. Decouple a component with SQS or SNS, and move part of it to Lambda. Write a few CLI or SDK scripts, for example stopping EC2 instances at night and starting them in the morning. Each task folder's `labs.md` has guided versions.

### Service FAQs

AWS service FAQs cover much of what the exam asks and confirm your understanding. If you read any, prioritize Lambda, DynamoDB, API Gateway, SQS, SNS, Kinesis, KMS, Cognito, CloudFormation, CodeDeploy, and Amazon Bedrock. Example: the [AWS Lambda FAQs](https://aws.amazon.com/lambda/faqs/).

## Final weeks plan

The dates assume the exam on Wednesday, December 30, 2026. Shift days around the holidays as needed; keep the order.

### Three to two weeks out (December 9 to 22)

- Finish any task still below the 80% mastery standard.
- Work through [bank 1](exam-banks/bank-1.md), [bank 2](exam-banks/bank-2.md), and [bank 3](exam-banks/bank-3.md) in study mode, about one bank every four days. Log every miss and guess.
- After each bank, reread the skill notes behind your most frequent mistakes-log skills.
- Re-answer the bank questions you missed, at least a day later.

### Final week (December 23 to 29)

| Day | Date | Plan |
|---|---|---|
| -7 | Wed, Dec 23 | Take [mock exam 1](full-exams/exam-1-questions.md) under a 130-minute timer. Record the raw score before reading the answer key, then log every miss and guess |
| -6 | Thu, Dec 24 | Review mock exam 1 explanations. Reread the skill notes for your three weakest skills |
| -5 | Fri, Dec 25 | Light day: read the Domain 1 and Domain 2 sections of the cheat sheet; review due flashcards |
| -4 | Sat, Dec 26 | Take [mock exam 2](full-exams/exam-2-questions.md) under a timer. Score, then log |
| -3 | Sun, Dec 27 | Review mock exam 2. Compare both exams by skill and by mistake type, then re-drill the bank questions for the weakest skills |
| -2 | Mon, Dec 28 | Re-answer every open mistakes-log entry. Read the whole cheat sheet once |
| -1 | Tue, Dec 29 | No new material. Read the mistakes log's "Clue" and "Right answer" columns and the cheat sheet's numbers tables. Check the exam appointment, ID requirements, and, for online proctoring, the system check. Sleep |

If both mock exams land well below the 72% proxy, consider rescheduling rather than cramming. The retake rule (14 days and a full fee) makes a failed attempt more expensive than a short delay.

### Exam day (Wednesday, December 30)

- Read the [cheat sheet](cheat-sheet.md) once, top to bottom, in about 30 minutes. Don't stop to look things up.
- Arrive or check in early with the identification your appointment confirmation asks for.
- During the exam, follow the two-pass plan above: answer everything, flag anything slow, and use the last 5 minutes to confirm nothing is blank.

## Sources

- Merged from: `archive/aws-dva-notes/Markdown Files/29. Exam Review & Tips.md`, `archive/dva-c03/practice-questions/index.md`, and the study-workflow sections of `archive/dva-c03/README.md`, `archive/aws-dva-notes/README.md`, `archive/dva-c03-study-notes/README.md`, and `archive/dva-c03/exam-bank/README.md` (plain text, not links). Exam technique lines from `archive/aws-dva-notes/Markdown Files/31. Exam Day Cheat Sheet.md` moved here.
- [AWS Certified Developer - Associate (DVA-C03) exam guide](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html): question types, scoring, domains, target candidate, and out-of-scope tasks.
- [DVA-C02 and DVA-C03 comparison](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-comparison.html): weight changes, added content, moved content.
- [DVA-C03 out-of-scope services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-out-of-scope-services.html): AWS Shield Standard and Advanced listed as out of scope.
- [Mentions of AWS services on the exam](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/dva-03-service-mentions.html): short service names and the in-exam Help feature.
- [AWS Certification FAQs](https://aws.amazon.com/certification/faqs/): 14-day retake wait, full fee per attempt, 2-year wait after passing.
- [AWS Certification after-testing policies](https://aws.amazon.com/certification/policies/after-testing/): results within 5 business days.
- [AWS Certified Developer - Associate](https://aws.amazon.com/certification/certified-developer-associate/): 150 USD exam fee.
