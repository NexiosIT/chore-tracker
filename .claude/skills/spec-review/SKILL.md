---
name: spec-review
description: >
  Spec review: a structured quality and consistency review of one multi-file spec folder.
  Use when the user wants to check whether a spec is ready for approval, implementation or generation.
---

# Spec Review Skill

## Purpose

This skill performs a structured, reproducible review of a multi-file specification. It evaluates completeness, internal consistency, cross-spec alignment, quality, and readiness. It produces a categorized report with actionable findings and a quantitative readiness score.

The review is thorough but constructive: every finding includes a concrete suggestion. Findings are categorized by severity so the author can prioritize fixes. The skill runs through the full checklist even when the spec looks good. Surface-level quality can mask subtle inconsistencies between files.

## When to Use

Use this skill when you need to evaluate whether a specification is ready for the next phase: approval, implementation, or code generation. It works on any multi-file spec that combines behavioral scenarios (Gherkin), API contracts (OpenAPI), flow diagrams (Mermaid), and optionally component trees (YAML).

Typical triggers: a spec has been authored and needs review before sign-off, a reviewer wants a structured checklist to follow, or an author wants to self-check before requesting review.

## Inputs

Before starting, confirm with the reviewer:

- **Spec path**: Path to the specification folder containing the files to review
- **Standards**: Path to any coding/design standards the spec should comply with (optional but recommended)
- **Related specs**: Other specs that share entities, dependencies, or interactions with this one (optional)
- **Readiness rubric**: A scoring rubric mapping quality criteria to point values (optional: the skill provides a default rubric if none is supplied)

Load the spec folder and identify which files are present. The skill adapts to whatever files exist. It does not require all file types.

## Review Workflow

### Phase 1: Structural Completeness

Verify all files that should be present are present and non-empty. Build a file inventory:

| File | Purpose | Present? | Non-empty? |
|------|---------|----------|------------|
| Metadata file (e.g., `meta.yml`) | Identity, actors, entities, dependencies |  |  |
| Scenario file (e.g., `scenarios.feature`) | Gherkin behavioral scenarios |  |  |
| Contract file (e.g., `contract.openapi.yml`) | API request/response/error schemas |  |  |
| Flow diagram (e.g., `flow.mermaid`) | State machine or sequence diagram |  |  |
| Constraints (e.g., `constraints.md`) | Non-functional requirements |  |  |
| Decision tables (e.g., `decisions.md`) | Complex branching logic |  |  |

Determine which optional files should be present based on the spec's content:
- Flow diagram: recommended when there are 3+ scenarios (multiple paths to visualize)
- Constraints: recommended when there are non-functional requirements
- Decision tables: recommended when there is complex conditional logic

**Log findings** for missing files that should be present given the spec's content.

---

### Phase 2: Internal Consistency

Check that all files within the spec agree with each other. This is the highest-value phase. Inconsistencies between files are the most common source of downstream bugs.

**Metadata ↔ Scenarios:**
- Actor(s) in metadata match actor(s) in scenario Given steps
- Preconditions in metadata correspond to Given steps
- Postconditions in metadata correspond to Then steps in the happy path

**Metadata ↔ Contract:**
- Endpoint or operation in metadata matches the path/operation in the contract
- Entities in metadata align with schema objects in the contract

**Scenarios ↔ Contract:**
- Every field referenced in scenario steps exists in a request, response, or error schema
- Every error scenario maps to a defined HTTP status code and error schema
- Happy path Then steps align with the success response schema

**Contract ↔ Component Tree** (when both present):
- Fields displayed in the component tree exist in the response schema
- Data types in the component tree are compatible with contract types

**Scenarios ↔ Flow Diagram** (when both present):
- Flow paths correspond to scenario outcomes (happy path and error branches)
- States in the flow diagram map to meaningful scenario boundaries

**Log findings** for every inconsistency found. Each finding must reference both files and the specific fields/lines involved.

---

### Phase 3: Cross-Spec Consistency

Check alignment with related specs and the broader system. This phase requires related specs to be loaded. If none are provided, note that cross-spec checks were skipped and log a suggestion to provide related specs.

- **Dependencies exist**: Every spec ID referenced in dependencies actually exists in the provided context
- **Bidirectional dependencies**: If Spec A calls Spec B, Spec B's "called by" should include Spec A
- **Entity consistency**: Shared entities (same name across specs) should have compatible field definitions
- **No scenario contradictions**: Identical preconditions should not lead to incompatible outcomes across specs

