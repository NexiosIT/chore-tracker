# Intent Engineering SDLC — Glossary

> Framework vocabulary for this Intent Engineering SDLC product-hub — what the codes and terms in command output, specs, and reports mean. For your **product's own domain terms** (entities, business concepts), see [`../product/glossary.md`](../product/glossary.md). Shipped and kept fresh by `ie update` — read it, don't edit it.

## Identifiers

Stable codes the framework issues — handles you reference and type, not abbreviations to decode.

| Code | Means | Example |
|------|-------|---------|
| `UC-NNN` | **Use case** spec — one folder of specification files | `UC-013` |
| `FT-NNN` | **Feature** — a unit of work grouping specs/screens through the lifecycle | `FT-007` |
| `SCR-NNN` | **Screen** spec — one UI screen | `SCR-002` |
| `STD-NNN` | **Standard** — a reusable engineering rule applied during generation | `STD-007` |
| `CP1`–`CP5` | **Checkpoint** — one of the 5 generation stages (see below) | `CP3` |
| `INT-NNN` | **Reasoning-trace entry** — one logged judgment call made during generation | `INT-001` |
| `BR-{n}` | **Bug report** — an in-flight defect-correction record for the *current* feature, before finalize | `BR-3` |
| `ft/NNNN` / `fix/NNNN` / `imp/NNNN` / `bl/NNNN` | **Reservation tag** — not an artifact: a git tag on `origin` claiming a feature, post-delivery-fix, technical-improvement or backlog-item number so two people working at once cannot both take it. Pushed by `ie feature create`, `ie fix create`, `ie imp create` and `ie backlog add`; the push *is* the lock, because `origin` rejects a tag that already exists. On `origin` they are kept forever and never cleaned, which is what makes `git ls-remote --tags origin "ft/*" "fix/*" "imp/*" "bl/*"` a usable list of numbers allocated but not merged yet. A tag in *your* checkout that `origin` has never seen is a different thing — the leftover of a run interrupted between creating the tag and pushing it — and any command that allocates a number deletes it, because it holds a number for you and for nobody else. So allocating a number is not quite read-only on disk, even though it writes nothing into the hub; tags outside those four namespaces are never touched. A number with no tag at all is normal — created offline, or with `--force`. Use cases, screens and standards get no tag by design: they are allocated several times in one sitting, so they rely on the origin scan and the duplicate check instead. | `ft/0123`, `fix/0007`, `imp/0004`, `bl/0012` |

`FT` above is the **default** spelling of the *feature* series only. It is the `naming.feature_prefix` field in your hub's `config.yml`, so a hub whose other tools key on its own work-item convention can write features as `TASK-005` or `proj-FT-006` instead. `UC`, `SCR` and `STD` are fixed — do not change them. What never changes for a feature id either: it is `{prefix}-{number}`, the number is always last, and the width comes from `naming.id_padding`.

If you change the feature prefix, **nothing needs renaming** — ids created before the change keep working, and `ie doctor` lists them so you know the hub carries a mix. The numbering carries on where it left off, too: rename after `FT-047` and your next feature is `TASK-048`, not `TASK-001`. A feature number belongs to the hub rather than to the prefix in front of it, so two features can never end up sharing one — and if two ever do, `ie doctor` and `ie integrity check` both call it a duplicate however it is spelled. One thing does need doing: run **`ie update --code-repos`**, so each service repo's CI picks up the new spelling. Its "is this a feature PR?" check matches branch names against the prefix, and until it is synced that check quietly stops recognising your branches — which switches off the gates that verify a feature PR carries its manifest.

Three spellings may never be chosen for the feature prefix — `STD` (standards), `FIX` (post-delivery corrections) and `IMP` (technical improvements) — nor may it collide with `UC` or `SCR`. Any command that reads the prefix refuses a `config.yml` that breaks that rule, and `ie doctor` reports it as an error, because two series with one prefix cannot be told apart. The comparison is exact, so case matters: `uc` is a different prefix from `UC`, and legal — but a pair that differs only in case is a trap for whoever reads the folder listing next, so pick something clearly distinct.

