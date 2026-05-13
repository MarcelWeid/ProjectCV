# tests/

Test code, fixtures, and evaluation artifacts.

Structure (created in TASK-101 and TASK-404):

- `ProjectCv.UnitTests/` — fast, in-memory, gate every PR
- `ProjectCv.IntegrationTests/` — Testcontainers Postgres, mocked Azure, gate every PR
- `redteam/cheap/` — mocked LLM, asserts on prompt assembly, gate every PR
- `redteam/live/` — real Azure, asserts on model behaviour, gate `release-*` tags only
- `ProjectCv.Eval/` — RAG evaluation harness, gate `release-*` tags only
- `fixtures/corpus/` — synthetic test corpus (candidate + 3 companies), separate from real data
- `load/` — k6 load test scripts, run once in Phase 4

See `docs/08_test_strategy.md` for the full pyramid and traceability matrix.
