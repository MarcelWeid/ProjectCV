# Work Breakdown & Schedule — ProjectCV

**Document status:** Draft v0.1
**Owner:** [You]
**Last updated:** 2026-05-12
**Companion documents:** 01_project_charter.md through 06_legal_compliance.md

---

## 0. About this document

This document turns the plan into execution. The charter set a six-phase envelope of ~8–9 calendar weeks at part-time effort. This document breaks each phase into concrete tasks with effort estimates, dependencies, and acceptance criteria, identifies the critical path, refreshes the risk picture from what's been learned during design, and pins a hard MVP cutoff so v1.0 actually ships.

### 0.1 Estimation units and conventions

- **Effort is in focused hours**, not calendar time. A focused hour is uninterrupted work with the right context loaded. Tax this when planning your calendar: a typical evening yields 1.5–2 focused hours; a typical Saturday yields 4–6.
- **A part-time week** in this plan = **10 focused hours**. Adjust the calendar projection if your reality differs.
- **Task IDs** are `TASK-NNN`, stable, never reused. Phase prefix in description for readability.
- **Effort buckets:** S = ≤ 2h, M = 2–4h, L = 4–8h, XL = 8–16h (split if you can).
- **Buffer factor:** I add 25% to all phase totals before converting to calendar weeks. Software estimation is famously optimistic; the buffer absorbs reality.

### 0.2 What this document is not

- Not a Gantt chart. Calendar dates fluctuate; relative ordering and dependencies don't.
- Not a backlog. The backlog of v1.1+ items lives in §6.
- Not a contract with yourself. Estimates are best-effort; the discipline is **adjusting the cutoff list (§7) rather than the calendar.**

---

## 1. Phase summary

The six phases from the charter, with the design-phase view now updated:

| Phase | Goal | Effort (focused hours) | + 25% buffer | Calendar (at 10h/wk) |
|---|---|---|---|---|
| 0. Setup & decisions | Accounts, domain, repo, tooling | 12h | 15h | 1.5 weeks |
| 1. RAG core (offline) | Scraper + candidate ingest + retrieval working via CLI | 28h | 35h | 3.5 weeks |
| 2. Backend API & agent | .NET API with full agent loop and guards | 32h | 40h | 4 weeks |
| 3. Frontend | Chat UI, streaming, citations, disclosure, footer | 16h | 20h | 2 weeks |
| 4. Security hardening | Threat model verification, red-team suite, kill switch | 14h | 17.5h | 1.75 weeks |
| 5. Ops & go-live | Deploy, observability, Impressum & privacy, public repo polish | 16h | 20h | 2 weeks |
| **Total to v1.0** | | **118h** | **148h** | **~14.7 weeks** |
| 6. Polish & sharing | Pre-scrape remaining companies, iterate, share | ongoing | ongoing | ongoing |

**Reality check vs the charter's 8–9 week estimate.** The charter's number was directional and didn't have the design depth we now have. The honest revised estimate is ~15 weeks of part-time work at 10h/week. **This is fine.** It's better to know now than to discover it at week 8.

Two ways to shorten the schedule if 15 weeks feels long:

1. **Increase weekly hours.** 15 weeks at 10h/week = ~7.5 weeks at 20h/week. If you can carve out a couple of Saturdays at 6 focused hours, the calendar compresses fast.
2. **Cut MVP further.** See §7 — there are already significant items below the cutoff. Some of them (e.g. the second language, the citations panel polish, multiple companies pre-scraped) could move further down if needed.

I don't recommend (3) reducing estimates without changing scope; that's how projects die.

### 1.1 Critical path

```
Phase 0 ─▶ Phase 1 ─▶ Phase 2 ─▶ Phase 4 ─▶ Phase 5
                    └─▶ Phase 3 ─▶ Phase 4
```

Phase 3 (frontend) is parallelisable with the back half of Phase 2 once the API contract is stable. In practice, for a one-person project, you'll do them sequentially — but if you find yourself blocked on something in Phase 2 (e.g. waiting for an Azure quota approval), pivot to Phase 3 work rather than idling.

