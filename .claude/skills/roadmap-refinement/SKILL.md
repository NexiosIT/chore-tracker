---
name: roadmap-refinement
description: >
  Roadmap refinement: epic-level planning that sketches the features of the next iteration.
  Use when the user wants to decide what to build next or to reprioritize epics.
  Not for one feature's requirements (use feature-requirements).
---

# Roadmap Refinement Skill

## Purpose

This skill runs an interactive roadmap refinement session. This is the strategic planning layer that decides *what* to build next and *why*. It works at the epic level: each epic groups related features that together deliver a meaningful product outcome.

The output is a refined roadmap with two distinct layers:

1. **Detailed scope**: The epics and features you're about to build. Features are sketched with enough clarity to feed into feature briefing (acceptance criteria, business rules). This is the contract for the current planning horizon.
2. **Future context**: What's coming later, kept deliberately abstract but captured as *constraints on the detailed scope*. Future work actively influences today's design decisions.

The key principle: **the future constrains the present without defining it.** "We'll add Redis caching later" becomes "caching will be introduced for recipe search in a future iteration. Current implementation must keep query logic behind a service boundary so the caching layer can be inserted without touching endpoints." Actionable without being prescriptive.

## When to Use

Use this skill when you need to decide what to build next. Typical moments: starting a new iteration, completing a batch of features and needing to pick the next ones, quarterly planning, or when new information (customer feedback, competitor moves, technical discoveries) changes priorities.

