# DVA-C03 study-mode exam banks (3 x 65 questions)

Three full-length, 65-question practice exams in study mode. The correct answer and a full explanation, including why each wrong option is wrong, sit directly under each question. Use these banks for drilling across the whole exam, not for simulating exam conditions.

## Where the banks fit

| | Exam banks (this folder) | [Full mock exams](../full-exams/index.md) | Task practice questions |
|---|---|---|---|
| **Purpose** | Drill all domains with immediate feedback | Simulate the real exam | Check one task right after studying it |
| **Format** | Answer and explanation under each question | Questions only; answers in a separate answer key | Answer and explanation under each question |
| **Size** | 3 banks x 65 questions (195 total), domain-weighted and interleaved | 2 exams x 65 questions | A few questions per task, in each task folder's `practice-questions.md` |
| **When to use** | After you finish all four domains, before the timed exams | Final weeks, under a 130-minute timer | While working through a task |
| **Visuals** | Mermaid diagrams (15 to 16 per bank) and a few AWS documentation images with source credit | Text only | Varies by task |

Do not confuse the banks with the full mock exams. Reading an explanation before answering turns a bank question into a study session, which is fine here. In the full mock exams, cover the answer key, time yourself, and check only at the end.

## What each bank covers

Each bank ([`bank-1.md`](bank-1.md), [`bank-2.md`](bank-2.md), [`bank-3.md`](bank-3.md)) is a complete 65-question exam that follows the DVA-C03 blueprint:

- **Domain mix,** interleaved throughout:
  - Domain 1, Development with AWS Services: 20 questions (30.8%)
  - Domain 2, Security: 17 questions (26.2%), including several on Task 2.3 (Amazon Bedrock access control, Bedrock Guardrails, AI agent least privilege, and personally identifiable information (PII) in logs and retrieval data)
  - Domain 3, Testing and Deployment: 14 questions (21.5%)
  - Domain 4, Troubleshooting and Optimization: 14 questions (21.5%)
- **Choose TWO questions:** 3 per bank, 9 in total.
- **Developer-level detail:** IAM policy JSON, CLI commands, CloudFormation and AWS Serverless Application Model (AWS SAM) snippets, SDK calls, and `buildspec.yml` and `appspec.yml` fragments appear where the real exam would expect you to reason about them.
- **Distinct scenarios:** the three banks explore the same domains from different scenarios instead of repeating questions.
- **Skill links:** each explanation ends with a "See" link to one or two skill notes, for example [2.1.6 Least-privilege permissions](../../domain-2-security/task-2.1-authentication-and-authorization/2.1.6-least-privilege-permissions.md). Use that skill number when you log a miss.
- **Out-of-scope items:** a few questions cover services that are not on the DVA-C03 service list (AWS AppSync, AWS Shield, Amazon Comprehend, Amazon CodeWhisperer, AWS Config, Amazon Macie, Amazon GuardDuty, Amazon Inspector, AWS Security Hub, AWS Trusted Advisor, AWS Compute Optimizer, AWS Batch, AWS CloudHSM, AWS Fault Injection Service). They carry a "Not on the DVA-C03 service list" label. Review in-scope material first.

## Suggested order

1. **Bank 1.** Read each question, commit to an answer, then read the full explanation, including every wrong option. When a topic feels shaky, follow the skill link and close the gap instead of memorizing the letter.
2. **Bank 2.** Same process. You will see familiar concepts (Lambda cold starts, IAM evaluation logic, rollback strategies) from new angles. That overlap is intentional.
3. **Bank 3.** Same again. If you now answer most questions correctly before reading the explanation, you are close to ready.
4. **Targeted re-drill.** Reopen each bank and answer only the questions you missed, plus any you guessed. Reread the "why the others are wrong" bullets; the misconception usually lives there.
5. **Move to the timed exams.** When you score well across all three banks without reading ahead, take [mock exam 1](../full-exams/exam-1-questions.md) and then mock exam 2 under real conditions: 65 questions, 130 minutes, answer key only at the end.

The [exam strategy](../exam-strategy.md) guide describes the mistakes log and the final-week plan that these steps feed into.

## Files

| File | Questions | Mermaid diagrams | AWS documentation images |
|---|---|---|---|
| [`bank-1.md`](bank-1.md) | 65 | 15 | 1 |
| [`bank-2.md`](bank-2.md) | 65 | 16 | 2 |
| [`bank-3.md`](bank-3.md) | 65 | 16 | 2 |

All three files share one structure: a short header with the domain mix, then `## Q1` to `## Q65`, each followed by **Correct Answer**, **Explanation**, and **Why the others are wrong**. Diagrams sit inside the explanation, with a prose walk-through below each one. Image-based questions credit the AWS documentation page the image came from.
