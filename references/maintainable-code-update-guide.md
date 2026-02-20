# Maintainable Code Update Guide

Use this guide after runtime extraction and before finalizing code changes.

## Objective

Reproduce behavior and visuals from runtime evidence while keeping code easy to read, test, and evolve.

## Core Principles

- Evidence first: every behavior/style change should map to captured evidence.
- Minimal surface change: preserve existing public APIs unless parity requires change.
- Separation of concerns: avoid mixing state machine logic, markup, and style mapping in one block.
- Deterministic output: avoid hidden time-based/stateful side effects unless required by captured behavior.

## Required Implementation Standards

1. Component boundaries
- Split oversized components into leaf primitives, state containers, and utility helpers.
- Prefer one responsibility per component.
- Keep shared logic in hooks/utilities, not duplicated event handlers.

2. Props and types
- Use explicit TypeScript interfaces for all public component props.
- Keep optional props truly optional with explicit defaults.
- Avoid `any` and avoid widening unions without evidence.
- Keep prop names domain-clear and stable.

3. State and behavior
- Model UI states with explicit unions/enums when states are finite.
- Keep derived state derived; do not duplicate source-of-truth state.
- Keep event handlers pure and intention-revealing.
- Isolate browser globals (`window`, `navigator`, `document`) behind guards or adapters.

4. Semantics and accessibility
- Do not nest interactive elements (`button` inside `a`, `a` inside `button`).
- Use anchor for navigation and button for in-place actions.
- Ensure interactive controls have accessible names.
- Preserve keyboard access and focus visibility.
- Keep ARIA minimal and valid; prefer native semantics first.

5. Rendering and performance
- Use stable keys for dynamic lists; avoid index keys when order/content can change.
- Memoize only when there is clear benefit and stable dependencies.
- Avoid unnecessary re-renders caused by inline object/function churn in hot paths.
- Keep expensive parsing/formatting outside render when feasible.

6. Styling and design tokens
- Reuse existing token system (CSS vars/theme) instead of redefining tokens per file.
- Avoid duplicating global `:root` variables across multiple component stylesheets.
- Keep class naming consistent and local to component ownership.
- Preserve responsive behavior with clear breakpoints and no contradictory rules.

7. Code hygiene
- Keep files focused; extract helper modules when file complexity grows.
- Prefer declarative code over imperative DOM manipulation.
- Add short comments only for non-obvious intent.
- Remove dead code, stale props, and unused captures before shipping.

## Update Workflow (Patch Existing Code)

1. Map captures to existing components.
- Create a mapping table: `captured identity -> local component file`.
- Identify which components require behavior change vs style-only change.

2. Define safe contract.
- Mark stable props and exported interfaces that should remain unchanged.
- If a contract change is required, document migration impact.

3. Implement in passes.
- Pass A: semantic/structure fixes.
- Pass B: state and interaction parity.
- Pass C: style parity and responsive adjustments.
- Pass D: cleanup/refactor for readability.

4. Validate continuously.
- Run structure diff checks against fixtures.
- Re-check critical interaction variants.
- Re-check style anchor nodes and typography.
- Re-run lint/tests if available.

## Hard Fail Conditions (Must Fix Before Delivery)

- Invalid interactive nesting.
- Broken keyboard interaction after update.
- Regressions in captured core states.
- New duplicate global design tokens.
- Unstable list keys for non-static collections.
- Added `any` in public interfaces without justification.

## Recommended Deliverable Notes

Include a short maintainability section in final output:

- Files changed and why.
- Public API changes (if any).
- Refactors done for readability/testability.
- Known limitations and non-deterministic behavior exceptions.
- Follow-up improvements not required for parity.
