---
name: standards-authoring
description: >
  Standards authoring: one project-wide STD document plus its index and changelog entries.
  Use when the user wants to add, change or deprecate a standard, or promote a CI-detected pattern.
  Not for a use-case constraint (use use-case-authoring).
---

# Standards Authoring Skill

## Purpose

This skill guides the addition, change, or deprecation of a **standard**: a project-wide,
repeatable rule that constrains code generation ("how we always build"). It exists to make
one fragile operation safe: a standard lives in **three places that must stay in sync**, and
forgetting any one of them fails silently.

| File | Role | Failure if forgotten |
|------|------|----------------------|
| `standards/{category}/STD-{NNN}-{name}.md` | The rule itself | Index points at a missing file |
| `standards/index.yml` | Registry + applicability rules | **Standard is silently never injected into generation** |
| `standards/changelog.md` | Audit trail | History gap; reviewers can't see what changed |

The most dangerous case is forgetting the `index.yml` entry: the document exists, looks
authored, but Phase 4 manifest assembly never selects it. No generation ever applies it,
and no error appears. This skill always writes (or updates) all three together.

This skill is interactive and uses a co-pilot approach: it surfaces the standard-vs-constraint
boundary, the right category, and the applicability scope *before* writing, then verifies the
three files are consistent at the end.

## Background: read these once

- **Standards vs constraints.** Standards = repeatable, project-wide. `constraints.md`
  = non-functional requirements specific to one use case. This skill owns the first; the
  use-case-authoring skill owns the second.
- **Two-layer evolution.** Standards are introduced manually (Phase 0) or promoted by
  human review of CI-detected patterns. Promotion is a judgment call. This skill assists it and
  does not replace it.
- **Stack-aware partition.** `general` / `api` / `nfr` are stack-agnostic (always
  considered, filtered by service type). Any other directory is a framework name matching the
  `framework` field in `services/*.yml` and is injected only when a touched service declares it.
- Index schema: `schemas/standards-index.schema.json` (bundled with the CLI).
- Model document to imitate for structure: the highest-numbered `STD-*` file already in the hub.

## Workflow

### Phase 1: Classify: is this even a standard? → Gate 1

Before authoring anything, settle two questions in conversation:

