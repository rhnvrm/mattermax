# Combined performance contribution verification

This checkpoint combines PRs #39 through #47 over the original 38 Mattermax
patches. All 47 patches apply cleanly to the pinned v12.0.2 source. The original
38 patch files remain byte-identical, retaining their author credits, including
Karan Sharma's (@mr-karan) feature work.

The first six changes overlap in imports and appended tests. Integration
retains both test/benchmark sets and deduplicates imports. The three newer
changes cherry-pick without conflicts. Store layers and mocks regenerate
without changes. A declaration check compares the union of the independent
changes with the integrated source: 2,769 non-import Go declarations across
25 files match, including unchanged declarations. Compilation and tests
separately check imports and types. Integration adds no production behavior
beyond the submitted contributions.

[report.json](report.json) records source/tree hashes, exact test selectors,
counts, build checks and smoke outcomes. Selected race suites passed
727 parent/subtest checks with no failures or skips
(sqlstore: 332, app: 207, api4: 158, localcachelayer: 30). Coverage includes
post/thread storage, slowmode and archive rules,
preference batches and plugin access, receipt privacy, reaction batching and
cache invalidation, attachment preparation, real previews, retries, sanitization,
file-policy filtering, image syntax/proxies and permalink handling.

The public backend command builds with an explicit integration build number,
source hash and BuildEnterpriseReady=false. Build info contains no enterprise
or sourceavailable tag, and symbols contain no server/v8/enterprise path.
The installed Nix SDK omits nm; the matching SDK's standard cmd/nm executable
performs the same symbol check. The release script's criteria are unchanged.
No enterprise code or new third-party dependency was used.

A loopback-only backend instance loaded the settings overlay, migrated a new
temporary PostgreSQL database, and returned HTTP 200 for system ping and the
anonymous client-config endpoint. Ping reported OK; config reported Mattermax
and version 12.0.2. Plugins/jobs were disabled, the environment was allowlisted,
and file storage was temporary. The process, generated database and runtime
directory were cleaned up afterward. Existing databases were untouched.

## Reproduce the selected checks

Use this branch's patches with the upstream disposable PostgreSQL and Redis
test setup. Set TEST_DATABASE_POSTGRESQL_DSN, then:

```sh
just workspace
just check
cd work/src/server
make setup-go-work BUILD_NUMBER=12.0.2-1.integration BUILD_ENTERPRISE=false BUILD_ENTERPRISE_READY=false BUILD_ENTERPRISE_DIR=/nonexistent
```

Run the commands recorded in report.json from work/src/server. go.work must
contain only the local server and public modules. Workspace commit IDs may
differ after git am; compare source_tree_sha to verify the reconstructed code.

The report records the backend build command, revision and binary SHA-256.
It is an unbranded backend verification build. Apply the tag and symbol criteria
from scripts/build-release.sh. If the SDK omits nm, build its standard tool with
`GOWORK=off go build -o /tmp/mattermax-go-tools/nm cmd/nm` and use that executable.
Failure to run or pass either check leaves enterprise exclusion unverified.

This branch verifies the submitted contributions and adds no upstream PR. The
full branded frontend and packaging workflow remains untested. Selected suites
do not establish complete API equivalence; startup probes do not establish
chat capacity or production readiness. Standalone benchmark reports retain
their original source and scope. Reproduce them on their recorded PR branches;
their ratios are not combined-stack throughput measurements.
