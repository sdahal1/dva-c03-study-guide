# DVA-C03 Exam Bank — Study Mode (3 × 65 = 195 Questions)

This folder is a **study-mode** question bank: three full-length (65-question) practice exams where
the correct answer and a complete explanation — including *why every wrong option is wrong* — sit
immediately after each question. It exists for **drilling**, not for simulating exam conditions.

## Study mode vs. the timed full exams

| | `exam-bank/` (this folder) | [`../practice-questions/full-exams/`](../practice-questions/full-exams/) | [`../practice-questions/`](../practice-questions/) (domain banks) |
|---|---|---|---|
| **Purpose** | Learn/drill: see the answer and reasoning right away | Simulate the real exam experience | Drill one domain at a time, lightly |
| **Format** | Answer + full explanation (right *and* wrong options) directly under each question | Questions only; answers live in a separate `*-answer-key.md` file | Answer + explanation under each question, same as this folder |
| **Size** | 3 exams × 65 questions (195 total), domain-weighted and interleaved | 2 exams × 65 questions | 4 files × ~11–15 questions, one per domain |
| **When to use** | Day-to-day study: read a question, try to answer, immediately check your reasoning against the explanation | Final rehearsal: time yourself at ~90 seconds/question, no peeking, score yourself against the key afterward | Quick, focused review of a single weak domain |
| **Visuals** | Mermaid diagrams (~15–16 per bank) for flows that benefit from a picture (IAM policy evaluation, CI/CD pipeline stages, state machine logic, service maps); a few questions also embed real, curl-verified AWS documentation screenshots with source credit | Text only | Text only |

**Don't confuse the two blind full exams in `full-exams/` with these three banks.** If you open a
bank file and read the explanation under a question before attempting it, you've turned it into a
study session, not a test — that's fine *here*, but do the opposite in `full-exams/`: cover the
answer key, time yourself, and only check afterward.

## What's covered

Each of the three banks ([`bank-1.md`](bank-1.md), [`bank-2.md`](bank-2.md), [`bank-3.md`](bank-3.md))
is a complete, independently-usable 65-question exam matching the real DVA-C03 blueprint:

- **65 questions** = 50 scored + 15 unscored on the real exam (you won't be told which are which on
  exam day, which is why every question here is written and explained to the same standard).
- **Domain weighting**, interleaved throughout rather than grouped by domain:
  - Domain 1 — Development with AWS Services: **20 questions (30.8%)**
  - Domain 2 — Security: **17 questions (26.2%)** — including several questions on Task 2.3
    (security of AI/ML services: Bedrock access control, Guardrails, AI agent least privilege,
    PII in logs/RAG data perimeters), the newest task statement on the exam
  - Domain 3 — Testing and Deployment: **14 questions (21.5%)**
  - Domain 4 — Troubleshooting and Optimization: **14 questions (21.5%)**
- A small number of **"Choose TWO"** multi-select questions per bank (3 each, 9 total), matching the
  real exam's mix of single- and multi-answer questions.
- **Developer-focused detail throughout**: IAM policy JSON, CLI commands, CloudFormation/SAM
  snippets, boto3/SDK call signatures, and buildspec/appspec fragments appear wherever the real
  exam would expect you to reason about them, not just name a service.
- **195 questions, all mutually distinct** — distinct across all three banks, and distinct from
  every question in [`../practice-questions/`](../practice-questions/) and
  [`../practice-questions/full-exams/`](../practice-questions/full-exams/). No scenario, correct
  answer, or core concept repeats across any of the five question sets in this certification folder.

## Suggested usage order

1. **First pass — `bank-1.md`.** Read each question, commit to an answer, then read the full
   explanation (including every wrong option) before moving on. Follow the cross-links into
   [`../domain-1-development/`](../domain-1-development/) through
   [`../domain-4-troubleshooting-optimization/`](../domain-4-troubleshooting-optimization/) and
   [`../../../services/`](../../../services/) whenever a topic feels shaky — don't just memorize the
   answer letter, close the underlying knowledge gap.
2. **Second pass — `bank-2.md`.** Same drilling process. By now you should notice overlapping
   *concepts* (e.g. Lambda cold starts, IAM evaluation logic, CI/CD rollback strategies) explored
   from new angles and services — that's intentional; the three banks are built to reinforce the
   same domains from different scenarios rather than repeat questions.
3. **Third pass — `bank-3.md`.** Same again. If you're now answering most questions correctly
   *before* reading the explanation, you're close to exam-ready.
4. **Targeted re-drill.** Re-open any bank and re-answer only the questions you got wrong the first
   time (search for the service/topic name). Re-read the "why the others are wrong" bullets for those
   — the wrong-option reasoning is usually where the real misconception lives.
5. **Graduate to the timed exams.** Once you're consistently scoring well across all three banks
   without reading ahead, move to [`../practice-questions/full-exams/exam-1-questions.md`](../practice-questions/full-exams/exam-1-questions.md)
   and `exam-2-questions.md` under real exam conditions (65 questions, ~90 seconds each, no
   explanations visible until you check the answer key afterward) to validate your readiness and
   practice pacing.

## Files

| File | Questions | Mermaid diagrams | Real AWS doc images |
|---|---|---|---|
| [`bank-1.md`](bank-1.md) | 65 | 15 | 1 |
| [`bank-2.md`](bank-2.md) | 65 | 16 | 2 |
| [`bank-3.md`](bank-3.md) | 65 | 16 | 2 |

All three files share the same structure: a short header stating the domain mix, then
`## Q1` … `## Q65`, each followed immediately by `**Correct Answer:**`, `**Explanation:**`, and
`**Why the others are wrong:**`. Diagrams (where present) sit inside the Explanation, walked through
step by step in prose immediately below the diagram. Image-based questions credit the exact AWS
documentation page the screenshot was pulled from.
