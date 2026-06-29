# PLAN — pmc-oa-cancer-corpus

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## Executive summary

Cancer research moves fast and is scattered across hundreds of journals. Researchers,
data scientists, patient advocates, and tool builders who want a clean, license-clear,
reproducible slice of the **open-access cancer literature** to read, mine, or benchmark
against have no canonical, well-documented corpus to start from. They must each re-derive
one from PubMed Central (PMC), re-solve the per-article licensing puzzle, and re-handle the
edge cases — duplicating fragile, error-prone work that most get subtly wrong (especially
licensing and the handling of case reports).

This project produces a **curated, license-segregated, fully-provenanced corpus of
open-access cancer literature plus normalized metadata**, drawn exclusively from the
**PMC Open Access (OA) Subset** and **Europe PMC OA** holdings, filtered to oncology by
MeSH. The deliverables are: (1) a **corpus manifest** (the authoritative list of included
articles with per-article identifiers, license, license group, provenance, and cancer-type
tags); (2) **normalized metadata** (MeSH → cancer-type mapping, study-type classification,
an aggregate-vs-individual-data signal, full-text-availability flags); (3) **license-segregated
redistributable bundles** built only from articles whose individual license permits reuse;
and (4) a **datasheet** and reproducible **build pipeline** so anyone can regenerate or audit
the corpus from source. The corpus is research-/builder-facing infrastructure, not
patient-facing content.

The headline gate is **per-article licensing within the cancer guardrails**. "PMC OA" is
not one license — it is a mix of CC0, CC-BY, CC-BY-SA, CC-BY-ND, CC-BY-NC(-SA/-ND), and
legacy "no-CC-code" articles whose reuse terms differ sharply. We resolve and record the
license for **every** article, segregate the corpus by reuse rights, and **never** silently
fold a non-commercial or no-license article into a commercial-use bundle. Binding cancer
guardrails apply on top: **only open-access, aggregate, or de-identified content** is in
scope; controlled-access data (dbGaP, EGA, individual-level biobanks) and any
identifiable patient data are out of scope; provenance is attached to every record; and any
derived **patient-facing** artifact is treated as a separate `high`-risk, oncologist-and-advocate-reviewed,
"not medical advice" deliverable (and is *out of scope for this corpus project*).

Risk tier is **low** for the corpus and metadata itself (it republishes already-public,
openly-licensed scholarly text and describes it). The real risks are **legal/factual**:
mislabeling an article's license, leaking a non-commercial article into a commercial bundle,
or treating an article that embeds identifiable patient media as ordinary text. The plan
front-loads the licensing + cancer-guardrail gate as a hard, auditable, blocking step.

## Problem & beneficiaries

**Who is helped.**
- **Cancer researchers and bioinformaticians** who need a clean, citable, reproducible
  literature slice for reviews, hypothesis generation, or as a retrieval corpus.
- **NLP / ML builders** building biomedical text-mining, retrieval-augmented tools, or
  benchmarks who need a corpus with *clear, machine-readable reuse rights* (commercial vs
  non-commercial) so they do not unknowingly train on non-commercial text.
- **Patient advocates and advocacy orgs** who want a sourced map of what the open
  literature says about a cancer type — feeding (separately, under `high`-risk review)
  downstream plain-language efforts. This corpus supplies the *evidence base*, never the
  advice.
- **Systematic reviewers / evidence-synthesis teams** who need a deduplicated, metadata-rich
  starting set with provenance.
- **The open-science commons** generally: a maintained, audited corpus reduces duplicated,
  often-incorrect license handling across the field.

**The verified need.** The *general* need is well established: PMC OA is large and widely
used, but it is delivered as raw bulk packages with mixed licenses and no oncology
curation, and downstream reusers repeatedly mishandle the license partitions and case-report
edge cases. We treat the general need as real. The **specific, per-partner need is
TO BE SECURED**: we have not yet confirmed a named research group, advocacy org, or archive
that has agreed to consume or host a release. Until a named beneficiary confirms they will
use/accept a release, tasks carry `verifiedNeed: false`. This honesty matters because
"delivered, not merged" requires the corpus to actually be *used or accepted* by a
beneficiary, not merely produced.

