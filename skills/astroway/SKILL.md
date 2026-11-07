---
name: astroway
description: Use when the user asks for astrology, horoscope, natal chart, synastry, transits, Vedic or KP astrology, BaZi, Zi Wei, feng shui, Human Design, tarot, numerology, I Ching or similar calculations, or is building an app, bot or site feature on the AstroWay API (api.astroway.info, mcp.astroway.info). Covers which hosted MCP server to use, the birth data to ask for, keeping responses small with fields and precision, and what calls cost.
---

# AstroWay

AstroWay is an astrology calculation API. Every endpoint is also a tool on a hosted MCP server. Results are computed, not written: never add a position, aspect, date or score that a tool did not return.

## 1. Pick the server

| URL | Key | Use it for |
|---|---|---|
| `https://mcp.astroway.info/mcp/docs` | none | Looking things up: `find_endpoint` (method, URL, credit cost, required fields for every operation), `search_docs`, `read_doc`, `list_docs`. Spends nothing. |
| `https://mcp.astroway.info/mcp/<subject>` | yes | Calculating. One subject per server. |
| `https://mcp.astroway.info/mcp` | yes | The whole catalogue. Its tool list is several MB, so most clients read it slowly or cut it off. Prefer a subject server. |

Subjects (the live list, with descriptions, is `GET https://mcp.astroway.info/domains`, no key):

| Subject | What is on it |
|---|---|
| `core` | Natal charts, planets, houses, aspects, dignities, chart images, the calendar (moon phase, retrogrades, ingresses, eclipses) |
| `vedic` | Vargas, dashas, panchang, shadbala, yogas, doshas, Jaimini, KP, Vastu |
| `chinese` | BaZi four pillars, Zi Wei Dou Shu, Chinese calendar, feng shui |
| `hellenistic` | Lots, sect, triplicity rulers, zodiacal releasing, profections, horary |
| `tarot` | Rider-Waite-Smith, Marseille, Lenormand: draws, spreads, readings |
| `numerology` | Pythagorean, Chaldean, Kabbalistic, Vedic numerology |
| `human-design` | Bodygraph, centres, channels, incarnation cross, penta, group overlay |
| `forecast` | Transits, progressions, directions, returns, cosmobiology |
| `relationships` | Synastry, composite, compatibility scores, family dynamics |
| `psychological` | Archetypal and psychological readings |
| `divination` | Geomancy, I Ching, runes, Mayan calendar, Kabbalah, palmistry |
| `applied` | Business, financial and wellness astrology |
| `reports` | Finished PDF reports (expensive, see section 4) |
| `platform` | Webhooks, streaming, AI interpretation endpoints |

Every subject server also carries `astroway_cost_estimate`.

**Setup.** Auth is `Authorization: Bearer <key>` on the hosted servers. Keys look like `aw_live_...` or `aw_test_...` (sandbox, spends nothing). Free key: https://api.astroway.info/dashboard/sign-up (10,000 credits a month, no card).

- Claude Code: `claude mcp add --transport http astroway https://mcp.astroway.info/mcp/core --header "Authorization: Bearer $ASTROWAY_API_KEY"`
- Cursor, `.cursor/mcp.json`: `{"mcpServers":{"astroway":{"url":"https://mcp.astroway.info/mcp/core","headers":{"Authorization":"Bearer ${env:ASTROWAY_API_KEY}"}}}}`
- claude.ai: Settings, Connectors, add `https://mcp.astroway.info/mcp`, choose "Sign in when needed". No key; it signs in to the AstroWay account.
- Other clients: https://api.astroway.info/en/agent-setup/

Never write a key into a file that gets committed. Read it from an environment variable.

**Never invent a tool name or endpoint path.** If nothing in the tool list fits, call `find_endpoint` on the docs server. If that finds nothing either, tell the user the API has no such calculation.

## 2. Ask for birth data, do not guess it

