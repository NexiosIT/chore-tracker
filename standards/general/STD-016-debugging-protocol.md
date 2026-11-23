---
id: STD-016
name: "Debugging Protocol"
version: "1.1.2"
category: general
status: active
last_reviewed: "2026-10-08"
---

# Debugging Protocol

> No fix without root-cause investigation first. Every checkpoint test failure MUST be investigated systematically before a fix is attempted.

## Motivation

Blind retry — changing code and re-running without understanding the failure — wastes the `max_retries` budget and masks deeper issues. Systematic investigation produces targeted fixes that address root causes.

## Rules

On any checkpoint test-gate failure, follow four phases in order — do not jump to a fix:

1. **Root cause** — read errors/stack traces completely (line numbers, codes); reproduce consistently; review what this checkpoint changed; trace data flow upstream from the failing assertion to where expected and actual diverge.
2. **Pattern analysis** — find similar working code in the codebase; re-read the relevant standard and compare against its examples; list every difference, however small.
3. **Hypothesis** — state one specific hypothesis ("X is the root cause because Y"); make the smallest change to test it, one variable at a time; if it doesn't fix it, form a NEW hypothesis rather than layering fixes.
4. **Implementation** — write a failing regression test for the bug (STD-014), implement ONE change addressing the ONE root cause (no bundled refactors), then verify the full gate (STD-015).

**Escalation (within `max_retries`, default 2):** each retry runs all four phases using information from the previous failure. After the final retry fails, STOP — write `cp{N}-result.yml: failed`, record a `type: investigation` reasoning-trace entry (what was investigated, hypotheses tested, what was ruled out, suspected cause), run `ie generation update --repo . --status checkpoint_failed --checkpoint <N>` (never edit `generation-status.yml` by hand: the command counts the failure in `.flowforge/metrics.yml`), and escalate to the engineer. A checkpoint run as a dispatched subagent does not run that command: it returns the failure, and the agent that dispatched it records the status once, since parallel checkpoints writing the same status file lose writes. Never attempt further fixes after the budget is exhausted.

## Enforcement

Enforced procedurally by the `/ie/generate` workflow, not loaded into generation context — marked `injection: workflow` in `standards/index.yml` (dual-layer). This file is the canonical rule; `/ie/generate` embeds and enforces it on every failure.

## Applicability

- **Service types:** all
- **Domains:** all
- **Exceptions:** None. Applies regardless of promotion tier.

Part of the **Agent Execution Discipline** with STD-014 (TDD) and STD-015 (Verification) — the three are enforced together.

## References

- STD-014: TDD Discipline · STD-015: Verification Protocol
