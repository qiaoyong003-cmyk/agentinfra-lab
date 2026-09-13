# #07 Cross-Model Blind Audit

> A successful run is not a verified result. An independent model review is not a vote.

## The Problem

A complex multi-stage computation completed, yet some of its outputs did not reconcile with the underlying records. One documented check showed **4,567 input records and 4,556 output records**. The disposition of the remaining **11** had not been adequately explained. That does **not**, by itself, prove that 11 records were lost; it establishes a reconciliation gap requiring investigation.

In this kind of workflow, a final “success” status says little about whether a parameter changed between stages, a field was populated, an exclusion propagated downstream, or a claim remained supported by the data. Manual inspection alone is difficult to scale across the steps and their dependencies.

The response was a controlled **cross-model blind audit**: use different models to expose possible blind spots, then require original evidence and human review before accepting an observation.

<img src="../assets/07-cross-model-audit/01-cover.png" width="520" alt="4,567 input records versus 4,556 output records; 11 require an explanation">

[Optional animated cover preview](../assets/07-cross-model-audit/01-cover-animation.gif)

## Why the Boundary Was Hard to See

The analysis passed through multiple inputs, execution steps, intermediate outputs, and final summaries. A plausible summary could coexist with an inconsistent source version or a missing downstream field.

The audit therefore compared **claims with execution evidence**, not merely one summary with another: a declared version against output metadata, a claimed count against the raw table, and an exclusion decision against the file consumed downstream.

<img src="../assets/07-cross-model-audit/02-dependency.png" width="520" alt="Input, computation, summary, and verification form a dependent workflow">

## First Calibrate the Reviewers

Before the live audit, the team prepared **36 paired test cases**, each with a flawed and a clean version. Three models evaluated both versions, giving **216 model × case × version judgments**. This is a count of judgments, **not** a count of distinct real-world problems.

There were **35 deliberately injected issues**. At the content level, the three models detected **35/35, 34/35, and 35/35**, respectively. The clean counterparts also mattered: their false-positive rates differed (**5.6%, 16.7%, and 0%**). These are results from this bounded test set, not general accuracy estimates or a guarantee for future audits. The figures are not assigned to named models here because the publishable summary provides them in order without an explicit model-to-rate mapping.

The paired design prevents a reviewer from appearing effective simply by calling every item defective.

<img src="../assets/07-cross-model-audit/03-blind-test.png" width="520" alt="36 paired blind tests produce 216 judgments; calibration data is not the live defect count">

## Roles, Independence, and Read-Only Evidence

- **Claude** split the workflow into bounded review batches, set consistent constraints, compared the reports, and drafted targeted follow-up questions.
- **Codex and Grok** received the same task independently, without seeing each other's answers. They inspected permitted execution logs, parameter and version records, intermediate tables, and underlying outputs in read-only mode.
- External methodological claims required verifiable official documentation or other authoritative sources; a model's recollection alone was insufficient.
- Sensitive raw inputs were excluded from the review material, and access scope was restricted.

The models were **not** asked to vote on which conclusion was true. Agreement could point to a shared observation, but reviewers might still depend on the same underlying source or make correlated mistakes. Disagreement became a reason to inspect specific evidence again.

The terminal panels below are **reconstructed, de-identified interface illustrations**. They contain no pixels, file paths, or findings from the supplied private screenshots.

<img src="../assets/07-cross-model-audit/04-dual-terminal.png" width="520" alt="Illustration of separate Grok and Codex read-only reviews with identical instructions">

[Optional animated dual-review illustration](../assets/07-cross-model-audit/04-dual-terminal-animation.gif)

## Four Batches, Two Rounds

The live workflow was divided into **four batches**. Codex and Grok each produced one first-round report per batch: **eight independent first-round reports**. After cross-comparison, both models received focused follow-ups, producing **eight second-round reports**.

The eight initial reports contained **87 raw finding entries**: 38 reported by one reviewer and 49 by the other. They included overlapping descriptions, repeated manifestations of one root cause, and findings classified under multiple headings. Therefore:

> **87 raw entries ≠ 87 independent, confirmed defects.**

The second round reopened **16 focused questions** across the four batches (3 + 4 + 5 + 4), including factual disagreements, severity disputes, and evidence gaps. A focus question is not a count of confirmed defects either.

For a reproducible measure of review volume, the **16 live review reports** contain **5,466 lines and approximately 306,000 characters**. That excludes the prompts, blind-test outputs, design review, and remediation notes. No reliable human-hours figure is available; elapsed file timestamps are not active work time.

<img src="../assets/07-cross-model-audit/05-audit-funnel.png" width="520" alt="Four batches, 8 plus 8 reports, 87 raw entries and 16 focused second-round questions">

## Examples That Required Evidence, Not Confidence