- A natal chart needs date, **time** and **place**. If any is missing, ask before calling. A guessed time moves the houses and the Ascendant (and can change the Moon sign) without any error.
- If the user does not know the time, say what is lost (houses, Ascendant, possibly the Moon sign) and send `timeUnknown: true` instead of `time`.
- Convert the place to `latitude` and `longitude` yourself.
- Send `timezone` as an IANA name (`Europe/Kyiv`, `America/New_York`) or `"auto"` to look it up from the coordinates. The server applies the offset that was in force on that date, summer time included. Do not compute an offset yourself: Kyiv in May 1990 was UTC+4, not +2 or +3. Send `timezoneOffset` (hours, e.g. `5.5`) only when the local clock offset is actually known, for example births before 1970.
- `timezoneOffset` defaults to 0 (UTC) when neither is sent. That is a silent wrong answer, not an error.
- Dates are `YYYY-MM-DD`, times `HH:MM` or `HH:MM:SS`, 24-hour local time.

## 3. Ask for less

Every tool, and every REST endpoint, takes two extra parameters:

- `fields`: comma-separated dotted paths to keep, relative to `data`. Example for a natal chart: `"planets.name,planets.longitude,planets.isRetrograde,houses.ascendant,houses.mc,houses.cusps"`.
- `precision`: decimal places to keep. `2` is finer than any chart is drawn.

A full natal chart is about 70 to 90 KB. With `fields` and `precision: 2` the same call is around 1 KB, at the same credit price. Paths that matched nothing come back in `_fields_unmatched`; a request where nothing matched is `400 FIELDS_UNKNOWN`. Do not repeat a call only to retry an unmatched path; read what came back.

Reading a chart: sign index is `floor(longitude / 30)` (0 Aries, 1 Taurus, ... 11 Pisces), degree in sign is `longitude mod 30`.

## 4. Know the price before you call

- Every tool description ends with a line like `[Cost: 20 credits (Tier 2)]`. Read it.
- `astroway_cost_estimate` prices a list of endpoints without running them. `find_endpoint` on the docs server gives the same prices with no key.
- Typical prices: a natal chart 20 credits, planets or moon phase 10, a daily horoscope 50. Finished PDF reports are thousands of credits each (most are 5,000).
- **Ask the user before** anything marked Heavy or Premium, any report, or any plan whose total is more than a few hundred credits.
- Do not call the same tool twice with the same input. Identical repeat requests are usually answered from cache, but do not rely on it.
- Balance: REST responses carry `X-Credits-Used`, `X-Credits-Remaining`, `X-Credits-Limit` and `X-Credits-Reset` headers, and the dashboard shows the cycle: https://api.astroway.info/dashboard/

## 5. When writing code against the REST API

- Base URL `https://api.astroway.info/v1`. Header `X-Api-Key: <key>`. JSON body on POST.
- Success is `{"ok": true, "data": {...}}`. An error is `{"ok": false, "error": {"code": "...", "message": "..."}}`. Show the API's own message; it says why.
- **No browser calls.** Responses carry no CORS headers, so a fetch from the page fails. Put the key and the call in a server route, and never expose the key under a public prefix (`NEXT_PUBLIC_`, `VITE_`).
- Rate limit a public page in front of the key: every visitor spends the owner's credits.
- SDKs: `npm install @astroway/sdk`, `pip install astroway`, `composer require astroway/sdk`, `go get github.com/astroway/astroway-go`, `dotnet add package Astroway.Sdk`.
- Spec: `https://api.astroway.info/v1/openapi.json`. Agent notes: `https://api.astroway.info/AGENTS.md`. Function definitions for non-MCP agents: `GET /v1/agent/tools?format=openai` (or `anthropic`), free.
- Working examples: https://github.com/astroway/starter-astrology-chat (MCP chatbot), https://github.com/astroway/starter-human-design and https://github.com/astroway/starter-bazi (REST).

## 6. Writing the answer

- Use only values from tool results in this conversation.
- Say which house system or school was used when it matters (Placidus is the default house system; Lahiri the default ayanamsa for sidereal).
- Astrology output is for reflection and entertainment. Do not present it as medical, legal or financial advice.