**Log findings** for mismatches. If related specs are not available, log this as a limitation rather than a finding.

---

### Phase 4: Quality Assessment

Evaluate spec quality beyond structural correctness. This is where craftsmanship matters.

- **Scenarios use concrete examples**: Real names, numbers and dates instead of "some value" or "valid input". Concrete examples catch edge cases that abstract placeholders miss.
- **Error scenarios are specific**: Each error has a distinct cause and expected behavior, not generic "invalid input" catch-alls.
- **Contract schemas have examples**: Properties include `example` values for documentation and test generation.
- **Descriptions are meaningful**: Goal statement, scenario names, and field descriptions convey intent clearly.
- **Formatting follows conventions**: Gherkin style, OpenAPI structure, YAML formatting are consistent and follow any provided standards.
- **No hardcoded visual values in component trees**: All visual values (colors, spacing, typography) reference design tokens, never hardcoded values.

**Log findings** for quality issues. These are typically Suggestions or Warnings rather than Blockers.

---

### Phase 5: Readiness Scoring

**Do not compute a score of your own**. The quantitative readiness number comes from the CLI: run `ie spec readiness --path <spec> --json` and quote its `score` and `grade` as *the single-spec structural score*. The readiness rework removed one conflation: two different numbers, the mechanical CLI score and an LLM-judged one, both printed as "N/100". A recorded "100/100" then did not say which instrument produced it.

Your review contributes the **findings** (Phases 1–4): the cross-spec consistency, ownership, and semantic judgment that the structural score explicitly does not verify. Read the CLI grade knowing:
- a failed `severity: blocking` rubric check caps the grade below `ready` regardless of points (G4-02);
- not-applicable conditional checks redistribute their weight (the `not_applicable` list in the JSON shows which);
- the score verifies single-spec structure only. It never verifies cross-spec consistency, ownership, or flow validity.

## Output Format

### Summary

One paragraph overview: what the spec covers, overall quality assessment, and recommendation (approve / revise / reject).

### Findings

Organized by severity. Each finding includes a reference ID, its defect-class slug, description, file location, and a concrete suggestion for how to fix it. The class slug is one of `module-ownership`, `seam-existence`, `prose-scenario`, `invariant-strength`, `sibling-parity`, `rendered-binding`, `flow-parse`. Write out the slug, never a bare letter code (STD-023). The slug makes recurrence per class measurable after the spec gates. If a finding fits none of the seven classes, give it no slug rather than a forced one.

**Blocking** (must resolve before the spec can proceed):
- [B-001] ({class-slug}) {description}. File: {file}, Location: {line/field}. Suggestion: {how to fix}

**Warnings** (should resolve, but do not block):
- [W-001] ({class-slug}) {description}. File: {file}, Location: {line/field}. Suggestion: {how to fix}

**Suggestions** (optional improvements):
- [S-001] {description}. Suggestion: {how to improve}

### Structural Score (quoted from the CLI)

> `ie spec readiness`: **{score}/100, grade {grade}**. Single-spec structural score. It does not verify cross-spec consistency, ownership, or flow validity.
> {If the grade was capped: "Grade capped below ready by failed blocking check(s): {ids}."}

### Recommendation

Findings count: {blocking} blocking, {warnings} warnings, {suggestions} suggestions.

One of the following. The verdict keys on findings + the CLI grade, never on a score band of your own:
- **Approve**: Zero blocking findings AND CLI structural grade `ready`. Spec is ready.
- **Revise**: Any blocking finding, or CLI grade `needs_work`. Return to author with findings.
- **Reject**: CLI grade `blocked`, or the findings demand re-shaping. Requires substantial rework.

## Edge Cases

- **Incomplete spec (some files missing)**: Review what exists. Score absent-but-needed files as 0 in their category. Note which files are missing in the Summary.
- **Single-file spec**: The skill works on whatever is present. Cross-file consistency checks are skipped. The score adjusts. Categories for absent files score 0 only if the file should have been present.
- **No standards provided**: Skip standards-based formatting checks in Phase 4. Note this limitation. Suggest providing standards for a more thorough review.
- **No related specs provided**: Skip Phase 3 (cross-spec consistency). Note this in the report. Suggest providing related specs if dependencies are declared in metadata.
- **Custom readiness rubric**: If the caller provides a rubric with different categories or weights, use that instead of the default. The output table adapts to match.
