---
id: STD-023
name: "Human-Readable Output"
version: "1.1.0"
category: general
status: active
last_reviewed: "2026-09-02"
---

# Human-Readable Output

> Every human-facing output MUST be understandable cold — by a reviewer with no other document open. Plain language, expand a coded term on first use, never invent a code scheme, lead with one locator line. Every sentence MUST have one reading: active voice, one instruction per sentence, short, no phrasal verbs, no semicolons, hedges kept.

## Motivation

Review findings, checkpoint summaries, and status messages are read by a human deciding whether to sign off. Framework shorthand the agent coins on the fly — `W-1`, an ad-hoc `CP-xyz` — has no referent the reader can look up, and forces a context switch at the moment attention is scarcest. Comprehension is the goal, and more prose is not comprehension: a wall of explanation reads as poorly as a wall of codes. The rule is clarity through *plain and short* — not volume.

## Rules

Applies to every human-facing output the workflow produces (checkpoint reports, review findings, status, blocking explanations). It does **not** apply to machine-readable output (see Applicability).

1. **Plain over coded.** Prefer a plain clause to a coined label. Write `the controller skips the validation UC-014 requires`, not `W-1: validation gap`.
2. **No invented identifiers.** Reference only identifiers the framework issued — `UC-013`, `FT-007`, `CP3`, `STD-007`. Never mint your own scheme (`W-1`, `ISS-2`, `F-3`) to enumerate findings; a plain list or a table's own row number is enough.
3. **Expand on first use.** The first time a stable coded term appears in an output, expand it once — `Checkpoint 2 (CP2)`, `use case UC-013` — then use the bare code. Two words, not a sentence.
4. **One locator line.** Lead each checkpoint or review report with a single orientation line so the reader knows where they are: `Checkpoint 2 of 5 · domain logic · next: interface layer`. One line, never a paragraph.
5. **Brevity is part of clarity.** Expand once and briefly; link `product-hub-docs/GLOSSARY.md` rather than re-explain a term inline; cut any sentence that carries no information for the reviewer.

### Sentence shape

These rules follow the structural part of ASD-STE100 (Simplified Technical English). They apply to every human-facing string generation emits: error messages, log messages, contract descriptions, command and tool descriptions, README prose, checkpoint reports, review findings. Pick the mode by surface:

- **Strict** for a string a machine or another agent parses before a human does: error messages, log messages, OpenAPI and AsyncAPI `description` fields, command and tool descriptions. Apply every rule below in full.
- **STE-flavored** for prose a human reads: README text, checkpoint reports, review findings, blocking explanations. Apply every rule below; the word cap is a target rather than a hard limit when a condition or a scope qualifier needs the extra words.

6. **Active voice, named actor.** `The service rejects an order without a customer id.` Not `Orders without a customer id are rejected.` Passive is allowed only when the actor is unknown or irrelevant.
7. **One instruction or one fact per sentence.** `Open the file. Read line 3.` Not `Open the file and read line 3, then check that it matches.` Three or more steps or conditions go in a list, not a sentence.
8. **Short sentences.** At most 20 words for an instruction or an error message, at most 25 for a description. Split, never compress: dropping the subject, the verb or an article to fit the cap is a violation (`Files not backed up will be lost` has two readings).
9. **One plain verb.** No phrasal verbs (`start`, not `spin up`; `remove`, not `take off`; `contact`, not `reach out`). Pick one verb for one action and reuse it throughout the document (`check` everywhere, not `check` / `verify` / `confirm` for the same act). Prefer the verb to its noun form (`analyze the log`, not `perform an analysis of the log`).
10. **No semicolons, short noun stacks.** Split a sentence instead of joining it with a semicolon. Stack at most three words as one noun phrase (`fuel pump valve`); rewrite a longer stack with a preposition or a relative clause.
11. **Keep the hedge.** `The request may have failed` stays `may have`. A rewrite that turns a hedge into a fact, or adds a cause the source did not state, is a different claim, not a shorter one. Keep a compound tense where the simple form loses the meaning (`the job has completed` says the output is available now).

## Enforcement

Enforced procedurally by the `/ie/generate` and `/ie/review-checkpoint` workflows, not loaded into generation context — marked `injection: workflow` in `standards/index.yml` (dual-layer mechanism). This file is the canonical rule; the workflows embed and enforce it on every human-facing output. The product-hub `product-hub-docs/GLOSSARY.md` is the canonical definition of the framework's stable terms — an output may link it, but must still read clearly with nothing else open (Rule 1).

## Applicability

- **Service types:** all
- **Domains:** all
- **Exceptions:** Machine-readable output is exempt — `--json` payloads and file artifacts under `.flowforge/` are consumed by tools, not read for comprehension, and use the framework's coded identifiers directly. Applies regardless of promotion tier.

Introduced by the output-readability discipline; uses the same `injection: workflow` enforcement mechanism as the Agent Execution Discipline standards STD-014 / STD-015 / STD-016. The sentence-shape rules (6 to 11) carry the structural rules of ASD-STE100 only. Its word-choice rules depend on the ASD approved-word dictionary, which is not redistributable, and are out of scope.

## References

- STD-003: Naming Conventions (the framework code schemes this standard keeps legible)
- STD-032: Comment Discipline (the few comments it permits follow rules 6 to 11 as any other prose does)
- ASD-STE100 Issue 9 (2025), rule sections on sentence structure: https://www.asd-ste100.org
