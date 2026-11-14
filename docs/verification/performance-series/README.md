# Combined performance contribution verification

This checkpoint combines PRs #39 through #57 over the original 38 Mattermax
patches. All 57 patches apply cleanly to pinned v12.0.2. Existing patches retain
their authorship, including Karan Sharma's (@mr-karan) feature work. This
fork-only branch adds no upstream PR.

PR #57 integrates without conflicts. All 56 earlier patches remain
byte-identical. Its two new test/benchmark files match the standalone
contribution. The patch differs only in Git index hashes and hunk line
positions. No extra production edits were needed. Store interfaces, generated
adapters and mocks are unchanged since the eighteen-contribution checkpoint.

The declaration union matches 3,191 non-import Go declarations across 50 files,
covering 137 contributed declaration changes. A temporary-copy negative control
rejects an extra declaration. [report.json](report.json) records source/tree
hashes, commands, checks and scopes. Both held emoji-name-capacity and
partial-index experiments remain excluded.

Fresh selected race suites under TZ=UTC passed 1,050 parent/subtest events with
no failures or skips (app: 363, api4: 135, localcachelayer: 211, sqlstore: 341).
They cover post preparation, link/image filtering, read receipts, attachment
ordering and access policies, reactions/cache invalidation, priority and
acknowledgement reads, thread flags/counts and history APIs. The emoji-name
oracle verifies exact order, duplicates and nilness against the legacy regex,
including malformed/long inputs, exhaustive short combinations and generated
strings. Normal fuzz seeds also pass under the race detector.

The complete PostStore suite runs through SQL and cache layers. Reply-count
checks include missing/duplicate roots, null/deleted values, root-only input and
forced chunks. ThreadStore and staged root-index compatibility checks cover
boundaries and empty roots. Concurrent migration downgrade/upgrade checks verify
that the original index is restored before the replacement is removed, and the
replacement exists before the original is dropped. Historical broad results
retain their original revisions and schema; their counts are not added here.

All 199 benchmark cases pass single-iteration execution/output checks. The case
set is the exact union of the prior 182 and the new 17. These timings support no
performance claim. The mixed case warms/checks all five fixture types, while its
single timed iteration covers one type. Standalone reports retain their exact
code and measurement scopes, including PR #55's write/storage costs and PR #56's
mixed full-save/control results.

PR #57's HTTP comparison and earlier app-only regressions remain scoped to its
standalone source. Those HTTP requests were not replayed against this combined
stack. Ratios must not be multiplied into a combined throughput estimate.
Standalone runners intentionally require their original parent tree; use the
contribution's branch for those measurements.

The current public backend builds with the production tag, explicit build
number/source hash and BuildEnterpriseReady=false. Build info excludes
enterprise/sourceavailable tags, and symbols exclude server/v8/enterprise paths
using the unchanged release script's criteria and matching SDK nm tool.

A loopback-only instance loads the settings overlay and migrates a fresh
PostgreSQL database, including slowmode and staged root-index migrations. Ping
and anonymous client config return HTTP 200 with Mattermax 12.0.2 and the
expected integration build number. Jobs/plugins are disabled and environment
variables are allowlisted. The server, database and runtime directory are
removed.

## Retained release packaging

Full branded packaging and packaged-asset HTTP smoke passed at the earlier ten-
contribution source d4194a4b, with root b99123a6. Those archive and executable
hashes remain scoped to that source. Its archive contains all 48 patches from
that checkpoint and excludes server/enterprise and Git metadata. Both packaged
executables pass the release script's tag/symbol checks; translation and overlay
checks pass. Root HTML and its initial JavaScript assets return HTTP 200.

Full packaging has not been rerun with PRs #49 through #57. The webapp,
branding, overlay and release scripts are unchanged; the current backend is
built and checked separately. No release tag, deployment or published release is
created.

The earlier frontend build reports 77 webpack warnings, including three missing
internal_serializeStyles exports from @mui/styled-engine, plus Sass, CSS
ordering and asset-size warnings. Redux postbuild reports a missing index.d.ts
copy while exiting zero. HTTP delivery does not establish JavaScript execution.
Browser flows and the MUI warnings remain outside this verification.

## Reproduce

Use Go 1.26.10 and disposable PostgreSQL/Redis services. Set
TEST_DATABASE_POSTGRESQL_DSN and TEST_REDIS_ADDR. Set TZ=UTC for race suites to
match the upstream date-test assumption, then start from this branch:

```sh
just workspace
just check
cd work/src/server
make setup-go-work BUILD_NUMBER=12.0.2-1.integration-nineteen BUILD_ENTERPRISE=false BUILD_ENTERPRISE_READY=false BUILD_ENTERPRISE_DIR=/nonexistent
```

Run the race, benchmark-smoke and backend build commands in report.json from
work/src/server. Use only server and public in go.work. Git-am commit IDs can
differ; compare source trees and file hashes when reconstructing the workspace.
Apply the unchanged release script's tag/symbol exclusion criteria. Full branded
packaging, browser flows and production load tests have not been run at this
revision. Selected suites do not establish complete HTTP API equivalence.
