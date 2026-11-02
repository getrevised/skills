# Revised tools, fields and limits

Reference for the Revised MCP server and REST API. One account sits behind both, and one per-minute rate limit is shared between them. When this file and a live `whoami` disagree, trust `whoami`.

## Interfaces

**MCP**: `https://getrevised.com/api/mcp`, Streamable HTTP, stateless. Nine tools: `whoami`, `list_categories`, `search_domains`, `get_domain`, `reveal_domain`, `hold_domain`, `list_holds`, `extend_hold`, `release_hold`. The client signs in through the browser (OAuth) and holds its own token; there is no key to paste. An API key (`Authorization: Bearer rvd_...`) also works, for scripts and containers.

**REST**: `https://getrevised.com/api/v1`, docs at https://getrevised.com/docs/api, OpenAPI at `/api/v1/openapi.json`. API key only.

```bash
curl "https://getrevised.com/api/v1/domains?limit=5&rsMin=60" \
  -H "Authorization: Bearer $REVISED_API_KEY"
```

Use MCP for shortlisting a handful of candidates. Use REST for a pass over the whole directory (`cursor` paging) or for change polling.

## whoami first

`whoami` (MCP) or `GET /api/v1/me` (REST) is free and returns what everything else is bounded by:

- `plan`: `free`, `starter` or `business`.
- `auth`: `oauth` or `api_key`.
- `reveals`: `limit`, `remaining`, `used_this_month`, `resets_at`, `unlimited`, `featured_credits`, `rollover`. `unlimited: true` means no ceiling and nothing spends. `limit: null` means the same thing; it is not zero. Free has `limit: 0` (no reveals).
- `rate_limit`: `per_minute`, `remaining_this_minute`, `resets_at` (fixed window, not sliding).
- `holds`: `limit` (0 means the plan cannot hold) and `window_hours`.
- `sync_feed`: whether the REST `since` feed is included.

Report the plan and reveal position to the user before anything that could spend.

## Plans

| | Free | Starter | Business |
|---|---|---|---|
| Price | free | see https://getrevised.com/pricing | see https://getrevised.com/pricing |
| Reveals | none | no ceiling | no ceiling |
| Holds at once | none | 5 | 20 |
| Hold window | none | 48 h | 168 h |
| Requests per minute | 20 | 60 | 120 |
| New listings visible | after 7 days | immediately | immediately |
| `since` change feed (REST) | no | no | yes |

Open-tier names are visible with their names on every plan. Read live values from `whoami`.

## list_categories

Call once before the first search. Free and cached. Returns `categories` with `code`, `label`, `listings` and `open` (how many are open tier). Codes are Revised's own; "marketing" and "software" are not codes. `search_domains` refuses an unknown code (`INVALID_ARGUMENT`) rather than returning an empty result. Some listings have no category and are in none of the counts. There is no REST equivalent.

## search_domains

Free, unlimited within the rate limit, never spends a reveal. Arguments:

| Argument | Meaning |
|---|---|
| `category` | Code(s) from `list_categories`, comma-separated. |
| `tld` | Exact TLD without the dot: `com`, `org`, `co.uk`, `com.au`. Comma-separate for several. |
| `tier` | `open`, `regular`, `featured`. |
| `spam` | Band ceiling: `low` or `moderate`. |
| `min_revised_score` | 0-100, against the low end of the published range. |
| `min_age_years` | Excludes rows where age is unknown. |
| `min_citability` | How often sources AI assistants cite link to this domain. Excludes null rows. |
| `min_linked_from` | How many tracked sites (Wikipedia, Hacker News, universities, major publishers) link in. |
| `recommended` | Revised's heuristic: any category (the literal `other` counts), at least one recognisable linking site, a score floor. |
| `query` | Free text over masked hint, tags, category and blurb. Does not search real domain names. Weak; prefer filters. |
| `sort` | `featured` (default), `rd`, `linkedfrom`, `rs`, `age`, `citability`, `value`/`price-desc`, `price-asc`, `newest`, and others the tool schema lists. Null rows sort last. |
| `fields` | `summary` (default) or `full`. Summary is roughly a quarter to a third the size. |
| `limit` | Clamped, not refused. Maximum 50. |

Rules:

- A value the server cannot apply is **refused** with `INVALID_ARGUMENT` naming the argument and legal values. A call that returned rows had every filter applied.
- Read `total` (matched) as well as `count` (returned). `truncated` is `count < total`. **There is no paging**: narrow filters or change sort.
- The tags and blurb the query searches are not returned on a row.

## Reading a listing

- **Bands**: `"100-500"` is a range, not a rounded figure. Revised Score is published as a range. Do not average bands or show a midpoint as a measurement. Shapes: `"10-50"`, `"<100"`, `"250+"`, `"1K-5K"`.
- **`name_disclosed`**: whether `domain` is a real name. When false, `domain` is a partial mask such as `g▓▓▓▓▓▓farms`: show it to a person, never give it to a registrar. `mask_hint` is on every row and is not the test.
- **Always present**: referring domains, backlinks, Revised Score range, snapshots, tier, TLD. **Present where known, else `null`**: age, category, Agent Citability. `metrics_missing` lists each row's gaps.
- **Null is absent data, never zero.** Minimum filters exclude null rows.
- **`linked_from`** names tracked sites that link in (`linked_from_count` is how many). Most human-legible signal on a row; can be empty. (`linked_by` and `cited_by*` are older aliases.)
- **`agent_citability`**: preview score plus the AI-cited sources that link to it. `null` means no row in the current index, not zero. Full rows carry `basis`: `seed_links` means AI-cited sources link to the domain and are listed in `sources`; `link_graph` means none do, `sources` is genuinely empty, and the score is the domain's link-graph position alone (it cannot exceed 70). Weigh a `link_graph` score below a `seed_links` one.
- **`snapshots`**: banded count of archived captures; evidence the domain was a real site. A floor, not a guarantee.
- **Traffic**: `visitor_traffic_band` (`traffic` on a summary row) is a coarse band of how popular the site was with real visitors while live, or null if it was never in the top 1M.
- **`availability_checked_at`**: when the name was last confirmed unregistered. A stamp, not a live check. A first reveal re-checks, and holding a name whose stamp is over an hour old re-checks. Treat a stale stamp as needing your own registrar check.
- **`hold_state: "other"`** with `hold_expires_at`: another account holds it, and the name is masked for everyone, including unlimited plans.

