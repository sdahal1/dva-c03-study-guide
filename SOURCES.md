# Sources

This guide merges four study collections into one structure. The originals remain on the local machine in `archive/`, which Git ignores, so they are not part of the GitHub repository.

## Authority

Use these to settle scope and technical questions:

- [AWS Certified Developer - Associate](https://aws.amazon.com/certification/certified-developer-associate/)
- [DVA-C03 exam guide](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html)
- [AWS documentation](https://docs.aws.amazon.com/)

AWS owns its service names, certification marks, documentation, and diagrams. Notes link to AWS pages and embed some AWS documentation images with source captions.

## Merged collections

| Collection | What it contributed | Rights status |
|---|---|---|
| DVA-C03 study notes (`archive/dva-c03-study-notes/`) | Concise notes by skill number, glossary, cheat sheet | No license or provenance record in the workspace |
| DVA-C03 detailed guides (`archive/dva-c03/`) | Deep dives, worked scenarios, labs, practice questions, full exams, exam banks | No license or provenance record in the workspace |
| [yashsinghviwork/aws-dva-notes](https://github.com/yashsinghviwork/aws-dva-notes) (`archive/aws-dva-notes/`) | Service-by-service notes, exam tips and traps, DVA-C03 AI chapter, exam-day cheat sheet | Public repository with no license, so the author keeps all rights |
| Official exam guide summary | `00-foundations/exam-guide.md` and the study tracker | Paraphrased from AWS |

Each skill note's Sources section names the archived files it merged, as plain text, and links the AWS pages used to settle conflicts.

## Not merged

- `udemy-course-notes/` stays separate and git-ignored. It holds notes from a paid course still in progress.
- Duplicate PDF exports of the service notes were not carried over.
- `flashcards/` was left unchanged.

## Keep the repository private

Material from `aws-dva-notes` is adapted throughout the notes, and its source repository grants no license. Private personal study use is reasonable. Before making this repository public, get the author's permission or rewrite the affected material, and confirm the origin of the two `dva-c03` collections.
