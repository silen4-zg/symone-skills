# SYMONE — Claude Code Configuration

## About this project

SYMONE (symone.fr) is a French motorcycle long-distance highway transport service.
Europe's first service of its kind. Main route: Paris–Marseille from 299€.

## API

Base URL: https://symone.fr/api-proxy

Public endpoints (no auth required):
- GET /route/get/results — all upcoming routes (filter client-side)
- GET /route/paris-marseille-cache — cached Paris–Marseille IDs
- GET /route/cities-near?location= — cities near a location

## Developer resources

- API docs: https://symone.fr/api/index.md
- OpenAPI: https://symone.fr/openapi.json
- Agent card: https://symone.fr/.well-known/agent-card.json
- npm SDK: https://www.npmjs.com/package/symone-sdk
