# Chrome DevTools MCP Playbook

Use this for reliable evidence capture with DevTools MCP tools.

## Core Sequence

1. Create and lock context.
- `new_page` for each target environment (source site, local clone).
- `list_pages` and `select_page` before each capture block.

2. Stabilize UI.
- `take_snapshot` to confirm key nodes exist.
- Dismiss blocking overlays via `click`.
- Re-run `take_snapshot` to verify unobstructed targets.

3. Capture with script-first extraction.
- Use one `evaluate_script` to return all related evidence in one payload:
  - props
  - owner chain
  - HTML snippets
  - bounding rects
  - computed styles
- Prefer structured JSON output over screenshots.

4. Capture interaction variants deterministically.
- Pattern: `click` -> `wait_for` marker -> `evaluate_script` assertion.
- Assert state change explicitly (class, aria state, text, value, style token).
- If state does not mutate in headless mode, mark as non-deterministic and infer behavior from runtime source (`type.toString()`).

5. Capture stylesheet evidence.
- `list_network_requests` filtered to stylesheet resources.
- `get_network_request` with `responseFilePath` to save CSS.
- Extract matching rules by class tokens from target subtree.

6. Cross-tab parity checks.
- Keep source and reconstruction in separate tabs.
- Run the same style capture function on both tabs.
- Diff only anchor nodes, not the entire DOM.

## Reliability Rules

- Always verify selected page before `evaluate_script`.
- Capture and log viewport dimensions with every run.
- Reload with `navigate_page` (`type: "reload"`, `ignoreCache: true`) after style changes.
- Save raw JSON/CSS artifacts for reproducibility.
- Prefer `take_snapshot` and scripts for actionable evidence; screenshots are secondary.

## Pitfalls And Fixes

- Wrong-tab capture:
  - Fix: enforce `list_pages` + `select_page` checkpoints.

- Fiber missing on chosen node:
  - Fix: climb to nearest composite fiber via parent chain.

- Non-repeatable measurements:
  - Fix: keep a fixed viewport and capture in one session.

- Visual drift despite similar CSS:
  - Fix: verify font loading (`document.fonts.check`) and localize missing font assets.
