---
name: feature-requirements
description: >
  Feature requirements authoring: a sprint-sized requirements document with WHEN/SHALL/IF/THEN
  acceptance criteria and a decision catalog. Use when the user wants to write or change the
  requirements for one feature. Not for epic-level planning (use roadmap-refinement).
---

# Feature Requirements Skill

## Purpose

This skill turns a feature idea into a structured requirements document that a scrum team can pick up and implement in a single sprint. The output is precise, testable, and consistent. Every requirement has a user story and acceptance criteria written in WHEN/SHALL/IF/THEN notation so there's zero ambiguity about what "done" means.

The skill is interactive by design. Rather than generating requirements in one shot, it walks through a structured conversation: understand the product context, identify the feature, discuss scope and behavior, resolve open questions, and only then produce the final document. This matters because requirements that skip the discussion phase tend to miss edge cases, overscope, or bake in assumptions nobody validated.

Two files are produced:
1. **Requirements document**: the feature requirements in a consistent, implementation-ready format
2. **Decision catalog**: a companion file recording every design choice made during the discussion, with the question, the chosen answer, and why

## Workflow

### Phase 1: Understand the product

The user provides a directory path containing product-level context. Read these files to build a mental model of the product before discussing any feature:

- **Roadmap**: what's planned, what's been delivered, what's next
- **Application landscape**: the systems involved, how they interact
- **Tech stack**: languages, frameworks, infrastructure, databases
- **Glossary**: domain terms and their definitions
- **Data model**: entities, relationships, key attributes
- **Mission / product vision**: why this product exists, who it serves

Don't assume a fixed directory structure. Look for files with names like `roadmap.*`, `glossary.*`, `architecture.*`, `tech-stack.*`, `domain-model.yml` (the domain registry), `mission.*`, `vision.*`, `README.*`. If the directory is sparse, ask the user what's available rather than guessing. Skim efficiently. You need the big picture, not every detail.

After reading, give the user a short confirmation of what you found: "I've read your product context. You're building [X], using [Y stack], with [Z] as the main entities. Ready to pick a feature."

### Phase 2: Identify the feature

Two paths here:

**From a backlog or roadmap**: If the product context includes a roadmap or backlog with planned features, present the next candidates and let the user pick. Show the feature name and a one-line summary for each candidate.

**User brings the idea**: The user describes what they want to build. Acknowledge it and move to discussion.

Either way, you should end this phase with a clear one-sentence statement of the feature: "We're building [feature name]: [one sentence description]."

### Phase 3: Discuss and shape the feature

This is the heart of the skill. Have an interactive conversation to nail down exactly what the feature does. The goal is to reach **zero open questions** before producing the output.

**How to run the discussion:**

Start by proposing a high-level breakdown of the feature into 3-7 functional areas (these will become the requirements). For each area, ask targeted questions about behavior, edge cases, and error handling. Listen to the user's answers and refine.

The discussion should feel like a refinement session with a product owner, not an interrogation. Ask one or two questions at a time, build on answers, and surface things the user might not have thought of ("What happens when X fails?" / "Should this work for unauthenticated users too?" / "What's the max limit here?").

**Size guardrails:**

A well-scoped feature for this skill produces roughly 3-7 requirements, each with 4-9 acceptance criteria. If the feature is growing beyond this:

- Gently flag it: "This is getting quite broad. A scrum team would struggle to deliver all of this in a single sprint. Can we split it?"
- Suggest a natural split point: "The core CRUD operations feel like one sprint, and the notification/integration layer could be a follow-up feature."
- Let the user decide. If the user says "I know it's big, I want it all in one doc", accept it. The guardrail is advisory, not a hard block.

Signs a feature is too big: more than 7 requirements emerging, acceptance criteria getting deeply nested, multiple independent user personas, or the feature spanning more than one bounded context.

**Decision catalog:**

Throughout the discussion, design choices will come up: "should we paginate or load all?", "280 chars or 500?", "soft delete or hard delete?". For each decision:

1. Note the question
2. Record the chosen answer
3. Capture the reasoning in one sentence

Track decisions as the conversation unfolds, without a pause to log them formally. You'll compile them into the companion file at the end.

### Phase 4: Competitor / standards research (optional)

Before finalizing, ask the user: "Would you like me to research how competitors or industry standards handle [this feature]? This can surface patterns or edge cases we haven't considered."

If yes, do a focused web search on how well-known products in the same space implement similar functionality. Look for:

- Common UX patterns (e.g., how do major platforms handle comment threading?)
- API design conventions (e.g., pagination standards, error response formats)
- Size limits, rate limits, or other constraints that are industry norms
- Features that users commonly expect as table stakes

Present findings as a brief summary during the discussion, not as a formal report. Use them to validate or challenge the decisions made so far. If research surfaces something the user hadn't considered, discuss it and add the resolution to the decision catalog.

### Phase 5: Final check

Before generating output, do a sweep:

1. **Open questions**: Are there any unresolved items? If yes, surface them and resolve them now.
2. **Completeness**: Does every functional area have clear behavior defined for both the happy path and error cases?
3. **Size check**: Final count of requirements and acceptance criteria. Flag if it's outside the 3-7 requirements range.
4. **Consistency**: Do glossary terms match across requirements? Are the same entities referred to the same way?

Tell the user: "I have zero open questions. Ready to generate the requirements document and decision catalog. Go ahead?"

### Phase 6: Generate output

Produce two files:

#### File 1: Requirements document

Use this exact structure. The acceptance criteria format is non-negotiable. It uses WHEN/SHALL/IF/THEN notation for testability.

