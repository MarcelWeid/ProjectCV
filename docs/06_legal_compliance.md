# Legal & Compliance — ProjectCV

**Document status:** Draft v0.1
**Owner:** [You]
**Last updated:** 2026-05-12
**Companion documents:** 01_project_charter.md, 02_requirements_specification.md, 03_solution_architecture.md, 04_threat_model.md, 05_rag_design.md

---

## ⚠️ Important caveat

This document is a working framework for thinking about the legal and compliance obligations of ProjectCV. **It is not legal advice.** It is written by a project-management collaborator (an AI assistant) with no licence to practise law in Germany, the EU, or anywhere else.

The two artefacts the site must publish — the Impressum and the privacy notice (Datenschutzerklärung) — should be reviewed by a German-qualified lawyer before publication, or at minimum generated using a reputable Impressum-/Datenschutz-Generator (e.g. eRecht24, Dr. Schwenke, Datenschutz-Generator.de) and adapted to this project's specifics. The cost of getting these wrong is real (Abmahnung from competitors or rights-holders, fines under §13 DDG / §41 BDSG, GDPR enforcement actions). The cost of getting them reviewed is small in comparison.

Everything below is operationally useful — the checklists, the structure of what each artefact must contain, the per-company scrape review process. The final wording is on you and your reviewer.

---

## 0. About this document

Three regulatory regimes converge on ProjectCV:

1. **GDPR (Regulation 2016/679)** — applies the moment a recruiter's IP address touches the server. Defines what we may do with personal data and what we must tell people about it.
2. **EU AI Act (Regulation 2024/1689)** — transparency obligations under Article 50 apply to AI systems that interact with natural persons. ProjectCV is squarely in scope.
3. **German national law** — the Impressum requirement under §5 DDG (formerly §5 TMG), the data-protection elements of the BDSG, and the general civil-law exposure for scraping.

Plus a fourth, transversal concern: **scraping**, which sits across copyright (UrhG), database rights (UrhG §§87a–87e), terms of service (contract law), and trade-mark law.

The document is structured by regime, then by artefact. Each section ends with a concrete operational output: a checklist, a template outline, or a decision rule.

### 0.1 Scope reminder

- Single site, single language to the regulator (German law applies because the site targets the German market and the candidate operates from Germany).
- One operator: the candidate. No employees, no third-party processors except Azure (Microsoft Ireland), Hetzner (Hetzner Online GmbH, Germany), and whoever the candidate uses for transactional email if any.
- No marketing tracking, no analytics, no advertising — by design (CR-011).
- Two classes of "users": recruiters who chat, and the candidate themselves as operator.

---

## 1. GDPR posture

### 1.1 What personal data is processed

This is the question every privacy notice opens with. Here's the honest inventory:

| Data | When | Why | Legal basis | Retention |
|---|---|---|---|---|
| IP address (raw) | Every HTTP request | Routing, abuse prevention | Art. 6(1)(f) — legitimate interest in operating the service securely | Not stored raw beyond the request; HMAC-hashed value retained in logs |
| Hashed IP (HMAC) | Every chat request | Rate limiting, abuse detection | Art. 6(1)(f) — legitimate interest | 30 days (SR-032) |
| Token-hash | When `?c=` is present | Identify which recruiter/company token is in use | Art. 6(1)(f) — legitimate interest | 30 days for logs; token record itself retained until revoked |
| Message content | During the active request only | Generating the response | Art. 6(1)(f) — legitimate interest (the recruiter knowingly initiated the chat) | **Not retained.** Discarded at end of request. (DR-020) |
| Verbose logs (message content) | Only when `LOG_VERBOSE=true` | Debugging | Art. 6(1)(f) — legitimate interest, narrow | Off by default; if on, ≤ 30 days |
| Browser local state (chat history) | While the tab is open | UX continuity | Not stored server-side, so Art. 6 doesn't engage server-side | Cleared on reload / "clear chat" |
| Candidate's own data (the corpus) | Always | The whole point of the site | The candidate is the data subject; legal basis is "it's about themselves and they put it there" | Until the candidate removes it |
| Recruiter PII *inadvertently* typed into chat (e.g. their own name) | If they type it | Side-effect, not requested | Not requested, not retained — see "message content" | Not retained beyond the request |

