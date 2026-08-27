---
name: guardix
description: Use the guardix CLI to read smart-contract audit results and create or manage Release Audits that reconcile external audit reports with source and deployments. Use when working in a repository connected to Guardix, when the user asks about findings, vulnerabilities, audit status, a finding code like VAU-3, third-party auditor coverage, release readiness, or any Guardix workflow.
---

# Guardix CLI

`guardix` is the command-line client for the Guardix smart-contract audit
platform. When you are working in a git repository connected to Guardix, use
it to pull real audit results instead of guessing about security state.

## When to use this

- The user asks "are there findings", "what did the audit say", "is this
  contract reviewed", "what vulnerabilities were found".
- The user wants to review third-party audit reports against a release,
  source snapshot, private source archive, or deployed contract.
- You are about to change a smart contract and should check known findings first.
- The user mentions Guardix, an audit, or a finding code like `VAU-3`.

## Prerequisites

```bash
guardix --version        # not installed? https://github.com/guardixdev/guardix
                         # release-audit commands need >= 0.3.4; run: guardix upgrade
guardix auth status      # exits 1 when not signed in
```

Not signed in? You can drive the browser-approval login yourself
(requires guardix >= 0.3.4):

```bash
guardix auth login --no-browser --json
```

The first JSON document on stdout is
`{"status": "awaiting_approval", "verification_url": ..., "user_code": ...}`.
Relay the URL to the user, tell them the pairing code to expect, and keep the
command running — it polls until they approve in the browser, then prints
`{"status": "authenticated"}` and saves the key locally. Do not paste API
keys into chat; in CI use `--api-key` or the `GUARDIX_API_KEY` env var
instead.

## Core commands

Repo-scoped commands infer the repository from the current directory's git
origin — run them inside a connected checkout and omit `--repo`.

```bash
guardix status                       # latest audit + open-finding summary for this repo
guardix finding list                 # findings of the latest audit
guardix finding get VAU-3            # one finding in detail
guardix audit list                   # audits for this repo
guardix audit get --audit '#5'       # one audit's status
```

Add `--json` to any command for structured output you can parse.

## Release Audits

Treat a Release Audit as a separate product from a full Guardix code audit. It
meta-reviews external auditor reports and reconciles them with supplied source
and optional deployment addresses.

Inside a connected checkout, start with:

```bash
guardix release-audit start \
  --report audit.pdf \
  --non-interactive --json --wait
```

For standalone or multi-source releases, repeat `--report`, `--repo-url`, and
`--source-zip`; add deployed contracts as
`--deployment chain:0xaddress[:label]`. Source is required by default.
Use `--report-only` only when the user explicitly accepts weaker
reconciliation.

Use the resumable workflow when uploads or orchestration need separate steps:

```bash
guardix release-audit create --project "Protocol v2" --json
guardix release-audit report add <id> audit.pdf
guardix release-audit source add-git <id> https://github.com/org/repo --ref v2
guardix release-audit source add-zip <id> private-source.zip
guardix release-audit run <id> --json --wait
```

When wait exits `3`, this is not an assessment failure. Inspect
`next_action`; payment or intake input is required. Resolve intake with:

```bash
guardix release-audit discovery show <id> --json
guardix release-audit discovery show <id> <question-id> --json
guardix release-audit discovery answer <id> <question-id> --answer @answer.json --json
guardix release-audit discovery recheck <id> --json
guardix release-audit discovery proceed <id> --json
```

Use `discovery show` to obtain the exact kind-shaped answer example. Do not
guess discovery answer fields. Waivers and `--acknowledge-gaps` are recorded on
the final assessment; require the user's reason and explicit approval.

## Agent contract

- `--json` — structured stdout; on failure, stdout carries
  `{"error":{"code","message","hint","status"}}`.
- `--quiet` — suppress informational stderr (hints, inferred values).
- `--non-interactive` — never prompt; fail fast when input is missing.
- Exit codes: `0` success, `1` failure, `2` wait timed out, `3` Release Audit
  payment or intake action required.

When repo inference fails in `--json` mode the error envelope carries a stable
`.error.code` — branch on it instead of parsing prose:

- `repo_required` — no `--repo` and no GitHub origin in the current directory.
- `repo_not_connected` — origin parsed, but the repo isn't connected to Guardix.
- `repo_ambiguous` — the slug is connected in multiple teams.

## Full command surface

```bash
guardix manifest --quiet
```

Returns every command, flag, env var, and exit code as one versioned JSON
document. Read it when you need a command or flag not listed above — do not
guess flag names.
