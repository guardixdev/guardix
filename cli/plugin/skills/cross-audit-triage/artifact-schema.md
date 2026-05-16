# Triage Artifact Schema

`cross-audit-triage.json` is the structured output of this skill. It is
**self-contained** — a renderer (a hosted viewer, another tool) needs nothing
but this file to display the full comparison.

`schema_version` is pinned. Bump it on any breaking change; consumers must
branch on it. Current version: **1**.

## Top-level shape

```json
{
  "schema_version": 1,
  "generated_at": "2026-05-15T12:00:00Z",
  "repository": "acme/vault",
  "reference_commit": "a2135c4",
  "sources": [ ... ],
  "findings": [ ... ],
  "issues": [ ... ],
  "summary": { ... }
}
```

`repository` and `reference_commit` are optional (a triage may span only
external reports with no Guardix audit). `reference_commit` is the commit the
comparison is anchored to — default to the Guardix audit's commit when present.

## `sources[]` — one per provider

```json
{
  "provider_label": "trail-of-bits",
  "commit": "9f1e0a2",
  "report_path": "audits/tob-2026-03.pdf",
  "finding_count": 14
}
```

`commit` is the commit that provider audited (may differ per source — surface
that to the user). `report_path` is the local path the findings were read from;
for a Guardix source use `"guardix-cli"`.

## `findings[]` — every normalized finding from every source

The common shape from SKILL.md Step 2, plus two added fields:

```json
{
  "finding_uid": "trail-of-bits:TOB-7",
  "issue_id": "X-3",
  "source_label": "trail-of-bits",
  "original_id": "TOB-7",
  "title": "Withdrawal queue has no user-controlled exit",
  "severity_raw": "Medium",
  "severity_normalized": "medium",
  "contract": "WithdrawalQueue",
  "function": "requestWithdrawal",
  "file": "src/WithdrawalQueue.sol",
  "line": 412,
  "root_cause": "...",
  "recommended_fix": "...",
  "description": "..."
}
```

`finding_uid` is unique within the artifact (`<provider_label>:<original_id>`).
`issue_id` links the finding to its correlated issue in `issues[]`.

## `issues[]` — the correlated issues (table rows)

```json
{
  "issue_id": "X-3",
  "title": "Withdrawal queue has no user-controlled exit",
  "severity_normalized": "high",
  "verdict": "likely-true-positive",
  "verification": "verified-in-source",
  "checked_at": "src/WithdrawalQueue.sol:412",
  "rationale": "Source confirms no caller-initiated path empties the queue; ...",
  "member_finding_uids": ["guardix:VAU-2", "trail-of-bits:TOB-7"],
  "providers": ["guardix", "trail-of-bits"]
}
```

`verdict` is one of `likely-true-positive`, `likely-false-positive`, `disputed`,
`needs-manual`. `verification` is one of `verified-in-source` (the verdict was
checked against the implementation) or `unverifiable-from-source` (the verdict
depends on deployment config / off-chain state not in the code — always paired
with a `needs-manual` verdict). Every issue carries one of the two — there is no
unassessed issue. `checked_at` cites the `file:line` the validation rested on.
`severity_normalized` is the issue-level severity (typically the max across
members). `providers` is the deduplicated list of provider labels that reported
the issue — its length drives the consensus/sole-reporter counts.

## `summary`

```json
{
  "total_issues": 12,
  "unanimous": 3,
  "sole_reporter": 5,
  "by_verdict": { "likely-true-positive": 7, "likely-false-positive": 2, "disputed": 2, "needs-manual": 1 },
  "by_verification": { "verified-in-source": 11, "unverifiable-from-source": 1 },
  "by_severity": { "critical": 1, "high": 4, "medium": 4, "low": 2, "info": 1 }
}
```

`unanimous` = issues reported by every source. `sole_reporter` = issues a single
source raised. Both are derived from `issues[].providers` against
`sources[].provider_label` — store them so a renderer needn't recompute.
`by_verification` is the verification coverage — the count that was checked
against the source versus the count that could not be.
