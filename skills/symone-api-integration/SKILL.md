---
name: symone-api-integration
description: Integrate the public SYMONE motorcycle transport API into an app, script or agent, starting in the sandbox and then switching to production. Covers trip search with cursor pagination, nearby cities, batch searches with Idempotency-Key, asynchronous search jobs, the NLWeb /ask endpoint, rate limits with Retry-After, JSON errors, versioning, and the ready-made Python SDK, npm CLI and MCP servers. Use when writing, reviewing or debugging code that calls symone.fr/api-proxy, or when a developer asks how to show SYMONE trips, prices or availability in their product. No API key; the API is read-only and cannot book or pay.
license: MIT
compatibility: Needs HTTPS access to symone.fr. Examples use curl, Python 3.9+ or Node.js 18+.
metadata:
  author: SYMONE SAS
  version: "1.0"
---

# Integrate the SYMONE API

Public REST API for SYMONE scheduled motorcycle transport trips in France. No account and no key. It only reads: bookings and payments happen on https://symone.fr/moto.

## When to use

- Adding SYMONE trips, prices or free places to a website, app, chatbot or agent.
- Writing a script that monitors departures between two cities.
- Reviewing or fixing code that calls `symone.fr/api-proxy`.

For a one-off trip search for a traveller, use the `symone-trip-planner` skill instead.

## Bases

| Environment | Base URL |
| --- | --- |
| Sandbox | `https://symone.fr/api-proxy/v1/sandbox` |
| Production | `https://symone.fr/api-proxy/v1` |
| Legacy alias (unversioned) | `https://symone.fr/api-proxy` |

The sandbox has the same paths, fixed sample trips dated 2030 and never production data. Every sandbox response has the header `X-Symone-Sandbox: true`, and object responses contain `"sandbox": true`.

## Steps

1. **Start in the sandbox.** Point the code at the sandbox base and check the `X-Symone-Sandbox` header in a test.

   ```bash
   curl -i "https://symone.fr/api-proxy/v1/sandbox/route/ai/search?depart=Paris&arrival=Marseille&limit=5"
   ```

2. **Search trips**: `GET {base}/route/ai/search`.
   - Query: `depart`, `arrival` (case-insensitive substrings that also match stopovers), `date_from`, `date_to` (`YYYY-MM-DD`), `limit` (1-100, default 50), `page` (>= 1), `cursor`.
   - Response: `{ routes: [{ id, date, depart, arrival, layovers, price_min, price_max, slots, times }], total, page, limit, total_pages, has_next, next_cursor, book_url }`.
   - Prices are in EUR; `null` means on quote. `slots` holds the free motorcycle places, one number per leg.

3. **Paginate with the cursor.** While `has_next` is true, call again with `cursor=<next_cursor>` and the same filters (`page` is ignored when a cursor is present). A `Link: <...>; rel="next"` header gives the same next URL.

   ```python
   import json, time, urllib.parse, urllib.request
   from urllib.error import HTTPError

   BASE = "https://symone.fr/api-proxy/v1/sandbox"  # switch to .../v1 for production

   def get(path, params):
       url = f"{BASE}{path}?{urllib.parse.urlencode(params)}"
       for attempt in range(2):
           try:
               with urllib.request.urlopen(url, timeout=30) as response:
                   return json.load(response)
           except HTTPError as error:
               if error.code == 429 and attempt == 0:
                   time.sleep(float(error.headers.get("Retry-After", "1")))
                   continue
               raise RuntimeError(json.load(error).get("error", f"HTTP {error.code}"))

   routes, params = [], {"depart": "Paris", "arrival": "Marseille", "limit": 100}
   while True:
       page = get("/route/ai/search", params)
       routes += page["routes"]
       if not page["has_next"]:
           break
       params = {**params, "cursor": page["next_cursor"]}
   ```

4. **Handle errors and limits.**
   - Errors are JSON: `{ "error": "message" }` (400 for invalid parameters such as a bad cursor or missing location, 404 for an unknown job).
   - Limits per client per minute: 180 reads, 30 actions (cities-near, batch, jobs, `/ask`), 60 MCP calls. A 429 carries `Retry-After` (seconds): wait, retry once, then back off.
   - Cache search results for a few minutes instead of polling in a loop.

5. **Use the other endpoints when they fit.**
   - Nearby served cities: `GET {base}/route/cities-near?location=Paris, Île-de-France, France` returns `{ cities, source }`. `location` is required.
   - Several searches at once: `POST {base}/route/ai/batch` with `{ "queries": [ {depart, arrival, date_from, date_to}, ... ] }` (1 to 10), plus an `Idempotency-Key` header so a retry is safe. Returns `{ results: [{ index, query, routes, total }], book_url }`.
   - Asynchronous search: `POST {base}/route/ai/jobs` with `{depart, arrival, date_from, date_to}` returns 202 `{ job_id, status: "pending", poll_url }`. Poll `GET {base}/route/ai/jobs/{job_id}` until `status` is `completed` (read `result`) or `failed`. Jobs expire after 10 minutes (then 404).
   - Natural language: `GET https://symone.fr/ask?q=Paris%20Marseille` or `POST https://symone.fr/ask` with `{ "query": { "text": "..." }, "prefer": { "streaming": false } }` returns `{ _meta, results: [schema.org Trip], answer? }`. `/ask` has no sandbox.

6. **Hand the user over for booking.** Show `book_url` (`https://symone.fr/moto`) next to the trips, and the quote form `https://symone.fr/moto?tab=tab-devis` when nothing matches. Never build a booking or payment flow on top of this API, and never present a trip as reserved.

7. **Switch to production.** Change only the base URL to `https://symone.fr/api-proxy/v1`. Keep the `/v1` prefix; read the versioning and deprecation policy at https://symone.fr/api/versioning.md.

## Shortcuts

- Python SDK (standard library only): `pip install symone`, then `SymoneClient(sandbox=True).search_routes(depart="Paris")`. It follows cursors (`iter_routes`), retries 429 once and raises typed errors.
- Node.js CLI: `npx symone-cli --sandbox search --from Paris --json`.
- MCP for agents: `https://symone.fr/mcp` (search) and `https://symone.fr/docs/mcp` (documentation). `npx symone-cli mcp-config cursor` prints a client config.

## Checklist before shipping

- [ ] Tests run against the sandbox and assert `X-Symone-Sandbox: true`.
- [ ] Pagination uses `next_cursor` with the same filters.
- [ ] 429 handled with `Retry-After`; no tight polling loops.
- [ ] `{ "error" }` messages are logged; empty results lead to the quote form.
- [ ] The UI links to https://symone.fr/moto and never claims a booking.

## Links

- Developer portal: https://symone.fr/developers (Markdown: https://symone.fr/developers.md)
- API reference: https://symone.fr/api/index.md · OpenAPI: https://symone.fr/openapi.json
- Versioning: https://symone.fr/api/versioning.md · LLM index: https://symone.fr/llms.txt