**Three things are deliberately *not* in this table** because we don't process them:
- Cookies for analytics/tracking. None.
- Email addresses, names, phone numbers, anything from the recruiter beyond what they type. Not collected, no field, no form.
- Profiling or automated decision-making about the recruiter. We don't classify them, we don't score them, we don't decide anything about them. Art. 22 GDPR doesn't engage.

### 1.2 Roles under GDPR

| Role | Entity | What it means |
|---|---|---|
| **Data controller** | The candidate (natural person) | Decides why and how data is processed. Liable. |
| **Data processor — infrastructure** | Hetzner Online GmbH | Hosts the VM and database. AVV (Auftragsverarbeitungsvertrag / DPA) required. Hetzner publishes a standard one. |
| **Data processor — LLM** | Microsoft Ireland Operations Ltd (Azure OpenAI) | Processes message content during inference. AVV required. Microsoft's DPA is part of the Online Services Terms. |
| **Data processor — alerts** | TBD (depending on Th3 / OR-003 decision) | If a transactional email service (Postmark, Brevo, etc.) is used, AVV required. **Decision still open.** |
| **Third country?** | No, if Azure is configured to EU region and EU data residency. Yes if not. | Drives whether SCCs / TIA are needed for Azure. |

**Operational tasks:**

- [ ] Sign Hetzner's AVV (online, at registration or in account settings).
- [ ] Confirm Microsoft Azure OpenAI is configured for EU region (West Europe / Sweden Central) and that data residency is enforced.
- [ ] Microsoft's DPA is accepted automatically with Azure; verify and keep a PDF copy.
- [ ] Decide on alerts channel (Th3); if a third-party service, sign its DPA.

### 1.3 Data-subject rights

A recruiter is, in GDPR terms, a data subject. Their rights apply:

| Right | How we satisfy it |
|---|---|
| Art. 15 — access | We hold almost nothing. A subject access request gets the answer "we process: your hashed IP, hashed token, request timestamps and metadata; we do not store message content; here is the period of retention." Provide via email response within one month. |
| Art. 16 — rectification | Not meaningfully applicable (no profile to rectify). |
| Art. 17 — erasure | The 30-day rolling retention satisfies this by default. On request: identify the logs by hashed values they can prove correspond to them (difficult — accepted) and delete. |
| Art. 18 — restriction | On request, set the daily kill-switch and/or revoke the relevant token. |
| Art. 20 — portability | Not applicable (no structured profile). |
| Art. 21 — objection | Same as restriction. |
| Art. 77 — complaint to supervisory authority | Named in the privacy notice. For DE residents: the data protection authority of their Land. |

**The candidate's email for data-subject requests** should be a real, monitored mailbox. The privacy notice will name it. SRS open item: confirm which mailbox (Th3 is adjacent — alerts mailbox could double as the DSR mailbox).

### 1.4 What we tell the data subject

The privacy notice (§4 below) is the artefact. Article 13 GDPR lists the mandatory contents; the template in §4.2 covers each.

### 1.5 What we do *not* do

For symmetry with §1.1, the things we deliberately don't do — and that we therefore don't need a legal basis or disclosure for:

- No profiling (Art. 22) — no automated decisions about the recruiter.
- No special-category data (Art. 9) — health, biometric, political views, etc. If a recruiter chooses to type such data, we don't retain it. The bot is instructed not to ask for it.
- No cross-border data transfers outside the EU/EEA — provided Azure is configured to an EU region.
- No data sales, no data sharing, no third-party advertising.

Each of these is a one-line affirmation in the privacy notice ("We do not...") and saves a lot of disclosure obligation.

---

## 2. EU AI Act

The AI Act entered into force in August 2024 with staged enforcement. The provisions relevant to ProjectCV are in force as of **2 February 2025** (prohibited practices and AI literacy) and **2 August 2026** (general-purpose AI obligations and most transparency requirements). The transparency duties in **Article 50** — which is where ProjectCV sits — are effective from 2 August 2026.

### 2.1 Risk classification

ProjectCV is **not** a high-risk system under Annex III. It's also not a prohibited practice under Article 5. It falls under Article 50 (transparency obligations for certain AI systems).

