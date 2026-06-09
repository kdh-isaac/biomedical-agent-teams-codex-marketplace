# Claim Ledger Template

Use this template for source-backed biomedical outputs, omics reports,
evidence audits, translational scans, manuscript support, and deep research
council runs. Keep rows atomic: one claim per row.

| claim_id | atomic_claim | claim_type | context | evidence_items | evidence_relation | uncertainty | audit_status | allowed_final_wording |
|---|---|---|---|---|---|---|---|---|
| CL-001 |  | descriptive / mechanistic / causal / prognostic / predictive / therapeutic / translational / feasibility / safety / IP-strategy / method / limitation | species; cell type; disease/model; assay; endpoint; cohort/dataset; perturbation; biological unit | PMID/DOI/accession/NCT/file/artifact/retrieval date | direct / indirect / proxy / contradictory / missing / not checked | low / moderate / high + reason | unchecked / needs audit / pass / pass-with-caveats / block | final-safe wording or empty if blocked |

## Excluded Or Not Verified Claims

Use this section for useful ideas, interpretations, or claims that should not
enter the final conclusion yet.

| item_id | excluded_or_not_verified_claim | reason_excluded | minimum_evidence_needed |
|---|---|---|---|
| EX-001 |  | not source-checked / unsupported / proxy-only / contradicted / provenance gap / unsafe to disclose |  |

## Writer Rule

The final writer may use only `allowed_final_wording` from rows with
`audit_status` of `pass` or `pass-with-caveats`. Any other useful point stays in
`Excluded Or Not Verified Claims`.
