# Forge TerminusDB candidate

The default Forge branch, `forge/12.1-rc-pipe-fix`, starts at upstream
`ca2242e1dbb9cc40946b8b7c457fdb65f550d6e7` and applies the minimal
worker-pipe cleanup patch proposed to DFRNT on September 30, 2026.
The fix commit is `9b75c346bd141af88b4e86484319b1bfded03dea`.
Upstream's `12.1-rc` image reports the internal version `12.0.6`.

Cleanup unregisters an output pipe before closing it, under the same mutex
as the watchdog's health probe and cancellation decision. This prevents
normal cleanup from being mistaken for a client disconnect. This branch
contains no request-tracing instrumentation.

## Build without running tests

```sh
docker build \
  --build-arg SKIP_TESTS=true \
  --build-arg SWIPL_VERSION=10.0.1 \
  --build-arg TERMINUSDB_GIT_HASH=9b75c346bd141af88b4e86484319b1bfded03dea \
  --label org.opencontainers.image.revision=9b75c346bd141af88b4e86484319b1bfded03dea \
  --label org.opencontainers.image.source=https://github.com/Forge-Systems/terminusdb \
  -t forge-terminusdb:12.1-rc-9b75c346 .
```

`SKIP_TESTS=true` skips both Rust and Prolog build-time test stages. Keeping
SWI-Prolog at the source's default 10.0.1 isolates this patch in the A/B
comparison; it is not a remedy for separate SWI-Prolog GC defects.

## Evidence and validation

The original maintainer package includes the failed 324-test replay, HTTP
and worker traces, and two instrumented patched replays with 324 passes each.
Raw traces and request bodies are kept with Forge's private Studio work,
not published in this public fork.

The new clean candidate has been built but has not been tested. Forge's
prepared runner compares immutable image IDs, repeats the historical
324-test queue at the exact Studio source commit, and exercises current
merge/rebase/schema compatibility and HTTP response cancellation. Planned
limits are 1 CPU / 1 GiB and 0.5 CPU / 1 GiB with swap disabled, plus an
unrestricted comparison. Logs, pytest/JUnit output, image identity, Docker
resource configuration, and exit/OOM state are preserved for each run.

Cancellation during a backpressured final flush still requires validation.
Passing finite replays does not establish that every possible race is absent.
GitHub Actions are disabled until Forge explicitly enables them; no tests
run on push. No Studio deployment is changed by this fork.
