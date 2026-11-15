# Beyond Merge Rate — Performance-Related Pull Requests from Coding Agents

Public dataset for *Beyond Merge Rate*, an empirical study of performance-related pull requests associated with coding agents. The unit of analysis is one GitHub pull request. This repository is the frozen corpus behind that study: the master table, GitHub activity records, topic and task-type labels, and structured LLM analyses.

## Dataset at a glance

Counts below were checked against `finaldatabase/pr_master/perf_prs_expanded_final.csv`, the matching Parquet file, and `finaldatabase/per_pr/`. They agree with `finaldatabase/summary/coverage_stats.json` (`pr_count`, `status_counts`, `merge_rate`).

| Item | Value |
| --- | --- |
| Terminal pull requests | 1,183 |
| Merged | 694 |
| Closed without merge | 489 |
| Merge rate | 58.7% (694 / 1,183) |
| Open pull requests in this snapshot | 0 |
| Coding agents | 5 |
| Repositories | 418 |
| PR authors (`user`) | 224 |
| `created_at` range | 2024-12-26 to 2025-07-30 |

`status` is derived in `refresh.py`: `merged` when `merged_at` is present, otherwise `closed` once `state` is no longer `open`. In this snapshot every row has GitHub `state = closed`, `merged_at` is present for exactly the 694 merged rows, and `closed_at` is present for all 1,183. The unrounded ratio is stored as `merge_rate` (`0.5866441251056635`).

On 2026-09-21 the freeze removed 36 pull requests that were still open (`summary/terminal_freeze_report.json`). Those 36 plus the 1,183 retained rows are the pre-freeze count of 1,219. That earlier figure is the size before removal. It is not a second corpus in this repository. Current status counts are the master table and the top-level `status_counts` in `coverage_stats.json`. `terminal_freeze_report.json` and `status_refresh_report.json` are logs of that removal and of the refresh that preceded it.

Agent composition of the terminal set:

| Agent | PRs | Merged | Closed |
| --- | ---: | ---: | ---: |
| OpenAI_Codex | 629 | 460 | 169 |
| Devin | 225 | 74 | 151 |
| Copilot | 206 | 89 | 117 |
| Cursor | 89 | 49 | 40 |
| Claude_Code | 34 | 22 | 12 |
| **Total** | **1,183** | **694** | **489** |

## Repository layout

```
.
├── finaldatabase/          # frozen corpus
├── analysis_viz/           # reports and figures built from the distilled table
├── humanvalidation/        # coder workbooks for the recheck sample
├── output_pr_analysis/     # historical LLM batch ledger
├── full_analysis_distilled.csv
├── refresh.py
├── freeze_terminal_dataset.py
├── run_pr_analysis.py
├── generate_full_analysis.py
├── generate_rq_analysis.py
├── prompt.md
├── schema.json
├── manual_recheck_sample.md
└── {pr_id}_analysis.json   # six few-shot copies; canonical files are under per_pr/
```

| Path | Role |
| --- | --- |
| `finaldatabase/pr_master/` | Full master table (CSV and Parquet) and the narrower paper CSV |
| `finaldatabase/paper_source_copy/` | Byte-identical copy of the paper CSV |
| `finaldatabase/auxiliary/` | Corpus-level GitHub tables stacked across PRs |
| `finaldatabase/classification/` | BERTopic assignments, topic info, and AIDev task-type rows |
| `finaldatabase/per_pr/{id}/` | One directory per master-table `id` |
| `finaldatabase/summary/` | Coverage stats, freeze and refresh logs, API caches |
| `full_analysis_distilled.csv` | 1,183 × 56 table flattened from the per-PR analysis JSON |
| `analysis_viz/` | `FullAnalysis.md`, `RQ_Analysis.md`, figure scripts, and rendered figures |
| `humanvalidation/` | `A.xlsx` and `B.xlsx`, the workbooks named by `manual_recheck_sample.md` |
| `output_pr_analysis/` | July 2026 LLM batch ledger (`batch_run_report.json`, `batch_run_report.md`) |

`analysis_viz/figures/` and `analysis_viz/rq_analysis_figures/` hold the rendered plots. Files in `summary/` whose names start with `_` are sample or debug JSON. `_hits.txt` at the repository root is a path list of analysis files.

The batch ledger records analysis runs. Its own note says some ids in that log were later removed while still open. The terminal PR list is the master table.

## Records and identifiers

The corpus key is the GitHub issue id: `id` on the master table and `pr_id` on auxiliary tables, analysis JSON, and `full_analysis_distilled.csv`. `number` is the pull-request number inside a single repository and repeats across repositories. `per_pr/` directory names are this issue id. All 1,183 directories match the master `id` values, with no extras and no gaps.

`pr_master/perf_prs_expanded_final.csv` and `.parquet` are the same 1,183 × 24 table (unique `id`, no duplicate keys). Columns fall into four groups:

| Group | Columns |
| --- | --- |
| GitHub record | `id`, `number`, `title`, `body`, `user_id`, `user`, `state`, `created_at`, `closed_at`, `merged_at`, `repo_id`, `repo_url`, `html_url` |
| Study fields | `row_1based`, `agent`, `status`, `detection_source` |
| Task-type fields (stored as AIDev columns) | `aidev_task_type`, `aidev_task_reason`, `aidev_task_confidence` |
| Topic and classifier fields | `Topic`, `Probability`, `Representative_document`, `llm_output` |

`Topic` is an integer topic id, including `-1`. Topic names are in `classification/bertopic_topic_info.csv` (`Name`).

`POP_PULL_Requests_LLM_filtered_final.csv` is the narrower 1,183 × 14 table used as the paper source. It keeps `id`, `number`, `title`, `body`, `agent`, `user_id`, `user`, `state`, `created_at`, `closed_at`, `merged_at`, `repo_id`, `repo_url`, and `html_url`. The same bytes are stored under `pr_master/` and `paper_source_copy/`. The dropped columns are `row_1based`, `status`, `detection_source`, the three `aidev_task_*` columns, `Topic`, `Probability`, `Representative_document`, and `llm_output` (`PAPER_BASE_EXCLUDE` in `refresh.py`). On this CSV, a merged PR is one with a non-empty `merged_at`.

### Auxiliary tables and per-PR slices

`auxiliary/` stacks the per-PR Parquet files. Where a file exists both as a stack and as a per-PR slice, the slice row totals equal the aggregate table. A PR with no reviews, comments, linked issues, or (in one case) commits has no file of that type. `run_pr_analysis.py` treats a missing per-PR file as an empty table.

| Aggregate file | Rows | PRs represented | Join key | Per-PR file |
| --- | ---: | ---: | --- | --- |
| `pr_timeline.parquet` | 14,915 | 1,183 | `pr_id` | `timeline.parquet` |
| `pr_commit_details.parquet` | 31,009 | 1,182 | `pr_id` | `commit_details.parquet` |
| `pr_commits.parquet` | 4,116 | 1,182 | `pr_id` | `commits.parquet` |
| `pr_comments.parquet` | 1,993 | 601 | `pr_id` | `comments.parquet` |
| `pr_reviews.parquet` | 1,419 | 346 | `pr_id` | `reviews.parquet` |
| `pr_review_comments_v2.parquet` | 1,539 | 203 | `pull_request_review_id` → `pr_reviews.id` | `review_comments.parquet` |
| `related_issue.parquet` | 204 | 203 | `pr_id` | `related_issue.parquet` |
| `issues_linked.parquet` | 196 | — | `id` (issue id) | — |
| `repository.parquet` | 418 | — | `id` (repo id) | — |

Every review-comment row matches a review, and those reviews cover 203 PRs, the same 203 directories that contain `review_comments.parquet`. `repository.id` is the same set as master `repo_id` (418 repositories). `issues_linked.parquet` is issue detail keyed by issue id (196 issues). The PR with no commit files is `2759847657`.

`coverage_stats.json` matches these row counts except for the timeline. It still records 13,491 timeline rows. The current `pr_timeline.parquet` has 14,915. The difference, 1,424 events, is the backfill logged in `summary/timeline_end_actor_fill.json` (117 PRs, all in the master table). Use the Parquet file for timeline rows, and use `coverage_stats.json` for the status split.

### Classification tables

| File | Contents |
| --- | --- |
| `bertopic_topic_assignments.parquet` | 1,183 rows, one per master `id`, including `Topic` and `Probability` |
| `bertopic_topic_info.csv` | 53 topics (ids `-1` through `51`). The `Count` column sums to 1,183 |
| `aidev_pr_task_type_for_perf_prs.parquet` | 1,183 rows, one per master `id`, with `aidev_task_type`, `aidev_task_reason`, and `aidev_task_confidence` |

## LLM analysis artifacts

`schema.json` is the output template. Top-level keys are `pr_id`, `meta`, `quantitative_metrics`, `perf_labels`, `structured_analysis`, `evidence`, and `data_coverage`. Field descriptions in that file are in Chinese. `prompt.md` is the English instruction block placed in front of the PR payload.

Each `per_pr/{id}/` directory contains `{id}_analysis.json`. All 1,183 files have `pr_id` equal to the directory name and `meta.status` equal to the master `status` (694 merged, 489 closed). `{id}_analysis_run.json` stores model, token usage, and cost for that call (1,177 files). One directory also contains a saved workflow prompt, `{id}_workflow_prompt.txt`.

Six root files are few-shot copies, byte-identical to the canonical JSON under `per_pr/`:

`3022909076`, `3074351366`, `3125029980`, `3145702280`, `3194284966`, `3228424652`

Those six ids are `FEWSHOT_PR_IDS` in `run_pr_analysis.py`. They are the six PRs without an `*_analysis_run.json`. The per-PR copies are the records to use.

