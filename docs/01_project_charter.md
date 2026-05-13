# Project Charter — Recruiter Chatbot ("ProjectCV")

**Document status:** Draft v0.1
**Owner:** [You]
**Last updated:** 2026-05-12

> Working title: ProjectCV. Replace with the real product name once chosen.

---

## 1. Problem statement

A traditional CV and LinkedIn profile force a recruiter to do two jobs: extract relevant facts about the candidate, *and* mentally map them onto the hiring company's context. That mapping is exactly what gets a candidate to the shortlist — and it's exactly what most CVs leave to chance.

For an experienced IT project manager, the relevant facts are too numerous and too contextual to surface evenly in a static document. The right project example for one company is rarely the right one for the next — industry, culture, tech stack, and the role's specific shape all change which parts of a career are most relevant.

## 2. Vision

A single shareable link — `projectcv.example/?company=<token>` — that opens a chat interface. The bot answers a recruiter's questions about the candidate using two grounded knowledge sources:

1. **Candidate corpus** (CV, project portfolio, working principles, references) — authoritative facts about the candidate.
2. **Company corpus** (pre-scraped public information about the recruiter's company — values, products, recent press, org structure) — context for tailoring answers.

The bot uses both to produce answers that are honest about the candidate *and* framed in the company's language. It also serves as a live demonstration of the candidate's engineering, security, and product-thinking capabilities, because the project itself is the work sample.

## 3. Success criteria

| Dimension | Criterion | How measured |
|---|---|---|
| Recruiter signal | At least 3 recruiters interact with the bot and reference it in a conversation | Manual log of recruiter feedback |
| Engineering signal | The GitHub repo is judged "above bar" by at least 2 senior engineers in the candidate's network | Direct review request |
| Technical robustness | No successful prompt injection that causes the bot to say something false or harmful about the candidate in red-team testing | Documented red-team test suite passes |
| Cost discipline | Monthly Azure OpenAI spend stays under €30 in normal use; hard ceiling at €100 with automatic cutoff | Budget alerts + per-token cap |
| Availability | 99% over any rolling 30-day window (this is a portfolio site, not a bank) | Uptime monitor |

## 4. In scope

- Public-facing website with embedded chat UI
- .NET (ASP.NET Core) backend exposing a chat API
- Postgres + pgvector on Hetzner Managed DB for vector storage and operational data
- Offline scraper (separate tool) that ingests ~20 pre-curated companies
- Azure OpenAI integration for embeddings and chat completion
- Company-scoped access via opaque tokens passed in URL (`?c=<token>`)
- Rate limiting per token and per IP
- Hardening against prompt injection, jailbreaks, cost-exhaustion abuse
- Public GitHub repository with code, infrastructure-as-code, and documentation
- Transparency features (AI disclosure, source citations)
- GDPR-compliant handling of any recruiter-entered data
- German imprint (Impressum) and privacy notice

## 5. Out of scope

- User accounts, login, or session persistence beyond the current chat
- Multi-tenant or multi-candidate support (single candidate, single deployment)
- On-demand scraping at request time (all company data is pre-curated)
- Voice or video interaction
- Mobile apps (responsive web only)
- Real-time scraping refresh (companies are re-scraped manually, on the order of weeks)
- Storing or processing recruiter PII beyond what is strictly necessary to operate the chat
- Any kind of "matching" or scoring the recruiter against the candidate
- Acting as a system of record for the candidate's CV — the canonical CV lives elsewhere

## 6. Key assumptions

- The candidate has an active Azure subscription with Azure OpenAI access provisioned.
- The candidate has a Hetzner Cloud account with budget for one CX-class VM and one managed Postgres instance (~€20–30/month combined).
- Target companies' public websites can be scraped within fair-use and ToS bounds (verified case-by-case during the legal/compliance task).
- Recruiters will accept a "this is an AI bot grounded in public information" disclosure without friction.
- The candidate is willing to red-team the bot before sharing the link.

## 7. Top risks

| # | Risk | Likelihood | Impact | Initial response |
|---|---|---|---|---|
| R1 | Bot hallucinates a claim about the candidate (false project, wrong dates) | Medium | High (credibility) | Strict grounding, refusal-to-answer when retrieval is weak, explicit citations |
| R2 | Prompt injection causes the bot to say something embarrassing or off-brand | Medium | High (credibility) | Input filtering, output filtering, red-team test suite, conservative system prompt |
| R3 | Cost exhaustion attack — someone scripts thousands of queries | Medium | Medium (€€€) | Rate limits, hard daily budget cap with automatic kill switch |
| R4 | Scraped company data violates ToS or copyright | Low | Medium (legal) | Restrict to clearly public pages, robots.txt respect, store excerpts not full copies, legal review checklist before each company is added |
| R5 | GDPR exposure from logging recruiter prompts that contain personal data | Medium | Medium (legal) | Minimal logging, short retention, prompt sanitization, privacy notice |
| R6 | Project drags on and is never shipped | High | High (opportunity cost) | Time-box to a fixed MVP, prefer "shipped and rough" over "polished and pending" |

R6 is the biggest one and deserves to be named honestly. Portfolio projects die from scope creep more than from any technical problem.

## 8. Timeline envelope

The plan assumes evening/weekend work. Adjust elastically.

| Phase | Calendar duration | What "done" means |
|---|---|---|
| 0. Setup & decisions | 1 week | Charter approved, repo created, Hetzner + Azure accounts provisioned, domain registered |
| 1. RAG core (offline) | 2 weeks | Scraper works for 3 pilot companies; embeddings stored in pgvector; can retrieve relevant chunks via a CLI |
| 2. Backend API | 2 weeks | .NET API serves a grounded chat response end-to-end; auth tokens work; rate limiting works |
| 3. Frontend | 1–2 weeks | Recruiter can open the link, chat, see citations, get a sensible refusal on bad prompts |
| 4. Security hardening | 1 week | Threat model documented; red-team suite green; budget kill switch tested |
| 5. Ops & go-live | 1 week | Deployed to Hetzner, monitoring up, Impressum + privacy notice published, repo public |
| 6. Polish & sharing | ongoing | Pre-scrape remaining target companies, iterate on the system prompt, share with first recruiters |

**Total to first share-ready version: ~8–9 weeks of part-time work.** That's the headline number to remember.

## 9. Stakeholders

| Role | Person | Interest |
|---|---|---|
| Sponsor & owner | [You] | All of it |
| Primary users | Recruiters at ~20 target companies | A useful, honest, working bot |
| Secondary reviewers | Engineers in your network | Code quality, architecture, security |
| Regulator (passive) | German/EU data protection authorities | GDPR compliance |
| Vendor | Anthropic / Microsoft / Hetzner | None — you're a paying customer |

## 10. Decision log (open items)

| # | Decision needed | Owner | By |
|---|---|---|---|
| D1 | Product name | You | Before repo creation |
| D2 | Domain name | You | Before phase 5 |
| D3 | Exact list of pilot companies (3) and full set (~20) | You | Before phase 1 / phase 6 |
| D4 | LLM model tier (GPT-4o vs. GPT-4o-mini for cost) | You + me | During phase 2 |
| D5 | ~~UI tone~~ **Resolved 2026-05-13 in DEC-006.** Leicht persönlich-sachlich (Variante 2), Hauptsprache Deutsch, Anrede spiegelt Recruiter (Sie/Du). | Done | Done |

---

## What this document is not

This charter is intentionally short. It is not a Statement of Work for a vendor, not an ASPICE-conformant project plan, and not a contract. It is a **shared mental model** for a one-person project that wants to retain the discipline of a real one without drowning in process. The companion documents (Requirements, Architecture, Threat Model, Test Strategy, etc.) will go deeper where depth pays off.
