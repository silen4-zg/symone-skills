---
name: symone-docs-answers
description: Answer questions about SYMONE (long-distance motorcycle transport in France) and its developer platform from the official documentation, using the SYMONE docs MCP server (search_docs, fetch_doc) or the public Markdown pages, and cite the source page. Use for questions on how the service works, what is included, prices and options, contact, legal and privacy pages, or the public API, MCP servers, sandbox, rate limits and versioning. Never guess - if the documentation does not say it, say so and give the contact details.
license: MIT
compatibility: Needs HTTPS access to symone.fr (docs MCP server https://symone.fr/docs/mcp or the Markdown pages).
metadata:
  author: SYMONE SAS
  version: "1.0"
---

# Answer from the SYMONE documentation

## When to use

- "How does SYMONE work? Do I travel with my bike?"
- "What is included in the price? What happens if I cancel?"
- "What are the API rate limits? Is there a sandbox?"
- Any question whose answer should come from SYMONE's own pages rather than from memory.

For live dates, prices and free places, search trips instead (`symone-trip-planner` skill or the MCP server `https://symone.fr/mcp`). The documentation does not list live departures.

## Tools

1. **Docs MCP server** `https://symone.fr/docs/mcp` (Streamable HTTP, read-only, no auth):
   - `search_docs` `{ "query": "...", "limit": 5 }` returns the best matching sections with their `path` and URL.
   - `fetch_doc` `{ "path": "api/index.md" }` returns the full Markdown of one page.
2. **Without an MCP client**, the server also accepts one plain JSON-RPC request (no session needed):

   ```bash
   curl -s https://symone.fr/docs/mcp \
     -H "Content-Type: application/json" -H "Accept: application/json, text/event-stream" \
     -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"search_docs","arguments":{"query":"rate limits"}}}'
   ```

3. **Or read the Markdown pages directly**: start with the index `https://symone.fr/llms.txt`, then for example `https://symone.fr/developers.md`, `https://symone.fr/api/index.md`, `https://symone.fr/api/versioning.md`, `https://symone.fr/pricing.md`, `https://symone.fr/faq.md`, `https://symone.fr/contact.md`.

Useful paths: `llms.txt`, `llms-full.txt`, `index.md`, `moto.md`, `pricing.md`, `faq.md`, `contact.md`, `a-propos.md`, `privacy.md`, `confidentialite.md`, `developers.md`, `api/index.md`, `api/versioning.md`, `auth.md`, `integrations.md`, `status.md`.

## Steps

1. **Search.** Call `search_docs` with the question or its keywords. Many pages are in French: if the first search is weak, search again with French keywords (for example "moto seule tarif", "annulation", "assurance", "bagages").
2. **Read.** Open the one or two best pages with `fetch_doc` (path from the results) so the answer relies on the full text, not only on a snippet.
3. **Answer briefly** in the user's language, using only what the pages say. Quote figures exactly (prices in EUR, limits, dates).
4. **Cite** the page URL(s), for example `Source: https://symone.fr/pricing.md`.
5. **When the docs are silent or unclear**, say so plainly and give the contacts: reservation@symone.fr, 01 71 32 35 05 (+33 1 71 32 35 05), or the quote form https://symone.fr/moto?tab=tab-devis.

## Example

Question: "Is there a test environment for the SYMONE API?"

1. `search_docs {"query": "sandbox test environment"}` → `developers.md`, `api/index.md`.
2. `fetch_doc {"path": "developers.md"}`.
3. Answer: "Yes. Use https://symone.fr/api-proxy/v1/sandbox: same paths, fixed sample trips dated 2030, never production data; responses carry the header X-Symone-Sandbox: true. Source: https://symone.fr/developers.md"

## Rules

- Never invent prices, policies, schedules or legal terms; never promise a booking. Bookings and payments only happen on https://symone.fr/moto.
- Prefer the most specific page; if two pages disagree, say so and give both URLs.
- The docs MCP server allows 60 calls per minute; on HTTP 429 wait the `Retry-After` delay.
