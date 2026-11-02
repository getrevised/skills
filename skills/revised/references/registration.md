# Registering the domain

Revised sells nothing. The user pays a registrar. Cloudflare, Vercel and Porkbun let an agent run the whole check, quote and purchase flow through an MCP server, CLI or API, on the user's own signed-in account. Run the purchase call only after the user has confirmed the name, term, total price and auto-renew setting in this conversation. Never handle card numbers, registrar passwords or API secrets in chat or in files.

## Before buying (any registrar)

1. The candidate passed `vetting.md` and, if redirecting, the redirect map in `redirects.md` is drafted and approved.
2. Use only a real name from a disclosed row (`name_disclosed: true`). Never give a registrar a masked `domain` string.
3. Check availability **and price** live at the registrar. `availability_checked_at` is when Revised last looked; the name can be gone now. Search results at a registrar can be cached; use its check or quote call.
4. Expired names sometimes sit in redemption or pending delete. If the registrar says unavailable but the Revised stamp is recent, say so; the user may be racing another registrant. Do not suggest backorder services unless asked.
5. Premium or aftermarket pricing is the user's decision. Say plainly when a price is above the standard rate.
6. Registrant contact: the user's own details. Prefer WHOIS privacy where offered.
7. Confirm with the user, in one message: name, years, total price and currency, auto-renew on or off, which registrar and account. Wait for a yes.

## Choosing a registrar

| | Agent access | Pays from | Best when |
|---|---|---|---|
| **Cloudflare Registrar** | Cloudflare MCP server or Registrar API (beta) | Account's default payment method | Redirects will run on Cloudflare (Bulk Redirects, a Worker) |
| **Vercel** | `vercel domains` CLI or Vercel MCP server | Vercel team billing | The user's site is on Vercel |
| **Porkbun** | Porkbun MCP server or API v3, with dry run | Prepaid account credit | A TLD the other two lack; a no-charge dry run before buying |
| Any other | The user, by hand | | The TLD is not offered by the three, or the user has an existing registrar |

Country-code TLDs vary: `com.au`, for example, needs an Australian-accredited registrar and the registrant must meet eligibility rules. Check the TLD at the registrar before recommending it. Do not recommend a registrar based on referral incentives.

## Cloudflare Registrar

Connect one of:

- **Cloudflare MCP server**: `https://mcp.cloudflare.com/mcp`, OAuth sign-in (the user picks permissions). Claude Code: `claude mcp add --transport http cloudflare-api https://mcp.cloudflare.com/mcp`. It exposes the whole Cloudflare API through `search` and `execute`; use them to find and call the Registrar endpoints below.
- **REST**: `https://api.cloudflare.com/client/v4` with an API token the user creates (never pasted into a checked-in file).

Flow (Registrar API, beta):

1. Search is optional and cached: `GET /accounts/{account_id}/registrar/domain-search?q=<name>`.
2. Live check and price: `POST /accounts/{account_id}/registrar/domain-check` with `{"domains": ["example.com"]}`. Treat this price as the price.
3. Confirm with the user (see above).
4. Register: `POST /accounts/{account_id}/registrar/registrations` with `{"domain_name": "example.com"}`. Usually done in seconds; it may return `202 Accepted` with a workflow URL to poll.

Notes:

- Priced at cost, charged to the account's default payment method, WHOIS privacy on by default. Registrations are non-refundable.
- The API supports a curated subset of TLDs; check Cloudflare's Registrar docs. Premium names need explicit fee acknowledgement.
- The account's default registrant contact is used, or contact details can be sent inline.
- The domain is on Cloudflare DNS from the start, so Bulk Redirects or a Worker can serve the redirect map straight away.
- Docs: https://developers.cloudflare.com/registrar/

## Vercel

Connect one of:

- **Vercel CLI**: the user runs `vercel login` once.
- **Vercel MCP server**: `https://mcp.vercel.com`, OAuth sign-in.

CLI flow:

1. `vercel domains check example.com` and `vercel domains price example.com`.
2. Confirm with the user.
3. `vercel domains buy example.com`. It asks for confirmation of the price, auto-renew and registrant contact in an interactive terminal. Run without a terminal, it returns a structured error and the next commands, and the user finishes the purchase themselves (terminal or dashboard). That handoff is by design; do not work around it.

