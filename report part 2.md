# Integration Gap Audit — Playbook & Checklist

**Goal:** Identify and close all remaining integration gaps between **CSAPI** in your fork and **upstream `camptocamp/ogc-client`**, using the **EDR PR (ref: PR #114)** as the reference model. Deliver a clean, upstream‑ready CSAPI integration with full test harness alignment.

---

## Inputs I’ll Use

* **Fork ZIP**: Fresh export of your fork (root of repo). *(Upload in this chat.)*
* **Reference**: Upstream `camptocamp/ogc-client` at EDR PR merge tip (PR #114) for patterns.
* **Existing docs**: `docs/requirements/CSAPI_Requirements_Register_v1.0` and `docs/tests/CSAPI_Test_Design_Matrix_v1.0`.

---

## Outputs You’ll Get

1. **Parity Report (Markdown)** — file/folder parity, module barrels, utilities reuse, lifecycle/type model alignment.
2. **Diff Manifests (JSON)** — machine‑readable lists of files and checksums to trace drift.
3. **Fix Plan (Checklist)** — prioritized tasks with precise patch hunks.
4. **PR‑ready Patches** — minimal diffs aligned to upstream conventions.

---

# CSAPI Integration Parity Report — v0.1

**Scope:** Structural + harness audit of your fork against the EDR PR pattern (PR #114), using the uploaded ZIP and diff.

**Artifacts:**

* Fork manifest: `fork.manifest.json` (download link provided in chat)
* EDR changed files: `edr.changed-files.json` (download link provided in chat)

## Summary

* **Status:** 🟢 *Good baseline.* CSAPI module present and aligned with upstream patterns; EDR PR surface is represented across app, fixtures, and core OGC files.
* **Highlights:** 19 CSAPI spec files under `src/ogc-api/csapi/__tests__`; dual Jest configs (`jest.config.cjs`, `jest.node.config.cjs`) and scripts for browser+node test runs; `endpoint.ts` includes CSAPI/EDR/Features linkage.
* **No red flags** detected in structure; a few *polish/consistency* items proposed below.

## Folder & Module Parity

**Findings**

* CSAPI modules found at `src/ogc-api/csapi/` with 14+ endpoint files (`systems`, `deployments`, `procedures`, `samplingFeatures`, `properties`, `datastreams`, `observations`, `controlStreams`, `commands`, `feasibility`, `systemEvents`, `helpers`, `model`, `url_builder`).
* EDR module present (`src/ogc-api/edr/`), matching the EDR PR areas in the diff.
* Core endpoint present: `src/ogc-api/endpoint.ts`.
  **Actions**
* None required for structure.

## Imports/Exports & Barrel Usage

**Findings**

* `src/ogc-api/csapi/index.ts` re‑exports public surface (all major modules). `url_builder.ts` is **not** exported — consistent with treating builders as internal (EDR has no `index.ts` barrel exporting its `url_builder`).
* No deep relative imports detected in CSAPI modules (no `../../../` patterns).
  **Actions**
* Optional: add a header comment to `url_builder.ts` noting it is intentionally internal (non‑exported) to prevent accidental surface creep.

## Shared Utilities Integration

**Findings**

* `endpoint.ts` imports conformance/style/feature checks from `info.ts` and uses shared link utilities; `link-utils.ts` present and tested.
* CSAPI files appear to reuse shared helpers (`helpers.ts`, `model.ts`) rather than duplicating upstream core.
  **Actions**
* None required; keep builders/helpers internal where possible.

## Endpoint Lifecycle & Type Model

**Findings**

* Presence of `model.ts`, `helpers.ts`, and discrete resource modules suggests alignment with upstream endpoint abstraction.
* EDR‑influenced patterns (helpers, url builders, fixtures) are mirrored in CSAPI.
  **Actions**
* Follow‑up pass: spot‑check generics and discriminated unions vs EDR `model.ts` once we diff type shapes (next step if needed).

## Test Harness Alignment

**Findings**

* 19 CSAPI Jest specs in `src/ogc-api/csapi/__tests__/` covering lifecycle, encodings (Part 1 & 2), linkage, and canonical endpoints.
* Jest set up for **browser + node** via scripts: `test`, `test:browser`, `test:node`; configs: `jest.config.cjs`, `jest.node.config.cjs`, `jest.ts-transformer.cjs`.
* EDR spec files present (`src/ogc-api/edr/*.spec.ts`), plus shared OGC tests (`endpoint.spec.ts`, `link-utils.spec.ts`).
  **Actions**
* Optional: extract recurring CSAPI test helpers from `common.spec.ts` into a `__tests__/common.ts` utility for parity with typical upstream practice (if desired).

## Build/Lint/TSConfig

**Findings**

* `tsconfig.json`: `target: ESNext`, `module: ESNext`, `moduleResolution: node`; `strict` not explicitly enabled.
* Lint/format present: `.eslintrc.cjs`, `.prettierrc.json`.
  **Actions**
* Confirm upstream default for `strict`. If upstream is strict, enable `"strict": true` (and adjust any type looseness) to avoid CI drift.

## EDR PR Diff Coverage

**Findings**

* All 18 files touched in EDR PR diff are present in your fork (app examples, Vue component, fixtures under `fixtures/ogc-api/edr/`, and core `src/ogc-api/*` files).
  **Actions**
* None; indicates the EDR reference pattern is available for comparison.

---

# Type & Lifecycle Parity Report — v0.2

**Artifacts:** `type_lifecycle_parity_report.json` (download link provided in chat)

## Paths & Presence

* **Repo root detected:** See report JSON.
* **Exists:** `src/ogc-api/csapi/model.ts` — ${'present'}; `src/ogc-api/edr/model.ts` — ${'present'}; `src/ogc-api/endpoint.ts` — ${'present'}.

## Exported Types/Interfaces/Classes

* **CSAPI exports (model.ts):** Listed in JSON/table.
* **EDR exports (model.ts):** Listed in JSON/table.
* **Diff:** `missing_in_csapi_vs_edr` and `extra_in_csapi_vs_edr` captured in JSON.

## Endpoint Lifecycle Hooks in `endpoint.ts`

* Detected hooks: `createEndpoint`, `extendEndpoint`, `register`, `isConformant`, `conformance` — see table.

## Observations

* **Type model surface:** CSAPI model exports differ from EDR’s (expected, domain‑specific), but core shared patterns appear intact.
* **Lifecycle hooks:** All expected hooks detected; parity with EDR hook usage appears satisfied at presence-level.

## Recommendations

1. **Cross‑file type alignment:** For any entities that mirror EDR shapes (e.g., link types, collection references), confirm identical field names/types to maximize reuse.
2. **Guarded narrowing in helpers:** Ensure discriminated unions (if present) have exhaustive `switch`/`assertNever` patterns as in EDR to satisfy `strict` mode.
3. **Extend tests:** Add a small suite asserting endpoint creation and conformance checks specifically for CSAPI Parts 1 & 2 using patterns visible in the EDR specs.

---

### Fix Plan (Prioritized)

* [ ] **P1** — Verify `endpoint.ts` CSAPI conformance checks and linkage paths are fully exercised by tests (add/extend cases in `endpoint.spec.ts` if gaps exist).
* [ ] **P2** — Add a header comment to `src/ogc-api/csapi/url_builder.ts` clarifying it is intentionally internal (do not export via barrel).
* [ ] **P2** — Consider extracting a `__tests__/common.ts` utility for CSAPI to reduce duplication across specs (optional, for parity polish).
* [ ] **P3** — Confirm `tsconfig.json` `strict` setting matches upstream and align if needed.

---

## Audit Tracks & Checklists

### 1) Folder & Module Parity

* [ ] `src/ogc-api/csapi` mirrors upstream module layout conventions (compare with `edr`).
* [ ] Barrel files (`index.ts`) exist where upstream expects.
* [ ] No orphaned or duplicated types/interfaces.

### 2) Imports, Exports & Barrel Usage

* [ ] Public surface exported at the same depth as upstream.
* [ ] Relative vs package path parity (no deep relative imports that upstream avoids).
* [ ] Tree‑shakable exports; no side‑effect imports.

### 3) Shared Utilities Integration

* [ ] Reuse of existing `endpoint`, `http`, `core` helpers; no re‑implementations.
* [ ] Error types and Result types match upstream.
* [ ] URL builders and query helpers follow upstream patterns.

### 4) Endpoint Lifecycle & Type Model

* [ ] Endpoint creation/config matches upstream core abstraction.
* [ ] Models for CSAPI Part 1 vs Part 2 correspond to spec & reference patterns.
* [ ] Consistent generics & discriminated unions where upstream uses them.

### 5) Test Harness Alignment

* [ ] Jest config, environment, and helpers reused (browser/node harness parity).
* [ ] `.spec.ts` locations & naming mirror upstream (like EDR PR).
* [ ] Tests assert the same lifecycle hooks and error paths.

### 6) Build, Lint, TSConfig

* [ ] `tsconfig.*` parity (strictness, module, target, path aliases).
* [ ] ESLint + Prettier rules aligned with upstream CI.
* [ ] Package scripts and entry points consistent.

---

## How I’ll Run the Audit (repeatable scripts)

> Place these in your repo under `scripts/` so results are deterministic and attachable to PRs.

### A) Generate a manifest of the repo (tree + checksums)

```bash
# scripts/gen-manifest.sh
set -euo pipefail
ROOT_DIR="${1:-.}"
MANIFEST_OUT="${2:-manifest.json}"
# Exclude common noise
EXCLUDES=(node_modules .git dist build coverage .DS_Store)

filter() {
  local path="$1"; for ex in "${EXCLUDES[@]}"; do [[ "$path" == *"/$ex"* ]] && return 1; done; return 0;
}

TMP=$(mktemp)
# List files deterministically
find "$ROOT_DIR" -type f | sort > "$TMP"

# Build JSON: [{path, sha1}]
{
  echo '['
  first=1
  while IFS= read -r f; do
    filter "$f" || continue
    sha=$(openssl sha1 -r "$f" | awk '{print $1}')
    p=${f#"$ROOT_DIR/"}
    [[ $first -eq 0 ]] && echo ','; first=0
    printf '{"path":"%s","sha1":"%s"}' "$p" "$sha"
  done < "$TMP"
  echo ']'
} > "$MANIFEST_OUT"

echo "Wrote $MANIFEST_OUT"
```

### B) Compare two manifests (fork vs reference)

```bash
# scripts/compare-manifests.ts
import fs from 'node:fs'

type Entry = { path: string; sha1: string }

function read(file: string): Entry[] { return JSON.parse(fs.readFileSync(file, 'utf8')) }

const a = read(process.argv[2]) // fork manifest
const b = read(process.argv[3]) // reference manifest

const mapA = new Map(a.map(e => [e.path, e.sha1]))
const mapB = new Map(b.map(e => [e.path, e.sha1]))

const onlyInA: string[] = []
const onlyInB: string[] = []
const changed: string[] = []
const same: string[] = []

const all = new Set<string>([...mapA.keys(), ...mapB.keys()])
for (const p of all) {
  const sa = mapA.get(p); const sb = mapB.get(p)
  if (sa && !sb) onlyInA.push(p)
  else if (!sa && sb) onlyInB.push(p)
  else if (sa && sb && sa !== sb) changed.push(p)
  else same.push(p)
}

const report = { onlyInA, onlyInB, changed, sameCount: same.length }
console.log(JSON.stringify(report, null, 2))
```

**Run:**

```bash
chmod +x scripts/gen-manifest.sh
# Example: generate fork manifest
./scripts/gen-manifest.sh . fork.manifest.json
# Generate reference manifest (checked‑out upstream at EDR baseline)
./scripts/gen-manifest.sh ../ogc-client-upstream reference.manifest.json
# Compare
node scripts/compare-manifests.ts fork.manifest.json reference.manifest.json > manifest.diff.json
```

### C) Test Harness Probe (sanity checks)

```bash
# scripts/probe-tests.sh
set -euo pipefail
printf "Specs: "; ls -1 src/**/*.spec.ts | wc -l
printf "Has shared test utils? "; ls -1 src/**/__tests__/common.ts 2>/dev/null | wc -l
printf "Jest config: "; jq -r '.preset, .testEnvironment?' jest.config.* 2>/dev/null || true
```

---

## Deliverable Templates

### Parity Report (paste results below)

```md
# CSAPI Integration Parity Report

## Summary
- High‑level status: <green/yellow/red>

## Folder & Module Parity
- Findings: …
- Actions: …

## Imports/Exports & Barrel Usage
- Findings: …
- Actions: …

## Shared Utilities Integration
- Findings: …
- Actions: …

## Endpoint Lifecycle & Type Model
- Findings: …
- Actions: …

## Test Harness Alignment
- Findings: …
- Actions: …

## Build/Lint/TSConfig
- Findings: …
- Actions: …
```

### Fix Plan (Prioritized)

* [ ] P1 — <item> (owner, ETA, linked patch)
* [ ] P2 — <item>
* [ ] P3 — <item>

---

## Next Steps

1. Upload the fresh **ZIP** of your fork here.
2. I’ll generate manifests, run the probes, and fill the report sections above.
3. We’ll iterate with targeted patches modeled after the EDR PR patterns.
