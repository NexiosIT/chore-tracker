# Intent Engineering SDLC — End-to-End Walkthrough

> One small feature, start to finish. This is the **narrative** companion to [`WORKFLOW.md`](WORKFLOW.md) (the reference map) — read this once to see how the phases connect, then use WORKFLOW.md for the per-phase detail. New vocabulary (`CP3`, `UC-013`, *readiness*, *Iron Dome*)? → [`GLOSSARY.md`](GLOSSARY.md). Kept fresh by `ie update` — don't edit this file.

We'll build one feature: **"submit a support ticket"** — a customer files a ticket, the system stores it and returns its ID. One actor-goal, one operation: a textbook single use case.

---

## Before you start: which runtime?

Every `/ie/<verb>` below is the **Claude Code** gesture. If your hub also targets GitHub Copilot CLI, the same verb is `copilot --agent ie-<verb>`:

| You'll see | Claude Code | GitHub Copilot CLI |
|------------|-------------|--------------------|
| `/ie/shape-feature` | `/ie/shape-feature` | `copilot --agent ie-shape-feature` |

The walkthrough writes the Claude gesture; substitute the Copilot one if that's your runtime. Both run the *same* compiled workflow — including the interactive **gates** where the agent stops and asks you to confirm before continuing. If an agent ever runs end-to-end without pausing, that's a bug, not a feature — the gates are there to catch wrong assumptions early.

---

## 0a · Bootstrap the hub (~10 min)

```bash
ie init acme-support           # scaffold a product-hub, git-init it, activate the pre-push gate
cd acme-support
```

Then, in the hub, run the bootstrap conversation:

```
/ie/bootstrap
```

It asks for your product mission, tech stack, domains, and registers your services. Answer from what you know; gaps become warnings, not blocks. When it finishes you have `product/`, `catalog.yml`, a domain registry, and a services list.

> **Enabling Copilot?** Do it now: `ie runtime add copilot`, then commit the emitted `.github/agents/` + `AGENTS.md`. The command prints a reminder to also run `ie update --code-repos` once you have linked services (next step) — that's what compiles `ie-generate` into each service repo so `copilot --agent ie-generate` resolves there.

## 0b · Make a service generation-ready (~20 min)

IE doesn't scaffold service code — you do, with the native tool, then adopt it. Full runbook in [`WORKFLOW.md`](WORKFLOW.md#greenfield-make-a-service-generation-ready-phase-0b). Short version:

```bash
# scaffold (e.g. Spring Initializr / create-next-app), create the remote, push, then:
ie service init support-api --path ../support-api                 # adopt: creates .flowforge/, registers the repo
# In the service repo: author its own build/test workflow (.github/workflows/ci.yml)
# and make its checks required on the integration branch. IE ships no build/test CI;
# see the repo's docs/WORKFLOW.md → "Your own build and test CI".
ie service init support-api --path ../support-api --ci-configured  # mark the CI gate met
```

If you enabled Copilot in 0a, propagate its agents into the service now:

```bash
ie update --code-repos        # compiles ie-generate for every enabled runtime into each linked repo
```

## 1 · Intake (~5 min)

For a one-UC feature you can skip straight to shaping. For anything bigger, turn an epic into a feature brief first:

```
/ie/extract-brief
```

It writes the brief on its own `brief/<slug>` branch. Push that branch and open a pull request into the integration branch: merging it signs the brief off, and shaping starts only after that.

## 2 · Shaping — author the spec (~1–2 h, the heart of it)

```
/ie/shape-feature submit-support-ticket
```

This is interactive **by design**. The agent walks the gates and **pauses at each one** for your confirmation — feature scope, how many use cases, UC naming, dependencies. It opens the feature with `ie feature create submit-support-ticket`, which turns that slug into the feature's **id** and its folder — `features/active/FT-001-submit-support-ticket/`, the first feature in a fresh hub. Every later `ie feature` command names the feature by that id. When it authors the use case it:

1. Runs `ie spec create submit-ticket --domain tickets --service support-api` — this scaffolds the spec folder and **pre-fills** `meta.yml` (status/version/priority) and a `contract.openapi.yml` carrying the contract convention. The agent then **edits** those files; it never writes them from scratch.
2. Co-authors `scenarios.feature` (happy path + errors + edge cases), the contract, and an optional flow diagram with you.
3. Runs `ie spec validate --path specs/tickets/UC-001-submit-ticket` and fixes any errors **before** declaring the spec done.

You end with a complete spec folder and a readiness score. Aim for ≥80.

