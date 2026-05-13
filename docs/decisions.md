# Execution Decisions

A lightweight log of non-trivial decisions made during execution that are too small for a full ADR but too consequential to leave undocumented. Larger architectural decisions live in `docs/adr/`; planning-phase decisions live in their respective planning documents under `docs/`.

Format: each entry has a stable ID (`DEC-NNN`, never reused), the decision itself, the reasoning, the alternatives considered, and a note on how reversible it is.

---

## DEC-001 — Public URL and project naming (2026-05-12)

**Decision:**

- **Public URL:** `marcel.weidemann.family/cv`
- **Internal codename:** *ProjectCV* (placeholder, final name deferred until TASK-101 when it actually surfaces in code namespaces)
- **No separate product brand.** The site is descriptive, not branded.

**Why this URL:**

- Already-owned `weidemann.family` removes domain procurement entirely.
- The `marcel.*` subdomain prefix makes ownership unambiguous and leaves `weidemann.family` open for other family members in the future.
- The `/cv` path is honest about what's there — recruiters land and immediately know they're in the right place.
- No branding layer between the recruiter and the candidate; no risk of trademark collision or naming dating badly.

**Why no separate product name:**

- The "showcase" angle is delivered through the engineering and the open GitHub repo, not through a brand.
- Internal codename can be chosen later once we see how it actually reads in code (TASK-101); waiting costs nothing.
- *ProjectCV* as a placeholder in planning documents is unambiguous and harmless.

**Considered:**

- Branded project names (Anker, Sprechstunde, Bewerbot, etc.) — rejected: brand adds friction without clear benefit when the URL already names the candidate.
- `cv.weidemann.family` (subdomain instead of path) — rejected: locks `cv.*` at the family-domain root, less compositional.
- `cv.marcel.weidemann.family` (deeper subdomain) — rejected: marginally cleaner separation but more typing and DNS complexity for no real gain.
- New top-level domain — rejected: cost, no benefit, fragments online identity.

**Reversibility:** URL is moderately reversible (DNS change + a few config strings + redirect from old to new). Codename trivial to change in week 1, painful after Phase 2.

**Downstream effects:**

- TASK-002 reduces from "register a domain" to "add one DNS A-record"; can be deferred to Phase 5.
- Impressum (legal §3.2) names the URL but does not need a separate brand block.
- Privacy notice URL is `marcel.weidemann.family/datenschutz`; Impressum is `marcel.weidemann.family/impressum`.
- README and bot self-reference both use "Marcel's CV assistant" framing rather than a product name.

---

<!-- Add new decisions below as DEC-002, DEC-003, ... -->

## DEC-002 — VM vor DB provisioniert (2026-05-12)

**Decision:** Hetzner-VM in Phase 0 statt Phase 5 provisioniert und gehärtet.

**Why:** Manuell vorgezogen während der ersten Arbeitssession. Gerade Lust gehabt, lief gut. VM steht idle bis Phase 5, kostet im CX22-Tarif ~€4.50/Monat. Härtung jetzt verteilt die Arbeit; in Phase 5 muss "nur noch" deployt werden — kein Härtungs-Schritt mehr.

**Considered:**
- Plangemäß bis Phase 5 warten — verworfen, weil Motivation und Zeit da waren.
- Jetzt provisionieren und Härtung verschieben — verworfen, weil ungehärtete VM im offenen Netz ein konkretes Risiko ist (SSH-Brute-Force binnen Minuten).

**Reversibility:** Kostenseitig ~€13 über die ~3 Monate Idle-Zeit. Konfigurationsseitig durch Snapshot abgesichert.

**Notes:**
- Erste Provisionierung versehentlich mit Passwort-Auth — verworfen, Server gelöscht, neuer Server mit SSH-Key gleich bei Erstellung angelegt.
- Lernpunkt: SSH-Key bei Hetzner-Serverstellung im Erstellungsdialog hinterlegen, dann ist die Hälfte der Härtungsarbeit schon getan.
- TASK-501 ist damit faktisch abgeschlossen; TASK-502 (Ansible-Bootstrapping) bleibt für Phase 5 als reproduzierbares Setup-Skript, aber der akute Bedarf ist weg.
- Phase 5 verkürzt sich um ~3h.

