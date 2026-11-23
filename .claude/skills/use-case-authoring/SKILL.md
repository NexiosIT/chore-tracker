---
name: use-case-authoring
description: >
  Use case spec authoring: meta.yml, Gherkin scenarios and an OpenAPI contract for one backend
  operation. Use when the user wants to author or change a use case spec, its scenarios or its contract.

---

# Use Case Authoring Skill

## Purpose

This skill guides an engineer through authoring a complete use case specification. It goes from a one-sentence goal to a multi-file spec with behavioral scenarios, API contracts, and optional flow diagrams. The output is precise enough for a developer to implement directly or for an AI agent to generate code from.

The skill is interactive by design. It uses a co-pilot approach: ask proactive questions to surface missing error cases, edge cases, and ambiguity *during* authoring rather than discovering them later in code review. Quality gates run between each step to catch structural issues early.

Six files are produced (with three optional):

1. **meta.yml**: Identity, goal, actors, entities, preconditions, postconditions, dependencies, endpoint
2. **scenarios.feature**: Gherkin scenarios: happy path, error cases, edge cases with concrete examples
3. **contract.openapi.yml**: OpenAPI request/response/error schemas derived from scenarios
4. **flow.mermaid** (optional): Sequence or state diagram for multi-step or stateful use cases
5. **constraints.md** (optional): Non-functional requirements specific to this use case
6. **decisions.md** (optional): Decision tables for complex business logic

## Workflow

### Phase 1: Understand the product context

The user provides a directory path or product context files. Read these to build a mental model before authoring:

- **Data model**: entities, relationships, key attributes
- **Glossary**: domain terms and definitions
- **Tech stack**: languages, frameworks, infrastructure
- **Existing specs**: for cross-referencing and consistency
- **Standards**: conventions to follow

Don't assume a fixed structure. Look for the domain registry `domain-model.yml` (the entity vocabulary) and files with names like `glossary.*`, `tech-stack.*`, `README.*`. If sparse, ask what's available. Skim efficiently. You need the domain picture, not every detail.

After reading, confirm: "I've read your product context. You're building [X] with [Y stack], main entities are [Z]. Ready to define a use case."

### Phase 2: Establish identity and metadata → Gate 1

**Goal:** Define the use case's identity, scope, and architecture context by authoring `meta.yml`.

Run through these in conversation, not as an interrogation. Ask 1-2 questions at a time:

1. **Goal statement.** Ask the user to describe the use case in one sentence. Restate as: "As a {actor}, I want to {goal} so that {benefit}."

2. **Actor identification.** Confirm the primary actor. Probe: "Are there secondary actors involved?" *(Ask when actor field would be empty.)*

3. **Domain entities.** Ask: "What domain entities does this use case read or write?" Cross-reference with the data model. *(Ask when entities list is empty.)*

4. **Preconditions.** Ask: "What must be true before this can execute? Authentication, data state, prior steps?" *(Ask when preconditions are empty.)*

5. **Dependencies.** If entities overlap with existing specs: "Does this use case call others, or is it called by others?" *(Ask when no dependency information exists.)*

6. **External integrations.** If the use case calls an external service (a payment provider, a partner API, object storage): "Which external services does it call?" Fetch the registered vocabulary with `ie integrations list --json` and reference an integration by its id in `meta.yml` `integrations: [<id>]`. **Missing one?** Record an integration change request for the author to add to `integrations/index.yml`. The request holds the id, kind (`saas`/`partner-api`/`infrastructure`), intended `impl` (package or custom), behaviour standard, and required env-var *names*. Then reference the closest existing id (or leave a clearly-flagged TODO) so the spec stays authorable. Cross-cutting infrastructure every service uses (e.g. an identity provider) is pinned in the registry but need not be listed per use case. *(Ask only when the use case makes an outbound call.)* (Change-request curation rides feature approval.)

7. **API endpoint.** Ask: "What is the primary API endpoint? Method and path?" *(Ask when endpoint is empty.)*

8. **Postconditions.** Ask: "After successful execution, what's guaranteed to be true? State changes, events emitted?"

Author by **editing** the `meta.yml` that `ie spec create` scaffolded. It already carries the identity and the `status` / `version` / `priority` fields. Add the goal, actor, entities, integrations (only if the use case calls an external service), preconditions, postconditions, and endpoint. Keep every pre-filled field. A rewrite from scratch is how `status`/`version`/`priority` go missing.

