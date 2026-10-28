---
name: symone-trip-planner
description: Plan a long-distance motorcycle trip in France with SYMONE from start to finish - find the nearest served cities, compare scheduled dates, prices and free places (Paris - Marseille and other lines), then hand the traveller the booking link or the quote form. Use when someone wants to move or ship a motorcycle between French cities, with or without riding along, wants to skip a long highway ride, or asks for SYMONE dates, prices or availability. The SYMONE API is read-only; booking and payment only happen on symone.fr.
license: MIT
compatibility: Needs HTTPS access to symone.fr (REST API or the MCP server https://symone.fr/mcp).
metadata:
  author: SYMONE SAS
  version: "1.0"
---

# Plan a motorcycle trip with SYMONE

SYMONE carries motorcycles on a trailer along French highways. The rider travels in the van with the bike, or the bike travels alone. Main line: Paris ⇄ Marseille, from 299 €. Prices are in euros.

## When to use

- "I need to get my bike from Paris to Marseille next month."
- "Can someone take my motorcycle to the south of France? I don't want to ride 800 km."
- "When is the next SYMONE departure from Lyon? Is there room on the 12th?"
- Comparing dates or prices for moving a motorcycle between French cities.

Do not use this skill to book or pay: the API cannot do either. The traveller books on the website.

## Tools

Use the first one available:

1. **MCP server** `https://symone.fr/mcp` (Streamable HTTP, no auth): `search_routes` (`depart`, `arrival`, `date_from`, `date_to`, `page`, `limit`), `cities_near` (`location`), `get_booking_url`.
2. **REST API**, base `https://symone.fr/api-proxy/v1`, no key:
   - `GET /route/ai/search?depart=Paris&arrival=Marseille&date_from=2026-11-01&date_to=2026-11-30&limit=50`
   - `GET /route/cities-near?location=Lyon, Auvergne-Rhône-Alpes, France`
3. **CLI**: `npx symone-cli search --from Paris --to Marseille --date-from 2026-11-01 --date-to 2026-11-30` (add `--json` for raw data).

## Steps

1. **Collect the need.** Departure city, arrival city, date window and flexibility, number of motorcycles, whether the rider travels with the bike. If no date is given, search the next 60 days. Dates use `YYYY-MM-DD`.
2. **Check the cities.** City filters are case-insensitive substrings and also match stopovers. If a search returns nothing for a small town, call `cities_near` with "Town, Region, France" and propose the nearest served cities.
3. **Search.** Call `search_routes` (or `GET /route/ai/search`) with `depart`, `arrival`, `date_from`, `date_to`. If `has_next` is true, fetch the next page with `cursor=<next_cursor>` and the same filters.
4. **Read each trip.**
   - `date`: departure day.
   - `depart`, `arrival`, `layovers`: the route and its stopovers.
   - `times`: scheduled times along the route; the first one is the departure.
   - `price_min` / `price_max`: price range in EUR. `null` means "price on quote".
   - `slots`: free motorcycle places, one number per leg. A traveller who covers several legs needs a place on each, so the smallest number counts.
5. **Compare and present.** Show at most five options, sorted by date (or price if the traveller asked), for example:

   | Date | Route | Departure | Price | Free places |
   | --- | --- | --- | --- | --- |
   | Fri 6 Nov | Paris → Marseille (via Lyon) | 07:30 | 299–349 € | 2 |

   Point out the cheapest and the earliest option. Say that prices and places are indicative until the booking is confirmed on the website.
6. **Hand over.**
   - A trip fits: give `https://symone.fr/moto`, where the traveller picks the date, the option (rider and bike, or bike alone) and pays.
   - No trip fits (other city, other date, several bikes, special request): give the quote form `https://symone.fr/moto?tab=tab-devis`, plus reservation@symone.fr and 01 71 32 35 05 (+33 1 71 32 35 05).

## Example

User: "I'm in Paris and want my bike in Marseille around 10 November, I'll ride back."

1. `search_routes {"depart": "Paris", "arrival": "Marseille", "date_from": "2026-11-05", "date_to": "2026-11-15"}`
2. Two trips found → table with both, the cheaper one highlighted.
3. "Book the one you prefer on https://symone.fr/moto (choose 'moto + pilote' if you travel with the bike). For other dates, use the quote form https://symone.fr/moto?tab=tab-devis."

## Rules and edge cases

- Never say a trip is booked, reserved or held, and never ask for payment details: only the website books and takes payment.
- No result in the window: widen it (±7 days), try the reverse direction or a nearby city, then offer the quote form.
- HTTP 429: wait the `Retry-After` delay, retry once, then tell the user to try again shortly. Limits per minute: 180 searches, 30 for cities-near, 60 MCP calls.
- The sandbox (`https://symone.fr/api-proxy/v1/sandbox`, trips dated 2030) is only for testing code; never show sandbox trips to a traveller.
- Answer in the traveller's language. Keep the city names as the API returns them.

## Links

- Book: https://symone.fr/moto · Quote: https://symone.fr/moto?tab=tab-devis
- API docs: https://symone.fr/developers · https://symone.fr/api/index.md