> **What a gate looks like:** the agent stops, summarizes what it produced, and asks — e.g. *"This is one UC, `submit-ticket`. Is that the right boundary, or should viewing tickets be a second UC?"* — and waits for your answer. Steer here; it's far cheaper than fixing generated code later.

## 3 · Review & approve (~10 min)

Push the spec on a branch and open a PR. CI runs automatically: `ie spec register`, lint, readiness, Iron Dome integrity, and a Review Guide comment listing the files you need to read (spec metadata, the catalog and compiled tooling are folded away). Then:

```
/ie/review-spec
```

**What to read (and what to skip).** A feature folder mixes the deliverable with tool-generated indexes. Start from the feature's `specification/integrity-report.md` — its *How to review this feature* header lists the same split for that specific feature:

| Tier | Files |
|------|-------|
| **Review** — the deliverable | the use-case specs (`specs/<domain>/UC-*`), `specification/integrity-report.md`, and the **Key Decisions** table in `shaping/shape.md` |
| **Skim for context** | `shaping/plan.md` (implementation plan) |
| **Skip** — tool-generated indexes, not spec-review material | `manifest.yml`, `shaping/references.md` |

Review, address findings, and **merge the change to `main`**. Merging *is* the approval act — the feature auto-promotes `specified → approved` and the manifest assembles.

## 4 · Handover — hand the spec to the code repo (~5 min)

An `ie feature` verb that names an **existing** feature takes the **`FT` id** — `FT-001`, or the bare number `1` — never the slug. (`ie feature create` is the one that turns the slug into the id, and `/ie/*` commands take the slug too, which is why step 2 reads the way it does. One verb takes no id at all: `ie generation finalize`, which reads it from the manifest sitting in front of it — and it is under `generation`, not `feature`, because that is what it acts on.) `ie feature list` prints the ids.

```bash
ie feature handover FT-001   # cuts a feature branch from a fresh integration branch, writes .flowforge/manifest.yml
```

The manifest tells the code repo exactly what to build, by reference.

## 5 · Generate — from the **code repo** (~30 min)

```bash
cd ../support-api
git checkout <feature-branch>     # the branch handover created
```
```
/ie/generate
```

Generation runs in **five checkpoints** (CP1→CP5: tests → implementation → integration → UI/binding → review). The agent **pauses after each checkpoint** to show you what it produced — files, test results, the reasoning trace — and waits for your go-ahead. Review the high-risk items in the interpretation log as they appear.

## 6 · Verify & push (~15–60 min)

Read the AI Step Summary and sign off. When you `git push`, the **pre-push gate** runs your build + tests locally first (`ie hook pre-push` — cross-platform, runs on Windows/macOS/Linux) and blocks the push if they fail.

## 7–10 · Deliver

**In the code repo, before you open its PR** — tidy up the generation state so the integration branch never receives it:

```bash
ie generation finalize            # no id: the manifest names the feature
```

Then open the PR. Two workflows run on it: the repo's own build/test CI, and IE's `quality-gates.yml` (spec drift, contract validation, standards alerts). Both must be required checks on the integration branch — IE's asserts spec alignment only and cannot speak for the build. If the PR comment says the generation artifacts are still at the repo root, finalize has not run.

Repeat for every touched repo, moving between the two as you go: `ie feature handover` (back in the hub) → `/ie/generate` and `ie generation finalize` (in that repo) → its PR. **Once the last one has merged, record the delivery in the hub:**

```bash
ie feature deliver FT-001
```

`deliver` checks the evidence before it writes: every touched repo must carry `.flowforge/archive/{feature-id}/` on the integration branch, which is what proves that service both finalized *and* merged. It then writes the whole delivery record — `status: delivered`, each owned UC sealed to `implemented` (unless another in-flight feature also changes it), the promotion pattern credited — **where you are standing, uncommitted**, and prints the paths. Commit them yourself; don't leave them sitting, because `ie feature handover` refuses a hub that is dirty in spec paths, so an unnoticed delivery record blocks your next feature.

Where the hub's integration branch is protected, add `--on-branch`: deliver then cuts `delivery/{feature-id}` from a fresh integration branch, commits the record there and tells you which branch it left you on, so you can open it as one small PR. Add `--push` as well and it publishes that branch for you; otherwise it prints the `git push -u origin delivery/{feature-id}` to run. Every check runs either way; `--on-branch` verifies the feature exists at the base ref *before* it changes your branch, and refuses if it does not. If a delivery PR is rejected and you need to redo it, **delete the branch first** (`git branch -D delivery/{feature-id}`) — deliver refuses to reuse one that no longer contains the latest integration branch, because writing the record from a stale branch would revert whatever landed while the PR sat in review.