**Effects on plan:**
- WBS TASK-501: ✓ done
- WBS TASK-502: noch offen für Phase 5 (Ansible-Skript für Reproduzierbarkeit, nicht für akute Härtung)
- WBS TASK-002 (DNS A-Record): bleibt für Phase 5
- Phase 5 ist netto ~3h leichter

---

## DEC-003 — Azure-OpenAI-Setup unter Firmen-VS-Subscription (2026-05-13)

**Decision:**

- **Azure-Subscription:** Visual-Studio-Subscription über Firmen-Account (`marcel.weidemann@agilent.com`).
- **Tenant:** Firmen-Tenant.
- **Resource Group:** `marcel-private-projectcv-rg` mit eindeutigen Tags (owner, project, purpose) — signalisiert Privatcharakter.
- **Region:** West Europe.
- **Deployments:**
  - `gpt-4o`, Version 2024-11-20, Deployment-Type **Data Zone Standard (EUR)**, 300K TPM / 1.8K RPM, Auto-Upgrade aktiv.
  - `text-embedding-3-large`, Version 1, Deployment-Type **Standard**, 350K TPM / 2.1K RPM, kein Auto-Upgrade.
- **Modellwahl bestätigt ADR-002:** weiterhin GPT-4o end-to-end mit text-embedding-3-large.

**Why diese Subscription-Wahl:**

- Anstellungsvertrag erlaubt karrierebezogene Side-Projects ausdrücklich.
- VS-Subscription-Guthaben darf privat genutzt werden.
- Kontinuität nach möglichem Jobwechsel ist nicht kritisch — der Bot darf mit dem Account-Ende verschwinden.
- Hetzner (DB, Server, Recruiter-Daten) liegt komplett privat — die *datenverarbeitende* Hälfte ist sauber privat. Azure liefert nur LLM-Inferenz, ohne Inhalte zu persistieren.
- Tags und dedizierte Resource Group machen den Privatcharakter im Firmen-Tenant unmissverständlich sichtbar.

**Why GPT-4o trotz GPT-5-Familie verfügbar:**

- ADR-002 ist gut durchdacht; ein Wechsel würde Tuning-Aufwand und Re-Eval erfordern, ohne dass klare Vorteile dokumentiert sind.
- Sichere Wahl mit bekannten Charakteristiken (Function-Calling, Latenz, Kosten).
- Kostenmodell aus 03_solution_architecture.md §6.5 bleibt unverändert gültig.
- Upgrade auf neuere Modelle (gpt-4.1, gpt-5-mini, gpt-5.1) bleibt für v1.1+ offen — NFR-031 garantiert Config-only-Switch ohne Code-Änderung.

**Why "Data Zone Standard (EUR)" statt "Standard":**

- Reines "Standard" (regional, einzelne Region) wird von Azure für gpt-4o aktuell nicht angeboten — nur "Data Zone Standard" oder "Global Standard".
- "Data Zone Standard (EUR)" hält Daten innerhalb der EU-Datenzone → GDPR / CR-013 (Datenresidenz) gewahrt.
- "Global Standard" wäre für unsere Compliance-Anforderung nicht akzeptabel.
- Embedding-Modell konnte als reines "Standard" deployed werden — keine Anpassung nötig.

**Considered:**

- Privates Microsoft-Konto + neue Pay-as-you-go-Subscription anlegen — verworfen, weil VS-Guthaben da ist und Vertrag privaten Side-Project-Einsatz erlaubt. Bei Guthaben-Ende oder Vertrags-/Firmenwechsel re-evaluieren.
- gpt-4o-mini statt gpt-4o — verworfen für MVP, weil ADR-002 bewusste Qualitätswahl ist; Wechsel via Config möglich falls Eval später anders entscheidet.
- gpt-5-Familie — verworfen für MVP wegen unklarer Daten und Tuning-Aufwand; Backlog-Eintrag für v1.1+.

**Reversibility:**

