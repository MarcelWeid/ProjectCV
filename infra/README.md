# infra/

Infrastructure as code and deployment configuration.

Structure (most created in Phase 5 / TASK-501+):

- `terraform/` — Hetzner Cloud resource definitions (VM, firewall, optionally managed DB)
- `ansible/` — VM bootstrap playbook (Docker, Caddy, fail2ban, log rotation, Uptime Kuma)
- `docker-compose.yml` — production stack composition (api, caddy, uptime-kuma)
- `Caddyfile` — reverse proxy config (TLS, security headers, CSP)
- `azure-config.md` — manual operational notes for Azure OpenAI resource (region, training-opt-out, etc.)

The VM provisioned in Phase 0 (TASK-501, snapshot `hardened-base-2026-05-12`)
was created manually; Ansible playbook will reproduce that hardening for
reproducibility in TASK-502 during Phase 5.