Phase 4 (security) cannot start before Phase 2's agent loop and Phase 3's UI rendering both exist, because some of the red-team prompts need the full chain to test.

Phase 5 (ops) needs Phase 4 green to be deployable.

**Single longest serial chain:** Phase 0 → 1 → 2 → 4 → 5 = 102h focused (127.5h buffered) = ~13 weeks. Phase 3 fits inside this without extending it.

---

## 2. Phase 0 — Setup & decisions (12h)

The boring but essential phase. Skipping any of these guarantees friction later.

| ID | Task | Dep | Effort | Acceptance |
|---|---|---|---|---|
| TASK-001 | Product naming decision (D1 from charter) — pick a name (and a fallback) | — | S (1h) | Name chosen, domain availability checked |
| TASK-002 | Domain registration (D2, A2) | TASK-001 | S (1h) | DNS-ready domain in candidate's hands |
| TASK-003 | GitHub repo created, README skeleton, LICENSE (MIT or Apache-2.0), `.gitignore`, branch protection on `main` | TASK-001 | S (1h) | Repo exists, public, with starting README |
| TASK-004 | Hetzner Cloud account: billing alerts set, project created, AVV signed (compliance §1.2) | — | S (1h) | Project visible in console, AVV on file |
| TASK-005 | Hetzner Managed Postgres provisioned with `pgvector` extension enabled, allowlisted to candidate's IP | TASK-004 | S (1.5h) | `psql` connects from local machine, `CREATE EXTENSION vector` succeeds |
| TASK-006 | Azure account: subscription, OpenAI resource provisioned in EU region (West Europe or Sweden Central), GPT-4o and `text-embedding-3-large` deployments created, training-on-data disabled | — | M (2h) | API call to chat completion endpoint returns from local curl |
| TASK-007 | Local dev environment: .NET 9 SDK, Docker Desktop, Node.js LTS, PostgreSQL client, `direnv` for `.env` loading | — | M (2h) | `dotnet --version`, `docker run hello-world`, `psql` all work |
| TASK-008 | `.editorconfig`, `.gitattributes`, pre-commit hooks for secret scanning (`gitleaks` or `trufflehog`) | TASK-003 | S (1h) | Pre-commit blocks a deliberately-staged fake key |
| TASK-009 | Repo structure: `src/`, `tests/`, `docs/` (move these planning docs in), `companies/`, `config/`, `infra/` | TASK-003 | S (1h) | Folders exist with placeholder READMEs explaining each |
| TASK-010 | Decision capture: D4 (model tier — already resolved GPT-4o end-to-end), D5 (UI tone), Th2 (output filter forbidden-pattern list), L1 (Impressum address), L4 (first 3 pilot companies) | — | M (2h) | All five decisions recorded in `docs/decisions.md` |

**Phase 0 acceptance:** the candidate can run `git clone`, `direnv allow`, and have a working environment that talks to Hetzner DB and Azure OpenAI. No code yet — just plumbing.

**Risk to watch:** Azure OpenAI quota approval can take 1–10 business days for the larger model tiers. If quota is pending, start TASK-006 immediately and proceed with other tasks while waiting.

---

## 3. Phase 1 — RAG core, offline (28h)

The phase where the project becomes real. By the end, you have a CLI that ingests content and retrieves the right chunks for a question — without any LLM in the loop yet.

