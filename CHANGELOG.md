# Changelog — `supreme-court-drafting`

All notable changes to this plugin are documented here. Versioning follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`) with an `-alpha` / `-beta` suffix during pre-release.

---

## [Unreleased]

### Pending before v0.1.0 stable
- User-paste style references into all six `format-from-user.md` files
- First Registry-validation pass on a sample SLP-Civil filing
- Community feedback from AORs with active Supreme Court practice

---

## [0.1.0-alpha] — 2026-05-15

### Added

#### Plugin essentials
- `.claude-plugin/plugin.json` — plugin manifest
- `LICENSE` — MIT
- `NOTICE.md` — full provenance and privilege statement (10 sections)
- `README.md` — overview, skills inventory, agent pipeline, installation, roadmap
- `.gitignore` — standard exclusions plus SC-specific artifact paths

#### Shared infrastructure
- `skills/_drafting_common/SKILL.md` — anti-pollution rules, SC AI-use risk constraints, dual-citation discipline, Registry conventions, annexure `P-N` convention
- `skills/_sc_pleading_base/SKILL.md` — universal 17-section Supreme Court pleading skeleton, Cause Title template, AOR Certificate verbatim, Affidavit verification, annexure index, authority hierarchy

#### Case-type skills (6)
- `skills/slp-civil-draft/` — Special Leave Petition (Civil) under Article 136 + Order XXI SC Rules 2013 (Form 28)
- `skills/slp-criminal-draft/` — Special Leave Petition (Criminal) under Article 136 + Order XXII SC Rules 2013 (Form 32) with BNSS dual-citation
- `skills/writ-art32-draft/` — Writ Petition under Article 32 + Order XXXVIII; five writ types (Mandamus, Certiorari, Habeas Corpus, Prohibition, Quo Warranto); PIL credentials per *Balwant Singh Chaufal*
- `skills/transfer-petition-draft/` — Section 25 CPC (civil) / Section 406 BNSS (criminal); *Maneka Sanjay Gandhi* jurisprudence
- `skills/review-petition-draft/` — Article 137 + Order XLVII SC Rules + Order XLVII CPC; three-limb framing
- `skills/curative-petition-draft/` — *Rupa Ashok Hurra* foundational doctrine + Order XLVIII; Senior Advocate Certificate mandatory; *Yakub Memon* narrow reading

Each case-type skill includes a `SKILL.md` (structural metadata + hard rules + falsification triggers + provenance) and a `format-from-user.md` (style-reference template awaiting user paste).

#### Six-agent drafting pipeline
- `agents/reader/reader.md` — case folder ingestion, exhibit mapping (`P-N`), limitation computation, citation-discipline check
- `agents/format/format.md` — case-type SKILL.md loader, verbatim block insertion, dual-citation pre-flagging
- `agents/drafter/drafter.md` — fresh prose authoring, `[CITATION NEEDED]` discipline, pandoc `.docx` render
- `agents/verifier/verifier.md` — twelve SC-specific anti-hallucination checks (F1-F12) including AOR Certificate verbatim, P-N consistency, dual-citation enforcement, limb framing for Review/Curative, fundamental-right invocation for Article 32, custody-status presence
- `agents/refiner/refiner.md` — applies Verifier flags, restores verbatim blocks, SC Registry formatting enforcement, citation normalisation, AI-style marker removal
- `agents/overseer/overseer.md` — opposing-counsel and Bench lens, maintainability sweep, prayer realism, final draft

### Quality-gate audit (per `05-quality-gates.md` of the India-Legal Corpus Pipeline)

- Gate 1 · Copyright firewall — ✅ PASS (zero corpus prose transcribed; only public-domain statutory recitations verbatim)
- Gate 2 · Rule 36 BCI firewall — ✅ PASS (no marketing, placeholders for advocate name, free MIT, no commercial channel)
- Gate 3 · NOTICE.md doctrine — ✅ PASS (all placeholders, structural patterns trace to Constitution / SC Rules 2013 / public-domain authorities)
- Gate 4 · Statute currency — ✅ PASS (BNSS / BSA / BNS dual-citation enforced in `_drafting_common` + criminal skills)
- Gate 5 · Bench scope honesty — ✅ PASS (Supreme Court of India only; each skill declares its specific statutory authority)
- Gate 6 · Falsification triggers — ✅ PASS (each `SKILL.md` carries triggers and handling)

### Authorities cited (provenance trace)

**Constitutional:** Articles 14, 19, 20, 21, 22, 25, 32, 134, 134A, 136, 137, 142, 144, 145
**Statutory:** CPC 1908 (Sec 25, Orders XIX, XLVII), BNSS 2023 (Sec 406, 482, 528), Limitation Act 1963, Oaths Act 1969
**Court Rules:** SC Rules 2013 (Orders IV, IX, XXI, XXII, XXXVIII, XLVII, XLVIII, LVI; Forms 28, 32)
**Jurisprudence (cited as authority only, zero prose lifted):** Pritam Singh v. State AIR 1950 SC 169 · State of Uttaranchal v. Balwant Singh Chaufal (2010) 3 SCC 402 · Tilokchand Motichand v. H.B. Munshi (1969) 1 SCC 110 · Maneka Sanjay Gandhi v. Rani Jethmalani (1979) 4 SCC 167 · Rupa Ashok Hurra v. Ashok Hurra (2002) 4 SCC 388 · P.N. Eswara Iyer v. Registrar (1980) 4 SCC 680 · Northern India Caterers (India) Ltd. v. Lt. Governor of Delhi (1980) 2 SCC 167 · Lily Thomas v. Union of India (2000) 6 SCC 224 · Mohd. Arif v. Registrar (2014) 9 SCC 737 · Yakub Memon v. State of Maharashtra (2015) 9 SCC 552

### Provenance

Patterns encoded in this plugin trace to public-domain procedural authority only. Cross-validation against the India-Legal Corpus Pipeline (Phase 03 reports for SLP, Writ, Petition, Affidavit) was used to confirm *which patterns are used in practice* — never to lift *language*. The plugin operates under the `corpus-as-verifier-not-source` doctrine documented in the pipeline's build kit.

No drafted prose has been transcribed from the corpus or from any third-party advocate's drafts.