## get_domain

Takes `id` (8 characters, case-sensitive; store verbatim) or `name`. `name` only resolves for names you are already entitled to: an open-tier listing, one you revealed, or, on a plan with no reveal ceiling, any listing. Any other name answers not-found whether or not it is listed. Treat that as "no answer", never "not listed".

## Tiers and republication

| Tier | Public on getrevised.com | Cost to reveal | May repeat outside the conversation |
|---|---|---|---|
| `open` | Yes | Free | Yes |
| `regular` | Masked | 1 credit if the plan has a ceiling | No |
| `featured` | Masked, curated | `reveals.featured_credits` | No |

Branch on `tier`, never on the boolean `featured` field and never on whether `domain` is non-null. A name you can see is not a name you may publish. Before writing any name into a file, repo, post or message the user will share, check `tier == "open"`.

## reveal_domain

The only call that spends, and the grant is permanent. Repeats are free. The result carries `credits_charged`, `was_free` and, when free, `reason`:

- `open_tier`: already public.
- `already_revealed`: bought earlier.
- `unlimited_plan`: plan has no ceiling (a no-op that records the reveal).
- `refunded`: the first-reveal availability re-check found the name registered since the last check, so the credit was returned. Tell the user.

Free plan: reveals are refused (`QUOTA_EXHAUSTED`) except on open listings. Ask the user before any reveal that would spend a credit.

## Holds

`hold_domain` reserves a listing inside Revised for the plan's window so no other account can reveal or take it. It costs no reveal. On Starter and Business (no reveal ceiling) a hold needs no reveal on record. On a plan with a ceiling it does, and `hold_domain` answers `REVEAL_REQUIRED` until you call `reveal_domain`.

- Free accounts cannot hold.
- A hold is not registration. Nothing outside Revised is reserved.
- `extend_hold` pushes `expires_at` back by `extend_hours`, once per hold (`can_extend`). Never release and re-hold: a 30-day own-cooldown starts when a hold ends, while anyone else may take the listing at once.
- `release_hold` with `outcome: "registered"` triggers a fresh availability check and retires the listing.
- `list_holds` returns active holds (with their domains) and the most recent ended ones.

## Change polling (Business, REST only)

`GET /api/v1/domains?since=<ISO instant>` returns changes including tombstones (`status: "delisted"`) for listings that left. Pass the `synced_at` from the first page of the previous run as the next `since`; expect repeats; upsert on `id` by `updated_at`. Free and Starter get `402 upgrade_required`. Without it, `cursor` paging reaches every row on any plan.

## Rate limits

Fixed one-minute windows: Free 20, Starter 60, Business 120 (per key on REST; everything connected over OAuth shares one window per account). MCP and REST calls share the window. Over the limit: `rate_limited` / HTTP 429 with a retry delay. Wait it out.

## Errors

| Code | Meaning | Do |
|---|---|---|
| `INVALID_ARGUMENT` | Argument outside what the tool accepts. | Fix and call again; nothing was filtered. |
| `INVALID_REQUEST` | Required argument missing. | Supply it. |
| `QUOTA_EXHAUSTED` | Reveals not available or used up. Carries `resets_at`, `credits_needed`, `credits_remaining`. | Stop revealing. Work from the free metrics. Tell the user about the upgrade or reset date. Do not retry. |
| `FEATURED_REQUIRES_PRO` | Free plan against a curated listing (the name is historical; any paid plan can reveal). | Search with `tier: "open"` or say an upgrade is needed. |
| `HELD_BY_OTHER` | Another account holds it. | Move on or return after `hold_expires_at`. |
| `DOMAIN_TAKEN` | A hold's re-check found it registered. | Tell the user; do not retry. |
| `REVEAL_REQUIRED` | Hold without a reveal on record. | Reveal first. |
| `HOLDS_REQUIRE_PRO` | Plan allows no holds (name is historical). | Say so; do not retry. |
| `HOLD_CAP_REACHED` | All slots used. Carries `limit`. | Release one you are done with. |
| `HOLD_COOLDOWN` | Your own hold ended in the last 30 days. Carries `available_at`. | Wait. |
| `ALREADY_EXTENDED` | One extension per hold. | Nothing to do. |
| `HOLD_OVER` | Hold already ended. | Cannot extend or release. |
| `HOLD_NOT_FOUND` | No such hold of yours. | Re-read `list_holds`. |
| `NOT_FOUND` | No such listing, left the directory, or newer than your plan's visibility window. | Re-search. |
| `rate_limited`, `insufficient_scope`, `invalid_token` | Transport-level; arrive as HTTP status, no tool result. | Wait; or reconnect and accept the consent screen. |

## Things not to do

- Do not republish a non-`open` name.
- Do not present a listing as vetted. Revised filters spam and link-scheme profiles before listing; that is a floor.
- Do not burn the allowance exploring. Search is free.
- Do not claim a hold secures the name.
