# Steal Any React Component

Website-agnostic skill pack for reconstructing React components from live websites using runtime Fiber evidence, DOM snapshots, and style parity checks.

## What Is In This Repo

- `react-fiber-reconstruction/`
  - `SKILL.md`
  - `agents/openai.yaml`
  - `references/fiber-capture-snippets.md`
  - `references/style-parity-snippets.md`
  - `references/chrome-devtools-mcp-playbook.md`

## Prerequisites

- Node.js `20.19.0+` or `22.12.0+`
- `npm`
- Chrome installed locally (for `chrome-devtools-mcp`)

Chrome DevTools MCP requirements reference:
- https://github.com/ChromeDevTools/chrome-devtools-mcp

## 1) Install The Skill With `npx skills add`

Skills CLI docs:
- https://skills.sh/docs

Install this repo and select the skill folder:

```bash
npx skills add https://github.com/ManojINaik/steal-any-react-component --skill react-fiber-reconstruction
```

Then use it in prompts like:

```text
$react-fiber-reconstruction reconstruct the hero component from https://example.com
```

## 2) Setup Chrome DevTools MCP

Official project:
- https://github.com/ChromeDevTools/chrome-devtools-mcp

### Generic MCP config example

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

### Codex CLI example

```bash
codex mcp add chrome-devtools -- npx chrome-devtools-mcp@latest
```

### Claude Code example

```bash
claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest
```

## 3) Recommended Usage Flow

1. Add the skill with `npx skills add ... --skill react-fiber-reconstruction`.
2. Enable Chrome DevTools MCP in your client.
3. Prompt with the skill trigger:
   - `$react-fiber-reconstruction` + target URL + component scope.
4. Capture Fiber + DOM + stylesheet evidence.
5. Reconstruct component and run structure/style parity verification.

## 4) Troubleshooting

- No Fiber found on selected node:
  - Move up DOM ancestors and capture nearest composite React fiber.
- State interactions do not update in headless mode:
  - Validate behavior from `type.toString()` and document non-deterministic gaps.
- Style mismatch after reconstruction:
  - Diff computed styles on anchor nodes and port matching stylesheet rules.
- Typography mismatch:
  - Check `document.fonts.check(...)` and host missing webfonts locally.

## 5) Notes

- Use this only for websites and components you are authorized to inspect/reproduce.
- Keep raw capture artifacts for reproducibility and auditing.

## References

- Skills docs: https://skills.sh/docs
- Chrome DevTools MCP: https://github.com/ChromeDevTools/chrome-devtools-mcp
