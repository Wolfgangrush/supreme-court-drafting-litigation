# supreme-court-drafting

> **Open-source Claude-compatible plugin for drafting pleadings before the Supreme Court of India.**
>
> Six-agent drafting pipeline · six SC case-type skills · Supreme Court Rules 2013 conventions · strict citation-fabrication firewall.
>
> Released under MIT. Open infrastructure for the legal community. No commercial engagement offered through this repository — see Disclaimer below.

> ⚠️ **AI can make mistakes. Always verify the output.**
>
> This software generates assistive drafts and suggestions only. Every legal claim, citation, statute reference, procedural step, deadline calculation, and ground of relief must be independently verified by a qualified human practitioner before filing, advising a client, or relying on the output. The publisher accepts no liability for outputs used without verification.

> 🛡️ **Privacy primitive — Reader agent invokes the gateway:** This drafting plugin's **Reader agent** (the first agent in the 6-agent pipeline) calls [pseudonymisation-gateway](https://github.com/Wolfgangrush/pseudonymisation-gateway) (MIT · wolfgang_rush) on the user's case folder BEFORE any cloud-LLM call. Real client names · government IDs · case numbers · phone numbers · currency amounts are replaced with placeholders (`[PERSON_1]` · `[AADHAAR_1]` · `[CASE_NO_1]` · etc.) in a session-scoped in-memory token map that never touches disk. Downstream agents (Format · Drafter · Verifier · Refiner) work entirely on the sanitized text. The **Overseer agent** (the final agent) calls `desanitize()` to restore real values in the final pleading before it reaches the file system. Cloud LLM vendors never see your client's real PII.


## 🚀 Install — wolfgang_rush marketplace

