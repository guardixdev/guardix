# Correlation Rubric

The judgment reference for `cross-audit-triage`. Read this before correlating
(Step 3) and before adjudicating (Step 4). The methodology here is the same one
Guardix's own audit pipeline uses to deduplicate findings across its internal
sources — applied across *providers* instead.

## 1. Severity normalization

Providers use different scales and different labels. Do **not** build a
label-to-label lookup table — it rots the moment a firm rebrands its scale.

**Principle:** map each provider's severity onto Guardix's five levels —
`critical`, `high`, `medium`, `low`, `info` — by **impact and likelihood**, not
by the label's wording. A firm's "Major" is not necessarily your "High"; a
tool's "Warning" is usually `low` or `info`.

When a report includes its own severity legend (most firm PDFs do), read it and
re-derive the mapping from that legend's impact/likelihood definitions.

Starter reference (override when a report's own legend says otherwise):

| Provider wording                              | Normalized |
|-----------------------------------------------|------------|
| Critical / Severe                             | critical   |
| High / Major                                  | high       |
| Medium / Moderate                             | medium     |
| Low / Minor                                   | low        |
| Informational / Note / Gas / Best-practice    | info       |

## 2. Correlation — the "one fix" test

Two findings describe the **same correlated issue** when:

> **Would one code change fix all of them?** If yes, they are one issue. If
> different code changes are required, they are distinct.

This decides every merge. Title wording, which provider reported it, and
severity disagreement are all irrelevant to the merge decision — only the fix.

**DO NOT over-split.** A single missing-control vulnerability commonly manifests
through several distinct triggers; every trigger is not a separate issue.
Examples that should collapse into ONE correlated issue:

- *"No user-cancel path for the withdrawal queue"* — whether the trigger is
  whitelist expiry, whitelist revocation, blacklisting the user, blacklisting
  the custodian, a dependency pause, custodian inaction, or shutdown mode. All
  resolved by adding one user-initiated cancel function. **One issue, not seven.**
- *"Share token has no KYC gate"* — whether one provider frames it as "non-KYC
  users earn yield" and another as "non-KYC users get stuck with irredeemable
  positions". **One issue**, resolved by overriding `_update` in the share
  contract.
- *"Shutdown-mode queue accounting is unit-inconsistent"* — even if one provider
  observed it via the burn calculation and another via the claim formula.
  **One issue.**

**DO split** when the proposed fixes are genuinely different code changes, even
if the findings share a component or vocabulary: a constructor zero-address
check versus a runtime-state bug in the same contract; a "view function lies
about state" bug (fix: rename/override the view) versus a state-transition bug
(fix: add a guard) in the same contract — these stay **separate** issues.

Be deliberate, not timid: when one umbrella fix would clearly resolve all
members, merge them even if their immediate triggers differ.

## 3. Recall-first, then split (named invariant)

Run correlation in two ordered passes:

1. **Pre-group recall-first** — group by affected component plus shared
   mechanism vocabulary (the subject / effect / cause of the bug). Be
   deliberately permissive; err toward grouping.
2. **Split** each group with the one-fix test from §2.

**Why this order:** a false merge can be corrected by splitting; a missed
correlation cannot be recovered — you never see the pair again. So group first,
split second. False merges hide bugs (bad), but false splits are exactly what
the audit consumer is complaining about — they see seven copies of one issue and
lose trust.

## 4. Adjudication — true vs false positive

Assign each correlated issue one verdict: `likely-true-positive`,
`likely-false-positive`, `disputed`, `needs-manual`. **Every issue is checked
against the implementation — one by one, no exceptions, no bundling.** A verdict
you did not ground in code you actually read is not a verdict; that issue is
`needs-manual` / `unverifiable-from-source`, and you say so plainly rather than
guessing.

**Consensus is a signal, not a verdict.** Multiple providers agreeing *raises*
confidence; it does not settle it. A finding three providers missed can still be
real. A finding three providers agree on can still be wrong. Always read the
actual contract source before deciding.

**False-positive smells** — these pull a verdict toward `likely-false-positive`
regardless of how many providers reported the issue:

- The finding flags a **well-known safe pattern** as a vulnerability — e.g.
  OpenZeppelin/Solmate ERC4626 virtual-offset / virtual-shares as if it were
  exploitable, a `nonReentrant` modifier that is already present, `SafeERC20`
  already in use, dead-shares-to-`address(0)`.
- The finding **contradicts itself** — its own text says the code is correct,
  "works as intended", "follows the standard pattern", "already mitigated by",
  "cannot be exploited", yet it is still filed as a finding.
- The "vulnerability" requires a precondition the code makes impossible, or the
  attack the finding describes reverts in the actual source.

**True-positive signals:** a concrete exploit path that the source code permits;
a state the contract can actually reach; an invariant the code genuinely fails
to enforce. Multiple independent providers describing the same concrete path is
strong corroboration.

Use `disputed` when providers genuinely conflict and the source does not clearly
settle it. Use `needs-manual` when adjudication needs information you do not have
(off-chain assumptions, deployment config, governance intent).

Every verdict is **Guardix's assessment**, with a rationale — never stated as
objective fact. Labeling a paid auditor's finding a false positive carries
weight; show your reasoning.
