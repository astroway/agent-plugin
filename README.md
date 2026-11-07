# AstroWay plugin for Claude Code and Cursor

Teaches your coding agent to build on the [AstroWay astrology API](https://api.astroway.info) the way it is meant to be used, and connects it to the hosted MCP servers at `mcp.astroway.info`.

One install gives the agent:

- **A skill** (`skills/astroway/SKILL.md`) with the rules that matter: which of the 14 subject servers to use for which question, ask for birth date, time and place instead of guessing them, send time zones as IANA names, ask for small responses with `fields` and `precision`, read the price on every tool and ask before expensive calls, never invent an endpoint.
- **The docs server**, `https://mcp.astroway.info/mcp/docs`, with no key: `find_endpoint` returns the method, URL, credit cost and required fields of any of the API's operations, so the agent looks things up instead of guessing.
- **One subject server with your key**, for real calculations: natal charts, transits, Vedic, BaZi, Human Design, tarot, numerology and the rest.

## Install in Claude Code

```
/plugin marketplace add astroway/agent-plugin
/plugin install astroway@astroway
```

Claude Code asks for two settings when the plugin is enabled:

| Setting | What to enter |
|---|---|
| AstroWay API key | An `aw_live_...` or `aw_test_...` key. Stored in your system keychain, not in `settings.json`. |
| Subject server | `core` by default. Pick `vedic`, `chinese`, `tarot`, `human-design`, `forecast`, `relationships` and so on for other work. |

Change them later with `/config` or `claude plugin configure astroway`. Without a key the docs server and the skill still work; the subject server reports an authorisation error until a key is set.

## Install in Cursor

Clone the plugin into Cursor's local plugin folder and reload the window (**Developer: Reload Window**):

```
git clone https://github.com/astroway/agent-plugin ~/.cursor/plugins/local/astroway
```

Cursor loads the skill, the rule in `rules/astroway.mdc` and the two servers from `mcp.json`. The keyed server reads the `ASTROWAY_API_KEY` plugin variable, which you set under **Plugins**, **Configure**.

If you would rather wire the servers yourself, put this in `~/.cursor/mcp.json` or the project's `.cursor/mcp.json` and keep the key in your environment:

```json
{
  "mcpServers": {
    "astroway-docs": { "url": "https://mcp.astroway.info/mcp/docs" },
    "astroway": {
      "url": "https://mcp.astroway.info/mcp/core",
      "headers": { "Authorization": "Bearer ${env:ASTROWAY_API_KEY}" }
    }
  }
}
```

## Get a key

[api.astroway.info/dashboard/sign-up](https://api.astroway.info/dashboard/sign-up). The free plan is 10,000 credits a month and needs no card. A natal chart is 20 credits; every tool states its own price. Use an `aw_test_*` sandbox key while you build.

## The subject servers

| Server | Covers |
|---|---|
| `/mcp/core` | Natal charts, planets, houses, aspects, dignities, chart images, the calendar |
| `/mcp/vedic` | Vargas, dashas, panchang, shadbala, yogas, doshas, Jaimini, KP, Vastu |
| `/mcp/chinese` | BaZi, Zi Wei Dou Shu, Chinese calendar, feng shui |
| `/mcp/hellenistic` | Lots, sect, zodiacal releasing, profections, horary |
| `/mcp/tarot` | Rider-Waite-Smith, Marseille, Lenormand |
| `/mcp/numerology` | Pythagorean, Chaldean, Kabbalistic |
| `/mcp/human-design` | Bodygraph, channels, incarnation cross, penta |
| `/mcp/forecast` | Transits, progressions, directions, returns |
| `/mcp/relationships` | Synastry, composite, compatibility |
| `/mcp/psychological` | Archetypal and psychological readings |
| `/mcp/divination` | Geomancy, I Ching, runes, Mayan calendar, Kabbalah |
| `/mcp/applied` | Business, financial and wellness astrology |
| `/mcp/reports` | Finished PDF reports |
| `/mcp/platform` | Webhooks, streaming, AI interpretation |

All of them live under `https://mcp.astroway.info`. The current list is always at [mcp.astroway.info/domains](https://mcp.astroway.info/domains). `https://mcp.astroway.info/mcp` serves the whole catalogue at once, which is more than most agents can read in one go.

## What is in this repository

```
.claude-plugin/plugin.json       Claude Code manifest, with the key and subject settings
.claude-plugin/marketplace.json  lets `/plugin marketplace add astroway/agent-plugin` find it
.mcp.json                        servers for Claude Code
.cursor-plugin/plugin.json       Cursor manifest
mcp.json                         servers for Cursor
rules/astroway.mdc               the short version, as a Cursor rule
skills/astroway/SKILL.md         the guidance itself, read by both
```

There is no code to run and nothing to build. The plugin contains no key.

## Links

- [Agent setup for every client](https://api.astroway.info/en/agent-setup/)
- [Credits and prices](https://api.astroway.info/en/credits/)
- [Astrology chat starter](https://github.com/astroway/starter-astrology-chat): the same servers inside a web app
- [Privacy](https://api.astroway.info/en/privacy/) and [terms](https://api.astroway.info/en/terms/)

## Licence

MIT.
