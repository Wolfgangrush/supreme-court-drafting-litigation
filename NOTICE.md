# NOTICE — Provenance and Privilege Statement

This document is part of the public release of the `supreme-court-drafting` plugin (v0.1.0 and onwards). It declares the provenance of the plugin's content, in order to address any question about advocate-client privilege, client confidentiality, professional ethics, and personal-data protection that may be raised by any reader, complainant, regulator, Bar Council disciplinary authority, or Supreme Court Registry.

This NOTICE is published in plain language so that any reader — practising advocate, Advocate-on-Record, judge, Bar Council officer, regulator, member of the public, fellow developer — can understand the position without ambiguity.

---

## 1. What this plugin contains

This plugin contains the following categories of content, and **only** the following categories of content:

(a) **Procedural skeletons** — the structural shape of pleadings before the Supreme Court of India as mandated by the Constitution, the Supreme Court Rules 2013, and the Supreme Court Registry Practice Directions (Cover Page, Index, Synopsis, List of Dates, Cause Title, Opening Address, Petition Proper, Statement of Facts, Questions of Law, Grounds, Main Prayer, Interim Prayer, Declarations, Signature Block, AOR Certificate, Affidavit, Annexure Index).

(b) **Formatting conventions** — Registry-compliant text-formatting conventions of the Supreme Court of India — A4 paper, Times New Roman 14, 1.5 line spacing, 4 cm left margin, annexure marker prefixing (`P-N` / `R-N`), Synopsis page limit, List of Dates tabular format, AOR Certificate verbatim layout.

(c) **Constitutional and statutory references** — citations to the Constitution of India (Articles 32, 136, 137, 141, 142, 144), the Supreme Court Rules 2013 (Orders IV, IX, XXI, XXII, XXXVIII, XL, XLVII, XLVIII, and Schedule Forms 28 and 32), the Code of Civil Procedure 1908 (Section 25, Order XLVII), the Bharatiya Nagarik Suraksha Sanhita 2023 (Section 406), the Limitation Act 1963, and other public enactments cited as procedural authority.

(d) **Procedural rule references** — citations to public rules of court (Supreme Court Rules 2013, Supreme Court Practice and Procedure Office Procedure Handbook, Supreme Court PIL Rules, Bar Council of India Standards of Professional Conduct and Etiquette under Section 49(1)(c) of the Advocates Act 1961).

(e) **Generic placeholders** — every variable in every template is marked with a placeholder such as `[Petitioner]`, `[Respondent]`, `<DD.MM.YYYY>`, `[Impugned Order Reference]`, `[AOR Name]`, `[AOR Registration Code]`, `[Annexure P-1]`. No placeholder is filled with any specific person's name, any specific date, any specific case number, or any specific identifying information.

(f) **Anti-hallucination workflow** — six agents (Reader, Format, Drafter, Verifier, Refiner, Overseer) that operate on a case folder supplied by the user. The plugin itself contains no case folder.

(g) **Case-citation discipline** — the Drafter agent is constrained from generating case names + citations from training memory; every case citation in the output must trace to a user-supplied source. This is stricter than the corresponding rule in the `bombay-hc-drafting` sister plugin because the Supreme Court has expressly cautioned against AI-fabricated citations.

---

## 2. What this plugin does NOT contain

This plugin does **not** contain any of the following, and has never contained any of the following at any point in any committed version:

(a) The text, language, or substance of any drafted pleading by any advocate, Advocate-on-Record, or compiler other than the author of this plugin. Every connective phrase, every authored line, every skeleton paragraph in every skill file has been written fresh by the author for this plugin.

(b) Any text taken verbatim from any commercially compiled or privately compiled corpus of pleadings. Where the India-Legal Corpus Pipeline has been used to cross-validate structural patterns, the pipeline operates under the express `corpus-as-verifier-not-source` doctrine documented in the build kit: the corpus tells us *which patterns are used in practice* but never supplies the *language* that is encoded.

(c) Any specific client's name, address, contact information, case number, FIR number, judgment text, deposition text, or any other identifier that could link the plugin to any real proceeding before the Supreme Court of India or any subordinate court.

(d) Any privileged communication between any advocate and any client.

(e) Any document or material that would attract Section 132 of the Bharatiya Sakshya Adhiniyam 2023 (corresponding to Section 126 of the Indian Evidence Act 1872) on professional communications.

(f) Any solicitation of legal services, any commercial offering, any advertisement of the author's practice, or any inducement to retain the author. The plugin is published under Bar Council of India Rule 36 (Conduct and Etiquette — restriction on advertising and solicitation) as open-source infrastructure released free of cost, without any commercial channel.

---

## 3. The legal distinction this NOTICE relies on

The substantive legal distinction is between:

**(a) Privileged client communication** — protected under Section 132 of the Bharatiya Sakshya Adhiniyam 2023 (corresponding to Section 126 of the Indian Evidence Act 1872). This is the communication itself between the advocate and the client, the advice given, the instructions received, and the documents shared in the course of the retainer.

