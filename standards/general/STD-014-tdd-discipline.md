---
id: STD-014
name: "TDD Discipline"
version: "1.1.0"
category: general
status: active
last_reviewed: "2026-06-01"
---

# TDD Discipline

> All production code in CP1-CP3 MUST be written using the RED-GREEN-REFACTOR cycle — no production code without a failing test first. CP4-CP5 use adapted test-first.

## Motivation

Code written before its test verifies the implementation, not the requirement — it passes by construction and misses spec misunderstandings. The cycle forces incremental, verifiable progress and yields a test suite that mirrors the spec.

## Rules

1. **RED** — write one failing test for the next unit of behavior; run it; confirm it fails because the feature is missing (not a typo/import error).
2. **GREEN** — write the minimal code to pass; run it; confirm every other test still passes.
3. **REFACTOR** — only while green: remove duplication, improve names, re-run after each step; add no new behavior.
4. **Checkpoint scope** — CP1-CP3: strict RED-GREEN-REFACTOR. CP4 (presentation): test-first skeleton before the component; full cycle not required for visual/layout code. CP5 (deployment): validation-first (app starts, config resolves) before infra changes.
5. **Gherkin mapping** — each scenario maps to at least one cycle in CP2: Given → arrange, When → act, Then → assert.

## Enforcement

This standard is **enforced procedurally by the `/ie/generate` workflow**, not loaded into generation context — it is marked `injection: workflow` in `standards/index.yml` (dual-layer). The rules above are the canonical source the generate template embeds and enforces at every checkpoint; edit this file to change the rule, and `/ie/generate` carries the enforcement.

## Applicability

- **Service types:** all (backend, frontend, gateway)
- **Domains:** all
- **Exceptions:** None. Proven patterns may skip inter-checkpoint pauses but not the cycle itself.

Part of the **Agent Execution Discipline** with STD-015 (Verification) and STD-016 (Debugging) — the three are enforced together.

## References

- STD-009: Test Patterns (stack-specific test conventions)
