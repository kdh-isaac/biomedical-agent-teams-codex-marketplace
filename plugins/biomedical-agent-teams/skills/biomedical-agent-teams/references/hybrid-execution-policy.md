# Hybrid Execution Policy

BMAT is lead-controlled and inline-first. The main assistant owns the protocol
lock, source scope, central claim ledger, workflow-run state, and final
synthesis. Spawned subagents are a review and parallelization tool, not the
default way to execute every role.

## Execution Strategies

| strategy | use when | avoid when |
|---|---|---|
| `inline_only` | The request is quick, narrow, conceptual, or low risk. | The user asked for independent review or the answer depends on high-confidence source-backed claims. |
| `inline_first_selective_review` | The main workflow is best kept coherent, but selected reviewer independence improves audit quality. | The task needs several independent domain teams to work in parallel. |
| `team_level_selective_dag` | The question has separable decision axes such as idea generation, omics feasibility, translational scouting, experiment design, and evidence audit. | The task has a single claim or one obvious workflow recipe. |
| `user_requested_full_spawn` | The user explicitly requests broad spawning after being told it is usually inefficient. | The user did not authorize broad spawning or the budget/privacy boundary is unclear. |
| `blocked` | Required runtime capability, safety, privacy, or scope constraints prevent the requested execution pattern. | A narrower inline or selective-review workflow can answer safely. |

## Selective Spawn Review

Use this pattern when the most efficient workflow is inline, but the final
answer should look and behave like an audited output.

1. Main lead runs the command recipe inline.
2. Main lead builds or updates the central claim ledger.
3. Spawn only reviewer roles that materially improve the output:
   - `claim-level-evidence-verifier`
   - `citation-verifier`
   - `contradiction-red-team`
   - `biostats-repro-auditor`
   - `omics-provenance-validator`
   - `risk-of-bias-study-quality-auditor`
   - `safety-ethics-privacy-dual-use-auditor` when triggered
4. Reviewer outputs must map to claim IDs or clearly state that no ledger was
   available.
5. Main lead accepts, rejects, or downgrades reviewer findings in the ledger
   before final writing.

Do not call same-model separate-pass review independent unless a spawned
subagent, separate model, tool-backed validator, external verifier, or human
reviewer actually performed the review.

## Team-Level Selective DAG

Use this pattern when several command-level teams can answer independent parts
of a broad research decision.

Phase 0, inline lead lock:

- protocol/context lock
- entity normalization
- evidence scope and source-corpus plan
- execution strategy and spawn budget
- team dependency graph
- central claim-ledger skeleton

Phase 1, independent spawned team bundles when useful:

- `idea-discovery-team`: candidate hypotheses, mechanism/ranking frame, and
  useful excluded ideas.
- `omics-analysis-team`: public-data feasibility, S1-S3 plan, or locked audit.
  Full `run` requires S1-S3 validation before S4/S5 claims.
- `translational-scout-team`: trial, regulatory, IP, operational, and safety
  boundary scan.

Phase 2, dependency-bound spawned team bundles when useful:

- `experiment-design-team`: run after the lead narrows one or more candidate
  hypotheses or designs.
- `evidence-audit-team`: run after final candidate claims, draft text, or result
  claims are available.

Phase 3, inline synthesis:

- merge accepted spawned-team outputs into the central claim ledger
- keep rejected or not-verified material out of final wording
- run claim/citation/post-write validation
- report execution strategy, spawned outputs, skipped gates, and downgrade
  reasons

## Anti-Patterns

- Spawning every role by default.
- Spawning every team by default.
- Letting spawned teams spawn nested child agents without explicit approval.
- Running `experiment-design-team` before a hypothesis is narrowed.
- Running `omics-analysis-team --mode run` before accession/cohort, assay,
  biological unit, contrast/endpoint, and S1-S3 validation are locked.
- Treating team output as final truth without central claim-ledger mapping.
- Hiding reviewer objections or contradiction findings in the synthesis.

## Mode Defaults

| mode | default strategy | typical spawn budget |
|---|---|---|
| quick | `inline_only` | 0 |
| standard | `inline_only` or `inline_first_selective_review` | 0-1 |
| deep | `inline_first_selective_review` or `team_level_selective_dag` | 1-3 |
| audit | `inline_first_selective_review` with optional team DAG for multi-axis audits | 1-4 |
| omics plan | `inline_only` unless feasibility axes are independent | 0-1 |
| omics run | `inline_first_selective_review` after S1-S3 locks | 1-3 |

## Required Spawned Output Contract

Every spawned reviewer or spawned team must return:

- objective and assigned scope
- inputs, files, metadata, and sources checked
- tools, skills, commands, or databases used
- key findings, contradictions, risks, and confidence level
- files changed, if any, or `none`
- checks run, skipped checks, and reasons
- recommended handoff
- claim IDs or ledger rows affected when available

A bare "done" is not sufficient for BMAT review or team output.
