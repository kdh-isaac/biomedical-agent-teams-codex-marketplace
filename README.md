# Biomedical Agent Teams Codex Marketplace

Codex Desktop marketplace package for the Biomedical Agent Teams plugin.

Current plugin version: `0.4.9`.

## Install

Clone this repository, then register the local marketplace path:

```powershell
git clone https://github.com/kdh-isaac/biomedical-agent-teams-codex-marketplace
codex plugin marketplace add "<path-to-clone>"
codex plugin add biomedical-agent-teams@biomedical-agent-teams-marketplace
```

The plugin body is in `plugins/biomedical-agent-teams/` and exposes the
`biomedical-agent-teams` skill with 35 agent prompts, 6 command recipes, loop
engineering resources, connector binding, a lightweight lazy-loaded router,
agent registry metadata, Codex
reviewer-agent TOML templates, workflow-run spawned instance tracking,
team-level DAG output tracking, deterministic artifact validators, and
validator-backed release gates with golden-case eval coverage for PMID drift,
contradiction, and overclaim detection.

## Workflow Structure

```mermaid
flowchart TD
    accTitle: BMAT v0.4.9 Workflow Structure
    accDescr: Router-first BMAT workflow with lazy-loaded commands, artifact spine, optional team DAG, reviewer, and loop lanes, deterministic validators, and golden eval release gates.

    request["User request<br/>or BMAT alias"]
    router["Lightweight SKILL.md router<br/>select narrow command recipe"]
    preflight["Runtime capability preflight<br/>scope + source + risk + strategy"]
    source_lock["Source corpus lock<br/>PMID / DOI / accession / dataset provenance"]
    strategy{"Execution strategy"}
    inline["Inline command recipe work<br/>lead-controlled specialist synthesis"]
    ledger["Central artifact spine<br/>claim ledger + workflow-run state + stage evaluation"]
    postwrite["Post-write validation<br/>ledger-only final wording"]
    release{"Release gate"}
    label["Final workflow label<br/>Full / Contract / Compact / Limited / Partial / Blocked"]

    request --> router --> preflight --> source_lock --> strategy
    strategy --> inline --> ledger --> postwrite --> release --> label

    subgraph team_dag["Optional team-level selective DAG"]
        direction TB
        team_plan["team_spawn_lanes<br/>phase + depends_on + nested policy"]
        phase1["Phase 1 outputs<br/>idea / omics / translational"]
        phase2["Phase 2 outputs<br/>experiment design / evidence audit"]
        team_outputs["team_output_artifacts<br/>artifact_id + path + checks + ledger_handoff"]
        dag_guard{"bmat_validate.py DAG guard<br/>unique IDs + prior-phase dependencies"}
        team_plan --> phase1 --> phase2 --> team_outputs --> dag_guard
    end

    subgraph review_lane["Optional selective spawned review"]
        direction TB
        registry["agent-registry.json<br/>codex-agents/*.toml"]
        instances["spawned_agent_instances<br/>unique instance_id + output_artifact"]
        rcontract["spawned-agent-output contract"]
        rhandoff["accepted findings<br/>ledger handoff"]
        registry --> instances --> rcontract --> rhandoff
    end

    subgraph loop_layer["Optional recurring loop"]
        direction TB
        loop_recipe["loops/*.md recipe"]
        loop_state["loop_state.json"]
        loop_check["bmat_loop_check.py"]
        loop_recipe --> loop_state --> loop_check
    end

    subgraph eval_gate["Package and release validation"]
        direction TB
        package_check["bmat_package_check.py<br/>router size + lazy-load references"]
        selftest["bmat_selftest.py<br/>dependency-free smoke"]
        golden_schema["validate_golden_eval_schema.py"]
        golden_gate["run_golden_eval.py --strict --gate<br/>PMID drift + contradiction + overclaim"]
        package_check --> selftest --> golden_schema --> golden_gate
    end

    strategy -. "broad dependent axes" .-> team_plan
    dag_guard --> ledger
    ledger -. "independent audit needed" .-> registry
    rhandoff --> ledger
    strategy -. "watch / inbox / triage" .-> loop_recipe
    strategy -. "package maintenance" .-> package_check
    loop_check --> ledger
    golden_gate --> release
```