Concretely:

- **Article 50(1)** — "providers shall ensure that AI systems intended to interact directly with natural persons are designed and developed in such a way that the natural persons concerned are informed that they are interacting with an AI system." This applies to us.
- The exception ("unless this is obvious from the point of view of a natural person who is reasonably well-informed, observant and circumspect") does *not* save us — a chat interface is not obviously AI to a non-technical recruiter.

**Decision rule:** the AI disclosure must be present, plain-language, and visible before the first message is sent (FR-006). Burying it in the privacy notice doesn't count.

### 2.2 Provider vs deployer

Under the AI Act, the candidate is both:

- **Provider** of the chatbot system (built and operated by the candidate).
- **Deployer** of the underlying GPT-4o model (which is the provider's — Microsoft/OpenAI's — GPAI model).

Most of the heavy obligations (technical documentation, conformity assessment, registration) apply to high-risk systems, not Article-50-only systems. For us the obligation reduces to: **disclose, document the system in a basic way (which this set of docs already does), and don't claim it does things it doesn't.**

### 2.3 What we do operationally

- [x] AI disclosure on first chat load (FR-006).
- [x] System docs in the public repo — this set of documents *is* the technical documentation.
- [ ] Add a brief "what this system is" paragraph to the site footer, linking to the GitHub repo (already planned per A5 resolution in the Architecture doc).
- [ ] No claims of human equivalence or impersonation — the bot is framed as an *assistant introducing the candidate*, not as the candidate. The persona rule (FR-020) handles this.

### 2.4 Output marking (Article 50(2))

Article 50(2) requires providers of generative AI systems to mark outputs as artificially generated, in a machine-readable way (e.g. watermarking) where technically feasible. **This obligation falls on the *model provider* (Microsoft/OpenAI), not on the *application*.** ProjectCV passes through GPT-4o output; we are not the GPAI provider.

We are not obligated to add watermarks ourselves. But citing the sources of the answer (FR-015) achieves a related goal — making the provenance of the answer transparent — and that's worth keeping for trust reasons even though it isn't legally required of us.

### 2.5 AI literacy (Article 4)

Article 4 obliges providers and deployers to take measures to ensure a "sufficient level of AI literacy" of their staff and persons dealing with the operation on their behalf. For a one-person project, this collapses to: the candidate should know what they're doing. Reading these documents and the OWASP LLM Top 10 satisfies it in substance; no separate training programme needed.

---

## 3. German national law

### 3.1 Impressum (§5 DDG)

**Non-negotiable.** Any site that targets a German audience and is operated commercially-or-similarly (the bar is low — "telemedia" with a recognisable provider) must publish a German-law-compliant Impressum, reachable from every page in no more than two clicks.

Even though ProjectCV is a personal portfolio rather than a commercial offering, the safe assumption is that the Impressum requirement applies. Bandbreite Abmahnung-Kultur in DE makes "no, mine doesn't count" a losing argument; the cost of adding an Impressum is zero.

**Mandatory contents per §5 DDG:**

