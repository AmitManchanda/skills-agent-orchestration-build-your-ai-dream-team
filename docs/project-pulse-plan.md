# Project Pulse implementation plan

## Summary

Build Mona’s Project Pulse dashboard as a lightweight static frontend that helps contributors understand project ownership, current status, recent activity, priority, and opportunities to contribute.

Deliver a polished, accessible, responsive dashboard with visible project cards and status badges, backed by JSON data and opened through **Run Project Pulse Dashboard** in VS Code.

**Scope:** Three application files and one launch configuration. No framework, package installation, build pipeline, backend, additional JavaScript file, or new test infrastructure is planned.

This document is the implementation plan; no dashboard files are implemented by this planning step. Agents must not stage, commit, or push changes.

## Repository findings

The plan follows:

- `.github\project-pulse-brief.md`
- `.github\agents\planner.agent.md`
- `.github\agents\orchestrator.agent.md`
- `.github\agents\designer.agent.md`
- `.github\agents\coder.agent.md`
- `.github\steps\2-step.md`
- `.github\steps\3-step.md`
- `docs\agent-team.md`
- Relevant exercise workflows and existing configuration.

Observed state:

- `app\` does not currently exist.
- `.vscode\tasks.json` exists; `.vscode\launch.json` does not.
- No existing dashboard files or application dependency manifests were found.
- The repository targets a Codespace/devcontainer exercise, while the current workspace is native Windows.
- `.devcontainer\devcontainer.json` lists Copilot and GitHub Actions extensions, but does not explicitly configure a Python debugger.
- `.git\HEAD` points to `main`. Working-tree cleanliness and staged changes were not verified; do not assume a clean checkout.
- `.github\workflows\3-step.yml` checks file existence, required phrases, and JSON parsing. It does **not** prove rendering or launch behavior.
- `scripts\validate-exercise.sh` validates exercise/template scaffolding and includes checks against tracked learner answer files. It is not a dashboard acceptance suite and should not be treated as one.

Before implementation, the Orchestrator should obtain a read-only `git status --short` and inspect any newly present target files. Preserve unrelated learner work.

## File ownership

All filesystem paths below are relative to the Windows workspace root.

| File | Exclusive implementation owner | Responsibility |
|---|---|---|
| `app\index.html` | Coder | Semantic document, dashboard shell, inline rendering logic, data loading, summaries, and loading/error/empty states |
| `app\project-data.json` | Coder | Strict JSON, representative project records, contributor summaries, agreed schema |
| `.vscode\launch.json` | Coder | Strict JSON, server command, working directory, readiness matching, explicit frontend URL |
| `app\styles.css` | Designer | Layout, visual hierarchy, cards, badges, priorities, typography, responsive behavior, accessibility styling |

The exercise uses the identifiers `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json`; these refer to the Windows paths above.

**Designer:** Own visual and accessibility decisions and implement only the assigned stylesheet. Review HTML accessibility without editing Coder-owned files.

**Coder:** Own markup, behavior, data, and launch configuration. Implement Designer’s agreed semantic requirements without editing the stylesheet.

**Orchestrator:** Approve shared contracts, assign bounded work, manage dependencies, coordinate integration, and report blockers. Do not implement files.

Preserve `.vscode\tasks.json`, `.devcontainer\`, agent definitions, workflows, scripts, README, and existing documentation. No evidence justifies alternative ownership or additional files.

## Shared contracts: approve before parallel implementation

### Data contract

`app\project-data.json` must be a strict JSON object containing a top-level `projects` array.

Each project must contain these nonempty string fields:

| Field | Meaning |
|---|---|
| `name` | Project’s display name |
| `owner` | Responsible person or team |
| `status` | Current project status |
| `recentActivity` | Concise recent progress or update |
| `priority` | Explicit priority label |
| `summary` | Proposed additional field: concise purpose and a useful contributor-oriented next step |

The five exercise-required fields are mandatory. Add `summary` because the brief explicitly requests contributor-friendly summaries. If a later record omits or leaves `summary` blank, render a neutral fallback using available project facts rather than inventing contribution guidance.

Proposed initial vocabulary:

- Status: `In progress`, `At risk`, `Blocked`, `Complete`.
- Priority: `High`, `Medium`, `Low`.
- Active projects: `In progress`, `At risk`, and `Blocked`.
- Needs attention: `At risk` and `Blocked`.

Treat priority and risk as different concepts: show priority separately and communicate risk through the explicit status label. Do not infer risk from high priority alone.

Use several clearly identified demonstration projects unless approved project records are supplied. Include different statuses and priorities. Preserve JSON order; no sorting or filtering is required.

Unknown nonempty status or priority values remain visible with neutral styling. Missing required fields, wrong field types, and malformed records trigger an explicit data-validation error rather than silently changing the dataset.

### DOM and CSS contract

Approve this structure and hook vocabulary before implementation:

| Hook | Intended element or purpose |
|---|---|
| `.dashboard` | Main dashboard landmark |
| `.dashboard-header` | Title and contributor-oriented introduction |
| `.dashboard-summary` | Data-derived total, active, and attention counts |
| `.project-grid` | Responsive project collection |
| `.project-card` | One semantic project article |
| `.project-card__title` | Project heading |
| `.project-card__summary` | Contributor-friendly summary |
| `.project-card__meta` | Labeled owner and metadata |
| `.project-card__activity` | Labeled recent activity |
| `.status-badge` | Visible textual status badge |
| `.priority-badge` | Visible textual priority badge |
| `.dashboard-message` | Loading, empty, or error message |
| `.dashboard-message--error` | Error-state treatment |

Use explicit modifier classes for the agreed status and priority vocabulary, plus neutral fallbacks. The Coder must map values to an allowlist of modifiers rather than inserting arbitrary data into class names.

Markup requirements:

- Exact document title and visible main heading: **Project Pulse**.
- Reference `styles.css` and load `project-data.json` using relative paths.
- One main landmark and a logical heading hierarchy.
- Each project rendered from JSON as a `.project-card`.
- Clearly labeled owner, status, recent activity, and priority.
- Cards are not interactive unless a real action is introduced.
- A polite live-status region for load results; a distinct accessible error announcement.
- JavaScript-disabled guidance.
- Data-derived text inserted as text, not interpreted as HTML.

### Behavior contract

- Keep rendering logic inline in `app\index.html`; do not add another application file.
- Load JSON over the served origin.
- Check unsuccessful HTTP responses separately from JSON parsing and schema validation. Fetch can return a response for an unsuccessful HTTP status, so response status must be handled explicitly. ([github.com](https://github.com/whatwg/fetch/issues/103))
- Distinguish loading, valid data, valid empty data, and failure.
- Use a bounded loading timeout so stalled requests do not leave the page permanently loading.
- On failure, retain the dashboard heading and useful recovery guidance; do not substitute misleading sample data.
- For an empty array, show “No projects to display” and zero counts.
- Render counts only from successfully validated records.
- Keep recent activity as supplied text; do not invent timestamps or a “live” refresh claim.
- Preserve full information when content wraps.

### Launch contract

Coder must create `.vscode\launch.json` with:

- Strict JSON, without comments or trailing commas.
- Standard launch-file version and a configurations array.
- Configuration name **Run Project Pulse Dashboard**.
- Launch working directory exactly `${workspaceFolder}/app`.
- Exercise command exactly `python3 -m http.server 5500`.
- `serverReadyAction` with a pattern capturing the port from Python’s actual startup output.
- `uriFormat` exactly `http://localhost:%s/index.html`.
- An external browser-opening action.

