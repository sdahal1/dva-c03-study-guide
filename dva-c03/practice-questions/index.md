# DVA-C03 Practice Questions

A 50-question practice bank for the AWS Certified Developer – Associate (DVA-C03) exam, weighted
to match the exam's domain percentages. Each question is a realistic, scenario-based,
developer-focused exam question (often with a code/API/CLI-level detail) with 4 answer options,
the correct answer(s) marked, a full explanation, and a breakdown of why each wrong option is
wrong — cross-linked to the relevant [`services/`](../../../services) docs and domain README files
so a missed question points straight back to the material to re-study.

See [`../README.md`](../README.md) for the full DVA-C03 study plan and domain breakdown.

## This isn't the only practice material — here's the full picture

This folder (50 domain-split questions) is for studying ONE domain at a time. Two more
resources exist for different stages of prep: **[`full-exams/`](full-exams/index.md)** — 2
full-length, **timed, blind** 65-question mock exams (separate question/answer-key files)
matching the real exam's exact item count and weighting — and **[`../exam-bank/`](../exam-bank/README.md)**
— 3 more 65-question **study-mode** banks (bank-1/2/3) with the answer and explanation
inline under each question. Use domain-split questions while learning a domain, exam-bank
for full-scope drilling after all domains are done, full-exams last under real time pressure.

## Question counts by domain

| File | Domain | Exam weight | Questions | Multi-select | Notes |
|---|---|---|---|---|---|
| [`domain-1-questions.md`](domain-1-questions.md) | 1 — Development with AWS Services | 30% | 15 | 2 | Includes 1 AI-assisted development question (code generation/review) |
| [`domain-2-questions.md`](domain-2-questions.md) | 2 — Security | 26% | 13 | 2 | **5 questions (Q9–Q13) dedicated to the new Task 2.3** (AI service security) |
| [`domain-3-questions.md`](domain-3-questions.md) | 3 — Testing and Deployment | 22% | 11 | 2 | Includes 2 AI-assisted development questions (test generation, deployment approvals) |
| [`domain-4-questions.md`](domain-4-questions.md) | 4 — Troubleshooting and Optimization | 22% | 11 | 1 | Includes 1 AI-assisted development question (error analysis) |
| **Total** | | **100%** | **50** | **7 (14%)** | **4 AI-assisted dev questions + 5 Task 2.3 questions = 9 AI-related questions (18% of bank)** |

Question count per domain is proportional to its exam weight (e.g. Domain 1 at 30% of a 50-question
exam ≈ 15 questions; Domain 2 at 26% ≈ 13; Domains 3 and 4 at 22% each ≈ 11). Roughly 14% of
questions use the **(Choose TWO)** multi-select format, consistent with the real exam's mix of
"multiple choice & multiple response" (per the exam overview table in
[`../README.md`](../README.md)).

### New-content coverage (explicitly required by the DVA-C03 transition)

- **Task 2.3 (new) — AI service security**: 5 of Domain 2's 13 questions (Q9–Q13) — access
  control for Bedrock, VPC endpoints/PrivateLink for private connectivity, content filtering via
  Bedrock Guardrails, prompt injection protection, and AI agent security — directly against
  [`../domain-2-security/ai-service-security-risks.md`](../domain-2-security/ai-service-security-risks.md).
- **AI-assisted development pattern** (tested across all 4 domains per the exam's "What's new vs
  DVA-C02" section): 4 questions total — Domain 1 Q15 (code generation/review), Domain 3 Q7 & Q11
  (test generation, deployment approvals), Domain 4 Q10 (error analysis) — against
  [`../domain-1-development/ai-assisted-development.md`](../domain-1-development/ai-assisted-development.md).

## How to use this question bank

1. **Study the domain first, test second.** Read the relevant `certifications/dva-c03/domain-N-.../README.md`
   (and any linked `services/` docs) before attempting that domain's questions — these questions
   are built directly from the task statements, not trivia, so the domain docs are the actual prep
   material.
2. **Answer before looking.** Each question lists 4 options before the answer — cover the
   "Correct Answer" section (e.g. scroll slowly, or open in an editor and fold below the question)
   and commit to an answer first, including on multi-select questions.
3. **Read every explanation, not just the ones you got wrong.** The "why wrong options are wrong"
   section for options you correctly ruled out often contains a second distinct fact worth knowing
   (e.g. a service's actual limit, or a common real-exam distractor pattern) — skimming only your
   mistakes undersells the material.
4. **Follow the links on a miss.** Every explanation links to the specific `services/` doc or
   domain README section the question is drawn from — if a question stumps you, that link is the
   fastest path back to the source material, not a general re-read of the whole domain.
5. **Weight your review time like the exam weights the domains.** Domain 1 (30%) and Domain 2
   (26%) are worth more than half the exam combined — if time is short, prioritize those two files
   and the Task 2.3 AI-security questions specifically, since that's the single newest task
   statement on DVA-C03.
6. **Re-attempt multi-select questions carefully.** "(Choose TWO)" questions require *both* correct
   options to count as right — on the real exam, a multi-select question with only one correct
   selection out of two required typically scores as wrong, so practice treating partial credit as
   not credit.
7. **Use this alongside, not instead of, [`labs.md`](../domain-1-development/labs.md)-style hands-on
   exercises** in each domain folder — these questions test recognition of the correct pattern in a
   scenario, while the labs build the muscle memory of actually configuring the service.

## Status

✅ Complete — 50/50 questions written across all 4 domain files, each with full explanations and
wrong-answer rationale, cross-linked to `services/` and domain documentation.
