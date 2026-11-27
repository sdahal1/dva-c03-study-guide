# DVA-C03 full-length mock exams

Two timed, full-length mock exams for the AWS Certified Developer - Associate (DVA-C03) exam. Each exam mixes all four domains in one sitting, like the real test. Questions and answers live in separate files so you can take the exam blind and grade it afterward.

Use these last, under exam conditions. For learning one task at a time, use the `practice-questions.md` file in each task folder (for example, [Task 1.1 practice questions](../../domain-1-development-with-aws-services/task-1.1-build-application-code/practice-questions.md)). For full-scope drilling with answers shown inline, use the [study-mode exam banks](../exam-banks/README.md). The [exam-prep overview](../README.md) explains when to use each set.

## Files

| File | Contents |
|---|---|
| [`exam-1-questions.md`](exam-1-questions.md) | Mock exam 1: 65 questions, options A to D (A to E for multiple response), no answers shown |
| [`exam-1-answer-key.md`](exam-1-answer-key.md) | Mock exam 1: correct answers, explanations, why each distractor is wrong, and links to the matching skill notes |
| [`exam-2-questions.md`](exam-2-questions.md) | Mock exam 2: 65 different questions, same format |
| [`exam-2-answer-key.md`](exam-2-answer-key.md) | Mock exam 2: matching answer key |

## Format

- **65 questions per exam.** The real exam has 50 scored and 15 unscored questions that are not identified, so treat every question as if it counts.
- **Question types:** multiple choice (one correct answer) and multiple response (two correct answers out of five options), labeled **Choose TWO** in the stem. Each exam has 5 Choose TWO questions.
- **Domain mix,** interleaved rather than grouped:

  | Domain | Weight | Questions per exam |
  |---|---|---|
  | 1. Development with AWS Services | 30% | 20 |
  | 2. Security (includes Task 2.3, AI service security) | 26% | 17 |
  | 3. Testing and Deployment | 22% | 14 |
  | 4. Troubleshooting and Optimization | 22% | 14 |
  | **Total** | **100%** | **65** |

- **Scenario style:** most questions describe an application and a requirement, often with a code snippet, IAM policy, CLI command, or SDK detail, followed by four or five plausible options.
- **Skill links:** every answer in the answer keys links to one or two skill notes, for example [1.3.4 DynamoDB design and operations](../../domain-1-development-with-aws-services/task-1.3-use-data-stores/1.3.4-dynamodb-design-and-operations.md). Record misses by that skill number.
- **Out-of-scope items:** a few questions cover services that are not on the DVA-C03 service list (AWS AppSync, AWS Cloud9, AWS Shield, AWS Config, Amazon Macie, Amazon GuardDuty, Amazon Inspector, AWS Security Hub, AWS Trusted Advisor, AWS Cost Explorer, AWS Budgets). Those questions carry a "Not on the DVA-C03 service list" label. They are kept for background; review in-scope material first.

## How to take a mock exam

1. **Set a 130-minute timer** before opening the questions file. Stop answering when it goes off. The [exam strategy](../exam-strategy.md) guide covers pacing: about 2 minutes per question, with a flag-and-return pass.
2. **Answer every question before opening the answer key,** including a best guess on anything uncertain. The real exam has no penalty for guessing, and unanswered questions count as wrong.
3. **Record your raw score** (correct out of 65) before reading any explanation.
4. **Use about 72% (47 of 65) as a readiness proxy.** AWS reports a scaled score from 100 to 1,000 with 720 to pass and does not publish the conversion, so a raw percentage is only an approximation.
5. **Read every explanation, including the ones you got right.** The "why the others are wrong" bullets often hold a second fact, such as a limit or a common distractor.
6. **Log each miss and each lucky guess** in your mistakes log with the skill number from the study link, the clue you missed, and the reason the distractor fooled you. See [the mistakes-log method](../exam-strategy.md#mistakes-log).
7. **Take the two exams a few days apart** and compare where you lose points. A pattern across both (for example, Task 2.3 questions or Choose TWO questions) tells you more than either raw score.

## Accuracy notes

Explanations were checked against the accuracy rules used across this guide. Quotas change, so confirm any number you plan to rely on in current AWS documentation. The [cheat sheet](../cheat-sheet.md) lists the current values for the limits that most often appear in older practice material.
