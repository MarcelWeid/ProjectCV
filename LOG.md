# Work Log — ProjectCV

Tägliches Arbeits-Journal. Format pro Eintrag:

- **Worked on:** welche TASK-IDs aus dem WBS
- **Done:** konkret erreicht
- **Stuck on:** offene Probleme
- **Next session:** wo morgen weitergeht
- **Decisions made:** Verweise auf decisions.md
- **Notes:** alles andere — Lernen, Beobachtungen, Hinweise an mein zukünftiges Ich

Jeder Eintrag wird beim Committen mit der zugehörigen Arbeit zusammen festgehalten.

---

## 2026-05-12 (Di, 120 min) — erste Session

**Worked on:**
- Plansession-Nachgang: TASK-001 (Naming-Entscheidung) abgeschlossen
- TASK-501 (VM-Provisionierung) vorgezogen aus Phase 5
- Härtungsanteil von TASK-502 (im Plan eigentlich Ansible-Bootstrapping, hier manuell)

**Done:**
- Public URL festgelegt: `marcel.weidemann.family/cv`. Kein separater Markenname. (DEC-001)
- Hetzner CX22, Ubuntu 24.04 LTS, Hostname `ragtime` provisioniert.
  - Erste Provisionierung mit Passwort-Auth war ungehärtet; verworfen und neu angelegt mit SSH-Key bei der Erstellung.
- Hetzner-Firewall `projectcv-vm`: SSH/22 nur von Heim-IP, sonst dicht.
- Non-root User `marcel` mit `sudo`-Rechten; SSH-Key übernommen.
- `/etc/ssh/sshd_config`: PasswordAuthentication no, PermitRootLogin no.
  - Override-Datei `/etc/ssh/sshd_config.d/50-cloud-init.conf` geprüft.
- fail2ban aktiv (`systemctl is-active` → active).
- unattended-upgrades enabled (`systemctl is-enabled` → enabled).
- UFW aktiv: deny incoming default, 22/80/443 TCP erlaubt, IPv4 + IPv6.
- Snapshot `hardened-base-2026-05-12` in Hetzner angelegt — Rollback-Punkt.
- Diese LOG.md und decisions.md angelegt.

**Stuck on:** —

**Next session:**
- TASK-006 (Azure-OpenAI-Setup): Verifikationen 1–4 durchgehen — Service-Zugang, Model-Deployment GPT-4o + text-embedding-3-large, TPM-Quota, Deployment-Type Standard (regional, nicht global).
- VS-Subscription-Guthaben prüfen, Spending Limit verifizieren.
- Bei grünem Durchlauf: DEC-003 schreiben ("Azure-OpenAI-Setup vorgezogen").

**Decisions made:**
- DEC-001 (Public URL + kein Markenname)
- DEC-002 (VM vor DB provisioniert)

**Notes:**
- Lernpunkt: SSH-Key gleich bei der Hetzner-Serverstellung hinterlegen, nicht über Passwort-Auth-Server nachträglich. Hat eine Iteration gekostet.
- Hostname `ragtime` darf bleiben — keiner sieht es außer mir.
- VM steht ab jetzt idle bereit, ~€4.50/Monat × ~3 Monate = ~€13 für die vorgezogene Provisionierung. Phase 5 wird dafür schlanker.
- TASK-001 (Naming) wurde während der Plan-Diskussion gelöst, nicht als eigene WBS-Task — der ursprüngliche Plan hatte "Produktname suchen" überschätzt; die Antwort war "kein Produktname nötig, URL beschreibt es."

**Risk watch:**
- R8 (Candidate Corpus) — heute keine Zeile geschrieben. Ab morgen: eine Projektbeschreibung pro Abend, parallel zur Tech-Arbeit. Sonst blockt es Phase 1.
- R6 (Scope Creep / Never-Ships) — heute eher gegenteilig: vorgezogen statt verzögert. Solange das kein neues *zusätzliches* Scope schafft, ist es harmlos.
---

## 2026-05-13 (Mi, 1. Block ~45 min) — TASK-006: Azure-OpenAI-Setup

**Worked on:** TASK-006 (Azure-OpenAI-Quota) vorgezogen.

**Done:**
- Azure-OpenAI-Resource unter Firmen-VS-Subscription, eigene Resource Group `marcel-private-projectcv-rg` mit Tags (`owner`, `project`, `purpose`).
- Region: West Europe.
- Deployment `gpt-4o` (Version 2024-11-20), Deployment-Type "Data Zone Standard (EUR)", 300K TPM.
- Deployment `text-embedding-3-large` (Version 1), Deployment-Type "Standard", 350K TPM.
- Erste Resource versehentlich ohne Resource-Group-/Tag-Konvention angelegt → gelöscht, neu sauber angelegt.

