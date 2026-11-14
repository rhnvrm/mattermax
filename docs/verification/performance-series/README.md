# Combined performance contribution verification

This checkpoint combines PRs #39 through #55 over the original 38 Mattermax
patches. All 55 patches apply cleanly to pinned v12.0.2. Existing patches retain
their authorship, including Karan Sharma's (@mr-karan) feature work. This
fork-only branch adds no upstream PR.

PR #55 integrates without conflicts. All 54 earlier patches remain
byte-identical. Its six new source files, including all four migration files,
match the standalone contribution. The new patch differs only in Git index
hashes and hunk line positions. No extra production edits were needed. Store
interfaces, generated adapters and mocks are unchanged since the previous
checkpoint.

The declaration union matches 3,182 non-import Go declarations across 46 files.
A temporary-copy negative control rejects an extra declaration.
[report.json](report.json) records source/tree hashes, commands, checks and
scopes. Both the held emoji-name and partial-index experiments are excluded.

Fresh selected race suites passed 1,017 parent/subtest events with no failures
or skips (app: 332, api4: 135, localcachelayer: 211, sqlstore: 339). They cover
post preparation, link/image filtering, read receipts, attachment ordering and
access policies, reactions/cache invalidation, priority/acknowledgement reads,
thread flags/counts and history APIs. The complete PostStore suite runs through
both SQL and cache layers. ThreadStore and new root-index compatibility checks
include null/deleted/boundary values and empty roots. Staged concurrent
migration downgrade/upgrade checks verify that the original index is restored
before the replacement is removed, and the replacement exists before the
original is dropped.

Historical broad results keep their original revisions and schema. Their counts
are not added to the current run. The new index composes with the slowmode index
and existing thread-count changes without extra code or migration edits.

All 146 benchmark cases pass single-iteration execution/output checks. These
timings support no performance claim. Standalone reports retain their exact
source scopes, including PR #55's write/storage costs and reply-count controls.
They are not combined-stack throughput measurements; their ratios must not be
multiplied together.

The current public backend builds with the production tag, an explicit build
number/source hash and BuildEnterpriseReady=false. Build info excludes
enterprise/sourceavailable tags and symbols exclude server/v8/enterprise paths,
using the unchanged release script's criteria and matching SDK nm tool.

A loopback-only instance loads the settings overlay and migrates a fresh
PostgreSQL database, including both the slowmode and root-index migrations. Ping
and anonymous client config return HTTP 200 with Mattermax 12.0.2 and the
expected integration build number. Jobs/plugins are disabled and the environment
is allowlisted. The server, generated database and runtime directory are
removed.

## Retained release packaging

Full branded packaging and packaged-asset HTTP smoke passed at the earlier ten-
contribution source d4194a4b, with root b99123a6. Those archive and executable
hashes remain scoped to that source. Its archive contains all 48 patches from
that checkpoint and excludes server/enterprise and Git metadata. Both packaged
executables pass the release script's tag/symbol checks; translation and overlay
checks pass. Root HTML and its initial JavaScript assets return HTTP 200.

Full packaging has not been rerun with PRs #49 through #55. The webapp,
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
TEST_DATABASE_POSTGRESQL_DSN and TEST_REDIS_ADDR, then start from this branch:

```sh
just workspace
just check
cd work/src/server
make setup-go-work BUILD_NUMBER=12.0.2-1.integration-seventeen BUILD_ENTERPRISE=false BUILD_ENTERPRISE_READY=false BUILD_ENTERPRISE_DIR=/nonexistent
```

Run the race, benchmark-smoke and backend build commands in report.json from
work/src/server. Use only server and public in go.work. Git-am commit IDs can
differ; compare source trees and file hashes when reconstructing the workspace.
Apply the unchanged release script's tag/symbol exclusion criteria. Full branded
packaging, browser flows and production load tests have not been run at this
revision. Selected suites do not establish complete HTTP API equivalence.