## The five generation checkpoints (CP1–CP5)

Code generation runs in 5 concern-based stages; tests gate each one.

| Stage | Concern |
|-------|---------|
| **CP1** | Domain model & contracts (entities, DTOs, OpenAPI/AsyncAPI) |
| **CP2** | Domain logic & rules (business logic, state machines) |
| **CP3** | Interface layer (endpoints, event publishers/consumers) |
| **CP4** | Presentation (UI components, design tokens) |
| **CP5** | Deployment topology (infra, config) |

## Core concepts

| Term | Means |
|------|-------|
| **Product-hub** | The repo holding specs, standards, and product context — the source of truth. Created by `ie init`. |
| **Code-repo** | A service's own repo, where `/ie/generate` runs and code is produced. |
| **Sandbox-hub** | A third repo shape: a *mockup* hub. It turns a feature brief into throwaway screens built from your product's **real** components, so a feature can be looked at before it is specified. It holds no specs, generates no code, and is optional. Created with `ie sandbox-hub init <name>`, and linked to this hub — from its side, once — with `ie sandbox-hub init . --hub <path-to-this-hub>`. This hub stores nothing about it. |
| **Mockup runtime** | Which front-end framework a sandbox-hub's screens are written in. Settled when the sandbox is created, and fixed for that instance's life. |
| **Brief** | `features/briefs/<slug>/` — a feature's requirements before it is shaped: `requirements.md` and `decisions.md`, written by `/ie/extract-brief`. `requirements.md` opens with frontmatter (`title`, `description`, `domains`, optional `epic`) so commands can decide which briefs to read without opening them. `ie brief list` prints that frontmatter for every brief, with any domain that is not in `domain-model.yml` and any epic that is not in the roadmap. `ie doctor` warns about a brief that has no frontmatter, an unknown domain or an unknown epic. |
| **Brief branch** | `brief/<slug>` — the branch a brief is written on, cut by `ie brief start <slug>`. Merging it into the integration branch through an approved pull request is the brief's sign-off: a brief on the integration branch is signed off, one only on its brief branch is not. The review happens on that pull request; the brief's frontmatter records no review state. |
| **Backlog item** | `backlog/BL-NNN.yml` — an idea or a chore captured in the hub before anyone writes a brief for it: `id`, `title`, `type` (`idea` or `chore`), `raised_by` (a git host login), `why`, and optionally a roadmap `epic` and `links`. Optional: a hub that holds at least one item has opted in, and only then does `/ie/extract-brief` read, match or suggest items. Added by `/ie/backlog` through `ie backlog add`, which reserves the number on origin with a `bl/NNNN` tag; a number is never reused. `/ie/extract-brief` may start a brief from one, and `ie backlog remove` deletes it on that brief's `brief/<slug>` branch, so it leaves the backlog when the brief is signed off. |
| **Backlog branch** | `backlog/BL-NNN` — the branch `ie backlog add` commits items on, named after the first item it holds. Items reach the integration branch through an approved pull request from it. |
| **Taken** | A backlog item deleted on an open `brief/*` branch, local or on origin: a brief is being written from it. `ie backlog list --branches` names that branch, and `/ie/extract-brief` does not offer the item. A brief branch abandoned without merging keeps its item taken until the branch is deleted, locally and on origin. |
| **Mockup feedback** | `features/briefs/<slug>/mockup-feedback.md` — what a mockup review sent back. It arrives from a sandbox-hub beside the brief it responds to, carrying a header naming which sandbox produced it and at what revision. Advisory input for `/ie/shape-feature`; prose only, and it touches no spec, status or hash. |
| **Draft screen spec** | `features/briefs/<slug>/draft-screens.md` — the mockup step's by-product, folded back beside the feedback. It records the screens the analyst worked out on screen, in prose, so that work reaches shaping instead of being thrown away with the mockup. Advisory like the feedback, and with it the *only* pair a sandbox-hub ever writes here. Where a manifest points at it, `/ie/generate` reads it at CP4 as context for intent — never as a spec, and never as source to copy. |
| **Spec** | A use case (`UC-NNN`): a *folder* of files (Gherkin, OpenAPI, Mermaid, …), not a single document. |
| **Contract** | A spec's request/response/event schema — `contract.openapi.yml` (REST) and/or `contract.asyncapi.yml` (events). |
| **Standard** (`STD`) | A reusable engineering rule the generator must apply (naming, error handling, …). |
| **Constraint** | A *feature-specific* non-functional requirement (`constraints.md`), applied like a standard. |
| **Manifest** | The context package handed from hub to code-repo for generation — file *paths*, not content (`.flowforge/manifest.yml`). |
| **`.flowforge/`** | The protocol directory in a code-repo (manifest, checkpoint results, handover, reasoning trace). |
| **Wave** | A build-order group — services generated together because their dependencies are already built (cross-repo handover). |
| **Iron Dome** | The spec-integrity gate — confirms specs, and producer↔consumer event contracts, are consistent before they're trusted. |
| **Readiness** | A single-spec *structural* score (0–100) with a grade: **ready** · **needs work** · **blocked**. A failed blocking check caps the grade below `ready` regardless of points; the score never verifies cross-spec consistency, ownership, or flow validity — that is what review is for. |
| **Risk tier** | `elevated` or `standard` — how bad a miss on this *feature* would be. Computed from three machine signals plus four recorded judgment answers (DDL/privilege, authz/session, tenant boundary, sensitive data) at shape-feature Step 3; written only by `ie feature risk --record`. Selects ceremony depth: `elevated` = a fresh-context review pass per spec and the full walk; `standard` = lint + checklist and a targeted surface walk — never no walk. Distinct from the **promotion tier**: promotion is per-*pattern* trust earned by history ("how much do we trust the generator on this shape of work"); risk is per-*feature* consequence ("how bad is a miss here"). A `proven` pattern can still carry an `elevated` feature. |
| **Reasoning trace** | The log of judgment calls (`INT-NNN`) the generator records — interpretations, assumptions, deviations. |
| **Bug report** (`BR`) | A scoped, in-flight defect-correction record — filed with `/ie/bug-report`, lives in the hub at `bug-reports/<service>/<feature-id>/BR-{n}.yml`. No new FT-id — it operates on the current feature before finalize. The report and its spec corrections are written on a per-feature bug-report branch (`bug-reports/<feature-id>`, prefix from `config.yml` `bug_report_branch_prefix`), cut from the hub commit the feature was handed over from and kept off the integration branch until delivery so they are reviewable; `deliver` checks they landed there. |
| **Finalize** | Tidying up the generation state in a code repo before its PR: `ie generation finalize` moves `.flowforge/manifest.yml`, the checkpoint results and the generation logs into `.flowforge/archive/{feature-id}/`, so the integration branch never carries live generation state. Run **in that repo, on its feature branch**; it resolves no hub and needs no network. `--revert` puts everything back if review sends the work back (refused once the finalize commit has merged). |
| **Delivery record** | The one hub commit that says a feature shipped: `status: delivered`, each owned UC sealed to `implemented` (except one another in-flight feature also changes, which waits for that feature), the promotion pattern credited, and, when the hub has a data model, the feature's tables marked `delivered`. Written by `ie feature deliver <id>` (uncommitted where you stand, or committed on a `delivery/{feature-id}` branch with `--on-branch` when the hub's integration branch is protected), and only after every touched repo carries its finalized archive on the integration branch — that archive is what proves the code both finalized *and* merged. |
| **Data model** | `product/erd.mermaid` — the hub's table-level data model, one Mermaid ER diagram. Optional. Every table carries one status class: `delivered` (on the integration branch with its feature delivered), `built` (a feature in flight adds or changes it), `planned` (on the roadmap, nothing built yet) or `external` (a person or system outside the platform). The first comment line holds the date it was last drawn. `ie erd init` draws the first one from the linked services' SQL migrations and the domain registry; `ie erd mark-built` (run by `/ie/shape-feature`) marks a feature's tables `built`; `ie feature deliver` marks them `delivered` and removes the ones the feature dropped. It is not checked against the code, so it can disagree with it. `ie doctor` warns about a table with no status class. |
| **Flow** | `product/flows/<flow-id>.bpmn` — one process the application runs across use cases, actors and features, such as onboarding a client, as a BPMN 2.0 file. Optional: a hub that holds at least one flow has opted in, and only then do commands read, offer or check flows. Each step may carry `ie:epic`, `ie:domains` and `ie:useCases`, naming its roadmap epic, its domain contexts and its use cases. A flow describes the application as it is today, so it changes only when a delivery changes it, through `/ie/flow`. Agents write only the process; `ie flow layout` writes the diagram coordinates. `ie doctor` warns about a flow that does not parse and a step that names an unknown epic, context or use case. |
| **Draft flow** | `features/briefs/<slug>/flows/<flow-id>.bpmn` — a brief's proposed change to a flow, or a new flow, written by `/ie/extract-brief` so the reviewer sees the change as a diagram in the brief's pull request. The brief's `flows` frontmatter lists the flows it changes. `/ie/flow <flow-id> --from-brief <slug>` applies it after delivery and deletes it. |
| **Epic brief** | `product/epics/E-NN-<slug>.md`: one short description of one roadmap epic, detailed enough for architects to estimate it. Written by `/ie/epic-brief E-NN`. Its frontmatter holds `epic`, `title`, `integrations`, `depends_on` and, once an architect estimates it, an `estimate` block. Its body holds thirteen sections, from Outcome to Estimate. Each requirement is one line, `- R<n> <text>`, with no acceptance criteria. `/ie/extract-brief` reads it as background and never writes it. |
| **Target flow** | `product/epics/flows/<flow-id>.bpmn`: a process as it will be after its epics are delivered, drawn by `/ie/flow <flow-id> --target` before any feature exists. Same BPMN rules as a flow. Each step names its epics in `ie:epic`, a space-separated list. It may share its id with a flow under `product/flows/`, and then shows the target version of that process. Delivery never changes or deletes it. |
| **Ready to estimate** | An epic brief that passes the ready check. Every section is present and not empty, and `None.` counts as content. The brief has at least one requirement. The epic, each `depends_on` epic and each integration exist. Open unknowns do not fail the check. `/ie/epic-brief` runs the check at its end, `ie doctor` warns about each brief that fails it, and `ie estimate` shows the status. |
| **Stale estimate** | An estimate whose epic brief changed after it was recorded, or whose brief now fails the ready check. A change is anything outside the `estimate` block changed after the last commit that changed that block, uncommitted changes included. A stale estimate still counts in the total, so the total never looks smaller than the project. `ie estimate` names each stale epic so an architect checks it again. |
| **Size table** | The `estimation.sizes` block in `config.yml`: a `[min, max]` range of person-days for each size `XS`, `S`, `M`, `L` and `XL`. Each product fills its own values. `ie estimate` turns each estimated size into person-days with it. When it holds no values, `ie estimate` prints sizes and counts only. |
| **Database diagram** | `features/active/<id>/specification/db-diagram.mermaid` — one feature's schema change, drawn by the `dba` worker during `/ie/shape-feature`: the database as its migrations leave it, with the feature's changes marked. A new table or column is outlined green and marked `NEW`, a changed one amber and `CHANGED`, a dropped one red and `DROPPED`, each citing the use case that implies it. It opens with `%% feature: <id>`, `%% base: <migration base>` and `%% delta: <use case ids>`; `%% delta: none` marks a feature that changes no table. The hub's pre-push check refuses a push of a feature in `shaping` or `specified` whose diagram is missing, names another feature, or does not parse. |
| **Archive** (a feature) | The end of a feature's life in the hub: `ie feature archive <id>` moves its folder and its bug reports to `features/archive/`. Run **in the hub**, refused unless the feature is `delivered`, and the daily cron does it once a delivered feature has been quiet for the window in `config.yml`. Archived features stay **readable** — `ie feature status`, `ie integrity check` and blast-radius analysis all resolve them — and are never written again. |
| **FIX** | The artifact a post-delivery defect produces: `FIX-NNN`, stored under `features/active/post-delivery-defect/`, carrying the same manifest shape as a feature — so handover, generation, finalize and delivery are the same commands. Created by `ie fix create` (usually through `/ie/fix-defect`), it records which delivery introduced the defect and debits that delivery's promotion pattern (an `IMP` parent has none, so nothing is debited). It skips shaping, and it is never a reopened feature. |
| **Post-delivery fix rail** | Where a defect in *delivered* work goes: `/ie/fix-defect`, run from the hub, files a `FIX` artifact against the delivered feature — so a shipped feature is corrected without ever being reopened. `ie bug-report` is its in-flight counterpart, and covers only work before finalize. |
| **IMP** | The artifact a technical change produces: `IMP-NNN`, stored under `features/active/technical-improvement/`, carrying the same manifest shape as a feature. Created by `ie imp create` (usually through `/ie/improve`), it states `current_state` and `target_state`, lists the `affected_repos` it spans, and records every hub fact it corrected with the text that fact used to carry. It skips shaping, records no risk tier, and neither credits nor debits promotion. It takes one of three **shapes** (see *IMP shape*). |
| **IMP shape** | Which of three kinds of technical change an `IMP` is, in its `shape` field (absent means *substitution*). **substitution** retires technology A for B, witnessed by the old thing being gone. **optimization** moves a named, measured metric past a bar, carrying a `metric` witness (metric, measurement, baseline, better, target) — the witness is a numeric delta. **qualitative** flips a structural invariant from failing to holding, carrying an `invariant` witness (property, check) — the witness is a check re-run. Every shape names one observation that separates after from before; an `IMP` that can name none is not an `IMP`. |
| **Improvement rail** | Where a technical change goes when no product behaviour changes with it — retiring a technology, moving a measured metric, or flipping a structural invariant (see *IMP shape*): `/ie/improve` in the hub files one `IMP` across every affected repo, and `/ie/apply-improvement` applies it in each. `/ie/generate` refuses an `IMP` by name. The test for this rail against the feature rail is whether an acceptance scenario moves: if one does, it is a feature. |
| **Spec touch** | A spec edit that changes a representation the technology owns — a contract field type, a datetime literal inside a scenario step, prose naming an engine — without changing what the product does. Recorded on an `IMP` under `spec_touches`, deliberately not `spec_changes`: the latter feeds readiness, risk, the delivery seal and promotion credit, and a technology change earns none of them. A touch scopes Iron Dome, re-registers the spec hash, and is hash-pinned at handover like a changed spec; it is not scored, sealed or credited. |
| **Blast radius** | The `ie integrity check` result reviewed before regenerating after a bug-report spec fix — confirms the edit doesn't conflict with other in-flight specs (Iron Dome). |
| **Metrics trail** | `metrics.yml` beside a feature's, fix's or improvement's manifest: its stage history, each failed check run, review verdicts, the model tokens of each agent workflow run on it and, after delivery, each repo's generation counts and tokens. An entry an agent made also names the AI model it ran on. Written only by the commands that observe each fact, never by hand. Names no person, and is not a measure of anyone's productivity. Viewed with `ie metrics render`. |
| **Calendar time** | How stage durations are measured on the metrics trail: wall-clock time between status changes, **including waiting**. Not effort, and not an input to estimates or planning — which is why it is always shown beside the quality counts for the same feature. |
| **Distinct findings** | On the metrics trail, the set of different things a check failed on, added up from its failed runs (a readiness rule id, an integrity finding id, or for `ie spec validate` the failing file with its use case folder). Each failed run is kept with what it failed on, so the page can show which run found what. Read it before the failed-run count: rerunning a check raises the run count, not the findings. |
| **Rework round** | A recorded review verdict that sent the work back (`revise` or `reject`), on a checkpoint, a final-review lens, a use case or an improvement's verification. Counted from the recorded verdicts, never stored. Shown separately for spec reviews and for generation. |
| **Escape** | A defect that got past delivery: counted for a feature once for every `FIX` whose `corrects` names it. Derived when the trail is read, never stored. |
| **`unavailable`** | On the metrics trail, a value that could not be observed — written as `unavailable`, never as zero or an estimate. A feature already in progress when the trail started shows its totals as `unavailable`, because their beginning was never recorded. |