**Stuck on:** —

**Next session:** Cluster A (Repo, Pre-commit, Repo-Struktur).

**Decisions made:** DEC-003.

**Notes:**
- "Standard" ohne Data-Zone-Zusatz wird für gpt-4o aktuell nicht angeboten. EU-Data-Zone ist GDPR-konform.
- Modell-Retirement gpt-4o: 1. Oktober 2026 → R9 vorgemerkt.
- Subscription-Frage gut diskutiert: Vertrag erlaubt karrierebezogene Side-Projects, Hetzner ist privat, Kontinuität nach Jobwechsel nicht kritisch → Firmen-VS akzeptabel.
- TASK-006 plangemäß 1-10 Werktage Wartezeit → in 45min komplett erledigt.

**Risk watch:**
- R8 (Candidate Corpus) — null Zeilen.
- R9 (NEU: gpt-4o-Retirement Okt 2026) — Migrations-Session Sommer 2026.

---

## 2026-05-13 (Mi, 2. Block ~75 min) — Cluster A: Repo aufsetzen

**Worked on:** TASK-003 (Repo + README + LICENSE), TASK-008 (Pre-commit-Hooks, Secret-Scanner), TASK-009 (Verzeichnisstruktur).

**Done:**
- GitHub-Repo `github.com/MarcelWeid/ProjectCV` public, MIT-lizenziert.
- Verzeichnisstruktur angelegt: `src/`, `docs/`, `config/`, `companies/`, `tests/`, `infra/`, jeweils mit README zur Orientierung.
- Root-Dotfiles: `.gitignore`, `.editorconfig`, `.gitattributes`, `.pre-commit-config.yaml`, `.env.example`.
- GitHub Actions Workflow für Gitleaks-Secret-Scan auf Push und PR.
- Alle 8 Planungsdokumente plus `decisions.md` und `LOG.md` in `docs/`.

**Stuck on:** —

**Next session:**
- TASK-005 (Hetzner Managed Postgres mit pgvector) — würde Phase-1-Vorbereitung abschließen.
- Oder TASK-007 (lokale Dev-Env) — wenn Lust auf Tooling.
- Oder eine R8-Session (Candidate Corpus anfangen).

**Decisions made:** —

**Notes:**
- Beim ersten Commit fünf Dot-Files versehentlich in `docs/` mit-kopiert. Beim zweiten Commit (`git status` aufmerksam gelesen) gefangen, sauber gefixt. Plus `LICENSE.txt` → `LICENSE` für GitHub-Lizenzerkennung. Plus Zeilenenden auf LF normalisiert.
- Lernpunkt: `git status` ist die Pflicht-Lektüre vor `git commit`. `git add .` ohne Review ist eine Falle.
- Repo ist public-from-day-1, was die ehrliche Showcase-Position ist. Erster Recruiter-Link ist noch Wochen weg; die unvermeidlichen kleinen Holprigkeiten im frühen Verlauf sind okay und sogar authentisch.

**Risk watch:**
- R8 (Candidate Corpus) — weiterhin null Zeilen geschrieben. Stand 3. Tag in Folge. Wird langsam ernst.
- R6 (Scope Creep) — heute neutral; sauber im WBS-Scope geblieben.

---

## 2026-05-13 (Mi, 3. Block ~20 min) — TASK-004: AVV mit Hetzner

**Worked on:** TASK-004 (Hetzner-AVV signieren).

**Done:**
- AVV mit Hetzner Online GmbH online abgeschlossen via Cloud Console.
- Art der Daten: nur "Protokolldaten" — passt zur Architektur (keine persistenten Inhaltsdaten, nur gehashte IPs und technische Logs).
- Kreis der Betroffenen: nur "Kunden und Interessenten des Auftraggebers" — die Recruiter, die die Website besuchen.
- PDF-Bestätigung lokal abgelegt (außerhalb des Repos).

**Stuck on:** —

**Next session:** TASK-005 (Postgres mit pgvector).

**Decisions made:** —

**Notes:**
- AVV-Formular hatte nur Checkboxen, kein Freitext. Datenkategorien-Beschreibung gehört stattdessen später in die Datenschutzerklärung auf der Website.
- Compliance-Status nach 06_legal_compliance.md §1.2: Hetzner-AVV ✓, Microsoft-DPA noch offen, Alerts-Provider TBD (Th3).
---

## 2026-05-13 (Mi, 4. Block ~90 min) — TASK-005a/b: Postgres + pgvector lokal

