# Retail Market Research

Python retail analytics and local language-model evaluation for customer purchasing, product demand, country sales, and inventory planning.

The workflow turns transaction records into five decision briefs. A separate FLAN-T5 experiment tests whether a small model can report supplied evidence accurately. Business rules and numerical findings are verified in Python before they appear in the reporting application.

## Business questions

- Which recorded customers purchase repeatedly, and what does the available customer coverage support?
- Which products lead in positive units, and how do transaction reversals affect that picture?
- Where does this retailer record the most sales revenue?
- Which monthly changes can be compared fairly when the final month is incomplete?
- What evidence is missing before competitor or market-share claims can be made?

## Example findings

| Finding from the supplied retailer records | Decision supported |
|---|---|
| 2,845 of 4,338 identified customers made at least two qualifying purchase invoices: 65.58%. | Review purchase intervals and product choices before designing a marketing test. This is repeat purchasing among identified customers, not an all-customer retention rate. |
| Product 23843 recorded 80,995 positive units but a signed balance of zero after negative quantities. | Review transaction reversals before using a gross-sales ranking to justify replenishment. |
| November 2011 had GBP 1,503,866.78 in positive sales revenue, the highest complete month. December 2011 ends on December 9. | Review inventory timing using comparable periods; a partial December total cannot establish falling full-month demand. |

These are historical findings for one retailer. Marketing and inventory actions are recommendations, not measured outcomes of an intervention.

## View the briefs

The committed outputs can be reviewed without downloading a model or running training:

- [Combined retail research briefs](results_v3/verified_reports/retail_research_briefs.md)
- [Customer purchasing](results_v3/verified_reports/customer_brief.md)
- [Product demand and inventory](results_v3/verified_reports/inventory_brief.md)
- [Country sales and coverage](results_v3/verified_reports/country_brief.md)
- [Monthly sales trends](results_v3/verified_reports/monthly_brief.md)
- [Competitor-evidence requirements](results_v3/verified_reports/competitor_brief.md)

Each brief distinguishes measured findings, recommendations, and limitations. The reporting application uses prepared CSV summaries, Python calculations, and fixed templates. Its outputs are separate from the raw model evaluation.

## Run locally

The managed project requires Python 3.14 or later and uv. Its recorded dependency environment includes CUDA-enabled PyTorch; model training requires the configured NVIDIA CUDA environment.

```powershell
git clone --config core.autocrlf=true https://github.com/rwnycvet/retail-market-research.git
cd retail-market-research
uv sync --locked
uv run python build_verified_briefs.py
```

The application reads the committed summaries under `results/` and writes JSON and Markdown briefs under `results_v3/verified_reports/`. Regenerating these briefs does not require the original spreadsheet or a training run.

See [reproduction notes](docs/reproducibility.md) for the original training environment, frozen dataset fingerprints, and optional model evaluation commands.

## Data and methods

The analysis uses the supplied [UCI Online Retail dataset](https://archive.ics.uci.edu/dataset/352/online+retail), covering December 2010 through December 9, 2011.

- Keep one occurrence of each exact duplicate row, as a documented cleaning assumption.
- Exclude cancellation invoices, nonpositive quantities, and nonpositive unit prices from positive-sales analysis.
- Retain otherwise valid sales without CustomerIDs for revenue and product totals.
- Calculate positive sales revenue as `Quantity * UnitPrice` in GBP, before return and cancellation deductions.
- Group products by StockCode; compare positive units with cancellation-inclusive signed quantities.
- Count distinct qualifying invoices per identified customer when measuring purchase frequency.
- Compare only consecutive complete months; flag the partial final month.

To rebuild the summaries, place `Online Retail.xlsx` in `data/` and run `uv run python analyze_data.py`. The spreadsheet and local model weights are excluded from Git.

## Model evaluation

The FLAN-T5 Small experiment uses five equally represented task families: consumer behavior, country coverage, market trends, product demand, and competitor research.

| Split | Prompt examples | Underlying scenarios |
|---|---:|---:|
| Training | 640 | 320 |
| Validation | 160 | 80 |
| Test | 160 | 80 |

Each scenario has two instruction variations kept in the same split. All v3 examples are synthetic. Several tasks supply figures already computed by Python; results assess evidence reporting and interpretation rather than independent arithmetic. Splits share templates and reasoning patterns despite source-group and identical-evidence separation.

The recorded run pins `google/flan-t5-small` revision `0fc9ddf78a1e988dac52e2dac162b0ede4fd74ab` and Transformers `4.57.6`. It used six epochs, batch size four, learning rate `0.00005`, seed `928`, and 960 optimizer updates. Epoch 6 had the lowest validation loss: 0.020701, from an initial 3.468904.

| Test measure | Original model | Fine-tuned model |
|---|---:|---:|
| Strict reference matches | 0/160 | 132/160 (82.5%) |
| Scenarios with both variations matching | 0/80 | 64/80 |
| Empty responses | 12 | 0 |

Strict agreement normalizes whitespace only. Zero strict matches do not mean zero factual accuracy. Manual review found 17 customer-warning errors and 11 monthly-trend issues among the fine-tuned mismatches. The model still requires verification before business use.

### Geographic-label audit and safeguards

A controlled 16-case regression audit varies four geographic labels and four customer counts. Both models scored 8/16 and missed every required below-five-customer warning. Equal label-level performance reflected consistent failure, not proof of population fairness.

The Python safeguard scored 16/16 on the defined `identified_customers < 5` rule. This application-level result is separate from model performance and does not establish broad fairness.

- [Evaluation and limitations](results_v3/evaluation.md)
- [Training record](results_v3/expanded_run/training_run.json)
- [Original and fine-tuned responses](results_v3/expanded_run/model_comparison.json)
- [Strict reference scores](results_v3/expanded_run/strict_reference_scores.json)
- [Audit decision scores](results_v3/fairness_audit/decision_scores.json)

## Repository guide

| Location | Purpose |
|---|---|
| `analyze_data.py`, `results/` | Retail data preparation and measured summaries |
| `build_*_brief.py`, `build_verified_briefs.py`, `evidence_rules.py` | Reporting application and deterministic business rules |
| `training_data_v3/`, `train_multitask_v3.py` | Expanded synthetic dataset and recorded training workflow |
| `results_v3/`, `score_v3_responses.py`, `*_fairness_audit_v3.py` | Model comparisons, evaluation, and controlled audit |
| `submission_v1/`, `training_data_v2/`, `results_v2/` | Preserved earlier experiments and investigation evidence |
| `docs/` | Reproduction notes and original coursework documentation |

## Scope, provenance, and AI assistance

This project began as Per Scholas coursework and was organized as a portfolio case study. The [original report](docs/coursework/report.pdf) and [coursework record](docs/coursework/README.md) preserve that provenance. The graded `sba928-resubmission` branch remains available as the original reviewed version.

AI tools assisted with explanations, code scaffolding, debugging, and documentation. The source, saved generations, quantitative checks, and known failures are retained for review.

The records do not establish national demand, customer motives, competitor performance, or market share. Exact reorder quantities require stock levels, supplier lead times, and other operational data. Recorded sales do not capture demand lost through stockouts. Synthetic model scores do not establish production readiness or generalization to new retail tasks.
