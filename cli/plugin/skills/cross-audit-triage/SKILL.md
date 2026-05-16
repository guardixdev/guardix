---
name: cross-audit-triage
description: Reconcile smart-contract audit findings from multiple providers — load every source, correlate findings that describe the same issue, adjudicate true vs false positive, and build one comparison table. Use when the user has audit results from more than one source (Guardix + an external firm, two external firms, a firm + a tool like Slither/Mythril) and wants them compared, deduplicated, or triaged.
---

# Cross-Audit Triage

Teams commission audits from several providers. Each returns a different set of
findings — overlapping, contradictory, some real, some false positives. The job
of this skill is to line them all up and answer: *which findings are the same
issue, and which are actually real?*

Everything here runs **locally**. You read the reports, you correlate, you
adjudicate. There is no server-side triage step.

## When to use this

- The user has audit findings from **more than one source** and wants them
  compared / reconciled / deduplicated / triaged.
- The user asks "do these auditors agree", "which of these findings are real",
  "compare these audit reports", "triage these PDFs".
- Distinct from the `guardix` skill (reading one Guardix audit's results).

## Step 1 — Gather every source

Collect each provider's findings into raw form. Sources are format-agnostic:

- **Guardix audit** — `guardix finding list --json` for the finding set, then
  `guardix finding get <CODE> --json` per finding for full detail (`root_cause`,
  `recommended_fix`, `locations`). The `guardix` skill documents the `--json`
  contract and error envelope; reuse it.
- **External firm reports** — usually PDFs. Read them with the Read tool. Also
  markdown or HTML reports.
- **Tool output** — Slither / Mythril / Aderyn JSON or SARIF. Read with the Read
  tool.

**The contract source must be on disk** — this skill validates every finding
against the real code (Step 4), so it runs inside a checkout of the audited
repository. If the source is not present, get it first: clone the repo and
check out the audited commit. You cannot judge a finding true or false from
report prose alone.

Record, for each source: a short `provider_label` (e.g. `guardix`, `trail-of-bits`,
`slither`), the commit it audited if stated, and the report path.

## Step 2 — Normalize to the common shape

Convert every finding from every source into one canonical record. Field names
mirror Guardix's own `Finding` model so the mental model stays consistent:

```
{
  "source_label":        "trail-of-bits",
  "original_id":         "TOB-7",                  // the provider's own id/code
  "title":               "...",
  "severity_raw":        "Medium",                 // exactly as the provider wrote it
  "severity_normalized": "medium",                 // critical|high|medium|low|info
  "contract":            "WithdrawalQueue",
  "function":            "requestWithdrawal",
  "file":                "src/WithdrawalQueue.sol",
  "line":                412,
  "root_cause":          "...",
  "recommended_fix":     "...",
  "description":         "..."
}
```

Severity normalization is judgment, not a label-to-label map — **read
`correlation-rubric.md` before you do it.**

## Step 3 — Correlate (read the rubric first)

**Read `correlation-rubric.md` now.** It carries the matching methodology — the
"would one code change fix all members?" test, the recall-first-then-split rule,
and worked examples. Apply it:

1. Pre-group recall-first by affected component + mechanism vocabulary. Be
   permissive — over-grouping is recoverable, a missed correlation is not.
2. Split each group with the one-fix test.

Each resulting cluster is one **correlated issue**. Give it a stable id (`X-1`,
`X-2`, …). A cluster may have one member (an issue only one provider raised).

## Step 4 — Validate every finding against the source

This step is **exhaustive and non-negotiable**: every issue is validated, one by
one, against the actual contract source — including every sole-reporter
single-finding issue. Never bundle findings into a catch-all row to skip
validation; never sample; never leave an issue unassessed. If there are 90
findings, you check 90.

For each issue:

1. Open the implementation of every contract/function its member findings cite.
2. Trace the claim against the real code — can the described state be reached?
   Does the cited line do what the report says? Does a guard elsewhere already
   prevent it?
3. Assign a **verdict**: `likely-true-positive`, `likely-false-positive`,
   `disputed`, or `needs-manual`.
4. Assign a **verification** status:
   - `verified-in-source` — you read the implementation and the verdict is
     grounded in specific code you cite.
   - `unverifiable-from-source` — the verdict genuinely depends on information
     not in the code (deployment config, off-chain assumptions, external-protocol
     state). Pair this with a `needs-manual` verdict and state what is missing.
   Every issue ends as one or the other — there is no "skipped".
5. Write a one-line rationale citing the specific `file:line` you checked.

The rubric's TP/FP section governs the verdict — consensus across providers
raises confidence but never decides it. Frame every verdict as *Guardix's
assessment*, not objective truth.

## Step 5 — Emit the outputs

Produce both:

1. **A markdown comparison matrix** — one row per correlated issue; columns:
   issue id, title, each provider (that provider's `severity_raw` + `original_id`,
   or blank if it did not report the issue), normalized severity, verdict,
   verification (`verified-in-source` / `unverifiable-from-source`), rationale.
   A summary header: total issues, how many every provider agreed on, how many a
   single provider raised, how many adjudicated false-positive, and the
   verification coverage (e.g. "88 of 90 verified in source, 2 unverifiable").
2. **A structured artifact** — write `cross-audit-triage.json` per
   `artifact-schema.md` (read it for the exact shape). Self-contained: source
   list, every normalized finding, the correlated issues with members and
   verdicts, the summary counts.

Then offer two follow-ups (do not do them unprompted):

- Sync verdicts back to Guardix findings with `guardix finding review`.
- Upload the artifact for a hosted view (if `guardix triage upload` exists).

## Honesty constraints

- Never invent a correlation to make the table tidy. If a match is genuinely
  uncertain, give the issue a `disputed` verdict and say so in the rationale.
- A finding only one provider raised is **not** automatically a false positive —
  one auditor catching what others missed is common.
- A finding every provider agreed on can still be a false positive — consensus
  on a known-safe pattern is still wrong. The rubric lists the smells.