| ID | Task | Dep | Effort | Acceptance |
|---|---|---|---|---|
| TASK-101 | Solution structure: `ProjectCv.Domain`, `ProjectCv.Infrastructure`, `ProjectCv.Api`, `ProjectCv.Scraper`, `ProjectCv.IngestCandidate`, `ProjectCv.TokenAdmin`, `ProjectCv.Eval`, `ProjectCv.Tests` | TASK-009 | M (2h) | `dotnet build` succeeds across the solution |
| TASK-102 | Database schema and EF Core migrations: `chunks`, `companies`, `tokens`, `daily_spend`, `rate_buckets`, `kill_switch` (per Th1 resolution); indexes per RAG §4.2 | TASK-005, TASK-101 | M (3h) | `dotnet ef database update` runs cleanly; pgvector and FTS indexes present |
| TASK-103 | Embedding service abstraction — interface + Azure OpenAI implementation + simple in-memory fake for tests | TASK-006 | M (2h) | Unit test: real implementation returns a 3072-dim vector for "hello world" |
| TASK-104 | Chunker — recursive splitter with priority separators, target/min/max/overlap per RAG §3 | TASK-101 | L (5h) | Unit tests over: a long markdown file, a German page, a degenerate single-word input, an empty input |
| TASK-105 | HTML cleaner using AngleSharp — main-content extraction, hidden-text stripping, whitespace normalisation | TASK-101 | L (5h) | Tests against three fixture HTML files (clean site, noisy site, hidden-text injection); audit log reports stripped hidden-element count |
| TASK-106 | Candidate ingest CLI — recursive directory read, off-limits flag handling, chunk + embed + upsert with idempotency | TASK-102, TASK-103, TASK-104 | M (3h) | CLI run twice on the same folder produces 0 new rows the second time; off-limits file is skipped |
| TASK-107 | Scraper CLI — config-driven crawl, `robots.txt` respect, user-agent identification, depth/page bounds, deny-paths, audit log | TASK-102, TASK-103, TASK-104, TASK-105 | XL (8h) | Dry-run against a real company config produces audit log with expected URLs; second run produces 0 new rows (idempotency); `robots.txt` disallow honoured in a unit test |
| TASK-108 | Retrieval queries — semantic and keyword as repository methods, scoped by `corpus_kind` and `company_id` | TASK-102 | M (3h) | Integration test against seeded DB: cross-corpus leak test fails to retrieve other corpus chunks |
| TASK-109 | CLI debug command: `query "..."` — takes a question, prints which chunks each tool would return. No LLM yet. | TASK-108 | M (2h) | Manual verification: useful answers for 5 sample questions over a small seeded corpus |
| TASK-110 | Seed eval set — write first 15 entries of `golden.yml` covering candidate-side questions only | TASK-109 | M (3h) | File exists, format matches RAG §6.2 |

**Phase 1 acceptance:** candidate runs `ingest-candidate ~/cv/` and `scrape companies/pilot-1.yml`, then `query "Tell me about leadership style"` and gets sensible retrieved chunks. The RAG plumbing works end-to-end *minus* the LLM.

**Risk to watch:** the chunker has the most edge cases of any single component. Budget for TASK-104 may stretch; that's acceptable, it's the foundation.

---

## 4. Phase 2 — Backend API & agent (32h)

The phase where it becomes a chatbot.

