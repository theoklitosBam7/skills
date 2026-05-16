---
name: plan-visualizer
description: |
  Creates rich, self-contained HTML plan files with visual diagrams, flows, and
  interactive elements. Output goes to ~/.agent/plans/ with a timestamped filename.
  Use whenever producing a plan, architecture proposal, task breakdown, or
  implementation roadmap — especially when the plan involves multiple steps,
  components, sequences, or decision trees that benefit from visual representation.
  Triggers on: "create a plan", "plan this out", "design this", "break this down",
  "write a plan for", "map out", "outline the approach", "document the architecture".
---

# Plan Visualizer

Produce a rich, self-contained HTML plan file and save it to `~/.agent/plans/`.

## Output

**Location**: `~/.agent/plans/<descriptive-slug>-YYYYMMDD-HHMMSS.html`

- Use a short, kebab-case slug that describes the plan (e.g., `auth-refactor`, `api-design`, `onboarding-flow`).
- Append a timestamp: `-20260514-143022`.
- Open `~/.agent/plans/` to confirm it exists; create it if not.

## File Requirements

- **Self-contained**: one `.html` file, no build step, works when opened directly in a browser.
- **No local dependencies**: everything either inline or loaded from a CDN (CDN URLs are fine — the browser handles them).
- Use the template at `assets/plan-template.html` as the starting point (copy it, then fill in content).

## Template Usage

1. Read `assets/plan-template.html` to understand the structure and available components.
2. Copy it to the target output path.
3. Replace every `{{PLACEHOLDER}}` with real content.
4. Keep sections that apply; **delete** sections that don't (including their `<!-- comment -->` block).

## Design System

The template uses a clean, modern design language:

- **Typography**: Inter (via Google Fonts CDN) for body, bold tight-tracking for headings. No emoji in section headings.
- **Color**: Indigo accent (`#4f46e5` light / `#818cf8` dark). Semantic colors for badges and callouts.
- **Cards**: Liquid-glass effect with `backdrop-filter: blur(12px)` and subtle inner highlight.
- **Animations**: CSS `fadeUp` keyframe on all content blocks; pulsing dot in sidebar; smooth hover transitions on cards and tasks.
- **Scroll progress**: Gradient progress bar pinned to the top of the viewport.
- **Dark mode**: Toggles via button in the header; theme icon swaps between moon and sun SVG.
- **Mobile**: Sidebar collapses behind a hamburger button; overlay dismisses on tap; sections stack vertically.
- **Print**: Sidebar, progress bar, and action buttons are hidden; layout collapses to single column.

## Visual Components to Pick From

| Need | Component |
|---|---|
| Architecture / module relationships | Mermaid `flowchart` or `graph` |
| API / system interactions | Mermaid `sequenceDiagram` |
| State machine or lifecycle | Mermaid `stateDiagram-v2` |
| Data model / schema | Mermaid `erDiagram` |
| Timeline / phases | Mermaid `gantt` + Phase Timeline HTML |
| Ordered steps | Numbered task cards (HTML) with checkboxes |
| Decision logic | Mermaid `flowchart` with diamonds |
| File/folder structure | `<pre><code>` tree block |
| Risk or priority matrix | HTML table with color badges |
| Decisions table | Two-column table with badge-highlighted choices |
| Stat summary | 3-card stat grid |
| Collapsible detail | `<details>/<summary>` (styled with chevron) |

Mermaid.js is loaded from CDN in the template. Write diagram code between `<pre class="mermaid">` tags.

## Content Sections

Every plan **must** include at least:

| # | Section | Placeholder(s) | Required? |
|---|---------|----------------|-----------|
| 1 | **Overview** | `{{TITLE}}`, `{{DATE}}`, `{{AUTHOR}}`, `{{STATUS}}`, `{{SUMMARY}}`, `{{GOAL}}` | Yes |
| 2 | **Architecture** | `{{ARCHITECTURE_DESCRIPTION}}` | Yes (or delete) |
| 3 | **Implementation Steps** | `{{STEP_N_TITLE}}`, `{{STEP_N_DESC}}`, `{{STEP_N_EST}}` | Yes |
| 4 | **Risks & Open Questions** | `{{OPEN_QUESTION_N}}`, `{{RISK_N}}`, `{{ASSUMPTION_N}}` | Yes |

Recommended sections (include when relevant):