- [ ] Full legal name (Vor- und Nachname for natural persons)
- [ ] Postal address (a real, deliverable address — not a PO box for §5 DDG, though PO box may suffice for some content; safest is the candidate's residential or business postal address)
- [ ] Contact: email and either a phone number or other "rapid contact" channel
- [ ] (If applicable) Tax ID under §27a UStG — for a private portfolio with no commercial offering, typically not applicable
- [ ] (If applicable) Trade register entry — not applicable for a private individual
- [ ] (If applicable) Professional regulation body — not applicable here

**Privacy-friendly address options.** Publishing one's home address is uncomfortable but generally required. Two mitigations:

1. **Use a c/o address.** A friend or family member with a separate address can serve.
2. **Use an Impressumservice / Ladungsfähige Anschrift service.** Several providers (e.g. Autorenwelt for authors, Impressum-Service for individuals) provide a forwarding address for ~€10–30/month. Not all services are accepted as legally sufficient — check the specific provider's Rechtssicherheit claim and supporting case law.

**Decision required.** The candidate decides the address question before publishing. The template in §3.2 leaves it as a placeholder.

### 3.2 Impressum template

To live at `/impressum` on the site and be linked from every page footer.

```
Angaben gemäß § 5 DDG

[Vollständiger Name]
[Straße und Hausnummer / c/o-Adresse]
[PLZ Stadt]
Deutschland

Kontakt:
E-Mail: [eine erreichbare Adresse]
Telefon: [optional, sonst eine andere zeitnahe Kontaktmöglichkeit]

Verantwortlich für den Inhalt nach § 18 Abs. 2 MStV:
[Vollständiger Name, Anschrift wie oben]

Haftungsausschluss:
Die Inhalte dieser Seite werden mit größtmöglicher Sorgfalt erstellt. Der Anbieter
übernimmt jedoch keine Gewähr für die Richtigkeit, Vollständigkeit und Aktualität
der Inhalte. Die Nutzung der Inhalte erfolgt auf eigene Gefahr.

KI-Hinweis:
Der Chatbot auf dieser Seite ist ein KI-System auf Basis von Azure OpenAI (GPT-4o).
Antworten werden automatisiert erzeugt und können Fehler enthalten. Verbindliche
Auskünfte erteilt ausschließlich [Name] persönlich.
```

The "KI-Hinweis" addition is not strictly mandatory in the Impressum (it's the privacy notice's territory), but bundling it here in DE-flavoured language is good practice and adds one more place where the AI disclosure is visible.

### 3.3 Datenschutzerklärung (privacy notice)

See §4 below. Lives at `/datenschutz` and is linked from every page footer.

### 3.4 Other German specifics

- **Cookie law (TTDSG, since 2021).** We use no cookies (no analytics, no tracking). A cookie banner is therefore not required. The privacy notice will state this affirmatively.
- **LfDI / Aufsichtsbehörde.** The competent data protection authority is the LfDI / DSB of the candidate's federal state (e.g. LfDI BW for Baden-Württemberg, BlnBDI for Berlin, BfDI federally). Named in the privacy notice.
- **Telemedia / Mediendienste registration.** Not required for a personal site.

---

## 4. Privacy notice (Datenschutzerklärung)

The single most important compliance artefact, after the Impressum. Article 13 GDPR is the legal anchor; §4.2 below is an outline ordered by the Article 13 catalogue.

### 4.1 Where it lives

- `/datenschutz` on the site
- Linked from every page footer
- Linked from inside the chat UI (next to or under the AI-disclosure banner) — this is the FR-006-adjacent link

### 4.2 Outline

Each section is the *structure* of what to write; the actual text should be drafted by the candidate (or a lawyer / generator) using project specifics.

**1. Verantwortlicher (Controller — Art. 13(1)(a))**

> Same name and contact as the Impressum.

**2. Datenschutzbeauftragter (DPO — Art. 13(1)(b))**

> Not required: a one-person controller with no large-scale or special-category processing falls below the §38 BDSG threshold. State so explicitly.

**3. Zwecke und Rechtsgrundlagen (Purposes and legal bases — Art. 13(1)(c))**

For each of:

- Operating the chatbot (Art. 6(1)(f) — legitimate interest in providing the service the recruiter requested)
- Abuse prevention and security (Art. 6(1)(f) — legitimate interest in protecting the service)
- Optional: technical logs (Art. 6(1)(f))

For each: name the legitimate interest concretely.

**4. Empfänger und Drittländer (Recipients and third countries — Art. 13(1)(e), (f))**

- Hetzner Online GmbH, Germany — hosting and database. AVV in place. No third-country transfer.
- Microsoft Ireland Operations Ltd — Azure OpenAI inference. AVV in place via Microsoft's DPA. EU region configured (West Europe / Sweden Central). State whether any sub-processor in a third country is involved; Microsoft's published list is the reference.
- (If applicable) Alerts provider — to be named once Th3 is resolved.

**5. Speicherdauer (Retention — Art. 13(2)(a))**

> Verbatim per the table in §1.1 above:
> - Chat content: not retained beyond the active request.
> - Hashed IP / token in logs: 30 days, then deleted.
> - Tokens themselves: until revoked.

**6. Betroffenenrechte (Data-subject rights — Art. 13(2)(b))**

> Recite Art. 15–21. Name the controller's email for requests. State that exercising these rights is free.

**7. Beschwerderecht (Right to complain — Art. 13(2)(d))**

> Right to lodge a complaint with the competent supervisory authority. Name the candidate's Land authority.

**8. Erforderlichkeit (Whether providing data is required — Art. 13(2)(e))**

> Not required to enter any personal data. The chat works without personally identifying yourself. Some data (IP) is inevitably processed by virtue of the connection.

**9. Automatisierte Entscheidungsfindung (Automated decision-making — Art. 13(2)(f))**

> No automated decision-making in the Art. 22 sense. The bot answers questions; it does not score, rank, or decide about the recruiter.

**10. Cookies & Tracking**

> No cookies. No analytics. No tracking. No social-media plug-ins. State affirmatively.

**11. Hinweis zu KI / AI Notice**

> Bridge to the AI Act disclosure. State plainly:
> - The chatbot is an AI system.
> - It uses GPT-4o (Azure OpenAI) for generation, and `text-embedding-3-large` for retrieval.
> - It is grounded in pre-curated information about the candidate, and (when a company-scoped link is used) about a specific company.
> - Answers may contain errors. Binding statements come from the candidate personally.
> - The candidate does not use chat content for training or for any purpose beyond responding to the request.

**12. Kontakt zu Datenschutzfragen**

> Email for DSR and privacy questions. Same as Impressum or separate, candidate's choice.

**13. Änderungen dieser Erklärung**

> Date of last update; statement that the controller may update the notice; how changes will be communicated.

### 4.3 Practical generation path

Realistic ways the candidate can get to a publishable text:

1. **eRecht24 Datenschutz-Generator** (paid). Long-established; widely accepted; offers a German-language wizard.
2. **Datenschutz-Generator.de** (paid). Updated for current case law; well-regarded.
3. **Hand-write from the outline above and have it reviewed.** Lower cost in money, higher cost in time and risk.

**Recommendation:** use a generator, then adapt sections 4.4 / 4.11 (the AI-specific parts) by hand using the language in this document. Generators don't yet handle bespoke AI services well.

---

## 5. Scraping — the per-company review framework

The architecturally interesting question. The legal landscape for web scraping in Germany / EU is unsettled in places. Three layers to consider per company:

### 5.1 The three legal hooks

**Copyright (UrhG).** A website's text, images, and structure can be subject to copyright. Scraping the text of a public webpage to extract excerpts is generally permissible under §§44a–60h UrhG (temporary copies, quotation, text-and-data mining), particularly:

- **§44a — Temporary acts of reproduction.** Allows transient copies necessary for technical processing.
- **§44b — Text and data mining.** Permits reproduction of lawfully accessible works for the purpose of text and data mining unless the rightholder has expressly reserved use in a *machine-readable* way. The "machine-readable opt-out" is a key concept — `robots.txt` and similar are arguably such a reservation, though case law is still developing.
- **§51 — Quotation.** Permits quoting from works to the extent justified by the purpose.

**Operational rule:** respect `robots.txt` strictly (CR-020); honour any TDM reservation that's discoverable; store *excerpts* (chunks) rather than verbatim full pages.

**Database rights (UrhG §§87a–87e).** Protects the "substantial investment" in a database. Mostly relevant for structured data sources (e.g. a company's catalogue). Our scraping is corporate-marketing-content focused, which is unlikely to meet the §87a definition of a protected database.

**Terms of Service (contract law).** A site's ToS may prohibit scraping. Whether this binds a non-logged-in scraper is legally contested (no consideration, no acceptance, no individualised agreement). Conservative position: read each target company's ToS, and if it explicitly prohibits scraping, don't scrape — pick a different target. Aggressive position: ToS not validly incorporated absent acceptance. **Conservative position is the right one for a portfolio site.** Reputational risk dominates legal risk here.

### 5.2 Per-company legal review checklist

Lives in the company's `companies/<id>.yml` file (RAG doc §2.2). The checklist below is what the `legal_review` block must answer. The scraper refuses to run on a company until this block is complete and `reviewed_at` is within 90 days.

```yaml
legal_review:
  reviewed_at: <ISO date>
  reviewer: <your name>

  # Required answers
  source_classification: <one of:
    "public marketing content"            # corporate about/products/values pages — usually OK
    "press release / public news"         # explicit press content — usually OK
    "blog post / opinion piece"           # marginal — depends on terms
    "documentation / manuals"             # check copyright explicitly
    "user-generated / community content"  # avoid — third-party IP risk
    "paywalled / login-required"          # MUST NOT scrape
    "social media / aggregator"           # avoid — platform ToS risk
  >

  robots_txt_checked: <true/false>
  robots_txt_allows_target_paths: <true/false>

  tos_url: <URL of the target site's ToS>
  tos_explicitly_prohibits_scraping: <true/false>
  tos_explicitly_prohibits_ai_training: <true/false>     # increasingly common — note "TDM opt-out"
  # if either of the above is true → DO NOT scrape this company

  tdm_opt_out_machine_readable: <true/false>             # e.g. /.well-known/tdm-reservation,
                                                          # ai.txt, X-Robots-Tag headers
  # if true → DO NOT scrape

  copyright_concerns:
    level: <none / low / medium / high>
    notes: <free text — e.g. "site uses Creative Commons BY-NC", or
                          "press releases marked 'for reproduction'", or
                          "looked clean">

  pii_in_content:
    level: <none / low / medium / high>                 # leadership photos with bio? team page?
    notes: <free text>
  # if level >= medium → scope `deny_paths` to exclude people-pages

  trademark_concerns:
    notes: <free text — e.g. "Acme™ used throughout; we'll use the same usage in citations">

  decision: <one of:
    "proceed"
    "proceed with reduced scope"
    "do not scrape — replace with another company">

  reduced_scope_paths_added_to_deny: <list of paths if applicable>
```

The checklist makes the legal judgment **explicit and auditable**. It also makes the wrong choice obvious: if the reviewer cannot tick the right boxes, the scraper refuses.

### 5.3 What we do at the technical layer to support compliance

Already specified in the SRS and RAG design; recapping the controls that pull weight legally:

- `User-Agent` is identifiable and points to a contact URL (FR-033).
- `robots.txt` is fetched and respected (FR-032, CR-020).
- Crawl is bounded: `same_origin_only`, `max_depth`, `max_pages`, `allow_paths` and `deny_paths` per company (RAG §2.2). Aggressive crawl behaviour is the most likely thing to draw a complaint, separately from legality.
- Polite request rate: 1 request per second per host, configurable. Below this is fine; above is needlessly aggressive.
- We store chunks, not full pages. The longest stored excerpt is ≤ 800 tokens (RAG §3.2), well within "excerpt" rather than "copy."
- Citations attribute back to the source URL (FR-015). When the recruiter clicks the citation, they go to the original. We are sending traffic *to* the source, not laundering it.

### 5.4 What we do *not* scrape, explicitly

A negative list, committed to the repo as `companies/_global_deny.md`:

- Paywalled or login-required pages
- Social media (LinkedIn, Twitter/X, Facebook, etc.) — platform ToS, regardless of company's wishes
- Aggregators (Glassdoor, Kununu, Indeed, etc.) — third-party platform content
- PDFs of unclear provenance
- Annual reports unless explicitly published as press content
- Anything behind a captcha (legally a clear signal of "you're not welcome")
- Anything that would require JavaScript execution to render — outside our scraper's design, and crossing that line tends to indicate non-public content

### 5.5 Special handling for one's *own* employer

The candidate's current and past employers may overlap with target companies. Three rules:

1. Never scrape internal-only content the candidate has access to via their employment. The scraper accepts the same URLs as a non-logged-in stranger.
2. Don't include content the candidate signed an NDA about, even if it's been incidentally indexed publicly. The legal review checklist should flag this.
3. Past employers can be in the company list if they're a realistic recruiter target now. Be straightforward about the prior relationship in the candidate corpus (the bot will naturally reference it).

---

## 6. Operational compliance checklist before launch

Single consolidated pre-launch checklist. Each item maps to where it's defined.

**Disclosure & transparency:**

- [ ] AI-disclosure banner on the chat UI, visible before first message (FR-006 / AI Act Art. 50(1))
- [ ] Impressum at `/impressum`, linked from every page footer (§3.1)
- [ ] Privacy notice at `/datenschutz`, linked from every page footer and from the chat UI (§4)
- [ ] "Built by [name], source on GitHub →" link in the footer (A5 resolution)

**DPAs / AVVs in place:**

- [ ] Hetzner AVV signed
- [ ] Azure / Microsoft DPA reviewed and kept on file
- [ ] Alerts-channel DPA (if applicable, depends on Th3)

**Azure config:**

- [ ] Region set to EU (West Europe or Sweden Central)
- [ ] Data residency confirmed in Azure portal
- [ ] Training-on-data disabled (CR-013)

**Per-company readiness (per company before scraping):**

- [ ] `legal_review` block in `companies/<id>.yml` complete and dated within 90 days
- [ ] `robots.txt` checked manually as well as programmatically
- [ ] Scraper dry-run reviewed (FR-034 audit log inspected)

**Mailbox:**

- [ ] A real, monitored mailbox is named in the Impressum and privacy notice for DSR and contact
- [ ] Auto-reply (optional) acknowledges receipt within one working day

**Documentation:**

- [ ] This document, the threat model, and the architecture are public in the repo (substantively the "AI Act technical documentation" obligation)

---

## 7. Things this document deliberately doesn't try to do

Honest list:

- **Final wording for the Impressum or privacy notice.** Use a generator and/or a lawyer.
- **Legal advice on whether to scrape a specific company.** The per-company checklist is a *framework* for the candidate's judgment, not a verdict.
- **Resolution of the contested points in EU scraping law** (TDM opt-out machine-readability standard, ToS-binding-without-acceptance). The framework is conservative on these; that's the right posture for a portfolio site, not a research piece.
- **Cross-jurisdictional compliance.** This document assumes the candidate operates from Germany and the site primarily addresses a German-speaking audience. Recruiters from elsewhere may have local rights too; we address them under GDPR generally and trust that's sufficient at this scale.
- **Compliance with non-EU AI regulations.** If the site ends up reaching a US, UK, or non-EU audience materially, that's a separate review. Not in scope for MVP.

---

## 8. Open questions

| # | Question | Owner | Affects |
|---|---|---|---|
| L1 | ~~Impressum address strategy~~ **Resolved 2026-05-13 in DEC-007.** Wohnanschrift + Mobilnummer + E-Mail. Privacy-Trade-off bewusst akzeptiert (Adresse wird öffentlich trotz aktiver WHOIS-Privacy beim Registrar). | Done | Done |
| L2 | Mailbox for DSR / contact — new dedicated address, or candidate's existing email? | You | §4.2 §12, OR-003 (Th3 adjacent) |
| L3 | Privacy notice generation — eRecht24 generator (~€10/mo), Datenschutz-Generator.de (one-off ~€100), or lawyer review (~€300+)? | You | §4 |
| L4 | ~~The first three pilot companies for phase 1 — names?~~ **Resolved 2026-05-13 in DEC-008.** SEW-Eurodrive, DKFZ, Exxeta. Skelett-Configs in `companies/` erstellt; konkrete `legal_review`-Blocks und Crawl-Pfade folgen pre-flight vor TASK-107. | Done | Done |
| L5 | Should the privacy notice mention the GitHub repo as part of the openness/transparency story, or treat it as out-of-scope to the privacy notice? | You | §4.2 |
| L6 | ~~Default on ambiguous robots.txt / ToS?~~ **Resolved 2026-05-12: Conservative default — do not scrape on ambiguity.** If `robots.txt` is unclear, if ToS is silent rather than permissive, if TDM permissions cannot be confirmed, the company is either excluded or scoped to clearly-public paths only (press releases, official "about" page, values page). The reasoning: the project's success metric is recruiter outreach, and a single "you scraped us without permission" reply kills that outcome regardless of whether it would have been legally defensible. The marginal target-set reduction is acceptable. The per-company `legal_review` checklist (§5.2) operationalises this — under `decision`, ambiguity maps to `"proceed with reduced scope"` or `"do not scrape — replace with another company"`, never to unrestricted `"proceed"`. | Done | §5.2 |
