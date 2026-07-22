# Project Pulse — Implementation Plan

## 1. Overview

Project Pulse is a small, static, dependency-free dashboard that displays project status/"pulse" information (project name, status, priority, owner, last-updated date, etc.) for a set of projects. It is built from three static assets — HTML, CSS, and a JSON data file — with no build step and no backend. It must be runnable and previewable directly inside the VS Code Codespace using a `.vscode/launch.json` configuration, and it must look like a polished dashboard on first view, not a bare HTML page.

Goal: A learner can open the Codespace, launch Project Pulse via the configured launch target, and see a responsive, accessible dashboard of project cards rendered from `project-data.json`, styled per `app/styles.css`, structured and scripted per `app/index.html`.

## 2. File Assignments

| File | Purpose | Owner |
|---|---|---|
| `app/index.html` | Semantic HTML structure for the dashboard (header, `.dashboard` container, project card markup/template, footer); contains or references the JS that fetches `project-data.json` and renders `.project-card` elements into the DOM | **Coder** (structure/logic owner) — Designer contributes markup/semantic guidance that Coder implements |
| `app/styles.css` | All visual styling: dashboard layout, `.project-card` styling, status badges, priority treatment, spacing, typography, responsive breakpoints, focus states | **Designer** |
| `app/project-data.json` | Static data source: array of project objects (name, status, priority, owner/team, last updated, description, optional URL) that the dashboard renders | **Coder** (defines/authors schema and sample data; Designer may advise on which fields are needed for visual hierarchy — e.g., priority/status fields) |
| `.vscode/launch.json` | VS Code launch configuration to preview/open the dashboard from the Codespace (e.g., browser preview or Live Server-style launch pointing at `app/index.html`) | **Coder** |

Note: Because `.dashboard` and `.project-card` are CSS hooks that must exist in both HTML (as class names) and CSS (as selectors), Designer and Coder must agree on the exact hook names *before* either finalizes their file — see Dependencies below.

## 3. Designer Responsibilities

Scope: `app/styles.css` (owned) + markup/structural guidance for `app/index.html` (advisory, implemented by Coder).

1. **Layout**
   - Define the dashboard container as `.dashboard` — likely CSS Grid or Flexbox with `auto-fit`/`minmax` for responsive card wrapping.
   - Specify a max content width, consistent gutter/gap spacing, and page padding.
2. **Visual hierarchy**
   - Design `.project-card` treatment: card background, border-radius, box-shadow, padding.
   - Define status badge styling (e.g., `.status-active`, `.status-at-risk`, `.status-blocked` or similar) with distinct color coding — must not rely on color alone (see accessibility).
   - Define priority treatment (e.g., a colored left border, icon, or badge for High/Medium/Low) that is visually distinct from status.
   - Establish typographic scale: dashboard title, card title, metadata text, ensuring clear size/weight contrast.
3. **Accessibility**
   - Ensure color contrast ratios meet WCAG AA (4.5:1 text, 3:1 for large text/UI components) for all badge/text combinations.
   - Require status/priority to be conveyed with text labels or icons, not color alone.
   - Provide visible `:focus` states for any interactive elements (links, buttons) — no `outline: none` without a replacement.
   - Advise Coder on semantic HTML: use `<main>`, `<header>`, `<h1>`/`<h2>` hierarchy, `<ul>/<li>` or `<section>` for card lists, `aria-label` where needed, `alt` text if any images/icons are used.
4. **Responsive behavior**
   - Define breakpoints (e.g., single-column stack under ~600px, 2-column ~600–900px, multi-column grid above ~900px).
   - Ensure cards remain readable and tappable at mobile widths (no horizontal scroll, adequate touch target sizing).
5. **Deliverable format**
   - Final CSS in `app/styles.css` using the deterministic class hooks `.dashboard` and `.project-card` (required), plus any additional named classes needed for badges/priority (documented in code comments so Coder/learner can trace which classes map to which data field).
   - Report: design decisions made, tradeoffs (e.g., grid vs flexbox choice), and any markup requirements Coder must satisfy in `index.html` (exact class names/structure expected).

## 4. Coder Responsibilities

Scope: `app/index.html`, `app/project-data.json`, `.vscode/launch.json` (and any minimal inline/external JS needed).

