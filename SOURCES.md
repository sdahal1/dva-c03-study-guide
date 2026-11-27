# Sources and publishing notes

This file records which workspace material can enter a public GitHub repository and which material needs a rights check first.

## Primary authority

Use these sources to verify exam scope and technical claims:

- [AWS Certified Developer - Associate](https://aws.amazon.com/certification/certified-developer-associate/)
- [DVA-C03 exam guide](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html)
- [AWS documentation](https://docs.aws.amazon.com/)

AWS owns its service names, certification marks, documentation, diagrams, and badge artwork. Linking to AWS material does not transfer ownership or grant redistribution rights.

## Included collections

### `dva-c03-study-notes/`

Concise notes organized by the DVA-C03 domains. The workspace does not contain a separate license or provenance record for this collection. Confirm authorship before applying an open-source license.

### `dva-c03/`

Detailed domain guides, labs, practice questions, and exam banks. The workspace does not contain a separate license or provenance record for this collection. Confirm authorship and the rights to any embedded diagrams or copied questions before public publication.

### `flashcards/`

Generated study-card source files derived from repository notes. Keep traceable source references in each card. Do not publish cards based on material that the repository cannot redistribute.

## Excluded local collections

The root `.gitignore` excludes these directories from Git:

### `aws-dva-notes/`

This directory mirrors material associated with [`yashsinghviwork/aws-dva-notes`](https://github.com/yashsinghviwork/aws-dva-notes). The source repository does not expose a license in the local copy. Attribution alone does not grant redistribution rights, so keep the mirror out of this repository unless its owner provides a license or permission.

### `udemy-course-notes/`

These notes derive from a paid course. Keep them local unless the course owner permits redistribution. Paraphrasing reduces copying but does not settle every copyright or contractual issue.

### PDF exports

The PDF files duplicate Markdown content and increase repository size. `.gitignore` excludes directories named `PDF Files`.

## Before making the repository public

- Confirm authorship and licensing for `dva-c03/` and `dva-c03-study-notes/`.
- Remove or replace AWS badge artwork and diagrams if their reuse terms do not allow publication.
- Check practice questions for copied exam or commercial question-bank content.
- Scan Git history and files for credentials, account IDs, email addresses, and personal data.
- Check links and time-sensitive service limits against current AWS documentation.
- Choose a license only for material you have the right to license.

Until those checks are complete, use a private GitHub repository.
