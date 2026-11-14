# Combined performance contribution verification

This checkpoint combines PRs #39 through #51 over the original 38 Mattermax
patches. All 51 patches apply cleanly to pinned v12.0.2. The original patches
retain their authorship, including Karan Sharma's (@mr-karan) feature work.
This fork-only branch adds no upstream PR.

PR #51 integrates without conflicts. All 50 earlier patch files remain
byte-identical. The new patch and its four changed source files match the
standalone contribution exactly. No extra production edits were needed.

The declaration comparison matches 2,946 non-import Go declarations across
36 files. Its negative control rejects an unexpected declaration in a
temporary copy. [report.json](report.json) records source/tree hashes,
commands, test scopes, build checks and startup probes.

Fresh app/API/cache/priority-ack SQL race suites passed 501 parent/subtest events
with no failures or skips (app: 283, api4: 107, localcachelayer:
37, sqlstore: 74). They cover reaction decoding and invalidation, actual Redis/LRU
compatibility and isolation, post preparation, receipts, file ordering and
policy filtering, shared metadata isolation, content limits, preview retries,
burn-on-read resets, image/proxy handling and permalink previews. Autolink
checks compare both returned links and existing malformed-input panic outcomes;
the shared Markdown parser remains unchanged.

Earlier file-info SQL/cache checks and broader store/cache runs remain
available in immutable historical report links. Public/platform dependencies
are byte-identical. Reaction caching changed in PR #49 and priority/ack reads
changed in PR #51, so earlier store/cache runs stay scoped to their original
source. Fresh selected suites cover current cache and changed SQL paths,
including batch boundaries, NULL fields and partial-error behavior. Historical
counts are not added to the fresh run.

All 64 benchmark cases pass single-iteration execution/output checks.
These timings support no performance claim. The ordered fixtures include and
validate PostList.Order, including priority and acknowledgement reads.
Standalone paired timing reports remain scoped to their recorded PR sources;
they are not measurements of combined-stack throughput.

The current public backend builds with the production tag, an explicit
integration build number, source hash and BuildEnterpriseReady=false. Build
info has no enterprise/sourceavailable tags; symbols contain no
server/v8/enterprise path. The matching SDK's standard cmd/nm runs the same
exclusion gates as the unchanged release script.

A loopback-only instance loads the settings overlay and migrates a fresh
PostgreSQL database. Ping and anonymous client config return HTTP 200,
reporting Mattermax 12.0.2 and the expected integration build number. Jobs and
plugins are disabled, the environment is allowlisted and storage is temporary.
The server, generated database and runtime directory are removed afterward.
Existing databases are untouched.

## Retained release packaging

The full branded release build and packaged-asset HTTP smoke passed at the
previous ten-contribution source d4194a4b, with root b99123a6. Those archive and
executable hashes stay scoped to that source. The source archive contains all
48 patches from that checkpoint and excludes server/enterprise and Git
metadata. Both packaged executables pass the release script's tag/symbol
checks; the translation and overlay checks pass. Root HTML and its initial
JavaScript assets return HTTP 200 from the packaged server.

Packaging has not been rerun with PRs #49 through #51. The webapp, branding,
overlay and release scripts are unchanged; the new backend is built and checked separately.
No release tag, deployment or published release is created.

The earlier frontend build reports 77 webpack warnings, including three
missing internal_serializeStyles exports from @mui/styled-engine, plus Sass,
CSS ordering and asset-size warnings. Redux postbuild reports a missing
index.d.ts copy while exiting zero. HTTP asset delivery does not establish
JavaScript execution. MUI warnings and browser flows need separate frontend
investigation.

## Reproduce

Use Go 1.26.10 and disposable PostgreSQL/Redis test services. Set
TEST_DATABASE_POSTGRESQL_DSN and TEST_REDIS_ADDR, then start from this branch:

```sh
just workspace
just check
cd work/src/server
make setup-go-work BUILD_NUMBER=12.0.2-1.integration-thirteen BUILD_ENTERPRISE=false BUILD_ENTERPRISE_READY=false BUILD_ENTERPRISE_DIR=/nonexistent
```

Run the fresh race, benchmark-smoke and backend build commands in report.json
from work/src/server. Use the recorded environment flags and only the server
and public modules in go.work. Git-am commit IDs can differ; compare
source_tree_sha and the recorded file hashes when reconstructing the code.

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
