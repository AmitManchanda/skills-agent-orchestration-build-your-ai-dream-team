# Project Pulse agent team

Mona's Project Pulse dashboard will be built by four custom agents defined in `.github/agents/`. The models below are the assignments in those definitions.

| Agent | Assigned model | Definition | Responsibility |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | `.github/agents/orchestrator.agent.md` | Coordinates specialists without implementing code itself. Turns the Planner's output into phases, assigns explicit file scopes, manages dependencies and blockers, and verifies the integrated result. |
| Planner | Claude Opus 5.5 (copilot) | `.github/agents/planner.agent.md` | Researches the repository and relevant documentation without writing code. Produces ordered implementation steps, file assignments, dependencies, parallel and sequential work, edge cases, validation expectations, and open questions. |
| Coder | GPT-5.5 (copilot) | `.github/agents/coder.agent.md` | Implements dashboard code and data within its assigned scope, using clear structure, explicit errors, and deterministic, testable behavior. Creates assigned runnable-app support and validates changes before reporting completion. |
| Designer | Gemini 3.7 Flash (copilot) | `.github/agents/designer.agent.md` | Handles UI/UX, accessibility, information hierarchy, interaction flow, responsive behavior, and visual design within its assigned scope. Makes Project Pulse a polished dashboard with project cards, status badges, clear priorities, and readable spacing. |

## How the team will work together

1. The Orchestrator asks the Planner to turn `.github/project-pulse-brief.md` into an implementation plan with phases, dependencies, file ownership, and validation expectations.
1. The Orchestrator assigns the Designer the dashboard experience and styling, and the Coder the static app implementation. The deliverables are `app/index.html`, `app/styles.css`, and `app/project-data.json`, plus `.vscode/launch.json` for running the dashboard. Each file has an explicit owner; overlapping work or work dependent on earlier output runs sequentially. Independent tasks with disjoint file scopes can run in parallel.
1. The Designer establishes accessible, responsive project cards, status badges, priority treatment, and clear typography, including deterministic CSS hooks such as `.dashboard` and `.project-card`. The Coder implements the dashboard and a top-level `projects` data array with `name`, `owner`, `status`, `recentActivity`, and `priority`, so contributors can quickly understand active projects, ownership, status, recent activity, and priority or risk.
1. When assigned launch support, the Coder creates strict JSON for the **Run Project Pulse Dashboard** configuration, sets `cwd` to `${workspaceFolder}/app`, and opens `index.html` rather than a server directory listing. The Coder validates its changes, the Designer reports design decisions and validation recommendations, and the Orchestrator verifies integration and reports the outcome and any blockers.

All agents stay within their assigned scope and leave staging, committing, and pushing to the learner through explicit Copilot CLI prompts.