Three things make `deliver` refuse, each naming the repos it applies to and what to do about them:

- **A repo has not merged its archive yet.** Finalize it and merge that PR.
- **A repo's evidence could not be read at all** — a registered checkout that no longer exists (a moved or renamed clone leaves a registration that still looks healthy), or a fetch that failed. This *blocks*: delivering without it would record a merge nobody checked. Fix the registration or connectivity, or pass `--allow-unverified` for the deliberate offline case — it names each waived repo in its output and its `--json`, so capture that run if you need the audit trail; the delivery record itself does not carry the waiver.
- **A repo is back in flight** — it merged its archive earlier, but a live `.flowforge/manifest.yml` for this feature is there again (which is what a scoped re-handover restores). That work is unfinished and unmerged, so the earlier merge does not speak for it. Finish it, or drop it. `--allow-unverified` does not waive this one: it covers *absent* evidence, and this is evidence *against*.

Nothing signals that the set of repos is complete: `ie feature status FT-001 --repos` shows where each one is (`merged` once a repo's archive is on the integration branch, even after the head branch is deleted), and `ie doctor` says *ready to deliver* once they have all merged — and says so explicitly when it could not reach a repo, rather than reporting a pass it did not verify.

That is the last thing you have to do. **Archiving happens on its own**: the daily cron moves a `delivered` feature — and its bug reports — into `features/archive/` once it has been quiet for the window in `config.yml` (`automation.auto_archive_after_days`, 14 by default). To do it now instead, run `ie feature archive FT-001` in the hub; it refuses anything not yet `delivered`, and like `deliver` it moves files without committing, so commit what it prints. An archived feature stays **readable** — `ie feature status`, `ie integrity check` and blast-radius analysis all resolve it — and nothing writes there again.

---

## When something's wrong

Fix the **spec**, not the generated code. Hand-patched generated code drifts from its spec and gets overwritten on the next run.

- **Feature still in flight, defect is scoped** — run `/ie/bug-report` from the code repo. It diagnoses, classifies against the four feedback categories, applies the narrowest spec fix, checks blast radius, and regenerates from only the invalidated checkpoint onward — no full re-shape, no re-touching services the fix doesn't own.
- **Fix is bigger than a targeted correction** — re-shape (`/ie/shape-feature submit-support-ticket`), re-handover, re-generate.

### After finalize, `/ie/bug-report` refuses — here is the way out

`ie bug-report` keys off the root `.flowforge/manifest.yml`, and finalize archives it, so the in-flight loop is closed by construction from that moment on. What to do next depends on whether the finalize commit has merged:

| State | `ie bug-report` | Route |
|---|---|---|
| finalized, code PR still open | refuses | `ie generation finalize --revert` → manifest restored → bug-report works normally |
| finalized + merged, not delivered, **every** touched repo merged | refuses | `ie feature deliver <id>` → then the post-delivery fix rail |
| finalized + merged, other repos still generating | refuses **in that repo only** | `ie feature handover <id> --spec <UC>` (scoped) restores that repo's manifest → bug-report / `/ie/generate` resume |

The third row is legal by design, not an accidental reopen: handover refuses a `delivered`/`archived` feature, and a feature whose sibling services are still generating is neither. One caveat for that route — if the merged feature branch still exists on origin, handover switches to it rather than cutting a fresh one; delete it first if you want a fresh base.

Row 2's second step is the post-delivery fix rail: **`/ie/fix-defect`**, run from the hub. It classifies the defect, files a `FIX` against the delivery that introduced it (`ie fix create`), and debits the pattern that delivery was credited for. From there the rail is the commands you already know — `ie fix handover` → `/ie/generate` → `ie generation finalize` → `ie fix deliver` — because a fix carries the same manifest shape as a feature. A delivered feature is never reopened: `ie bug-report` refuses one outright, however the manifest in a code repo looks.

## Where to go next

- **Per-command detail** → [`WORKFLOW.md`](WORKFLOW.md) and [`../.claude/commands/ie/`](../.claude/commands/ie)
- **Terms** → [`GLOSSARY.md`](GLOSSARY.md)
- **UI feature?** seed the design system first → [`DESIGN-SYSTEM.md`](DESIGN-SYSTEM.md) — hand-author it, or bind to a client's existing component/token library via its MCP server (`/ie/design-system-vendor-connect`; build the server first with `ie vendor-mcp` if the client hasn't)
- **Found a bug in this feature before finalize?** → `/ie/bug-report` (from the code repo)
