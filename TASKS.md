# TASKS — pmc-oa-cancer-corpus

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## How these tasks map to Hee-Lee Oss

Each task below becomes a Hee-Lee Oss **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug ID from the tables (e.g. `pmc-cancer-model-001`).
- `title` — the table's Title.
- `project` — `pmc-oa-cancer-corpus`.
- `type` — one of `code | research | writing | data | design-spec | maintenance` (per table).
- `lane` — `donated` for all current tasks (no funded escrow). The optional large-scale
  extraction item in the backlog is `funded` and therefore declares `fundedBudgetUsd`.
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["cancer-research","open-access","literature","bioinformatics"]`.
- `riskTier` — `low | medium | high`. Corpus/metadata = `low`; license-tiering and
  cancer-guardrail **judgement** tasks = `medium`. No `high` tasks here — patient-facing work is
  out of scope (it would be a separate `high`-risk, oncologist-+-advocate-reviewed project).
- `urgent` — boolean; `false` for all current tasks.
- `deliverable` — `pr | dataset | document | translation`. Pipeline/validator code → `pr`;
  the corpus + bundles → `dataset` (these are openly-licensed, redistributable artifacts);
  schema/datasheet/policy docs → `document`.
- `tokenEstimate` — `small | medium | large` (Size column).
- `status` — `open | in-progress | review | delivered | done`; all start `open`.
- `context`, `objective`, `acceptanceCriteria[]`, `resources[]`, `output` — per task.
- `requestor` — **TO BE SECURED** until a partner/consumer is confirmed.
- `verifiedNeed` — **`false`** until a named consumer (lab/advocacy/NLP group) agrees to use or
  accept a release, or a Zenodo release is adopted by an external reuser (general need is real;
  per-release delivery need is unproven).
- `outputLicense` — `CC-BY-4.0` for our metadata/manifest/datasheet; `MIT` for pipeline/validator
  code. **Redistributed source articles retain their original CC license** (recorded per article);
  bundles carry per-bundle LICENSE manifests and are never relicensed by us.

---

## Milestone M0 — Foundation & cold-start

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pmc-cancer-reviewer-014 | Name/secure the License+compliance reviewer (blocking gate role) | research | small | low | document | — | Maintainer |
| pmc-cancer-model-001 | Canonical corpus-record model + manifest schema | writing | small | low | document | — | Technical |
| pmc-cancer-gate-002 | License-tiering + cancer-guardrail triage checklist (blocking gate) | design-spec | medium | medium | document | model-001 | License+compliance |
| pmc-cancer-query-003 | Versioned cancer inclusion query (MeSH C04 + supplements) | research | small | medium | document | model-001 | Oncology |
| pmc-cancer-license-004 | License resolver + tiering (OA-field × JATS cross-check) | code | medium | low | pr | model-001, gate-002 | Technical |
| pmc-cancer-bundle-005 | License-segregated bundle builder + CI fail-on-violation assertion | code | medium | low | pr | license-004 | Technical |
| pmc-cancer-outreach-006 | Consumer/partner outreach + Zenodo fallback path | research | small | low | document | — | Steward |
| pmc-cancer-pilot-007 | End-to-end pilot corpus for one cancer subdomain | data | large | medium | dataset | model-001, gate-002, query-003, license-004, bundle-005, outreach-006, reviewer-014 | License+compliance, Technical |

**Acceptance criteria — key tasks**

- **model-001 (canonical record model + manifest schema)**
  - [ ] Documents every record field: `pmcid, pmid, doi, title, journal, pubYear, authors[],
        license {id, url, source, permitsRedistribution, permitsDerivatives, commercialUse,
        snapshotRef}, licenseTier, provenance {sourceService, retrievedAt, snapshotVersion,
        meshVersion, attribution}, mesh[], cancerTypes[], studyType, dataAvailability
        {aggregateOnly, mentionsControlledAccess, datasetRefs[]}, patientContentFlag, fullText,
        exclusion, specVersions`.
  - [ ] Manifest is JSON/JSON-LD, schema-validatable; bundles + datasheet are projections of it.
  - [ ] States deliverable boundary: corpus + metadata only; **no** patient-facing content; our
        contributions CC-BY-4.0, code MIT, source articles retain original CC license.
  - [ ] Includes a worked example record skeleton.

- **gate-002 (license-tiering + cancer-guardrail gate)**
  - [ ] Enumerates `licenseTier` mapping for every license id (CC0/BY/BY-SA/BY-ND/BY-NC/BY-NC-SA/
        BY-NC-ND/no-CC-code/NIHMS) → {commercial, non-commercial, no-derivatives, metadata-only,
        excluded}; **no default-allow** (unknown/missing → metadata-only/exclude).
  - [ ] Objective criterion: enters a redistributable bundle only if `permitsRedistribution:true`
        (and `permitsDerivatives:true` for derivative processing) set from the OA license field
        **cross-checked against JATS `<license>`**; any conflict = FLAG.
  - [ ] Encodes the **cancer guardrails**: OA/aggregate/de-identified only; controlled-access
        (dbGaP/EGA/biobanks) and identifiable patient data out of scope; controlled-access
        references only *noted*; COSMIC/OncoKB flagged, TCGA-GDC-open/GEO-open referenceable.
  - [ ] Encodes the **identifiable-patient-content** screen (case-report + media heuristics) →
        flagged articles forced metadata-only; never re-identify.
  - [ ] Requires license snapshot per license type (committed copy + SHA-256 + Wayback URL).
  - [ ] Produces a committed, reviewable PASS/FLAG/EXCLUDE + tier artifact per article batch.

- **bundle-005 (segregated bundle builder + fail-test)**
  - [ ] Builds bundles strictly keyed on `licenseTier`; emits per-bundle LICENSE manifests.
  - [ ] Ships a fixture where a non-commercial (and a flagged-patient-content) article is
        deliberately mis-routed and the build **fails in CI**.
  - [ ] Deterministic output; records the manifest content hash; code MIT; `pnpm build && pnpm
        test && pnpm lint` green; DCO signed-off.

- **pilot-007 (end-to-end pilot corpus)**
  - [ ] Pilot subdomain chosen for tractable size + strong CC-BY coverage, on a realistic
        acceptance path first (informal consumer **or** self-serve Zenodo DOI).
  - [ ] Every article passed gate-002 with a committed tier/guardrail artifact; **0** tier
        leakage and **0** patient-content leakage in redistributable bundles.
  - [ ] License correctness audited on a stratified sample (target 100%); provenance 100% complete.
  - [ ] Deterministic build; manifest hash recorded and reproduces on a clean rebuild.
  - [ ] Datasheet (composition/collection/uses/limitations/guardrails) produced.
  - [ ] Release **accepted** via informal channel or a published **Zenodo DOI** with the Steward's
        acceptance evidence artifact (`outcomes/<release-id>.json`) recorded — or submitted with
        the blocker surfaced.

**M0 Definition of Done:** License+compliance reviewer named (blocking role filled before pilot
review); canonical record model + manifest schema + license-tiering/cancer-guardrail gate +
versioned cancer query published; license resolver + segregated bundle builder green in CI with
golden fixtures (incl. the fail-on-violation test); one pilot corpus built end-to-end with a
datasheet, **0 tier/patient-content leakage**, 100% provenance, reproducible manifest hash, and
**accepted via informal channel or a Zenodo DOI** (evidence recorded) — or submitted with the
blocker surfaced; ≥ 1 partner-outreach thread opened.

---

## Milestone M1 — Gate hardened + reproducible full build

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pmc-cancer-patient-008 | Identifiable-patient-content detector + fixtures | code | medium | medium | pr | gate-002, license-004 | License+compliance, Technical |
| pmc-cancer-snapshot-009 | Pinned-snapshot ingestion + license-text snapshot capture | code | medium | low | pr | model-001, license-004 | Technical |
| pmc-cancer-fullbuild-010 | Full deterministic oncology corpus build from snapshot | data | large | medium | dataset | patient-008, snapshot-009, query-003, bundle-005 | License+compliance, Technical |
| pmc-cancer-partner-011 | Secure first confirmed consumer/partner (or proven Zenodo path) | research | small | low | document | outreach-006 | Steward |

**Acceptance criteria — key tasks**

- **patient-008 (patient-content detector)**
  - [ ] Implements the reviewer-approved signal set (case-report study-type + clinical-image /
        full-face / patient-media / individual-level supplementary signals); flagged → metadata-only.
  - [ ] Ships synthetic fixtures (flagged + clean); flagged cases must be forced metadata-only in CI.
  - [ ] Never extracts, enhances, links, or re-identifies; flags and records reason only.
  - [ ] Code MIT; tests + CI green; no real flagged content committed.

- **snapshot-009 (pinned ingestion + license snapshot)**
  - [ ] Ingests from official PMC OA file lists / OA Web Service + Europe PMC REST within rate
        limits; **no scraping; no committed API keys**.
  - [ ] Pins snapshot date + MeSH/JATS/SPDX versions into `specVersions`.
  - [ ] Captures license snapshot per license type (committed copy + SHA-256 + Wayback URL);
        `license.snapshotRef` records path + hash + Wayback timestamp.

- **fullbuild-010 (full corpus build)**
  - [ ] Full C04 oncology corpus built deterministically from the pinned snapshot; manifest hash
        recorded and reproduces on clean rebuild.
  - [ ] gate-002 + patient-008 applied across the build; committed audit artifact shows 0 tier
        leakage, 0 patient-content leakage, 100% provenance.
  - [ ] Per-tier bundles + per-bundle LICENSE manifests + datasheet emitted.

- **partner-011 (first confirmed consumer)**
  - [ ] A named lab/advocacy/NLP group confirms they will use/accept a release, **or** a Zenodo
        release is demonstrably adopted by ≥ 1 external reuser.
  - [ ] Consumption/acceptance mechanism documented; relevant tasks flip to `verifiedNeed:true`
        with `requestor` set.

**M1 Definition of Done:** patient-content detector implemented + enforced in CI; license
resolution cross-check + tiering hardened with negative/FLAG cases in CI; full deterministic
oncology corpus built from a pinned snapshot (manifest hash recorded) with 0 tier/patient-content
leakage and 100% provenance; license-snapshot capture working; ≥ 1 confirmed consumer **or** a
proven Zenodo release path.

---

## Milestone M2 — Metadata normalization & quality at scale

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pmc-cancer-gold-012 | Human-labeled gold set (cancer-type + study-type) | data | medium | medium | dataset | fullbuild-010 | Oncology, Technical |
| pmc-cancer-taxonomy-013 | MeSH→cancer-type taxonomy + study-type classifier | code | large | medium | pr | gold-012, query-003 | Oncology, Technical |
| pmc-cancer-dataavail-015 | Aggregate-vs-individual-data signal extractor | code | medium | medium | pr | fullbuild-010, gate-002 | License+compliance, Technical |
| pmc-cancer-release2-016 | Second full corpus release + data-quality report | data | large | medium | dataset | taxonomy-013, dataavail-015 | License+compliance, Technical |

**Acceptance criteria — key tasks**

- **gold-012 (gold set)**
  - [ ] ≥ 200 articles, stratified by cancer type, human-labeled for cancer-type + study-type;
        labels sourced + provenance recorded.
  - [ ] Inter-annotator agreement recorded; gold set committed (metadata-only; no flagged content).

- **taxonomy-013 (taxonomy + classifier)**
  - [ ] MeSH C04 → cancer-type taxonomy documented and versioned.
  - [ ] Classifier scored on the gold set meets thresholds: cancer-type ≥ 0.90 P / ≥ 0.85 R;
        study-type reported with a confusion matrix; thresholds enforced in CI.
  - [ ] Every tag carries provenance (source MeSH terms / fields); code MIT, CI green.

- **dataavail-015 (aggregate-vs-individual signal)**
  - [ ] Parses data-availability statements (and, where licenses permit derivative processing,
        full text) to set `aggregateOnly`, `mentionsControlledAccess`, `datasetRefs[]`.
  - [ ] ND-licensed articles processed for *facts only* (no derivative full-text redistribution),
        per the reviewer-confirmed ND rule.
  - [ ] Spot-audited sample (≥ 50) for correctness; results recorded.

**M2 Definition of Done:** gold set published; cancer-type/study-type classifier meets gold-set
thresholds in CI; aggregate-vs-individual-data signal implemented + spot-audited; a second full
release with a data-quality report shows measurable curation-quality improvement vs the M1
baseline; ≥ 1 verifiable external reuse event recorded.

---

## Milestone M3 — Refresh, reuse & sustainability

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pmc-cancer-refresh-017 | Incremental refresh/sync + retraction/removed-from-OA handling | maintenance | medium | medium | pr | snapshot-009, fullbuild-010 | License+compliance, Technical |
| pmc-cancer-reuse-018 | Track + verify external reuse events | research | small | low | document | release2-016 | Steward |
| pmc-cancer-maint-019 | Maintenance cadence + steward handoff doc | writing | small | low | document | refresh-017, partner-011 | Maintainer, Steward |

**Acceptance criteria — key tasks**

- **refresh-017 (refresh/retraction handling)**
  - [ ] Incremental sync detects new/updated/**retracted/removed-from-OA** articles vs the prior
        snapshot and re-runs the gate on changes.
  - [ ] Articles that leave OA or are retracted are removed from redistributable bundles and kept
        metadata-only with a recorded reason; release changelog + new manifest hash emitted.
  - [ ] Code MIT; CI green; deterministic.

- **reuse-018 (reuse tracking)**
  - [ ] ≥ 1 named consumer using a release in their work, **or** ≥ 2 verifiable external reuse
        events (citation / Zenodo derivative / dependent public repo), each with external evidence.
  - [ ] Events recorded in the outcome ledger alongside acceptance evidence artifacts.

**M3 Definition of Done:** refresh/sync + retraction handling working and deterministic; ≥ 1
named consumer using a release (or ≥ 2 verifiable reuse events); maintenance cadence documented
and a steward identified for ongoing source-tracking + consumer liaison.

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| pmc-cancer-europepmc-020 | Europe PMC annotation (SciLite) enrichment | code | medium | medium | pr | If annotation reuse terms verified compatible |
| pmc-cancer-extract-021 | Large-scale structured extraction pass (funded) | data | large | medium | dataset | **Funded lane**: metered run via `packages/runner` with a hard per-task budget cap; `fundedBudgetUsd` required |
| pmc-cancer-i18n-022 | Non-English OA cancer article inclusion (multilingual) | data | large | medium | dataset | Needs language-aware MeSH mapping + reviewer |
| pmc-cancer-croissant-023 | Croissant ML metadata descriptor for releases | code | small | low | pr | Improves ML reuse; emits schema.org/Croissant manifest |
| pmc-cancer-dash-024 | Outcome dashboard (acceptance + reuse events) | code | small | low | pr | Reads the outcome ledger; supports success-metric tracking |

---

---

## Generated task index

> Auto-generated by the Hee-Lee Oss task-decomposition agent on 2026-06-29.
> All files below are in `tasks/` and passed schema validation.

| File | ID | Title | Type | Lane | Priority | Risk | Deliverable | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| pmc-cancer-model-001.json | pmc-cancer-model-001 | Canonical corpus-record model + manifest schema | writing | donated | high | low | document | open |
| pmc-cancer-gate-002.json | pmc-cancer-gate-002 | License-tiering + cancer-guardrail triage checklist (blocking gate) | design-spec | donated | high | medium | document | open |
| pmc-cancer-query-003.json | pmc-cancer-query-003 | Versioned cancer inclusion query (MeSH C04 + supplements) | research | donated | high | medium | document | open |
| pmc-cancer-license-004.json | pmc-cancer-license-004 | License resolver + tiering (OA-field × JATS cross-check) | code | donated | high | low | pr | open |
| pmc-cancer-bundle-005.json | pmc-cancer-bundle-005 | License-segregated bundle builder + CI fail-on-violation assertion | code | donated | high | low | pr | open |
| pmc-cancer-outreach-006.json | pmc-cancer-outreach-006 | Consumer/partner outreach + Zenodo fallback path | research | donated | medium | low | document | open |
| pmc-cancer-pilot-007.json | pmc-cancer-pilot-007 | End-to-end pilot corpus for one cancer subdomain | data | donated | high | medium | dataset | open |
| pmc-cancer-patient-008.json | pmc-cancer-patient-008 | Identifiable-patient-content detector + fixtures | code | donated | high | medium | pr | open |
| pmc-cancer-snapshot-009.json | pmc-cancer-snapshot-009 | Pinned-snapshot ingestion + license-text snapshot capture | code | donated | high | low | pr | open |
| pmc-cancer-fullbuild-010.json | pmc-cancer-fullbuild-010 | Full deterministic oncology corpus build from snapshot | data | donated | high | medium | dataset | open |
| pmc-cancer-partner-011.json | pmc-cancer-partner-011 | Secure first confirmed consumer/partner (or proven Zenodo path) | research | donated | medium | low | document | open |
| pmc-cancer-gold-012.json | pmc-cancer-gold-012 | Human-labeled gold set (cancer-type + study-type) | data | donated | medium | medium | dataset | open |
| pmc-cancer-taxonomy-013.json | pmc-cancer-taxonomy-013 | MeSH→cancer-type taxonomy + study-type classifier | code | donated | medium | medium | pr | open |
| pmc-cancer-reviewer-014.json | pmc-cancer-reviewer-014 | Name/secure the License+compliance reviewer (blocking gate role) | research | donated | high | low | document | open |
| pmc-cancer-dataavail-015.json | pmc-cancer-dataavail-015 | Aggregate-vs-individual-data signal extractor | code | donated | medium | medium | pr | open |
| pmc-cancer-release2-016.json | pmc-cancer-release2-016 | Second full corpus release + data-quality report | data | donated | medium | medium | dataset | open |
| pmc-cancer-refresh-017.json | pmc-cancer-refresh-017 | Incremental refresh/sync + retraction/removed-from-OA handling | maintenance | donated | medium | medium | pr | open |
| pmc-cancer-reuse-018.json | pmc-cancer-reuse-018 | Track + verify external reuse events | research | donated | medium | low | document | open |
| pmc-cancer-maint-019.json | pmc-cancer-maint-019 | Maintenance cadence + steward handoff doc | writing | donated | low | low | document | open |
| pmc-cancer-europepmc-020.json | pmc-cancer-europepmc-020 | Europe PMC annotation (SciLite) enrichment | code | donated | low | medium | pr | open |
| pmc-cancer-extract-021.json | pmc-cancer-extract-021 | Large-scale structured extraction pass (funded) | data | funded | low | medium | dataset | open |
| pmc-cancer-i18n-022.json | pmc-cancer-i18n-022 | Non-English OA cancer article inclusion (multilingual) | data | donated | low | medium | dataset | open |
| pmc-cancer-croissant-023.json | pmc-cancer-croissant-023 | Croissant ML metadata descriptor for releases | code | donated | low | low | pr | open |
| pmc-cancer-dash-024.json | pmc-cancer-dash-024 | Outcome dashboard (acceptance + reuse events) | code | donated | low | low | pr | open |

**Total: 24 task files (1 seed + 23 generated). Fan-out dimensions: none.**

---

## Example task JSON

```json
{
  "id": "pmc-cancer-model-001",
  "title": "Canonical corpus-record model + manifest schema",
  "project": "pmc-oa-cancer-corpus",
  "type": "writing",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "open-access", "literature", "bioinformatics"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "The PMC Open Access Subset and Europe PMC OA holdings contain a large, mixed-license body of cancer literature but no oncology-curated, license-clear, reproducible corpus. Before building any corpus, the project needs one canonical per-article record model and a manifest schema that all outputs (license-segregated bundles, datasheet, normalized metadata) are projections of. The corpus is research-/builder-facing infrastructure: no patient-facing content, no controlled-access or identifiable patient data, provenance on every record. Our contributions are CC-BY-4.0 (code MIT); redistributed source articles retain their original CC license and are never relicensed.",
  "objective": "Define the canonical corpus-record model and the machine-validatable manifest schema that every per-corpus task projects from, including license, license-tier, provenance, cancer-guardrail, and patient-content-flag fields.",
  "acceptanceCriteria": [
    "Record model documents all fields: pmcid, pmid, doi, title, journal, pubYear, authors[], license {id, url, source, permitsRedistribution, permitsDerivatives, commercialUse, snapshotRef}, licenseTier (commercial|non-commercial|no-derivatives|metadata-only|excluded), provenance {sourceService, retrievedAt, snapshotVersion, meshVersion, attribution}, mesh[], cancerTypes[], studyType, dataAvailability {aggregateOnly, mentionsControlledAccess, datasetRefs[]}, patientContentFlag {flagged, reason, action}, fullText {available, format, reuseAllowedInBundle}, exclusion {excluded, reason}, specVersions {jats, mesh, datasheet}.",
    "Manifest is JSON/JSON-LD and validates against a committed JSON Schema; bundles and datasheet are documented as projections of the canonical record.",
    "Model states the scope boundary explicitly: corpus + metadata only, no patient-facing content, no controlled-access/identifiable patient data, provenance required on every record.",
    "Licensing rules are captured: our metadata/manifest/datasheet are CC-BY-4.0, pipeline code is MIT, and redistributed source articles retain their original CC license (never relicensed).",
    "At least one filled-in worked example record skeleton is included.",
    "pnpm build && pnpm test && pnpm lint pass for any committed schema tooling; commit is DCO signed-off."
  ],
  "resources": [
    "C:\\code\\hee-lee-oss\\planning\\projects\\pmc-oa-cancer-corpus\\PLAN.md",
    "C:\\code\\hee-lee-oss\\planning\\ROADMAP.md",
    "C:\\code\\hee-lee-oss\\packages\\schema\\src\\schemas.ts",
    "PMC Open Access Subset + NCBI OA Web Service (license fields)",
    "Europe PMC REST API; MeSH Neoplasms tree (C04); JATS; SPDX/Creative Commons license list",
    "Datasheets for Datasets (Gebru et al.)"
  ],
  "output": "A canonical corpus-record model definition plus a machine-validatable manifest JSON Schema, committed to the project repo and ready for reuse by all per-corpus build, gate, and metadata tasks.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```