```markdown
---
title: "<Feature name>"
description: "<One sentence: the outcome the feature delivers, and for whom.>"
domains: [<context-name>, ...]
epic: <E-NN, when the feature comes from a roadmap epic; omit otherwise>
flows: [<flow-id>, ...]  # only when the hub has product/flows/ and the feature changes a process; omit otherwise
---

# Requirements Document

## Introduction

<1 paragraph. What the feature does, who it's for, and any pre-existing infrastructure
or context that's relevant. Written so someone unfamiliar with the product gets the
picture in 30 seconds.>

## Glossary

<Define every domain term, component name, API name, table name, or role that appears
in the requirements below. Use bold for the term, followed by a colon and definition.
Format: **Term**: Definition.>

---

## Requirements

### Requirement N: <Requirement Title>

**User Story:** As a <role>, I want to <action>, so that <benefit>.

#### Acceptance Criteria

1. WHEN <trigger/condition>, THE <component> SHALL <expected behavior>.
2. IF <condition>, THEN THE <component> SHALL <expected behavior>.
3. WHILE <state>, THE <component> SHALL <expected behavior>.
...
```

**Frontmatter rules:**

- The frontmatter is how other commands decide whether to read this brief at all. They see only these fields, so write them for a reader who has not seen the brief.
- `title` is the feature name. `description` is one sentence, specific enough to tell this feature apart from its neighbours.
- Write `title` and `description` in double quotes. Unquoted, a value that contains a colon is not valid YAML, and the brief can then not be listed without reading it. Write a double quote inside the value as `\"`.
- `domains` lists every bounded context the feature touches, using the context names in `domain-model.yml`. Never invent a name. If the feature touches an area with no context yet, ask the user which context it belongs to, and write a name that is not in `domain-model.yml` only after the user confirms it. `ie brief list` and `ie doctor` report that name as unknown until the registry gains it.
- `epic` is the `id` of an epic in the `epics` list of the `product/roadmap.md` frontmatter, written exactly as it appears there. Leave the key out when the feature has no epic.
- `flows` lists the id of each flow under `product/flows/` whose steps the feature adds or changes, or the id of a new flow the feature starts. Leave the key out when the hub has no `product/flows/` or the feature changes no process. For each listed flow, add a `## Flow changes` section after the requirements with one subsection per flow, `### <flow-id>`, naming each step the feature adds, changes or removes, its actor, and the step before and after it. A new flow always has a draft under `features/briefs/<slug>/flows/`, because the draft is the new flow. `ie doctor` warns about a listed flow that is neither under `product/flows/` nor a draft under `features/briefs/<slug>/flows/`.
- When you update an existing brief, update the frontmatter in the same edit if the scope changed. A brief without frontmatter gains it on update.
- The frontmatter records no review state: no `status`, no sign-off and no review questions. A brief on the integration branch is signed off; a brief that exists only on its `brief/<slug>` branch is not. The review happens on the brief's pull request. When a reviewer requests changes there, answer each comment in the brief on its `brief/<slug>` branch.

**Acceptance criteria writing rules:**

- Every criterion is one testable statement. A developer should be able to write exactly one test for it.
- Use WHEN for trigger-based behavior (user does X, system does Y).
- Use IF/THEN for conditional logic and error handling.
- Use WHILE for state-based behavior (during loading, during submission).
- SHALL is the operative word. It means "must". No ambiguity.
- Name the component that's responsible (e.g., THE Comment_API, THE Post_Card).
- Include specific values where they exist (HTTP status codes, character limits, field names).
- Error cases are first-class. They get their own criteria, not footnotes.
- Order: happy path first, then validation errors, then auth errors, then edge cases.

**Glossary rules:**

- Every component, table, API endpoint, or role mentioned in acceptance criteria gets a glossary entry.
- Glossary entries should be specific enough that a developer can identify exactly what's being referred to.
- Use PascalCase_With_Underscores for multi-word component names (e.g., Comment_API, Post_Card, Comments_Table).

**Requirement sizing:**

- Each requirement maps to one user story: one coherent unit of value.
- Aim for 4-9 acceptance criteria per requirement. Fewer than 4 usually means you're missing error cases. More than 9 usually means the requirement should be split.
- The full document should have 3-7 requirements.

#### File 2: Decision catalog

Save as a companion file alongside the requirements document. Format:

```markdown
# Decision Catalog: <Feature Name>

| # | Question | Decision | Rationale |
|---|----------|----------|-----------|
| 1 | <What was the question?> | <What was decided?> | <Why?> |
| 2 | ... | ... | ... |
```

Keep it tight: one row per decision, rationale in one sentence. This is a quick-reference log, not a design document. It exists so future readers understand why things are the way they are.

## File naming and location

Save the files in the product context directory (or a subdirectory the user specifies). Default names:

- `requirements-<feature-kebab-name>.md`
- `decisions-<feature-kebab-name>.md`

Ask the user if they have a preferred location or naming convention.

## Target

- Ask your open questions and wait for the answers before you write the requirements document. The conversation is the point. Never write the document in the turn that receives the request, even when the user asks you to skip the discussion.
- Describe observable behaviour in each acceptance criterion. Leave code, SQL and library calls to the implementation.
- Name a concrete, checkable outcome in each criterion, such as a status code, a field value or a limit.
- Write at least an auth-failure, a validation-failure and a not-found criterion for every API endpoint.
- Record a choice in the decision catalog only when it has a real alternative.
- Write one user story per requirement.
