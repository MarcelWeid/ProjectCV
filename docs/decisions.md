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
