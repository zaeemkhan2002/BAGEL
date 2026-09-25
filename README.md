# BAGEL — Anonymous Code Repository

Supplementary notebooks for "Efficient and Adaptable Detection of Malicious LLM Prompts via
Bootstrap Aggregation" (ICLR 2027 submission). These reproduce the paper's core results: the
random-selection ensemble, the optional DistilBERT router, per-model threshold calibration,
the out-of-distribution adaptability pilot (model_11), and the dataset-distance check that
justifies treating the held-out test set as genuinely out-of-distribution.

## Files

### `bagel_router_anon.ipynb`
Main experiment notebook. Builds the 9-promptcop ensemble (`model_1..model_8, model_10`),
evaluates Random Selection vs. the DistilBERT-routed selection strategy (§3.3–3.4 of the
paper) across ensemble sizes `n = 1..9`, and compares two thresholding regimes — a shared
fixed threshold and the per-model calibrated threshold described in §3.5 (the latter is what's
reported in the paper; the shared-threshold runs are kept in for transparency on how that
design choice was reached). Also contains the adaptability pilot: retrains the DistilBERT
router to include `model_11` (§5.2) and compares ensemble performance on WildJailbreak-eval
before (`k=9`) and after (`k=10`) its addition. Produces the F1/ASR/FPR curves and tables
underlying Table 1, Figure 3, and Figure 4.

Requires the 9 (soon 10, after `model_11`) fine-tuned promptcop checkpoints and the DistilBERT
router checkpoint as inputs — see `train_model_11_anon.ipynb` for how `model_11` specifically
is produced; the other promptcops follow the same fine-tuning procedure over their respective
training datasets (§3.2).

### `train_model_11_anon.ipynb`
Fine-tunes `model_11`, the additional promptcop used in the out-of-distribution adaptability
experiment (§5.2, RQ3). Trains a fresh Prompt Guard 2 (86M) classifier on
`allenai/wildjailbreak`'s `train` split — a dataset disjoint from the original 9-dataset
training corpus and from the `eval` split used in the held-out test set, so there's no
train/test leakage. Includes an optional throughput-benchmarking step to estimate full-run
time before committing to it. Output feeds directly into `bagel_router_anon.ipynb`'s
before/after comparison.

### `bagel_ood_distance_anon.ipynb`
Validates that the 5 held-out OOD test datasets are distributionally distinct from the 9
training datasets, rather than just disjoint by source label. Embeds every dataset's prompts
into a shared sentence-embedding space and computes multiple distance/divergence metrics
between each OOD set and each training set, then reports each OOD dataset's nearest training
neighbor. This is the evidence behind the paper's claim that reported test performance
reflects genuine generalization to out-of-distribution attacks, not near-duplicate leakage
across dataset boundaries.

## Reproducing paper results

1. Fine-tune the 9 base promptcops on their respective training datasets (§3.2), or use
   provided checkpoints if included.
2. Run `train_model_11_anon.ipynb` to produce the 10th, out-of-distribution promptcop.
3. Run `bagel_router_anon.ipynb` top to bottom — produces the main F1/ASR/FPR results,
   the DistilBERT router comparison, and the before/after adaptability comparison.
4. Run `bagel_ood_distance_anon.ipynb` independently to reproduce the distributional
   distance analysis (no dependency on steps 1–3; only needs the raw datasets).

## Notes

- All notebooks were run on Google Colab; paths and credentials have been anonymized for
  double-blind review. Set `SAVE_DIR`/`MODELS_DIRECTORY`/etc. to your own storage location
  before running, and authenticate with your own Hugging Face token where a cell calls for one
  (required for the gated `nvidia/Aegis-AI-Content-Safety-Dataset-2.0` dataset).
- Random seeds are fixed (`SEED = 42`) throughout for reproducibility.
- `bagel_router_anon.ipynb` contains results for two thresholding regimes: a single
  shared/global "ideal" threshold (fit once across the whole calibration set) and per-model
  calibrated thresholds (fit independently per promptcop, §3.5). The shared-threshold results
  were an earlier approach that was ultimately **abandoned**; per-model calibration is what the
  paper reports throughout. Both are left in the notebook for transparency on how that design
  choice was reached.
