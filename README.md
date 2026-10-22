# Support-loss and retraining-calibrated knowledge-graph unlearning

Anonymous source-code supplement.

## Installation

Use Python 3.10 or newer and a PyTorch installation suitable for the available
CPU/CUDA hardware. From this directory:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
export PYTHONPATH="$PWD${PYTHONPATH:+:$PYTHONPATH}"
export OMP_NUM_THREADS=1
export MKL_NUM_THREADS=1
export OPENBLAS_NUM_THREADS=1
python -m pytest tests -q
```

The dependency list describes imports required by this source snapshot; it is
not a recovered lockfile of the experimental machines. GPU jobs default to
`cuda:0`; use `--device cpu` for supported CPU runs. Large full-vocabulary rank
audits and the stronger-training grid can require substantial compute.

## Data layout

Supply the public datasets separately, retaining the original triple order.
Paths are relative to the directory passed through `--data-root`:

| Dataset | Files |
| --- | --- |
| FB15K-237 | `FB15k-237/train.txt`, `valid.txt`, `test.txt` |
| CoDEx-M | `codex/data/triples/codex-m/train.txt`, `valid.txt`, `test.txt` |
| NELL-995 | `kb_reasoning/NELL-995/raw.kb` |

Each line is a whitespace-separated head, relation, tail triple. The loader
constructs sorted entity/relation vocabularies. NELL-995 uses the deterministic
holdout implemented in `schemacert/data.py`; do not substitute a different split
when checking the recorded request hashes.

## Experiments included

All paths below are under `scripts/experiments/`.

| Experiment | Entry points |
| --- | --- |
| Support-loss requests and cross-family audits | `run_entity_request_robustness.py`, `run_supportloss_unified.py`, `run_supportloss_cross_family.py`, `evaluate_family_restart_extended.py` |
| W5 mean-tie calibration, 36 settings | `rerun_w5_mean_tie.py`, `rerun_w5_lowmid_null.py`, `bootstrap_w5_mean_tie_query_ci.py` |
| Matched background-context placement | `run_unrelated_placement_probe.py`, `aggregate_unrelated_placement_probe.py` |
| Third independent reference | `run_w5_third_reference.py`, `aggregate_w5_three_reference_null.py`, `run_w7_refc_probe.py` |
| Stronger training | `run_w6_well_trained_distmult.py`, `run_w7_stronger_training_grid.py`, `aggregate_w7_stronger_training_grid.py` |
| Reference-free correction (W8) | `w8_reference_free_correction/run_w8_reference_free_placement.py`, corresponding aggregate script |
| Correction under stronger training | `w8_stronger_transfer/run_w8_stronger_transfer.py`, corresponding aggregate script |
| Published-method rank and placement audits | `run_published_method_rank_audit.py`, `run_published_method_placement_probe_v2.py`, corresponding aggregate scripts |
| Correction transferred to published methods (W9) | `w9_method_transfer/run_w9_method_transfer.py`, corresponding aggregate script |
| Scale/correction decomposition (W12) | `w8_reference_free_correction/run_w12_scale_rfpc_decomposition.py` |
| Reference-free norm correction (W13) | `w8_reference_free_correction/run_w13_reference_free_norm.py`, `analyze_w13_placement.py` in the same directory |
| Norm-matched controls (W14) | `w8_reference_free_correction/run_w14_rf_norm_matched_controls.py` |
| Norm mechanism analysis | `w8_reference_free_correction/analyze_section7_norm_mechanism.py` |
| Oracle norm diagnostic | `audit_norm_match_oracle_v2.py`, `w8_reference_free_correction/analyze_normoracle_placement.py` |

Additional ablations, collectors and shared utilities are retained because they
support the experimental workflows. The earlier placement implementation is
retained alongside v2 for dependency and historical-comparison compatibility.
Oracle diagnostics and reference-free corrections are distinct experiments.

## Reproduction inputs and order

This is a code-only archive: datasets, checkpoints, raw query-rank dumps and
computed results are not bundled. Some evaluation scripts consume artifacts
from earlier stages rather than training models themselves. `CLI_REFERENCE.md`
lists the actual arguments and defaults extracted from every command-line
entry point. Each script also supports `--help`.

1. Prepare the dataset layout above. Generate request-robustness artifacts using
   `run_entity_request_robustness.py` for each dataset and seed 17, 29 and 43.
2. Run the W5 high/exact-high and low/mid scripts using the historical-input
   contracts below. Store both output trees below a common W5 directory.
3. Run the background-context placement probe on those W5 checkpoints.
4. Fit and evaluate W8 using the W5 and placement roots; aggregate the 36 cells.
5. Run the stronger-training grid, third-reference checks and correction-transfer
   branches separately. Their cohorts and reference procedures differ.
6. Run W12/W13/W14 and the norm analyses using their declared upstream artifacts.