- Modellwahl: hoch (Config-Wechsel, NFR-031).
- Subscription-Wechsel: moderat (Tenant-Wechsel würde neue Resource + neue Keys + Re-Deployment bedeuten).
- Deployment-Type: niedrig (Azure-Restriktion).

**Effects on plan:**

- WBS TASK-006 (Azure-OpenAI-Quota): done in einer Session statt 1-10 Werktage Wartezeit.
- Phase 0 deutlich verkürzt.
- 03_solution_architecture.md §6.5 Kostenmodell bleibt unverändert gültig.
- 06_legal_compliance.md §1.2: Azure-Processor-Eintrag stimmt; EU-Region und Training-Opt-out via Standard-Konfiguration gegeben.
- **Neues Risiko R9 (Modell-Retirement Okt 2026)** → siehe WBS-Update; Migrations-Session Sommer 2026 vormerken.

**Notes:**

- Erste Resource-Erstellung landete versehentlich ohne dedizierte Resource Group / Tags — bemerkt, gelöscht, neu mit Tagging-Konvention angelegt.
- Modell-Retirement Oktober 2026 für gpt-4o: nicht akut (MVP wird vorher fertig), aber Migration für Sommer 2026 als R9 vorgemerkt.
- Embedding-Retirement April 2027: deutlich entspannter, Embedding-Wechsel würde Re-Indexierung erzwingen.
---

## DEC-004 — Postgres + pgvector self-hosted via Docker (2026-05-13)

**Decision:** Replace planned Hetzner Managed Postgres with self-hosted Postgres 17 + pgvector 0.8.2 running in Docker via Compose. Local dev instance runs in WSL; production instance will be deployed to the Hetzner VM in Phase 5 using the same Compose file with different environment variables.

**Why:** Hetzner Managed Postgres does not offer pgvector in a form usable for this project. Self-hosting via the official `pgvector/pgvector:pg17` image is the lowest-friction replacement that preserves all design properties of ADR-001 (hybrid retrieval in one table, EF Core compatibility, known tooling).

**Considered alternatives:** see ADR-001 (revised).

**Reversibility:** High at MVP stage. Schema, queries, and EF Core mappings remain compatible across any Postgres + pgvector deployment. Migration to a managed alternative later would be straightforward if the market evolves.

**Effects on plan:**

- Verifies pgvector 0.8.2 working locally with cosine and L2 distance operators (TASK-005b).
- TASK-005 effectively split: local-dev part done (this session); VM deployment moves to Phase 5 alongside TASK-503.
- ADR-001 revised in `03_solution_architecture.md`.
- **New risk R10 (self-hosted DB operational burden)** added to `07_work_breakdown.md` §10.
- 06_legal_compliance.md §1.2: Hetzner remains data processor (VM-level only); the database is now operated by the controller directly, but the underlying compute is still Hetzner — DSGVO roles unchanged in substance.

**Notes:**

- Container running locally in WSL via Docker Engine (no Docker Desktop). Bind to `127.0.0.1:5432` only — not exposed to other network interfaces.
- Named volume `projectcv-db-data` for persistence across `docker compose down`.
- Init script `001-extension.sql` runs `CREATE EXTENSION vector` on first start; idempotent (`IF NOT EXISTS`).
- Connection string for app config: `Host=localhost;Port=5432;Database=projectcv;Username=projectcv;Password=<from .env>;SslMode=Disable` for local; production will use SSL on the VM.

---

## DEC-005 — .NET 10 statt .NET 9 (2026-05-13)

**Decision:** .NET 10 SDK (LTS) als Entwicklungs- und Runtime-Plattform, statt der ursprünglich in 03_solution_architecture.md geplanten .NET 9.

**Why:**
- .NET 9 war STS (Standard Term Support); Support läuft Mai 2026 aus — genau jetzt.
- .NET 10 ist LTS (3 Jahre Support, bis Nov 2028) und seit Nov 2025 GA.
- Für ein Projekt mit 4-6 Monaten aktiver Entwicklung + Pflege ist LTS klar besser.
- Plan-Edit minimal: nur Versionsangabe in Architektur/WBS anpassen, keine konzeptionelle Änderung.
- SDK war bereits auf der WSL-Dev-Maschine installiert (10.0.104) — keine Installation nötig.