## Lifecycles

The states an artifact moves through. The canonical states live in the hub's `CLAUDE.md` (Naming + Status); shown here for orientation.

| Lifecycle | States, in order |
|-----------|------------------|
| **Spec status** | `draft` → `ready` → `implemented` → `touched` → `deprecated` |
| **Feature status** | `shaping` → `specified` → `approved` → `delivered` → `archived` — the phase between handover and delivery is *observed* from git (`ie feature status --repos`), never stored |
| **Promotion tier** | `New` (incremental, every checkpoint) → `Established` (3 successes, reduced) → `Proven` (5+, single-pass eligible) — how far a generation strategy is trusted to run unattended. Orthogonal to the per-feature **risk tier** (consequence): trust never erases consequence. |
| **Bug-report status** | `open` → `classified` → `spec_updated` → `blast_radius_clear` → `regenerating` → `resolved`. Two short-circuits from `classified`: → `routed_elsewhere` (Standard Gap, no spec/code change) or → `not_a_bug` (also reachable from `open` — closer inspection found no actual defect). |

## Spec formats

A spec folder carries each concern in its native format, so standard tooling works.

| File | Format | Captures |
|------|--------|----------|
| `scenarios.feature` | Gherkin | Behavior (Given / When / Then) |
| `contract.openapi.yml` | OpenAPI | Sync REST request / response / errors |
| `contract.asyncapi.yml` | AsyncAPI | Async event channels / messages |
| `flow.mermaid` | Mermaid | State machines, sequence diagrams |

