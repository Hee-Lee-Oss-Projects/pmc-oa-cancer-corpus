# pmc-oa-cancer-corpus

> Cancer research moves fast and is scattered across hundreds of journals. Researchers, data scientists, patient advocates, and tool builders who want a clean, license-clear, reproducible slice of the *  ·  **Risk tier:** low  ·  **Status:** planning

Cancer research moves fast and is scattered across hundreds of journals. Researchers, data scientists, patient advocates, and tool builders who want a clean, license-clear, reproducible slice of the **open-access cancer literature** to read, mine, or benchmark against have no canonical, well-documented corpus to start from. They must each re-derive one from PubMed Central (PMC), re-solve the per-article licensing puzzle, and re-handle the edge cases — duplicating fragile, error-prone work that most get subtly wrong (especially licensing and the handling of case reports).

**Definition of shipped:** datasheet + provenance) that: (1) passes the license + cancer-guardrail gate with a committed audit artifact showing **100% license correctness on the audit sample and zero tier/patient-content leakage**; (2) is deterministically reproducible (manifest hash reproduces); (3) meets

This is an **Hee-Lee Oss** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Platform: https://github.com/jdev1977/hee-lee-oss

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
hee-lee-oss browse
hee-lee-oss next --repo Hee-Lee-Oss-Projects/pmc-oa-cancer-corpus --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **low** — deeds are *delivered, not merged*; standard review applies.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