1. **`app/project-data.json` — data schema (do this first)**
   - Define a JSON array of project objects. Suggested schema:
     ```json
     [
       {
         "id": "string",
         "name": "string",
         "status": "On Track | At Risk | Blocked | Completed",
         "priority": "High | Medium | Low",
         "owner": "string",
         "lastUpdated": "YYYY-MM-DD",
         "summary": "string"
       }
     ]
     ```
   - Populate with realistic sample data (5–8 projects) covering all status/priority variants so Designer's badge styling can be visually validated.
   - Keep it strict, valid JSON (no comments, no trailing commas).
2. **`app/index.html` — structure and rendering logic**
   - Build semantic HTML skeleton per Designer's guidance: `<header>` with dashboard title, `<main class="dashboard">` as the card container, optional `<footer>`.
   - Implement JS (inline `<script>` or `app/app.js` if introduced — confirm with Orchestrator if a separate JS file is preferred over inline) that:
     - Fetches `project-data.json` via `fetch('./project-data.json')`.
     - Handles fetch errors explicitly (e.g., render a visible error message in the dashboard if the fetch fails — important since `file://` fetch will fail without a server, reinforcing the need for `launch.json` to serve the app properly).
     - Renders each project as a `.project-card` element using the exact class hooks Designer specifies (`.project-card`, status/priority badge classes), including project name, status badge, priority indicator, owner, last-updated date, and summary text.
     - Uses safe DOM construction (e.g., `textContent` for user/data-driven text, not `innerHTML` with unsanitized interpolation) to avoid injection issues even though data is static/trusted.
   - Link `app/styles.css` via `<link rel="stylesheet" href="styles.css">`.
