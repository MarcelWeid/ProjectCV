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