# src/

.NET solution and source code.

Projects (created in TASK-101):

- `ProjectCv.Domain` — domain models, no infrastructure dependencies
- `ProjectCv.Infrastructure` — EF Core, Azure OpenAI client, repositories
- `ProjectCv.Api` — ASP.NET Core Web API, agent loop, tool router, guards
- `ProjectCv.Scraper` — standalone CLI for company corpus ingestion
- `ProjectCv.IngestCandidate` — standalone CLI for candidate corpus ingestion
- `ProjectCv.TokenAdmin` — standalone CLI for token issuance / revocation
- `ProjectCv.Eval` — RAG evaluation harness
- `ProjectCv.Tests` — unit and integration tests

See `docs/03_solution_architecture.md` §2 for the container view.
