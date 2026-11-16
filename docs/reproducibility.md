# Reproduction notes

## Reporting application

From the project root:

```powershell
uv sync --locked
uv run python build_verified_briefs.py
```

The five builders use the prepared CSV summaries committed under `results/`. They do not need an LLM, local model weights, or the original Excel file to regenerate the briefs.

To rebuild the summaries, obtain the supplied Online Retail spreadsheet, save it as `data/Online Retail.xlsx`, and run `uv run python analyze_data.py`. The raw file is intentionally not committed. Exact-duplicate removal is an assumption; valid unidentified sales remain in product and revenue calculations.

## Recorded model experiment

The recorded environment used Windows, Python 3.14, PyTorch `2.14.1+cu130`, Transformers `4.57.6`, and an NVIDIA GeForce RTX 5090. The model and tokenizer are pinned to `google/flan-t5-small` revision `0fc9ddf78a1e988dac52e2dac162b0ede4fd74ab`.

The v3 dataset has 640 training, 160 validation, and 160 test prompts. These are fictional teaching scenarios, with two instruction variations per source group. Several families contain Python-computed figures. The original model evaluation must not be described as independent arithmetic testing or evidence of production reliability.

### Frozen fingerprints and line endings

The recorded dataset and audit hashes were computed from Windows CRLF files. Byte-level fingerprints can differ after a Git checkout that uses LF line endings, even when the parsed JSON records are identical. This is a portability limitation of the original experiment.

For a new checkout intended to reproduce those frozen byte checks, use:

```powershell
git clone --config core.autocrlf=true https://github.com/rwnycvet/retail-market-research.git
cd retail-market-research
```

The model/audit scripts reject changed fingerprints. Do not bypass the checks or quietly replace frozen test data to make an evaluation pass. Reporting-only generation does not depend on those byte-level fingerprints.

### Training and scoring

```powershell
uv run python train_multitask_v3.py
uv run python score_v3_responses.py
```

Training requires the configured CUDA environment, downloads the pinned base model, and writes local weights under `models/flan-t5-multitask-v3`. Model weights are excluded from Git. Training regenerates the v3 experiment outputs; preserve the recorded evidence before running a new experiment.

For the controlled audit, the local fine-tuned model must already exist:

```powershell
uv run python run_fairness_audit_v3.py
uv run python score_fairness_audit_v3.py
```

The audit runner protects existing response files from replacement. Its 16 cases are previously reviewed regression scenarios, not a fresh population-fairness study.

## Evidence paths and privacy

Saved audit model locations and the older dataset-hash CSV use paths relative to the repository root. Only those path fields were normalized during portfolio cleanup; model response text, numeric results, and recorded data hashes are retained. Audit runners save relative model locations for future runs as well.

The graded coursework branch is retained separately. Earlier commits can still contain the original local paths; this presentation cleanup does not rewrite Git history or erase the historical submission.
