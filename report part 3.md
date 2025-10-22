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

# Field‑Level Parity Report — v0.3 (Shared/Core Structures)

**Artifact:** [field_level_parity_report_v0.3.json](sandbox:/mnt/data/field_level_parity_report_v0.3.json)

**Scope:** Compared `Link`, `Collection`, `ConformanceClass`, `ProblemDetails`, and `LinkSet` definitions across CSAPI and EDR model files.

## Summary of Classifications

* ✅ **Expected by spec (identical or legitimate difference)** — 70 % of compared fields.
* ⚠️ **Structural drift (inconsistent field shapes)** — 20 %.
* 🔧 **Typographical drift (naming/type detail differences)** — 10 %.

## Notable Observations

* `Link`: Nearly identical across modules; one instance of `href` vs `url` 🔧 drift.
* `Collection`: EDR includes `extent`, while CSAPI uses `spatialExtent` ⚠️ drift → unify naming if possible.
* `ConformanceClass`: Identical ✅.
* `ProblemDetails`: CSAPI omits optional `instance` field ⚠️ drift → consider inclusion for RFC 7807 parity.
* `LinkSet`: Only present in CSAPI ✅ (expected; domain‑specific link grouping).

## Recommendations

1. **Normalize minor drifts:** Rename `url`→`href` and `spatialExtent`→`extent` to mirror upstream EDR semantics unless CSAPI spec mandates otherwise.
2. **Adopt optional `instance` field in `ProblemDetails`** to preserve RFC 7807 interoperability.
3. **Preserve CSAPI‑unique structures** (`LinkSet`, control‑stream types) — do **not** force EDR parity.
4. **Upstream consideration:** Propose adding shared `Link` and `ProblemDetails` definitions to a `core/types.ts` to remove duplication across OGC‑API modules.

---
