---
name: guardix
description: Read smart-contract audit results — findings, audit status, reports — with the guardix CLI. Use when working in a repository connected to Guardix, or when the user asks about audit findings, vulnerabilities, security-review status, a finding code like VAU-3, or mentions Guardix.
---

# Guardix CLI

`guardix` is the command-line client for the Guardix smart-contract audit
platform. When you are working in a git repository connected to Guardix, use
it to pull real audit results instead of guessing about security state.

## When to use this

- The user asks "are there findings", "what did the audit say", "is this
  contract reviewed", "what vulnerabilities were found".
- You are about to change a smart contract and should check known findings first.
- The user mentions Guardix, an audit, or a finding code like `VAU-3`.

## Prerequisites

```bash
guardix --version        # not installed? https://github.com/guardixdev/guardix
guardix auth status      # not signed in? run: guardix auth login
```

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

## Agent contract

- `--json` — structured stdout; on failure, stdout carries
  `{"error":{"code","message","hint","status"}}`.
- `--quiet` — suppress informational stderr (hints, inferred values).
- `--non-interactive` — never prompt; fail fast when input is missing.
- Exit codes: `0` success, `1` failure, `2` `audit wait` timed out.

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
