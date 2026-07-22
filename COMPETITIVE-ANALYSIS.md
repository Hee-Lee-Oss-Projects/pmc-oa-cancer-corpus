# Competitive + Improvement Analysis — `pmc-oa-cancer-corpus`

> Scope: a curated, license-clear, fully-provenanced open-access **cancer** literature corpus +
> normalized metadata, drawn from the **PMC Open Access Subset** and **Europe PMC OA**, filtered
> to oncology by MeSH (C04), for text-mining / meta-research. Cancer-wide sibling of
> `ewing-literature-corpus`. Reviewed against PLAN.md / TASKS.md v0.1.0 (2026-06-28).

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually strong on licensing rigor and honesty (`verifiedNeed:false`, Zenodo
fallback, outcome-based metrics, bypass-proof tier gate). The gaps below are concrete.

**A. The license-tier mapping is subtly wrong on the commercial tier — and it matters.**
The plan's "Commercial-use tier" lists "CC0, CC-BY (4.0/3.0), CC-BY-SA, CC-BY-ND" and the
fixtures map `BY-ND → commercial`. NCBI's own grouping does put CC0/BY/BY-SA/BY-ND in the
`oa_comm` package, *but* "commercial" there means "commercial use of the package is allowed";
it does **not** mean every article is freely *modifiable*. CC-BY-SA imposes **share-alike** on
derivatives (a downstream ML reuser who fine-tunes and redistributes inherits SA obligations),
and CC-BY-ND **forbids derivatives entirely**. Folding BY-SA and BY-ND into one "commercial =
do-what-you-want" bucket invites exactly the downstream error the project exists to prevent. The
record model has `permitsDerivatives` and a `no-derivatives` tier, but the *prose tier table*
and the M0 fixtures conflate "commercial-use-allowed" with "derivatives-allowed." These are two
orthogonal axes (commercial-yes/no × derivatives-yes/no × share-alike-yes/no) and should be
modeled as **independent boolean flags**, not a single 4-value `licenseTier` enum. Source:
NCBI PMC OA Subset groupings (CC0/BY/BY-SA/BY-ND = oa_comm; NC family = oa_noncomm; rest =
oa_other) — https://pmc.ncbi.nlm.nih.gov/tools/openftlist/

