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