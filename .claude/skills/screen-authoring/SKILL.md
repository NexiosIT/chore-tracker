---
name: screen-authoring
description: >
  Screen spec authoring: a YAML file that composes use cases into one page with sections,
  orchestration and responsive rules. Use when the user wants to author or change a screen spec.
---

# Screen Authoring Skill

## Purpose

This skill guides an engineer through authoring a screen specification. This is a structured YAML file that defines how use cases are composed into a page layout. Screens are the **composition layer**. Use cases own behavior. Screens define *where* UCs appear, *how* they load, *how* they interact, and *how* they adapt across viewports.

The key principle: **screens reference UCs; UCs do not reference screens.** This keeps specs composable. The same UC can appear on multiple screens in different aspects (form, list, detail, summary, card, hero).

One file is produced: a screen spec YAML with four sections:
1. **Metadata**: identity, route (no layout shell: the scaffold + design system own it)
2. **Sections**: page regions, each referencing a UC with an aspect, load trigger, and component tree
3. **Orchestration**: cross-section interactions (on_success/on_error reactions)
4. **Responsive**: mobile/tablet adaptations (reorder, stack, hide, collapse)

## Workflow

### Phase 1: Understand the product context

The user provides a directory path or product context. Read available files to understand:

- **Existing use case specs**: these are what the screen will reference
- **Data model**: entities and relationships
- **Tech stack**: frontend framework, design system
- **Existing screens**: for consistency within the same app
- **Design tokens**: a component tree may reference **only** tokens that exist in the app's design system. Get the authoritative list with `ie design-system tokens --app <app> --json` (the orchestrator supplies `<app>`); those names, and only those, are valid `tokens:` values. If the hub reports `adopted: false`, fall back to the frontend standards' token guidance. Still use token names, not hardcoded values.

Don't assume a fixed structure. Look for `specs/`, `screens/`, `standards/`, `product/` directories. If sparse, ask what's available.

After reading, confirm: "I've read the product context. I can see [N] use case specs available. Ready to compose a screen."

### Phase 2: Page purpose and route

1. Ask the engineer to describe what this page shows and why a user visits it.
2. Confirm the **route pattern** and any route parameters (e.g., `/products/:id`, `/dashboard`).
3. **Layout is not authored here.** The app shell (header, nav, sidebar, the wrapper every page shares) is owned by the native scaffold and themed by the design system. A screen is page *content* that renders inside that shell. Do **not** invent a layout file or a `_layouts/` reference. Leave `layout_ref` unset (the default). Set it only to bind a named design-system layout component that already exists (P2). Never point it at a file you would have to create.
4. Fill in the metadata:

```yaml
id: "SCR-{NNN}"
name: "{Human-readable name}"
slug: "{kebab-case-slug}"
route: "{/path/:params}"
status: "draft"
description: "{One-sentence summary}"
# layout_ref: omit. The scaffold + design system own the app shell.
```

**Validation:**
- Route does not conflict with existing screen routes

---

### Phase 3: Section composition

For each section of the page, work through these in order:

**3a. Identify the UC.** Ask: "What use case powers this section?" Validate the UC exists.

**3b. Choose the aspect.** Ask: "How should this UC appear?" Options:
- `form`: input form (create/update)
- `list`: collection of items
- `detail`: single-item read view
- `summary`: condensed overview
- `card`: compact card representation
- `hero`: prominent display (banner, splash)

**3c. Set the load trigger.** Ask: "Should this load immediately, when scrolled near, or on user action?"
- `page-load`: eager, loads immediately (use for above-the-fold content)
- `scroll-near`: lazy, loads when scrolled into/near viewport (use for below-the-fold)
- `user-action`: on-demand, loads on click/expand (use for tabs, accordions)
- If `user-action`: determine the trigger element and accessible label

**3d. Define the component tree.** Build the UI structure:
- Map response schema fields from the UC to UI components
- Set every `tokens:` slot to a token **name from the `ie design-system tokens` vocabulary**. Raw values (`#fff`, `16px`, font names) and invented names are not valid.
- **Missing a token?** Record a design-system change request for the author to add via the design system. The request holds the token name, intended meaning and suggested layer (`_base` vs this app). Then reference the closest existing token (or leave a clearly-flagged TODO) so the spec stays authorable.