**Considered:** .NET 9 wie geplant — verworfen wegen auslaufendem Support.

**Reversibility:** Hoch — Wechsel auf 9 oder zurück auf 10 wäre eine Target-Framework-Änderung in den `.csproj`-Dateien. Bei Phase-1-Start noch kein Aufwand; später ein Build-Test.

**Effects on plan:**
- 03_solution_architecture.md §0.2 (Stack at a glance): "ASP.NET Core (.NET 9)" → ".NET 10".
- 07_work_breakdown.md TASK-007 angepasst.
- Kein Einfluss auf Architektur-Designentscheidungen, Kostenmodell, Threat Model.

**Notes:**
- Installiert via Standardpaketquelle in WSL (Ubuntu 24.04).
- `dotnet ef` Global Tool: Version 10.0.8.
- End-to-End-Verifikation: .NET 10 → Npgsql → Docker-Postgres 17 → pgvector 0.8.2 erfolgreich.

---

## DEC-006 — Bot-Tonalität, Sprache und Anrede (2026-05-13)

**Decision:**

- **Ton:** "Leicht persönlich, sachlich." Der Bot spricht *über* Marcel mit der Wärme eines Kollegen, der ihn gut kennt — sachorientiert, nicht kumpelig, nicht steril. Erlaubt sind kleine Wertungen ("das war die richtige Entscheidung") und Bezugnahmen aufs Unternehmen, wenn die Quelle es trägt. Verboten bleibt Schönfärben oder Überschwang.
- **Hauptsprache:** Deutsch. Deutsche Recruiter sind Hauptzielgruppe.
- **Sprachwechsel:** Der Bot antwortet in der Sprache der jeweils letzten Recruiter-Nachricht. Englisch funktioniert vollwertig (FR-011). Sprachwechsel mitten in der Konversation wird übernommen.
- **Anrede im Deutschen:** Default ist "Du". Bot beginnt jede Konversation mit "Du". Sobald der Recruiter erstmals siezt (ein einziges "Sie", "Ihnen", "Ihre" o.Ä. genügt), wechselt der Bot ab der nächsten Antwort auf "Sie" und bleibt dort für den Rest der Konversation. Einzelne lockere "du"-Floskeln nach dem Wechsel triggern keinen Rückwärts-Wechsel — Du→Sie ist einseitig.

**Why:**

- Variante 2 (sachlich-persönlich) trifft den Sweet Spot zwischen "kalt" (Variante 1) und "zu meinungsstark" (Variante 3). Hat Wärme, ohne Persönlichkeit zu erfinden.
- Deutsch als Default reflektiert den Markt, den Marcel wirklich anspricht.
- Anrede-Spiegelung passt zur Variante-2-Logik ("guter Kollege") und vermeidet sowohl "zu distanziert für ein Tech-Startup" als auch "zu kumpelig für einen Konzern-Recruiter".
- Du-Default reflektiert die wahrscheinlichste Zielgruppe (Tech-Recruiter, Startup-Kontext). Recruiter, die siezen würden, signalisieren das schnell durch ihre erste Antwort — und der Bot wechselt dann hoch. Konservativ wäre Sie-Default, wirkt aber bei Tech-Audiences leicht steif.
- Einseitiger Wechsel (Du → Sie, nicht zurück): vermeidet wechselhaftes Verhalten, das den Bot inkonsistent wirken ließe. Wenn ein Recruiter zwischendurch ein "du" einstreut, war das nicht zwingend ein Register-Wechsel.
- Du-Default reflektiert die wahrscheinlichste Zielgruppe (Tech-Recruiter, Startup-Kontext). Recruiter, die siezen würden, signalisieren das durch ihre erste Antwort — und der Bot wechselt dann hoch. Konservativ wäre Sie-Default, wirkt aber bei Tech-Audiences leicht steif.
- Asymmetrische Switch-Sensibilität (ein "Sie" genügt; ein einzelnes "du" nicht zurück): deutsche Sprecher siezen selten versehentlich, ein "Sie" ist absichtlich. Ein eingestreutes "du" kann dagegen aus einer Floskel kommen ("kannst du mir nochmal X erklären") und ist kein klares Register-Signal.