## Design system

The single source for UI design values that every screen spec and generated frontend binds to by name. Seed it before shaping UI features — see [`DESIGN-SYSTEM.md`](DESIGN-SYSTEM.md).

| Term | Means |
|------|-------|
| **Design system** | The hub's `design-system/` artifact — the canonical token vocabulary UI specs consume. |
| **Design token** | A named design value referenced in component trees (`color-primary`, `spacing-md`) instead of a hardcoded `#0B5CD8` / `16px`. |
| **Primitive** | A raw value (a palette colour, a spacing step). Lives **only** in `_base` — the brand must not fork across apps. |
| **Semantic token** | A meaning mapped to a primitive (`color-primary → brand-blue-600`). Referenced by name; may be overridden per app. |
| **`_base`** | The shared brand core (`design-system/_base/`) — owns primitives and a component's contract; every app resolves against it. |
| **Theme** | A per-app layer (`design-system/<app>/`) that adds or overrides *semantic* tokens for one app's look. |
| **Component inventory** | `design-system/<layer>/inventory.yml` — what each component/primitive a screen may reference *is*: its realization (`impl`), behaviour contract and variants. On a vendor-bound layer it is an **override layer**, not a catalog: only what the vendor cannot decide. |
| **Realization pattern** | The standard an inventory entry's `impl.pattern` names — how a component of this kind is *produced* where the product has not built one yet, recorded with `ie design-system set-impl --pattern`. It is the pointer a generating agent binds to instead of guessing, and it never overrides copying the component the front-end application already has. The standard is authored in this hub: what a product's components are generated from is a product fact, so no framework field names a tool. Its document travels with the reference when the design system is mirrored into a mockup hub. |
| **Vendor binding** | `design-system/<layer>/vendor.yml` — three committed facts (which MCP server, which component platform, which framework) that bind a layer to a client's own design system. Optional; a hub that hand-authors its design system has none. |
| **Vendor lock** | `design-system/<layer>/vendor.lock.json` — the vendor's catalog, **generated** by `ie design-system vendor refresh` and never hand-edited: components with their prop contracts, the package and export to import, per-package versions, vendor tokens. Committed, so every check and every generation reads it offline. Resolution reads your inventory first and the lock last, which is why a refresh can rewrite it without ever conflicting with a decision you made. |

