# Requirements Specification — ProjectCV

**Document status:** Draft v0.1
**Owner:** [You]
**Last updated:** 2026-05-12
**Companion documents:** 01_project_charter.md

---

## 0. About this document

This is the Software Requirements Specification (SRS) for ProjectCV — the personal recruiter-chatbot website described in the project charter. It is the single source of truth for *what* the system must do. *How* it does any of it belongs in the Architecture document.

### 0.1 Conventions

- **Requirement IDs** are stable and unique within their category. Once assigned, an ID is never reused. If a requirement is removed, its ID is marked `[withdrawn]` and not reassigned.
- **Priority** uses MoSCoW: **M** = Must (MVP-blocking), **S** = Should (target for v1.0), **C** = Could (nice-to-have, deferred without regret).
- **Verification method** is one of: **T** = automated test, **I** = inspection/code review, **D** = demonstration, **A** = analysis / manual review.
- Where a requirement constrains another (e.g. a security control on a functional behaviour), the dependency is noted in the **Refs** column.

### 0.2 Categories

| Prefix | Category | Owner of verification |
|---|---|---|
| FR | Functional Requirements | Test suite |
| NFR | Non-Functional Requirements | Test suite + ops checks |
| SR | Security Requirements | Threat model + red-team suite |
| CR | Compliance & Legal Requirements | Manual review + legal checklist |
| DR | Data Requirements | Schema review + ingestion tests |
| OR | Operational Requirements | Runbook + monitoring checks |

---

## 1. Functional Requirements

### 1.1 Access & entry

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| FR-001 | The system shall serve a single-page public website at a configured domain. | M | D | — |
| FR-002 | The system shall accept a company-scoping token via the URL query parameter `?c=<token>`. | M | T | — |
| FR-003 | If a valid `c` token is present, the chatbot shall load with the corresponding company context active. | M | T | FR-002, DR-010 |
| FR-004 | If the `c` token is missing, invalid, expired, or revoked, the chatbot shall load in **generic mode** (candidate corpus only, no company context). | M | T | FR-002, SR-014 |
| FR-005 | The system shall not require the recruiter to register, log in, or provide an email address to use the chatbot. | M | I | — |
| FR-006 | The system shall display an unambiguous "This is an AI assistant" disclosure on first chat load, before the first user message is sent. | M | I | CR-002 |

### 1.2 Conversation

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| FR-010 | The chatbot shall accept free-text questions in a chat-style interface. | M | D | — |
| FR-011 | The chatbot shall respond in the same language as the user's most recent message, supporting at minimum English and German. | M | T | — |
| FR-012 | The chatbot shall ground every substantive answer about the candidate in retrieved chunks from the candidate corpus. | M | T | DR-001, SR-001 |
| FR-013 | The chatbot shall ground every substantive answer about the recruiter's company in retrieved chunks from that company's corpus, if and only if a valid company token was supplied. | M | T | DR-010, FR-003 |
| FR-014 | The chatbot shall return a clear, friendly refusal-to-answer when the candidate corpus does not contain enough information to answer factually, rather than fabricating. The refusal shall suggest the recruiter contact the candidate directly for that question. | M | T | SR-001 |
| FR-015 | The chatbot shall expose, alongside each response, the list of source chunks it used (title + short excerpt + corpus of origin). The recruiter may toggle this view; it is collapsed by default. | S | D | FR-012, FR-013 |
| FR-016 | The chatbot shall preserve conversation history within the current browser session only. History shall not survive a page reload or be sent to the server beyond the active request. | M | T | DR-020, CR-010 |
| FR-017 | The chatbot shall offer a "start over" / "clear chat" control that drops the current in-memory history. | S | D | FR-016 |
| FR-018 | The chatbot shall stream responses token-by-token to the UI for perceived responsiveness. | S | D | NFR-002 |