**B. The OA-membership gate vs. NIHMS author manuscripts is underspecified — the single most
important factual risk.** The plan correctly says NIHMS-without-CC are metadata-only. But it
conflates two distinct PMC populations: (1) the **OA Subset** (CC/CC-like, reuse-permitted,
machine-readable license field) and (2) the much larger **Author Manuscript Collection** (NIH
Public Access Policy deposits) — which is *a different collection*, often carries **no copyright
statement or license terms at all**, and is **not** in the OA Subset. NCBI is explicit: "author
manuscripts added to PMC in compliance with a funder policy often do not include copyright
statements or license terms," and "not all articles in PMC are available for text mining or
other reuse; many are under copyright." The plan should state as a **hard invariant**: *only*
articles that are members of the OA Subset file lists (or Europe PMC's OA subset) are eligible —
membership in OA is the gate, license string is the *tiering within* it. A NIHMS PMCID that is
not in the OA file list must never be full-text-bundled even if a stray `<license>` tag looks
permissive. Sources: https://pmc.ncbi.nlm.nih.gov/about/copyright/ ,
https://pmc.ncbi.nlm.nih.gov/tools/openftlist/

**C. "phe_timebound" / time-bound and revoked licenses not handled.** PMC has a
`phe_timebound` package (publisher-granted temporary OA, e.g. pandemic-era) whose terms **expire**.
The plan's M3 retraction/removed-from-OA handling covers article *removal* but not *license
expiry/downgrade* while the article stays in PMC. A snapshot-pinned corpus can outlive a
time-bound grant; the refresh logic must detect license-field *changes*, not just membership
changes. Source: https://pmc.ncbi.nlm.nih.gov/tools/ftp/

**D. Europe PMC text-mining / annotation reuse terms are assumed, not pinned.** Europe PMC's OA
subset (~5.7M+ articles) is CC-or-similar and redistributable, but the **Annotations API**
(SciLite, 2B+ annotations) and the **gold-standard Annotations Corpus** have their own
attribution/reuse terms that are *not* automatically the same as the article licenses. The
backlog item `europepmc-020` correctly says "if annotation reuse terms verified compatible," but
the plan never records *which* Europe PMC product (article OA full text vs. annotations vs.
gold corpus) is in scope or pins its terms. Sources:
https://europepmc.org/annotationsapi , https://www.nature.com/articles/s41597-023-02617-x

**E. Corpus scale, versioning, and DOI strategy are vague.** The plan never states a *target
scale* (PMC OA cancer slice is plausibly 1–3M+ articles across C04). Metrics are all
quality/correctness; there is no honest size estimate, no statement of compute/storage footprint
for full-text bundles, and no decision on **per-release vs. concept (versioned) Zenodo DOIs**
(Zenodo supports both; the plan should commit to a concept-DOI with versioned children so
"cite the corpus" resolves to "latest" while old releases stay pinned). The "byte-identical
manifest hash" reproducibility target is also fragile: upstream PMC/Europe PMC are *mutable*
(articles get added, license fields corrected, content revised) — byte-identical rebuild only
holds against a **locally archived snapshot**, not a live re-pull. State that explicitly or the
metric is unachievable.

**F. Dedup is named but not specified.** "Dedup by DOI/PMID" (step 5) is too thin. PMC↔Europe
PMC overlap is *massive* (Europe PMC mirrors PMC), so the two sources will collide on nearly
every article; preprint↔published version pairs, multiple PMCIDs, and DOI-less older articles
all need rules. CORD-19's experience is instructive: "because of the diversity of paper sources,
duplication is unavoidable," and they kept a conservative identifier-cluster approach that still
retained dupes. The plan needs an explicit clustering/precedence rule (canonical source,
version preference, identifier-conflict handling). Source:
https://arxiv.org/pdf/2004.10706

**G. Cancer-topic classification accuracy targets may be miscalibrated.** Targets are ≥0.90
precision / ≥0.85 recall on a ≥200-article gold set. Two issues: (1) a MeSH-C04 filter is itself
the *de facto* classifier, so "cancer-type tag accuracy vs. gold set" partly measures **MeSH
indexing quality / NLM curation lag**, not the project's model — author-manuscript and recent
articles are often **not yet MeSH-indexed**, so a pure-MeSH filter silently under-recalls the
newest literature (the most valuable slice). (2) A 200-article gold set cannot support 0.90/0.85
claims across dozens of C04 cancer types with tight confidence intervals; either reduce the
granularity of the per-type claim or enlarge/stratify the gold set with reported CIs.

**H. Overlap/scope vs. `ewing-literature-corpus` is asserted but not operationalized.** The plan
calls itself the "general sibling" but never defines the **shared-pipeline contract**: is
`ewing-literature-corpus` a *filtered view* of this corpus (same record model, same gate, narrower
MeSH query), a *separate build*, or an *upstream* that this generalizes? Without a stated
relationship, the two will drift (divergent license logic is the worst-case — the exact failure
mode the gate exists to prevent). Recommend: declare the cancer-wide corpus the **canonical
pipeline + record model**, and Ewing (and any future disease-specific corpus) a *parameterized
query + datasheet* over the same gate.

**I. Redistribution-rights edge cases missing.** (1) **Supplementary materials** often carry
*different* licenses than the article body (or none) — the plan screens supplementary media for
patient content but never addresses supplementary-file *licensing*. (2) **Third-party figures**
embedded under "rights reserved" inside an otherwise CC-BY article (a known JATS reality) are not
handled — CC-BY at the article level doesn't license a re-used copyrighted figure. (3) The
"facts are not copyrightable, so ND metadata extraction is fine" claim is *broadly* right but
jurisdiction-sensitive (EU **sui generis database right** can attach to systematic extraction);
flag for the license reviewer rather than asserting.

**J. Minor:** "5.7M" Europe PMC OA figure and PMC scale should be cited with a snapshot date in
the datasheet; "low risk" tier is defensible but the *license-mislabeling* impact is High, so
the risk register's "low overall" framing slightly undersells the headline failure mode (the
plan does acknowledge this in prose — make the table reflect it).

---

## 2. Competitive landscape (real, cited)