## Domain registry

The single source for your **entity vocabulary** that every spec's `entities[]` binds to by name — the domain-side sibling of the design system. Populate it with `ie domain init` (or author it during bootstrap); list it for shaping with `ie domain entities`.

| Term | Means |
|------|-------|
| **Domain registry** | The hub's `domain-model.yml` — the canonical, context-scoped list of domain entities. Spec `entities[]` values resolve against it; unresolved names warn (then block once curated). |
| **Bounded context** | A named area of the domain, **equal to a UC's `domain` field** (and its `specs/{domain}/` folder). Each entity belongs to exactly one context. |
| **Canonical identity** | An entity's resolved `context/entity` name (e.g. `billing/invoice`). What Iron Dome scopes on — aliases and typos collapse to it; same-name-different-context entities stay distinct. |
| **Alias** | A synonym or shortening that resolves to a canonical entity (`bill` → `invoice`). Case/plural variants need no alias — the registry normalizes those. |
| **Projection** (`projection_of`) | A context's own read-model copy of another context's entity (`reporting/invoice` `projection_of: billing/invoice`). Declares a legitimate cross-context copy; Iron Dome pairs the projection with its source. |
| **`status: curated`** | The per-hub flag that flips registry resolution from warn to **block** and activates UC-granularity enforcement. Set it in a reviewed PR once the registry is complete. |