1. **Standard or constraint?** Ask: "Does this rule apply to *every* feature that fits its
   scope, or only to one specific use case?"
   - Project-wide and repeatable (e.g., "modals always trap focus", "all list endpoints
     paginate", "no magic values") → **standard**, continue here.
   - Specific to one use case (e.g., "*this* dashboard must load in < 2s", "*this* endpoint
     needs MFA above $10k") → **constraint**. Stop. Tell the user this belongs in that use
     case's `constraints.md` and is authored via `/ie/shape-uc` (use-case-authoring skill).
     Do not add it to `standards/`.

2. **Is it worth a standard yet?** Ask: "Has this pattern appeared enough times to
   be a rule, or is it a one-off we're guessing at?" Standards are promoted from observed,
   repeated practice, not invented speculatively. If it's premature, suggest leaving it as a
   draft (`status: draft`) or a CI-detected candidate rather than an active rule. Never block,
   but make the call explicit.

**Gate 1: Scope**
- Confirmed project-wide and repeatable (not feature-specific) → **blocking**. If feature-specific, redirect to `/ie/shape-uc` and stop.
- Promotion rationale stated (why it's a rule now) → warning.

---

### Phase 2: Categorize and identify → Gate 2

1. **Pick the category**. Ask what the rule governs:
   - **Stack-agnostic**: applies regardless of framework → `general` (naming, repo hygiene,
     git, TDD, no-magic-values), `api` (contract-level: envelopes, error categories,
     versioning), or `nfr` (performance, accessibility, observability budgets).
   - **Stack-coupled**: only meaningful for a specific framework → a directory named exactly
     after the `framework` field in `services/*.yml` (e.g., `spring-boot`, `nextjs`, `dotnet`).
     Read `services/*.yml` and confirm the framework value. If the rule is for a stack not yet
     present, create the new directory. No code change is needed. The manifest assembler keys
     off the directory name.

2. **Assign the ID.** Run `ie id next standard` and use the number it reports. Never read
   `standards/index.yml` and increment: the index shows only what has merged, so it cannot see a
   standard someone has pushed on an unmerged branch, and two authors working at the same time
   would be handed the same number. **IDs are global and sequential across all categories**. They are not
   per-category (e.g., `STD-021` lives under `spring-boot` even though `STD-020` is `general`).
   Never reuse a retired ID.

3. **Name the file.** `STD-{NNN}-{kebab-name}.md`, placed under `standards/{category}/`.

4. **Define applicability.** Ask which service types the rule constrains:
   - `service_types`: subset of `["backend", "frontend", "gateway"]`, or `[]` for all.
   - `domains`: default `[]` (all domains). Set a non-empty list **only** to scope the standard
     to specific spec domains (the `specs/{domain}/` directory names). The assembler then
     selects it only for generations touching one of those domains. Most standards
     are cross-domain; leave `[]` unless the rule genuinely applies to one domain only.
   - Note: for a stack-coupled standard, the framework directory already gates injection; the
     `service_types` filter narrows further (e.g., a `nextjs` standard is `["frontend"]`).
   - `paths`: optional. List the file globs this standard governs, relative to a code repo root.
     Write each glob layout-neutral, so one standard fits several repo layouts.
     For example, write `**/controller/**` or `**/app/**`, not `src/main/java/com/acme/controller/**`.
     `ie update --code-repos` writes the Rules section into each selected code repo as a path-scoped rule.
     Claude Code and Copilot load that rule when they touch a matching file.
     Leave `paths` out for a standard without a clear file type, such as naming or git hygiene.

**Gate 2: Identity**
- Category chosen and the directory exists (or is being created for a declared/extending stack) → **blocking**.
- ID is the number `ie id next standard` reported, not one read off the index → **blocking**.
- `applies_to.service_types` decided (explicit `[]` is a valid answer) → **blocking**.
- `applies_to.paths` is absent or holds only layout-neutral globs → warning.
- For a stack-coupled category, the directory name matches a `framework` in `services/*.yml` (or the user confirms they are introducing that stack) → warning.

---

### Phase 3: Author the document → Gate 3

Write `standards/{category}/STD-{NNN}-{name}.md` following the established structure. Open the
highest-numbered existing `STD-*` file in the hub and mirror its shape:

```markdown
---
id: STD-{NNN}
name: "{Human Readable Name}"
version: "1.0.0"
category: {category}
status: active
last_reviewed: "{today's date, ISO YYYY-MM-DD}"
---

# {Human Readable Name}

> One-sentence MUST rule stated imperatively: the single thing this standard enforces.

## Motivation

Why this rule exists: the cost of not following it. Concrete, not abstract.

## Rules

1. Numbered, testable rules. State exceptions inline (e.g., "except 0, 1, -1 used idiomatically").
2. ...

## Examples

### Compliant
\`\`\`{lang}
// minimal example that follows the rule
\`\`\`

### Non-Compliant
\`\`\`{lang}
// minimal example that violates it, with a comment saying why
\`\`\`

## Applicability

- **Service types:** {all | backend | frontend | gateway}
- **Domains:** all
- **Exceptions:** {explicit carve-outs, or "none"}

## References

- {related STD-XXX or ADR-XXXX}
```

Authoring rules:
- Write the Rules section so it reads alone. A path-scoped rule carries the Rules section only, without Motivation or Examples.
- The blockquote is the rule in one sentence. If you can't state it in one sentence, the
  standard is doing too much. Split it.
- Every rule must be **testable**: a reviewer (human or agent) can decide pass/fail without
  guessing. "Handle errors gracefully" is not a rule; "every endpoint returns a typed error
  body with an HTTP 4xx/5xx code" is.
- Always include both a Compliant and a Non-Compliant example. The contrast is what makes the
  rule generatable.
- Take domain knowledge from the team. Every rule comes from how the team actually builds, confirmed
  with the user.

**Gate 3: Document quality**
- Frontmatter complete (`id`, `name`, `version`, `category`, `status`, `last_reviewed`), and `id`/`category` match Phase 2 → **blocking**.
- One-sentence MUST rule present → **blocking**.
- At least one Compliant and one Non-Compliant example → **blocking**.
- Rules are testable (no vague language) → warning.
- No placeholder content (TBD, TODO) → **blocking**.

---

### Phase 4: Register: the 3-file sync → Gate 4

This is the whole point of the skill. After the document is written, register it.

1. **Add the `index.yml` entry.** Append to the `standards:` list. All schema-required fields:

   ```yaml
   - id: "STD-{NNN}"
     name: "{Human Readable Name}"          # match the document frontmatter
     category: "{category}"
     path: "standards/{category}/STD-{NNN}-{name}.md"   # must match the real file path
     description: "{one-sentence summary of what it enforces}"
     status: "active"                        # only "active" is injected at Phase 4
     version: "1.0.0"
     discovery_layer: "manual"               # or "guardian-review" if promoted from CI detection
     last_reviewed: "{today's date, ISO}"
     applies_to:
       service_types: [{decided in Phase 2}]
       domains: []
       paths: [{decided in Phase 2}]       # optional: omit for a standard without a clear file type
   ```

   The `path` MUST point at the file you just wrote. This is the link the assembler follows.
   Validate the entry against `schemas/standards-index.schema.json`.

   *Optional `injection` field:* omit it for normal standards (defaults to `context`: loaded
   into the generation context). Set `injection: "workflow"` **only** for a standard that is
   enforced procedurally by a workflow template (e.g. the execution-discipline standards embedded
   in `/ie/generate`). Such standards are excluded from manifest injection so their
   bodies aren't loaded redundantly. This is rare. Most standards are `context`.

2. **Append the `changelog.md` row:**

   ```
   | {today's date, ISO} | added | STD-{NNN} | {Name}: {brief description} |
   ```

   Use `updated` or `deprecated` as the action for changes (see "Changing / deprecating" below).

**Gate 4: Sync verification (the gate that matters)**
- The `STD-*` file exists at exactly the `path` recorded in `index.yml` → **blocking**.
- `id`, `name`, `category`, `status`, `version` match between the document frontmatter and the index entry → **blocking**.
- The index entry passes the schema (`standards-index.schema.json`) → **blocking**.
- A `changelog.md` row was added → **blocking**.
- Recommend the user run `ie doctor` to catch stack/standards drift (e.g., a stack-coupled standard whose framework no service declares). Report its result.

## Changing or deprecating an existing standard

- **Change:** edit the `STD-*` document, bump `version` (semver: patch for wording/examples,
  minor for a new rule, major for a breaking redefinition) in BOTH the frontmatter and the
  index entry, update `last_reviewed` in both, and add an `updated` changelog row. Re-run Gate 4.
- **Deprecate:** set `status: deprecated` in BOTH the frontmatter and the index entry (keep the file
  as the audit trail), add a `deprecated` changelog row, and note the
  replacement standard if there is one. Deprecated standards are not injected at Phase 4.

## Output

All three files written and verified in sync:
- `standards/{category}/STD-{NNN}-{name}.md`: the standard document
- `standards/index.yml`: entry added/updated
- `standards/changelog.md`: row appended

Plus the result of the recommended `ie doctor` run.

## Co-pilot principles

- **Settle the boundary first.** The single most common mistake is putting a feature-specific
  rule in `standards/`. Always run Phase 1 before authoring.
- **One standard, one rule.** If the blockquote needs an "and", consider two standards.
- **Always run the registration gate.** A document with no index entry is invisible to generation.
- **Take rules from the team.** Standards reflect how the team actually builds, confirmed with the user.
- **Be conversational.** Ask 1–2 questions per phase, not an interrogation.

## Target

- Put stack-agnostic rules in `general`/`api`/`nfr`. Put framework rules in the directory named after their framework.
