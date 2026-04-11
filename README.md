<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="banner-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="banner-light.svg" />
  <img src="banner-light.svg" alt="VERTAAUX" width="800" />
</picture>

[![npm](https://img.shields.io/npm/v/%40vertaaux%2Fmcp-server?label=%40vertaaux%2Fmcp-server)](https://www.npmjs.com/package/@vertaaux/mcp-server)
[![MCP Registry](https://img.shields.io/badge/MCP_Registry-listed-0f172a)](https://registry.modelcontextprotocol.io)
[![License](https://img.shields.io/github/license/VertaaUX/mcp-server)](LICENSE)
[![Smithery Skills](https://img.shields.io/badge/Smithery-vertaaux-orange)](https://smithery.ai/skills/petri-lahdelma/vertaaux)

**MCP server for VertaaUX.ai — run UX & accessibility audits, generate fixes, and monitor quality from your LLM or IDE.**

</div>

This server exposes 38 tools via the [Model Context Protocol](https://modelcontextprotocol.io) so AI coding agents can audit live URLs, explain findings, generate framework-aware patches, and open draft PRs — all without leaving the conversation.

## Quick Start

### MCP Official Registry

```bash
npx -y @modelcontextprotocol/cli install io.github.PetriLahdelma/vertaaux-mcp
```

### npm

```bash
npm install -g @vertaaux/mcp-server
VERTAAUX_API_KEY=vx_live_... vertaaux-mcp
```

### npx (zero install)

```bash
VERTAAUX_API_KEY=vx_live_... npx -y @vertaaux/mcp-server
```

Get your API key at [vertaaux.ai/settings/api](https://vertaaux.ai/settings/api).

## IDE Integration

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "vertaaux": {
      "command": "npx",
      "args": ["-y", "@vertaaux/mcp-server"],
      "env": {
        "VERTAAUX_API_KEY": "vx_live_..."
      }
    }
  }
}
```

### VS Code

Add to `.vscode/settings.json`:

```json
{
  "mcp.servers": {
    "vertaaux": {
      "command": "npx",
      "args": ["-y", "@vertaaux/mcp-server"],
      "env": {
        "VERTAAUX_API_KEY": "vx_live_..."
      }
    }
  }
}
```

### Cursor

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "vertaaux": {
      "command": "npx",
      "args": ["-y", "@vertaaux/mcp-server"],
      "env": {
        "VERTAAUX_API_KEY": "vx_live_..."
      }
    }
  }
}
```

## What Your Agent Can Do

Once connected, your AI agent gains access to 38 tools across the full audit-to-fix lifecycle:

### Audit

| Tool | Description |
|---|---|
| `audit_url` | Run UX & accessibility audit on a deployed URL |
| `audit_repo` | Static analysis on local codebase (React/Vue/Svelte/HTML) |
| `audit_artifact` | Audit from HAR files or Lighthouse JSON |
| `audit_a11y` | Multi-engine accessibility audit (axe-core + AccessLint + custom) |
| `compare_competitors` | Side-by-side UX metrics against competitor URLs |
| `capture_screenshot` | Capture annotated screenshot of a URL |

### Fix

| Tool | Description |
|---|---|
| `explain_finding` | Deep-dive: WCAG criteria, repro steps, before/after code |
| `suggest_fix` | Framework-aware search/replace patch with confidence score |
| `generate_pr` | Open a draft GitHub PR with patches applied atomically |
| `run_verification_suite` | Verify a patch resolves the issue before committing |

### Monitor

| Tool | Description |
|---|---|
| `diff_audits` | Compare two audits — fixed, broken, and regressed findings |
| `verify_fixes` | Post-deploy verification against a baseline |
| `create_schedule` | Cron-based recurring audits with score alerts |
| `policy_check` | Evaluate audit against `vertaa.policy.yml` thresholds |

### Manage

| Tool | Description |
|---|---|
| `get_findings` | Paginated findings with severity/rule filtering |
| `triage_finding` | Set triage state (open/in_progress/resolved/wont_fix) |
| `create_webhook` | Register webhooks for audit event notifications |
| `get_quota` | Check remaining credits and tier |
| `generate_sarif` | Export audit as SARIF 2.1.0 for GitHub Code Scanning |

## Workflow Example

```
1. audit_url({ url: "https://staging.myapp.com", mode: "deep" })
   → Returns scored findings across 7 UX categories

2. suggest_fix({ audit_id: "...", finding_id: "button-name:a1b2c3d4" })
   → Framework-aware patch (React/Vue/Svelte detected automatically)

3. generate_pr({ patches: [...], repo: "owner/repo", branch: "fix/a11y" })
   → Draft PR with patches applied atomically via Git Trees API

4. verify_fixes({ url: "...", baseline_audit_id: "..." })
   → Confirms fixes landed in production
```

## Scoring Categories

Every audit scores across seven weighted categories:

| Category | Weight | What it measures |
|---|---|---|
| Accessibility | 20% | WCAG compliance, ARIA usage, color contrast |
| Conversion | 20% | CTA clarity, form UX, trust signals |
| Usability | 20% | Navigation, cognitive load, responsiveness |
| Clarity | 15% | Value proposition, scanability, messaging |
| Information Architecture | 10% | Navigation depth, grouping, consistency |
| Semantic Markup | 8% | Heading hierarchy, landmark regions, HTML5 |
| Keyboard Navigation | 7% | Focus order, tab traps, skip links |

## Environment Variables

| Variable | Required | Default | Purpose |
|---|---|---|---|
| `VERTAAUX_API_KEY` | Yes | — | API authentication key |
| `VERTAAUX_API_BASE` | No | `https://vertaaux.ai/api/v1` | API endpoint URL |
| `PORT` | No | `8787` | HTTP transport port |
| `GITHUB_TOKEN` | No | — | GitHub API access for `generate_pr` |

## Prompt Templates

Pre-built workflow prompts for common scenarios:

| Prompt | Description |
|---|---|
| `quick_audit` | Audit a URL and summarize top issues with fix recommendations |
| `fix_accessibility` | Full audit → patch → PR comment workflow |
| `compare_ux` | Compare against competitors and identify UX gaps |
| `monitor_regression` | Set up scheduled monitoring with alerts |
| `audit_codebase` | Static analysis on local codebase |

## Resources

The server exposes MCP resources via `vertaa://` URIs:

| URI Pattern | Description |
|---|---|
| `vertaa://audits/{auditId}` | Full audit result |
| `vertaa://audits/{auditId}/summary` | Lightweight summary |
| `vertaa://audits/{auditId}/findings/{findingId}` | Single finding detail |
| `vertaa://history/{encodedUrl}` | Audit history for URL |
| `vertaa://history/{encodedUrl}/trend` | Score trend analysis |
| `vertaa://guidelines/{topic}` | UX guidelines (buttons, forms, navigation, color-contrast, errors, content) |

## Related

- [@vertaaux/cli](https://www.npmjs.com/package/@vertaaux/cli) — CLI for terminal and CI/CD workflows
- [@vertaaux/sdk](https://www.npmjs.com/package/@vertaaux/sdk) — JavaScript SDK
- [VertaaUX Agent Skills](https://github.com/VertaaUX/agent-skills) — Agent Skills for Claude Code, Codex, Cursor, and more
- [vertaaux.ai](https://vertaaux.ai) — Web dashboard

## License

MIT
