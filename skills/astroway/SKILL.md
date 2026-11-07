---
name: astroway
description: Use when the user asks for astrology, horoscope, natal chart, synastry or compatibility, transits, progressions, Vedic or KP astrology, panchang, BaZi, Zi Wei, feng shui, Human Design, tarot, numerology, I Ching or similar calculations, or is building an app, bot or site feature on the AstroWay API (api.astroway.info, mcp.astroway.info). Covers which endpoint family answers which question, connecting the hosted MCP servers, the birth data to ask for, what calls cost, and the REST contract for writing code.
---

# AstroWay

AstroWay is an astrology calculation API: REST at `https://api.astroway.info/v1`, and every endpoint is also a tool on a hosted MCP server. Results are computed, not written: never add a position, aspect, date or score that a call did not return.

Three rules that prevent most failures:

1. **Never invent a path, a field or a tool name.** Look it up with `find_endpoint` on the keyless docs server `https://mcp.astroway.info/mcp/docs`, or in `https://api.astroway.info/v1/openapi.json`. If neither has it, tell the user the API has no such calculation.
2. **Ask for missing birth data.** Date, time and place. Do not guess a time and do not compute a UTC offset yourself (section 4).
3. **Know the price first.** Every tool states `[Cost: N credits]`. Ask before a finished report or any plan of calls totalling more than a few hundred credits (section 5).

## 1. Pick the path

| Situation | Use |
|---|---|
| Answering the user in this conversation | The MCP tools of one subject server (section 3) |
| Writing application code | REST with `X-Api-Key` from an environment variable (section 7), or an official SDK |
| Proving the response shape before any key exists | The keyless endpoints: `POST /v1/public/chart`, `POST /v1/public/synastry`, `POST /v1/public/human-design`, `GET /v1/public/horoscope/daily?sign=aries`, `GET /v1/public/moon-phase`, `GET /v1/reference/signs` |
| Looking something up | The docs server: `find_endpoint` (method, URL, price, required fields), `search_docs`, `read_doc`. No key, no credits |

## 2. Which endpoint answers which question

Prices as the tools state them on 2026-11-07. The full list, every operation with its price, is in `https://api.astroway.info/llms-full.txt`.

| Question | Endpoint | Server | Credits |
|---|---|---|---|
| Natal chart: planets, houses, aspects | `POST /v1/chart` | core | 20 |
| Positions only, no houses | `POST /v1/planets` | core | 10 |
| Chart image | `POST /v1/render/wheel-western` (also accepts a ready `/v1/chart` result) | core | 10 |
| Sign horoscope, no birth data | `POST /v1/horoscope/daily` (also weekly, monthly, yearly) | core | 50 |
| Moon phase, retrogrades, eclipses | `POST /v1/moon-phase`, `/v1/retrograde-periods`, `/v1/eclipses` | core | 10, 50, 100 |
| Where to live: astrocartography, relocation | `POST /v1/acg`, `/v1/relocation` | core | 50 |
| Two people: aspects between charts | `POST /v1/synastry` | relationships | 50 |
| Two people: one score for a dating app | `POST /v1/match/score` | relationships | 50 |
| Relationship chart | `POST /v1/composite` | relationships | 50 |
| What is happening now or on a date | `POST /v1/transits` | forecast | 50 |
| Transits over a period | `POST /v1/transit-calendar` | forecast | 100 |
| Year ahead | `POST /v1/solar-return`, `/v1/progressions` | forecast | 50 |
| Vedic chart | `POST /v1/vedic/kundli`, vargas under `/v1/vedic/varga/D1` ... `D60` | vedic | 50, 20 |
| Panchang for a day and place | `POST /v1/vedic/panchang/full` | vedic | 20 |
| Dasha periods | `POST /v1/vedic/dashas/vimshottari/maha` (and antar, pratyantar) | vedic | 20 |
| Kundli matching | `POST /v1/vedic/compatibility/ashtakoot` | vedic | 50 |
| Vastu for a floor plan | `POST /v1/vastu/plan` | vedic | 20 |
| BaZi, Zi Wei, flying stars | `POST /v1/bazi/chart`, `/v1/ziwei/chart`, `/v1/chinese/feng-shui/flying-star` | chinese | 20, 20, 10 |
| Human Design bodygraph | `POST /v1/human-design` | human-design | 50 |
| Horary question | `POST /v1/horary` | hellenistic | 20 |
| Tarot draw | `POST /v1/tarot/rider-waite/draw/three-card` (also Marseille, Lenormand) | tarot | 10 |
| Numerology | `POST /v1/numerology/pythagorean/chart` (also chaldean, kabbalistic, vedic), `/v1/destiny-matrix/ladini` | numerology | 20 |
| I Ching | `POST /v1/iching` | divination | 10 |
| Written interpretation of a chart | `POST /v1/interpret/natal` (also synastry, transits) | platform | 100 |
| Finished PDF report | `POST /v1/reports/natal` and the other report types | reports | 5,000 and up |