| ID | Task | Dep | Effort | Acceptance |
|---|---|---|---|---|
| TASK-201 | ASP.NET Core Web API skeleton, health endpoint, structured logging (Serilog JSON sink) | TASK-101 | M (2h) | `/health` returns 200; log lines are JSON with the SR-030 fields |
| TASK-202 | Tool router — interface + four tool implementations bound to retrieval methods; allow-list validator | TASK-108 | M (3h) | Unit test: calling `search_candidate_semantic` returns chunks; calling `delete_database` returns "tool not found" |
| TASK-203 | Azure OpenAI chat client with function calling — wrapper around the SDK, streaming output | TASK-006 | M (3h) | Integration test: send a question, get a tool call back; respond with tool result; get a final answer |
| TASK-204 | Agent loop — iteration cap, stop conditions, tool execution per RAG §3.1 | TASK-202, TASK-203 | L (5h) | Tests: happy path, iteration-cap hit, model returns non-tool-call → finishes |
| TASK-205 | System prompt and retrieval template as `config/*.md` files; assembly logic | TASK-204 | M (2h) | Changing the prompt file changes behaviour without recompilation |
| TASK-206 | Token guard — HMAC validation, company-context attachment, generic-mode fallback | TASK-102 | M (2h) | Tests: valid token → company context; invalid/expired/missing → generic mode |
| TASK-207 | Rate limiter — per-token and per-IP sliding-window in Postgres | TASK-102 | M (3h) | Test: 31 requests in an hour to the same token returns 429 on the 31st |
| TASK-208 | Budget guard — daily / monthly caps, pre-flight check and post-flight accounting | TASK-102, TASK-204 | M (3h) | Test: setting daily cap to €0.01 trips degraded mode on the second request |
| TASK-209 | Input sanitiser — length cap, control-character strip, Unicode NFKC | TASK-201 | S (1.5h) | Tests against the standard injection-via-encoding payloads (zero-width, RTL override, etc.) |
| TASK-210 | Response streamer — SSE with metadata trailer (tool calls, citations, cost) | TASK-204 | M (3h) | Manual test: `curl --no-buffer` shows tokens streaming, ends with metadata event |
| TASK-211 | `/chat` endpoint composition — wire all guards, agent, streamer together | TASK-206..210 | M (2h) | End-to-end test: real Azure call with a test corpus returns a grounded answer |
| TASK-212 | Kill switch — `/admin/kill` and `/admin/revive` endpoints, bearer-token guarded, persistent flag in DB | TASK-201, TASK-102 | M (2h) | Hitting kill returns the chat to degraded mode within 5 seconds; revive restores |
| TASK-213 | Configuration plumbing — `appsettings.json` for non-secrets, env vars for secrets, validation on startup | TASK-201 | S (1.5h) | App fails fast on missing Azure key with a clear error |
| TASK-214 | Cost accounting — token-to-EUR conversion using configurable price table; per-turn cost recorded | TASK-208 | S (1.5h) | Test: a known-token-count interaction yields the expected cost |

**Phase 2 acceptance:** `curl -X POST localhost:5000/chat -d '{"message":"Tell me about your leadership style"}' ?c=token` returns a grounded streaming response that cites sources, respects rate limits, and increments the daily-spend counter.

**Risk to watch:** TASK-204 (agent loop) is the most subtle piece in the project. The first version will work; the *tuned* version takes longer. Time-box at 8h; if not converging, ship a working-but-blunt version and tune in Phase 6.

---

## 5. Phase 3 — Frontend (16h)

| ID | Task | Dep | Effort | Acceptance |
|---|---|---|---|---|
| TASK-301 | Vite + TypeScript scaffold; build pipeline; output bundle target | TASK-009 | S (1.5h) | `npm run build` produces a single index.html + assets |
| TASK-302 | Page layout: hero/intro, chat panel, footer with Impressum / Datenschutz / GitHub links | TASK-301 | M (3h) | Looks reasonable on desktop and mobile; LCP < 2.5s on 4G (NFR-003) |
| TASK-303 | AI disclosure banner — visible before first message, dismissible to chat surface | TASK-302 | S (1.5h) | Banner shown on first load; persistent within session even after dismissal of a header element |
| TASK-304 | Chat input + history rendering — markdown-rendered output (allow-list: bold, italic, lists, code, links with `rel="noopener noreferrer"`) | TASK-302 | M (3h) | Pasting `<script>alert(1)</script>` into bot output renders as text |
| TASK-305 | SSE consumer — token-by-token rendering, error handling, reconnection | TASK-301 | M (3h) | Manual test against backend: tokens appear progressively, network blip handled gracefully |
| TASK-306 | Citations panel — collapsed by default, titles shown, URL on hover (per A4 resolution) | TASK-305 | M (2h) | Each cited source has File + Section visible; URL appears only on hover/click |
| TASK-307 | "Clear chat" / start-over control | TASK-305 | S (1h) | Drops in-memory history; new request starts fresh |
| TASK-308 | Impressum and Datenschutz pages — static HTML, content from §3.2 and §4.2 of legal doc once finalised | TASK-302 | S (1h) | Both reachable, linked from footer, content from candidate's finalised text |

