# DVA-C03 Full-Length Mock Exams

Two brand-new, **full-length mock exams** for the AWS Certified Developer – Associate (DVA-C03)
exam, built to match the *actual exam format* — not the domain-split practice banks in
[`../`](../). Each mock exam mixes all four domains together in one sitting, exactly like the real
test, instead of grouping questions by domain.

These are entirely separate from, and do not duplicate, the 50-question domain-split bank in
[`../domain-1-questions.md`](../domain-1-questions.md), [`../domain-2-questions.md`](../domain-2-questions.md),
[`../domain-3-questions.md`](../domain-3-questions.md), and [`../domain-4-questions.md`](../domain-4-questions.md).
Use that bank to study one domain at a time; use these two exams to simulate exam day.

## Files

| File | Contents |
|---|---|
| [`exam-1-questions.md`](exam-1-questions.md) | Mock Exam 1 — 65 questions, A–D/A–E options, **no answers shown** |
| [`exam-1-answer-key.md`](exam-1-answer-key.md) | Mock Exam 1 — correct answer(s), full explanation, why every distractor is wrong, study links |
| [`exam-2-questions.md`](exam-2-questions.md) | Mock Exam 2 — 65 **different** questions, same format |
| [`exam-2-answer-key.md`](exam-2-answer-key.md) | Mock Exam 2 — matching answer key |

## Format — matches the real DVA-C03 exam

- **65 total questions** per exam: 50 scored + 15 unscored (unidentified) questions, mixed together
  exactly as they are on the real test — you can't tell which 15 are unscored, so treat every
  question as if it counts.
- **Question types**: standard multiple choice (1 correct answer, 3–4 distractors) and multiple
  response (2 correct answers out of 5+ options), explicitly labeled **"Choose TWO"** in the
  question stem wherever it applies. Each exam has 5 Choose-TWO questions.
- **Domain weighting** (questions are shuffled/interleaved across domains, not grouped):

  | Domain | Weight | Questions per exam |
  |---|---|---|
  | 1 — Development with AWS Services | 30% | 20 |
  | 2 — Security (includes Task 2.3 AI service security) | 26% | 17 |
  | 3 — Testing and Deployment | 22% | 14 |
  | 4 — Troubleshooting and Optimization | 22% | 14 |
  | **Total** | **100%** | **65** |

- Every question is a realistic, scenario-based developer question — an application/requirement
  description, frequently with a code snippet, IAM policy, CLI command, or API/SDK detail, followed
  by a clear question stem and 4–5 plausible options. Domain 2's Task 2.3 content (the newest
  DVA-C03 task statement — AI service security for Bedrock/Amazon Q) appears in both exams.

## How to take a mock exam

1. **Time yourself at 130 minutes** — the real DVA-C03 exam allows 130 minutes for 65 questions.
   Set a timer before you open `exam-1-questions.md` (or `exam-2-questions.md`) and stop writing
   answers when it goes off, exactly like exam day.
2. **Don't peek at the answer key until you're done.** Answer every question in the questions file
   first — including a best guess on anything you're unsure of (there's no penalty for guessing on
   the real exam either) — then, and only then, open the matching `-answer-key.md` file to grade
   yourself.
3. **Record your raw score** (number correct out of 65) before reading a single explanation, so you
   get an honest read on where you'd actually land.
4. **Passing bar: 720/1000 (scaled score), commonly treated as roughly 72%.** AWS scores this exam
   on a 100–1000 scale, not a raw percentage, and the exact scoring curve isn't published — but
   72% correct on a practice exam (about 47/65) is the standard approximation used to gauge
   readiness against the real 720 cut score. Treat it as a useful proxy, not a guarantee of your
   real scaled score.
5. **Review every explanation, including ones you got right.** The "why the others are wrong"
   section under each question often contains a second fact worth knowing — a service limit, a
   real exam distractor pattern, or an edge case — not just confirmation that you picked correctly.
6. **Follow the study links on a miss.** Every answer links back to the specific `services/` doc or
   `domain-N-.../` material the question is drawn from — if a question stumps you, that's the
   fastest path back to the source material.
7. **Take both exams, ideally a few days apart**, and compare where you miss points — a pattern
   across both exams (e.g. always missing Task 2.3 AI-security questions, or always missing
   Choose-TWO questions) is more informative than either exam's single raw score.
8. **Use these after**, not instead of, the domain-split bank in [`../`](../) and the domain study
   material in `certifications/dva-c03/domain-N-.../README.md` — these exams test whether you can
   apply that material under real exam conditions (mixed domains, time pressure), they aren't the
   primary way to learn the content.

## Status

✅ Complete — 2 full 65-question mock exams (130 questions total, no overlap between the two),
each with a complete answer key, matching the real DVA-C03 format and domain weighting.
