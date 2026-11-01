# Project agent instructions

## Project

Personal CV: one self-contained HTML page for Paul-Andrei Abrudan, served at `https://foarte.top/cv`. Status: live only after the user copies `index.html` to the server; edits here are not live until then.

## Stack and layout

- One file: `index.html`, with inline CSS and JavaScript. Fonts load from Google Fonts. No package manager, build step or framework.
- Canonical and Open Graph URLs point to `https://foarte.top/cv`. Keep them in sync if the URL changes.
- The server copy lives at `/srv/foarte-cv/index.html`. The user installs it there; see `REDEPLOY.md` in the `foarte-top/minipc-frontend` repo.

## Coding style and rules

- Follow existing patterns and match the surrounding code.
- Keep it one file with inline CSS and JS. Do not add a build step or a framework.
- Do not add comments unless asked.
- Commit or push only when asked. Never add AI co-author trailers (e.g.
  `Co-Authored-By: Claude`) to commits.

## Testing and verification

- No automated tests or linter are configured. Open `index.html` in a browser and check the layout at phone width (no horizontal scroll) and at desktop width.
- Do not call the change done until that check has been made.

## Output-heavy commands

- **Delegate output-heavy commands to a cheap model.** When a command's output
  is large but the answer you need is small (parsing logs, sifting build or CI
  output, scanning long dumps), prefer handing it to a cheap subagent and
  taking back only the conclusion: DeepSeek via the `deepseek` MCP
  (`run_task`) or Claude Haiku. Give it the exact command and what to report;
  it reads, it does not edit. This needs no explicit opt-in.
- **Test runs print failures, not passes.** Run suites in a mode that hides
  passing tests and shows only failures plus the summary: e.g. `cargo test -q`,
  `cargo nextest run --status-level fail`, `pytest -q`, `vitest --reporter=dot`,
  `go test ./...` without `-v`, `dotnet test -v q`. If the runner has no such
  mode, filter its output down to the failures. Never pull hundreds of passing
  lines into context.

## Planning

- For large tasks, plan first with a strong model, then implement with cheaper
  models to save tokens.
- State the goal and the stopping condition before starting.

## Subagent usage

These rules are always in effect, even when the `dispatching-parallel-agents`
skill is not loaded. If you do dispatch subagents, follow them; the skill has
the full workflow.

- **Explicit opt-in only.** Never dispatch, spawn, or delegate to subagents
  unless the developer explicitly asks for it in the current task (e.g. "use
  subagents", "dispatch these in parallel", "delegate this"). A task looking
  parallelizable is not authorization, and neither is a past instruction. If
  parallel work would clearly help, ask once; absent a yes, do the work yourself,
  sequentially. Exception: delegating an output-heavy command to a cheap model
  (see "Output-heavy commands") needs no opt-in.
- **Cheap models for subagents.** The coordinating session runs the expensive
  model; helpers run a cheap/fast one: ChatGPT Luna, DeepSeek v4.1 Flash, Claude
  Sonnet, or Muse-Spark. Pick the cheapest that can do the job, never silently a
  premium model, and state the chosen model in the dispatch and the summary.
- **Poll at the harness cadence.** Polling too often destroys prompt-cache
  retention. Claude Code: check every 4 minutes. Codex and opencode running
  OpenAI models: check every 25 minutes, otherwise wait for completion, and do
  not poll faster. Manual polling only — no tight auto-poll loops. If a result
  arrives earlier, use it immediately.
