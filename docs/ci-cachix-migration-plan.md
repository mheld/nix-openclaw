# CI cache migration plan (Garnix → Cachix) — paused

Status: **paused**. Branch captures WIP so `main` can return to Garnix while we re-evaluate.

## Goal
- Get back to **reliable cached builds** for nix-openclaw.
- Avoid CI hard-stops like the Garnix “monthly CI quota exhausted” failure.

## Why we started this
- Garnix checks were failing with: `You have exhausted your monthly CI quota`.
- That prevents building → prevents cache population → makes installs slow.

## What we changed in this branch (WIP)
- Added GitHub Actions workflow(s) intended to:
  - build the same target set as `garnix.yaml` (Linux + macOS)
  - push build outputs to a Cachix binary cache
- Reworked/removed Garnix-specific cache verification.
- Removed `garnix.yaml` (later reverted plan: keep it on main while we test Garnix recovery).

## Key missing piece / blocker
Cachix has two separate capabilities:
1) **Cache management** (create cache, toggle visibility, keys, members)
2) **Data plane** (upload NARs / store paths)

The token we generated appears to be **data-plane only** (JWT scope looked like `tx`).
- It can’t authenticate to Cachix management endpoints (`/api/v1/organization`, `/api/v1/token`, etc.), which are required to create/own a cache.
- Result: we can’t fully “self-serve” cache provisioning via API with that token alone.

## If we resume: recommended path
### Option A (simple)
- Create a Cachix cache manually in the Cachix UI (name: `nix-openclaw`, public).
- Then CI only needs:
  - `CACHIX_CACHE_NAME=nix-openclaw`
  - `CACHIX_AUTH_TOKEN=<push token>`

### Option B (fully automated)
- Obtain a Cachix token with **management** permissions (org/account) that can:
  - read org/account (`GET /api/v1/organization`)
  - create cache (`POST /api/v1/cache/{name}`)
  - set visibility / generate signing key

## Proposed CI shape (once unblocked)
- Workflow: `Nix Build Cache`
  - matrix-ish: Linux + macOS
  - run `nix build` for target list
  - push store paths to Cachix (`cachix watch-exec ...`)
- (Optional) Workflow: `Cache Only`
  - verify required outputs exist in cache (nice-to-have guard)

## Rollback / safety
- Keep Garnix config on `main` until Cachix is proven.
- Only then remove Garnix checks/config.

## Notes
- Any additional secrets should be stored in `nix-secrets` and synced into GitHub Actions secrets.
