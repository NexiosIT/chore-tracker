---
id: STD-015
name: "Verification Protocol"
version: "1.1.1"
category: general
status: active
last_reviewed: "2026-10-07"
---

# Verification Protocol

> No completion claim without fresh verification evidence. Every checkpoint gate and the final generation status MUST be backed by a freshly executed proof command with observable output.

## Motivation

"Tests should pass" is a guess, not verification. Stale or assumed evidence ships undefined functions, missed requirements, and false completion claims. Every claim must rest on a freshly executed command with observable output.

## Rules

1. **5-step gate** — before writing any `cp{N}-result.yml` with `status: passed` or running `ie generation update --repo . --status completed`: **IDENTIFY** the proof command → **RUN** it fresh (not cached, not from memory) → **READ** the full output and exit code → **VERIFY** it confirms the claim → **RECORD** the evidence. Skipping any step invalidates the verification.
2. **Evidence** — record into the checkpoint result's `verification_evidence` field:
   ```yaml
   verification_evidence:
     command: "npm test"            # exact command executed
     exit_code: 0
     summary: "47/47 tests passed"  # key output line confirming the claim
     timestamp: "2026-05-15T..."
   ```
3. **No hedging** — banned in completion claims: "should pass", "probably", "seems to", "I believe", "based on the code", and premature "Done!/Perfect!" before running. State definitive, evidence-backed results.
4. **Bug fixes** — a fix claim requires the full red-green cycle: regression test red before the fix, green after, then full suite green.
5. **Scope** — every checkpoint gate, every retry, and finalization; all promotion tiers (proven patterns skip pauses, not verification). Skipped checkpoints (`status: skipped`) need no evidence.

## Enforcement

Enforced procedurally by the `/ie/generate` workflow, not loaded into generation context — marked `injection: workflow` in `standards/index.yml` (dual-layer). This file is the canonical rule; `/ie/generate` embeds and enforces it at every gate.

## Applicability

- **Service types:** all
- **Domains:** all
- **Exceptions:** Skipped checkpoints (`status: skipped`) require no verification evidence.

Part of the **Agent Execution Discipline** with STD-014 (TDD) and STD-016 (Debugging) — the three are enforced together.

## References

- STD-014: TDD Discipline
