---
name: react-fiber-reconstruction
description: Reconstruct React components from running websites without source access by using React Fiber metadata and DOM/CSS snapshots. Use for clone/recreate/reverse-engineer requests on production React UI, including props-to-HTML capture, type.toString extraction, structural and computed-style verification, stylesheet/font parity, dependency ordering, and fallbacks for animation or state-driven mismatches.
---

# React Fiber Reconstruction

Use this skill to rebuild React components from live pages with evidence-first extraction and a verification loop.

## When To Use

- Reconstructing React UI from a running website when source code is unavailable.
- Rebuilding one or more production components with behavior and style parity requirements.
- Reverse-engineering component props/state from live runtime evidence.

## Workflow

1. Confirm scope and permission.
- Verify the user is authorized to inspect and reproduce the target UI.
- Restrict extraction and output to approved components/assets.

2. Capture runtime evidence from the page.
- Open the target route in a desktop browser context.
- Identify target DOM nodes and retrieve attached React Fiber nodes.
- Record per instance:
  - component identity key (in-memory `fiber.type` identity)
  - display name (`displayName` or `name`)
  - serializable props
  - rendered HTML snippet (`outerHTML`)
  - owner-chain path
  - `type.toString()` (when accessible)

3. Capture state and interaction variants.
- Capture at least 3 meaningful variants when possible (default, active, submitted/open, disabled/loading, etc.).
- For each interaction, use deterministic assertion:
  - action
  - wait/confirm marker
  - post-action capture
- If scripted interactions do not mutate state reliably, infer behavior from `type.toString()` and document it.

4. Capture style and asset evidence.
- Capture computed styles for anchor nodes (container, title, subtitle, CTA cluster, controls).
- Capture active/inactive style deltas for stateful controls.
- Export relevant stylesheet responses and isolate rules for captured classnames/selectors.
- Record font stack and font-load status (`document.fonts.check(...)`) for key text nodes.

5. Group by component identity.
- Bucket captures by `fiber.type` identity.
- Keep grouped props/HTML examples together for each component.
- Preserve provenance for each capture (URL, timestamp, viewport).

6. Reconstruct code leaf-to-root.
- Reconstruct leaf components first.
- Reconstruct parent components only after leaf verification passes.
- Require typed props, explicit defaults, deterministic rendering, and clear fallbacks.

7. Verify structure (render-and-diff loop).
- Render against captured fixtures.
- Normalize HTML before diffing.
- Feed focused mismatches back into patch iterations.
- Retry 3-5 rounds before fallback.

8. Verify visual parity.
- Compare computed-style snapshots between target and reconstructed components at anchor nodes.
- Verify typography and spacing (font family, weight, size, line height, letter spacing, max-width, gaps).
- Port captured stylesheet rules when broad visual drift exists.
- Re-check font loading; if failing, host needed fonts locally and re-verify.

9. Apply fallback for non-deterministic behavior.
- If mismatch is caused by animation timing, environment variance, or hidden internal state, freeze unstable sections as snapshot wrappers.
- Explicitly document unresolved deterministic gaps.

10. Assemble final output.
- Deliver reconstructed component code, required styles/assets, and verification artifacts.
- Include explicit limitations and residual risks.

## Capture Rules

- Prefer observed evidence over inferred behavior.
- Omit non-serializable values (functions, DOM nodes, symbols) and note omissions.
- Keep captures in one browser session when type identity stability matters.
- Use website-agnostic selector strategy in analysis (semantic anchors first, class hashes second).
- Keep raw logs for reruns and audits.

## Deliverables

- Reconstructed component files with typed interfaces.
- `verification-report.md` with fixture-level pass/fail.
- `style-parity-report.md` with key-node computed-style and typography checks.
- `limitations.md` for dynamic-state, animation, or environment caveats.

## References

- Read `references/fiber-capture-snippets.md` before runtime extraction.
- Read `references/style-parity-snippets.md` for computed-style/font/stylesheet parity checks.
- Read `references/chrome-devtools-mcp-playbook.md` for optimal DevTools MCP capture flow and pitfalls.
- Use official React documentation and web search when React internals need confirmation.
