# R&D: reaction emoji-name capacity

This candidate is held from upstream. It reduces allocations, but the longer
60-post history run has a 4.7% slower median and is slower in all three pairs.
The patch and measurements are retained on this fork for investigation. They
are not a recommendation to merge this change.

Post preparation gathers emoji names from message text, attachments and
interactive blocks, then appends names from reactions. Reserving room for the
known reaction count avoids repeated slice growth. Extraction, sorting,
duplicate removal and feature-flag behavior stay unchanged.

This contribution is independent of the other performance PRs. It starts from
the original 38-patch Mattermax baseline and changes two production lines:
a standard-library import and a capacity reservation. All inherited patches
remain byte-identical, preserving their features and authorship. No public API,
schema, configuration or third-party dependency changes.

## Measurements

Three alternating-order pairs of processes per variant use Go 1.26.10,
GOMAXPROCS=4 and disposable local PostgreSQL/Redis services. A Go overlay restores
only post_metadata.go for baseline. Both variants use identical fixtures,
validation and fixed iteration counts. [results.json](results.json) records
every sample, median, range, source hash and command.

The focused benchmark uses 100,000 iterations per case and covers zero, one,
three, twelve and sixty reactions, with and without emoji in the message.
Repeated reaction names exercise sorting and duplicate removal. Output and
reaction-name preservation are checked before and after the timed loop.

The App benchmark uses 1,000 iterations per case. It prepares complete stored,
ordered pages with three reactions per post, warm reaction caches and priority
reads explicitly enabled. Each timed iteration checks page order and reaction
metadata. Database fixture creation and cache warming are outside the timer.
There are no files or remote links.

| Case | Before B/op | After B/op | Before allocs/op | After allocs/op |
| --- | ---: | ---: | ---: | ---: |
| History/posts=1 | 17224 | 17182 | 311 | 309 |
| History/posts=60 | 186812 | 182969 | 4111 | 3991 |
| Names/text=false/reactions=0 | 16 | 16 | 1 | 1 |
| Names/text=false/reactions=1 | 32 | 32 | 2 | 2 |
| Names/text=false/reactions=12 | 512 | 208 | 6 | 2 |
| Names/text=false/reactions=3 | 128 | 64 | 4 | 2 |
| Names/text=false/reactions=60 | 2177 | 1040 | 8 | 2 |
| Names/text=true/reactions=0 | 240 | 240 | 5 | 5 |
| Names/text=true/reactions=1 | 304 | 304 | 6 | 6 |
| Names/text=true/reactions=12 | 688 | 464 | 8 | 6 |
| Names/text=true/reactions=3 | 432 | 320 | 7 | 6 |
| Names/text=true/reactions=60 | 2353 | 1264 | 10 | 6 |

The 60-post page uses 120 fewer allocations in every trial. Median
allocated bytes are 186812 before and 182969
after. The allocation reduction is consistent; the history latency result is worse.
The initial 1,000-iteration 60-post median is 1.103470 ms before and 1.128728 ms
after, about 2.3% slower, with non-overlapping ranges. Its single-post control
has a 13.5% slower median with overlapping ranges.

A follow-up uses the same source and fixtures with 5,000 iterations per case
in three more alternating-order pairs. [history-followup.json](history-followup.json)
retains every raw sample and command. The 60-post median is 0.930569 ms before
and 0.973968 ms after, about 4.7% slower. Before ranges from 0.893170 to
0.933276 ms; after ranges from 0.930197 to 0.974372 ms. Ranges overlap slightly,
but every paired process is slower after the change, including the reversed
order pair. Allocation counts remain 4,111 before and 3,991 after. The longer
single-post median improves about 1.5%, with overlapping ranges.

The focused three-reaction helper improves from 162.6 to 117.5 ns/op and from
four to two allocations. That does not establish faster history preparation.
The full-page regression remains unresolved, so no upstream PR is submitted.

These measurements cover this standalone change over its recorded baseline.
They do not measure combined-stack throughput, and ratios from separate PRs
must not be multiplied together.

## Validation

Selected emoji, custom-emoji, post preparation and app/API reaction race suites
passed 71 parent/subtest events with no failures or skips. Existing tests cover
message/attachment extraction, duplicates, custom-emoji filtering and omission
of interactive-block emoji when the feature is disabled. All twelve benchmark
cases also pass execution/output checks under the race detector; those timings
are not performance evidence.

All 39 patches apply cleanly. The public backend builds with only the production
tag and passes the release script's enterprise tag/symbol exclusion criteria.
[validation.json](validation.json) records the exact source, commands and check
scopes. Full branded packaging, browser checks and production load tests were
not run for this contribution.

## Reproduce

Use Go 1.26.10 and the normal public server/public go.work. Set
TEST_DATABASE_POSTGRESQL_DSN and TEST_REDIS_ADDR to disposable local test services.
After just workspace, run from this repository's root:

```sh
GOMAXPROCS=4 python3 docs/benchmarks/reaction-emoji-name-capacity/benchmark.py --workspace work/src --baseline HEAD^ --trials 3 --name-iterations 100000 --history-iterations 1000 --output /tmp/reaction-emoji-name-capacity.json
```

Reproduce the longer follow-up separately:

```sh
GOMAXPROCS=4 python3 docs/benchmarks/reaction-emoji-name-capacity/benchmark.py --workspace work/src --baseline HEAD^ --benchmarks history --trials 3 --history-iterations 5000 --output /tmp/reaction-emoji-history-followup/results.json
```

Run the race command in validation.json from work/src/server. Compare the source
tree hash when reconstructing the workspace, since git-am commit IDs can differ.