### 1.3 Bot behaviour and persona

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| FR-020 | The chatbot shall answer in the first person ("I have…") when speaking about the candidate, framed as the candidate's assistant rather than as the candidate. | M | A | — |
| FR-021 | The chatbot shall not invent project names, employers, certifications, dates, or technical skills that do not appear in the candidate corpus. | M | T (red-team) | SR-001 |
| FR-022 | The chatbot shall not state opinions about specific third parties (competitors of the recruiter's company, former employers, named individuals) that are not present in the candidate corpus. | M | T (red-team) | SR-001 |
| FR-023 | The chatbot shall be capable of mapping a candidate experience to a company context when grounding allows it (e.g. "How does your experience relate to our work on X?" → grounded answer drawing from both corpora). | S | A | FR-012, FR-013 |
| FR-024 | The chatbot shall refuse to discuss salary expectations, willingness to relocate, notice period, or other negotiation-sensitive topics, deferring those to direct conversation with the candidate. | M | T (red-team) | — |
| FR-025 | The chatbot shall refuse to roleplay as the candidate making binding statements (e.g. "Confirm that you accept this offer"). | M | T (red-team) | SR-002 |

### 1.4 Admin & ingestion

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| FR-030 | The system shall provide a CLI tool (the *scraper*) to ingest a company's public web content into the company corpus. | M | T | DR-010 |
| FR-031 | The scraper shall accept a configuration file specifying the company name, seed URLs, and crawl depth/scope per company. | M | I | — |
| FR-032 | The scraper shall respect `robots.txt` and not crawl URLs disallowed by it. | M | T | CR-020 |
| FR-033 | The scraper shall identify itself with a descriptive `User-Agent` including a contact URL. | M | T | CR-020 |
| FR-034 | The scraper shall produce, for each ingestion run, an audit log listing every URL fetched, its HTTP status, and whether it was indexed. | M | I | OR-010 |
| FR-035 | The scraper shall be idempotent: re-running against the same source must not duplicate chunks in the vector store. | M | T | DR-013 |
| FR-036 | The system shall provide a CLI command to generate, revoke, and list company tokens. | M | T | FR-002, SR-014 |
| FR-037 | The system shall provide a CLI command to ingest the candidate corpus from a local directory of source files (Markdown, plain text, CV PDF). | M | T | DR-001 |

---

## 2. Non-Functional Requirements

### 2.1 Performance

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| NFR-001 | The first token of any chat response shall appear within 3 seconds at the 95th percentile under normal load (≤ 5 concurrent active chats). | S | T (load test) | — |
| NFR-002 | The full response for a typical question (≤ 200 tokens output) shall complete within 10 seconds at p95. | S | T (load test) | NFR-001 |
| NFR-003 | The website's initial page load (Largest Contentful Paint) shall be under 2.5 seconds on a 4G connection. | S | T (Lighthouse) | — |

### 2.2 Availability & reliability

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| NFR-010 | The system shall achieve at least 99% availability over any rolling 30-day window. | M | A (uptime monitor) | — |
| NFR-011 | If Azure OpenAI is unreachable, the chatbot shall display a clear, friendly degraded-mode message rather than a generic error. | M | T | — |
| NFR-012 | If the vector store is unreachable, the chatbot shall display a clear, friendly degraded-mode message and shall not fall back to ungrounded LLM output. | M | T | SR-001 |

### 2.3 Cost

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| NFR-020 | The system shall enforce a configurable hard ceiling on Azure OpenAI spend per calendar day. Once the ceiling is reached, the chat shall enter degraded mode (NFR-011) until the next day. | M | T | SR-020 |
| NFR-021 | The system shall enforce a configurable hard ceiling on Azure OpenAI spend per calendar month. Once the ceiling is reached, the chat shall enter degraded mode until manually re-enabled. | M | T | SR-020 |
| NFR-022 | Expected steady-state monthly infrastructure + LLM cost shall not exceed €50 under normal usage assumptions documented in the Architecture. | S | A | — |

### 2.4 Maintainability

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| NFR-030 | All deployable infrastructure shall be defined as code (Terraform, Ansible, Docker Compose, or equivalent) and reproducible from the public repo + a private secrets file. | M | I | — |
| NFR-031 | The system prompt, retrieval parameters, and other LLM-tunable behaviours shall be configuration, not code, and changeable without recompilation. | S | I | — |
| NFR-032 | Code shall include automated tests with a meaningful coverage of the chat orchestration and retrieval logic (no formal % target; the test suite must cover the happy path, refusal paths, and at least 20 red-team prompts). | M | I | SR-005 |

### 2.5 Portability

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| NFR-040 | The backend shall be runnable in a local development environment using Docker Compose without requiring access to Hetzner. Azure OpenAI access remains required for actual chat. | M | T | — |
| NFR-041 | The system shall support swapping Azure OpenAI for another OpenAI-compatible endpoint via configuration, without code changes. | C | I | — |

---

## 3. Security Requirements

These are the requirements the threat model will trace into. The full STRIDE-by-component analysis lives in `04_threat_model.md`.

### 3.1 Grounding & output integrity

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| SR-001 | The system shall not produce factual claims about the candidate that are not supported by retrieved chunks from the candidate corpus. Verified via a red-team suite of at least 20 prompts attempting to elicit fabrication. | M | T (red-team) | FR-012, FR-021 |
| SR-002 | The system shall not produce any output purporting to be a binding statement from the candidate (offer acceptance, salary commitment, employment terms). | M | T (red-team) | FR-025 |
| SR-003 | The system prompt shall not be disclosed to the user, even when explicitly asked. | M | T (red-team) | — |
| SR-004 | The system shall sanitise retrieved chunks before injecting them into the LLM prompt, neutralising indirect prompt injection attempts embedded in scraped content. | M | T | DR-010 |
| SR-005 | The system shall maintain a red-team test suite of prompts attempting prompt injection, jailbreak, fabrication, and persona-break attacks. The suite shall run in CI and shall be green before any deployment. | M | T | NFR-032 |

### 3.2 Access control & abuse prevention

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| SR-010 | The system shall rate-limit chat requests per `c` token to a configurable threshold (default: 30 requests / hour / token). | M | T | — |
| SR-011 | The system shall rate-limit chat requests per source IP address to a configurable threshold (default: 60 requests / hour / IP). | M | T | — |
| SR-012 | The system shall apply a configurable maximum input length per message (default: 1000 characters). | M | T | — |
| SR-013 | The system shall apply a configurable maximum output token length per response (default: 500 tokens). | M | T | NFR-020 |
| SR-014 | Company tokens shall be opaque, high-entropy, individually revocable, and bound to a single company. Token revocation shall take effect within 60 seconds. | M | T | FR-002, FR-036 |
| SR-015 | The system shall serve HTTPS only and redirect all HTTP traffic to HTTPS. HSTS shall be enabled. | M | T | — |
| SR-016 | The system shall set a strict Content-Security-Policy header that disallows third-party scripts other than those explicitly required and listed in the Architecture. | M | T | — |

### 3.3 Secret & key handling

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| SR-020 | Azure OpenAI keys, database credentials, and all other secrets shall be supplied to the backend via environment variables or a secrets file outside the repo. No secret shall ever be committed to the public repo. | M | T (pre-commit hook + CI scan) | — |
| SR-021 | The Azure OpenAI key in use shall be scoped to the minimum permissions required (single deployment, no admin) and shall be rotatable without downtime by holding two valid keys. | S | I | — |
| SR-022 | Database credentials used by the application shall be a least-privilege role with read/write on the application schema only — no superuser, no DDL outside migrations. | M | I | — |

### 3.4 Logging & PII

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| SR-030 | The system shall log enough information to diagnose abuse and operational issues, but shall not log full user message content by default. Default log fields: timestamp, token-hash, IP-hash, message length, retrieval hit count, response token count, latency, outcome. | M | I | CR-010, CR-011 |
| SR-031 | Verbose logging including full prompt/response content shall be possible only via a configuration flag intended for debugging, off in production by default. | M | I | SR-030 |
| SR-032 | Log retention shall not exceed 30 days for any log line containing a token-hash or IP-hash. | M | I | CR-010 |

---

## 4. Compliance & Legal Requirements

### 4.1 Disclosure & transparency

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| CR-001 | The website shall display a German-law-compliant Impressum (legal imprint) reachable from every page. | M | I | — |
| CR-002 | The website shall clearly state, before the user sends their first message, that they are interacting with an AI system. | M | I | FR-006 |
| CR-003 | The website shall publish a privacy notice (Datenschutzerklärung) explaining what data is processed, on what legal basis, where it goes, and for how long it is retained. | M | I | — |
| CR-004 | The privacy notice shall be reachable from every page and from a link inside the chat UI. | M | I | CR-003 |

### 4.2 GDPR

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| CR-010 | The system shall not store recruiter chat content beyond what is required to handle the active request, except as covered by SR-030 (minimal logs). | M | I | SR-030 |
| CR-011 | The system shall not use third-party analytics, advertising, or tracking that involves personal data, unless explicitly named in the privacy notice and supported by a lawful basis. The MVP shall use no such third-party services. | M | I | CR-003 |
| CR-012 | If first-party usage metrics are collected (e.g. count of chats per day), they shall be aggregated and non-identifying. | S | I | — |
| CR-013 | Azure OpenAI usage shall be configured to disable training-on-data and to use a region/data-residency option compatible with EU GDPR expectations. | M | I | — |

### 4.3 Scraping

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| CR-020 | The scraper shall respect `robots.txt`. | M | T | FR-032 |
| CR-021 | The scraper shall only fetch publicly accessible URLs (no authentication, no paywalled content, no scraping of social media APIs). | M | I | — |
| CR-022 | For each company added to the corpus, a brief legal review checklist (terms of service, copyright, source classification) shall be completed and stored in the repo as part of that company's ingestion config. | M | I | — |
| CR-023 | The scraper shall store only excerpts and embeddings sufficient for retrieval, not full verbatim copies of source pages, where the source is potentially copyright-protected. Chunk size shall be tuned to support fair-use-style excerpting. | S | I | CR-022 |

---

## 5. Data Requirements

### 5.1 Candidate corpus

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| DR-001 | The candidate corpus shall be ingested from a local directory of source files maintained by the candidate, not from public web sources. | M | I | FR-037 |
| DR-002 | The candidate corpus shall be the sole authoritative source for factual claims about the candidate. | M | I | SR-001 |
| DR-003 | Each chunk in the candidate corpus shall carry metadata: source file, section heading (if any), and a freshness date. | M | I | — |
| DR-004 | The candidate may mark specific source files as **off-limits** in their ingestion config; off-limits files shall not be ingested. | S | T | — |

### 5.2 Company corpora

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| DR-010 | Each ingested company shall have an isolated corpus, addressable by company ID, and retrievable only when the matching `c` token is presented. | M | T | FR-003, SR-014 |
| DR-011 | Each chunk in a company corpus shall carry metadata: company ID, source URL, fetch timestamp, and content type. | M | I | — |
| DR-012 | The system shall support re-ingesting a company without data loss for other companies. | M | T | FR-035 |
| DR-013 | Re-ingestion of a previously ingested URL shall replace, not duplicate, its chunks. | M | T | FR-035 |

### 5.3 Conversation & operational data

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| DR-020 | Conversation history shall live only in the client browser and the active request's memory on the server. It shall not be persisted to the database. | M | I | FR-016, CR-010 |
| DR-021 | The database shall persist: company configurations, company tokens (hashed), aggregated usage counters, and ingested chunks. Nothing else relating to recruiters shall be persisted. | M | I | — |

---

## 6. Operational Requirements

| ID | Requirement | Priority | Verification | Refs |
|---|---|---|---|---|
| OR-001 | The backend shall expose a health-check endpoint suitable for an external uptime monitor. | M | T | NFR-010 |
| OR-002 | The backend shall expose structured logs to stdout, suitable for collection by a standard log shipper. | M | I | SR-030 |
| OR-003 | The system shall send an alert to the candidate (email or equivalent) when the daily spend ceiling is hit, when the monthly spend ceiling is hit, or when the health check has been failing for more than 5 minutes. | M | T | NFR-020, NFR-021, OR-001 |
| OR-010 | Scraper audit logs (FR-034) shall be stored in the repo (as data files outside the public scope, or as redacted summaries inside it) for traceability of what was ingested when. | M | I | FR-034 |
| OR-011 | Database backups shall be enabled via the managed Postgres instance's built-in backup feature, with point-in-time recovery covering at least the last 7 days. | M | I | — |
| OR-012 | The repository shall contain a documented runbook covering: how to deploy, how to rotate keys, how to add a company, how to revoke a token, how to handle a spend-ceiling event, and how to roll back. | S | I | — |

---

## 7. Open questions feeding back into the charter

These surfaced while writing the requirements and should be answered before Architecture is finalised:

| # | Question | Affects |
|---|---|---|
| Q1 | Should the chatbot support follow-up questions referencing earlier turns (multi-turn coreference), or treat each question independently? Multi-turn is more natural but harder to ground reliably. | FR-016, SR-001, Architecture |
| Q2 | Is German required at MVP or acceptable at v1.0? FR-011 currently says "at minimum" — confirm. | FR-011, schedule |
| Q3 | What's the candidate's preferred channel for the alerts in OR-003 — email, Telegram, something else? | OR-003 |
| Q4 | Will the candidate self-host an SMTP relay or use a transactional email service (e.g. Postmark, Brevo) for alerts? Affects CR-011 / privacy notice scope. | OR-003, CR-011 |
| Q5 | Do we want a "kill switch" page the candidate can hit from a phone to immediately disable the chat (independent of the daily-ceiling mechanism)? | NFR-020, OR-012 |

---

## 8. Traceability — placeholder

A full traceability matrix (Requirement → Design element → Test) will be added in the test strategy document (`08_test_strategy.md`). The `Refs` column above is the seed for it.