**Partner org.** TO BE SECURED. Candidate channels: an academic cancer-informatics lab; a
patient-advocacy research program (e.g. a rare-cancer foundation's science team); a
biomedical-NLP community (BioNLP / BLAH / Europe PMC ecosystem); and a self-serve archival
fallback (a versioned **Zenodo** record with a DOI) that does not depend on a third party
accepting. M0 includes explicit partner outreach; no partner is assumed.

## Goals and non-goals

**Goals**
- Produce a **reproducible, auditable corpus build pipeline** that, from a pinned snapshot of
  PMC OA / Europe PMC, emits a curated oncology corpus deterministically.
- Resolve and record the **license for every article**, segregate the corpus into
  reuse-rights tiers (commercial-OK / non-commercial-only / no-derivatives / metadata-only),
  and make the segregation machine-readable and non-bypassable.
- Filter to cancer via an explicit, versioned **MeSH-based inclusion query** and normalize
  metadata (cancer-type tags, study-type classification, aggregate-vs-individual-data signal).
- Attach **provenance to every record** (PMCID/PMID/DOI, source service + retrieval date,
  snapshot version, license URL + snapshot, MeSH version).
- Publish a **datasheet** (Datasheet-for-Datasets) describing composition, collection,
  uses, limitations, and ethical/guardrail handling.
- Enforce the **cancer guardrails** as a hard gate: open-access/aggregate/de-identified
  only; identifiable-patient-content detection and exclusion; no controlled-access data.

**Non-goals**
- We do **not** produce any patient-facing or clinical content, summaries, or advice. Any
  such derivative is a separate `high`-risk, oncologist-+-advocate-reviewed project and is
  explicitly **out of scope here**.
- We do **not** ingest controlled-access data (dbGaP, EGA, individual-level biobanks) or any
  identifiable patient data, and we do **not** de-identify or re-identify anything ourselves.
- We do **not** include articles whose license cannot be verified to permit our use; they are
  flagged and either dropped or kept **metadata-only**, never folded into redistributable
  full-text bundles.
- We do **not** scrape publisher sites or paywalled content; sources are PMC OA / Europe PMC
  OA official services only.
- We do **not** mix license tiers in a single bundle, and we do **not** modify/redistribute
  derivative full text of **no-derivatives (ND)** articles (metadata + unmodified originals
  only, where the OA service itself distributes them).
- We do **not** assert biomedical conclusions; extracted metadata is descriptive and
  source-cited, not interpretive.

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Vanity metrics ("articles ingested") are excluded;
ingestion only counts when paired with verified license correctness and a real consumer.

| Metric | Baseline | Target (first 6 months) |
| --- | --- | --- |
| **License correctness** on an audited sample (article license tier matches source ground truth) | n/a | **100%** on a stratified audit sample of ≥ 200 articles; **0** non-commercial/no-license articles found in a commercial bundle |
| Identifiable-patient-content **leakage** (articles with flagged identifiable media wrongly in a redistributable bundle) | n/a | **0** (zero-tolerance gate) |
| Corpus **release accepted/used by a beneficiary** (last-mile delivered: hosted, cited, or adopted) | 0 | ≥ 1 release accepted by a named partner **or** a versioned Zenodo DOI adopted by ≥ 1 external reuser |
| Provenance completeness (records with full, verifiable provenance) | n/a | **100%** of included records carry PMCID + license + source + snapshot version |
| Cancer-type tag accuracy vs a human-labeled gold set | n/a | ≥ 0.90 precision / ≥ 0.85 recall on a ≥ 200-article gold set |
| Reproducibility: independent rebuild from the pinned snapshot reproduces the manifest | n/a | Byte-identical manifest hash from a clean rebuild (deterministic) |
| Confirmed consumers/partners | 0 | ≥ 1 secured (or self-serve Zenodo fallback proven) |
| Verifiable external reuse events (citation / fork / downstream tool) | 0 | ≥ 1 with externally verifiable evidence |

Notes: a "reuse event" must be externally verifiable (a citation, a Zenodo download/derivative
record, a public repo depending on the corpus). Self-reported reuse does not count. "License
correctness 100% on the audit sample" is a **release-blocking** gate, not a stretch goal.

**Quantifying corpus quality (so DoDs are checkable).**
- *License integrity* — every article has a resolved `licenseTier ∈ {commercial, non-commercial,
  no-derivatives, metadata-only}` with a cited source field; bundles are built **by tier** and a
  CI check fails the build if any bundle contains an article of a disallowed tier.
- *Curation precision/recall* — cancer-type tagging and study-type classification are scored
  against a committed, human-labeled **gold set** (≥ 200 articles, stratified by cancer type).
- *Reproducibility* — the build is deterministic from a pinned snapshot; the manifest's
  content hash is recorded and must reproduce on a clean rebuild.

## Scope

**In scope**
- A **corpus manifest** (authoritative include-list): per article — PMCID, PMID, DOI,
  title, journal, pub year, license (SPDX/CC id), `licenseTier`, license URL + snapshot ref,
  source service + retrieval date, snapshot/version, MeSH version, cancer-type tags,
  study-type class, `dataAvailability` signal (aggregate vs individual / controlled-access
  mentions), full-text availability + format, and exclusion/flag reasons where applicable.
- **Normalized metadata**: MeSH (C04 Neoplasms tree) → cancer-type taxonomy mapping;
  study-type classification (review / clinical trial / observational / preclinical / case
  report / methods); an **aggregate-vs-individual-data signal** flag.
- **License-segregated redistributable bundles**: separate, clearly-labeled bundles per
  reuse tier (e.g. `commercial-use/`, `non-commercial-use/`), built only from articles whose
  individual license permits redistribution, in the format the OA service distributes
  (JATS XML / text), plus a per-bundle LICENSE manifest.
- **Datasheet-for-Datasets** + provenance records + a **reproducible build pipeline** (code).
- The **cancer-guardrail + license gate** (triage, detection, recording) for the corpus.

**Out of scope**
- Patient-facing content, clinical summaries, treatment information, or any medical advice
  (separate `high`-risk project; this corpus only supplies the sourced evidence base).
- Controlled-access data (dbGaP, EGA, individual-level biobanks), any identifiable patient
  data, and any de-identification/re-identification work.
- Non-OA / paywalled / publisher-site scraping; PMC author-manuscript content whose reuse
  terms are not CC-clear is metadata-only.
- Folding no-license or non-commercial articles into commercial bundles; modifying/redistributing
  derivative full text of ND-licensed articles.
- Biomedical interpretation, ranking, or conclusions; building search UIs or live services.
- Linking to or republishing **dataset** content referenced by papers; we only *note* a
  paper's data-availability statement. Where a paper references external datasets, the same
  license gate applies (GDC/GEO **open tiers** permissible to *reference*; COSMIC/OncoKB and
  any non-commercial/custom-terms source → **flag**, never bundle).

## Solution approach & architecture

This is a **content/data-curation project with supporting software** (a deterministic build
pipeline + validators + a license/guardrail gate). It produces a corpus and metadata, not a
runtime service.

**Pipeline (per snapshot → corpus build)**
1. **Snapshot & pin** — pin a dated snapshot of the PMC OA file lists + Europe PMC OA index
   and the **MeSH** vocabulary version. Everything downstream is reproducible from this pin.
2. **Cancer filter (inclusion query)** — select articles via an explicit, versioned MeSH
   query over the **Neoplasms (C04)** tree (plus curated supplementary terms), recorded as a
   committed query artifact with its result count and snapshot.
3. **License resolution & tiering (gate, blocking)** — for every candidate article, resolve
   its license from the OA service's authoritative license field + the article's
   `<license>`/`<copyright>` JATS metadata; map to an SPDX/CC id and a `licenseTier`. Records
   with conflicting or missing license evidence are FLAGGED (metadata-only or excluded), never
   default-allowed.
4. **Cancer-guardrail screen (gate, blocking)** — confirm open-access/aggregate/de-identified
   only; run the **identifiable-patient-content detector** (case-report + media heuristics, see
   Data/licensing); flag controlled-access-data mentions; route any flag to exclude/metadata-only.
5. **Metadata normalization** — MeSH → cancer-type tags; study-type classification;
   aggregate-vs-individual-data signal; dedup by DOI/PMID.
6. **Provenance capture** — PMCID/PMID/DOI, source service, retrieval date, snapshot version,
   license URL + snapshot ref, MeSH version, attribution string.
7. **Bundle build (by tier)** — assemble redistributable bundles strictly per `licenseTier`;
   a build-time assertion fails if any article lands in a disallowed bundle.
8. **Validate & datasheet** — schema-validate the manifest, run the gold-set evaluation, emit
   the data-quality report and the Datasheet; record the manifest content hash.
9. **Review & release** — License reviewer + technical reviewer sign off; a human publishes the
   release (Zenodo DOI / partner hand-off). No automated publishing.

**Canonical corpus-record model.** A single internal record per article is the source of
truth; the manifest, bundles, and datasheet are *projections* of it. Fields:
`pmcid`, `pmid`, `doi`, `title`, `journal`, `pubYear`, `authors[]` (already-public byline),
`license {id (SPDX/CC), url, source ("oa-service"|"jats"|"both"), permitsRedistribution:boolean,
permitsDerivatives:boolean, commercialUse:boolean, snapshotRef}`,
`licenseTier ("commercial"|"non-commercial"|"no-derivatives"|"metadata-only"|"excluded")`,
`provenance {sourceService, retrievedAt, snapshotVersion, meshVersion, attribution}`,
`mesh[]`, `cancerTypes[]`, `studyType`, `dataAvailability {aggregateOnly:boolean,
mentionsControlledAccess:boolean, datasetRefs[]}`,
`patientContentFlag {flagged:boolean, reason, action}`,
`fullText {available:boolean, format ("jats"|"txt"|"none"), reuseAllowedInBundle:boolean}`,
`exclusion {excluded:boolean, reason}`, `specVersions {jats, mesh, datasheet}`.

**Tech stack.** TypeScript, ESM, pnpm workspaces (per Elyos conventions). The pipeline is a
small set of Node packages with minimal dependencies. Sources accessed via the **NCBI PMC OA
Web Service / FTP file lists** and the **Europe PMC REST API** (both official, no scraping,
respect rate limits, no API keys committed). Corpus artifacts are JATS XML / text + JSON/JSON-LD
manifests + Markdown datasheet. No runtime service; runs locally or in CI.

**Pinned source/spec versions** (recorded in `specVersions`; bumped only by a deliberate task):
- **PMC OA Subset** file lists + license field — pinned by snapshot date.
- **Europe PMC** REST API — pinned API version.
- **JATS** (Journal Article Tag Suite) — pinned version for full-text parsing.
- **MeSH** — pinned year/version (the C04 tree changes between releases).
- **SPDX / Creative Commons** license identifiers — pinned list.
- **Datasheets for Datasets** (Gebru et al.) — pinned template.

**Key decisions.**
- *Canonical-record-first*: manifest/bundles/datasheet are projections of one record model.
- *License gate is blocking and bypass-proof*: bundle assembly is keyed on `licenseTier` and a
  CI assertion fails the build on any tier violation — segregation cannot be skipped by hand.
- *Determinism*: build from a pinned snapshot; record the manifest content hash for rebuild
  verification.
- *Metadata-only is a first-class outcome*: articles we cannot redistribute still get a metadata
  record (facts/citations are reusable) so the corpus is complete without violating licenses.

## Data, licensing & compliance

**This is the critical section, and it leads with the binding cancer guardrails.**

### Binding cancer guardrails (non-negotiable, applied as a blocking gate)
1. **Open-access / aggregate / de-identified only.** In scope: PMC OA Subset / Europe PMC OA
   articles (already published, openly distributed) and the aggregate facts within them.
   **Out of scope, always:** controlled-access data (dbGaP, EGA, individual-level biobanks) and
   **any identifiable patient data**. We never seek, ingest, store, or link controlled-access or
   identifiable patient-level content. If an article *references* controlled-access data, we
   record only that fact (`mentionsControlledAccess: true`) — we never follow it.
2. **Identifiable-patient-content screen.** Some published articles (esp. **case reports**)
   may embed potentially identifiable patient media (clinical photographs, rare-disease detail,
   full-face/identifying images, supplementary patient-level tables). Every case-report-classified
   article and every article with image/supplementary-media signals is routed through the
   **patient-content detector**; flagged articles are kept **metadata-only** (no full-text
   redistribution in a bundle) and the reason recorded. We **never** attempt to identify, link,
   or re-identify any individual, and we never enhance or extract identifying media.
3. **No medical advice; corpus is research-facing.** This project ships *literature + metadata*,
   not guidance. Any downstream **patient-facing** artifact derived from it is a **separate
   `high`-risk project** requiring oncologist **and** patient-advocate review and an explicit
   "not medical advice" framing — explicitly out of scope here.
4. **Provenance on every assertion.** Every record and every normalized tag links to its source
   (PMCID/DOI + the field/source it was derived from). No unsourced claims.

### Source licenses (resolved and tiered per article — never assumed for the set)
"PMC OA" is a mix; we resolve **each article** and assign a `licenseTier`:
- **Commercial-use tier** — CC0, CC-BY (4.0/3.0), CC-BY-SA, CC-BY-ND* — permit redistribution
  and (except ND) derivatives, including commercial. *(ND permits redistribution of the
  *unmodified* article only; see below.)*
- **Non-commercial tier** — CC-BY-NC, CC-BY-NC-SA, CC-BY-NC-ND — redistributable for
  non-commercial use only; kept in a clearly-labeled `non-commercial-use/` bundle, **never** in
  the commercial bundle.
- **No-derivatives handling** — for any **ND** license (CC-BY-ND, CC-BY-NC-ND): we redistribute
  only the **unmodified** article as distributed by the OA service; we **do not** create or
  redistribute derivative/transformed full text. Descriptive metadata and source-cited facts
  about ND articles are fine (facts are not copyrightable); modified text is not.
- **Metadata-only / excluded tier** — legacy **"no-CC-code"** PMC articles, PMC **author
  manuscripts (NIHMS)** without CC terms, and any article whose license cannot be verified:
  these are **never** redistributed as full text; we keep a metadata-only record (identifiers +
  citation + MeSH) or exclude entirely, with the reason recorded. **No default-allow, ever.**

**Objective "permits redistribution / derivatives" criterion.** An article enters a
redistributable bundle only if `license.permitsRedistribution: true` (and, for derivative
processing, `permitsDerivatives: true`) is set from the **authoritative OA-service license
field cross-checked against the article's JATS `<license>` metadata**, with the SPDX/CC id and
URL recorded. Conflict between the two sources, a missing license, or an unparseable license =
metadata-only/exclude — never a guess.

**Referenced external datasets (the prompt's named cases).** When an article cites datasets, we
record references but apply the same gate before any *linking/labeling* as reusable:
- **TCGA / GDC open tier, GEO (open)** — open; permissible to *reference/label* as open-access.
- **COSMIC, OncoKB** — non-commercial / custom terms → **flag**; recorded as
  `mentionsControlledAccess`-adjacent / custom-terms and never represented as freely reusable.
We do **not** ingest or bundle any of this dataset content; this corpus is literature only.

**Provenance model.** Every included record stores: PMCID, PMID, DOI, source service, retrieval
timestamp, snapshot version, license id + URL + a **license snapshot** (committed copy of the
license text/page + SHA-256 + Wayback save URL), MeSH version, and the required attribution
string. Provenance is part of the committed deliverable and is 100%-complete by gate.

**Privacy/PII stance.** The dominant privacy concern is *upstream* identifiable patient content
in already-published articles. Handled by the mandatory patient-content screen (guardrail #2):
flagged articles are metadata-only and the concern surfaced; we never de-identify, re-identify,
extract, or enhance identifying media. Author bylines are already-public scholarly metadata and
are retained as published. We hold no individual-level health data.

**Attribution & output license.** Each redistributed article retains its **original CC license
and attribution** (we do not relicense source articles). Our **own contributions** — the
manifest, normalized metadata, taxonomy, datasheet — are licensed **CC-BY-4.0**; the **build
pipeline / validator code is MIT**. Per-bundle LICENSE files spell out the included articles'
licenses and the reuse tier.

## Quality, review & risk gates

**Risk tier: low** for the corpus + metadata (already-public, openly-licensed scholarly text,
described with provenance). The material risks are legal/factual (license mislabeling,
tier leakage, patient-content leakage), mitigated by hard gates below. **Note:** any
*patient-facing* derivative would be `high` and is out of scope here.

**Required review before a release is "done":**
- **License + compliance reviewer** (mandatory, every release): audits a stratified sample for
  license-tier correctness and confirms **zero** non-commercial/no-license articles in the
  commercial bundle and **zero** patient-content-flagged articles in any redistributable bundle.
  This is the **hard, non-skippable gate**.
- **Technical reviewer** (mandatory): confirms the build is deterministic, the manifest validates
  against the schema, the gold-set evaluation runs, and CI is green.
- **Oncology / domain reviewer** (consulted): sanity-checks the cancer inclusion query, the
  cancer-type taxonomy, and the patient-content heuristics. *(Required sign-off only if any
  patient-facing derivative is ever proposed — which would move that work to a separate
  `high`-risk project.)*

**Test fixtures & golden files (so "CI green" means something).** Built only from
synthetic/public fixtures (never embedding flagged content):
- **License-tiering fixtures** — articles with each license id (CC0/BY/BY-SA/BY-ND/BY-NC/
  BY-NC-SA/BY-NC-ND/no-CC-code/NIHMS) that must map to the correct tier; **negative cases**
  (conflicting OA-field vs JATS license) that must FLAG.
- **Bundle-segregation assertion** — a fixture corpus where a non-commercial article is
  deliberately mis-routed must **fail the build** in CI.
- **Patient-content detector fixtures** — synthetic case-report metadata with/without image &
  patient-media signals; flagged cases must be forced metadata-only.
- **Determinism check** — a clean rebuild from the pinned fixture snapshot must reproduce the
  manifest content hash.
- **Gold-set evaluation** — committed human-labeled gold set; cancer-type/study-type classifier
  scored against it, thresholds enforced in CI.

**Definition of Shipped.** A versioned corpus release (manifest + license-segregated bundles +
datasheet + provenance) that: (1) passes the license + cancer-guardrail gate with a committed
audit artifact showing **100% license correctness on the audit sample and zero tier/patient-content
leakage**; (2) is deterministically reproducible (manifest hash reproduces); (3) meets the
gold-set accuracy thresholds; and (4) is **accepted/used by a named beneficiary or published as a
citable Zenodo DOI with ≥ 1 verifiable external reuse** — recorded by the Steward as an
acceptance evidence artifact (`outcomes/<release-id>.json`). Producing the corpus is *not* shipped;
verified acceptance/reuse by a beneficiary is.

## Roadmap & milestones

**M0 — Foundation & cold-start (thin)**
- Goal: define the record model + gate, build a small **pilot corpus for one cancer subdomain**
  end-to-end (license-segregated, provenanced, datasheeted), and begin partner outreach.
- **Cold-start de-risking.** To avoid building a corpus nobody uses, the pilot is gated on a
  realistic acceptance path *before* building, in priority order: (a) an **informal channel** —
  a lab/advocacy/NLP contact who has agreed to look at a release; failing that, (b) a **self-serve
  fallback** — a versioned **Zenodo** record with a DOI we publish ourselves. The pilot must use
  one so M0 yields a real *accepted/citable* outcome, not a "produced, unused" one. Pilot
  subdomain chosen for tractable size and clear licensing (e.g. a single cancer type with strong
  CC-BY coverage).
- Exit criteria: (1) canonical record model + manifest schema + cancer-guardrail/license gate
  checklist published; (2) deterministic pilot build pipeline + validators green in CI with
  golden fixtures (incl. the bundle-segregation fail-test); (3) the gate applied to the pilot set
  with a committed audit artifact (100% license correctness on sample, zero leakage); (4) one
  pilot corpus released end-to-end with a datasheet and **accepted via informal channel or a
  Zenodo DOI** (acceptance evidence recorded) — or submitted with the blocker surfaced; (5) ≥ 1
  partner-outreach thread opened; (6) License+compliance reviewer named.

**M1 — Gate hardened + reproducible full build**
- Goal: make the license + guardrail gate rigorous and scale the pilot to a full reproducible
  oncology corpus build from a pinned snapshot.
- Exit criteria: (1) license-resolution cross-check (OA field vs JATS) + tiering codified with
  the negative/FLAG cases enforced in CI; (2) patient-content detector implemented with fixtures
  and applied across the build; (3) full deterministic build from a pinned snapshot with a
  recorded manifest hash; (4) license snapshot capture working (committed copy + SHA-256 +
  Wayback); (5) ≥ 1 confirmed consumer/partner **or** proven Zenodo release path.

**M2 — Metadata normalization & quality at scale**
- Goal: high-quality normalized metadata (cancer-type, study-type, aggregate-data signal),
  measured against a gold set, across the full corpus.
- Exit criteria: (1) MeSH→cancer-type taxonomy + study-type classifier meeting the gold-set
  thresholds (≥ 0.90 P / ≥ 0.85 R); (2) aggregate-vs-individual-data signal implemented and
  spot-audited; (3) full datasheet + data-quality report for the full corpus; (4) a second corpus
  release with measurable curation-quality improvement vs M1 baseline; (5) ≥ 1 verifiable external
  reuse event.

**M3 — Refresh, reuse & sustainability**
- Goal: keep the corpus current with PMC updates, handle retractions/withdrawn-OA articles,
  and demonstrate downstream reuse + a maintenance model.
- Exit criteria: (1) incremental refresh/sync process detecting new/updated/**retracted/
  removed-from-OA** articles and re-running the gate; (2) deprecation handling (articles that
  leave OA are removed from bundles, retained metadata-only with reason); (3) ≥ 1 named consumer
  using a release in their work (or ≥ 2 verifiable reuse events); (4) documented maintenance/refresh
  cadence + steward identified.

Dependencies: M1 depends on the M0 record model + gate; M2's normalization depends on M1's
full build; M3 depends on a body of accepted releases.

## Work breakdown

The itemized, schema-mapped backlog lives in `TASKS.md`, organized by the milestones above:
each milestone has a task table (`ID | Title | Type | Size | Risk | Deliverable | Depends on |
Reviewer`), acceptance criteria for the most important tasks, and a Definition of Done. A
sized-but-unscheduled backlog and one complete, schema-valid example Task JSON are included
there. The cancer-guardrail + license gate is the spine that every per-corpus task depends on.

## Governance, roles & stakeholders

- **Maintainer (Owner):** TBD — owns the pipeline, record model, taxonomy, and backlog.
- **License + compliance reviewer:** TBD (name TO BE SECURED) — mandatory, **non-skippable**
  gatekeeper for license tiering and the cancer guardrails; no release ships without this
  sign-off. Filled **before the M0 pilot release is reviewed** (blocking prerequisite, not a
  parallel hire). May rotate among ≥ 2 qualified reviewers, but ≥ 1 named qualified reviewer must
  exist at all times or the build/release halts. Until named, all tasks remain `verifiedNeed:false`.
- **Technical reviewer(s):** rotation verifying determinism, schema validity, validators, and CI.
- **Oncology / domain reviewer:** consulted on the inclusion query, cancer-type taxonomy, and
  patient-content heuristics; **required `high`-risk sign-off only** if any patient-facing
  derivative is ever proposed (out of scope here).
- **Steward (last-mile owner):** TBD — owns consumer/partner relationships and records acceptance
  + reuse (the "delivered" signal). Critical: Definition of Shipped is acceptance/reuse, not
  production.
- **Partner / requestor:** TO BE SECURED — named lab / advocacy program / NLP group, or the
  self-serve Zenodo fallback.

## Dependencies & integrations

- **Sources (pinned):** PMC Open Access Subset (NCBI OA Web Service + FTP file lists), Europe PMC
  REST API, PubMed metadata, MeSH vocabulary (C04). Official services only; no scraping; rate
  limits respected; no API keys committed.
- **Standards/specs (pinned — see Tech stack):** JATS, SPDX + Creative Commons license ids,
  Datasheets for Datasets (Gebru et al.), schema.org/Dataset for the manifest descriptor.
- **External archive (self-serve fallback):** Zenodo (versioned record + DOI).
- **Elyos pieces:** Task JSON schema (`packages/schema`), the donated-lane CLI workspace/PR flow
  (`packages/cli`), good-deed definition + refusal guardrails. No funded-lane/runner dependency
  (donated lane). *If* a large-scale extraction pass is later funded, it would run via
  `packages/runner` with a hard budget cap (see Backlog).

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| Mislabeling an article's license (e.g. NC treated as commercial) | Medium | High | Resolve from OA field **and** JATS, cross-check; conflict/missing = metadata-only; mandatory reviewer audit (100% on sample); record SPDX id+URL+snapshot | License+compliance reviewer |
| Non-commercial / no-license article leaking into the commercial bundle | Medium | High | Bundle build keyed on `licenseTier`; CI assertion **fails build** on any tier violation; fixture fail-test | Maintainer |
| Article with identifiable patient content republished in a bundle | Low | High | Patient-content screen on all case-reports/media; flagged → metadata-only; zero-tolerance gate; never re-identify | License+compliance reviewer |
| Accidentally touching controlled-access / individual-level data | Low | High | Sources restricted to PMC OA / Europe PMC OA; controlled-access references only *noted*, never followed; explicit non-goal | Maintainer |
| Cancer inclusion query too broad/narrow (poor curation) | Medium | Medium | Versioned MeSH query; gold-set precision/recall thresholds; oncology reviewer sanity-check | Maintainer |
| Non-deterministic build → unreproducible corpus | Medium | Medium | Pin snapshot + MeSH + spec versions; record manifest hash; CI rebuild check | Technical reviewer |
| Source articles retracted / removed from OA after release | Medium | Medium | Refresh/sync detects retractions/removals; remove from bundles, keep metadata-only with reason | Maintainer |
| No consumer secured → corpus produced but unused (fails "delivered") | Medium | High | M0 outreach + self-serve Zenodo fallback; steward role; `verifiedNeed:false` until secured | Steward |
| Scope creep into patient-facing summaries / advice | Medium | High | Explicit non-goal; any such work spun out as separate `high`-risk oncologist-+-advocate-reviewed project | Maintainer |
| Rate-limiting / ToS issues with NCBI/Europe PMC services | Low | Medium | Use official bulk file lists + APIs within documented limits; no scraping; cache pinned snapshot | Technical reviewer |

## Security & privacy

- **Threat surface is small** (no runtime service; corpus + metadata files + a build pipeline).
  Main surfaces are CI and the released artifacts.
- **Secrets handling:** the pipeline needs no credentials by default (PMC OA/Europe PMC are
  open). Any optional API token (e.g. higher NCBI rate limits) is supplied at runtime by the
  human and **never** written to logs, receipts, or committed files (per Elyos rules).
- **Patient privacy (dominant concern):** handled by guardrail #2 — identifiable-patient-content
  screen forces flagged articles to metadata-only; we never ingest controlled-access/individual-level
  data and never de-identify/re-identify. No individual-level health data is stored.
- **Abuse/misuse prevention:** refuse and flag any task that would steer the corpus toward
  re-identification, surveillance, building patient-targeting tools, laundering non-OA/non-commercial
  text as freely reusable, or producing unqualified medical advice. The corpus stays descriptive,
  source-cited, and license-true.

## Sustainability & maintenance

- **Post-delivery ownership:** the maintainer keeps the pipeline current with PMC OA / Europe PMC /
  MeSH / JATS changes; the steward maintains consumer relationships and records outcomes.
- **Refresh:** an incremental sync detects new/updated/retracted/removed-from-OA articles and
  re-runs the gate; releases are versioned with a changelog and a fresh manifest hash. Stale or
  drifted records become `maintenance` tasks.
- **Outcome tracking:** the steward records acceptance + verifiable external reuse against the
  success metrics, reviewed each milestone. Determinism + pinned snapshots make every past release
  auditable and rebuildable.

## Open questions

- Which named consumer (lab / advocacy program / NLP group) becomes the first confirmed partner,
  vs. relying on the self-serve Zenodo DOI fallback?
- Scope of the cancer filter: all neoplasms (C04) vs. a curated subset of high-need cancer types
  for the first full build? (Default: pilot one subdomain in M0, full C04 in M1.)
- How do we treat **CC-BY-ND** for *metadata extraction*? (Decided: facts/metadata yes; derivative
  full-text no — confirm with the License reviewer that fact extraction is acceptable for ND.)
- Where exactly is the license-text snapshot stored? (Default, mirroring sibling projects:
  committed copy + SHA-256 + Wayback URL; per-license-type, since CC license texts are shared.)
- Do we include **non-English** OA cancer articles in the first build, or English-first then expand?
- What is the precise, reviewer-approved definition of an "identifiable-patient-content" signal for
  the detector (sensitivity vs. over-flagging)?
- For the aggregate-vs-individual-data signal, is parsing the data-availability statement sufficient,
  or is full-text scanning needed (and acceptable under ND licenses)?

## References

- Elyos work rules — `C:\code\elyos\CLAUDE.md`
- Good Deed Definition + risk tiers — `C:\code\elyos\docs\good-deed-definition.md`
- Task JSON schema — `C:\code\elyos\packages\schema\src\schemas.ts`
- Portfolio roadmap (Track 8 cancer guardrails) — `C:\code\elyos\planning\ROADMAP.md`
- Sibling project (license/PII gate pattern) — `C:\code\elyos\planning\projects\open-data-datasheets\PLAN.md`
- PMC Open Access Subset + NCBI OA Web Service / FTP file lists (license fields)
- Europe PMC REST API + OA holdings
- MeSH (Medical Subject Headings), Neoplasms tree (C04)
- JATS (Journal Article Tag Suite); schema.org/Dataset
- Datasheets for Datasets (Gebru et al., 2018/2021)
- SPDX license list; Creative Commons CC0 / CC-BY / CC-BY-SA / CC-BY-ND / CC-BY-NC family
- Zenodo (archival DOI fallback)

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified during drafting and **applied** to the
plan above (and to `TASKS.md`). Each is concrete and reflected in the documents.

1. **Led the compliance section with the binding cancer guardrails** (open-access/aggregate/
   de-identified only; controlled-access + identifiable patient data out of scope), as required —
   guardrails appear *first* in "Data, licensing & compliance," not buried.
2. **Replaced the false "PMC OA = one license" assumption** with explicit **per-article license
   resolution and tiering** (commercial / non-commercial / no-derivatives / metadata-only).
3. **Added a bypass-proof bundle-segregation gate**: bundles keyed on `licenseTier` with a **CI
   assertion that fails the build** on any tier violation, plus a deliberate fixture fail-test.
4. **Handled the CC-BY-ND no-derivatives case explicitly** (redistribute unmodified only; metadata/
   facts yes, derivative full text no) — a subtlety most corpora get wrong.
5. **Handled legacy "no-CC-code" PMC articles and NIHMS author manuscripts** as metadata-only/
   excluded with no default-allow, instead of silently bundling them.
6. **Added a license cross-check** (OA-service license field **vs** JATS `<license>` metadata) with
   conflict → FLAG, so a single mis-tagged source field cannot cause leakage.
7. **Added an identifiable-patient-content detector** for case reports / media, forcing flagged
   articles to metadata-only — directly operationalizing guardrail #2.
8. **Made controlled-access handling concrete**: references are *noted* (`mentionsControlledAccess`)
   but never followed/ingested; dbGaP/EGA/biobanks named as hard out-of-scope.
9. **Encoded the prompt's named dataset cases** (TCGA/GDC open + GEO open = referenceable;
   COSMIC/OncoKB non-commercial/custom = flag) into the compliance section.
10. **Separated patient-facing work into a distinct `high`-risk project** (oncologist + advocate +
    "not medical advice"), keeping this corpus correctly `low` risk and research-facing.
11. **Made the corpus reproducible/deterministic**: pinned snapshot + MeSH/JATS/SPDX versions and a
    **recorded manifest content hash** with a CI rebuild check.
12. **Versioned the cancer inclusion query** as a committed artifact (MeSH C04 + supplements) with
    result counts, rather than an ad-hoc search.
13. **Added a human-labeled gold set** with enforced precision/recall thresholds for cancer-type and
    study-type classification, so "curation quality" is measured, not asserted.
14. **Added an aggregate-vs-individual-data signal** so reusers can see, per article, whether it
    reports aggregate or individual-level data and whether it mentions controlled-access sources.
15. **Adopted the canonical-record-first model** (manifest/bundles/datasheet are projections) to
    avoid maintaining parallel metadata formats inconsistently.
16. **Specified provenance to 100% completeness as a gate** (PMCID/PMID/DOI + license + source +
    snapshot version + MeSH version + attribution on every record).
17. **Specified the license-snapshot storage format** (committed copy + SHA-256 + Wayback URL,
    per-license-type since CC texts are shared) — mirroring the sibling project, no ambiguity.
18. **Defined per-bundle LICENSE manifests** and retention of each article's original CC license/
    attribution (no relicensing of sources); our own contributions CC-BY-4.0, code MIT.
19. **Made cold-start acceptance realistic** with a self-serve **Zenodo DOI fallback** so M0 can hit
    a real *accepted/citable* outcome without a third party.
20. **Set honest `verifiedNeed: false`** across tasks until a named consumer/partner is secured, and
    `requestor: TO BE SECURED`, per Elyos honesty conventions.
21. **Added retraction / removed-from-OA handling** in M3 (remove from bundles, keep metadata-only
    with reason) — corpora go stale and articles get withdrawn.
22. **Made all success metrics outcome-based and beneficiary-centric** (license correctness, zero
    leakage, accepted/reused), explicitly excluding "articles ingested" vanity counts.
23. **Constrained source access** to official OA services/file lists with rate-limit respect and
    **no scraping, no committed API keys** — closing a ToS/secrets risk.
24. **Named a non-skippable License+compliance reviewer gate** that must be filled *before* the M0
    pilot review, with a rotation rule to avoid a single-person bottleneck.
25. **Added a refusal/abuse section** specific to this corpus (no re-identification, no patient
    targeting, no laundering non-OA/non-commercial text as reusable, no medical advice).

---

## Review sign-off

**Reviewer pass (self-review for completeness + correctness).**

- **Structure:** All 17 required H2 sections present and in the spec order; metadata header matches
  the global convention (Status/Version/date/Owner/Lane). Appendix A (25 applied improvements) and
  this Review sign-off appended. ✔
- **Cancer guardrails:** "Data, licensing & compliance" leads with the binding guardrails
  (OA/aggregate/de-identified only; controlled-access + identifiable patient data out of scope;
  COSMIC/OncoKB flagged; TCGA-GDC-open/GEO-open referenceable; provenance on every assertion;
  patient-facing = separate `high`-risk). ✔
- **Schema correctness:** Re-checked `TASKS.md` Task JSON against `packages/schema/src/schemas.ts` —
  all required fields present; enums valid (`type`, `lane`, `priority`, `riskTier`, `deliverable`,
  `tokenEstimate`, `status`); `acceptanceCriteria` non-empty; `output` non-empty;
  `verifiedNeed:false`; donated tasks omit `fundedBudgetUsd` (only the funded backlog item includes
  it). Corrected during review: ensured the example JSON uses `deliverable: "dataset"` legitimately
  (this project *does* redistribute openly-licensed corpus artifacts, unlike the sibling
  data-documentation project) and that `domain` is an array. ✔
- **Risk tier consistency:** Corpus tasks are `low`; license-tiering/guardrail-judgement tasks are
  `medium` (human license/compliance judgement); no `high` tasks exist because patient-facing work
  is out of scope — consistent across PLAN and TASKS. ✔
- **Honesty:** `verifiedNeed:false` and `requestor: TO BE SECURED` throughout until a partner is
  confirmed; success metrics are outcome-based; the self-serve Zenodo fallback makes "delivered"
  reachable. ✔
- **Internal consistency:** Milestones in PLAN match milestone sections in TASKS; every per-corpus
  task depends on the gate; DoDs reference the same acceptance-evidence-artifact mechanism. ✔
- **Gaps flagged for humans (in Open questions):** first confirmed consumer; ND fact-extraction
  acceptability; precise patient-content-signal definition; cancer-type scope of first full build;
  multilingual inclusion. These require human/reviewer decisions and are surfaced, not guessed.

**Outcome:** Plan and tasks are internally consistent, schema-valid, and guardrail-compliant.
No blocking defects found in review; the items above are the intended human decision points.
