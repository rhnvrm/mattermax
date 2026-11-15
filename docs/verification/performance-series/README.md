# Combined performance contribution verification

This checkpoint combines PRs #39 through #58 over the original 38 Mattermax
patches. All 58 patches apply cleanly to pinned v12.0.2. Existing patches retain
their authorship, including Karan Sharma's (@mr-karan) feature work. This
fork-only branch adds no upstream PR.

PR #58 integrates without conflicts. All four changed source files and its
exported patch match the standalone contribution byte-for-byte. All 57 earlier
patch files remain byte-identical. No extra production edits were needed.
Store interfaces, generated adapters and mocks are unchanged since the previous
checkpoint.

The declaration union matches 3,323 non-import Go declarations across 54 files,
covering 156 contributed declaration changes. A temporary-copy negative control
rejects an extra declaration. [report.json](report.json) records source/tree
hashes, commands, checks and scopes. Both held emoji-name-capacity and
partial-index experiments remain excluded.

Fresh selected race suites under TZ=UTC passed 1,266 parent/subtest events
with no failures or skips (model: 41, app: 489, api4: 184, localcachelayer: 211, sqlstore: 341).
They cover post preparation, creation, updates and history; link/image filtering;
read receipts; attachment ordering and access policies; reactions and cache
invalidation; priority and acknowledgement reads; and thread flags/counts.
The new model tests compare attachment output against the legacy JSON decoder,
including generated inputs, ownership, numeric/Unicode boundaries, custom
marshaling and fuzz seeds.

An initial overlapping run failed: the 1,000-concurrent-update app test hit
PostgreSQL's 100-connection ceiling while a cache package was running, and the
app package later reached its 20-minute timeout. The failed attempt and database
log hashes are retained in report.json. That concurrent test passed alone on
both the original 38-patch baseline and the combined source. The full selected
app suite then passed alone with a 30-minute limit. The accepted count uses
that app rerun plus the four successful packages from the initial run, without
adding duplicate control passes. No production or database settings changed.

The complete PostStore and ThreadStore suites run through SQL and cache layers.
Staged root-index migration checks verify the index ordering during both upgrade
and downgrade. Historical broad results keep their original revisions and schema;
their counts are not added here. Public model code now includes PR #58 and has
fresh selected race coverage. Platform and module files remain unchanged.

All 213 benchmark cases pass single-iteration execution/output checks. The case
set is exactly the union of the prior 199 with nine model accessor and five
stored-history cases. These timings support no performance claim. The mixed
emoji-history case warms/checks all five fixture types but times one type at 1x.

Standalone HTTP and component measurements retain their exact source scopes,
including PR #58's slower action fallback and PR #57's earlier app-only
regressions. HTTP comparisons were not replayed against this combined stack.
Ratios must not be multiplied into a combined throughput estimate. Standalone
runners require their original parent tree; use each contribution's branch.

The current public backend builds with the production tag, explicit build
number/source hash and BuildEnterpriseReady=false. Build info excludes
enterprise/sourceavailable tags, and symbols exclude server/v8/enterprise paths
using the unchanged release script's criteria and matching SDK nm tool.

A loopback-only instance loads the overlay and migrates a fresh PostgreSQL
database, including slowmode and staged root-index migrations. Ping and
anonymous client config return HTTP 200 with Mattermax 12.0.2 and the expected
integration build number. Jobs/plugins are disabled and environment variables
are allowlisted. The server, database and runtime directory are removed.

## Current bottleneck profiles

[Profiles](profiles/report.json) record fresh CPU and allocation samples for
plain, warm-reaction and 64-field attachment history on the combined source.
They include fixture/test startup and select bottlenecks; their timings do not
measure a before/after improvement.

The attachment fixture still spends most preparation CPU beneath Attachments.
Field validation/copying and repeated AllStrings collection remain substantial;
InteractiveBlocksImageURLs also decodes the full attachment to collect its URL
fields. The next experiment should measure whether that work can be reduced
while preserving unsupported/custom-value fallback behavior and object ownership.
Plain history spends most preparation CPU in the two batched priority and
acknowledgement reads. Warm-reaction history still includes cache decoding and
those SQL reads. These profiles do not justify multiplying standalone gains.

## Retained release packaging

Full branded packaging and packaged-asset HTTP smoke passed at the earlier ten-
contribution source d4194a4b, with root b99123a6. Those archive and executable
hashes remain scoped to that source. Its archive contains all 48 patches from
that checkpoint and excludes server/enterprise and Git metadata. Both packaged
executables pass the release script's tag/symbol checks; translation and overlay
checks pass. Root HTML and its initial JavaScript assets return HTTP 200.

Full packaging has not been rerun with PRs #49 through #58. The webapp,
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
TEST_DATABASE_POSTGRESQL_DSN and TEST_REDIS_ADDR. Set TZ=UTC and create the
unbranded workspace with just workspace, then run just check. Use only server
and public in go.work. Run the race, benchmark and public backend build commands
recorded in report.json from work/src/server. Run the app race suite separately
from the store suites when PostgreSQL has only 100 connections.

Git-am commit IDs can differ; compare source trees and file hashes when
reconstructing the workspace. Full branded packaging, browser flows and
production load tests have not been run at this revision. Selected suites do
not establish complete HTTP API equivalence.