When two endpoints seem to fit, ask the user what the result is for rather than calling both.

## 3. Connect the MCP server

| URL | Key | Use |
|---|---|---|
| `https://mcp.astroway.info/mcp/docs` | none | Look things up. Spends nothing |
| `https://mcp.astroway.info/mcp/<subject>` | yes | Calculate. Subjects: `core`, `vedic`, `chinese`, `hellenistic`, `tarot`, `numerology`, `human-design`, `forecast`, `relationships`, `psychological`, `divination`, `applied`, `reports`, `platform` |
| `https://mcp.astroway.info/mcp` | yes | The whole catalogue, several MB of tool list. Most clients read it slowly or cut it off; prefer a subject |

The live list with descriptions: `GET https://mcp.astroway.info/domains` (no key). Every subject server also carries `astroway_account_status` (balance) and `astroway_cost_estimate` (prices a plan of calls without running it).

Auth on the hosted servers: `Authorization: Bearer <key>`. Keys are `aw_live_...` or `aw_test_...` (sandbox, spends nothing). Free key with 10,000 credits a month, no card: https://api.astroway.info/dashboard/sign-up.

- Claude Code: `claude mcp add --transport http astroway https://mcp.astroway.info/mcp/core --header "Authorization: Bearer $ASTROWAY_API_KEY"`
- Cursor, `.cursor/mcp.json`: `{"mcpServers":{"astroway":{"url":"https://mcp.astroway.info/mcp/core","headers":{"Authorization":"Bearer ${env:ASTROWAY_API_KEY}"}}}}`
- Clients without HTTP transport: `npx -y @astroway/mcp` with `ASTROWAY_API_KEY` in the environment.
- claude.ai: Settings, Connectors, add `https://mcp.astroway.info/mcp`, choose "Sign in when needed". It signs in to the AstroWay account instead of using a key.
- Other clients: https://api.astroway.info/en/agent-setup/

Never write a key into a file that gets committed. Read it from an environment variable.

## 4. Birth data: ask, do not guess

- A natal chart needs date, **time** and **place**. If any is missing, ask before calling. A guessed time moves the houses and the Ascendant, and can change the Moon sign, without any error.
- If the user does not know the time, say what is lost (houses, Ascendant, possibly the Moon sign) and send `timeUnknown: true` instead of `time`. Only `/v1/chart` and `/v1/synastry` accept it; others answer `400 TIME_UNKNOWN_UNSUPPORTED`, which means: ask for the time.
- Convert the place to `latitude` and `longitude` yourself. The API does not geocode, and `city` is only a label.
- Send `timezone` as an IANA name (`Europe/Kyiv`, `America/New_York`), or `"auto"` to look it up from the coordinates. The server applies the offset in force on that date, summer time included. Do not compute an offset yourself: Kyiv in May 1990 was UTC+4, not +2 or +3. Abbreviations (`EST`) and text offsets (`+03:00`) are refused.
- Send `timezoneOffset` (hours, `5.5`) only when the clock offset is actually known, for example from a birth certificate. When both arrive, `timezone` wins.
- **Sending neither means UTC**, silently. The response says `timezoneOffset: 0` and looks fine. Always send one.
- Field names are strict: `lat`, `lng`, `lon` and `tz` are `400 INVALID_FIELD`, and the error names the right field. Dates are `YYYY-MM-DD`; times `HH:mm` or `HH:mm:ss`, 24-hour local time.
- Two people go in as `chart1` and `chart2`, each with its own date, time, place and timezone.