**Considered:**

- Variante 1 (formal-professionell) — verworfen als zu kalt; lässt Marcel "abgehoben" wirken.
- Variante 3 (subtile Persönlichkeit) — verworfen, weil der Bot zu sehr eigene Stimme bekommt; Gefahr, dass Recruiter sich auf den Bot beziehen statt auf Marcel.
- Englisch als Hauptsprache — verworfen, weil Hauptzielgruppe deutschsprachig ist; Englisch ist eingebaut, nicht hauptsächlich.
- Festes "Sie" oder festes "Du" — verworfen, weil Tech-/Konzern-Recruiter unterschiedliche Erwartungen mitbringen.

**Reversibility:** Hoch. Alle drei Aspekte sind im Systemprompt konfigurierbar; ein Anredewechsel ist ein Prompt-Edit, keine Code-Änderung (NFR-031).

**Effects on plan:**

- **Phase 2 Systemprompt (TASK-205)** muss umsetzen:
  - Bot-Eröffnung mit "Du" als Default.
  - Erkennen, wenn die letzte Recruiter-Nachricht ein "Sie"-Pronomen oder eine "Sie"-konjugierte Verbform enthält → ab nächster Antwort siezen.
  - Sobald einmal gesiezt: Bot bleibt beim Sie, auch wenn folgende Recruiter-Nachrichten "du"-Formen enthalten.
  - Bei englischen Recruiter-Nachrichten ist die Anrede-Frage moot — Englisch hat nur "you".
- **Phase 3 Frontend-Copy (TASK-302, TASK-303)** sollte zur Tonalität passen — Hero-Text, Placeholder, Disclosure-Banner.
- **Test-Strategie:** Eval-Set sollte Tonalitäts-Prüfung enthalten (manuelle Stichprobe nach §6.1 von 08_test_strategy.md — "war die Antwort im erwarteten Ton?").
- **Red-Team:** keine direkte Auswirkung; Refusal-Texte werden in Phase 2 entsprechend formuliert (FR-014).

**Notes:**

- Bot-Eigenbezug: "Marcels CV-Assistent" (gemäß DEC-001), nicht "Marcel selbst". Wichtig wegen der Spiegelungs-Logik: der Bot ist ein Kollege Marcels, kein Marcel-Impersonator.
- Refusal-Sprache nach FR-014 in Variante-2-Stil: "Da müsste ich passen — das ist in meinen Quellen nicht vermerkt. Marcel kann das aus erster Hand besser beantworten." (Mit Sie/Du je nach Recruiter-Anrede.)

---

## DEC-007 — Impressum-Angaben (2026-05-13)

**Decision:**

- **Adresse:** Eigene Wohnanschrift (vollständig: Straße, Hausnummer, PLZ, Ort).
- **Kontakt:** Eigene Mobilnummer und eine erreichbare E-Mail-Adresse.
- **Beide Angaben sind ladungsfähig und §5-DDG-konform.**

**Why:**

- Wohnanschrift ist die pragmatische Wahl: rechtlich völlig unproblematisch, kostenlos, keine Drittabhängigkeit. Risiko (Stalking, ungewollte Post, Datenbroker-Indexierung) als gering eingeschätzt für die Zielgruppe (Recruiter).
- Mobilnummer wurde ergänzt, nachdem klargestellt wurde: §5 Abs. 1 Nr. 2 DDG plus EuGH-Rechtsprechung (C-298/07) fordern neben E-Mail noch einen zweiten unmittelbaren Kommunikationskanal. Nur-E-Mail wäre eine bekannte Abmahn-Schwachstelle in DE.

**Considered:**