MCP flow:

1. `check_domain_availability_and_price` with `names: ["example.com"]`.
2. `get_purchase_quote` with `product: "domain"`: returns the live `purchasePrice` and an `idempotencyKey`.
3. Confirm with the user.
4. `buy_domain` with `domain`, `teamId`, `years` (as quoted), `expectedPrice` (the quoted `purchasePrice`), `idempotencyKey`, `autoRenew`, the full registrant `contact` (first and last name, email, phone in E.164, address, city, state, zip, two-letter country) and `confirm: true`. The order is rejected if the live price no longer matches the quote.
5. It returns an `orderId`; `get_domain_order` shows when registration completes.

Redirects: attach the domain to a Vercel project and use `vercel.json` (`references/redirects.md`). Docs: https://vercel.com/docs/domains

## Porkbun

Connect one of:

- **Porkbun MCP server**: `https://mcp.porkbun.com/mcp`, OAuth sign-in (setup: https://porkbun.com/mcp). It exposes the API v3 calls below as tools; read its tool list.
- **API v3**: `https://api.porkbun.com/api/json/v3` with an API key pair from the user's account, loaded from environment variables. Buying needs the key's `purchase` permission; a manage-only key can still dry-run.

Flow:

1. `POST /domain/checkDomain/{domain}`: `avail` and `price` (USD).
2. `GET /domain/getRegistrationRequirements/{tld}`: continue only if `apiRegisterable` is `true`; otherwise the user registers on the website.
3. `POST /domain/create/{domain}` with `"dryRun": true`: returns the would-be cost, the balance and `wouldSucceed`, and charges nothing.
4. Confirm with the user.
5. `POST /domain/create/{domain}` with `cost` (integer US cents, matching the quote) and `agreeToTerms: "yes"`, plus an `Idempotency-Key` header so a retry cannot register or charge twice.
6. `GET /domain/get/{domain}` to verify.

Notes:

- Purchases spend prepaid account credit (`GET /account/balance`, in cents). A top-up (`POST /account/topup`) charges the user's saved card: ask before calling it.
- Defaults: at most $100 per API registration, and a $100 monthly API spend limit until the account holder changes it (`MONTHLY_SPEND_LIMIT_EXCEEDED`). Do not raise limits for the user.
- The account email and phone must be verified before the first registration (`VERIFICATION_REQUIRED`).
- Porkbun's URL forwarding forwards a whole hostname; it is not a per-URL redirect map. For a redirect map, point the nameservers at Cloudflare or the user's host (`POST /domain/updateNs`) and serve the map there.
- Docs: https://porkbun.com/api/json/v3/documentation

## Registration settings to recommend

- At least one year; multi-year is optional.
- Registrar lock and auto-renew on.
- The user's own contact details, with WHOIS privacy.
- DNSSEC only if the DNS host supports it end to end.

## After registration

1. Confirm the registration completed (the registrar's get or order call, or the user).
2. If a Revised hold is active: `release_hold` with `outcome: "registered"`. This triggers a fresh availability check and retires the listing so it is not offered to someone else.
3. DNS:
   - Cloudflare Registrar: already on Cloudflare DNS.
   - Elsewhere, to use Cloudflare for redirects: add the domain as a site in Cloudflare and switch nameservers at the registrar to the two Cloudflare gives; wait for "Active".
   - Vercel: add the domain to the project that will serve redirects.
   - For Cloudflare redirect rules, records must be proxied (orange cloud); a placeholder A record such as `192.0.2.1` is fine.
4. HTTPS: the host or Cloudflare issues the certificate. Plain http must 301 to https.
5. Check with `dig +short <domain>` and `curl -sI https://<domain>/`.
6. Only then deploy the redirects (see `redirects.md`).

## Do not

- Do not call any purchase, top-up or limit change without the user's yes to the exact name and price in this conversation.
- Do not enter payment details, ask for card numbers or registrar passwords, or paste API secrets into chat or files.
- Do not drive a registrar's checkout web page in a browser to pay.
- Do not register several names "to see what sticks". Register what the user will use.
- Do not tell the user the registered name has any particular ranking potential.
