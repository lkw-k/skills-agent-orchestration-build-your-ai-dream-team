# Project Pulse Dashboard Implementation Plan

## Summary

Build a lightweight static Project Pulse dashboard that helps contributors quickly understand active projects, ownership, status, recent activity, priority or risk, and contributor-friendly summaries. The app will consist of a semantic HTML entry point, a responsive stylesheet, and JSON-backed project data.

The Orchestrator will coordinate the Planner, Designer, and Coder using non-overlapping file ownership. The Coder will implement the functional dashboard and launch configuration. The Designer will define the information hierarchy, accessibility requirements, responsive behavior, and visual treatment without modifying the Coder-owned files unless explicitly reassigned.

The finished VS Code launch configuration must serve files from `app/` and open `app/index.html` directly. It must not open the server root or expose a directory listing as the first screen.

## Scope and file ownership

| File | Owner | Responsibility |
|---|---|---|
| `app/index.html` | Coder | Create the semantic dashboard shell, project card markup or rendering logic, required data loading, and accessible labels/states. |
| `app/styles.css` | Designer | Define the complete visual system, layout, responsive behavior, card styling, status badges, priority treatment, spacing, typography, focus states, and deterministic hooks such as `.dashboard` and `.project-card`. |
| `app/project-data.json` | Coder, with Designer input | Provide deterministic sample content in a top-level `projects` array. Each project must contain `name`, `owner`, `status`, `recentActivity`, and `priority`; include contributor-friendly summary text through the agreed data or markup strategy. |
| `.vscode/launch.json` | Coder | Add the `Run Project Pulse Dashboard` configuration, serve from `${workspaceFolder}/app`, and open `index.html` rather than the app directory. |
| `.github/project-pulse-brief.md` | Reference only | Use as the product and workflow requirements source. Do not modify. |
| `.github/agents/*.agent.md` | Reference only | Follow the defined responsibilities and file-scope rules. Do not modify. |

No additional application files should be introduced unless the Orchestrator explicitly approves them. In particular, application behavior should remain within `app/index.html` if no JavaScript file is in scope.

## Agent responsibilities

### Orchestrator

- Translate this plan into ordered execution phases.
- Assign each specialist an explicit file scope.
- Prevent Designer and Coder from editing the same file concurrently.
- Resolve integration decisions, especially how JSON data is rendered and how the launch configuration serves the app.
- Review the integrated result against the brief and validation criteria.
- Ensure no agent stages, commits, or pushes changes.

### Planner

- Confirm repository conventions and available run/debug support before implementation.
- Maintain the phase order, ownership boundaries, dependencies, risks, and acceptance criteria in this plan.
- Identify any launch-environment dependency, such as the static server or browser debugger used by `.vscode/launch.json`.
- Review the final implementation plan for missing requirements and edge cases.

### Designer

- Define the dashboard’s information hierarchy so the first viewport clearly communicates that this is Project Pulse.
- Specify a polished card-based layout with visible project names, owners, status badges, recent activity, priority/risk, and contributor summaries.
- Establish readable spacing, typography, contrast, color meaning, responsive breakpoints, and keyboard focus treatment.
- Ensure status and priority are not communicated by color alone.
- Define responsive behavior for narrow screens and avoid layouts that require horizontal scrolling.
- Own all styling changes in `app/styles.css`.
- Report design decisions and validation recommendations to the Orchestrator.

### Coder

- Create `app/index.html` with semantic structure and the dashboard integration.
- Load `app/project-data.json` deterministically and render all required project fields.
- Decide and document the contributor-summary representation, ensuring each project has a short readable summary even though `summary` is not a required JSON field.
- Add clear loading and error states for data retrieval; do not silently display an empty dashboard when the JSON cannot be loaded.
- Create `app/project-data.json` with representative, deterministic project records.
- Create `.vscode/launch.json` with the exact launch name `Run Project Pulse Dashboard`.
- Set the launch working directory to `${workspaceFolder}/app`.
- Configure the launch flow to start a static server rooted at `app/` and open `/index.html` directly.
- Validate the launch behavior and report any environment dependency explicitly.

## Dependencies and implementation decisions

### Runtime dependencies

- The dashboard should use plain HTML, CSS, and browser JavaScript only.
- No framework, build tool, package manifest, or external runtime dependency is required.
- Loading JSON through `fetch()` requires the app to run through HTTP rather than directly from a `file://` URL. The launch configuration must therefore start or use a static server.
- The selected launch mechanism must be available in the Codespace. If a VS Code extension is required, identify it explicitly in the implementation report; do not silently assume a nonstandard extension.
- Prefer a deterministic server command already available in the environment, such as the Python standard-library HTTP server, if compatible with the repository’s Codespace. The server’s working directory must be `${workspaceFolder}/app`, and the browser URL must end in `/index.html`.

### Data contract

`app/project-data.json` must have this shape:

```json
{
  "projects": [
    {
      "name": "Example project",
      "owner": "Contributor name",
      "status": "Active",
      "recentActivity": "Short recent activity description",
      "priority": "High"
    }
  ]
}
```

The implementation must:

- Require a top-level `projects` array.
- Render every project record consistently.
- Display all five required fields.
- Use stable, human-readable status and priority values.
- Handle an empty array without breaking the page.
- Surface malformed or unavailable data through an explicit user-visible error state.
- Keep content deterministic so visual and manual validation are repeatable.

The brief requires a contributor-friendly summary but does not require a `summary` JSON property. Before implementation, the Orchestrator should choose one consistent approach: either add an optional `summary` property to each record and render it, or derive a short summary from the required fields. Adding `summary` is preferred because it preserves clear content ownership and avoids brittle text generation, while still retaining all required properties.

## Ordered implementation phases

### Phase 1: Requirements and integration decisions

**Owner:** Planner, coordinated by Orchestrator  
**Files:** None; reference `.github/project-pulse-brief.md`, agent definitions, and repository configuration.

Tasks:

1. Confirm the required deliverables and the required launch name.
2. Confirm that the final file set is limited to:
   - `app/index.html`
   - `app/styles.css`
   - `app/project-data.json`
   - `.vscode/launch.json`
3. Decide the JSON summary strategy.
4. Identify the static-server/debug mechanism for the launch configuration.
5. Record the acceptance criteria and handoff details for Designer and Coder.

**Dependency:** Must finish before implementation assignments are finalized.

### Phase 2: Parallel design and content/markup planning

**Owners:** Designer and Coder  
**Files:** Designer owns `app/styles.css` design direction; Coder owns `app/index.html` and `app/project-data.json`. No overlapping edits.

These tasks can run in parallel because they have separate file ownership:

**Designer tasks**

- Define the page structure conceptually: dashboard header, overview context, project grid, and project cards.
- Specify visual states for active, completed, blocked, at-risk, high-priority, and lower-priority projects as applicable.
- Define class hooks that the HTML must use, including `.dashboard` and `.project-card`.
- Specify responsive card behavior and accessibility requirements.
- Implement `app/styles.css` only if the Orchestrator assigns the file for direct editing during this phase.

**Coder tasks**

- Define semantic HTML structure and the rendering approach.
- Create deterministic project records with realistic names, owners, statuses, activity, priorities, and summaries.
- Ensure the HTML can consume the Designer’s agreed class hooks without embedding design-specific assumptions that conflict with the stylesheet.

**Handoff requirement:** Before integration, Designer and Coder must exchange the agreed class names, data fields, status vocabulary, and responsive assumptions. If the Designer directly edits `app/styles.css`, the Coder must not overwrite it.

### Phase 3: Functional implementation

**Owner:** Coder  
**Files:** `app/index.html`, `app/project-data.json`, `.vscode/launch.json`

Tasks:

1. Build the semantic HTML document with:
   - A meaningful page title.
   - A clearly identifiable Project Pulse heading.
   - A main dashboard region.
   - A project collection region with an accessible label.
   - Project cards that expose name, owner, status, recent activity, priority, and summary.
2. Add the minimal browser-side logic needed to load and render `project-data.json`.
3. Add explicit loading and error states.
4. Ensure status and priority values have text labels and are not conveyed through color alone.
5. Create the JSON data file with a valid top-level `projects` array.
6. Create `.vscode/launch.json` with:
   - Configuration name exactly `Run Project Pulse Dashboard`.
   - A static server rooted at `${workspaceFolder}/app`.
   - `cwd` set to `${workspaceFolder}/app`.
   - Browser launch/open behavior targeting `index.html`, such as `http://127.0.0.1:<port>/index.html`.
   - No URL that points only to the server root.
7. Keep `launch.json` strict JSON without comments.

**Dependency:** Requires Phase 1 decisions and the Designer/Coder contract from Phase 2. The Coder can implement the initial HTML and data while the Designer works on CSS, but final integration depends on the agreed hooks.

### Phase 4: Styling and integration

**Owner:** Designer, with Orchestrator integration review  
**Files:** `app/styles.css`; coordinated review of `app/index.html` and `app/project-data.json`

Tasks:

1. Implement the polished dashboard styling.
2. Add the required `.dashboard` and `.project-card` hooks.
3. Style status badges and priority/risk treatments with sufficient contrast.
4. Use rounded corners, restrained shadows, clear typography, and readable spacing.
5. Make the project grid responsive across desktop, tablet, and narrow mobile widths.
6. Add visible keyboard focus styles.
7. Ensure long project names, owners, activity text, and summaries wrap without overflowing.
8. Confirm that the stylesheet matches the actual class names and data states emitted by `index.html`.

**Dependency:** The Designer must have the Coder’s final class names and status/priority vocabulary before final integration. This phase must be sequential with final integration review if either agent needs to change shared markup assumptions.

