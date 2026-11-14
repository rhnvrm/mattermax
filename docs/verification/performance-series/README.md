# Combined performance contribution verification

This checkpoint combines PRs #39 through #48 over the original 38 Mattermax
patches. All 48 patches apply cleanly to pinned v12.0.2. The original 38 patch
files remain byte-identical, retaining author credits, including Karan Sharma's
(@mr-karan) feature work. This fork branch adds no upstream PR.

The initial overlaps in imports and appended tests retain both contributions.
PRs #45 through #48 integrate without conflicts. The declaration comparison
matches 2,906 non-import Go declarations across 29 files, including unchanged
declarations. A negative control rejects an unexpected declaration added to a
temporary copy. Integration introduces no extra production behavior.

[report.json](report.json) records source/tree hashes, exact commands, counts,
build checks and startup probes. Fresh race suites passed
413 parent/subtest events with no failures or skips
(app: 198, api4: 96, sqlstore: 57, localcachelayer: 62). They cover post
preparation, file ordering and policy filtering,
shared metadata isolation, content limits, preview retries, burn-on-read resets,
reactions, receipts, image/proxy handling and permalink previews.

The earlier broad store/cache results are retained separately at their tested
revision. Their production implementation and public/platform dependencies
are byte-identical to this source. Fresh file-info store/cache checks cover the
path used by attachment batching. The report does not relabel retained results
as a new run.

The corrected ordered benchmark fixtures all pass execution/output checks
across 26 cases. Their single-iteration timings support no performance claim.
PR #45 through #47 timing reports were corrected to populate PostList.Order;
the older fixtures skipped priority/acknowledgement reads and are superseded.
Standalone paired measurements remain scoped to their recorded PR branches,
not the combined stack.

The public backend builds with an explicit integration number, source hash and
BuildEnterpriseReady=false. Build info has no enterprise/sourceavailable tags;
symbols contain no server/v8/enterprise path. The installed SDK omits nm, so the
matching SDK's standard cmd/nm performs the release script's unchanged symbol
check. No enterprise code or new third-party dependency was used.

A loopback-only instance loaded the settings overlay and migrated a new
temporary PostgreSQL database. Ping and anonymous client config returned HTTP
200, reporting OK and Mattermax 12.0.2. Plugins/jobs were disabled; environment
variables were allowlisted and storage was temporary. The process, generated
database and runtime directory were removed afterward. Existing databases were
untouched.

## Reproduce

Use the upstream disposable PostgreSQL/Redis test setup and set
TEST_DATABASE_POSTGRESQL_DSN. Start from a fresh checkout of this branch:

```sh
just workspace
just check
cd work/src/server
make setup-go-work BUILD_NUMBER=12.0.2-1.integration BUILD_ENTERPRISE=false BUILD_ENTERPRISE_READY=false BUILD_ENTERPRISE_DIR=/nonexistent
```

Run the fresh test, fixture-smoke and build commands in report.json from
work/src/server, with GOFLAGS empty and only server/public modules in go.work.
Git-am commit IDs can differ; compare source_tree_sha to verify reconstructed
code. The report records the build revision and binary SHA-256.

Apply scripts/build-release.sh's tag and symbol criteria. If nm is absent,
build its standard tool with
`GOWORK=off go build -o /tmp/mattermax-go-tools/nm cmd/nm` and use that executable.
Failure to run or pass either check leaves enterprise exclusion unverified.

The full branded release build also passes using the unchanged `just build`
workflow, Node 24.11.1, npm 11.6.2 and Go 1.26.10. Both packaged executables
pass the script's tag and symbol checks. The branded translation check and
settings overlay pass. The source checksum matches; all 48 archived patches
match this branch, and the source archive contains no server/enterprise or Git
metadata. The report records archive and executable SHA-256 hashes.

The packaged server starts against another disposable database. Ping, client
config, root HTML and its two initial JavaScript assets return HTTP 200. The
client reports Mattermax 12.0.2 and the expected integration build number. The
process, database and temporary asset copy are removed after the check.

To repeat packaging, use Node ^24 and npm ^11, ensure `go tool nm` works, then
run `MATTERMAX_RELEASE=12.0.2-1.integration.b99123a just build` from the root.
The recorded build uses root b99123a6 and source d4194a4b; this documentation
commit follows that build. No release tag or published release is created.

Webpack reports 77 warnings, including three missing `internal_serializeStyles`
exports from `@mui/styled-engine`, Sass deprecations, CSS ordering conflicts
and asset-size warnings. Redux postbuild also reports a missing index.d.ts
copy while exiting zero. HTTP checks establish asset delivery, not JavaScript
execution. The MUI warnings need separate frontend investigation; full browser
flows remain untested.

Selected suites do not establish complete API equivalence, and startup probes
do not establish production capacity. Reproduce standalone benchmarks on their
recorded PR branches; do not combine their speed ratios into a
server-throughput claim.