## Runtimes

How this hub's commands, skills, and operator notes are made available to an AI coding tool. One runtime-neutral source is **compiled** per runtime, so every change is authored once and lands in all of them.

| Term | Means |
|------|-------|
| **Runtime** | A supported AI coding tool — `claude` (Claude Code), `copilot` (GitHub Copilot CLI), or `generic` (the cross-tool `AGENTS.md`). The hub's enabled set lives in `config.yml` (`runtimes:`, default `[claude]`). |
| **Capability** | A runtime-neutral unit of behaviour the scaffold ships — an **action** (`/ie/*` command), a **worker** (an internally-dispatched subagent like review-checkpoint), a **skill** (auto-triggered), an **instruction** (the operator anchor), a **hook** (the pre-push gate), or **settings**. |
| **Adapter** | The per-runtime translator that emits a capability into that tool's native files — Claude → `.claude/` + `CLAUDE.md`, Copilot → `.github/agents/*.agent.md` + `AGENTS.md`, generic → `AGENTS.md`. |
| **`AGENTS.md`** | The canonical, cross-tool operator anchor (read natively by Copilot and ~30 other tools). Claude reads its own `CLAUDE.md`; both are rendered from one source. |
| **Invocation** | How an operator runs an action per runtime, under one shared name: Claude `/ie/<verb>` · Copilot `copilot --agent ie-<verb>` (or the `/agent` picker). |
| **Parity / waiver** | The guarantee (`ie runtime verify`) that every capability emits for every enabled runtime — or carries a reviewed `unsupported` waiver (e.g. `generic` has no command surface). No silent gaps. |
| **Runtime commands** | `ie runtime list` (show the parity matrix) · `verify` (the gate) · `add <runtime>` / `remove <runtime>` (enable/disable + (de)compile its assets). |
