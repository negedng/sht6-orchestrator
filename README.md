# sht6-orchestrator

Standalone shadow-sync orchestrator for the sht6 simulation. Drives a four-pair sync between three repos:

- [`sht6-main`](https://github.com/negedng/sht6-main) — monorepo
- [`sht6-backend`](https://github.com/negedng/sht6-backend) — backend leaf
- [`sht6-frontend`](https://github.com/negedng/sht6-frontend) — frontend leaf

## Pairs

| Pair | Mono side (`a`) | Leaf side (`b`) |
|---|---|---|
| `backend` | `sht6-main/backend/` | `sht6-backend/` (root) |
| `frontend` | `sht6-main/frontend/` | `sht6-frontend/` (root) |
| `common-backend` | `sht6-main/common/` | `sht6-backend/src/common/` |
| `common-frontend` | `sht6-main/common/` | `sht6-frontend/src/app/common/` |

The dedicated `common-*` pairs encode the asymmetric canonical-common layout: a single `common/` directory on the monorepo maps to nested per-leaf paths on the leaves. Variant common files (under `backend/project-src/app/common/`) flow through the parent `backend` pair and stay out of the common-pair shadow chain.

`.shadowignore` files in both leaves exclude the canonical-common path from the parent pair so it can't leak in either direction.

See `shadow-config.json` for the wire format.

## Usage

```bash
npm install

npm run sync -- --from b   # leaves → monorepo (replay onto shadow refs on sht6-main)
npm run sync -- --from a   # monorepo → leaves (replay onto shadow refs on sht6-{backend,frontend})

# B' fast path — composed-squash recovery for cross-project merges with
# divergent-outer mapped parents (see sht6 scenario Phase 6 in the
# shadow-sync repo):
SHADOW_ALLOW_COMPOSED_SQUASH=1 npm run sync -- --from b
```

The orchestrator pins `shadow-sync` by commit SHA in `package-lock.json`. Refresh with `npm install github:negedng/shadow-sync` to pull a newer engine.
