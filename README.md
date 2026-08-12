# demo-showcase

One [PipeMesh](https://pipemesh.dev) pipeline touring the feature
surface — fake project, real runs on the production instance. Every
node exists to demonstrate something:

| Node | Feature |
| --- | --- |
| `ci` | Trigger workflow node ([`.pipemesh/ci.yaml`](.pipemesh/ci.yaml)): the whole verify DAG runs as a child workflow per revision, one fused verdict |
| `bundle` | Changed-path `rules:`, dependency `cache:` (`${checksum:}` keying), `artifacts:` production, `timeout_seconds` |
| `docs` | `allow_failure:` — a red docs check never blocks shipping |
| `regional_test[region=us\|eu]` | Matrix `as: jobs` — sibling lanes, each inheriting the bundle artifact along its own edge |
| `deploy[region=us\|eu]` | Lane-to-lane `needs:` via `${{ matrix.region }}`, plus a local typed component ([`announce`](.pipemesh/components/announce.yaml)) invoked with `setup:`/`with:` — the param value itself carries a matrix reference |
| `signoff` | DESIGN-V32 `approval:` gate — the run executes, then **parks** for a human decision, and the parked lane freezes (newer revisions queue behind it). Approve it in the UI to promote |
| `on_pull_request` | The same `ci.yaml` runs per PR; statuses land on the PR as `pipemesh/checks/*` |
| `on_schedule` | Weekly workflow ([`.pipemesh/health.yaml`](.pipemesh/health.yaml)), Mondays 08:00 UTC |

Implicit machinery on display without any keywords: stage gates
(`bundle` waits for the `verify` stage), release-flow cursors per job,
carry-forward of the last built artifact when `bundle` skips on a
docs-only revision.

Related demos: [`demo-matrix`](https://github.com/pipemesh/demo-matrix)
(matrix in depth, both modes, break/heal lane demos); `demo-private`
(private on purpose — the regression surface for authenticated paths).

_Trigger provenance demo: this PR exists to exercise the pull_request source._
