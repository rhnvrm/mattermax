# Combined performance contribution verification

This checkpoint combines PRs #39 through #44 over the original 38 Mattermax
patches. All 44 patches apply cleanly to the pinned v12.0.2 source. The original
38 patch files remain byte-identical, retaining their author credits, including
Karan Sharma's (@mr-karan) feature work.

The independent changes overlap in imports and appended tests. Integration
retains both test/benchmark sets and deduplicates imports. Store layers and
mocks regenerate without additional changes. A declaration check compares the
union of the independent changes with the integrated source: 2,710 non-import
Go declarations across 17 files match, including unchanged declarations. The
combined build/tests separately check imports and types. No new production
behavior is introduced by this integration.

[report.json](report.json) records source/tree hashes, exact test selectors,
counts, build checks and smoke outcomes. The selected store, app and API race
runs passed 301, 110 and 103 checks respectively, with no failures or skips.
Counts include parent and subtest events and exclude package summaries.
Coverage includes post/thread storage, slowmode, archive rules, preference
batches, receipt privacy, attachment preparation, real preview generation,
retry behavior, metadata sanitization and attachment-policy filtering.

The public backend command builds with an explicit integration build number,
source hash and BuildEnterpriseReady=false. Build info contains no enterprise
or sourceavailable tag, and its symbols contain no server/v8/enterprise path.
The installed Nix SDK omits the nm executable; the matching Go 1.26.10 cmd/nm
was built from that SDK and used for the same symbol check. No enterprise code
or new third-party dependency was used.

A loopback-only backend instance loaded the repository settings overlay, ran
its migrations in a new temporary PostgreSQL database, and returned HTTP 200
for /api/v4/system/ping and the anonymous client-config endpoint. Ping reported
OK; client config reported Mattermax and version 12.0.2. The instance used an
allowlisted environment, disabled plugins and job scheduling, and temporary
file storage. The server, generated database and temporary runtime directory
were cleaned up afterward. Existing databases were untouched.

## Reproduce the selected checks

Use this integration branch's patches and the upstream disposable PostgreSQL
and Redis test setup. Set TEST_DATABASE_POSTGRESQL_DSN, then:

```sh
just workspace
just check
cd work/src/server
make setup-go-work BUILD_ENTERPRISE=false BUILD_ENTERPRISE_READY=false BUILD_ENTERPRISE_DIR=/nonexistent
```

Run the exact commands in report.json from work/src/server. They use only the
local public module in go.work. The report also records the binary's source
revision and SHA-256; it is an unbranded backend verification build, rather than
a release package. A direct equivalent build uses:

```sh
integration_source_sha="$(git rev-parse HEAD)"
go build -p 2 -ldflags "-X github.com/mattermost/mattermost/server/public/model.BuildNumber=12.0.2-1.integration -X github.com/mattermost/mattermost/server/public/model.BuildHash=$integration_source_sha -X github.com/mattermost/mattermost/server/public/model.BuildEnterpriseReady=false" -o /tmp/mattermax-integrated-server ./cmd/mattermost
go version -m /tmp/mattermax-integrated-server
```

Apply the tag and symbol criteria from scripts/build-release.sh. If the SDK
omits nm, build its standard tool with
`GOWORK=off go build -o /tmp/mattermax-go-tools/nm cmd/nm` and use that
executable. Failure to run or pass either check leaves enterprise exclusion
unverified.

This is a verification branch for the submitted contributions. It does not add
another upstream PR. The full branded frontend and packaging workflow remains
untested. Selected suites do not establish complete API equivalence, and the
startup probes do not establish chat capacity or production readiness. Each
standalone benchmark retains its recorded source and scope; those results are
not combined-stack throughput measurements.
