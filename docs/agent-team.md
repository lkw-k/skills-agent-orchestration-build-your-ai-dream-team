# Agent team for Mona's Project Pulse dashboard

I am using GitHub Copilot CLI in a Codespace to orchestrate the work for Mona's Project Pulse dashboard. The team is organized around a clear division of responsibilities so the project can move from planning to implementation to polished UI design without overlap or confusion.

## Team overview

### Orchestrator
- Agent name: Orchestrator
- Model: Claude Opus 4.7 (copilot)
- Responsibility: Coordinates the entire workflow by breaking the work into phases, assigning files and tasks to specialist agents, sequencing dependencies, and validating that the final result fits together as a cohesive product.
- Defined in: `.github/agents/orchestrator.agent.md`

### Planner
- Agent name: Planner
- Model: Claude Opus 4.7 (copilot)
- Responsibility: Researches the repo, reviews relevant docs and constraints, identifies risks and edge cases, and produces a practical implementation plan with ordered steps, dependencies, file ownership, and validation expectations.
- Defined in: `.github/agents/planner.agent.md`

### Coder
- Agent name: Coder
- Model: GPT-5.5 (copilot)
- Responsibility: Implements the code, logic, and app behavior within the file scope assigned by the Orchestrator. This includes building the functional parts of the dashboard and validating the behavior before reporting completion.
- Defined in: `.github/agents/coder.agent.md`

### Designer
- Agent name: Designer
- Model: Gemini 3.1 Pro (copilot)
- Responsibility: Focuses on the UX, information hierarchy, accessibility, responsive layout, and visual polish of the dashboard so the result feels like a professional Project Pulse experience rather than a bare prototype.
- Defined in: `.github/agents/designer.agent.md`

## How the team works together

The Orchestrator acts as the project lead. It asks the Planner to research the repository and define a realistic implementation strategy, then converts that strategy into phases with clear ownership boundaries. The Planner identifies the necessary files, dependencies, and validation steps so work can be scheduled without conflicts.

Once the plan is ready, the Coder executes the functional implementation, while the Designer handles the visual and interaction design in parallel when their scopes do not overlap. The Orchestrator keeps the workflow coordinated by sequencing tasks that depend on each other and by checking that the code and design integrate cleanly into a single dashboard experience.

This hands-off but structured workflow lets each specialist focus on what they do best: planning, implementation, and design, while the Orchestrator keeps the overall project aligned with the goal of building Mona's Project Pulse dashboard in a disciplined, manageable way.
