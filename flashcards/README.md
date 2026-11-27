# Anki flashcards

Generated flashcards for the DVA-C03 study material belong in this directory.

## Workflow

1. Generate cards from the repository notes.
2. Review each card and its source reference.
3. Merge the approved cards into one deck JSON file.
4. Validate the JSON before importing it into Anki.
5. Run a read-only duplicate check through AnkiConnect.
6. Import only after reviewing the duplicate report.

Intermediate fragments and review files stay local because `.gitignore` excludes `fragments/` and `review/` directories. Commit only complete, reviewed deck files.

## Mobile access

Import the finished deck into desktop Anki, sync it to AnkiWeb, then sync AnkiMobile or AnkiDroid with the same account.