| # | Section | Placeholder(s) | When to include |
|---|---------|----------------|-----------------|
| 5 | **Context** | `{{CONTEXT}}` | Background on what led to this plan |
| 6 | **Non-Goals** | `{{NON_GOAL_N}}`, `{{NON_GOAL_N_DESC}}` | Explicitly scope out what is NOT in plan |
| 7 | **Dependencies** | `{{DEP_N}}`, `{{DEP_N_DESC}}` | Prerequisites that must exist before work |
| 8 | **Interaction Flow** | Mermaid `sequenceDiagram` | API calls, user journeys, event flows |
| 9 | **Timeline** | `{{PHASE_N_TITLE}}`, `{{PHASE_N_DESC}}`, optional Gantt | Phased delivery, milestone tracking |
| 10 | **Data Model** | Mermaid `erDiagram` | New or changed database schema |
| 11 | **Decisions & Tradeoffs** | `{{DECISION_N}}`, `{{CHOSEN_N}}`, `{{ALTERNATIVES_N}}`, `{{RATIONALE_N}}` | Document ADRs inline |
| 12 | **Testing Strategy** | `{{TEST_N_TITLE}}`, `{{TEST_N_DESC}}`, `{{TEST_N_TYPE}}` | How each phase will be verified |
| 13 | **Rollback & Migration** | `{{ROLLBACK_PLAN}}`, `{{MIGRATION_NOTES}}` | Destructive changes, data migration |
| 14 | **File Structure** | `{{FILE_TREE}}` | New directories, file tree |
| 15 | **API Contract** | `{{REQUEST_EXAMPLE}}`, `{{RESPONSE_EXAMPLE}}` | New or changed endpoints |
| 16 | **Change Log** | `{{CHANGELOG_DATE_N}}`, `{{CHANGELOG_ENTRY_N}}` | Plan revisions over time |

### Stat Cards

The Overview section includes a 3-card stat grid. Pick meaningful summary metrics:

- Files changed, estimated hours, services impacted
- API endpoints, database tables, new dependencies
- Team members, review cycles, risk score

### Badges

| Class | Color | Usage |
|---|---|---|
| `badge-success` | Green | Done, ready, approved |
| `badge-warning` | Amber | Pending, medium priority |
| `badge-danger` | Red | Risk, high priority, blocked |
| `badge-info` | Blue | Estimate, type tag |
| `badge-accent` | Indigo | Status, in progress |
| `badge-neutral` | Gray | Todo, untagged |

### Callouts

| Class | Border | Background | Usage |
|---|---|---|---|
| `callout-info` | Blue | Light blue | Informational notes |
| `callout-warning` | Amber | Light amber | Open questions, caution |
| `callout-danger` | Red | Light red | Risks, blockers |
| `callout-success` | Green | Light green | Assumptions, confirmed items |

### Phase Timeline

Horizontal phase bar with `class="phase active"` for the current phase. The active phase gets an accent-colored bottom border and tinted background.

### Task Checkboxes

Every task item has a `<input type="checkbox" class="task-checkbox">`. Clicking it dims the task and strikes through the title. Useful for tracking progress in the HTML file itself.

## Mermaid Quick Reference

```
flowchart TD
    A[Start] --> B{Decision?}
    B -->|Yes| C[Do X]
    B -->|No| D[Do Y]
    C --> E[End]
    D --> E
```

```
sequenceDiagram
    Client->>API: POST /login
    API->>DB: SELECT user
    DB-->>API: user row
    API-->>Client: 200 JWT
```

```
stateDiagram-v2
    [*] --> Idle
    Idle --> Processing: trigger
    Processing --> Done: success
    Processing --> Error: failure
    Done --> [*]
```

```
erDiagram
    USER { string id PK; string email; }
    ORDER { string id PK; string user_id FK; }
    USER ||--o{ ORDER : places
```

```
gantt
    title Project Timeline
    dateFormat  YYYY-MM-DD
    section Phase 1
    Task A   :active, a1, 2026-05-14, 7d
    Task B   :        a2, after a1,   5d
```

## Writing Good Plans

- **Title**: Action-oriented, specific. "Refactor auth to JWT" not "Auth changes".
- **Summary**: One paragraph. Problem + approach + expected outcome.
- **Goal**: Single sentence starting with "Enable...", "Reduce...", "Allow...".
- **Steps**: Ordered, actionable, each with a clear success criterion.
- **Risks**: Name the risk + mitigation, not just the risk.
- **Decisions**: Record what was chosen, what was rejected, and why. Future readers need the "why".
- **Non-Goals**: Explicitly out-of-scope items prevent scope creep and misaligned expectations.
- **Change Log**: Update when the plan is revised. Date + what changed + why.

## Section Selection Guide

Use this decision tree to pick which sections to include:

```
Is this a refactoring or migration?
  ├── Yes → Include: Rollback & Migration, Testing Strategy
  └── No  → Skip those sections

Does this plan touch the database?
  ├── Yes → Include: Data Model, Migration notes
  └── No  → Skip

Is there a new or changed API?
  ├── Yes → Include: API Contract, Interaction Flow
  └── No  → Skip

Is this a multi-phase effort?
  ├── Yes → Include: Timeline (phases + Gantt)
  └── No  → Skip

Is the plan scope ambiguous?
  ├── Yes → Include: Non-Goals, Context
  └── No  → Optional
```

## Common Pitfalls

- Do NOT omit any mermaid related HTML/CSS/JS blocks that are present in the template; we do NOT want broken diagrams
- Always output the plan under `~/.agent/plans` directory in HTML format; no build step, no dependencies beyond a browser. Append a timestamp to filename (-yyymmdd-hhmmss).
