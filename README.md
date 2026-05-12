# ProjectCV

> A grounded chatbot that introduces me to recruiters — answering questions about my work in the context of the company they're hiring for.

A recruiter opens a link with a company-scoped token (`?c=...`). The chatbot answers their questions about my experience, drawing on two pre-curated knowledge sources: a corpus about me (my CV, project history, working style), and a corpus about their company (publicly available material I've reviewed and ingested ahead of time). Answers are grounded in retrieved chunks from both, with citations.

The point isn't novelty for its own sake. It's that "tell me about your experience with X" gets a more useful answer when the system actually knows what X means at *their* company.

## At a glance

| | |
|---|---|
| Backend | ASP.NET Core (.NET 9), tool-using agent over Azure OpenAI (GPT-4o) |
| RAG | Postgres + pgvector (semantic) and `tsvector` (keyword) on Hetzner Managed DB; hybrid retrieval surfaced to the model as four tools |
| Frontend | Vanilla TypeScript + Vite, server-sent-events streaming |
| Hosting | Hetzner Cloud VM behind Caddy (auto-TLS, HSTS, strict CSP) |
| Scope | One candidate, no accounts, no analytics, no chat persistence |

## What's interesting

- **Four-tool agent, not a fixed RAG pipeline.** The model decides per turn whether to call `search_candidate_semantic`, `search_candidate_keyword`, `search_company_semantic`, or `search_company_keyword`. Retrieval reasoning is visible in the logs and citations — see [docs/03_solution_architecture.md](docs/03_solution_architecture.md#3-component-view--backend-api-internals).
- **Hardened against the obvious abuses.** Per-token and per-IP rate limits, daily and monthly Azure-spend hard ceilings, an out-of-band kill switch, retrieved-content fencing against indirect prompt injection, output filter against forbidden patterns. STRIDE-per-trust-boundary plus OWASP LLM Top 10 (2025) coverage in [docs/04_threat_model.md](docs/04_threat_model.md), with a red-team test suite gating each release.
- **Grounded or silent.** The bot refuses to answer factual questions when retrieval returns nothing strong. No confabulated employers, projects, or dates. Verified by a `golden.yml` eval set that runs on every release tag — see [docs/05_rag_design.md](docs/05_rag_design.md#6-eval-harness--the-most-important-section).
- **Compliance treated as design, not paperwork.** The scraper refuses to run on a company without a current `legal_review` block in its config. Chat content is never persisted. AI disclosure is visible before the first message. Details in [docs/06_legal_compliance.md](docs/06_legal_compliance.md).

## Repository

```
src/        .NET solution: API, scraper, candidate ingest, token admin, eval
docs/       Project planning documents (charter, SRS, architecture, threat model, RAG, legal, WBS, test strategy)
config/     System prompt and retrieval template, versioned
infra/      Terraform and Ansible for Hetzner provisioning
tests/      Unit, integration, red-team (cheap + live), eval harness, load test
companies/  Per-company scrape configurations (data itself not in this repo)
```

## What's *not* in this repo

- My CV content and the scraped company data. The repo is code and infrastructure; the corpora are private inputs to it.
- Any secrets. The build expects an `.env` it cannot see.

## Live site

[link to live deployment — to be filled in once Phase 5 completes]

## Reading order

If you have 60 seconds: this README and [the project charter](docs/01_project_charter.md).
If you have 15 minutes: add the [architecture](docs/03_solution_architecture.md) and the [threat model](docs/04_threat_model.md).
If you have an hour and you enjoy reading planning documents: the full eight-document set in `docs/` was written before any code, in roughly that order.

## Contact

[Name] — [email]