## 5. Cost awareness

- Every tool description ends with `[Cost: N credits (Tier x)]`. REST responses carry `X-Credits-Used`, `X-Credits-Remaining`, `X-Credits-Limit` and `X-Credits-Reset`.
- Typical prices: a natal chart 20, planets or moon phase 10, a sign horoscope 50, an interpretation 100, finished PDF reports thousands (most 5,000).
- `astroway_cost_estimate` prices a list of endpoints without running them; `find_endpoint` gives the same prices with no key.
- **Ask the user before** anything marked Heavy or Premium, any report, or any plan above a few hundred credits. Say the number.
- An identical request within 5 minutes is answered from cache with `X-Cache: HIT` at 0 credits. Still, do not call the same tool twice with the same input.
- A sandbox key (`aw_test_`) spends nothing on calculations, but AI interpretation, reports and rendering answer `402 SANDBOX_ENDPOINT_UNAVAILABLE` on it and need a live key.
- On Free, the month stops at 10,000 credits (`429 CREDITS_EXHAUSTED`). Do not retry that; tell the user.

## 6. Ask for less

Every tool and every REST endpoint takes `fields` (comma-separated dotted paths relative to `data`) and `precision` (decimal places). A full natal chart is 70 to 90 KB; `fields: "planets.name,planets.longitude,planets.isRetrograde,houses.ascendant,houses.mc"` with `precision: 2` is about 1 KB at the same price. Unmatched paths come back in `_fields_unmatched`; if nothing matched it is `400 FIELDS_UNKNOWN`.

Reading a chart: sign index is `floor(longitude / 30)` (0 Aries ... 11 Pisces), degree in sign is `longitude mod 30`. Placidus is the default house system, Lahiri the default ayanamsa for sidereal work.

## 7. Writing code against the REST API

- Base URL `https://api.astroway.info/v1`, header `X-Api-Key`, JSON body on POST. The full playbook with runnable curl, TypeScript and Python: `https://api.astroway.info/AGENTS.md`.
- Success is `{"ok": true, "data": {...}}`; an error is `{"ok": false, "error": {"code", "message", "details"?, "docs"?}}`. Branch on `error.code`, show the user `error.message`. Error reference: `references/errors-and-limits.md`.
- **The key stays on the server.** An ordinary key answers `403 ORIGIN_FORBIDDEN` to any request with an `Origin` or `Referer`, so a fetch from a page fails by design. Put the call in a server route and never expose the key under a public prefix (`NEXT_PUBLIC_`, `VITE_`). If the product truly needs browser calls, the owner creates a publishable `pk_` key bound to their domains, a credit budget and an endpoint scope: https://api.astroway.info/en/authentication/.
- Rate limit the public page in front of the key: every visitor spends the owner's credits. Store a computed natal chart; for a fixed birth it never changes.
- Retry network errors and `5xx` with backoff, and `429 RATE_LIMIT` after `Retry-After`. Do not retry other `4xx`.
- A server using the keyless endpoints should send `X-AstroWay-Site-URL: https://its-domain/`: 300 calls an hour per site instead of 30 per IP.
- SDKs: `npm install @astroway/sdk`, `pip install astroway`, `composer require astroway/sdk`, `go get github.com/astroway/astroway-go`, `dotnet add package Astroway.Sdk`.
- Function definitions for non-MCP agents: `GET /v1/agent/tools?format=openai` (or `anthropic`), free.
- Working examples: https://github.com/astroway/starter-astrology-chat (MCP chatbot), https://github.com/astroway/starter-human-design and https://github.com/astroway/starter-bazi (REST).

## 8. Writing the answer

- Use only values from results in this conversation.
- Say which house system, zodiac or school was used when it matters.
- Astrology output is for reflection and entertainment. Do not present it as medical, legal or financial advice, and keep any `disclaimer` field a response carries.