This skill does NOT produce feature briefs (that's the feature-requirements skill), spec-level artifacts (that's use-case-authoring), or screen specs (that's screen-authoring). It produces the *input* for feature briefing: feature sketches with enough context for the next step.

## Mapping Mode

Mapping mode lists every epic of a project before any feature is planned. A business analyst uses it at the start of a large project. Architects then estimate the project from one epic brief per epic.

In mapping mode:

- Run only "Identifying new epics" in Phase 3. Skip Phases 1, 2, 4, 5 and 6.
- Accept a roadmap that holds only themes. Turn each theme into one or more candidate epics.
- Write every epic the engineer names. Apply no limit to the count of epics or to the epics in the active band.
- Give each new epic the `later` priority band and no feature list.
- Do not sketch features and do not write future-context constraints.

A normal session keeps every limit below. When every epic is in the `later` band, ask the engineer which epics move to the active band. Then sketch the features of those epics in Phase 4.

## The Three-Level Pipeline

This skill is the first level in a three-level refinement pipeline:

| Level | Skill | Output | Audience | Strictness |
|-------|-------|--------|----------|------------|
| **1. Roadmap** | roadmap-refinement (this) | Epics + feature sketches + future constraints | Product owner | Directional |
| **2. Brief** | feature-requirements | Strict feature brief with acceptance criteria + business rules | Functional analyst | Contractual |
| **3. Shape** | use-case-authoring / screen-authoring | Gherkin, OpenAPI, screen specs | Developers + AI generation | Implementation-ready |

**The contract between levels:**
- The brief (level 2) is strict on *intent*. Acceptance criteria are business outcomes.
- The shaped spec (level 3) is authoritative on *edge cases*. Shaping discovers implementation details.
- Shaping can add sub-rules that refine a brief's acceptance criterion, but **cannot contradict or drop one** without going back to the brief owner.
- A traceability matrix in the feature manifest maps each acceptance criterion to the UC(s) that implement it. If a criterion has no UC, shaping is incomplete.

## Inputs

Before starting, establish context with the engineer:

- **Product roadmap**: The current roadmap artifact (any format: markdown, YAML frontmatter, plain text).
- **Product mission/vision**: To keep refinements aligned with the product direction.
- **Delivered features**: What's been shipped since the last refinement. This is the most important input. It tells you what's done and reveals gaps.
- **Active features**: Features currently being briefed, shaped, or built.
- **Planning horizon**: Ask the engineer: *"How many features do you want to detail for the next iteration? And what future work should I keep in mind as constraints on today's decisions?"*

The planning horizon question is critical. It calibrates the session: 2 features means a focused session; 8 features means a broader sweep. The future-context layer covers the next 2-3 iterations beyond the detailed scope. Anything further out is noise.

## Workflow

### Phase 1: Status Review

**Goal:** Build a clear picture of where things stand. Don't let the engineer start planning until they see reality.

1. **Walk delivered work.** For each completed feature, note what was built and when. Look for patterns: are certain areas getting more attention than others? Are there gaps between what was planned and what was shipped?

2. **Walk active work.** For features in progress, note their current state: briefing, shaping, building, or blocked. List the briefs with `ie brief list --group-by epic --json`. It returns each brief's title, description, domains and epic without the brief itself, and the `feature` that took a brief once it has been shaped. A brief with a `feature` is that feature's work, so count it once, under the feature. Open a brief only when those fields match the work in hand. A brief listed without frontmatter carries no epic, so read its `requirements.md` in full to place it.

3. **Produce an epic status board.** For each epic, show progress: how many features are delivered, active, or planned. Flag:
   - **Completed epics**: All features delivered. Suggest moving to Done.
   - **Stale epics**: In the active priority band but no feature activity in a while.
   - **Orphan features**: Delivered or active features that don't belong to any epic.
   - **Status drift**: Features whose actual state doesn't match the roadmap's record.

4. **Present the board.** Ask: *"Here's where things stand. What would you like to focus on?"* Offer options: identify next epics, update existing epics, reprioritize, close completed epics, or all of the above.

**[PAUSE: confirm the engineer has reviewed the status and chosen a direction.]**

---

### Phase 2: Scope Sizing

**Goal:** Calibrate the session before diving into planning.

1. Ask: *"How many features are you planning to detail for this iteration? (Rough estimate is fine: 2? 5? 8?)"*
2. Ask: *"What's coming later that I should keep in mind? Any known future work that constrains how we design the current features?"*
   - Probe for architectural constraints: "Will the tech stack change? New integrations planned? Scaling milestones?"
   - Probe for product constraints: "Any features you know you'll build that depend on getting the current ones right?"
3. Record the planning horizon: detailed scope (N features) + future context (2-3 iterations out).

This sizing drives Phase 3: the detailed scope gets feature sketches with acceptance criteria; the future context gets constraint statements.

---

### Phase 3: Epic Identification & Refinement

**Goal:** Help the engineer identify the next epics and refine existing ones. This is the core of the session.

#### Identifying new epics

Propose candidates based on evidence, in addition to the question "what do you want to build next?":

1. **Scan for signals.** Look at:
   - Product mission: are there success metrics not yet addressed by any epic?
   - Orphan features: do delivered features cluster around an unrecognized epic?
   - Gaps between epics: what user journeys are incomplete?
   - Active features: do any hint at a larger epic not yet named?

2. **Present 3-5 candidate epics.** For each, provide:
   - A concrete outcome title (something you could demo, not vague)
   - Why now (what signal suggests this is timely)
   - A rough feature sketch (2-4 bullet points of likely features)
   - Suggested priority

3. **Let the engineer pick.** They might take candidates as-is, modify them, or propose something entirely different. For each accepted epic, sketch the feature breakdown together.

#### Updating existing epics

For each epic the engineer selects:
- Update feature status from delivered/active records
- Add or remove features if scope changed
- Sharpen the outcome title if it's too vague to demo
- Capture lessons from shipped features

#### Reprioritizing

Walk through epics and ask about priority changes. Rules:
- Maximum 3 epics in the active priority band. More means nothing is truly prioritized
- Maximum 10 epics in the `now` and `next` bands together. The `later` band has no limit.
- Do not ask to park or merge an epic because the `later` band is long.
- When every epic is in the `later` band, ask the engineer which epics move to the active band.
- Moving an epic to active should trigger: *"Which features in this epic should we brief first?"*

---

### Phase 4: Feature Sketches (Detailed Scope)

**Goal:** For each feature in the detailed scope, produce a sketch with enough substance to feed into feature briefing.

For every feature the engineer identified as "next to build," produce a **feature sketch** with these sections:

#### Feature Sketch Format

```
## Feature: {name}

**Epic:** {parent epic}
**Why now:** {1-2 sentences: what makes this timely}

### What it does
{2-4 sentences describing the feature's behavior from the user's perspective.
Describe what the user experiences, not implementation details.}

### Acceptance Criteria
{3-7 criteria. Each is a business outcome, testable by a non-technical stakeholder.
Format: "Given [context], when [action], then [expected outcome]"
These are STRICT. They become the contract for feature briefing.}

### Business Rules
{Rules that constrain behavior. Things the system must enforce regardless of
how it's implemented. E.g., "A subscriber can only review a recipe that was
delivered to them" or "Rating must be 1-5 inclusive."}

### Future-Context Constraints
{Constraints from future work that affect this feature's design. Each constraint
names: (1) the deferred architectural decision, (2) the constraint it imposes
on the current implementation. E.g., "Caching will be introduced for search in
a future iteration. Current implementation must keep query logic behind a service
boundary so the caching layer can be inserted without touching endpoints."}

### Open Questions
{Anything unresolved that the feature briefing step needs to answer.
Don't pretend you know what you don't.}
```

**Quality checks on feature sketches:**
- Acceptance criteria are business outcomes, not implementation details. "Returns 201" is implementation; "review is saved and visible to other subscribers" is a business outcome.
- Business rules are constraints, not behaviors. "Users must be authenticated" is a constraint; "show login modal" is behavior.
- Future-context constraints are actionable. Each one names a concrete design constraint, not a vague "keep it flexible."
- Open questions are honest. If you're unsure about something, flag it rather than guessing.

---

### Phase 5: Future Context Layer

**Goal:** Capture upcoming work as explicit constraints on the detailed scope, not as detailed plans.

For each future epic or theme the engineer mentioned:

#### Future Context Format

```
## Future: {theme/epic name}

**Horizon:** {next iteration / 2-3 iterations out}
**Confidence:** {high (we're committed) / medium (likely) / low (exploring)}

### What it means for current work
{1-3 sentences describing how this future work constrains today's design decisions.
Be specific about the constraint, abstract about the solution.}

### Deferred Decisions
{Architectural or product decisions that are explicitly NOT being made now, but
that the current implementation must not foreclose. Each decision names what's
being deferred and what boundary the current work must respect.}
```

The future context layer is NOT a backlog. It's a set of constraints. The difference:
- Backlog: "Add Redis caching for search" (a task to do later)
- Future context: "Caching will be introduced. Keep query logic behind a service boundary." (a constraint on today)

---

### Phase 6: Review & Confirm

**Goal:** Present the complete refinement output and get confirmation before writing.

1. **Show the change summary:** New epics, updated epics, completed epics, feature sketches produced, future constraints captured.
2. **Show the updated roadmap** (complete, not just diffs).
3. **Ask for confirmation** before writing any files.
4. **After writing**, present next actions:
   - Which features are ready for briefing (point to feature-requirements skill / command)
   - Which open questions need resolution before briefing can start

## Output

A refined roadmap containing:

- **Epic inventory** organized by priority, with feature status tracking
- **Feature sketches** for the detailed scope (N features, each with acceptance criteria, business rules, future-context constraints, and open questions)
- **Future context layer** for upcoming work (constraints on current design, deferred decisions)
- **"Ready to brief" list** pointing to the next feature-requirements step

## Edge Cases

- **First refinement (no roadmap exists yet).** Start from scratch. Phase 1 is skipped (no status to review). Phase 2 is extra important. Calibrate the planning horizon carefully. Phase 3 starts with the mission/vision as the only input.

- **Major pivot (roadmap needs fundamental rework).** Don't try to preserve the existing structure. Start from Phase 2 with a clean slate, but use the old roadmap as input for what's been learned.

- **Single feature refinement.** The engineer wants to detail just one feature, not a full roadmap session. Skip Phase 1-3, go straight to Phase 4 (feature sketch) with Phase 5 (future context) as a brief coda.

- **Mapping a whole project.** The engineer wants to list every epic of a large project before any feature exists. Use mapping mode (see above).

- **Too many features requested.** If the engineer wants to detail more than 8 features, push back: *"That's a lot to detail in one session. Can we identify the top 3-5 and sketch the rest more lightly? The others can be detailed in the next refinement."* More than 8 detailed feature sketches usually means the planning horizon is too wide.

- **No future context.** If the engineer says nothing is planned beyond the current scope, note it but probe: *"Is there anything you know you'll eventually need (scaling, new integrations, regulatory requirements), even if it's not planned? Those constraints help us avoid painting ourselves into a corner."*