### Phase 5: Integrated validation and correction

**Owner:** Orchestrator, with Coder and Designer corrections in their assigned files

Tasks:

1. Inspect all four deliverables together.
2. Confirm the data contract, HTML structure, CSS hooks, and launch URL agree.
3. Launch through the VS Code `Run Project Pulse Dashboard` configuration.
4. Verify that the first browser view is the Project Pulse UI from `app/index.html`.
5. Confirm that no directory listing appears.
6. Check loading, populated, empty, and data-error states.
7. Perform responsive and accessibility review.
8. Assign any corrections to the original file owner; do not allow uncoordinated cross-scope edits.
9. Repeat launch and UI validation after corrections.

**Dependency:** Must occur after Phases 3 and 4. Corrections may run in parallel only when they affect separate files and do not change shared contracts.

## Parallel-work decisions

### Work that can run in parallel

- Designer’s visual design and stylesheet implementation can run in parallel with the Coder’s initial data and semantic markup implementation because their primary files are separate.
- Planner’s repository and environment research can run in parallel with both design and content preparation.
- Independent validation of JSON syntax, stylesheet structure, and HTML semantics can run in parallel after the initial files exist.

### Work that must be sequential

- The launch configuration cannot be finalized until the static-server strategy and port/opening behavior are selected.
- Final stylesheet integration must follow agreement on HTML class hooks and status/priority values.
- Integrated browser validation must follow completion of all four files.
- Any change to the data contract, rendering structure, or class names requires a handoff to the other affected owner before final validation.
- Launch validation must verify the exact browser destination after the server is started; checking only that a server responds is insufficient.

## Edge cases and risks

- **Direct file opening:** `fetch()` may fail when `index.html` is opened with `file://`. The launch workflow must use HTTP.
- **Directory listing:** Opening `http://host:port/` may show a listing depending on the server. The launch configuration must target `/index.html` explicitly.
- **Missing or malformed JSON:** Show an explicit error message in the dashboard region rather than silently rendering no cards.
- **Empty project list:** Show a meaningful empty-state message while preserving the page layout.
- **Unexpected field values:** Render unknown status or priority values with readable fallback text and a neutral visual treatment.
- **Long content:** Prevent overflow from long names, activity descriptions, summaries, and owner names.
- **Color-only communication:** Pair colors with visible text or icons/labels.
- **Keyboard access:** Interactive elements, if any are added, must be keyboard reachable and visibly focused.
- **Narrow screens:** Cards must stack or reflow without horizontal scrolling.
- **Launch-port conflicts:** Use a deterministic port and document the expected behavior if that port is already in use; do not make the app appear successful while opening the wrong server.
- **Unsupported launch type:** Confirm the chosen `launch.json` configuration type works in the Codespace before considering the task complete.

## Validation expectations

### File and data validation

- All four required files exist.
- `app/project-data.json` is valid strict JSON.
- The JSON has a top-level `projects` array.
- Every project includes:
  - `name`
  - `owner`
  - `status`
  - `recentActivity`
  - `priority`
- Every project also has a contributor-friendly summary through the agreed implementation.
- `.vscode/launch.json` is valid strict JSON.
- The launch configuration name is exactly `Run Project Pulse Dashboard`.
- The launch configuration uses `${workspaceFolder}/app` as its working directory.

### UI validation

- The page title and visible heading identify Project Pulse.
- Project cards are clearly visible and visually distinct.
- Each card displays the project name, owner, status, recent activity, priority, and summary.
- Status badges and priority/risk treatments are readable and accessible.
- The layout has clear spacing and a polished visual hierarchy.
- The page remains usable at desktop and narrow mobile widths.
- Long content does not overflow or break the layout.
- Focus indicators and text contrast are sufficient for keyboard and low-vision users.
- Loading, empty, and data-error states are understandable.

### Launch validation

- Run the VS Code configuration named `Run Project Pulse Dashboard`.
- Confirm the server’s document root is `app/`.
- Confirm the browser opens a URL ending in `/index.html`.
- Confirm the initial browser view is the rendered Project Pulse dashboard.
- Confirm the initial view is not a directory listing.
- Confirm `project-data.json` loads successfully through the launched server.
- Confirm refreshing the opened page preserves the dashboard rather than navigating to a server index.

### Validation tooling

Use only tools already present in the repository or Codespace. At minimum:

- Parse or otherwise validate both JSON files.
- Inspect the rendered page in the launched browser.
- Use browser developer tools or the existing environment to confirm there are no failed data requests or uncaught rendering errors.
- Perform a manual responsive and keyboard pass.
- Do not add a new build system, test framework, or dependency solely for this dashboard.

## Completion criteria

The work is complete when the four assigned files are present, the dashboard renders deterministic project data with the required information, the visual design meets the Project Pulse expectations, and the `Run Project Pulse Dashboard` launch configuration serves `app/` while opening `index.html` directly instead of a directory listing.
