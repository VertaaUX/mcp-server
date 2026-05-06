# VertaaUX MCP Server

The only MCP server with an autonomous audit, fix, and verify loop. Detects UX and accessibility issues across 7 categories, generates framework-aware patches (React, Vue, Angular, Svelte), opens atomic GitHub PRs via the Git Trees API, and verifies the fix landed in production. Built for CI/CD pipelines with policy-as-code thresholds.

## Why this server is different

- **`verify_fixes` loop**: close the audit, fix, re-audit cycle without leaving the agent loop. Budget-capped at 3 iterations to prevent runaway billing.
- **Framework-aware patches**: `suggest_fix` detects React/Vue/Angular/Svelte/Nuxt via the nearest `package.json` and emits idiomatic patches (JSX rewrites for React, HTML attrs preserved elsewhere).
- **Atomic Git Trees PRs**: `generate_pr` applies N patches in a single commit or zero. Conflict graph + AST gate (Babel, vue-eslint-parser, svelte/compiler) refuse unparseable patches before they reach the PR.
- **Deterministic finding IDs**: `rule:hash` format stable across audit runs so agents can reference findings without storing state.
- **Multi-engine a11y**: `audit_a11y` combines axe-core, AccessLint, and VertaaUX analyzers in a single call.
- **Policy-as-code**: `policy_check` mirrors the GitHub Action's threshold evaluator exactly so CI and agent verdicts match.

## Features

- **38 Tools** across audit, fix, PR, schedule, webhook, policy, and a11y categories
- **7 Prompt Templates** for common workflows
- **8 Resource URIs** for audit data and UX guidelines
- **Enterprise Controls**: domain allowlist, rate limiting, PII redaction
- **Dual Transport**: stdio (CLI/Desktop) + HTTP streaming (web)
- **Official MCP SDK**: spec-compliant via `@modelcontextprotocol/sdk`

## Install

### MCP Official Registry

```bash
npx -y @modelcontextprotocol/cli install io.github.PetriLahdelma/vertaaux-mcp
```

### npm

```bash
npm install -g @vertaaux/mcp-server
VERTAAUX_API_KEY=vx_live_... vertaaux-mcp
```

> **Drift policy:** `smithery.yaml`, `glama.json`, and `server.json` are auto-generated from the live MCP tool registry by `npm run generate:manifests`. **Never hand-edit them.** See [`docs/REGISTRY-PUBLISHING.md`](docs/REGISTRY-PUBLISHING.md) for the runbook.

## Quick Start

```bash
# Install & build
npm install && npm run build

# Run (stdio transport, for Claude Desktop, VS Code, Cursor)
VERTAAUX_API_KEY=vx_live_... npm start

# Run (HTTP transport, for web clients)
VERTAAUX_API_KEY=vx_live_... npm run start:http
```

## IDE Integration

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "vertaaux": {
      "command": "node",
      "args": ["/path/to/mcp-server/dist/index.js"],
      "env": {
        "VERTAAUX_API_KEY": "vx_live_..."
      }
    }
  }
}
```

### VS Code (with MCP extension)

Add to `.vscode/settings.json`:

```json
{
  "mcp.servers": {
    "vertaaux": {
      "command": "node",
      "args": ["./mcp-server/dist/index.js"],
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
      "command": "node",
      "args": ["/path/to/mcp-server/dist/index.js"],
      "env": {
        "VERTAAUX_API_KEY": "vx_live_..."
      }
    }
  }
}
```

## Environment Variables

| Variable | Required | Default | Purpose |
|---|---|---|---|
| `VERTAAUX_API_KEY` | Yes | — | API authentication key |
| `VERTAAUX_API_BASE` | No | `https://vertaaux.ai/api/v1` | API endpoint URL |
| `PORT` | No | `8787` | HTTP transport port |
| `GITHUB_TOKEN` | No | — | GitHub API access for `generate_pr` |

## Tools

### Audit Tools (Core)

| Tool | Description |
|---|---|
| **`audit_url`** | Run UX & accessibility audit on a deployed URL. Returns top 5 issues with severity breakdown. |
| **`audit_repo`** | Static analysis on local codebase (React/Vue/Svelte/HTML). Finds missing alt text, unlabeled buttons/inputs/links. |
| **`audit_artifact`** | Audit from HAR files (response times, failed requests, large payloads) or Lighthouse JSON (accessibility findings). |
| **`get_findings`** | Retrieve findings from a completed audit with filtering by severity, rule, and pagination. |
| **`get_audit`** | Get audit job status and results by job ID. |