**(b) Public procedural knowledge** — the rules of procedure prescribed by the Constitution, by statute, by the Supreme Court Rules 2013, and by the Court's Practice Directions. This is the same knowledge that appears in every legal-drafting textbook (V. Sudhish Pai, M.A. Qureshi, the Supreme Court Practice and Procedure Office Procedure Handbook). It is the same knowledge that every senior advocate teaches every junior. It is the same knowledge that every Advocate-on-Record applies in every petition filed. It is not anyone's private property.

This plugin contains only category (b). It does not contain, and has never contained, anything from category (a).

The plugin is structurally incapable of containing category (a), because the plugin's design separates the case folder (which is the user's own working directory, never committed to this repository) from the plugin's skill files (which contain only public procedural knowledge and placeholders).

---

## 4. Provenance of the plugin's content

Every section of every skill file in this plugin traces to one or more of the following public sources:

- The Constitution of India.
- The Supreme Court Rules 2013, including their Schedules and Forms.
- The Code of Civil Procedure 1908 and its Schedules.
- The Bharatiya Nagarik Suraksha Sanhita 2023 (and the Code of Criminal Procedure 1973 for transitional reference).
- The Limitation Act 1963.
- The Supreme Court Practice and Procedure (Office Procedure Handbook) of the Supreme Court Registry.
- Leading procedural textbooks (cited only as authority for understanding; never quoted verbatim).
- The validation reports from the India-Legal Corpus Pipeline Phase 03, which themselves cite only public-domain authority.

The author of this plugin has not transcribed any drafted prose from any third-party source into this plugin.

---

## 5. Anti-hallucination architecture

The plugin's drafting pipeline is built specifically to prevent the categories of error that the Supreme Court has cautioned against:

(a) The Reader agent will halt the pipeline if any statute referenced in the case folder is not supplied as a PDF and is not within the training-data-allowed list (Constitution, CPC, IPC/BNS, CrPC/BNSS, IEA/BSA, SC Rules 2013).

(b) The Drafter agent does not generate case names + citations from memory. Every citation traces to a user-supplied list, or appears as a `[CITATION NEEDED]` placeholder for the advocate to fill manually.

(c) The Verifier agent runs a fact-by-fact comparison against the case-facts file produced by the Reader, flagging any assertion in the draft that does not trace to a fact in the case-facts file.

(d) The Overseer agent reviews the draft with an opposing-counsel lens and flags weak prayers, attackable defects, and missing limbs of argument.

(e) Every input file accessed by the Reader is logged in `case-facts.md` Section 1 with timestamp and summary. The audit chain is `case-facts.md` → `format-shell.md` → `draft-v1.docx` → `verification-report.md` → `draft-v2.docx` → `opposing-notes.md` → `final-draft.docx`.

(f) The advocate-as-AOR remains the responsible signatory. The plugin produces a draft; the human AOR reviews, verifies, and signs.

---

## 6. The Advocate-on-Record discipline

The Supreme Court of India recognises only Advocates-on-Record (AORs) as parties competent to file petitions and represent parties before it (Order IV, SC Rules 2013). This plugin does not, and cannot, replace an AOR. Specifically:

(a) The plugin produces a draft pleading; it does not file the pleading.
(b) The plugin produces a placeholder AOR Certificate block; the AOR must add their name, registration code, signature, and date, and personally certify the contents per Order IV Rule 1(c).
(c) The plugin does not communicate with the Supreme Court Registry.
(d) The plugin does not act on behalf of any party.
(e) The plugin is a drafting aid for use by an AOR or by an advocate working under the supervision of an AOR.

---

## 7. Bar Council of India Rule 36 compliance

Bar Council of India Rules, Part VI, Chapter II, Section IV, Rule 36 restricts an advocate from soliciting work or advertising. This plugin is published in conformity with Rule 36:

(a) The plugin is open-source, released under the MIT licence, free of cost.
(b) The plugin does not offer paid services through this repository.
(c) The plugin does not advertise the author's practice.
(d) The plugin does not solicit retainer engagements.
(e) The plugin is published under the **Wolfgang Rush** open-source brand, a name used to publish open-source legal-tech infrastructure separately from the author's professional advocacy practice.

Future releases may introduce a commercial layer published under a separate corporate entity. At that point this section will be updated. The current release (v0.x.x) is open-source-only infrastructure with no commercial channel.

---

## 8. Liability and warranty disclaimer

The plugin is released under the MIT licence. The licence text reproduces the standard "no warranty" disclaimer.

In addition: the plugin is a drafting aid. The user-advocate retains full professional responsibility for the verification of facts, the accuracy of citations, the correctness of legal grounds, the propriety of the prayer, and the signature on every pleading filed before the Supreme Court of India or any other court. AI-generated drafting output is starting material, not a finished pleading.

The author of this plugin disclaims liability for any consequence arising from the use of the plugin by any user-advocate, including but not limited to errors in any draft produced, citations included in any draft, factual assertions made in any draft, or the filing of any draft before any court.

---

## 9. Contact

All inquiries about this plugin are handled via **GitHub Issues** on the project repository.

This project does not have an email contact channel and does not accept private legal-services inquiries through this repository.

---

## 10. Author

**Rushikesh R. Mahajan**, Advocate, enrolled with the Bar Council of Maharashtra and Goa, practising before the Bombay High Court (Nagpur Bench).

Project published under the **Wolfgang Rush** open-source brand for legal-tech infrastructure.
