# MCP Shield Action

GitHub Action that scans MCP servers for security issues, compliance gaps, and risk factors.

Uses [mcp-shield-cli](https://pypi.org/project/mcp-shield-cli/) — automated checks across compliance, security, and advisory suites.

## Quick start

```yaml
name: MCP Shield
on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - uses: thuggeelya/mcp-shield-action@v1
        with:
          server: 'node dist/index.js'
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `server` | yes | — | Server command (e.g., `node dist/index.js`, `npx -y @foo/bar`) |
| `fail-on` | no | `high` | Minimum severity to fail CI: `critical`, `high`, `medium`, `low`, `any` |
| `comment` | no | `true` | Post scan results as PR comment |
| `badge` | no | `true` | Generate badge URL in outputs |
| `ml` | no | `false` | Enable ML-based prompt injection detection |
| `version` | no | `latest` | mcp-shield-cli version to install |

## Outputs

| Output | Description |
|--------|-------------|
| `score` | Numeric score (0-100) |
| `grade` | Letter grade (A+ to F) |
| `badge-url` | shields.io badge URL |
| `report-json` | Path to JSON report file |

## Badge

Add a badge to your README after a successful scan:

```markdown
[![MCP Shield](https://img.shields.io/badge/MCP_Shield-A%2B%20(95)-brightgreen)](https://github.com/thuggeelya/mcp-shield)
```

The Action outputs a `badge-url` you can use directly:

```yaml
- uses: thuggeelya/mcp-shield-action@v1
  id: shield
  with:
    server: 'node dist/index.js'
- run: echo "Badge: ${{ steps.shield.outputs.badge-url }}"
```

## PR comment

When `comment: true` (default) and triggered by a pull request, the Action posts a summary:

> ## MCP Shield Report
> **Score: 57/100 (Grade: D)**
> Checks: 22 | Passed: 13 | Failed: 1 | Warnings: 7
>
> ### Findings
> - :x: **SEC-003** FAIL — Security score: 30/100
> - :warning: **SEC-002** WARN — 11 injection vectors

## Examples

### Strict mode (fail on any warning)

```yaml
- uses: thuggeelya/mcp-shield-action@v1
  with:
    server: 'node dist/index.js'
    fail-on: medium
```

### With ML detection

```yaml
- uses: thuggeelya/mcp-shield-action@v1
  with:
    server: 'node dist/index.js'
    ml: true
```

### npx-based server

```yaml
- uses: thuggeelya/mcp-shield-action@v1
  with:
    server: 'npx -y @modelcontextprotocol/server-memory'
```

### Use outputs in subsequent steps

```yaml
- uses: thuggeelya/mcp-shield-action@v1
  id: shield
  with:
    server: 'node dist/index.js'
    fail-on: critical
- run: |
    echo "Score: ${{ steps.shield.outputs.score }}"
    echo "Grade: ${{ steps.shield.outputs.grade }}"
    if [ "${{ steps.shield.outputs.score }}" -lt 50 ]; then
      echo "::warning::Low security score"
    fi
```

## License

Apache 2.0