### Fix & Verify Tools

| Tool | Description |
|---|---|
| **`explain_finding`** | Deep-dive into a finding: WCAG criteria, repro steps, fix guidance, before/after code examples. |
| **`suggest_fix`** | Generate search/replace patch with confidence score. Supports single and batch mode. |
| **`generate_patch`** | Generate accessibility fix patch for a specific issue from an audit. |
| **`run_verification_suite`** | Verify a patch fixes the issue without regressions via before/after audit. |
| **`generate_pr`** | Create a draft GitHub PR with fix patches. Requires `GITHUB_TOKEN`. |
| **`create_pr_comment`** | Generate a PR comment with suggestion blocks, ordered by severity. |

### Analysis Tools

| Tool | Description |
|---|---|
| **`analyze_component`** | Heuristic UX review of component code (no browser needed). Checks images, buttons, inputs, links. |
| **`run_llm_audit`** | Provider-agnostic LLM audit (Mistral/OpenAI via Vertaa adapter). |
| **`capture_screenshot`** | Capture screenshot by running a quick audit. |
| **`compare_competitors`** | Compare UX metrics against competitor URLs with category-level score deltas. |
| **`explain_issue`** | Format an issue into developer-friendly markdown guidance. |

### Management Tools

| Tool | Description |
|---|---|
| **`create_webhook`** | Register webhook for audit notifications. |
| **`list_webhooks`** / **`delete_webhook`** | Manage webhooks. |
| **`create_schedule`** | Cron-based scheduled audits with score threshold alerts. |
| **`get_schedule`** / **`list_schedules`** / **`update_schedule`** / **`delete_schedule`** | Manage schedules. |
| **`get_quota`** | Check plan and remaining credits. |
| **`get_engines`** | List available engine versions. |

### Accessibility Tools (Multi-Engine)

| Tool | Description |
|---|---|
| **`audit_a11y`** | Multi-engine accessibility audit using axe-core, AccessLint, and custom analyzers. Returns WCAG-mapped findings with structured fix suggestions and fixability ratings. Supports `min_impact` filtering and `mode` (basic/standard/deep). |
| **`diff_a11y`** | Compare current accessibility findings against a saved baseline. Returns fixed, new, and unchanged findings with net change summary. Requires a prior `audit_a11y` call to establish the baseline. |

### Deprecated

| Tool | Description |
|---|---|
| **`run_audit`** | **DEPRECATED** — Use `audit_url` instead. |

## Prompt Templates

Pre-built workflow prompts for common audit scenarios:

| Prompt | Description | Arguments |
|---|---|---|
| **`quick_audit`** | Audit a URL and summarize top issues with fix recommendations | `url` |
| **`fix_accessibility`** | Full audit → patch → PR comment workflow | `url` |
| **`compare_ux`** | Compare against competitors and identify UX gaps | `url`, `competitors`, `industry?` |
| **`monitor_regression`** | Set up scheduled monitoring with alerts | `url`, `frequency?` |
| **`audit_codebase`** | Static analysis on local codebase | `path` |

## Resources

The server exposes MCP resources via `vertaa://` URIs:

| URI Pattern | Description |
|---|---|
| `vertaa://audits/{auditId}` | Full audit result |
| `vertaa://audits/{auditId}/summary` | Lightweight summary |
| `vertaa://audits/{auditId}/findings/{findingId}` | Single finding detail |
| `vertaa://screenshots/{auditId}` | Screenshot metadata |
| `vertaa://screenshots/{auditId}/annotated` | Annotated screenshot |
| `vertaa://history/{encodedUrl}` | Audit history for URL |
| `vertaa://history/{encodedUrl}/trend` | Score trend analysis |
| `vertaa://guidelines/{topic}` | UX guidelines (buttons, forms, navigation, color-contrast, errors, content) |

## Enterprise Controls

Configure domain allowlists, rate limits, and PII redaction programmatically:

```typescript
import { configureEnterpriseControls } from './server.js';

configureEnterpriseControls({
  allowlist: {
    allowed_domains: ['*.example.com'],
    denied_domains: ['internal.example.com'],
  },
  budget: {
    max_requests: 100,
    max_pages: 50,
    max_duration_ms: 60000,
    max_concurrency: 3,
  },
  redaction: {
    redact_emails: true,
    redact_phone_numbers: true,
    redact_credit_cards: true,
    custom_patterns: [
      { name: 'api_key', pattern: 'sk_[a-zA-Z0-9]{20,}', replacement: '[REDACTED]' }
    ],
  },
});
```

## Example: Audit-to-PR Workflow

```
1. audit_url({ url: "https://example.com", mode: "deep" })
   → Returns audit_id with top 5 issues

2. get_findings({ audit_id: "...", severity: "critical" })
   → Returns all critical findings with deterministic IDs

3. suggest_fix({ audit_id: "...", finding_id: "button-name:a1b2c3d4" })
   → Returns search/replace patch with 85% confidence

4. run_verification_suite({ url: "...", selector: "button.submit", rule_id: "button-name" })
   → Verifies fix resolves the issue

5. create_pr_comment({ file_path: "src/Button.tsx", patches: [...] })
   → Generates PR comment with suggestion blocks
```

## Development

### Project Structure

```
mcp-server/
├── src/
│   ├── index.ts              # Main entry, tool registration
│   ├── server.ts             # MCP server config, resources, middleware
│   ├── a11y-tools.ts         # Multi-engine a11y audit & baseline diffing tools
│   ├── http.ts               # HTTP transport entry point
│   ├── prompts.ts            # MCP prompt templates
│   ├── analysis.ts           # Component analysis engine
│   ├── patch.ts              # Patch generation
│   ├── verification.ts       # Patch verification
│   ├── pr-comment.ts         # PR comment generation
│   ├── tools/
│   │   ├── audit-url.ts      # audit_url tool
│   │   ├── audit-repo.ts     # audit_repo tool (static analysis)
│   │   ├── audit-artifact.ts # audit_artifact tool (HAR/Lighthouse)
│   │   ├── get-findings.ts   # get_findings tool
│   │   ├── explain-finding.ts# explain_finding tool
│   │   ├── suggest-fix.ts    # suggest_fix tool
│   │   ├── generate-pr.ts    # generate_pr tool
│   │   └── index.ts          # Tool exports
│   ├── transports/
│   │   ├── stdio.ts          # Stdio transport (default)
│   │   └── http.ts           # HTTP streaming transport
│   ├── middleware/
│   │   ├── allowlist.ts      # Domain/path allowlist
│   │   ├── budget.ts         # Rate limiting & quotas
│   │   ├── redaction.ts      # PII redaction
│   │   └── index.ts          # Middleware stack
│   ├── resources/
│   │   ├── audit-results.ts  # vertaa://audits/* resources
│   │   ├── screenshots.ts    # vertaa://screenshots/* resources
│   │   ├── historical.ts     # vertaa://history/* resources
│   │   ├── legacy.ts         # Guidelines resources
│   │   └── index.ts          # Resource exports
│   ├── schemas/
│   │   ├── audit.ts          # Audit schemas (mode, findings)
│   │   ├── findings.ts       # Finding schemas
│   │   ├── controls.ts       # Enterprise control schemas
│   │   ├── errors.ts         # Error schemas
│   │   └── index.ts
│   ├── utils/
│   │   ├── error-recovery.ts # Structured errors with recovery guidance
│   │   ├── change-tracker.ts # Baseline change tracking
│   │   └── deterministic-id.ts # Stable finding IDs
│   └── index.test.ts         # Test suite
├── README.md
├── package.json
├── tsconfig.json
└── vitest.config.ts
```

### Testing

```bash
npm test              # Run test suite
npm run test:watch    # Watch mode
npm run test:coverage # Coverage report
```

### Building

```bash
npm run build         # TypeScript → dist/
```

## Error Handling

All errors include structured recovery guidance:

```json
{
  "code": "AUDIT_NOT_FOUND",
  "message": "Audit abc123 not found.",
  "recovery": {
    "action": "Start a new audit for this URL",
    "tool": "audit_url",
    "params": { "url": "https://example.com" }
  }
}
```

Error codes follow JSON-RPC 2.0: `-32700` (parse), `-32600` (invalid request), `-32601` (method not found), `-32602` (invalid params), `-32603` (internal error).

## API Reference

The MCP server communicates with the VertaaUX API v1. See the [API Documentation](https://vertaaux.ai/developer-docs).

## License

MIT