**PMC Open Access Subset (NCBI) — the upstream / baseline.** Millions of full-text articles in
JATS XML, pre-grouped into `oa_comm` / `oa_noncomm` / `oa_other` license buckets, via FTP + OA
Web Service + an AWS Open Data mirror. *Strengths:* authoritative, free, bulk, machine-readable
license field, the literal source of truth. *Weaknesses/gaps:* domain-agnostic (no oncology
curation), three coarse buckets (no per-article SPDX, no commercial×derivative×share-alike
decomposition), no provenance/datasheet layer, raw bulk packages that every reuser re-derives
from. https://pmc.ncbi.nlm.nih.gov/tools/openftlist/ , https://pmc.ncbi.nlm.nih.gov/tools/ftp/

**Europe PMC (+ Annotations API / SciLite / gold corpus) — closest functional competitor.**
~5.7M+ OA full-text articles, a REST API, and 2B+ text-mined annotations (genes, diseases,
organisms, chemicals, GO terms) in W3C Open Annotation / JSON-LD, plus a human-annotated
gold-standard corpus. *Strengths:* mature, annotation-rich, standards-based, EU-funded
longevity, *already* serves much of what this project's M2 promises. *Weaknesses/gaps:* not
oncology-curated, annotation reuse terms separate from article licenses (compatibility not
guaranteed), no per-article commercial-vs-NC *redistributable bundle segregation* aimed at ML
training, no oncology datasheet. https://europepmc.org/annotationsapi ,
https://www.nature.com/articles/s41597-023-02617-x , https://blog.europepmc.org/2023/03/

**CORD-19 (Allen Institute for AI) — the *model* to emulate and learn from.** 400K+ COVID-19
papers as structured JSON (title/abstract/body/refs), weekly updates, full-text where licensed,
huge downstream adoption (TREC-COVID, hundreds of NLP papers). *Strengths:* the gold-standard
example of a domain-scoped, license-aware, widely-adopted literature corpus with a datasheet and
a clear citation path. *Weaknesses/gaps:* COVID-only (now static-ish), acknowledged
**duplication** ("unavoidable... conservative clustering retains a few duplicates"),
keyword-search false-positive/negative inclusion noise, PDF-parsing artifacts, and license
heterogeneity it surfaced but didn't fully segregate into clean reuse tiers — precisely the gaps
this project targets. https://arxiv.org/abs/2004.10706 , https://github.com/allenai/cord19

