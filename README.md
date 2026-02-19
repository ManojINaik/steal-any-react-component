# Steal Any React Component

Website-agnostic skill pack for reconstructing React components from live websites using runtime Fiber evidence, DOM snapshots, interaction evidence, and style parity checks.

## Repo Structure

- `SKILL.md`
- `agents/openai.yaml`
- `references/fiber-capture-snippets.md`
- `references/style-parity-snippets.md`
- `references/chrome-devtools-mcp-playbook.md`
- `README.md`

## Prerequisites

- Node.js `20.19.0+` or `22.12.0+`
- `npm`
- Chrome installed locally (for `chrome-devtools-mcp`)

## 1) Install The Skill (`npx skills add`)

Skills docs:
- https://skills.sh/docs

Since this repo contains a single skill at root, preferred command:

```bash
npx skills add ManojINaik/steal-any-react-component
```

Full URL also works:

```bash
npx skills add https://github.com/ManojINaik/steal-any-react-component
```

Then use:

```text
$react-fiber-reconstruction reconstruct the hero component from https://example.com
```

## 2) Setup Chrome DevTools MCP

Project:
- https://github.com/ChromeDevTools/chrome-devtools-mcp

Generic MCP config example:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

Codex CLI example:

```bash
codex mcp add chrome-devtools -- npx chrome-devtools-mcp@latest
```

Claude Code example:

```bash
claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest
```

## 3) Recommended Usage Flow

1. Add the skill with `npx skills add ...`.
2. Enable Chrome DevTools MCP in your client.
3. Prompt with the skill trigger + URL + component scope.
4. Capture Fiber + DOM + interaction + stylesheet evidence.
5. Reconstruct and verify structure/style parity.

## 4) Troubleshooting

- No Fiber found on a selected node:
  - Move up DOM ancestors and capture nearest composite fiber.
- State interactions not updating in headless mode:
  - Validate behavior from `type.toString()` and document non-deterministic gaps.
- Style mismatch:
  - Diff computed styles on anchor nodes and port matching stylesheet rules.
- Typography mismatch:
  - Check `document.fonts.check(...)` and host missing webfonts locally.

## 5) Notes

- Use only on websites/components you are authorized to inspect/reproduce.
- Keep raw capture artifacts for reproducibility and auditing.

## References

- Skills docs: https://skills.sh/docs
- Chrome DevTools MCP: https://github.com/ChromeDevTools/chrome-devtools-mcp
