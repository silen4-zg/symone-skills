# SYMONE - Claude Code

Read `AGENTS.md` at the repository root: API facts, skills layout and rules.

Essentials:

- API base `https://symone.fr/api-proxy/v1`, sandbox `https://symone.fr/api-proxy/v1/sandbox`, no authentication.
- Trip search: `GET /route/ai/search?depart=Paris&arrival=Marseille&date_from=YYYY-MM-DD&date_to=YYYY-MM-DD`, then follow `next_cursor`.
- MCP servers: `https://symone.fr/mcp` (trips) and `https://symone.fr/docs/mcp` (docs). Add them with `claude mcp add --transport http symone https://symone.fr/mcp`.
- The API cannot book or pay: booking happens on https://symone.fr/moto.