```yaml
sections:
  - name: "{section-name}"
    use_case_ref: "{path to UC spec}"
    aspect: "{form|list|detail|summary|card|hero}"
    load_trigger: "{page-load|scroll-near|user-action}"
    component_tree:
      component:
        name: "{ComponentName}"
        type: "{container|heading|text|list|card|table|form|button|image}"
        tokens:
          padding: "{token-name}"
          background: "{token-name}"
        children:
          - name: "{ChildComponent}"
            type: "{type}"
            tokens: { ... }
            props:
              field: "{response field name}"
```

Repeat for each section. Order sections as they appear top-to-bottom on the page.

**Validation after all sections:**
- Every `use_case_ref` points to an existing spec
- No duplicate UC+aspect combinations within the same screen
- Every `tokens:` value is a token name present in `ie design-system tokens --app <app>`. Raw values and invented names fail. The orchestrator confirms with `ie design-system check` before finalizing.
- Above-the-fold sections use `page-load`; below-the-fold use `scroll-near` or `user-action`

---

### Phase 4: Cross-section orchestration

1. Ask: "When an action succeeds in one section, should anything else on the page change?"
2. For each interaction, define:
   - **source**: which section triggers the event
   - **on_success** reactions: `refresh` (reload section data), `show`/`hide` (toggle visibility), `navigate` (go to another route), `notify` (toast/banner)
   - **on_error** reactions: `notify` (error toast), `highlight` (mark section), retry prompt

```yaml
orchestration:
  - source: "{section-name}"
    on_success:
      - target: "{section-name or null}"
        action: "{refresh|show|hide|navigate|notify}"
        params:                          # for notify action
          type: "toast"
          variant: "success"
          message: "{message text}"
          duration_ms: 3000
    on_error:
      - target: null
        action: "notify"
        params:
          type: "toast"
          variant: "error"
          message: "{error message}"
```

**Validation:**
- All `source` values reference a defined section name
- All `target` values reference a defined section name or `null` (for page-level actions)
- Notification parameters include type, variant, message, and duration

---

### Phase 5: Responsive behavior

1. Ask: "On mobile, should any sections be reordered, stacked, hidden, or collapsed?"
2. Ask: "On tablet, any layout changes needed?"
3. For each breakpoint adaptation:
   - Identify the section and action: `reorder`, `stack`, `hide`, `collapse`
   - If `reorder`: specify the new position
   - If `collapse`: specify the expand trigger label

```yaml
responsive:
  - breakpoint: "mobile"              # <768px
    changes:
      - section: "{name}"
        action: "{reorder|stack|hide|collapse}"
        position: 1                   # for reorder
  - breakpoint: "tablet"              # 768-1024px
    changes:
      - section: "{name}"
        action: "{action}"
```

**Validation:**
- All section names in responsive rules match defined sections
- Critical content sections are not hidden on mobile (warn if they are)
- Reorder positions are valid (within the number of sections)

---

### Phase 6: Final review

Before writing the file, present a summary:
- Page purpose and route
- Section count with UC references and load triggers
- Orchestration rules count
- Responsive adaptations count

Ask the engineer to confirm before writing.

## Output

A complete screen spec YAML file containing:
- **Metadata**: id, name, slug, route, status, description (no `layout_ref`: the app shell is owned by the scaffold + design system)
- **Sections**: each with UC reference, aspect, load trigger, and component tree
- **Orchestration**: cross-section on_success/on_error interactions
- **Responsive**: breakpoint adaptations for mobile and tablet

## File naming and location

Default: `SCR-{NNN}-{kebab-name}.yml` in the directory the user specifies. Ask if they have a preferred location.

## Composition principles

- **Screens reference UCs; UCs don't reference screens.** This keeps specs composable.
- **The same UC can appear on multiple screens** in different aspects (a product UC might be `detail` on the product page and `card` on the search results page).
- **Load triggers are intentional.** Above-the-fold = `page-load`, below-the-fold = `scroll-near`, optional content = `user-action`.
- **Design Tokens only.** All visual values in component trees must reference tokens, never hardcoded values. This enforces consistency and theming support.
- **Orchestration captures page-level coordination** that no single UC owns, such as a refresh of section B when section A succeeds.

## Target

- Reference exactly one UC spec from each section, through `use_case_ref`.
- Record at least the mobile adaptations of every screen in `responsive`.