**Gate 1: Context & Preconditions**
Validate before proceeding:
- Goal statement present and follows "As a {actor}..." format → **blocking**
- Actor identified → **blocking**
- At least one entity identified → warning
- Any `integrations[]` id resolves in the registry (unresolved warns at `ie spec lint`) → warning
- At least one precondition listed → warning
- Endpoint defined with method and path → warning
- Present pass/warn/fail for each. Blocking checks must be resolved.

---

### Phase 3: Author scenarios → Gate 2

**Goal:** Write `scenarios.feature` with comprehensive behavioral coverage.

**3a. Happy path first.** Ask the user to walk through the successful execution step by step. Write as the first Gherkin scenario with concrete example values (real names, numbers and dates).

**3b. Probe for errors.** Based on the happy path, ask targeted questions:
- "What happens when {key_input} is missing, empty, or malformed?" *(Ask when fewer than 2 error scenarios exist.)*
- "Are there authorization rules? Can everyone do this?" *(Ask when no auth scenario exists.)*
Write error scenarios for each identified failure mode.

**3c. Probe for edge cases.** Select 1-2 based on context:
- If the UC involves writes: "What if the user tries this twice in quick succession?" *(Idempotency)*
- If numeric inputs: "Boundary values? Min/max for numbers, length limits?" *(Boundaries)*
- If external dependencies: "What if an external dependency is unavailable?" *(Resilience)*
- If concurrent access possible: "Two users doing this simultaneously?" *(Concurrency: lower priority, ask only if relevant)*

**3d. Check postconditions.** Ask: "Side effects? Emails, events, audit logs, state changes elsewhere?" *(Ask when postconditions list has fewer than 2 entries.)*

Write the complete `scenarios.feature` following Gherkin syntax with a Feature block including the user story.

**Gate 2: Scenario Completeness**
Validate before proceeding:
- Happy path scenario exists → **blocking**
- At least 2 error scenarios → **blocking**
- At least 1 edge case scenario → warning
- Scenarios use concrete examples (not placeholders) → warning
- Feature description complete with user story → warning
- No contradictions with existing specs (same preconditions leading to incompatible outcomes) → **blocking**
- Postconditions covered in Then steps → warning
- Present pass/warn/fail. Show scenario coverage summary: N happy, N error, N edge.

---

### Phase 4: Author contracts → Gate 3

**Goal:** Write `contract.openapi.yml` with concrete schemas derived from scenarios.

1. **Extract fields from scenarios.** Review all steps and identify every data field. List them and confirm.

2. **Build request schema.** For each input field:
   - Determine type, format, constraints
   - Probe ambiguous fields: "What format is '{field}'? String, number, date-time, uuid, enum?" *(Ask for untyped string fields.)*
   - Probe constrained fields: "Valid values for '{field}'? Should it be an enum?" *(Ask when a field appears to accept a fixed set.)*
   - Determine required vs optional: "Is '{field}' required or optional? Default value?" *(Ask when not in required array and no default.)*

3. **Build response schema.** For each output field: determine type, format, add example values.

4. **Build error schemas.** Map each error scenario to an HTTP status code and error response body.

5. **Cross-check alignment.** Verify all scenario fields appear in schemas. Flag gaps: "Scenarios mention '{field}' but it's not in the contract. Add it?" *(Critical: ask whenever misalignment is detected.)*

6. **Edit** the `contract.openapi.yml` that `ie spec create` scaffolded. Keep its templated `openapi:` version and follow the **conformance convention in its header comments**: schemas describe the UNWRAPPED response/error body in **camelCase**. The service adds the `{ data, error, meta }` envelope and the snake_case wire itself, so the contract leaves both out. Edit the scaffold, because a fresh file tends to re-introduce the envelope.

**Gate 3: Contract Concreteness**
Validate before proceeding:
- Request schema has properties defined (not opaque `type: object`) → **blocking**
- Response schema has properties defined → **blocking**
- At least one error response (4xx/5xx) defined → **blocking**
- No opaque objects anywhere in the contract → **blocking**
- Scenario-contract field alignment (all scenario fields exist in schemas) → **blocking**
- Error scenario-response code coverage → warning
- Field examples present → warning
- Present pass/warn/fail. Show scenario-contract alignment matrix.

