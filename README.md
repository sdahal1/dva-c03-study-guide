# AWS Certified Developer - Associate DVA-C03 study workspace

Study notes, labs, practice questions, and Anki material organized around the AWS Certified Developer - Associate DVA-C03 exam guide.

> AWS changes exam scope and service behavior over time. Use the official AWS exam guide and current AWS documentation as the final authority.

## Start here

| Goal | Open |
|---|---|
| Follow the complete study plan | [`exam-study-guide.md`](exam-study-guide.md) |
| Check the exam blueprint | [`exam-guide.md`](exam-guide.md) |
| Read concise domain notes | [`dva-c03-study-notes/`](dva-c03-study-notes/) |
| Study worked scenarios and service details | [`dva-c03/`](dva-c03/) |
| Review practice questions | [`dva-c03/practice-questions/`](dva-c03/practice-questions/) |
| Work through hands-on labs | Domain `labs.md` files under [`dva-c03/`](dva-c03/) |
| Use the Anki material | [`flashcards/`](flashcards/) |

New learners should start with the [developer basics](dva-c03-study-notes/00-developer-basics.md), then work through the four domain notes in order. Use the longer domain folders for concepts that need examples or hands-on practice.

## Exam outline

| Domain | Weight | Primary notes | Detailed material |
|---|---:|---|---|
| Development with AWS Services | 30% | [Domain 1 notes](dva-c03-study-notes/01-development-with-aws-services.md) | [Domain 1 folder](dva-c03/domain-1-development/) |
| Security | 26% | [Domain 2 notes](dva-c03-study-notes/02-security.md) | [Domain 2 folder](dva-c03/domain-2-security/) |
| Testing and Deployment | 22% | [Domain 3 notes](dva-c03-study-notes/03-testing-deployment.md) | [Domain 3 folder](dva-c03/domain-3-testing-deployment/) |
| Troubleshooting and Optimization | 22% | [Domain 4 notes](dva-c03-study-notes/04-troubleshooting-optimization.md) | [Domain 4 folder](dva-c03/domain-4-troubleshooting-optimization/) |

## Suggested study workflow

1. Read the matching concise domain note.
2. Use the detailed domain folder for scenarios, diagrams, and service comparisons.
3. Complete the domain labs.
4. Answer the domain practice questions without notes.
5. Record missed concepts by official skill number in [`exam-study-guide.md`](exam-study-guide.md).
6. Review the [cheat sheet](dva-c03-study-notes/05-cheat-sheet.md) and due Anki cards.

The study guide defines mastery as explaining the mechanism, distinguishing common alternatives, completing the related lab, and scoring at least 80% twice on different days.

## Repository map

```text
.
├── README.md                    GitHub landing page
├── exam-guide.md                Clean exam blueprint and scope
├── exam-study-guide.md          Study sequence and progress tracker
├── dva-c03-study-notes/         Concise domain-organized notes
├── dva-c03/                     Detailed guides, labs, and questions
└── flashcards/                  Anki deck source files
```

The local workspace also contains imported reference notes and course-derived notes. `.gitignore` keeps those directories out of the GitHub repository until their redistribution rights are confirmed. It also excludes duplicate PDF exports and intermediate flashcard fragments.

## Flashcards

The [`flashcards/`](flashcards/) directory contains source files for Anki decks. Generate and validate a complete deck before importing it into desktop Anki. After import:

1. Sync desktop Anki with an AnkiWeb account.
2. Install AnkiMobile on iPhone or iPad, or AnkiDroid on Android.
3. Sign in with the same account and sync the mobile app.

Sync before changing devices so Anki can preserve review history and scheduling.

## Reading the notes

GitHub renders the Markdown files, tables, and Mermaid diagrams in a browser. To work locally, use GitHub's **Code** button to copy the clone URL, clone the repository, and open the directory as an [Obsidian](https://obsidian.md/) vault.

## Sources and accuracy

The workspace follows this source order when materials disagree:

1. Official AWS exam guide and current AWS documentation
2. [`dva-c03-study-notes/`](dva-c03-study-notes/)
3. Detailed scenarios and practice material under [`dva-c03/`](dva-c03/)

Some notes summarize service behavior and quotas that AWS can change. Check current AWS documentation before relying on a number or limit. See [`SOURCES.md`](SOURCES.md) for publishing and attribution notes.

## License

No open-source license has been applied. Original material remains all rights reserved, and third-party material remains subject to its owners' terms. Resolve the source and licensing checklist in [`SOURCES.md`](SOURCES.md) before publishing additional imported material.
