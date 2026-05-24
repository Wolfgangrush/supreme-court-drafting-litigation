# Changelog — `supreme-court-drafting`

All notable changes to this plugin are documented here. Versioning follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`) with an `-alpha` / `-beta` suffix during pre-release.

---

## [0.2.3-alpha] — 2026-05-25

### Explicit per-agent invocation of `pair_md_to_docx.sh`

v0.2.2 documented the output-pairing rule in `_drafting_common/SKILL.md` and relied on every agent picking up the rule by reference. v0.2.3 makes the invocation EXPLICIT in each agent's prompt — Reader, Format, Drafter, Verifier, Refiner, Overseer — so the pairing happens deterministically rather than depending on inherited-rule compliance.

### Changed

- **Reader prompt** — after writing `case-facts.md`, explicit `pair_md_to_docx.sh case-facts.md` invocation appended.
- **Format prompt** — after writing `format-shell.md`, explicit invocation appended.
- **Drafter prompt** — explicit invocation appended (Drafter already had a pandoc command from v0.2.1; the helper invocation is now also documented as the canonical path).
- **Verifier prompt** — after writing `verification-report.md`, explicit invocation appended.
- **Refiner prompt** — after writing `draft-v2.md`, explicit invocation appended.
- **Overseer prompt** — after writing `opposing-notes.md` and `final-draft.md`, two explicit invocations appended.

### Why the change

User feedback 2026-05-25: relying on each agent to inherit the rule from `_drafting_common` is not robust enough. The Drafter has the pandoc command spelled out and it works; the other 5 agents had only the inherited rule. Explicit per-agent invocation makes the pairing deterministic. Once every agent reliably outputs both `.md` and `.docx`, a pipeline run on any forum becomes itself a calibration probe — the advocate visually inspects the rendered `.docx` from each stage and identifies any per-forum formatting gaps, without needing 14 separate gold-standard pleadings upfront.

---

## [0.2.2-alpha] — 2026-05-24

### Output-pairing discipline — every `.md` paired with `.docx`

Advocates do not natively read Markdown. Every pipeline output artifact (case-facts.md from Reader, format-shell.md from Format, draft-v1.md from Drafter, verification-report.md from Verifier, draft-v2.md from Refiner, opposing-notes.md + final-draft.md from Overseer) is now paired with a corresponding `.docx` rendered using the same locked Word styles in the shipped reference.docx.

### Added

- **`pair_md_to_docx.sh`** — helper script in `skills/<base>/` that every agent calls after writing a `.md` output. Wraps the two-step pandoc + fix_docx_tables.py pipeline so every agent produces a paired `.docx` without re-implementing the conversion logic.
- **OUTPUT-PAIRING DISCIPLINE** section in `_drafting_common/SKILL.md` documenting the per-agent output-pairing map (Reader → case-facts.{md,docx}; Format → format-shell.{md,docx}; Drafter → draft-v1.{md,docx}; Verifier → verification-report.{md,docx}; Refiner → draft-v2.{md,docx}; Overseer → opposing-notes.{md,docx} + final-draft.{md,docx}).

### Why the change

User feedback from the 2026-05-24 EPFO test demonstrated that the QC pipeline output (`verification-report.md`, `opposing-notes.md`) was not accessible to the advocate in their normal Word workflow. The advocate explicitly stated: "every note … needs to be docx too." v0.2.2 closes this gap.

### Clarification — per-court formatting

v0.2.1 propagated a single Bombay HC Nagpur pleading-style reference.docx across all 14 plugins. The structural styling (TNR 14pt 1.5 spacing 4cm-left margin Heading 1/2/3/4) is broadly defensible for pleading-style plugins (HC / SC / Tax / Rent / MACT / Banking / Company / Consumer / Labour / Family / IP / District Court) because the court-specific differences (cause-title text, annexure prefix, statutory opening, AOR Certificate language) live in the case-type SKILL.md (Drafter content) not the reference.docx (style template). For SC the universal style is correct as the SC Registry mandate matches the HC convention (A4 + TNR 14pt + 1.5 spacing + 4cm left margin). Court-specific content (P-1/P-2 annexure prefix instead of ANNEXURE-A; SYNOPSIS + LIST OF DATES instead of just INDEX; AOR Certificate verbatim) is rendered by the Drafter from the case-type skill. Per-bench fine-tuning (e.g., Delhi HC double-spacing under Original Side Rules 2018; Punjab & Haryana watermarked paper) is achieved by supplying a case-folder reference.docx override.

For the two TRANSACTIONAL plugins (indian-contracts-drafting-litigation + indian-property-drafting-litigation), v0.2.1 wrongly applied the pleading-style reference.docx. Those two plugins now ship a transactional-instrument variant (TNR 12pt single-spaced, no spaced section headers, no underline on headings) under their own v0.2.2 release.

---

## [Unreleased]

### Pending before v0.2.0 stable
- Reader / Format / Drafter context-caching so the three stages share one citation + skill load
- Optional Haiku-routing for the Reader stage (extraction is pattern-match work)
- First Registry-validation pass on a sample SLP-Civil filing under the v0.2 render path

---

## [0.2.1-alpha] — 2026-05-24

### Filing-grade format calibration

Inherits the v0.2.1 calibration from `indian-hc-drafting-litigation` (anchored to an actual filed Bombay HC Nagpur Second Appeal pleading) and applies it to the SC pipeline.

### Added

- **`fix_docx_tables.py`** post-pandoc script at `skills/_sc_pleading_base/fix_docx_tables.py`. Forces column widths on every table in the rendered .docx — 5-col (Sr.No / P-N / Particulars / Date / Pgs) = 8/8/60/14/10; 4-col = 10/10/65/15; 3-col = 10/75/15; 2-col (Dates–Events / Synopsis) = 18/82. Locks first-row bold + centered. Drafter runs this as the final post-pandoc step.
- **Heading 2 with UNDERLINE** in reference.docx for spaced section headers (`S T A T E M E N T   O F   F A C T S`, `Q U E S T I O N S   O F   L A W`, `G R O U N D S`, `M A I N   P R A Y E R`, etc.) — bold + UNDERLINED + centered + letter-spacing.
- **Heading 3 + Heading 4 styles** in reference.docx for unspaced bold-underlined section headers and left-anchored bold-underlined headings.
- **Page numbers at TOP CENTER** (Bombay HC Nagpur convention, matching the gold-standard pleading).

### Changed

- **Drafter pandoc command** is now TWO steps (pandoc → .docx, then `fix_docx_tables.py`). Step 2 is non-negotiable; skipping it produces stacking-column table defects.
- **reference.docx Heading 2 style** now includes UNDERLINE (bold + UL + centered + letter-spacing).

---

## [0.2.0-alpha] — 2026-05-24

### Critical render-defect repair + pipeline-optionality

This release inherits the v0.2.0-alpha fixes from `indian-hc-drafting-litigation` and adapts them to the Supreme Court pipeline. The v0.1.0 render path produced filing-grade Markdown but the pandoc → `.docx` conversion failed SC Registry expectations on multiple counts (title not bold, section headers left-aligned, table column-headers wrapping vertically, party block leaking onto cover pages, ~6,200-word bloat on routine pleadings). This release repairs the render path.

### Added

- **Pre-customised SC `reference.docx`** at `skills/_sc_pleading_base/reference.docx` with locked Word styles (TNR 14pt body, 1.5 line spacing, 4cm left / 2.5cm right-top-bottom margins, Heading 1 bold centered, Heading 2 bold centered with letter-spacing for the `S T A T E M E N T   O F   F A C T S` effect, Heading 3 bold left, fixed table layout).
- **`build_reference_docx.py`** — reproducible build script for the shipped reference.docx.
- **MARKDOWN HEADING DISCIPLINE** section in `_sc_pleading_base/SKILL.md` and `_drafting_common/SKILL.md` documenting the Markdown → Word-style mapping the Drafter must follow.
- **VERBOSITY DISCIPLINE** section in `_drafting_common/SKILL.md` setting per-case-type word-count targets (SLP target 6,000–9,000 words, ceiling 12,000; Art 32 WP target 5,000–8,000 ceiling 10,000; Transfer Petition target 2,500–4,000 ceiling 5,000; Review/Curative target 3,500–5,500 ceiling 7,000).
- **PIPELINE-OPTIONALITY** section in `_drafting_common/SKILL.md` — Verifier / Refiner / Overseer are now OPTIONAL QC layers. Default exit point is after Stage 3 (Drafter); the AOR decides whether to invoke the QC stages.
- **COVER-PAGE DISCIPLINE** — SYNOPSIS, LIST OF DATES, LIST OF ANNEXURES each begin on `\newpage` and carry ONLY court header + case-number + short cause-title + section header + content + Counsel/AOR block. Full party block stays on the Main Petition cover only.

### Changed

- **Drafter agent prompt** flipped on the Markdown-heading rule: the v0.1.0 rule "❌ NEVER use markdown formatting in the .docx body (no headers prefixed `##`)" was incorrect — it banned the very mechanism that maps Markdown to Word styles. The rule is replaced with: "✅ Markdown headings ARE required at the section level — pandoc maps them to the locked Word-heading styles in reference.docx."
- **Pandoc invocation documented end-to-end** in `_drafting_common/SKILL.md` §OUTPUT FORMAT. The Drafter MUST use the shipped reference.docx; auto-generating one in the case folder is now banned (it was the v0.1.0 defect source).

### Removed

- The "Triple-verify is mandatory" framing from `_drafting_common/SKILL.md` — replaced with explicit OPTIONAL framing per §Pipeline-optionality.

### Cost / token-budget note

Running the full 6-agent pipeline burns approximately 600K tokens per draft, which can exhaust an AOR's Claude session limit in one drafting cycle. v0.2.0 makes Stages 4–6 OPTIONAL so a baseline Reader → Format → Drafter run (~280K tokens) is sufficient for routine pleadings. The optional QC stages remain available for high-stakes matters (SLP final hearing, Curative against a 5-Judge Bench, Art 32 against the Union).

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