**Phase 3 acceptance:** the recruiter can open `https://[domain]/?c=token` and have a useful, polished conversation that looks credible.

**Risk to watch:** None high. The frontend is the most contained phase. The risk is over-polishing; ship "good enough," polish in Phase 6.

---

## 6. Phase 4 — Security hardening (14h)

| ID | Task | Dep | Effort | Acceptance |
|---|---|---|---|---|
| TASK-401 | Caddy configuration — TLS auto, HSTS, CSP, X-Frame-Options, Referrer-Policy | TASK-201 | M (2h) | Mozilla Observatory score A or better |
| TASK-402 | Output filter — regex-based forbidden-pattern matching against final responses; refuse-to-send on hit | TASK-211 | M (3h) | Test: bot trying to output "I accept the offer" gets rewritten to refusal |
| TASK-403 | Retrieved-content fencing — verify `<retrieved>...</retrieved>` template and trailing reminder are emitted correctly (LLM01 defence) | TASK-205 | S (1h) | Test: a poisoned chunk in retrieval is fenced and the model ignores the embedded instruction |
| TASK-404 | Red-team test suite — 22 prompts from threat model §5, split into `cheap/` (mocked LLM, run on every PR) and `live/` (real Azure, run on `release-*` tag) per Th4 | TASK-211, TASK-402 | L (5h) | CI green; suite documented in `tests/redteam/README.md` |
| TASK-405 | Eval harness wiring — extend `golden.yml` to 40 entries, hook up evaluation runner, output JSON report | TASK-110, TASK-211 | M (3h) | `dotnet test --filter Category=Eval` produces a metrics report; first run is the baseline |
| TASK-406 | Dependency vulnerability scan in CI — `dotnet list package --vulnerable`, `npm audit`, Trivy on container images | TASK-201, TASK-301 | S (1h) | CI fails on a deliberately-introduced vulnerable package |

**Phase 4 acceptance:** red-team suite green, eval baseline established, output filter active, Caddy serving with strong headers. **This is the gate before phase 5.**

**Risk to watch:** TASK-404 will surface real issues. Some of those issues mean going back into Phase 2 to adjust the system prompt or the input sanitiser. Budget mentally for one round-trip; if there are two, that's a signal to slow down rather than push through.

---

## 7. Phase 5 — Ops & go-live (16h)

| ID | Task | Dep | Effort | Acceptance |
|---|---|---|---|---|
| TASK-501 | Hetzner VM provisioned via Terraform: CX22, Ubuntu LTS, firewall, SSH-key-only access | TASK-004 | M (3h) | `terraform apply` from a clean state produces a usable VM |
| TASK-502 | Ansible playbook — Docker, Caddy, fail2ban, log rotation, Uptime Kuma | TASK-501 | M (3h) | Re-running playbook is idempotent; VM ready for app deploy |
| TASK-503 | Docker Compose for production: api, caddy, uptime-kuma; pinned image digests | TASK-211, TASK-401 | M (2h) | `docker compose up -d` brings the stack up; api reachable via Caddy |
| TASK-504 | GitHub Actions CI/CD — build images, push to GHCR, SSH deploy, rollback support | TASK-503 | M (3h) | A push to `main` lands a new container on the VM end-to-end |
| TASK-505 | Uptime Kuma configured — health checks, budget-tripped check, email/Telegram alerts (per Th3 / OR-003) | TASK-502, TASK-208 | S (1.5h) | Deliberately failing the health check triggers an alert within 5 minutes |
| TASK-506 | Impressum and Datenschutz finalisation — candidate's chosen address (L1) and DSR mailbox (L2); legal review or generator output applied | TASK-308 | M (2h) per legal §6 checklist — but most of this is the candidate's procurement, not coding |
| TASK-507 | Public repo polish — README with screenshots, link to live site, architecture overview, "how it works" section linking to `docs/` | TASK-503 | S (1.5h) | A reviewing engineer can grok the project from the README in 5 minutes |