1. **Record reconciliation:** 4,567 inputs and 4,556 outputs left 11 dispositions insufficiently explained in the recorded check. The conclusion was an unexplained reconciliation gap, not an assertion of confirmed data loss.
2. **Field completeness:** in a **936,774-row** result table, a critical field was empty throughout. A completed job and a populated row count would not reveal that failure.
3. **Downstream propagation:** an item already excluded by human review still appeared with a `Pass` status downstream. The response preserved the original output, created an auditable derived exclusion view, and added an enforcement check.

These examples describe different failure modes. They should not be added together as if they were three disjoint categories in a complete issue inventory.

<img src="../assets/07-cross-model-audit/06-field-evidence.png" width="520" alt="Three anonymized evidence checks: reconciliation, empty field, and a downstream Pass state">

## Actions Taken and Rules Retained

The remediation ledger identifies **at least 18 distinct executed actions** with a traceable observation, evidence check, action, and state. The actions span recalculation, field and declaration corrections, containment, complete rechecks, and automated validation. As one example, cross-table matching was changed to use a stable unique identifier and **119 records** were rerun through the check.

Six classes of deterministic QA were recorded as repeatable controls:

1. Compare a declared version with the version actually used at execution.
2. Require measured values on both sides of overlapping records.
3. Compare low-signal values with the measured background setting.
4. Compare a declared reference-data version with the version in actual output.
5. Check cross-table joins against stable unique identifiers across the full set.
6. Block previously excluded items from re-entering an active downstream set.

**18 actions are not 18 fully eliminated root causes.** Some actions make an unresolved limit visible, prevent a bad state from propagating, or correct an overclaim while a deeper rerun remains pending.

<img src="../assets/07-cross-model-audit/07-action-board.png" width="520" alt="18 documented executed actions and six types of automatic QA; 119 records rechecked in one action">

## What Stayed Open

The ledger explicitly preserves **at least five open work items**. Their categories include independent cross-version checking, rebuilding missing fine-grained evidence, extending recomputation to additional items, re-exporting downstream results under an updated parameter, and running a more specialized method.

These are deliberately **not** presented as fixed. Where the underlying work was incomplete, the recorded action instead marked the state as unassessed, invalidated an obsolete value, preserved provenance, or blocked an unsupported downstream claim.

The count of open items may overlap with the 18 actions: an executed containment action and a pending underlying rerun can refer to the same issue family. Subtracting five from 18 would be meaningless.

<img src="../assets/07-cross-model-audit/08-honest-closure.png" width="520" alt="At least five open work items remain tracked instead of being presented as solved">

## A Reusable Audit Protocol

1. **Calibrate** reviewers with flawed/clean pairs, including a false-positive check.
2. **Bound** the real audit by scope, permissions, task wording, and evidence requirements.
3. **Review independently** with the same task and no access to the other model's report.
4. **Compare observations**, identifying duplicates, severity disagreements, and shared source dependencies.
5. **Reopen narrow questions** against raw records, metadata, and authoritative documentation.
6. **Record a disposition** for every accepted issue: fix, downgrade/close, clarify provenance, or keep open.
7. **Turn deterministic recurring checks into QA**, and rerun them over the relevant population rather than only the first example.

## Relationship to Earlier Cases

Case #05 used multiple Agents to expand code-review coverage and emphasized that a review finding is only a candidate defect. This case goes further at the **evidence and execution boundary**: two independent reviewers examine the same real-world workflow, challenge each other's gaps, and return to source records before a conclusion is accepted.

The recurring reliability distinction is:

```text
Model agreement != independent evidence
Successful execution != verified output
Executed remediation != all underlying work complete
```

## Evidence and Limits

This public case is based on an anonymized audit summary and a deduplicated remediation ledger current to **2026-09-13**. The private source reports and execution data are not included. Readers can verify the arithmetic and the published counting rules here, but cannot independently reproduce the private analyses from this repository alone.

In particular:

- The 216 blind-test judgments, 87 raw finding entries, 16 reopened questions, 18 remediation actions, six QA classes, and at least five open tasks measure **different things**.
- Content-level blind-test detection does not establish a universal detection rate; the clean controls produced different false-positive rates.
- The 11-record example shows missing reconciliation in a recorded check; it does not establish permanent loss of 11 records.
- Model agreement is a review signal, not a new experiment or an independent source of truth.
- This case does not claim that every issue was fully resolved or that AI replaces expert judgment, additional data, or controlled validation.
- Original screenshots are intentionally excluded. The public images contain only aggregate, de-identified values and reconstructed interface placeholders; no private paths or substantive output are recoverable from them.

## Engineering Rule

> **Model output identifies where to inspect. Evidence determines what is true.**

The objective is not to make a review queue appear empty. It is to leave each claim with a traceable source, each executed action with a verified state, and each unresolved gap explicitly open.
