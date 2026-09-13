---
name: running-audits
description: Start and monitor Guardix smart-contract audits with the guardix CLI. Use when the user wants to run a new audit / security scan on a contract, kick off a Guardix audit, watch a running audit to completion, or check why an audit failed.
---

# Running Audits

This skill covers *starting and watching* Guardix audits. For reading the
findings of an audit that already finished, use the `guardix` skill instead.

## When to use this

- The user wants to **audit a contract** / run a security scan / "have Guardix
  look at this".
- The user wants to **wait for** a running audit, or check its progress.
- An audit failed and the user wants to know why or retry it.

## Prerequisites

```bash
guardix --version       # not installed? https://github.com/guardixdev/guardix
guardix auth status     # not signed in? run: guardix auth login
```

The repo must be connected to Guardix. Inside a connected checkout, `--repo`
is inferred from the cwd's git origin. `--ref` defaults to the current branch
when that branch exists on the remote. Interactive `guardix audit start` asks
before auditing a non-default branch, or when local commits or uncommitted files
are not on GitHub (the scan uses the GitHub commit, not your working tree).
Pass `--ref` to skip the prompt. A pushed default branch needs no flags:

```bash
guardix audit start                                   # infer repo + ref from cwd
guardix audit start --repo acme/vault --ref main      # explicit
guardix audit start --ref main --contract src/Vault.sol   # scope to one contract
guardix audit start --ref main --wait                 # start, then block to completion
```

Useful flags: `--contract <path>` (repeatable, scope the audit), `--doc-url` /
`--doc-text title::content` / `--doc-file` (give the auditor threat models and
design docs), `--include-repo-docs`, `--wait`, `--open`.

If the start needs payment, interactive `guardix audit start` opens Stripe
checkout and **waits until the user pays** (status leaves `pending_payment`).
The wait is silent: no `pending_payment` progress ticks. It then prints
`Payment received` and a follow-up `audit wait` command. Passing `--wait` on
start watches until the audit finishes. `--json` / `--non-interactive` only
print `checkout_url` unless `--wait` is also on start. The CLI never handles
card data. For invoices after the first payment, use `guardix billing portal`.

For non-interactive use (CI, agents) pass `--non-interactive`. Inside a
connected checkout you can omit `--repo` / `--ref`; if the current branch is
not on the remote the command errors instead of prompting.

## Watch a running audit

```bash
guardix audit get    --audit '#5'    # current status + progress checkpoint
guardix audit wait   --audit '#5'    # block until the audit reaches a terminal state
guardix audit logs   --audit '#5'    # live stream of agent logs
```

`audit wait` is built for CI: exit `0` on `completed`, a non-zero error on
`failed`, and **exit code `2`** specifically on `--timeout`. Branch on exit code
`2` to distinguish "still running" from "broke". Tune with `--interval` and
`--timeout`.

`#N` and short-commit audit selectors need `--repo` (or a connected checkout) to
disambiguate; full UUIDs work anywhere.

## When an audit failed

```bash
guardix audit get --audit '#5' --json    # failure_reason field explains why
guardix audit retry --audit '#5'         # re-run (consumes retry quota)
```

## Agent contract

Add `--json` for structured stdout; on failure stdout carries
`{"error":{"code","message","hint","status"}}`. `--quiet` silences
informational stderr. Exit codes: `0` success, `1` failure, `2` `audit wait`
timed out. Run `guardix manifest --quiet` for the full flag surface — do not
guess flag names.