- C/o-Adresse bei vertrauter Person → verworfen, weil keine geeignete dritte Person verfügbar/gewollt.
- Kommerzieller Impressums-Service → verworfen wegen monatlicher Kosten und "Warum braucht der das"-Anmutung bei einem Solo-Karriereprojekt.
- Nur-E-Mail im Impressum → verworfen wegen aktiver Abmahnrisiken in DE.
- Separate Bewerbungs-Telefonnummer (z.B. sipgate basic) → verworfen zugunsten der einfacheren Lösung mit der echten Mobilnummer; bewusste Akzeptanz von etwas Spam-Risiko.
- Kontaktformular mit 24h-Antwortzusage → verworfen, weil zusätzlicher Implementierungsaufwand und Mailbox-Pflicht.

**Trade-offs accepted:**

- **Adresse wird durchs Impressum öffentlich**, obwohl beim Domain-Registrar WHOIS-Privacy aktiv ist. Die Privacy-Maßnahme des Registrars wird damit für diesen Use-Case effektiv ausgehebelt. Verstanden und akzeptiert.
- **Mobilnummer kann Spam-Calls anziehen.** Risiko gering, weil das Impressum nicht die meist-besuchte Seite ist und Bots/Scraper primär die normalen Web-Seiten abgrasen.
- **Kein zusätzlicher Privacy-Layer** für Anschrift oder Telefon — wer sie nutzen will, kann es.

**Reversibility:**

- Mittel. Wechsel auf c/o oder Service wäre eine reine Impressum-Update-Aktion (HTML/MD-Edit, ein Commit, ein Re-Deploy). Plus: Domain-WHOIS-Daten ggf. aktualisieren.
- Wer die alte Version schon archiviert hat (Web-Archive, Caches), behält sie. Das ist die Realität öffentlich publizierter Adressen.

**Effects on plan:**

- **Phase 5 (TASK-506)** Impressum-Finalisierung — Inhalt steht jetzt fest. Verwendung des Templates aus `06_legal_compliance.md` §3.2 mit den entschiedenen Werten.
- **Datenschutzerklärung (TASK-506)** muss konsistent dieselbe Adresse/Mailadresse als Kontakt für DSR-Anfragen nennen.
- **L1 in Open Questions** aus `06_legal_compliance.md` §8 → resolved.

**Notes:**

- Keine anwaltliche Beratung in Anspruch genommen; bei späterem Unbehagen wäre eine 30-min-Rechtsberatung bei einem Fachanwalt (€100-150) gut investiert.
- Die finale Impressums-Datei wird erst in Phase 5 platziert (TASK-506). Bis dahin gibt es das Impressum noch nicht öffentlich — das ist okay, weil die Site selbst noch nicht öffentlich erreichbar ist.

---

## DEC-008 — Drei Pilot-Companies für den ersten Scrape (2026-05-13)

**Decision:** Drei initiale Pilot-Companies für den Scraper und die ersten Recruiter-Tokens:

| Company | Domain | Profil |
|---|---|---|
| SEW-Eurodrive | sew-eurodrive.de | Industrie-Mittelstand, Antriebstechnik (Bruchsal, DE) |
| DKFZ | dkfz.de | Öffentliche Forschungseinrichtung, Krebsforschung (Heidelberg, DE) |
| Exxeta | exxeta.com | Tech-Consulting / IT-Dienstleister (Karlsruhe, DE) |

**Why:**

- **Drei sehr unterschiedliche Profile:** Industrie-Mittelstand, öffentliche Wissenschaft, Tech-Consulting. Maximaler Lern-Effekt beim System-Prompt-Tuning, weil die Vokabular-Welten, Ton-Lagen und Inhaltsstrukturen sich deutlich unterscheiden.
- **Alle drei sind realistische Wechselziele** für das Karriereprofil. Kein "Lufträume-Scrapen".
- **Alle drei haben substantielle Web-Auftritte** mit klar identifizierbaren Karriere-Bereichen, Press-Releases und About-Seiten — genug Material für den Bot, um sinnvoll zu antworten.
- **Geografischer Schwerpunkt DE/Karlsruhe-Heidelberg-Bruchsal** passt zur Wohnortlage.

**Considered:**