**Worked on:** TASK-005 (local-dev-Teil), DB-Architekturentscheidung, WSL-Migration als Nebenarbeit.

**Done:**

- **DB-Setup lokal:** Postgres 17 + pgvector 0.8.2 via Docker Compose auf WSL (Docker Engine, kein Desktop).
  - `infra/docker-compose.yml` mit `pgvector/pgvector:pg17`-Image, Healthcheck, Named Volume `projectcv-db-data`, gebunden an `127.0.0.1:5432`.
  - `infra/db-init/001-extension.sql` enabled die Vector-Extension idempotent.
  - `.env` lokal mit generiertem Passwort (im Passwortmanager abgelegt, nicht im Repo).
  - Verifiziert: Extension installiert, Vector-Datentyp benutzbar, L2- und Cosine-Distance-Operatoren funktionieren.

- **DB-Architektur revidiert (DEC-004):** Hetzner Managed Postgres bietet kein nutzbares pgvector → self-hosted via Docker auf bestehender VM (Phase 5). ADR-001 entsprechend überarbeitet. Vier Alternativen (Qdrant, Weaviate, SQLite+vec, andere managed Optionen) verworfen mit dokumentierter Begründung.

- **WSL-Migration:** Repo nach `~/repos/ProjectCV` umgezogen (war auf Windows-Pfad `/mnt/c/...`). Sauberes Repo-Layout, schnellere Docker-Bind-Mounts.

- **SSH-Auth zu GitHub eingerichtet:** ed25519-Key in WSL erzeugt (ohne Passphrase — bewusste Wahl), auf GitHub hinterlegt (Title: "projectcv wsl"), Remote von HTTPS auf SSH umgestellt, getestet.

- **Git-Identität pro Repo:** `user.email` lokal auf GitHub-No-Reply-Adresse umgestellt (`159274752+MarcelWeid@users.noreply.github.com`), globale Firmen-Mail unverändert → künftige Commits in diesem Repo nutzen No-Reply, Arbeitsrepos bleiben unberührt.

- **Lost-and-found:** DEC-003 und R9 waren bei der WSL-Migration verloren gegangen (nur in der alten Output-Sandbox, nicht committet). Beide nachgereicht und zusammen mit DEC-004/R10/ADR-001 in einem Commit auf GitHub.

**Stuck on:** —

**Next session:**
- TASK-007 (lokale Dev-Env: .NET 9 SDK, Node.js LTS, ggf. weiteres).
- Oder TASK-010 (kleine Decisions: D5 UI-Ton, Th2 Output-Filter, L1 Impressum-Adresse, L4 Pilot-Companies).
- Oder R8 (erste Projektbeschreibung für Candidate Corpus — wird ernst, vier Tage in Folge offen).

**Decisions made:**
- DEC-004 (Postgres self-hosted via Docker).
- Implizit: WSL als primäres Dev-Environment.

**Notes:**

- Operativer Workflow ist jetzt: Doku-Updates **sofort committen**, nicht in der Sandbox liegen lassen. Verlust von DEC-003/R9 war direkt auf diese frühere Praxis zurückführbar.
- DB läuft komplett lokal via WSL Docker, kein Cloud-Dienst beteiligt. NFR-040 (local dev environment) ist erfüllt.
- VM-Setup für die DB kommt erst in Phase 5 — gleiche Compose-Datei, andere `.env`.
- Lernpunkt: WSL2-Docker funktioniert für unseren Use-Case prima ohne Docker Desktop. Volume-Performance auf `/mnt/c/...` wäre langsamer gewesen → Repo-Move nach `~/repos/` war richtig.

**Risk watch:**
- R8 (Candidate Corpus) — weiterhin null Zeilen. Vierter Tag. Wird *jetzt* ernst.
- R10 (self-hosted DB Operational Burden) — neu aufgenommen, Backup-Strategie ist Phase-5-Deliverable.
- R6 (Scope Creep) — heute neutral, sauber im WBS-Scope.

---

## 2026-05-13 (Mi, 5. Block ~30 min) — TASK-007: Dev-Env vervollständigt

**Worked on:** TASK-007 (lokale Dev-Env), Versionsabgleich .NET.

**Done:**
- .NET 10 SDK (10.0.104) war bereits installiert — Versionswahl bewusst getroffen (LTS, DEC-005).
- `dotnet ef` Global Tool 10.0.8 installiert.
- `psql` Client 16.13 installiert (Ubuntu apt).
- End-to-End-Test: kleines C#-Programm mit Npgsql verbindet sich zur lokalen Docker-Postgres, liest Postgres-Version (17.9) und pgvector-Version (0.8.2). Test-Programm anschließend gelöscht.

**Stuck on:** —

