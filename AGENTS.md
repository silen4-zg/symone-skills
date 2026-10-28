# AGENTS.md - SYMONE skills and plugin

This repository publishes SYMONE's agent skills and MCP servers as one [Agent Plugin](https://agent-plugins.org/specification). SYMONE (https://symone.fr) carries motorcycles on a trailer along French highways while the riders travel in the van; main line Paris ⇄ Marseille from 299 €.

## Layout

| Path | Content |
| --- | --- |
| `plugin.json` | Agent Plugins manifest. Closed schema: only `$schema`, `name`, `version`, `description`, `author`, `homepage`, `repository`, `license`, `keywords`, `extensions`. |
| `mcp.json` | The two remote MCP servers (Streamable HTTP, no auth). |
| `skills/<name>/SKILL.md` | [Agent Skills](https://agentskills.io/specification), one folder per skill. |
| `.claude/CLAUDE.md`, `.cursor/rules` | Short pointers for Claude Code and Cursor. |

Skills: `symone-route-search` (quick trip lookup), `symone-trip-planner` (plan a trip end to end and hand over the booking link), `symone-api-integration` (integrate the API in code, sandbox first), `symone-docs-answers` (answer from the official docs).

## SYMONE API facts

- REST base `https://symone.fr/api-proxy/v1`; sandbox `https://symone.fr/api-proxy/v1/sandbox` (sample trips dated 2030, header `X-Symone-Sandbox: true`). No authentication.
- Search: `GET /route/ai/search?depart=&arrival=&date_from=YYYY-MM-DD&date_to=YYYY-MM-DD&limit=1-100&cursor=` returns `routes`, `total`, `has_next`, `next_cursor`, `book_url`.
- Also: `GET /route/cities-near?location=`, `POST /route/ai/batch` (1-10 queries, `Idempotency-Key`), `POST /route/ai/jobs` + `GET /route/ai/jobs/{id}`, NLWeb `https://symone.fr/ask`.
- Errors are JSON `{ "error": "message" }`. Limits per client per minute: 180 reads, 30 actions, 60 MCP calls; 429 carries `Retry-After`.
- MCP: `https://symone.fr/mcp` (`search_routes`, `cities_near`, `get_booking_url`) and `https://symone.fr/docs/mcp` (`search_docs`, `fetch_doc`).
- Packages: `npx symone-cli` (Node.js CLI), `pip install symone` (Python SDK), `npx @rosentte104/symone-mcp` (stdio bridge).

## Rules for agents

- The API is read-only. Never claim a trip is booked, reserved or paid, and never ask for payment details. Booking: https://symone.fr/moto; quote for other routes or dates: https://symone.fr/moto?tab=tab-devis; reservation@symone.fr; 01 71 32 35 05.
- Use the sandbox for tests and examples that run code; never show sandbox trips to a traveller.
- Respect the rate limits: wait `Retry-After`, retry once, cache results.

## Editing this repository

- A skill folder name must equal the `name` in its frontmatter (lowercase letters, digits, single hyphens, at most 64 characters); `description` at most 1024 characters, saying what the skill does and when to use it. Keep each `SKILL.md` under 500 lines.
- `mcp.json` and `plugin.json` must use the same Agent Plugins version in `$schema` (1.0.0).
- Bump `version` in `plugin.json` when skills or servers change.
- Check skill discovery with `npx skills add silen4-zg/symone-skills --list`.

## Links

Developer portal https://symone.fr/developers · API https://symone.fr/api/index.md · OpenAPI https://symone.fr/openapi.json · LLM index https://symone.fr/llms.txt
