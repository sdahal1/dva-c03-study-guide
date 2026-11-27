# AWS Certified Developer - Associate DVA-C03 study guide

One study guide for the DVA-C03 exam, organized exactly like the official exam guide: four domains, twelve tasks, and one note for each of the 72 skills.

> AWS changes exam scope and service limits over time. The [official DVA-C03 exam guide](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html) and current AWS documentation are the final authority.

## Start here

1. Read the [exam guide summary](00-foundations/exam-guide.md) to see what the exam covers.
2. Skim [developer basics](00-foundations/developer-basics.md) if APIs, SDKs, IAM, or DynamoDB keys feel unfamiliar.
3. Work through the domains in order. Each domain README links its tasks, and each task README links its skill notes.
4. Track progress in the [study tracker](exam-study-guide.md). Every checkbox links to its skill note.
5. In the final weeks, switch to [exam prep](exam-prep/README.md): the cheat sheet, two full practice exams, and three exam banks.

## Domains

| Domain | Weight | Tasks |
|---|---:|---|
| [1. Development with AWS Services](domain-1-development-with-aws-services/README.md) | 30% | [1.1 Build application code](domain-1-development-with-aws-services/task-1.1-build-application-code/README.md) · [1.2 Lambda functions](domain-1-development-with-aws-services/task-1.2-develop-lambda-functions/README.md) · [1.3 Data stores](domain-1-development-with-aws-services/task-1.3-use-data-stores/README.md) |
| [2. Security](domain-2-security/README.md) | 26% | [2.1 Authentication and authorization](domain-2-security/task-2.1-authentication-and-authorization/README.md) · [2.2 Encryption and sensitive data](domain-2-security/task-2.2-encrypt-and-protect-sensitive-data/README.md) · [2.3 AI service security](domain-2-security/task-2.3-secure-ai-services/README.md) |
| [3. Testing and Deployment](domain-3-testing-and-deployment/README.md) | 22% | [3.1 Deployable artifacts](domain-3-testing-and-deployment/task-3.1-prepare-deployable-artifacts/README.md) · [3.2 Testing](domain-3-testing-and-deployment/task-3.2-test-applications/README.md) · [3.3 CI/CD deployment](domain-3-testing-and-deployment/task-3.3-deploy-with-ci-cd/README.md) |
| [4. Troubleshooting and Optimization](domain-4-troubleshooting-and-optimization/README.md) | 22% | [4.1 Root-cause analysis](domain-4-troubleshooting-and-optimization/task-4.1-root-cause-analysis/README.md) · [4.2 Observability](domain-4-troubleshooting-and-optimization/task-4.2-observability/README.md) · [4.3 Optimization](domain-4-troubleshooting-and-optimization/task-4.3-optimize-applications/README.md) |

## How the repository is organized

```text
.
├── README.md                     This page
├── exam-study-guide.md           Progress tracker, one checkbox per skill
├── 00-foundations/               Exam guide, developer basics, glossary,
│                                 service basics, in-scope service map
├── domain-1-development-with-aws-services/
│   ├── README.md                 Domain overview, skill map, case studies
│   └── task-1.1-build-application-code/
│       ├── README.md             Task overview and study checklist
│       ├── 1.1.1-coupling-and-decoupling.md
│       ├── ...                   One note per skill
│       ├── labs.md               Hands-on labs for the task
│       └── practice-questions.md Questions with full explanations
├── domain-2-security/
├── domain-3-testing-and-deployment/
├── domain-4-troubleshooting-and-optimization/
├── exam-prep/                    Cheat sheet, strategy, full exams, exam banks
└── flashcards/                   Anki deck files
```

Every skill note follows the same layout: what the exam expects, key terms, the concepts, exam triggers (scenario clue to answer), common traps, a quick-recall list, related skills, and sources. Previous and Next links at the top of each note run through all 72 skills in exam-guide order.

## Study workflow for each task

1. Read the task README for the big picture.
2. Read each skill note. Stop at the exam triggers table and test yourself on the clues.
3. Do the labs. They run against Floci, a free local AWS emulator, so they cost nothing.
4. Answer the practice questions without notes, then read every explanation, including for questions you got right.
5. Log mistakes by skill number in the [study tracker](exam-study-guide.md) and revisit those skills after a day.

The tracker treats a skill as complete when you can explain the mechanism without notes, rule out the usual distractors, finish the related lab, and score at least 80% on related questions twice on different days.

## Flashcards

The [`flashcards/`](flashcards/) folder holds the Anki deck files. Import a deck into desktop Anki, sync to AnkiWeb, then sync AnkiMobile or AnkiDroid with the same account to study on your phone.

## Accuracy

The notes merge four study collections. Where those sources disagreed, or where AWS has changed a limit recently, the notes use the value from current AWS documentation and cite the page. Examples:

- Lambda's maximum timeout is 30 minutes since November 19, 2026. Older material and some exam questions still say 15.
- SQS messages can be up to 1 MiB. Older material says 256 KB.
- DynamoDB transactions support up to 100 actions. Older material says 25.

Services mentioned in the notes but not on the DVA-C03 service list carry a "Not on the DVA-C03 service list" label.

## Sources and privacy

See [`SOURCES.md`](SOURCES.md). This repository includes material adapted from a third-party repository that has no license, so keep the GitHub repository private.