**Phase 5 acceptance:** the site is live, the repo is public and presentable, all compliance artefacts are in place, monitoring is firing alerts, the candidate has tested the kill switch in production. **Ready to share with the first recruiter.**

**Risk to watch:** TASK-506 has external dependencies (the candidate must actually procure or write the legal text). Schedule this in parallel with Phase 4 — there's no reason to wait until Phase 5 to obtain the generator subscription or commission the lawyer review.

---

## 8. The MVP cutoff — what does *not* ship in v1.0

R6 (scope creep) was the highest-risk item in the charter. The defence is to write the cutoff list explicitly, *before* feeling tempted to expand. These all live in the post-v1.0 backlog (§9 below). They are good ideas. They are not v1.0.

**Not in v1.0:**

- Cross-encoder reranker (RAG §8)
- Hypothetical Document Embeddings, multi-vector retrieval, query rewriting as a separate step (RAG §8)
- More than ~3 pilot companies pre-scraped. The remaining ~17 come in Phase 6, on demand as the candidate targets specific recruiters.
- PDF support in candidate ingest (RAG R1 — Markdown only at MVP)
- Anything dashboard-like for the candidate to "watch the bot in action." Logs in stdout + monthly retrospective are sufficient.
- Custom UI delight features (theming, terminal aesthetic, etc.). The site looks clean; that's enough.
- A "second candidate" or multi-tenant capability. Charter §5 already excluded this.
- Persistent recruiter sessions (account, login). Charter §5 already excluded this.
- Email/calendar integration ("book a call with the candidate"). Tempting; not v1.0.
- Internationalisation beyond German + English. Charter §5 already excluded; FR-011 doesn't expand.
- Self-service company token issuance (i.e. anyone-with-the-base-link can register a company). Tokens stay candidate-issued via CLI.

**The discipline:** if a feature is not on the WBS in §2–§7, it doesn't ship in v1.0. Move it to the backlog. Decide on v1.1+ after v1.0 has been in the wild for a month.

---

## 9. Backlog (v1.1+)

A holding area for things that will likely be worth doing later, in no particular priority order. Items here are reminders, not promises.

- Cross-encoder reranker (RAG §8)
- HyDE / query rewriting evaluation
- PDF support in candidate ingest
- Per-chunk summary at index time
- On-demand company ingestion ("I'm meeting Acme tomorrow, scrape them now")
- Conversation persistence with strict TTL (would require GDPR re-review)
- A small candidate-side dashboard with daily counters and last-N redacted log lines
- Telegram bot for kill switch / alerts (instead of just email)
- An "easter egg" mode — e.g. the bot interviewing the recruiter back; the visible "thinking" trace
- A second deployable instance for testing tuning changes against production traffic mirrors
- Automated freshness alerts when a company's `legal_review` block is older than 90 days
- WebAuthn-protected admin endpoints instead of bearer tokens

The eval harness (RAG §6) is the arbiter for anything that affects retrieval. The threat model (§5) is the arbiter for anything new that touches LLM behaviour.

---

## 10. Risk picture — refreshed

The charter listed six risks. The design phase has resolved some and surfaced others. Current state:

| # | Risk | Status |
|---|---|---|
| R1 | Bot hallucinates a claim about the candidate | **Materially mitigated.** Tool-use-mandatory system prompt, refusal-on-no-retrieval (FR-014), red-team R-08, citations (FR-015). Residual: human spot-check during phase 6 catches model drift. |
| R2 | Prompt injection causes off-brand output | **Materially mitigated.** Retrieved-content fencing, system prompt rules, output filter, red-team suite. Residual: novel attacks; the suite must grow over time. |
| R3 | Cost exhaustion attack | **Materially mitigated.** Per-token + per-IP rate limits, daily + monthly caps, kill switch, output token cap, iteration cap. Residual: distributed-IP attack in generic mode bounded by daily cap (AC-1 gap noted). |
| R4 | Scraped data violates ToS / copyright | **Materially mitigated.** Per-company `legal_review` checklist, conservative default on ambiguity (L6 resolution), reduced-scope option, structurally enforced via scraper refusal. |
| R5 | GDPR exposure from logging | **Materially mitigated.** No message content in default logs, 30-day retention, hashed identifiers, privacy notice explicit on what is and isn't kept. |
| R6 | Scope creep → never ships | **Active risk.** This is now the highest-likelihood remaining risk. Defence is §7 (MVP cutoff) and the discipline of updating §7 rather than reducing estimates when reality slips. |
| **NEW R7** | Estimation optimism — 15 weeks part-time may slip to 20+ if external dependencies (Azure quota, Impressum review, target company readiness) take longer than expected | **Monitored.** Buffer factor of 25% built into estimates; calendar visibility through phase acceptance gates. If at end of Phase 1 we're > 20% over budget, re-baseline the schedule rather than push. |
| **NEW R8** | The candidate's corpus itself is the deliverable for Phase 1 to be useful. If the candidate doesn't have ~30 pages of written content about themselves and their projects, the bot has nothing to ground on. | **To be addressed.** Add to Phase 0 prerequisites: candidate writes/curates corpus in parallel with TASK-001 through TASK-010. This is invisible work that gates Phase 1 acceptance. Budget separately: ~10h candidate-writing-time, not coding time. |

R8 is the new one worth attention. Without good corpus content, all the engineering above produces a bot that confidently refuses every question, because retrieval returns nothing. This is the *real* prerequisite for the system being any good, and it's the one task I cannot do anything about from this side.

---

## 11. Phase acceptance gates

Each phase ends with a concrete demonstration before the next begins. The discipline: do the demo, even alone, even if it feels theatrical. The demo forces honesty about what works.

| Phase | Demo |
|---|---|
| 0 | Show the working dev environment, repo structure, Hetzner DB connect, Azure OpenAI call from `curl` |
| 1 | Ingest the candidate corpus + one company; run `query` for 5 sample questions and show that retrieved chunks are relevant |
| 2 | `curl /chat` with a token, get a streaming grounded answer; trip the rate limit; trip the budget cap |
| 3 | Open the site in a browser, have a conversation, click citations, refresh and confirm history is gone |
| 4 | Run the full red-team suite; show the JSON eval report; demonstrate the kill switch in a test env |
| 5 | Open `https://[domain]/?c=token` from someone else's device; show Impressum, Datenschutz; confirm alerts fire on a simulated outage |

---

## 12. Open questions

| # | Question | Owner | Affects |
|---|---|---|---|
| W1 | Confirm 10h/week part-time assumption — if reality is 15h or 5h, the calendar projection in §1 needs adjustment | You | §1 |
| W2 | When in the schedule does the candidate write the corpus (R8)? Recommendation: in parallel with Phase 0 and continuing through Phase 1, so it's ready when ingest runs. | You | R8, Phase 1 |
| W3 | Are 3 pre-scraped companies enough for the v1.0 share? Or should the cutoff include having all ~20 ready? My recommendation: 3 is enough — share with recruiters at those 3 first, scrape new ones as outreach expands. | You | §8 |
| W4 | ~~Tolerance for adjusting the MVP cutoff once Phase 2 starts?~~ **Resolved 2026-05-12: Soft — additions OK if balanced 1-for-1 with removals.** Discipline: any new in-scope item written into §2–§7 must be paired with an explicit removal of comparable effort, moved to §9 with a one-line note. Estimates of the swap are honest, not aspirational. The candidate keeps a brief "swap log" at the top of §9 so the v1.0 trajectory is auditable. **Why this is safer than "decide in the moment":** the rule forces conscious choice and surfaces creeping expansion early. **Why this is safer than "strict":** real learning during Phase 2 (e.g. discovering the agent loop needs more orchestration than estimated) deserves to influence scope, not be suppressed. | Done | R6, §8 |
