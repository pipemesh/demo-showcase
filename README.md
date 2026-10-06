# demo-showcase

One [Pipemesh](https://pipemesh.io) pipeline touring the feature
surface — fake project, real runs on the production instance. Every
node exists to demonstrate something:

| Node | Feature |
| --- | --- |
| `ci` | `job_type: workflow` ([`.pipemesh/ci.yaml`](.pipemesh/ci.yaml)): the whole verify DAG runs as a child workflow per revision, one fused verdict; it reads what its jobs check out |
| `bundle` | `job_type: build` with `checkout: [app]` (a revert reuses the bundle), dependency `cache:` (`${checksum:}` keying), a `produces:` entry, `timeout_seconds` |
| `regional_test[region=us\|eu]` | Matrix `as: jobs` — sibling lanes, each consuming the bundle and testing it when it is new to the lane (`checkout: false`: the bundle is its only input) |
| `deploy_staging[region=us\|eu]` | `job_type: deploy` with `production: false` — each region stages only what its own test lane passed and smoke-tests it; a failing lane stops there while the other ships |
| `deploy[region=us\|eu]` | `job_type: deploy` with `production: true`; lane-to-lane `needs:` via `${{ matrix.region }}` (each region ships only what its staging passed), ships only a bundle new to its region, plus a local typed component ([`announce`](.pipemesh/components/announce.yaml)) invoked with `setup:`/`with:` — the param value itself carries a matrix reference |
| `on_pull_request` | The same `ci.yaml` runs per PR; statuses land on the PR as `pipemesh/checks/*` |
| `on_schedule` | Weekly workflow ([`.pipemesh/health.yaml`](.pipemesh/health.yaml)), Mondays 08:00 UTC |

Every job runs on `linux-arm64-micro` — a quarter of a core and 512 MiB —
since all they do is echo and grep.

Implicit machinery on display without any keywords: release-flow cursors
per job, and carry-forward of the last built bundle when `bundle` skips
on a README-only revision (so the regional lanes, seeing the same bundle,
skip too).

## The showcase keeps moving

`main` is driven, not edited: a driver replays the ten commits of the
[`showcase-series`](../../commits/showcase-series) branch onto `main`,
one every 30 seconds, and starts over after the tenth. Revisions arrive
faster than the pipeline settles them, so every job's cursor jumps to
the newest revision it can take. One commit in the series drops the eu
greeting: `deploy_staging[region=eu]` fails its smoke check while us
ships on, and the next commit's revision supersedes the failed one.
Each push carries the cycle number in `app/cycle.txt`, so a new cycle
builds everything again; within a cycle, the fix after the failure puts
back content built before, and its bundle is reused.

Changes to the showcase go to `showcase-series` (rebuild the series),
never to `main` directly.

Related demos: [`demo-matrix`](https://github.com/pipemesh/demo-matrix)
(matrix in depth, both modes, break/heal lane demos); `demo-private`
(private on purpose — the regression surface for authenticated paths).

The greetings in `app/main.txt` are what each staging lane smoke-tests.
