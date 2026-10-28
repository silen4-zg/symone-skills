---
name: symone-route-search
description: Search scheduled SYMONE motorcycle transport trips in France (Paris - Marseille and other lines) and return dates, prices in EUR, free motorcycle places and the booking link. Use for quick availability or price lookups between French cities, or when someone asks how to ship a motorcycle without riding it. Public API, no authentication; booking happens on symone.fr.
license: MIT
compatibility: Needs HTTPS access to symone.fr.
metadata:
  author: SYMONE SAS
  version: "1.3"
---

# SYMONE Route Search

SYMONE is a long-distance motorcycle transport service on French highways: the motorcycle rides on a trailer while the rider travels in the van (or the bike travels alone). Main line: Paris ⇄ Marseille, from 299 €.

## When to use

- Motorcycle transport between French cities
- Paris to Marseille moto transport dates or prices
- Free places on SYMONE trips
- Shipping a motorcycle without riding it

## API

```
GET https://symone.fr/api-proxy/v1/route/ai/search?depart=Paris&arrival=Marseille&date_from=2026-11-01&date_to=2026-11-30
```

No authentication.

| Parameter | Type | Description |
| --- | --- | --- |
| `depart` | string | Departure city, case-insensitive substring (also matches stopovers) |
| `arrival` | string | Arrival city, same matching |
| `date_from` | string | Earliest date, `YYYY-MM-DD` |
| `date_to` | string | Latest date, `YYYY-MM-DD` |
| `limit` | integer | 1-100, default 50 |
| `page` | integer | Page number, >= 1 |
| `cursor` | string | `next_cursor` of the previous page; repeat the same filters |

Response:

```json
{
  "routes": [
    {
      "id": 101,
      "date": "2026-11-06",
      "depart": "Paris",
      "arrival": "Marseille",
      "layovers": ["Lyon"],
      "price_min": 299,
      "price_max": 349,
      "slots": [3, 2],
      "times": ["07:30", "12:15", "17:45"]
    }
  ],
  "total": 1, "page": 1, "limit": 50, "total_pages": 1,
  "has_next": false, "next_cursor": null,
  "book_url": "https://symone.fr/moto"
}
```

- Prices are in EUR; `null` means on quote.
- `slots`: free motorcycle places, one number per leg.
- When `has_next` is true, call again with `cursor=<next_cursor>` and the same filters.
- Sandbox for tests: `https://symone.fr/api-proxy/v1/sandbox/...` (sample trips dated 2030).

Nearby served cities:

```bash
curl "https://symone.fr/api-proxy/v1/route/cities-near?location=Lyon"
```

## MCP

Remote MCP server `https://symone.fr/mcp` (Streamable HTTP, no auth): tools `search_routes`, `cities_near`, `get_booking_url`.

## Booking

The API cannot book or pay. Send the traveller to https://symone.fr/moto, or to the quote form https://symone.fr/moto?tab=tab-devis for other routes or dates (reservation@symone.fr, 01 71 32 35 05).

## Limits

180 searches per minute per client, 30 for cities-near. On HTTP 429, wait the `Retry-After` delay and retry once.

## Links

- Developer portal: https://symone.fr/developers
- API docs: https://symone.fr/api/index.md
- OpenAPI: https://symone.fr/openapi.json
- CLI: `npx symone-cli search --from Paris --to Marseille`