This plugin is part of the [wolfgang_rush plugin family](https://github.com/Wolfgangrush/wolfgang-rush-marketplace) — 14 Indian-court drafting plugins distributed via one Claude Code marketplace.

**Via Claude Code (CLI) — recommended for the plugin family:**

```bash
/plugin marketplace add Wolfgangrush/wolfgang-rush-marketplace
/plugin install supreme-court-drafting@wolfgang-rush
```

**Via Claude Desktop:** see the [Installation](#installation) section below for the `.zip` upload flow.

---

---

## Table of contents

1. [What this plugin does](#what-this-plugin-does)
2. [Case-type skills (full inventory with statutory authority)](#case-type-skills-full-inventory)
3. [The 6-agent drafting pipeline (what each agent does)](#the-6-agent-drafting-pipeline)
4. [Installation](#installation) — Claude Desktop application
5. [Your first SLP — step-by-step walkthrough](#your-first-slp--step-by-step-walkthrough)
6. [The Supreme Court Rules 2013 — what this plugin encodes](#the-supreme-court-rules-2013--what-this-plugin-encodes)
7. [Why MIT License (and not Apache 2.0, GPL, or anything else)](#why-mit-license)
8. [Sibling plugins (wolfgang_rush legal-tech family)](#sibling-plugins)
9. [Why this exists](#why-this-exists)
10. [Roadmap](#roadmap)
11. [Contributing](#contributing)
12. [Contact](#contact)
13. [Author and brand](#author-and-brand)
14. [Provenance and privilege statement](#provenance-and-privilege-statement)
15. [Disclaimer and Bar Council of India Rule 36 compliance](#disclaimer-and-bar-council-of-india-rule-36-compliance)
16. [License](#license)

---

## What this plugin does

This plugin lets an **Advocate-on-Record** (AOR) — or an advocate drafting under AOR supervision — point Claude at a case folder on disk and obtain a complete Supreme-Court pleading in `.docx` form, formatted to Supreme Court Rules 2013 + Registry Practice Directions, with the strict citation-fabrication firewall the Supreme Court has publicly cautioned every advocate to apply when using AI tools.

The pipeline is six agents running in sequence:

1. **Reader** — walks the case folder, extracts facts with a per-document audit log, identifies annexure candidates (`P-N` ordering), and halts if any required statute / judgment PDF is missing. The Reader will not let the pipeline proceed on incomplete inputs.
2. **Format** — loads the case-type-specific SC skill template (e.g. `slp-civil-draft`, `writ-art32-draft`), maps the extracted facts into the format placeholders, and computes the limitation window for the case type.
3. **Drafter** — writes the actual pleading. Synopsis + List of Dates (mandatory and Registry-checked per Practice Direction), Statement of Facts (chronological, narrative form, not bullet-point), Questions of Law, Grounds, Main Prayer, Interim Prayer, Annexure Index, AOR Certificate (verbatim from Order IV Rule 1(c)), Supporting Affidavit (Form 47 / Order 19 CPC), and any accompanying applications the facts require.
4. **Verifier** — anti-hallucination firewall **plus** citation-discipline firewall. The Verifier flags fabricated dates, mis-cited sections, orphan annexure markers, unsupported assertions, and — critically — any case-citation in the draft that does not trace back to a user-supplied source. Hallucinated case names + citations are the single biggest reputational risk in SC practice today; the Verifier is designed to catch them before the AOR sees the draft.
5. **Refiner** — applies Verifier flags, polishes language to SC Registry-formal register, enforces SC formatting (A4, Times New Roman 14, 1.5 line spacing, 4 cm left margin per Registry Practice Direction), removes AI-style markers.
6. **Overseer** — reads the polished draft with an opposing-counsel lens. Flags weak prayers, attackable defects, missing limbs, contradictory facts. Writes `opposing-notes.md` for the AOR to harden the draft before signature.

The output is what an Advocate-on-Record would file. **Not a template. Not a checklist. A pleading.**

---

## Case-type skills (full inventory)

The plugin ships **six case-type skills**, each grounded in the statutory authority and SC Rules 2013 reference below.

### 1. `slp-civil-draft` — Special Leave Petition (Civil)

**Statutory authority:** Article 136 of the Constitution + Order XXI of the Supreme Court Rules 2013 (Form 28 prescribed format). **Use case:** challenge to a final judgment, decree, determination, sentence, or order in any cause or matter passed by any High Court or any tribunal in India, on the civil side. **Facts the skill asks for:** the impugned judgment + its date, the cause-title chain up to the HC, the limitation position (90 days from the impugned judgment, or 60 days from refusal of HC certificate under Article 134A), the substantial questions of law of general importance, the prior litigation history. **Output:** complete SLP (Civil) with Synopsis + List of Dates + Questions of Law + Grounds + Main Prayer (leave to appeal + the relief sought if leave is granted) + Interim Prayer + AOR Certificate + Supporting Affidavit + the Section 4 / 5 limitation declaration where condonation is sought.

### 2. `slp-criminal-draft` — Special Leave Petition (Criminal)

**Statutory authority:** Article 136 of the Constitution + Order XXII of the Supreme Court Rules 2013 (Form 32 prescribed format). **Use case:** challenge to a final judgment, sentence, or order in any criminal cause or matter passed by any High Court or any tribunal exercising criminal jurisdiction. **Facts the skill asks for:** the impugned judgment + its date, the trial and HC procedural history, the conviction / acquittal under challenge, the sentence under challenge, the limitation position, the questions of law (procedural and substantive), the bail / custody position. **Output:** complete SLP (Criminal) with the same architecture as SLP (Civil), plus the conviction-specific Grounds (Section 304 IPC sustainable on evidence? appreciation of evidence by HC? sentencing principles? etc.) and the bail-pending-SLP application where the case requires.

### 3. `writ-art32-draft` — Writ Petition under Article 32

**Statutory authority:** Article 32 of the Constitution + Order XXXVIII of the Supreme Court Rules 2013. **Use case:** direct writ to the Supreme Court for enforcement of fundamental rights. **Facts the skill asks for:** the fundamental right violated (Article 14 / 15 / 16 / 17 / 19 / 21 / 21A / 25–28 / etc.), the State action or inaction constituting the violation, the respondent authority, the prior representations made (if any), the prayer. **Output:** complete Article 32 petition with the constitutional limbs front-loaded as the SC expects, the standing established, the relief specifically tied to a writ (mandamus / certiorari / habeas corpus / prohibition / quo warranto).

### 4. `transfer-petition-draft` — Transfer Petition

**Statutory authority:** Section 25 of the Code of Civil Procedure 1908 (for civil) / Section 406 of the Bharatiya Nagarik Suraksha Sanhita 2023 (corresponding to Section 406 CrPC 1973, for criminal) + Order XL of the Supreme Court Rules 2013. **Use case:** transfer of a suit / appeal / proceeding from a court in one State to a court in another State, or from one High Court to another. **Facts the skill asks for:** the proceeding to be transferred (court, case number, parties), the transferee forum sought, the grounds (territorial inconvenience / threat to fair trial / convenience of witnesses / connected matters), the supporting case-law. **Output:** complete Transfer Petition with the well-settled SC grounds — *Maneka Sanjay Gandhi v. Rani Jethmalani (1979) 4 SCC 167*; *Subramaniam Swamy v. Ramakrishna Hegde (1989) Supp 2 SCC 633*; *Anindita Das v. Srijit Das (2006) 9 SCC 197* — mapped to the facts.

### 5. `review-petition-draft` — Review Petition

**Statutory authority:** Article 137 of the Constitution + Order XLVII of the Code of Civil Procedure 1908 + Order XLVII of the Supreme Court Rules 2013. **Use case:** review of an SC judgment on the grounds of (i) discovery of new and important matter or evidence which was not within the petitioner's knowledge or could not be produced by the petitioner when the order was made, despite due diligence, or (ii) mistake or error apparent on the face of the record, or (iii) any other sufficient reason analogous to the foregoing. **Facts the skill asks for:** the judgment under review, the specific ground invoked (with full elaboration), the new evidence (if any) and why due diligence did not yield it earlier, the error apparent (if any). **Output:** complete Review Petition. **Important:** Review Petitions are, **as a rule, decided in chambers by circulation without oral hearing** (Order XLVII Rule 3 SC Rules 2013) — the petition is drafted as a complete written argument in itself, anticipating no oral hearing.

### 6. `curative-petition-draft` — Curative Petition

**Statutory authority:** the doctrine evolved by a Constitution Bench in **Rupa Ashok Hurra v. Ashok Hurra (2002) 4 SCC 388** + Order XLVIII of the Supreme Court Rules 2013. **Use case:** rare and exceptional cases where, after dismissal of the Review Petition, gross miscarriage of justice or violation of natural justice requires reopening — only when there is variation between the curative ground and the violation of principles of natural justice / abuse of process / a Judge having an apparent bias whose participation in the bench was unknown to the petitioner. **Facts the skill asks for:** the judgment, the Review Petition + its dismissal, the curative-specific ground (i.e. the gross miscarriage / natural-justice violation, not a relitigation of merits), and the **certification by a Senior Advocate** that the curative petition is fit to be filed (Order XLVIII Rule 2 SC Rules 2013 mandates this). **Output:** complete Curative Petition. Like the Review, the Curative is also **decided in chambers without oral arguments by default**, unless the Court directs otherwise. The Drafter writes the petition as a self-contained written argument.

### Shared infrastructure skills

- **`_drafting_common`** — anti-pollution rules, encoding standards, language conventions, AI-style-marker blacklist, SC-specific citation-discipline rules (no AI-generated case citations under any circumstance), common phrases.
- **`_sc_pleading_base`** — universal Supreme Court pleading skeleton (cause title, parties block, prayer template, Advocate-on-Record Certificate verbatim from Order IV Rule 1(c), Supporting Affidavit template per Form 47 / Order 19 CPC, mandatory declarations per the SC Rules 2013, annexure index template, Synopsis template, List of Dates template).

---

## The 6-agent drafting pipeline

The plugin is built on the **Anthropic Agent SDK** convention — six markdown agent files (`agents/<name>/<name>.md`) with YAML frontmatter declaring `name`, `description`, and `allowed-tools`. Each agent is invoked in sequence on a case folder and reads/writes specific files.

### 1. `reader` — first agent

**Reads:** every file in the case folder (PDFs, DOCXs, scanned images via OCR where present, emails, notes) + the case-type skill's `case-facts-questions.md`.
**Writes:** `case-facts.md` (with per-document audit log) + `annexure-candidates.md` (mapping documents to `P-N` annexure slots) + `missing-laws.md` (halts the pipeline if any required statute or referred judgment PDF is not supplied).
**Why it exists:** SC pleadings rest on a chronological List of Dates, an annexure-paginated record, and traceable citations. The Reader establishes all three from the source documents *before* drafting begins, so the Drafter has nothing to invent.

### 2. `format` — second agent

**Reads:** `case-facts.md` (from Reader) + `skills/<case-type>-draft/SKILL.md` + `skills/<case-type>-draft/format-from-user.md` + `skills/_sc_pleading_base/SKILL.md`.
**Writes:** `format-shell.md` — the case-type-specific template with all facts mapped to placeholders, and the limitation window computed (90 days for SLP from the impugned judgment, 30 days for Review from the judgment, etc.).
**Why it exists:** the Drafter should not have to figure out which Form to use, which limitation rule applies, or which mandatory declarations need to be inserted. The Format agent does this pre-substitution so the Drafter focuses on the narrative.

### 3. `drafter` — third agent

**Reads:** `case-facts.md` + `format-shell.md` + `skills/<case-type>-draft/SKILL.md` + `skills/_sc_pleading_base/SKILL.md` + the user-supplied law PDFs in `case-folder/laws/`.
**Writes:** `draft-v1.md` and `draft-v1.docx` — a complete `.docx` ready for the Verifier. The Drafter writes Synopsis + List of Dates + Statement of Facts + Questions of Law + Grounds + Main Prayer + Interim Prayer + Annexure Index + AOR Certificate + Supporting Affidavit + accompanying applications.
**Why it exists:** this is the actual drafting brain. **It does NOT invent case citations** — every citation in the draft must trace to a user-supplied source, or it is written as a `[CITATION NEEDED]` placeholder for the AOR to fill manually.

### 4. `verifier` — fourth agent

**Reads:** `draft-v1.md` + `case-facts.md` + law PDFs in `case-folder/laws/`.
**Writes:** `verification-report.md` flagging:

- **Fabricated dates** — a date in the draft that does not appear in `case-facts.md`
- **Mis-cited sections** — a statute section in the draft that does not match the law PDF supplied
- **Orphan annexure markers** — `ANNEXURE P-7` referenced in the body but not in the annexure list
- **Unsupported assertions** — a claim in the draft without corresponding fact in `case-facts.md`
- **Hallucinated case citations** — a case name + citation in the draft that does not trace to a user-supplied source. **This is the most important Verifier check.** The Supreme Court has publicly cautioned advocates against AI-generated content with fabricated citations; the plugin enforces that caution structurally.

**Why it exists:** SC practice runs on the assumption that what an AOR signs is true. The Verifier ensures the AOR is signing something the Verifier has actually checked.

### 5. `refiner` — fifth agent

**Reads:** `draft-v1.md` + `verification-report.md`.
**Writes:** `draft-v2.md` and `draft-v2.docx` — the polished draft. Corrects every Verifier flag, polishes language to SC formal register (*it is most respectfully submitted that*, *the impugned judgment proceeds on a misconception that*, *the question that arises for the consideration of this Hon'ble Court is*), strips AI-style markers, enforces SC Registry formatting per Practice Direction (A4, Times New Roman 14, 1.5 line spacing, 4 cm left margin).
**Why it exists:** SC Registry returns petitions for filing on the smallest deviation from Practice Direction formatting. The Refiner ensures the draft passes Registry first-pass review.

### 6. `overseer` — sixth and final agent

**Reads:** `draft-v2.md` + `case-facts.md`.
**Writes:** `opposing-notes.md` (attackable defects, weak Grounds, gaps in the Statement of Facts, prayer-form issues, AOR Certificate language issues) + `final-draft.docx`.
**Why it exists:** the strongest test of a pleading is what an opposing counsel would say. The Overseer reads the draft *as if* it had been served on her, and writes a critique. The AOR uses that critique to harden the draft before signing.

---

## Installation

This is a Claude-compatible plugin in the Anthropic plugin format, designed to run inside the **Claude Desktop application** (available at <https://claude.ai/download>). The plugin folder location depends on your OS:

| OS | Plugin folder path |
|---|---|
| **macOS** | `~/Library/Application Support/Claude/plugins/` |
| **Windows** | `%APPDATA%\Claude\plugins\` (typically `C:\Users\<you>\AppData\Roaming\Claude\plugins\`) |
| **Linux** | `~/.config/Claude/plugins/` |

Clone the plugin into that folder:

```bash
# macOS / Linux
mkdir -p ~/Library/Application\ Support/Claude/plugins   # adjust per OS table
cd ~/Library/Application\ Support/Claude/plugins
git clone https://github.com/Wolfgangrush/supreme-court-drafting-litigation.git supreme-court-drafting

# Windows (PowerShell)
mkdir -Force $env:APPDATA\Claude\plugins
cd $env:APPDATA\Claude\plugins
git clone https://github.com/Wolfgangrush/supreme-court-drafting-litigation.git supreme-court-drafting
```

Restart the Claude Desktop application.

### Anthropic Plugin Marketplace (when available)

When the plugin lands on the Anthropic Plugin Marketplace, you will be able to install it from inside the application's plugin browser without `git`. Until then, the manual clone steps above are canonical.

### Verifying the install

In a Claude session, type:

- *"draft slp civil"* — triggers `slp-civil-draft`
- *"draft slp criminal"* — triggers `slp-criminal-draft`
- *"draft writ 32"* — triggers `writ-art32-draft`
- `/slp-civil-draft` — explicit slash-invocation

Claude should respond by asking for the case folder path or the case-specific facts.

---

## Your first SLP — step-by-step walkthrough

Suppose you wish to draft a **Special Leave Petition (Civil)** against a final judgment of the Bombay High Court.

### Step 1 — create a case folder

```
~/Desktop/cases/
└── slp-civil-pname-DD-MM-YYYY/
    ├── facts/
    │   ├── hc-judgment-DD.MM.YYYY.pdf
    │   ├── trial-court-decree-DD.MM.YYYY.pdf
    │   ├── plaint-DD.MM.YYYY.pdf
    │   └── written-statement-DD.MM.YYYY.pdf
    ├── laws/
    │   ├── constitution-art-136.pdf
    │   ├── sc-rules-2013-order-xxi.pdf
    │   └── relevant-supreme-court-judgments/
    │       ├── pranay-sethi.pdf
    │       └── (etc.)
    └── notes.md
```

### Step 2 — launch Claude inside the case folder

```bash
cd ~/Desktop/cases/slp-civil-pname-DD-MM-YYYY/
claude
```

### Step 3 — invoke the skill

```
draft slp civil
```

The plugin runs the **Reader** first. It will read every PDF in `facts/`, write `case-facts.md` with the trial-court chronology and the HC procedural history, identify annexure candidates (`P-1` for the HC judgment, `P-2` for the trial decree, etc.), and halt if any referred law PDF is missing.

Review `case-facts.md`. Edit anything the Reader misread. Save.

### Step 4 — continue the pipeline

**Format → Drafter → Verifier → Refiner → Overseer** run in sequence. At the end you have:

- `draft-v1.docx` — initial draft
- `verification-report.md` — Verifier flags (review every flag carefully — the citation-discipline flag is the most important)
- `draft-v2.docx` — Refiner output
- `opposing-notes.md` — Overseer critique
- `final-draft.docx` — for the AOR's review

### Step 5 — AOR review

The AOR opens `final-draft.docx`, reads every paragraph, verifies every citation against the source PDF, signs the AOR Certificate, signs the Supporting Affidavit, and files. **The AOR is responsible for the pleading. The plugin is responsible for the first draft.**

---

## The Supreme Court Rules 2013 — what this plugin encodes

The plugin's `_sc_pleading_base` skill encodes:

- **Order IV Rule 1(c)** — Advocate-on-Record Certificate language, verbatim
- **Order V** — Verification of Pleadings
- **Order IX** — Affidavits (Form 47)
- **Order XXI** — Appeals by Special Leave (civil) and Form 28
- **Order XXII** — Appeals by Special Leave (criminal) and Form 32
- **Order XXXVIII** — Writ proceedings under Article 32
- **Order XL** — Transfer of cases
- **Order XLVII** — Review
- **Order XLVIII** — Curative
- **Order LV** — Forms (Form 47 Affidavit)
- **Registry Practice Directions** on paper size (A4), font (Times New Roman 14), line spacing (1.5), and left margin (4 cm)
- **Practice Direction on Synopsis** — Synopsis must not exceed three pages
- **Practice Direction on List of Dates** — chronological tabular form, with the impugned judgment date highlighted

The plugin reads the bench's idiom for the cause-title chain *up to* the HC (which the user supplies as part of the case facts), and applies SC Rules 2013 + Practice Direction formatting from the SC level downward. **No bench-config file is required for SC pleadings** — the Supreme Court has uniform conventions across its benches; the only variation is the case-type, which the user signals by which skill they invoke.

---

## Why MIT License

The plugin is released under the **MIT License**. This was a deliberate choice. The alternatives — and why they were rejected — are below.

### MIT vs the alternatives

| License | Suitable for this plugin? | Reasoning |
|---|---|---|
| **MIT** ✅ chosen | Yes | Permissive · 3-line attribution requirement · zero copyleft · zero patent-grant complexity · compatible with the Anthropic Plugin Marketplace TOS · compatible with adoption by AOR chambers and big-firm SC litigation teams · allows a future paid commercial layer to ship under a separate corporate entity without dual-license complexity. |
| **Apache 2.0** | Close second | Adds an explicit patent grant — but Indian procedural drafting content is non-patentable under Section 3(k) of the Patents Act 1970 (computer programs *per se* and algorithms are not patentable in India). The patent grant is dead weight, while the NOTICE-file ceremony adds friction. |
| **GPL-3.0** | ❌ Disqualifying | Copyleft would propagate to the AOR's case folder. The pleading — generated by the plugin from the AOR's case facts — could be argued to be a "derivative work", forcing the pleading itself to be GPL-licensed. No AOR can ship privileged client material under any open-source licence. Hard structural blocker. |
| **AGPL-3.0** | ❌ Disqualifying | Same problem as GPL-3, plus the network-use clause triggers if anyone integrates the plugin into a SaaS legal-tech product. Blocks all commercial integration. |
| **LGPL-3.0** | ❌ Awkward | Designed for shared libraries, not for plugins that produce derivative documents. Library exception does not map cleanly to "the plugin produces a `.docx` that the AOR signs and files". |
| **BSD-3-Clause / BSD-2-Clause** | Functionally equivalent to MIT | Slightly different attribution wording; no practical advantage. MIT is more widely understood. |
| **Unlicense / CC0** | ❌ Forfeits authorship | Drops the copyright assertion. The author loses moral rights under Section 57 Copyright Act 1957 and the project loses derivative-misuse traceability. |
| **Creative Commons (any variant)** | ❌ Wrong instrument | CC licences are designed for creative content (text, images, video) and are not recommended for software. They lack software-grade warranty disclaimers and patent-grant clarity. |

### One-paragraph rationale

The plugin is released under MIT because Advocates-on-Record and the litigation teams at chambers across India must be able to clone, fork, adapt, and integrate this plugin alongside their privileged client material without the licence propagating into their case folders or attaching to the pleadings they file in the Supreme Court. Only MIT (and equivalently BSD and Apache 2.0) satisfies that constraint. MIT was preferred over Apache 2.0 because the patent-grant language Apache adds carries no practical benefit in the Indian context (procedural drafting content is unpatentable under Section 3(k) of the Patents Act 1970), and MIT's three-line clarity is friendlier to AORs and senior counsel who will read the LICENSE file before adopting the plugin in chambers.

### Compatibility statement

MIT is compatible with:

- Apache License 2.0
- BSD 2-Clause and BSD 3-Clause
- GPL family (downstream-incorporation only; this repository cannot itself be relicensed)
- Anthropic Plugin Marketplace Terms of Service
- All major commercial-software integration policies

---

## Sibling plugins

This plugin is one in the **wolfgang_rush** family of Indian legal-drafting plugins. All thirteen siblings ship under the same six-agent pipeline (Reader → Format → Drafter → Verifier → Refiner → Overseer) and the family-of-plugins doctrine — each plugin narrowly scoped to one practice area / forum:

| Plugin | GitHub repo | Scope |
|---|---|---|
| `supreme-court-drafting` (this) | [supreme-court-drafting-litigation](https://github.com/Wolfgangrush/supreme-court-drafting-litigation) | SLPs · Writ Art 32 · Transfer · Review · Curative — Supreme Court of India |
| `indian-hc-drafting` | [indian-hc-drafting-litigation](https://github.com/Wolfgangrush/indian-hc-drafting-litigation) | Pleadings across all 25 Indian High Courts (bench-config-aware) |
| `district-court-drafting` | [district-court-drafting-litigation](https://github.com/Wolfgangrush/district-court-drafting-litigation) | Plaints · WS · CPC applications · BNSS complaints across 25+ States (state-config) |
| `indian-family-drafting` | [indian-family-drafting-litigation](https://github.com/Wolfgangrush/indian-family-drafting-litigation) | HMA · SMA · IDA · matrimonial · custody · DV Act · maintenance · adoption |
| `indian-contracts-drafting` | [indian-contracts-drafting-litigation](https://github.com/Wolfgangrush/indian-contracts-drafting-litigation) | MSA · NDA · employment · lease · sale · GPA · SHA · will · loan · arbitration |
| `indian-banking-drafting` | [indian-banking-drafting-litigation](https://github.com/Wolfgangrush/indian-banking-drafting-litigation) | DRT · SARFAESI · NI Act 138 · IBC §7 / §95 · DRAT |
| `indian-labour-drafting` | [indian-labour-drafting-litigation](https://github.com/Wolfgangrush/indian-labour-drafting-litigation) | ID Act · POSH · PG · EPF · ESI · MW · IESO + State exemplars |
| `indian-property-drafting` | [indian-property-drafting-litigation](https://github.com/Wolfgangrush/indian-property-drafting-litigation) | Gift · Exchange · Release · Trust · Wakf · Easement · Partition · Settlement · Mortgage · TIR |
| `indian-company-drafting` | [indian-company-drafting](https://github.com/Wolfgangrush/indian-company-drafting) | NCLT (241/242 · 245 · 230-232 · 66 · 252 · 213) · NCLAT (421 + 61) · IBC §9 / §10 |
| `indian-tax-drafting` | [indian-tax-drafting](https://github.com/Wolfgangrush/indian-tax-drafting) | Form 35 CIT(A) · Form 36 ITAT · Form 10A · Sec 148A · 263/264 · 271/270A · 144C · 201 · 260A |
| `indian-consumer-drafting` | [indian-consumer-drafting](https://github.com/Wolfgangrush/indian-consumer-drafting) | District / State / NCDRC + medical-negligence + product liability |
| `indian-mact-drafting` | [indian-mact-drafting](https://github.com/Wolfgangrush/indian-mact-drafting) | MV Act 1988 (2019 amended) · Sarla Verma + Pranay Sethi · state-config |
| `indian-ip-drafting` | [indian-ip-drafting](https://github.com/Wolfgangrush/indian-ip-drafting) | Copyright · Trade Marks · Patents · Designs + HC IP Divisions (post-IPAB-abolition) + Anton Piller / John Doe |

Each plugin can be installed independently, each plugin's Rule 36 firewall is narrow and reviewable, each plugin's bench / forum discipline is depth-validated within its scope, and the user installs only what they need.

---

## Why this exists

The Supreme Court of India is the apex court and the most procedurally stringent forum in India. Its Registry returns petitions for filing on the smallest formatting deviation. Synopsis exceeds three pages → returned. List of Dates not in tabular form → returned. Annexure prefix wrong (`Annexure-1` instead of `P-1`) → returned. AOR Certificate language paraphrased instead of recited verbatim from Order IV Rule 1(c) → returned.

Generic AI tools do not understand these conventions. They produce drafts that read superficially correct but fail Registry scrutiny on the small things.

This plugin encodes the SC conventions directly. The plugin's output passes Registry first-pass review when the case folder supplies accurate inputs.

Just as importantly: **the Supreme Court has publicly cautioned against AI-generated content with fabricated citations.** This plugin is built explicitly to prevent that failure mode. The Drafter agent does not generate case names + citations from training memory; every citation traces to a user-supplied list, or appears as a `[CITATION NEEDED]` placeholder for the AOR to fill manually.

---

## Roadmap

- [x] **v0.1.0-alpha (current)** — SLP (Civil + Criminal), Writ Art 32, Transfer Petition, Review Petition, Curative Petition + 6-agent pipeline + shared SC pleading base
- [ ] **v0.1.x** — bug fixes, Registry-format polish, citation-discipline refinements, language-register iteration
- [ ] **v0.x onward** — additional SC case-type skills (Caveats, Tagged Appeals from tribunals, Article 143 References, Election Petitions, standalone Article 136 Bail, etc.) as community-requested
- [ ] **v1.0.0** — Stable release after community-validated use by sitting AORs

The roadmap above is intentionally open-ended. Additional case-type skills will arrive as AORs identify the procedural-drafting patterns they would like the plugin to encode.

---

## Contributing

Advocates-on-Record and senior advocates with regular Supreme Court practice are invited to contribute Registry-validation feedback, format calibration on edge cases (AFT-origin SLPs, NCDRC appeals, NGT appeals, NCLAT appeals), and quality-gate refinements. Open an issue with the specific case type and your observations.

Pull requests welcome with a one-paragraph explanation and a reference to the SC Rules / Practice Direction / Constitution Bench judgment that justifies the change.

This plugin is open source under MIT.

---

## Contact

All inquiries and feedback via **GitHub Issues** on the project repository.

Bug reports · feature requests · jurisdiction-validation offers · improvements: open an issue.

This project does not have an email contact channel and **does not accept private legal-services inquiries through this repository**. No commercial engagement is offered through this plugin or its repository.

*(Future releases may introduce a commercial layer published under a separate corporate entity — at that point this section will be updated. v0.x.x is open-source-only infrastructure with no commercial channel.)*

---

## Author and brand

This plugin is authored by **Rushikesh R. Mahajan**, Advocate, enrolled with the Bar Council of Maharashtra and Goa, practising before the High Courts of India.

The plugin is published under the **wolfgang_rush** open-source brand — the author's publishing handle for legal-technology infrastructure. All commits to this repository are signed under the wolfgang_rush GitHub identity. The real-identity declaration appears here, and again in `NOTICE.md`, so that the Bar Council Rule 36 accountability mechanism (advocate-as-individual responsibility) is preserved transparently rather than displaced by the publishing handle.

---

## Provenance and privilege statement

See [`NOTICE.md`](./NOTICE.md) for the full provenance and privilege statement.

**In brief:** this plugin contains only public procedural knowledge — Supreme Court Rules 2013 conventions, Registry Practice Direction formatting, constitutional and statutory references, generic placeholders. It does **not** contain any specific client matter, communication, document, or personal data. The legal distinction between privileged client communications (protected under Section 132 of the Bharatiya Sakshya Adhiniyam 2023 / Section 126 of the Indian Evidence Act 1872) and general professional knowledge of procedural law (public, sharable, the same knowledge underlying every legal-drafting textbook) is set out in `NOTICE.md` in full.

The corpus-validation methodology used during the build is documented under the **corpus-as-verifier-not-source doctrine**: structural patterns are cross-validated against a public-record corpus to confirm what is used in practice, but the corpus never supplies the language that is encoded.

---

## Compliance posture — Supreme Court e-Committee AI framework

This plugin is **assistive drafting infrastructure**, not autonomous decision-making software. Its operational posture is aligned with the Supreme Court of India e-Committee's stated position on AI in legal work.

> *"AI and digital tools must be used as supportive instruments and should not be allowed to override judicial reasoning."*
>
> — **Justice Rajesh Bindal**, Judge, Supreme Court of India
> [*Judicial Process Re-engineering and Digital Transformation*](https://www.sci.gov.in/press-release-dated-april-12-2026/) conference, 11–12 April 2026
> Organised by the Supreme Court e-Committee in collaboration with the Department of Justice, Government of India.
> ([Coverage — Law Trend](https://lawtrend.in/ai-must-not-replace-judicial-reasoning-warns-supreme-court-justice-rajesh-bindal/))

The same posture underpins the Supreme Court's own AI infrastructure for the judiciary:

- **[SUPACE](https://www.drishtiias.com/daily-news-analysis/ai-portal-supace)** — *Supreme Court Portal for Assistance in Court Efficiency.* AI-enabled assistive tool launched on 6 April 2021 by then-CJI S.A. Bobde. Provides legal research, fact extraction, document review, and drafting assistance to judges and legal researchers. **By design, SUPACE is not a decision-making system** — it processes facts and surfaces them to the human user. The Supreme Court has recommended adoption across all Indian High Courts.

- **[SUVAS](https://www.drishtijudiciary.com/current-affairs/supreme-court-vidhik-anuvaad-software-suvas)** — *Supreme Court Vidhik Anuvaad Software.* AI-powered translation tool launched in November 2019 by then-CJI S.A. Bobde. Translates judicial documents, orders, and judgments between English and ten Indian regional languages.

### What this plugin does — and does not — do under that framework

**Does:**

- Generate structural skeletons of pleadings, drawing on public statutes, schedule forms, and court rules.
- Run a six-agent assistive pipeline (Reader → Formatter → Drafter → Verifier → Refiner → Overseer) over the user's case facts.
- Surface citations, procedural anchors, and bench-specific conventions for advocate review.

**Does NOT:**

- Generate final filings autonomously.
- Substitute for advocate professional judgment.
- Replace human verification.
- Operate without an enrolled advocate retaining full professional responsibility.

**Every draft produced through this plugin must be advocate-owned and human-verified before filing.** The enrolled advocate using this plugin retains full professional responsibility under the Advocates Act 1961 and the Bar Council of India Rules, including verification of facts, accuracy of citations, correctness of legal grounds, propriety of the prayer, and signature on every pleading filed.

This is the same standard the Supreme Court itself applies to its own AI infrastructure (SUPACE / SUVAS): **AI as supportive instrument, never as decision-maker.**

---

## Disclaimer and Bar Council of India Rule 36 compliance

This plugin is **open-source infrastructure released free of cost** under the MIT licence. It is published as a contribution to the legal community and to the broader open-source developer ecosystem.

**This plugin:**

- Does **not** constitute legal advice.
- Does **not** create an advocate-client relationship between the author and any user.
- Does **not** solicit professional work from any user, group, or audience.
- Does **not** advertise the professional services of the author or any advocate.
- Does **not** offer paid legal services, paid consultations, or commercial legal engagements through this repository or any associated channel.

**It is:**

- A drafting aid for use by **Advocates-on-Record and by advocates working under AOR supervision**, who retain full professional responsibility for every pleading produced.
- A reference implementation of open-source legal-tech infrastructure for Supreme Court of India practice.
- Released under the Bar Council of India Rules, Part VI, Chapter II, Section IV, **Rule 36** (Conduct and Etiquette — restriction on advertising and solicitation), to which the author and every Indian advocate is bound.

**Every advocate using this plugin is reminded:** the AOR retains full professional responsibility for the verification of facts, the accuracy of citations, the correctness of legal grounds, the propriety of the prayer, and the signature on every pleading filed. AI-generated drafting output is **starting material, not a finished pleading**.

---

## License

**MIT.** See [`LICENSE`](./LICENSE) for the full text.

Copyright (c) 2026 Rushikesh R. Mahajan (publishing as wolfgang_rush). Authored by Rushikesh R. Mahajan, Advocate, publishing under the wolfgang_rush open-source brand.
