---
name: package-manager
description: Audit, update, and manage packages and pinned versions across the monorepo. Detects drift between package.json, pyproject.toml, Cargo.toml, and versions.json.
argument-hint: "<audit|update|pin|remove> [package-name]"
---

# /packages - Manage Packages & Versions

## Usage

```
/packages audit          # scan all manifests, report drift
/packages update <pkg>   # update a specific package everywhere
/packages pin            # sync versions.json with current lockfiles
/packages remove <pkg>   # remove a package from all manifests
```

## Workflow

### 1. Discover Manifests

Scan the repo for:
- `**/package.json` — npm/yarn
- `**/pyproject.toml` — Python (uv/pip)
- `**/Cargo.toml` — Rust
- `**/Brewfile` — Homebrew
- `versions.json` — pinned version manifest (source of truth)

### 2. Audit

For each manifest:
- Parse current versions
- Compare against `versions.json` pins
- Flag: **drift** (manifest != pin), **unpinned** (not in versions.json), **stale** (pin but package removed)
- Run `npm audit` / `pip audit` / `cargo audit` for security advisories

### 3. Update

1. Update version in all manifests that reference the package
2. Run lock command (`npm install`, `uv lock`, `cargo update`)
3. Update `versions.json`
4. Run tests to verify
5. Report changes

### 4. Output

```
PACKAGE AUDIT
━━━━━━━━━━━━
✔ 42 packages in sync
⚠ 3 drifted (manifest != pin)
  typescript: pin 5.8.0, found 5.7.3 in packages/scrapy-cli/package.json
✘ 1 security advisory
  lodash < 4.17.21: prototype pollution
```

### 5. Safety

- Never auto-update major versions without confirmation
- Always run tests after updates
- Create a dedicated branch for bulk updates
