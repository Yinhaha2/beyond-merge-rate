# finaldatabase

Terminal snapshot of the performance-PR corpus. Layout, identifiers, and caveats are in the [repository README](../README.md).

| Item | Value |
| --- | --- |
| Pull requests | 1,183 |
| Status | 694 merged, 489 closed |
| Merge rate | 58.7% (694 / 1,183) |
| Open pull requests in this snapshot | 0 (36 removed on 2026-09-21) |

`pr_master/` matches these status counts, as does the top-level `status_counts` in `summary/coverage_stats.json`. `summary/terminal_freeze_report.json` lists the open pull requests that were removed. It is a removal log, not a second corpus.

`auxiliary/pr_timeline.parquet` has 14,915 rows. `coverage_stats.json` still records 13,491 timeline rows, from before the backfill in `summary/timeline_end_actor_fill.json` (1,424 events). Other auxiliary row counts in `coverage_stats.json` match the Parquet files.
