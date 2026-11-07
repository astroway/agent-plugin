# AstroWay errors and limits

Read when a call failed or when code needs to handle failures. The live reference with every code: https://api.astroway.info/en/errors/.

## Error codes worth branching on

| Status | Code | What to do |
|---|---|---|
| 400 | `INVALID_FIELD` | Rename the field to `details[].expected` (`lat` to `latitude`, `tz` to `timezone` or `timezoneOffset`). Also an unresolvable `timezone`: abbreviation, unknown name, text offset, or `auto` without coordinates |
| 400 | `INVALID_INPUT`, `MISSING_FIELDS`, `BAD_JSON` | Fix the body; `details` lists `{ path, message }` |
| 400 | `TIME_UNKNOWN_UNSUPPORTED` | This endpoint needs a birth time. Ask for it |
| 400 | `FIELDS_UNKNOWN` | None of the `fields` paths matched. Call once without `fields` to see the shape |
| 401 | `MISSING_API_KEY`, `INVALID_API_KEY` | The key is missing, wrong or revoked |
| 402 | `SANDBOX_ENDPOINT_UNAVAILABLE` | AI, reports and rendering need a live key |
| 402 | `PLAN_UPGRADE_REQUIRED`, `PLAN_PACK_MISMATCH` | Needs a higher plan or an add-on pack. Tell the user; do not retry |
| 402 | `SUBSCRIPTION_EXPIRED` | Paid period over; `details.renew_url` |
| 403 | `ORIGIN_FORBIDDEN` | A server key was used from a browser. Move the call to a server route |
| 403 | `ORIGIN_NOT_ALLOWED`, `ORIGIN_REQUIRED` | A `pk_` browser key used from a domain not on its list, or with no origin |
| 403 | `ENDPOINT_NOT_IN_SCOPE` | The key is limited to other endpoints; `details` names them |
| 422 | `CALCULATION_ERROR` | Valid input the engine cannot compute, for example Placidus near the poles. Offer Whole Sign or Equal |
| 429 | `RATE_LIMIT`, `PUBLIC_RATE_LIMIT` | Wait `Retry-After` seconds (or until `X-RateLimit-Reset`), then retry |
| 429 | `CREDITS_EXHAUSTED`, `SPEND_CAP_REACHED`, `KEY_BUDGET_EXHAUSTED` | Out of credits for the cycle, the account cap, or the key's own budget. Do not retry |
| 5xx | `INTERNAL_ERROR`, `AI_UNAVAILABLE`, `QUEUE_UNAVAILABLE` | Retry with exponential backoff; quote the `X-Request-Id` response header to support |

## Limits

- Keyless endpoints: 30 calls an hour per IP. With `X-AstroWay-Site-URL: https://your-domain/`, 300 an hour per site under a 600 an hour ceiling per IP. `/v1/public/synastry` counts 3.
- Keyed calls: per minute by plan (Free 10, Indie 30, Starter 120, Pro 400, Business 1,000). The current table: https://api.astroway.info/en/rate-limits-credits/.
- Headers on every response: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (Unix time), and on keyless calls `X-RateLimit-Scope` (`ip` or `site`). Read them rather than hardcoding.
- Credits: `X-Credits-Used`, `X-Credits-Remaining`, `X-Credits-Limit`, `X-Credits-Reset`. Prices per endpoint: https://api.astroway.info/en/credits/.