VS Code’s readiness action matches server output and substitutes the captured value into the URI; the pattern must match the real server banner rather than a guessed Node-style message. ([code.visualstudio.com](https://code.visualstudio.com/docs/debugtest/debugging-configuration))

**Preferred minimal approach:** Evaluate the built-in JavaScript debug terminal launch type, `node-terminal`, which supports a terminal command and working directory. Use it to start the preview command, not to debug Python application code. Validate the selected type against the installed VS Code schema and actual browser-opening behavior. ([github.com](https://github.com/microsoft/vscode-js-debug/blob/main/OPTIONS.md))

Do not assume that a generic `node` launch configuration can debug Python, or that a Python debugger extension is installed. If the minimal approach fails, surface the blocker before adding tooling or widening ownership.

Python serves the current directory by default and can produce a directory listing when no index page exists. Both the `app` working directory and explicit `/index.html` target are therefore essential. ([docs.python.org](https://docs.python.org/3/library/http.server.html))

If startup output is delayed, evaluate `PYTHONUNBUFFERED` in the configuration’s environment while preserving the required command; a nonempty value enables unbuffered Python output. ([docs.python.org](https://docs.python.org/3.12/using/cmdline.html?highlight=pythonunbuffered))

## Ordered implementation phases

### Phase 1 — Resolve prerequisites and freeze contracts

**Owners:** Orchestrator, with Designer and Coder recommendations.  
**File assignments:** Read-only inspection; no application edits.

1. Check repository state and reread any target files that now exist.
2. Confirm the preview environment: native Windows or the exercise Codespace.
3. Verify Python command availability and the installed VS Code launch schema.
4. Designer specifies visual direction, semantic requirements, badge modifiers, and responsive treatment.
5. Coder confirms schema, DOM structure, error policy, count definitions, and launch approach.
6. Orchestrator approves the shared contracts above and communicates them to both owners.

**Exit gate:** No unresolved DOM, CSS, data, or launch-interface decision that either implementation branch must guess.

### Phase 2A — Implement dashboard behavior, data, and launch support

**Owner:** Coder.  
**Files:** `app\index.html`, `app\project-data.json`, `.vscode\launch.json`.  
**Dependencies:** Phase 1 approval.

1. Create representative, clearly labeled demonstration records matching the schema.
2. Build the agreed semantic dashboard shell and relative file references.
3. Implement safe JSON-driven cards and accurate summary counts.
4. Implement loading timeout, empty state, explicit errors, neutral vocabulary fallbacks, and JavaScript-disabled guidance.
5. Create the approved launch configuration.
6. Validate JSON, field completeness, rendering behavior, and launch prerequisites.

Coder should implement data before final renderer verification. The renderer must consume the data contract, not require Designer’s unfinished stylesheet to function.

### Phase 2B — Implement polished responsive styling

**Owner:** Designer.  
**File:** `app\styles.css`.  
**Dependencies:** Phase 1 approval.

1. Establish system-font typography, colors, spacing, and a readable content width.
2. Style the dashboard header, count summary, and responsive project grid.
3. Add visible card surfaces with `border-radius`, `box-shadow`, and clear separation.
4. Style textual status and priority badges with sufficient contrast.
5. Style loading, empty, error, and neutral fallback states.
6. Support narrow screens, long content, zoom, and any genuine interactive element’s focus state.
7. Avoid required external fonts, images, animations, or asset downloads.

Designer may work against the frozen DOM contract. Changes discovered through integrated preview belong to Phase 3.

### Phase 3 — Integrate and resolve mismatches

**Owners:** Orchestrator coordinates; Coder and Designer retain their exclusive files.  
**Files:** The four assigned deliverables.  
**Dependencies:** Both Phase 2 branches complete.

1. Serve the integrated frontend.
2. Confirm cards and counts reflect the actual JSON.
3. Designer reviews hierarchy, readability, badge clarity, semantic structure, and responsive behavior.
4. Coder verifies loading and failure behavior and the actual launch experience.
5. Route each defect to its file owner.
6. If a contract must change, approve that change before dependent edits resume.

**Exit gate:** No missing hook, mismatched modifier, hidden required field, data inconsistency, or unverified launch blocker.

### Phase 4 — Acceptance validation and completion report

**Owners:** Coder performs functional and launch checks; Designer performs visual/accessibility review; Orchestrator verifies completion.  
**File assignments:** Read-only validation, with fixes restricted to existing ownership.  
**Dependencies:** Phase 3 integration.

Run the acceptance checks below. Report:

- Files created or changed.
- What was actually validated and in which environment.
- Any untested environment, missing prerequisite, or remaining risk.
- Confirmation that unrelated files were preserved.

Do not create new reports, screenshots, test fixtures, or tooling files unless separately requested. Git operations remain learner-controlled.

## Dependencies and parallel work decisions

| Work | Ordering decision | Reason |
|---|---|---|
| Initial read-only design and implementation research | May run in parallel | No edits and no dependency on the other task’s output |
| Contract approval → implementation | Must be sequential | Both owners require approved structure, vocabulary, and state behavior |
| Phase 2A Coder work ↔ Phase 2B Designer work | May run in parallel after approval | Disjoint file ownership **and** no unresolved data or DOM dependency |
| Data authoring → renderer verification | Sequential within Coder work | Verification requires actual schema-compliant records |
| Launch configuration preparation ↔ stylesheet implementation | May run in parallel after environment decision | Independent files and no runtime dependency on CSS |
| Integrated frontend review | Must follow both Phase 2 branches | Requires markup, data, and styling together |
| Contract-changing fixes | Must be coordinated sequentially | A change can invalidate another owner’s implementation |
| Independent HTML and CSS fixes | May run in parallel only after diagnosis | Allowed only if ownership is disjoint and neither fix depends on the other’s output |
| Final acceptance | Must follow integration and fixes | Must evaluate the final combined result |

Disjoint files alone do not establish independence. Pause parallel work whenever either owner needs an unresolved decision or output from the other.

## Edge cases and risks

| Case | Required handling |
|---|---|
| Missing JSON, unsuccessful HTTP response, or network failure | Clear error and recovery guidance; no endless loading |
| Invalid JSON or incorrect root shape | Distinct data error; no misleading counts |
| Missing or invalid required project field | Explicit validation failure identifying the problem |
| Empty `projects` array | Intentional empty state and zero counts |
| Missing optional summary | Neutral factual fallback |
| Unknown status or priority | Preserve visible value with neutral badge styling |
| Long names, activity, or summaries | Wrap without clipping or horizontal page scrolling |
| Duplicate project names | Render independently; do not use names as unique DOM IDs |
| HTML-like or Unicode content | Display safely as literal text |
| Slow/stalled request | Bounded timeout and recoverable error |
| JavaScript disabled or direct filesystem opening | Explain the required served preview and JavaScript dependency |
| Port 5500 occupied | Report the conflict; do not silently open another server or change ports |
| `python3` unavailable on Windows | Surface prerequisite; obtain approval for a Windows override or use Codespace |
| Unexpected startup banner or delayed output | Verify readiness matching and browser opening |
| Remote/browser-hosted VS Code | Verify actual URL forwarding; do not assume local desktop behavior |
| Server remains running after preview | Stop the process and verify the port is reusable |
| Native Windows path contains spaces | Use the workspace variable; avoid hardcoded machine paths |

Keep preview data non-sensitive. Python’s simple HTTP server is development-only, and the required command binds to all interfaces by default; do not treat it as production hosting or expose private project data. ([docs.python.org](https://docs.python.org/3/library/http.server.html))

## Validation expectations

### Files and repository scope

- Exactly the four assigned implementation files are created or changed, unless an approved pre-existing-file conflict requires adjustment.
- `.vscode\tasks.json` and exercise scaffolding remain unchanged.
- Review read-only git status and diff for unrelated changes.
- Both JSON files parse with a strict parser.

If `python3` is available, run from the workspace root:

- `python3 -m json.tool app\project-data.json`
- `python3 -m json.tool .vscode\launch.json`

These commands validate syntax, not schema or behavior. Separately verify the top-level array and every required field.

### Served frontend

- Start the preview from `app\`, using the approved command.
- Open `http://localhost:5500/index.html`.
- Confirm the page shows **Project Pulse**, not a directory listing.
- Confirm CSS and JSON requests succeed in browser developer tools.
- Confirm visible card count matches the JSON.
- Compare displayed owner, status, activity, priority, and summary against records.
- Verify total, active, and attention counts against the agreed definitions.
- Confirm no unexpected console errors.
- Reload and confirm deterministic results.

Use `curl.exe` on Windows for response inspection if useful; do not assume another test runner is installed.

### VS Code launch behavior

- Open the repository root as the workspace.
- Select **Run Project Pulse Dashboard** in Run and Debug.
- Start it without manually starting a separate server.
- Verify the command executes from `${workspaceFolder}/app`.
- Verify readiness detection opens the dashboard automatically at `/index.html`.
- Confirm relative CSS and JSON loading works through that opened URL.
- Stop the server and confirm another launch can reuse port 5500.
- Record the tested environment. Do not claim Windows and Codespace compatibility when only one was exercised.

Manual browser opening alone does not satisfy launch acceptance.

### Failure and empty states

During implementation validation, Coder may temporarily alter its own data file or use browser request blocking. Restore valid demonstration data afterward; do not leave fixtures.

Exercise:

- Empty array.
- Missing/blocked data request.
- Invalid JSON.
- Missing `projects` or non-array value.
- Missing required field and wrong field type.
- Unknown status and priority.
- Missing optional summary.
- Long content and literal HTML-like text.
- Slow or stalled request.
- JavaScript-disabled view.

Verify clear messages, accurate counts, safe text rendering, and recovery after restoring valid data and reloading.

### Accessibility and responsive design

- Verify landmarks, heading order, meaningful labels, and sensible reading order.
- Confirm asynchronous state messages are exposed accessibly without announcing every card.
- Keep badges understandable without color.
- Check keyboard navigation and visible focus for actual controls; do not make static cards unnecessary tab stops.
- Target at least 4.5:1 contrast for ordinary text and 3:1 for qualifying large text. ([w3.org](https://www.w3.org/WAI/WCAG20/Understanding/contrast-minimum.html))
- Check approximately 320, 768, and 1280 CSS-pixel widths, plus 200% zoom.
- Verify reflow at an effective 320 CSS-pixel width, including a 1280-wide viewport at 400% zoom, without lost content or horizontal page scrolling. ([w3.org](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html))
- Inspect long metadata, wrapped badges, and multiple card counts.
- Use available browser accessibility inspection or assistive technology; report unavailable checks rather than claiming they passed.

Exercise workflow checks are necessary compatibility checks, not a substitute for this served frontend, launch, accessibility, and failure-state validation.

## Material open questions

1. **Preview environment:** Must native Windows launch work immediately, or is the exercise Codespace the acceptance environment? Python availability and the proposed launch adapter remain unverified.
2. **Windows interpreter:** If `python3` is unavailable, may Coder add a verified Windows-only command override within `.vscode\launch.json` while retaining the required base command?
3. **Project content:** Are approved real records available? Otherwise, proceed with explicitly labeled demonstration data and do not present it as live team information.
4. **Vocabulary approval:** Confirm the proposed statuses, priorities, active-project definition, and additional `summary` field during Phase 1.

No further product decision is needed to begin prerequisite and contract review. Do not expand scope with filters, editing, authentication, persistence, or live integrations.
