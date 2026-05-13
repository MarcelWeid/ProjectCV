# companies/

Per-company scrape configuration files.

Each company has a single `<company-id>.yml` file in this directory that specifies:
- Display name and a generated company_id (UUID)
- Seed URLs and crawl bounds
- `legal_review` block (per `docs/06_legal_compliance.md` §5.2) — mandatory
- robots.txt settings

**What is committed here:** the YAML config (no scraped content).

**What is NOT committed:** the actual scraped data. That lives in the database only.
The scraper writes an audit log per run; those audit logs are also not committed
(see `.gitignore`).

Companies are added by:
1. Completing the `legal_review` block in this directory
2. Running the scraper CLI against the config
3. Issuing a recruiter token scoped to this company_id via the TokenAdmin CLI