3. **`.vscode/launch.json` — preview/run configuration**
   - Because this is a static HTML/CSS/JSON app with a `fetch()` call, it **cannot** be opened via bare `file://` (fetch of local JSON will be blocked by CORS/file-protocol restrictions in most browsers). The launch config must serve the app over HTTP, not just open the file directly.
   - Determine the serving mechanism (this is an open question — see below) — likely options:
     - Use the **Live Preview** / **Live Server** VS Code extension launch config type, or
     - A simple `"type": "node-terminal"`/`"type": "pwa-chrome"` config pointing at a local server URL after starting a lightweight static server (e.g., `python3 -m http.server` or `npx serve`) with `cwd` set to `${workspaceFolder}/app`.
   - Per Coder agent rules: set `cwd` to `${workspaceFolder}/app`, ensure the config opens `index.html` (or the served root, which serves `index.html` by default) rather than a directory listing, use strict JSON (no comments), and use deterministic names/ports (e.g., a fixed port like `5500` or `8080`).
   - Validate the launch config actually opens the dashboard successfully within the Codespace port-forwarding context (Codespaces forward ports automatically; confirm the launch config's URL/port aligns with what gets forwarded).

## 5. Dependencies

1. **Data schema before rendering logic**: `app/project-data.json` schema must be finalized before Coder writes the fetch/render logic in `app/index.html`, since field names drive the JS rendering code and the DOM structure Designer needs to style.
2. **CSS hook agreement before either file is finalized**: Designer's `.dashboard` / `.project-card` (and badge/priority class) naming must be agreed upon (even informally, e.g., via a shared comment block or quick sync) before Coder finalizes `index.html` markup and before Designer finalizes `styles.css` — otherwise selectors won't match generated markup. This can happen via Designer producing a short markup contract first, or Coder drafting a minimal skeleton first and Designer adapting — either order works as long as the class names converge.
3. **Serving mechanism before `launch.json`**: The Coder must decide/confirm how the app will be served (Live Preview extension vs. simple HTTP server command) before finalizing `.vscode/launch.json`, because the fetch-based JSON loading requires an HTTP context, not a raw file open. This decision should be made early since it affects `index.html`'s fetch behavior/testability.
4. **`index.html` before final `styles.css` polish**: Designer can draft styles against the agreed class-hook contract in parallel with Coder's HTML build, but final visual QA of `styles.css` (spacing, card sizing, badge alignment) depends on the actual rendered DOM output from `index.html` + `project-data.json`, so a final integration/visual pass happens after both are substantially complete.

## 6. Parallel Work Decisions

**Can run in parallel (non-overlapping file scope, no blocking data dependency):**
- Designer working on `app/styles.css` visual design **and** Coder authoring `app/project-data.json` schema/sample data — these touch different files and don't depend on each other's output (Designer just needs to know general field types like "status" and "priority" exist, which can be communicated as a quick assumption/contract up front rather than waiting).
- Coder drafting the semantic skeleton of `app/index.html` **and** Designer drafting `app/styles.css` against an agreed class-hook contract — as long as both sides commit to the same class names (`.dashboard`, `.project-card`, badge classes) before starting, they can work simultaneously.
- Coder investigating/prototyping `.vscode/launch.json` serving strategy can happen in parallel with Designer's CSS work, since it's an independent file with no dependency on styling.

**Must run sequentially (overlapping scope, blocking dependency, or needs prior output):**
- Finalizing `project-data.json` field names → then locking the JS rendering logic in `index.html` that reads those fields (rendering logic cannot be finalized until schema is stable).
- Confirming the serving mechanism (server type/port) → then writing the final `.vscode/launch.json` (launch config is meaningless without knowing what it's launching against).
- Integration/visual QA pass (verifying rendered cards match Designer's intended styling) must happen **after** both `index.html` (with real rendering) and `styles.css` are in place — this is a final sequential checkpoint before sign-off, not something that can be parallelized.

## 7. Validation Expectations

1. **Load without errors**: Opening the dashboard via the `.vscode/launch.json` configuration loads `app/index.html` with zero console errors/warnings in the browser dev tools.
2. **Data renders correctly**: All entries in `app/project-data.json` appear as distinct `.project-card` elements with correct name, status, priority, owner, last-updated date, and summary text — verify at least one card per status/priority variant renders with the correct corresponding badge styling.
3. **Fetch/serving works**: Confirm the app is served over HTTP (not opened as a raw `file://` path) so the `fetch('./project-data.json')` call succeeds; if it fails, the dashboard should show a graceful, visible error message rather than a blank page.
4. **Launch config works end-to-end**: `.vscode/launch.json` successfully starts/opens a preview from a clean Codespace state, lands on `index.html` (not a directory listing), and uses `cwd` = `${workspaceFolder}/app`.
5. **Responsive layout check**: Resize/simulate viewport widths (mobile ~375px, tablet ~768px, desktop ~1200px+) and confirm cards reflow per Designer's breakpoints without horizontal scrolling or overlapping content.
6. **Accessibility check**:
   - Semantic HTML present (`<header>`, `<main>`, appropriate heading levels).
   - Color contrast of text/badges meets WCAG AA when spot-checked.
   - Status/priority are distinguishable by more than color alone (text label or icon present).
   - Keyboard focus is visible on any interactive elements.
   - Any images/icons include `alt` text or `aria-hidden` if purely decorative.
7. **JSON validity**: `app/project-data.json` parses as strict valid JSON (no trailing commas/comments) — validate with a JSON linter or `JSON.parse` in devtools console.
8. **Visual match to intent**: First view of the dashboard should immediately read as "a Project Pulse dashboard" — visible cards, badges, spacing, and typography — not a bare unstyled HTML page (per Designer agent's explicit expectations).

## 8. Open Questions

- **Serving mechanism for `launch.json`**: Should the Coder rely on a VS Code extension (e.g., Live Preview / Live Server) assumed to be available in the devcontainer, or should the launch config spin up a plain `python3 -m http.server` / `npx serve` process via a `preLaunchTask` in `tasks.json`? This affects whether `.devcontainer` needs an extension recommendation added. Needs Orchestrator/learner decision before Coder finalizes `launch.json`.
- **Separate JS file vs. inline script**: Should rendering logic live inline in `app/index.html` (simplest, matches "small static dashboard" framing) or in a new `app/app.js` file (cleaner separation, but adds a file not explicitly listed in the required file set)? Recommend inline `<script>` in `index.html` unless Orchestrator/learner wants a dedicated JS file — flagging since the task's required file list only names four files.
- **Exact badge/priority class-naming contract**: Not yet agreed between Designer and Coder; must be settled at kickoff (quick sync or a shared short contract doc) to avoid rework.
- **Devcontainer port/extension availability**: Unconfirmed whether the Codespace's `.devcontainer` config already includes a live-server-capable extension; may need a quick check of `.devcontainer/` contents before Coder commits to a specific `launch.json` approach.
