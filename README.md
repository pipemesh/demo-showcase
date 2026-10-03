# demo-showcase

One [Pipemesh](https://pipemesh.io) pipeline touring the feature
surface — fake project, real runs on the production instance. Every
node exists to demonstrate something:

| Node | Feature |
| --- | --- |
| `ci` | `kind: workflow` ([`.pipemesh/ci.yaml`](.pipemesh/ci.yaml)): the whole verify DAG runs as a child workflow per revision, one fused verdict; it reads what its jobs check out |
| `bundle` | `kind: build` with `checkout: [app]` (a revert reuses the bundle), dependency `cache:` (`${checksum:}` keying), a `produces:` entry, `timeout_seconds` |
| `docs` | `allow_failure:` — a red docs check never blocks shipping |
| `regional_test[region=us\|eu]` | Matrix `as: jobs` — sibling lanes, each consuming the bundle and testing it when it is new to the lane (`checkout: false`: the bundle is its only input) |
| `deploy[region=us\|eu]` | `kind: deploy`; lane-to-lane `needs:` via `${{ matrix.region }}`, ships only a bundle new to its region, plus a local typed component ([`announce`](.pipemesh/components/announce.yaml)) invoked with `setup:`/`with:` — the param value itself carries a matrix reference |
| `on_pull_request` | The same `ci.yaml` runs per PR; statuses land on the PR as `pipemesh/checks/*` |
| `on_schedule` | Weekly workflow ([`.pipemesh/health.yaml`](.pipemesh/health.yaml)), Mondays 08:00 UTC |

Implicit machinery on display without any keywords: release-flow cursors
per job, and carry-forward of the last built bundle when `bundle` skips
on a docs-only revision (so the regional lanes, seeing the same bundle,
skip too).

Related demos: [`demo-matrix`](https://github.com/pipemesh/demo-matrix)
(matrix in depth, both modes, break/heal lane demos); `demo-private`
(private on purpose — the regression surface for authenticated paths).
