# USAGE — `supreme-court-drafting`

This document is the end-user guide for any Advocate-on-Record or advocate working under AOR supervision who wants to use this plugin to draft Supreme Court pleadings.

---

## 1. What you need before you start

| Requirement | Why |
|---|---|
| **A Claude-compatible runtime** | This plugin runs inside any Anthropic-compatible runtime that parses the standard SKILL.md plugin format — the official Anthropic plugin client, the Anthropic Desktop application, or a custom Anthropic-API-based runtime built to the same conventions |
| **An active Anthropic API account** | The plugin invokes the Anthropic API through your chosen runtime; usage is billed to your account |
| **Microsoft Word or LibreOffice** | To open and review the `.docx` output |
| **`pandoc`** (optional but recommended) | Installed via Homebrew (`brew install pandoc`) or apt (`sudo apt install pandoc`). The plugin uses pandoc to render the `.docx`. Without pandoc, falls back to `python-docx`. |
| **Your case folder** | A directory on your computer containing the case documents (PDFs of impugned order, lower-court pleadings, FIR/chargesheet for criminal, etc.) |

---

## 2. Install the plugin

The plugin is a directory of markdown skill files + agent files. Clone it to wherever your Anthropic runtime discovers plugins (consult your runtime's documentation for the exact path):

```bash
git clone https://github.com/wolfgang-rush/supreme-court-drafting \
  <your-anthropic-plugins-folder>/supreme-court-drafting
```

Verify the plugin loads in your runtime according to your runtime's standard plugin-listing mechanism.

---

## 3. One-time setup — paste your drafting style references

Each case-type skill has a `format-from-user.md` file under `skills/<case-type>-draft/`. These files request **your own preferred drafting style** for each section (Synopsis opener, List of Dates style, Cause Title formatting, AOR Certificate signature-block layout, etc.).

Open each file and paste your preferred style. The plugin uses these for connective-phrase alignment — it does NOT pull case-specific content from them.

You only do this once per Claude installation. Subsequent cases reuse your saved style.

```bash
# Open each format-from-user.md in your editor
cd ~/.claude/plugins/supreme-court-drafting/skills
$EDITOR slp-civil-draft/format-from-user.md
$EDITOR slp-criminal-draft/format-from-user.md
$EDITOR writ-art32-draft/format-from-user.md
$EDITOR transfer-petition-draft/format-from-user.md
$EDITOR review-petition-draft/format-from-user.md
$EDITOR curative-petition-draft/format-from-user.md
```

If you skip this step, the Drafter will fail-stop on the first invocation and remind you.

---

## 4. Prepare your case folder

A case folder is a directory on your computer containing everything the plugin needs to draft your pleading. Suggested layout:

```
~/cases/<client-code-or-matter-name>/
├── CLAUDE.md                      # case context (case type, court, AOR details)
├── citations.md                   # YOUR confirmed list of case citations
├── laws/                          # PDFs of statutes not in Claude's training data
│   ├── POCSO-Act-2012.pdf
│   ├── NDPS-Act-1985.pdf
│   └── (etc.)
├── 01-impugned-judgment.pdf       # the High Court judgment under challenge
├── 02-trial-court-order.pdf       # lower court orders (where relevant)
├── 03-FIR.pdf                     # for criminal SLP
├── 04-chargesheet.pdf             # for criminal SLP
├── 05-pleadings-before-HC.pdf     # writ petition / appeal memo filed at HC
└── (other case-specific documents)
```

### What goes in `CLAUDE.md` (the case context file)

```markdown
# Case Context — <Internal Case Reference>

## Case type
slp-civil           # or slp-criminal / writ-art32 / transfer-petition / review-petition / curative-petition

## Parties
Petitioner: [Petitioner Name], [Age], [Occupation], R/o [Address]
Respondent: [Respondent Name], [Status in lower court]

## Court of origin
[High Court name + Bench]

## Impugned order
Case number: [Case Type & Number]
Date of order: <DD.MM.YYYY>

## Limitation
Cause of action / date of impugned order: <DD.MM.YYYY>
Certificate of fitness under Art 134A applied for: yes / no
Certificate refused (date): yes-<DD.MM.YYYY> / no

## AOR engaged
Name: [AOR Name]
Registration code: [AOR Code]
```

### What goes in `citations.md` (your confirmed case citations)

```markdown
# Confirmed Case Citations

| Case | Citation | Cited for |
|---|---|---|
| Pritam Singh v. State | AIR 1950 SC 169 | Art 136 substantial question / gross injustice line |
| (add your case-specific citations here) | | |
```

The Drafter **only** uses citations from this file. It never generates a case name + citation from training memory. If a ground requires support not in this list, the Drafter inserts `[CITATION NEEDED]` for you to fill manually.

---

## 5. Run the pipeline

Open your Claude-compatible session inside your case folder. Invoke the case-type skill — either by typing the trigger phrase or the slash command:

```
draft slp civil
```

or:

```
/slp-civil-draft
```

Claude Code routes to the matching skill, which fires the six-agent pipeline:

1. **Reader** reads your case folder, builds `case-facts.md`, flags missing statute PDFs / unconfirmed citations / incomplete fields. If something is missing, the pipeline stops and tells you what to add.

2. **Format** loads the case-type SKILL.md, maps your case facts into the SC pleading structure, pre-fills verbatim statutory blocks (AOR Certificate per Order IV Rule 1(c), Declarations per Order XXI Rule 3(2) / 5, Affidavit verification per Form 47), produces `format-shell.md`.

3. **Drafter** writes the Statement of Facts, Questions of Law, Grounds, Prayer — original prose authored from your case facts. Inserts `[CITATION NEEDED]` placeholders where your `citations.md` does not cover an asserted proposition. Produces `draft-v1.md` and `draft-v1.docx`.

4. **Verifier** runs twelve SC-specific anti-hallucination checks against `draft-v1.md`. Produces `verification-report.md` with every flag.

5. **Refiner** applies the Verifier's flags, polishes language, enforces SC Registry formatting (A4, Times New Roman 14, 1.5 line spacing, 4cm left margin), normalises citations, removes AI-style markers. Produces `draft-v2.docx`.

6. **Overseer** reads `draft-v2.md` with the opposing-counsel and Bench lens. Anticipates maintainability objections, scope-of-Article-136 objections, prayer-realism issues. Produces `opposing-notes.md` and `final-draft.docx`.

At the end, your case folder contains:

```
case-folder/
├── (your original case documents)
├── case-facts.md
├── format-shell.md
├── draft-v1.md           draft-v1.docx
├── verification-report.md
├── draft-v2.md           draft-v2.docx
├── opposing-notes.md
└── final-draft.docx      ← OPEN THIS IN WORD
```

---

## 6. Review and finalise

Open `final-draft.docx` in Microsoft Word or LibreOffice. Apply tracked-changes review. The audit chain above documents every step — if anyone asks "how was this drafted," you have the full file chain.

You remain the responsible Advocate-on-Record. The plugin's output is starting material. **Read every word before you sign**. Verify every citation. Confirm the Statement of Facts matches the record. Tighten the Grounds. Adjust the Prayer.

When you are satisfied, print, sign, and present at the Supreme Court Registry.

---

## 7. What the plugin will NOT do

- It will not generate case citations from training memory. Every citation in your output traces to your `citations.md`.
- It will not fabricate facts. Every assertion traces to your case folder.
- It will not file the pleading for you.
- It will not draft the AOR Certificate language differently from Order IV Rule 1(c) — that is verbatim from the Rule.
- It will not replace your professional judgment on strategy, scope of Article 136 invocation, or relief sought.

---

## 8. Help and contact

- Bug reports / feature requests / Registry-validation feedback: open a GitHub Issue at https://github.com/wolfgang-rush/supreme-court-drafting/issues
- This project does not have an email contact channel and does not accept private legal-services inquiries.
- The plugin is released under the MIT licence — see `LICENSE`.
- For provenance and Bar Council Rule 36 compliance details, see `NOTICE.md`.

---

## 9. Quick troubleshooting

| Problem | Fix |
|---|---|
| Drafter halts with `STYLE.FAIL_STOP` | You have not pasted style references into `format-from-user.md`. Open the file and paste your preferred style. |
| Reader halts with `Need PDF` | You have referenced a statute (e.g., POCSO Act) whose PDF is not in your case folder's `laws/` subfolder. Add the PDF. |
| Verifier flags `F9 — fundamental right not invoked` (Art 32 case) | Your case-folder `CLAUDE.md` does not specify the fundamental right under Part III of the Constitution being invoked. Add the article (e.g., `Article 21 — right to personal liberty`). |
| Verifier flags many `F3 — citation not in citations.md` items | Your case folder references citations that you have not yet confirmed. Add them to `citations.md` AND re-run, OR accept the `[CITATION NEEDED]` placeholders and fill them manually post-draft. |
| `.docx` output looks unformatted | Pandoc may not be installed. The plugin falls back to python-docx, which produces a usable file but without the full SC Registry template. Install pandoc (`brew install pandoc`) for best results. |
| Verifier flags `F12 — placeholder violation` | A real name / date / case number has leaked into the draft where a placeholder should be. The Refiner restores the placeholder; verify the result before signing. |

---

## 10. The plugin in one sentence

**This plugin converts your Supreme Court case folder into a Registry-compliant `.docx` pleading, with every fact traceable to your case folder and every citation traceable to your confirmed list — and you remain the responsible Advocate-on-Record at every stage.**
