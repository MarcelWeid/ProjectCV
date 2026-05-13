# config/

Versioned configuration files that change bot behaviour without recompilation.
Treated as first-class architecture (see `docs/03_solution_architecture.md` §3.3).

Files (created in TASK-205):

- `system_prompt.md` — the bot's persona, ground rules, refusal logic
- `retrieval_template.md` — the structural fence around retrieved chunks (LLM01 defence)
- `forbidden_patterns.txt` — output filter regex list (Th2 resolution)

All changes go through PR review like any other code change.