The workflow is router-first: `SKILL.md` stays small and loads only the selected
command recipe. The lead agent owns preflight, source locking, the central
ledger, post-write validation, and the final label. Execution strategy is
either `inline_first_selective_review` (default, lead-controlled inline work
with selected reviewer roles) or `team_level_selective_dag` for broad,
dependent decisions that spawn command-level teams across the DAG above.
Optional lanes feed evidence back into that artifact spine. Team DAG claims
are proven by unique `team_spawn_lanes` and `team_output_artifacts` records
with prior-phase dependencies; reviewer execution is proven by unique
`spawned_agent_instances`; recurring loops are checked by
`bmat_loop_check.py`. Full-protocol release requires post-write validation
and `bmat_validate.py` on the complete bundle, while package releases
additionally run the package/selftest/golden-eval gates.

## Contents

- `.agents/plugins/marketplace.json`: local marketplace metadata.
- `plugins/biomedical-agent-teams/`: Codex plugin body.
- `plugins/biomedical-agent-teams/skills/biomedical-agent-teams/`: skill
  router, agents, commands, contracts, templates, references, loops, tests, and
  validators.

## Validation

The 0.4.9 package is validated with:

```powershell
python -m pip install ".[dev]"
python plugins/biomedical-agent-teams/skills/biomedical-agent-teams/scripts/bmat_package_check.py --root plugins/biomedical-agent-teams
python plugins/biomedical-agent-teams/skills/biomedical-agent-teams/scripts/bmat_selftest.py --root plugins/biomedical-agent-teams
python -m pytest -q
python plugins/biomedical-agent-teams/skills/biomedical-agent-teams/evals/validate_golden_eval_schema.py --tasks plugins/biomedical-agent-teams/skills/biomedical-agent-teams/evals/golden_tasks.jsonl --outputs plugins/biomedical-agent-teams/skills/biomedical-agent-teams/evals/sample_outputs.jsonl
python plugins/biomedical-agent-teams/skills/biomedical-agent-teams/evals/run_golden_eval.py --tasks plugins/biomedical-agent-teams/skills/biomedical-agent-teams/evals/golden_tasks.jsonl --outputs plugins/biomedical-agent-teams/skills/biomedical-agent-teams/evals/sample_outputs.jsonl --strict --gate
```

This is exactly what `.github/workflows/ci.yml` runs on every push/PR across
Ubuntu, macOS, and Windows for Python 3.10-3.13 (no live network or model
calls; all checks are deterministic and offline).

## Release Process

1. Decide the new version (semver). Update it in all four places together:
   `plugins/biomedical-agent-teams/skills/biomedical-agent-teams/VERSION`,
   `plugins/biomedical-agent-teams/.codex-plugin/plugin.json` (`version`),
   `plugins/biomedical-agent-teams/skills/biomedical-agent-teams/manifest.json`
   (`version` and `adapter_version`), and
   `plugins/biomedical-agent-teams/skills/biomedical-agent-teams/agent-registry.json`
   (`version`).
2. Add a `## v<version> Updates` section to both `README.md` and
   `plugins/biomedical-agent-teams/README.md`, describing what changed.
3. Run the Validation commands above locally; they must all pass.
4. Note whether the change is behavior-preserving (patch/minor) or changes
   validator pass/fail behavior for existing bundles (treat as a
   behavior-change release even if the version bump looks small, and say so
   explicitly in the changelog entry).
5. Commit, then tag and push: `git tag v<version> && git push origin main
   v<version>`.
6. `.github/workflows/release.yml` runs on the tag push. It refuses to
   release if the tag does not match `VERSION`/`plugin.json`/`manifest.json`,
   reruns the full gate suite, and publishes a GitHub Release with a
   `release-manifest.json` (commit SHA, gates passed, and an explicit
   `benchmark_status` field so a release never implies a live-model benchmark
   result it did not actually run).
