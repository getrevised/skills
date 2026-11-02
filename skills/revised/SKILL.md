---
name: revised
description: Find, vet, register and redirect expired (dropped) domain names that still carry backlinks, using the Revised MCP server (https://getrevised.com/api/mcp) or the Revised REST API. Use when a user wants an aged or previously-linked domain relevant to their own site or business, wants a specific expired domain's history and backlinks assessed, needs help registering one (Cloudflare, Vercel or Porkbun, which agents can drive), or wants a 301/410 redirect map from the old domain's linked URLs to their site. Covers connecting the MCP server, topical matching, vetting, reveal and hold rules, registration, and expired-domain-abuse policy limits.
---

# Revised

Revised is a directory of expired and dropped domain names that still carry backlinks. Every listing is a name that was registered, lapsed, and is available to register again. Revised sells nothing: it publishes what it knows about a name, and the user registers it at a registrar.

This skill works through the **Revised MCP server** at `https://getrevised.com/api/mcp`. Its tools are `whoami`, `list_categories`, `search_domains`, `get_domain`, `reveal_domain`, `hold_domain`, `list_holds`, `extend_hold` and `release_hold`. If they are not in your tool list, step 1 connects them. Do that before anything else.

A listing is evidence about a name, never a recommendation. Your job is to decide, with the user, whether a name is genuinely relevant to their site, and then to plan its use properly.

## Policy guardrail (read first)

Google's spam policies name **expired domain abuse**: buying an expired domain and using it mainly to manipulate search rankings with low-value or unrelated content. Legitimate uses are a domain topically related to the user's business, relevant 301s from old pages that have a real counterpart on the user's site, or rebuilding genuinely useful content on the name.

Do these things:

- Tie every candidate to the user's actual site or business. No topical match means no candidate, however strong the metrics.
- **Refuse, or flag and stop, any plan** that redirects an unrelated domain to a site purely for link equity, builds a private blog network (PBN), spins up thin content on the old name, or presents the new site as the previous owner or brand.
- Never promise ranking, traffic or penalty immunity. Nobody can.

Details and the Google link: `references/policy.md`. Read it before step 5 and again before step 8.

## Workflow

Work in order. Report to the user at the end of each step that produces a decision.

### 1. Connect the Revised MCP server

Check your tool list for the Revised tools above. If they are there, go to step 2. If not, add the server for the client you are running in:

- **Claude Code**: `claude mcp add --transport http revised https://getrevised.com/api/mcp`, then the user runs `/mcp`, picks **revised**, then **Authenticate** and signs in through the browser.
- **Codex**: `codex mcp add revised --url https://getrevised.com/api/mcp`, then `codex mcp login revised`.
- **Cursor**: add `"revised": {"url": "https://getrevised.com/api/mcp"}` under `mcpServers` in `~/.cursor/mcp.json`, then **Needs login** under Settings → MCP.
- **VS Code**: add `"revised": {"type": "http", "url": "https://getrevised.com/api/mcp"}` under `servers` in `.vscode/mcp.json`.
- **Claude.ai, Claude Desktop, ChatGPT, opencode and others**: https://getrevised.com/docs/agent-setup.
- **No browser (CI, container)**: REST at `https://getrevised.com/api/v1` with an API key (`rvd_...`) from https://getrevised.com/account, sent as `Authorization: Bearer`. Never write a key into a checked-in file.

Sign-in is OAuth and needs a human at a browser. You cannot finish it yourself: add the server, then ask the user to sign in, and wait. `claude mcp get revised` can show "Connected" before anyone has signed in; that is not signed in. A server added mid-session may need `/mcp` or a restart before its tools load.

The user needs a Revised account (https://getrevised.com). Creating one is a human step.

Tool and field semantics: `references/mcp-and-api.md`.

### 2. Account and plan

Call `whoami` (MCP) or `GET /api/v1/me` (REST) and trust its numbers over this list. Plans, as published (prices: https://getrevised.com/pricing):

- **Free**: browse and search everything, names shown only for `open`-tier listings, no reveals, no holds, new listings appear after 7 days.
- **Starter**: every name shown, no reveal limit, 5 holds, new listings immediately.
- **Business**: everything in Starter plus 20 holds, 120 requests a minute and the REST `since` change feed (for pipelines).

Do not quote prices or features from memory; send the user to the pricing page. Tell the user their plan. If they are on Free and the shortlist needs masked names, say a paid plan is needed; do not try to work around the mask.

### 3. Brief from the user's own site

Do not ask the user to describe their business from scratch. Read their site first: homepage, 3-5 key pages, sitemap.xml, and any about/pricing page. Extract:

- **Topic**: what they sell or publish, in a phrase.
- **Audience** and **language**.
- **Geography**: country or region served. This drives TLD choice (`com.au` for Australian local, `co.uk` for UK, `com` otherwise).
- **Page inventory**: the URLs worth receiving redirects later (step 8).

Call `list_categories` once. Revised category codes are its own, not English words; map the topic onto one or two codes you read off the list. If nothing fits, do not guess: search structurally on other filters.

Show the user the brief (topic, audience, language, geography, chosen categories and TLDs) and get a yes before searching widely.

### 4. Search and shortlist

Search is free and never spends a reveal. Filter structurally; free text is weak and does not match real domain names.

1. `search_domains` with `category`, `tld`, and metric floors (`min_revised_score` about 55 to start, `min_age_years`, `min_linked_from`, `min_citability` as relevant). Null metrics are dropped by floors, so a mild floor can empty the result: check `total`.
2. Use `fields: "summary"` first. Rank on referring domains, Revised Score, age, citability and linked-from count.
3. Read `total`, not just `count`. There is no paging on MCP: narrow filters or change `sort`.
4. Take full rows (`get_domain`) only for the 5-10 survivors.

Metrics are bands. A band is a range, not a measurement; do not average bands or present a midpoint as a figure.

### 5. Vet each candidate against the user's site

For each survivor, apply the checks in `references/vetting.md`. In short:

1. **Topical match** between what the old site was (Wayback history) and the user's site. This is the gate. A listing's category is a starting point, not a verdict: read the archive.
2. **Linkers**: read `linked_from` and `agent_citability` (`basis: seed_links` lists AI-cited sources; `link_graph` is graph position only, weaker). Are the linking sites relevant, real and editorial, or incidental?
3. **Red flags**: spam, adult, pharma, gambling or casino history; long parked stretches; scrape/doorway content; a different language or country than the user's; a trademark in the name.
4. **Freshness**: check `availability_checked_at`; anything stale needs a registrar check.

Apply the reject rules in `references/vetting.md` strictly. Present each survivor as: name, why it matches, what is linking, risks. State what you could not verify. Do not call anything "vetted" or "safe".

### 6. Reveal and hold

- `open`-tier names are already public and cost nothing to reveal.
- Free plan: no reveals; only `open` names are disclosed.
- Starter and Business: names arrive disclosed; `reveal_domain` only records the reveal.
- Check `whoami` and the listing's `tier` before any `reveal_domain`. On any plan with a ceiling (rare), ask the user before spending.
- To keep a candidate reserved inside Revised while the user decides, use `hold_domain`, then `extend_hold` once if needed. `release_hold` when decided; pass `outcome: "registered"` if it was registered. A hold reserves the listing inside Revised only. It does not register the name.
- **Republication**: only `open`-tier names may be repeated outside the conversation (public repo, blog post, shared doc, newsletter). `regular` and `featured` names stay in the conversation and the user's private workspace.

### 7. Register

Offer the user these registrars, all of which an agent can drive end to end (check, quote, buy, DNS):

- **Cloudflare Registrar**: at-cost pricing, through the Cloudflare MCP server (`https://mcp.cloudflare.com/mcp`) or the Registrar API. Best when the redirects will run on Cloudflare.
- **Vercel**: the `vercel domains` CLI or the Vercel MCP server. Best when the user's site is on Vercel.
- **Porkbun**: the Porkbun MCP server (`https://mcp.porkbun.com/mcp`) or API v3, with a free dry run. Wide TLD range.

Any other registrar works too, done by the user by hand. Each of the three covers only some country-code TLDs (`com.au`, for example, needs a local registrar and has eligibility rules). Check the TLD before recommending one.

Whichever you use:

1. Re-check availability **and price** at the registrar right before buying. The `availability_checked_at` stamp is not a live check.
2. **Ask the user to confirm the exact name, term, total price and auto-renew setting** before any call that charges. Registrations are generally non-refundable. Never enter card details or ask for a password; the user's own signed-in account pays.
3. After registration, set up DNS and HTTPS, then call `release_hold` with `outcome: "registered"` if a hold was in place.

Provider steps, tool names and DNS notes: `references/registration.md`.

### 8. Redirect plan

Before any redirect exists, build the map from the old site's real URLs:

1. List old URLs that were live, from the Wayback CDX API:
   `https://web.archive.org/cdx/search/cdx?url=<domain>/*&output=json&fl=original,statuscode&filter=statuscode:200&collapse=urlkey`
2. Keep the URLs that plausibly had inbound links (content pages, not assets, feeds, search or tracking URLs).
3. Map each to the closest genuinely relevant page on the user's site with a **301**.
4. Return **410** (Gone) for old URLs with no relevant counterpart.
5. **Never blanket-redirect everything to the homepage.** Google treats mass irrelevant redirects as soft 404s, and it is the pattern expired-domain abuse takes.
6. If more than a small share of old URLs have no honest counterpart, the domain is not a topical fit: say so and drop it.

Output a redirect map file (`redirects.csv` with `source,target,status,reason`) and an implementation snippet for the user's stack (Cloudflare Bulk Redirects, a small Worker, `vercel.json`, `_redirects`, or nginx). Templates: `references/redirects.md`.

Show the user the full map and get approval before they deploy it.

### 9. Afterwards

- Keep redirects in place for at least a year.
- Verify with `curl -sI` on a sample of source URLs: status, `Location`, no chains or loops.
- Monitor Google Search Console and Bing Webmaster Tools for crawl errors and soft 404s on the old domain.
- Never state outcomes as guaranteed. Link equity transfer and rankings are not controllable.
- Re-run the policy check (`references/policy.md`) if the plan changes, such as adding new content on the old domain.

## Rules you always follow

- Connect the Revised MCP server and call `whoami` before anything else; never guess limits.
- Search before you reveal; reveal only what the user will act on.
- Do not republish non-`open` names. Do not present a listing as vetted.
- Do not claim a hold secures the name; registration happens at a registrar.
- Never buy a domain without the user confirming the name and the price in this conversation.
- Treat `null` as absent data, not zero.
- Use `example.com` / `example.org` in any example you write; do not invent real domains.
- Stop and tell the user when a plan trips the policy guardrail, rather than quietly complying.

## References

- `references/mcp-and-api.md`: Revised MCP tools, fields, nulls, tiers, masks, errors, rate limits.
- `references/vetting.md`: topical match, linker quality, red flags, reject rules.
- `references/registration.md`: Cloudflare, Vercel and Porkbun step by step, availability recheck, DNS.
- `references/redirects.md`: CDX extraction, mapping rules, 301/410, snippets, verification.
- `references/policy.md`: expired domain abuse, legitimate uses, refusal rules.
- Docs: https://getrevised.com/docs/agent-setup