Of the 1,177 run files, 1,169 record `deepseek-v4-pro` and 8 record `deepseek-v4-flash`. `full_analysis_distilled.csv` has one row per `pr_id`, and its `status` and `agent` values match the master table. `generate_full_analysis.py` rebuilds that CSV and `analysis_viz/FullAnalysis.md`. `generate_rq_analysis.py` rebuilds `analysis_viz/RQ_Analysis.md` and `analysis_viz/rq_analysis_figures/`.

`manual_recheck_sample.md` draws a stratified recheck sample from this terminal set. It states that the six few-shot PRs were left out of the sampling frame (1,177 remaining), with random seed `20260925`, and that the sample has 60 merged and 60 closed PRs. The workbooks are in `humanvalidation/`.

## Usage

Load the full master table and join auxiliary rows on the issue id:

```python
import pandas as pd

master = pd.read_parquet("finaldatabase/pr_master/perf_prs_expanded_final.parquet")
timeline = pd.read_parquet("finaldatabase/auxiliary/pr_timeline.parquet")
timeline = timeline.merge(master, left_on="pr_id", right_on="id", how="left")

paper = pd.read_csv("finaldatabase/paper_source_copy/POP_PULL_Requests_LLM_filtered_final.csv")
paper["merged"] = paper["merged_at"].notna()
```

Review comments join through reviews:

```python
reviews = pd.read_parquet("finaldatabase/auxiliary/pr_reviews.parquet")
comments = pd.read_parquet("finaldatabase/auxiliary/pr_review_comments_v2.parquet")
comments = comments.merge(
    reviews[["id", "pr_id"]],
    left_on="pull_request_review_id",
    right_on="id",
    how="left",
)
```

A single PR's files are under `finaldatabase/per_pr/{id}/`. The flattened analysis table joins to the master table on `pr_id` = `id`.

## Provenance and scripts

How a PR entered the corpus is `detection_source` on the full master table:

| `detection_source` | PRs |
| --- | ---: |
| `llm_title_body_classifier` | 1,125 |
| `aidev_pr_task_type_perf_only` | 58 |

`llm_output` is populated on the 1,125 classifier rows and empty on the other 58. The 58 `aidev_pr_task_type_perf_only` rows all have `aidev_task_type = perf`. Across the full table the task-type label is broader than that inclusion rule:

| `aidev_task_type` | PRs |
| --- | ---: |
| feat | 502 |
| perf | 333 |
| fix | 194 |
| refactor | 54 |
| ci | 29 |
| build | 24 |
| test | 18 |
| docs | 15 |
| chore | 14 |

`Topic` / `Probability` and `classification/bertopic_*` are BERTopic fields stored with the corpus. `aidev_task_*` and `classification/aidev_pr_task_type_for_perf_prs.parquet` are the task-type fields stored with the corpus. Separate licenses for those upstream label sources are not in this repository.

`auxiliary/repository.parquet` holds metadata for the same 418 repositories (`full_name`, `language`, `forks`, `stars`, and the upstream repository `license` string). A license value is present for 395 repositories (12 distinct values). The most frequent primary languages are TypeScript (112), Python (81), and Rust (41). Those strings describe the upstream repositories.

| Script | What it does |
| --- | --- |
| `refresh.py` | Re-checks open PR status on the GitHub API, drops PRs that return 404, and updates the master table, auxiliary tables, and `summary/` |
| `freeze_terminal_dataset.py` | Removes PRs whose derived `status` is `open`, leaving merged and closed rows. Merge rate is merged / n |
| `run_pr_analysis.py` | Builds a prompt from `prompt.md`, `schema.json`, the master row, and `per_pr/{id}/`, calls the DeepSeek chat API, and writes `{id}_analysis.json` |
| `generate_full_analysis.py` | Writes `analysis_viz/FullAnalysis.md` and `full_analysis_distilled.csv` |
| `generate_rq_analysis.py` | Writes `analysis_viz/RQ_Analysis.md`, `rq_analysis_metrics.json`, and `rq_analysis_figures/` |
| `analysis_viz/*.py` | Renders the corpus and paper figures from the distilled table |

`refresh.py` reads a GitHub token from `GITHUB_TOKEN` or `GH_TOKEN`, or from a local token file. That local file is gitignored and is used by the refresh script. `run_pr_analysis.py` expects a DeepSeek API key in the environment. Re-running the refresh, freeze, or analysis scripts rewrites files under `finaldatabase/`. The checked-in tree is the frozen terminal snapshot described above.

`summary/coverage_stats.json` was generated at 2026-09-22T04:12:41Z. `summary/terminal_freeze_report.json` records the freeze at 2026-09-21T02:51:36Z. Some `merged_at` and `closed_at` values fall after the `created_at` window, up to 2026-09-01, because status was refreshed before the freeze.

## License and citation

This repository has no license file. No author list, DOI, or arXiv identifier is stored here. Cite the repository until a paper citation is added:

<https://github.com/Yinhaha2/beyond-merge-rate>
