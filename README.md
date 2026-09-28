# Correctover Security Scan — GitHub Action

An **independent, fail-closed second fuse** for AI agents and MCP runtimes. It scans your shipped JavaScript, npm packages and MCP configuration for **RCE, SSRF, cloud-credential hijack, tool poisoning and prompt injection** (aligned to OWASP AISVS), uploads SARIF to code scanning, and blocks the merge when a critical finding is present.

> Correctover is a verification layer that is **independent of the system being verified**. A control plane claiming "logs are available" is not the same as a runtime gate that fails closed. This action leaves evidence you and a third party can independently check.

## Quick start

Add to `.github/workflows/correctover.yml`:

```yaml
name: correctover-security
on:
  push:
    branches: [main]
  pull_request:

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      security-events: write   # required to upload SARIF
      contents: read
    steps:
      - uses: actions/checkout@v4
      - name: Correctover scan
        uses: DSHCorrectover/correctover-action@v1
        with:
          target: '.'
          mode: bundle
          fail-on: error
```

Every PR then gets:

- Findings annotated at **file + line** in the GitHub code-scanning UI.
- A **job summary** with the error/warning tally.
- A **failed check** when the configured threshold is breached.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `target` | `.` | File or directory to scan. |
| `mode` | `bundle` | `bundle` (shipped JS / npm package), `config` (an MCP config file), or `auto`. |
| `sarif-file` | `correctover-results.sarif` | Where the SARIF report is written. |
| `scanner-version` | `3.2.1` | `correctover-scan` version to run (dist-tag or exact version). |
| `fail-on` | `error` | `error`, `warning`, or `none`. |
| `upload-sarif` | `true` | Upload SARIF to GitHub code scanning. |

## What it detects

- `command-exec` — `child_process` `shell:true` / dynamic `exec` / `execSync` (command injection).
- `ssrf-protection` — cloud metadata (`169.254.169.254`, GCP metadata) and RFC1918 addresses in business code.
- `dynamic-eval` — `eval`, `new Function`, `node:vm` outside known-benign contexts.
- hardcoded secrets (`AKIA…`, `ghp_…`, Slack tokens), plaintext endpoints, missing MCP auth, timeout/kill-switch, permission gates and input validation.

Real cloud-SDK credential providers (`@aws-sdk`, `gcp-metadata`, provider chains), SSRF guard/blocklist code, and static build scripts are recognised and **not** reported.

## Scope and limits (please read)

This is **automated static signal scanning over a snapshot**. It does **not** prove the absence of reachable vulnerabilities, determine runtime exploitability, and is **not** a security audit, penetration test, certification, or statement of compliance. A clean report is not a guarantee of security.

For vendor acceptance or high-assurance use cases, Correctover also provides a **signed compliance report** and a **116-check manual deep audit** (5-day turnaround; if no critical-severity issue is found, you pay nothing). See [the official entry](https://dshcorrectover.github.io/agent-audit/).

## License

Elastic License 2.0. The standard is open; the client is source-available; the engine is closed source.