---

### Phase 5: Assembly and optional artifacts → Gate 4

**Goal:** Complete the spec with optional files and run final quality check.

1. **Flow diagram.** If 3+ scenarios or multi-step behavior:
   - Ask: "Would a sequence or state diagram help here?"
   - If yes, create `flow.mermaid` with happy path AND error branches
   - If a flow exists but only shows happy path: "Include error paths in the diagram?"

2. **Constraints.** If non-functional requirements exist specific to this UC (performance, security, rate limits):
   - Create `constraints.md`

3. **Decisions.** If complex business logic needs a decision table:
   - Create `decisions.md`

4. **Cleanup.** If any of the three optional files were pre-scaffolded but do not apply to this UC (they still contain only the scaffold template), delete them. Every check treats a missing optional file as not applicable. A leftover template scores as authored content.

5. **Readiness scoring.** Evaluate the complete spec:

| Category | Points | Checks |
|----------|--------|--------|
| scenarios.feature | 35 | Happy path (10), errors ≥2 (8), edge cases (5), concrete examples (5), feature description (3), no duplicates (4) |
| contract.openapi.yml | 25 | Request concrete (8), response concrete (8), error responses (5), field examples (4) |
| flow.mermaid | 10* | Flow present (5), error paths included (5) |
| meta.yml | 15 | Goal statement (5), preconditions (3), postconditions (3), entities listed (4) |
| Cross-file | 10 | Scenario-contract alignment (6), error scenario-response alignment (4) |
| **Total** | **100** | *Conditional checks redistribute when not applicable* |

**Gate 4: Readiness Check.** Read the CLI grade, not the raw score. A failed blocking check caps the grade below `ready` regardless of points (G4-02).
- Grade `ready` → ready
- Grade `needs_work` → needs work (address listed items)
- Grade `blocked` → blocked (resolve blocking issues)
- Flow diagram present if applicable → warning
- No placeholder content (TBD, TODO) → **blocking**
- Present full score breakdown and recommendation.

## Output

A complete use case spec folder containing:
- `meta.yml`: identity, goal, actors, entities, preconditions, postconditions, dependencies, endpoint
- `scenarios.feature`: Gherkin scenarios with happy path, errors, and edge cases
- `contract.openapi.yml`: OpenAPI schemas for request, response, and errors
- `flow.mermaid` (if applicable): sequence or state diagram
- `constraints.md` (if applicable): non-functional requirements
- `decisions.md` (if applicable): decision tables

Optional files that do not apply are absent, not left as unfilled templates.

## File naming and location

Save files in the directory the user specifies. Default structure:
```
<spec-folder>/
├── meta.yml
├── scenarios.feature
├── contract.openapi.yml
├── flow.mermaid          (optional)
├── constraints.md        (optional)
└── decisions.md          (optional)
```

Ask the user if they have a preferred location or naming convention.

## Co-pilot principles

- **Be conversational.** Ask 1-2 questions per section, not a wall of questions.
- **Probe don't interrogate.** Frame questions around what the system should do in edge cases, not as a checklist.
- **Never block unnecessarily.** If the user defers a question, note it. The gate catches gaps.
- **Reference existing work.** "I see the Product entity already has a reviews field in the data model..."
- **Take domain knowledge from the user.** Every business decision comes from the user.
- **Present every gate.** Present each gate before you go to the next phase.

## Target

- Ask your open questions and wait for the answers before you write any spec file. The conversation is the point. Never write spec files in the turn that receives the request, even when the user asks you to skip the discussion.
- Describe observable behaviour in domain language in every scenario. Leave SQL and library calls to the code.
- Name a concrete, checkable outcome in each Then step, such as a status code, a field value or an event.
- Write at least a validation-failure, a not-found and an auth-failure scenario for every endpoint. Gate 2 blocks a spec without error scenarios.
- Use concrete example values in scenarios: real names, numbers and dates.
- Write the contract after every question about its data fields is resolved.
- Cover one coherent unit of behaviour per use case. If one operation triggers another automatically, write the trigger as the use case and the effect as its outcome. Split only operations that a caller can start on their own.