- **Agilent Technologies** (aktueller Arbeitgeber) → verworfen wegen optischer und praktischer Probleme: User-Agent des Scrapers würde in Agilent-Server-Logs auftauchen; Verwirrung falls ein Agilent-Recruiter den Link erhält; spannungsreiche Außenwirkung trotz vertraglich erlaubter Side-Projects. Falls Agilent als Test-Case interessant ist (Insider-Wissen ermöglicht Antwort-Verifikation), gehört das in eine Test-Corpus-Fixture (`08_test_strategy.md §3.1`), nicht in den Production-Pilot.

**Trade-offs:**

- **Drei DE-zentrische Firmen** → Recruiter aus internationalen Märkten profitieren weniger vom company-spezifischen Kontext. Akzeptiert, weil Hauptzielgruppe deutschsprachig.
- **Legal-Review pro Firma steht noch aus.** Jede der drei braucht den `legal_review`-Block in der Config (per `06_legal_compliance.md §5.2`), bevor der Scraper läuft. Erfolgt in Phase 1 vor TASK-107 (Scraper-Run).
- **Bei DKFZ besondere Aufmerksamkeit** auf Datenschutz: Forschungsbereiche können personenbezogene Patient/Studienteilnehmer-Daten erwähnen (auch wenn aggregiert/anonymisiert publiziert). Die `deny_paths` werden entsprechend konservativ gesetzt.

**Reversibility:**

- Sehr hoch. Hinzufügen weiterer oder Entfernen einer dieser drei ist eine reine Config-Datei-Operation plus Token-Generierung.
- Keine harten Plan-Abhängigkeiten von genau dieser Auswahl.

**Effects on plan:**

- **L4 in `06_legal_compliance.md §8`** → resolved.
- **Phase 1 / TASK-107 (Scraper-Run)** hat damit drei konkrete Target-Configs.
- **Drei Skelett-Configs** in `companies/` werden erstellt (sew-eurodrive.yml, dkfz.yml, exxeta.yml), mit `legal_review`-Platzhaltern und vorläufigen Seed-URLs.
- Die jeweils realen Legal-Reviews (TOS prüfen, robots.txt-Check, Pfad-Allowlist/Denylist) sind ein "Pre-flight-Check" vor dem ersten Scrape-Lauf.

**Notes:**

- Skelett-Configs sind *vorläufig*. Die `seed_urls` und `allow_paths` werden bei der Legal-Review pro Firma verfeinert. Die `legal_review`-Blocks sind im Skelett mit `TBD` markiert.
- Bot wird mit allen drei Firmen-Kontexten getestet werden, bevor die Tokens an Recruiter rausgehen.

---

---

## DEC-009 — Output-Filter forbidden patterns (Th2) (2026-05-13)

**Decision:** Output-Filter mit zwei Kategorien aktivieren, streng:

- **Kategorie B — Binding Statements / Acceptance.** Verhindert, dass der Bot Verpflichtungen eingeht (Marcel/ich akzeptiere/zusage/garantier/kündige etc., DE+EN).
- **Kategorie C — System-Prompt / Internal Leakage.** Verhindert, dass interne Systemstrukturen ausgeplaudert werden (Tool-Namen, Systemprompt-Marker, Config-Schlüsselwörter).

Bei Match: **sofortige Refusal**, keine Maskierung, kein "Reparatur"-Versuch. Antwort wird durch eine generische Default-Refusal ersetzt:

> "Da müsste ich passen — die Antwort wäre an dieser Stelle nicht angemessen. Frag Marcel gerne direkt unter <impressum-mail>."

(Anrede gemäß DEC-006: Du-Default, spiegelt nach Recruiter-Verhalten.)

Plus: Log-Eintrag in der DB (`filter_hit` mit Pattern-ID, Timestamp, Token-Hash — kein Inhalt).

**Verworfen / explizit nicht im Filter:**

- **Kategorie A — Salary / Compensation Commitments.** Vertrauen wird auf Systemprompt-Ground-Rules gesetzt (FR-024). Falls später regelmäßige Durchbrüche, Kategorie A nachrüstbar ohne Architektur-Schaden.
- **Kategorie D — Sensitive Personal Information.** Corpus soll sauber sein; Filter ist nicht der richtige Ort dafür. Bei Bedarf in Phase 5 als Allowlist nachrüsten.
- **Pattern "meine Anweisungen lauten" / "my instructions are"** aus Kategorie C entfernt — kann harmlos sein, wenn der Bot über seinen Zweck spricht ("meine Anweisungen lauten, ehrliche Antworten zu geben"). Striktere Marker (Systemprompt, GROUND RULES, Tool-Namen) reichen.

