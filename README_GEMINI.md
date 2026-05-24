# 🇮🇳 Supreme Court Drafting — Gemini CLI Port

> The same `supreme-court-drafting` pipeline you know, now invokable from Gemini CLI. **Same 6-agent pipeline. Same canonical agent specs. Same filing-grade output. Different LLM brain.**

**Version:** Gemini port 2026-05-24 · **License:** MIT · **Publisher:** Wolfgang Rush

---

## What this port adds

This folder now contains a `.gemini/` directory alongside the existing `.claude-plugin/` and `agents/`:

```
supreme-court-drafting/
├── .claude-plugin/                  ← Claude plugin manifest (unchanged)
├── agents/                          ← Canonical 6-agent specs (unchanged · single source of truth)
│   ├── reader/reader.md
│   ├── format/format.md
│   ├── drafter/drafter.md
│   ├── verifier/verifier.md
│   ├── refiner/refiner.md
│   └── overseer/overseer.md
├── skills/ (6 case-type templates)                   ← case-type templates (unchanged)
├── .gemini/                         ← NEW · Gemini CLI commands
│   └── commands/
│       ├── reader.toml
│       ├── format.toml
│       ├── drafter.toml
│       ├── verifier.toml
│       ├── refiner.toml
│       ├── overseer.toml
│       └── pipeline.toml             ← runs all 6 sequentially
├── GEMINI.md                        ← NEW · Gemini hierarchical context (auto-loaded)
└── README_GEMINI.md                 ← NEW · this file
```

**Nothing else was modified.** Each Gemini command reads its canonical spec from `agents/<name>/<name>.md` via shell injection at call time — so the .md file remains the single source of truth. Edit the .md, both Claude and Gemini inherit the change.

---

## Install — 4 steps

### Step 1 — Install Gemini CLI (one-time, ~2 min)

Requires Node.js 20+ ([download](https://nodejs.org/)).

```bash
npm install -g @google/gemini-cli
gemini --version
```

### Step 2 — Set your Gemini API key

Get one from [Google AI Studio](https://aistudio.google.com/apikey).

**For client work → enable PAID billing first** (free tier sends prompts into training, which breaches BCI Rule 17).

```bash
echo 'export GEMINI_API_KEY="paste-your-key-here"' >> ~/.zshrc
source ~/.zshrc
```

### Step 3 — Prepare your case folder

```
<case-folder>/
├── format-from-user.md           ← advocate-supplied (required)
├── laws/                            ← put required statute PDFs here
└── <documents>.pdf                  ← case documents
```

### Step 4 — Launch Gemini CLI from this plugin folder

```bash
cd "supreme-court-drafting"
gemini
```

Gemini auto-loads `GEMINI.md` + all 7 slash commands in `.gemini/commands/`.

---

## Use — the 7 commands

| Command | Stage | What it does |
|---|---|---|
| `/reader <case-folder>` | 1 | Extract content per-document, map annexures, flag missing laws → `case-facts.md` |
| `/format <case-folder>` | 2 | Load template + substitute config → `format-shell.md` |
| `/drafter <case-folder>` | 3 | Fill format-shell with case facts → `draft-v1.docx` |
| `/verifier <case-folder>` | 4 | Anti-hallucination check vs case-facts → `verification-report.md` |
| `/refiner <case-folder>` | 5 | Apply verifier flags + polish → `draft-v2.docx` |
| `/overseer <case-folder>` | 6 | Opposing-counsel lens + harden → `opposing-notes.md` + `final-draft.docx` |
| `/pipeline <case-folder>` | — | Run all 6 in sequence |

### First run (smoke test)

```
> /reader /Users/<you>/Desktop/cases/sample-matter
```

Should produce `<case-folder>/case-facts.md` with annexure mapping + missing-law flags.

---

## Privacy — the honest version

| What you are running | Where your data goes |
|---|---|
| The pipeline TOMLs + agent specs + your case folder | **Your laptop only.** Plain files on disk. |
| **What Gemini sees** (case documents you let it read · your prompts) | **Google's Gemini servers.** Paid tier: NOT used for training. Free tier: MAY be used for training. |

**Decision rule for client data:**

- ✅ **Local Ollama** (use the bundled `ailawfirm-india` CLI with `"ai_provider": "ollama"`) → real client matters under BCI Rule 17
- ✅ **Gemini paid tier** → drafting templates, study, mock matters, public-record matters with client consent
- ❌ **Gemini free tier** → never for any client work
- ❌ **Any cloud LLM** for client data without client consent + DPA → breach of BCI Rule 17 + Advocates Act §35

---

## Forum support

**Forum:** Supreme Court of India (SC Registry conventions enforced)

**Case types covered:**
SLP-Civil · SLP-Criminal · Writ Art. 32 · Transfer Petition · Review · Curative (Rupa Hurra limbs)

---

## What this port does NOT do

- Does **NOT** modify any existing file in the plugin. Pure additive.
- Does **NOT** replace the Claude port. Both coexist.
- Does **NOT** invent its own agent specs. Reads canonical `.md` agent files at call time.
- Does **NOT** route around BCI Rule 17 / DPDP Act 2023. Honest privacy split surfaced at every run.

---

## Troubleshooting

### `gemini: command not found`
Node 20+ installed? `npm install -g @google/gemini-cli` succeeded? Restart terminal.

### Custom commands not appearing
Launch `gemini` from inside the plugin folder (the one containing `.gemini/`). Project commands require working-directory match.

### `cat: agents/<name>/<name>.md: No such file`
You launched `gemini` from a folder other than the plugin root. `cd` into the plugin folder first.

### Free vs paid tier
Check at [Google AI Studio billing](https://aistudio.google.com/app/billing). If billing not enabled, you are on free tier — **do not use for client data**.

---

## Differences from the Claude port (quick reference)

| Aspect | `.claude-plugin/` | `.gemini/` |
|---|---|---|
| File format | Markdown with YAML frontmatter | TOML |
| Context file | `CLAUDE.md` | `GEMINI.md` |
| Agent invocation | Via Claude's Agent tool | Via slash command |
| Argument passing | Claude's prompt body | `{{args}}` substitution |
| Discovery | Claude reads `.claude-plugin/` | Gemini reads `.gemini/` |

Both ports inherit the SAME canonical agent specs from `agents/<name>/<name>.md`. Edit once, both ports follow.

---

## ⚖️ Same compliance posture as the Claude port

This Gemini port is **assistive drafting infrastructure**, not autonomous decision-making software. Same Supreme Court e-Committee alignment:

> *"AI and digital tools must be used as supportive instruments and should not be allowed to override judicial reasoning."*
> — **Justice Rajesh Bindal**, Judge, Supreme Court of India

**Every draft — whether Claude or Gemini produced it — must be advocate-owned and human-verified before filing.**

---

## ⚠️ Third-party CLI tools — user assumes all risk

The publisher (Wolfgang Rush) does NOT recommend Gemini CLI specifically, receives no compensation from Google, and verifies no claims about Gemini's privacy posture. You have chosen to integrate this plugin with Gemini CLI and assume all risk under MIT license + Google's Gemini API terms.

For client matters under BCI Rule 17 → local Ollama. For non-client work → Gemini paid tier is acceptable.

---

`चलिए शुरू करें · Let's begin.` 🙏