**Next session:**
- TASK-010 (kleine Decisions: D5 UI-Ton, Th2 Output-Filter, L1 Impressum, L4 Pilot-Companies).
- Oder R8 (Candidate Corpus — fünfter Tag offen, wird langsam wirklich ernst).
- Nach TASK-010 ist Phase 0 zu 100% durch und Phase 1 (RAG-Core) kann starten.

**Decisions made:** DEC-005 (.NET 10 statt 9).

**Notes:**
- psql 16 redet problemlos mit Server 17 — Warnung über Major-Version-Diff ist erwartet, betrifft nur exotische Features.
- Lokales Passwort enthält URL-unfreundliches Zeichen — URL-Connection-String funktioniert nicht, getrennte Parameter (Host/User/Password als Env-Vars) funktionieren. Für Production-DB (Phase 5) Passwort URL-safe generieren (z.B. `openssl rand -hex 32`).

**Risk watch:**
- R8 (Candidate Corpus) — Tag 5 offen. Empfehlung an mich selbst: morgen unbedingt anfangen, auch wenn nur eine Projektbeschreibung.
- Sonst keine neuen Risiken.

---

## 2026-05-13 (Mi, Blöcke 6-9 ~85 min) — TASK-010: vier kleine Decisions

**Worked on:** TASK-010 in vier Teilen — alle vier offenen kleinen Decisions geschlossen.

**Done:**

**D5 — Bot-Tonalität, Sprache, Anrede** (DEC-006)
- Ton: "leicht persönlich, sachlich" (Variante 2 aus dem Vergleich) — Bot spricht über Marcel mit der Wärme eines Kollegen, der ihn gut kennt.
- Hauptsprache: Deutsch, Englisch nach Recruiter-Eingabe automatisch (FR-011).
- Anrede: "Du" als Default, wechselt zu "Sie" sobald der Recruiter erstmals siezt — danach einseitig (kein Rückwärts-Wechsel).

**L1 — Impressum-Adresse** (DEC-007)
- Wohnanschrift, Mobilnummer, E-Mail im Impressum.
- Alternativen (c/o, Service, nur-E-Mail) bewusst verworfen.
- Telefonsache nach EuGH-Hinweis (C-298/07) auf Mobilnummer korrigiert.
- Privacy-Trade-off bewusst akzeptiert: WHOIS-Privacy beim Registrar wird durch öffentliches Impressum effektiv ausgehebelt.

**L4 — Drei Pilot-Companies** (DEC-008)
- SEW-Eurodrive (Industrie-Mittelstand), DKFZ (öffentliche Forschungseinrichtung), Exxeta (Tech-Consulting).
- Agilent (aktueller Arbeitgeber) als ursprünglicher dritter Kandidat verworfen — Test-Wert überwiegt nicht die optischen Risiken.
- Drei Profile maximal unterschiedlich → Vokabular- und Ton-Lagen-Spektrum für späteres Systemprompt-Tuning.
- Skelett-Configs unter `companies/` mit TBD-Platzhaltern für legal_review.

**Th2 — Output-Filter forbidden patterns** (DEC-009)
- Schlanker Filter: Kategorien B (Binding Statements) + C (Internal Leakage), streng.
- Kategorien A (Salary) und D (PII) bewusst weggelassen — Vertrauen auf Systemprompt-Ground-Rules.
- Match-Verhalten: sofortige Refusal, kein Maskieren, plus Log-Eintrag.
- Edge-Case "meine Anweisungen lauten" bewusst durchgelassen.

**Stuck on:** —

**Next session:** **PHASE 0 IST KOMPLETT.** Nächster Schritt: Phase 1 (RAG Core, offline). Beginnt mit TASK-101 (Solution-Skeleton).

**Decisions made:** DEC-006, DEC-007, DEC-008, DEC-009.

**Notes:**
- Geschmacks-Entscheidungen mit konkreten Beispiel-Antworten (D5) greifbar zu machen, hat gut funktioniert.
- L1: anwaltliche Beratung nicht in Anspruch genommen; bei Bauchschmerzen später nachholbar.
- L4: Bei DKFZ besondere Aufmerksamkeit auf PII (Patient/Studien-Daten) — strenge deny_paths in der Skelett-Config.
- Th2: bewusst defensiv. Lieber zu wenig matchen als False-Positives. Nachrüstbar wenn Eval-Daten Lücken zeigen.

**Risk watch:**
- R8 (Candidate Corpus) — sechs Tage offen. Jetzt der konkrete Blocker für Phase 1, nicht mehr nur eine Empfehlung.

**Phase-0-Status:** ✓ 10 von 10 Tasks erledigt.