**S2ORC (Allen Institute for AI).** 8.1M open-access full-text papers + 80M+ abstracts/metadata
with parsed structure and linked references, released under **ODC-BY 1.0**. *Strengths:* huge,
cross-domain, structured full text, clean single redistribution license, NLP-ready. *Weaknesses:*
not biomedical-curated, the *single* ODC-BY wrapper papers over heterogeneous underlying article
licenses (a reuser can't tell commercial-OK from NC per source), no oncology metadata, no
patient-content screen. https://arxiv.org/abs/1911.02782 , https://github.com/allenai/s2orc

**PubTator Central / PubTator3 (NCBI).** Automated bioconcept annotations (genes, variants,
diseases, chemicals, species, cell lines) over 30M+ PubMed abstracts + ~3M PMC full-text TM
subset articles, in BioC-XML/JSON + REST + FTP. *Strengths:* deep, daily-updated NER/normalization
at scale, BioC-interoperable, free. *Weaknesses/gaps:* annotation layer, not a *license-segregated
redistributable corpus*; operates on the **PMC Text-Mining subset** (a different, broader
membership than the OA Subset — a licensing trap), no oncology curation or reuse-tier bundles.
https://academic.oup.com/nar/article/47/W1/W587/5494727 ,
https://www.ncbi.nlm.nih.gov/research/pubtator3/tutorial

**LitCovid (NCBI).** Curated, daily-updated COVID-19 literature hub with human-reviewed topic
categories. *Strengths:* the proof that *human-curated, single-disease* literature hubs deliver
real value and adoption. *Weaknesses:* COVID-only, metadata/topic hub (not a full-text
redistributable license-tiered corpus). https://www.ncbi.nlm.nih.gov/research/coronavirus/ ,
https://academic.oup.com/nar/article/49/D1/D1534/5964074

**OpenAlex (OurResearch).** Hundreds of millions of works/authors/topics, fully **CC0**, free
REST API + fortnightly data dumps, no rate limits. *Strengths:* unbeatable for metadata, topics,
OA-status, citation graph; CC0 removes all reuse friction. *Weaknesses/gaps:* **metadata/index,
not full text** — no JATS bodies, no license-segregated full-text bundles, topic tags are
algorithmic (not MeSH/oncology-grade). A complement, not a competitor, for full-text mining.
https://docs.openalex.org/ , https://en.wikipedia.org/wiki/OpenAlex

**BLURB / BioASQ / BC5CDR / NCBI-Disease / CRAFT — biomedical NLP benchmarks & gold corpora.**
BLURB = 13 datasets / 6 tasks over PubMed; BC5CDR = 1,500 PubMed articles (chemical-disease
relations); CRAFT = 97 full-text OA articles with rich semantic+syntactic annotation; CoMAGC =
gene-cancer relation corpus; TREC Precision Medicine pilots. *Strengths:* high-quality,
expert-annotated, the *evaluation* standard. *Weaknesses/gaps:* small, task-specific, mostly
abstracts, not designed as a broad reusable cancer *corpus* — they benchmark models, they don't
supply a license-clean oncology training/retrieval base. https://microsoft.github.io/BLURB/tasks.html ,
https://huggingface.co/datasets/bigbio/blurb

**The Pile / Hugging Face biomedical corpora (e.g. `pmc/open_access`, `allenai/cord19`,
`bigbio/*`).** *Strengths:* convenient, ML-pipeline-native (Parquet/datasets), large. *Weaknesses/
gaps:* exactly the cautionary tale — The Pile mixed licenses and later had to **remove components
under legal pressure** (Books3 via DMCA/Rights Alliance), illustrating the cost of *not* doing
per-source license segregation up front; HF biomedical dumps are typically un-tiered re-uploads
of PMC OA with no commercial/NC separation or provenance. https://en.wikipedia.org/wiki/The_Pile_(dataset) ,
https://huggingface.co/datasets/pmc/open_access

---

## 3. Gaps we can fill

No existing resource is simultaneously: **(a) cancer-wide and oncology-curated** (vs. CORD-19's
COVID-only, LitCovid's COVID-only, BLURB's task slices), **(b) per-article license-resolved and
reuse-tier-*segregated*** into clean commercial / non-commercial / no-derivatives /
metadata-only bundles (vs. S2ORC's single ODC-BY wrapper, PMC's 3 coarse buckets, HF's un-tiered
dumps, The Pile's license soup), **(c) provenance-complete + datasheeted** (Gebru-style, with
license snapshots + Wayback + SHA-256), **(d) deterministically reproducible from a pinned
snapshot**, and **(e) patient-content-screened** (the case-report identifiability hazard nobody
else operationalizes). Concrete fillable gaps:

1. **A "safe-to-train-on commercially" oncology bundle** — the artifact ML builders actually
   need and currently must assemble themselves (and usually get wrong). This is the headline gap.
2. **Per-article SPDX + decomposed reuse flags** (commercial × derivatives × share-alike ×
   redistribution) instead of NCBI's three buckets or S2ORC's one license.
3. **An auditable license-correctness record** (sample audit at 100%, zero-leakage gate) — a
   *trust* artifact no competitor publishes.
4. **OA-membership-true ingestion** that never confuses the OA Subset with the broader PMC
   text-mining subset / Author Manuscript Collection (the trap PubTator's TM-subset and naive HF
   dumps fall into).
5. **Oncology metadata layer** (MeSH→cancer-type taxonomy, study-type, aggregate-vs-individual
   signal) that Europe PMC/PMC don't provide and OpenAlex only approximates.
6. **Patient-identifiability screen** for case reports/media — a genuinely novel safety layer.

---

## 4. Differentiators to win

- **License segregation as a first-class, bypass-proof product** — CI fails the build if any
  article lands in a disallowed bundle; a deliberate mis-route fixture *must* fail. No competitor
  ships a corpus whose central guarantee is "this bundle is provably safe for *your* reuse tier."
- **Provenance + datasheet + reproducible-from-snapshot** as standard, not afterthought — turns
  the corpus into a *citable, auditable* scientific artifact (Zenodo concept-DOI), closing
  CORD-19's "trust but can't fully audit license" gap.
- **Cancer-wide curation with a shared, parameterized pipeline** — the corpus *is* the pipeline;
  `ewing-literature-corpus` becomes a query+datasheet over the same gate. **Win vs.
  disease-specific siblings:** one hardened license/guardrail gate, many disease views, zero
  divergent license logic. Ewing wins on depth (rare-disease completeness, expert curation);
  cancer-wide wins on breadth, reuse, and amortized engineering. Position them as
  **canonical-pipeline (cancer-wide) + flagship-instance (Ewing)**, explicitly cross-linked.
- **Patient-content safety screen** — a differentiator on *ethics/safety*, not just convenience,
  which matters to advocacy and IRB-sensitive consumers.
- **Honest "delivered, not merged"** — acceptance/reuse is the success metric, with a self-serve
  Zenodo fallback so success doesn't hinge on a third party. This is a credibility differentiator
  vs. academic corpora that ship-and-forget.

---

## 5. Claude API leverage

**Where Claude clearly helps (with human/CI guardrails):**

1. **Cancer-topic & study-type classification (assistive, second-pass).** Use Claude to classify
   study-type (review / RCT / observational / preclinical / case-report / methods) and to
   *recover recall* on recent or not-yet-MeSH-indexed articles where the C04 filter under-recalls
   (gap G). Run it as a **second opinion** that flags disagreements with MeSH for human review,
   not as the sole labeler. Anthropic's prompt-caching makes per-abstract classification cheap at
   corpus scale; the funded-lane `extract-021` item is the right home (hard budget cap).
2. **NER / annotation enrichment.** Claude can extract cancer types, genes, biomarkers,
   interventions, and outcomes from OA-licensed full text to bootstrap annotations — cross-checked
   against PubTator/Europe PMC where they overlap, used to *extend* coverage where they don't.
3. **Dedup / record linkage.** Claude is strong at fuzzy matching preprint↔published, multi-PMCID,
   and DOI-less-older-article clustering that string keys miss — as a *candidate proposer* feeding
   a deterministic confirm step (gap F).
4. **Metadata normalization.** MeSH→cancer-type taxonomy mapping, journal-name normalization,
   author byline cleanup, data-availability-statement parsing (aggregate-vs-individual signal).
5. **Gold-set building (force-multiplier, not authority).** Claude drafts candidate labels and
   rationales to make human annotators faster; **humans adjudicate**, and inter-annotator
   agreement (IAA) is computed on the *human* labels.
6. **Patient-content triage drafting.** Claude can surface candidate identifiability signals
   (clinical photos, full-face images, rare-detail case reports) for the reviewer — flag-only.

**Where Claude must NOT decide (hard lines):**

- **License / redistribution determinations** — these are read from the authoritative OA license
  field × JATS `<license>` and **human-verified**; Claude may *summarize* a license but never set
  `licenseTier`, `permitsRedistribution`, or OA-membership. A model hallucinating a license is the
  project's worst failure.
- **OA-membership** — a deterministic set-membership check against the official file lists, never
  an LLM judgment.
- **Annotation/classifier ground truth** — gold labels require human IAA + oncology-expert
  validation; Claude outputs are inputs, scored *against* the gold set, never the gold set.
- **No fabricated citations / provenance** — every record field traces to a source field;
  Claude-derived tags must carry "derived-by-model" provenance and never invent PMCIDs/DOIs.
- **No patient re-identification** — Claude is used to *flag for exclusion*, never to extract,
  enhance, link, or de-/re-identify.

---

## 6. Ten concrete optimizations

1. **Replace the 4-value `licenseTier` enum with orthogonal flags** (`commercialUse`,
   `permitsDerivatives`, `shareAlike`, `permitsRedistribution`) + a derived bundle key. Fixes the
   BY-SA/BY-ND conflation (finding A) and makes the share-alike obligation explicit.
2. **Make OA-Subset membership a hard, deterministic precondition** separate from license parsing,
   with an explicit invariant and a fixture proving a non-OA NIHMS PMCID is rejected (finding B).
3. **Add a `licenseValidFrom/validUntil` + time-bound (`phe_timebound`) handler** and refresh
   logic that detects license-field *changes*, not just membership changes (finding C).
4. **Pin and record *which* Europe PMC product** (OA full text vs. SciLite annotations vs. gold
   corpus) and its terms in `specVersions`; gate annotation reuse explicitly (finding D).
5. **Specify the dedup/record-linkage algorithm** (canonical-source precedence, version
   preference, identifier-conflict rules, Claude-assisted candidate clustering + deterministic
   confirm) with a dup-rate metric in the data-quality report (finding F).
6. **Commit to a Zenodo concept-DOI with versioned children**; redefine reproducibility as
   "byte-identical against the *archived* snapshot," and archive the snapshot itself (finding E).
7. **Add supplementary-file and embedded-third-party-figure license handling** (separate license
   resolution for supplements; default-exclude third-party "rights reserved" figures) (finding I).
8. **Publish an honest scale + footprint estimate** (expected article count per tier, storage,
   build time) in the datasheet, with a snapshot-dated PMC/Europe PMC size citation.
9. **Emit a Croissant ML metadata descriptor** (backlog `croissant-023`) *from M1*, not later —
   it's the lingua franca for HF/Kaggle ML reuse and a cheap adoption multiplier.
10. **Ship an MCP server / read API over the manifest** (see §7) so consumers query the corpus
    by cancer-type/license-tier without downloading multi-GB bundles — lowers adoption friction,
    the project's biggest risk (no consumer).

---

## 7. Parallel & perpendicular spin-offs

- **Reusable license-clean corpus pipeline (the real platform play).** Generalize the gate +
  record model + datasheet into a domain-agnostic toolkit; cancer and Ewing are the first two
  *instances*. Future instances (rare diseases, pediatric oncology, a given biomarker) are
  parameterized queries — near-zero marginal engineering. This is the highest-leverage spin-off.
- **`ewing-literature-corpus` as flagship-instance** — refactor it (and any disease corpus) to
  consume this pipeline; shared gate eliminates divergent license logic.
- **`oncogene-knowledge-graph`** — feed the (commercial-tier, OA) full text + Claude/PubTator NER
  into a gene/variant/cancer-type/biomarker KG; the corpus is the license-clean substrate.
- **`systematic-review-assist`** — the deduplicated, provenance-rich, study-type-classified set is
  an ideal PRISMA starting corpus; ship a "screening export" projection.
- **`biomarker-extraction`** — a funded-lane structured-extraction pass (biomarker → cancer-type →
  outcome) over commercial-OK full text, with a hard budget cap (`extract-021`).
- **Corpus MCP server** — serve manifest queries, per-tier slices, and provenance over MCP so
  Claude/agents retrieve license-clean cancer literature on demand (RAG without re-deriving the
  license puzzle). Strong fit with Hee-Lee Oss's agent-facing thesis.
- **License-tiering microservice / SPDX-for-PMC** — expose the OA-field×JATS cross-check +
  decomposed flags as a standalone service others can call; potential community standard.

---

## 8. Open questions for the maintainer

1. **`licenseTier` model:** will you decompose into orthogonal flags (commercial / derivatives /
   share-alike) or keep the 4-value enum? (Affects BY-SA/BY-ND correctness — finding A.)
2. **OA-membership invariant:** is "must be in the OA Subset file list" stated as a hard gate
   distinct from license parsing, with NIHMS-non-OA explicitly rejected? (Finding B.)
3. **Scale & footprint:** what's the honest expected article count (per tier) and storage/build
   budget for the full C04 build? Is full-text redistribution feasible at that scale, or
   metadata-first with on-demand full-text?
4. **Pipeline relationship to `ewing-literature-corpus`:** canonical-pipeline + instance, or two
   independent builds? Who owns the shared gate?
5. **Europe PMC scope:** article OA full text only, or also SciLite annotations / the gold corpus
   — and are their reuse terms verified compatible?
6. **Recall vs. MeSH lag:** do you accept under-recall of not-yet-indexed recent articles, or add
   a Claude-assisted recall layer with human confirmation?
7. **Reproducibility definition:** byte-identical against a *locally archived* snapshot (not a
   live re-pull) — confirmed? Where is the snapshot archived, and at what cost?
8. **Time-bound / revoked licenses:** does refresh detect license *downgrades* (`phe_timebound`
   expiry), not just removals?
9. **EU database right / sui-generis:** has the license reviewer confirmed systematic fact
   extraction from ND/NC articles is clear in target jurisdictions?
10. **First consumer:** which named lab/advocacy/NLP group, vs. committing to the Zenodo
    concept-DOI fallback for M0?