**Konkrete Pattern-Liste** (lebt in `config/forbidden_patterns.txt`, wird in Phase 2 / TASK-205 erstellt):

```regex
# Kategorie B — Binding Statements

# Deutsch
\b(?:ich|Marcel)\s+(?:akzeptier|nehme an|sage zu|stimme zu|verpflichte mich|garantier|zusichere)\w*
\b(?:ich|Marcel)\s+(?:k.ndige|trete bei|fange an|starte am)\b

# Englisch
\bI\s+(?:accept|agree to|commit to|will sign|guarantee|promise|hereby)\b
\b(?:Marcel|he|she)\s+(?:accepts|will sign|commits to|guarantees)\b

# Vertragsähnliche Phrasen
(?:hereby|hiermit)\s+(?:accept|akzeptiere|zusage|confirm|bestätige)

# Kategorie C — Internal Leakage

# Systemprompt-Marker
(?:System prompt|systemprompt|GROUND RULES|<system>|</system>)

# Persona-Marker
(?:You are an assistant|Du bist ein Assistent|Du bist Marcels)

# Tool-Namen
\b(?:search_candidate_(?:semantic|keyword)|search_company_(?:semantic|keyword))\b

# Interne Konfig-Hinweise
(?:API key|API-Schl.ssel|connection string|environment variable|\.env)
```

**Why:**

- B+C decken die "Reputations-Killer" und die "Architektur-Leak" ab — die zwei Restrisiken, bei denen ein einzelner Durchbruch peinlich wäre.
- A (Salary) und D (PII) werden bewusst dem Systemprompt überlassen, weil sie schwieriger zu treffen sind und ein zu strenger Filter False-Positives produzieren würde.
- Strenge Refusal (statt Maskieren) hält den Filter einfach und deterministisch. Maskieren erfordert Logik, wann genau und wie maskiert wird — komplexer, mehr Bugs.
- Edge-Case "meine Anweisungen lauten": durchlassen ist die richtige Wahl, weil das eine legitime Phrase sein kann, wenn der Bot seinen Zweck erklärt.

**Considered:**

- Alle vier Kategorien (A+B+C+D) → verworfen wegen False-Positive-Risiko bei großzügiger Salary-Erkennung und PII-Maskierung.
- Maskieren statt Refusal → verworfen wegen Komplexität; bei diesem Filter-Umfang nicht nötig.
- Allowlist für Kontaktdaten → auf Phase 5 verschoben, wenn Impressum-Werte final stehen.

**Reversibility:** Sehr hoch. Filter ist eine Textdatei mit Regex pro Zeile. Hinzufügen/Entfernen ist ein Edit + Re-Deploy.

**Effects on plan:**

- **Phase 2 / TASK-205 (System prompt + retrieval template als config)** — der Filter-Code lädt `config/forbidden_patterns.txt` und prüft jede LLM-Antwort vor Ausgabe.
- **Phase 2 / TASK-402 (Output filter)** — Implementierung dieses Filters. Eine neue Akzeptanzkriterium-Zeile: "Filter lädt Patterns aus Config, Test mit gestelltem Match liefert Refusal, Log-Eintrag erscheint."
- **Th2 in `04_threat_model.md` §7** → resolved.
- **Phase 5 / TASK-506** — Allowlist für Kontaktdaten kann hier nachgepflegt werden (Kategorie D in abgespeckter Form).

**Notes:**

- Pattern-Liste ist absichtlich konservativ. Lieber zu wenig matchen als zu viel; jeder Match ist eine Vollblockade.
- Falls in Phase 1/2 Eval-Daten zeigen, dass Salary-Themen durchrutschen (Recruiter fragt nach Gehalt, Bot nennt Zahl), Kategorie A nachrüsten.
- Pattern-Datei wird unter Git versioniert; jede Änderung geht durch PR-Review.