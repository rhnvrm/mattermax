# Combined performance contribution verification

This checkpoint combines PRs #39 through #53 over the original 38 Mattermax
patches. All 53 patches apply cleanly to pinned v12.0.2. Existing patches retain
their authorship, including Karan Sharma's (@mr-karan) feature work. This
fork-only branch adds no upstream PR.

PR #53 integrates without conflicts. All 52 earlier patch files remain
byte-identical. Its new test file matches the standalone contribution exactly.
The new patch differs only in Git index hashes and hunk line positions; removing
those format fields yields a byte-identical patch. No extra production edits
were needed. The held emoji-name capacity experiment is excluded.

The declaration comparison matches 2,995 non-import Go declarations across
39 files. Its negative control rejects an added declaration in a temporary
copy. [report.json](report.json) records source/tree hashes, commands, test
scopes, build checks and startup probes.

Fresh selected race suites passed 623 parent/subtest events with no
failures or skips (app: 318, api4: 109, localcachelayer: 37, sqlstore: 159).
They cover post preparation, link/image filtering, read receipts, attachment
ordering and access policies, reactions/cache invalidation, priority and
acknowledgement reads, and API history retrieval. Thread-unread tests cover
empty/duplicate team IDs, multiple teams, null and negative values, deleted/
unfollowed threads, lost membership, DM exceptions and zero-valued groups.
Preview-filter compatibility checks remain included. Autolink tests compare
returned links and existing malformed-input panic outcomes; the shared Markdown
parser remains unchanged.

Earlier broad SQL/cache checks remain available at immutable historical report
links and keep their original revisions. PR #53 changes thread-unread aggregation,
so the complete ThreadStore suite and its new edge cases are run freshly
alongside selected cache/priority checks. Historical counts are not added to
this run.

All 92 benchmark cases pass single-iteration execution/output checks. These
timings support no performance claim. Fixtures include and validate
PostList.Order. Standalone paired timing reports remain scoped to their
recorded PR sources; they are not combined-stack throughput measurements.

The current public backend builds with the production tag, an explicit
integration build number, source hash and BuildEnterpriseReady=false. Build
info has no enterprise/sourceavailable tags; symbols contain no
server/v8/enterprise path. The matching SDK's standard cmd/nm runs the same
exclusion criteria as the unchanged release script.

A loopback-only instance loads the settings overlay and migrates a fresh
PostgreSQL database. Ping and anonymous client config return HTTP 200,
reporting Mattermax 12.0.2 and the expected integration build number. Jobs and
plugins are disabled, the environment is allowlisted and storage is temporary.
The server, generated database and runtime directory are removed afterward.
Existing databases are untouched.

## Retained release packaging

Full branded packaging and packaged-asset HTTP smoke passed at the earlier
ten-contribution source d4194a4b, with root b99123a6. Those archive and executable
hashes remain scoped to that source. Its archive contains all 48 patches from
that checkpoint and excludes server/enterprise and Git metadata. Both packaged
executables pass the release script's tag/symbol checks; translation and overlay
checks pass. Root HTML and its initial JavaScript assets return HTTP 200.

Full packaging has not been rerun with PRs #49 through #53. The webapp,
branding, overlay and release scripts are unchanged; the current backend is
built and checked separately. No release tag, deployment or published release
is created.

The earlier frontend build reports 77 webpack warnings, including three missing
internal_serializeStyles exports from @mui/styled-engine, plus Sass, CSS ordering
and asset-size warnings. Redux postbuild reports a missing index.d.ts copy while
exiting zero. HTTP delivery does not establish JavaScript execution. Browser
flows and the MUI warnings remain outside this verification.

## Reproduce

Use Go 1.26.10 and disposable PostgreSQL/Redis services. Set
TEST_DATABASE_POSTGRESQL_DSN and TEST_REDIS_ADDR, then start from this branch:

```sh
just workspace
just check
cd work/src/server
make setup-go-work BUILD_NUMBER=12.0.2-1.integration-fifteen BUILD_ENTERPRISE=false BUILD_ENTERPRISE_READY=false BUILD_ENTERPRISE_DIR=/nonexistent
```

Run the fresh race, benchmark-smoke and backend build commands in report.json
from work/src/server. Use the recorded environment flags and only the server
and public modules in go.work. Git-am commit IDs can differ; compare
source_tree_sha and file hashes when reconstructing the code.

Apply scripts/build-release.sh's tag and symbol criteria. If nm is absent,
build the matching standard SDK tool with
`GOWORK=off go build -o /tmp/mattermax-go-tools/nm cmd/nm`. A failure to run or
pass either exclusion check leaves enterprise exclusion unverified.

For full packaging, use Node ^24, npm ^11 and a working go tool nm, then run
just build with a non-dev integration release name. The retained packaging
run used Node 24.11.1 and npm 11.6.2.

Selected suites do not establish complete API equivalence, and startup probes
do not establish production capacity. Reproduce standalone benchmarks on their
recorded PR branches; do not multiply their ratios together.
