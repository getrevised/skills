# Registering the domain

Revised sells nothing. The human registers the name at a registrar of their choice. You prepare and verify; you never handle payment, card details or registrar credentials.

## Before the user pays

1. Confirm the candidate passed `vetting.md` and, if redirecting, that the redirect map in `redirects.md` is drafted and approved.
2. Tell the user the exact name to register. Only give a real name if you got it from a disclosed row (`name_disclosed: true`). Never give a registrar a masked `domain` string.
3. Re-check availability at the registrar. `availability_checked_at` is when Revised last looked; the name can be gone now. A registrar search box or a `whois`/RDAP lookup is the check. If the name is registered, tell the user, and release any hold (the listing leaves the directory on its own once Revised notices).
4. Expired names sometimes sit in a registry redemption or pending-delete period. If the registrar says it is unavailable but the Revised stamp is recent, say so; the user may be racing another registrant. Do not suggest backorder services unless the user asks.
5. Check price at the registrar. Premium or aftermarket pricing is the user's decision.

## Registrar choice

Any registrar is fine. Suggest the user's existing one, or one that supports the DNS host they use. Do not recommend a registrar based on referral incentives. Registration details to tell the user:

- Register for at least one year; multi-year is optional.
- Turn on registrar lock and auto-renew.
- Use their own contact details (WHOIS privacy where the registrar offers it).
- Enable DNSSEC only if their DNS host supports it end to end.

## After registration

1. Ask the user to confirm the registration succeeded.
2. If a Revised hold is active: `release_hold` with `outcome: "registered"`. This triggers a fresh availability check and retires the listing so it is not offered to someone else.
3. DNS:
   - If the user is on Cloudflare: add the domain as a site, switch nameservers at the registrar to the two Cloudflare gives, wait for "Active".
   - Otherwise: use the registrar's DNS or the user's existing host.
   - Point the apex (and `www`, if redirecting both) so requests reach a place that can answer redirects: the user's host, a Worker, or a Cloudflare redirect rule. For Cloudflare redirect rules, the DNS records must be proxied (orange cloud) with any placeholder A record, e.g. `192.0.2.1`.
4. Add HTTPS: certificate issuance by the host or Cloudflare. Plain http must 301 to https.
5. Confirm with `dig +short <domain>` and `curl -sI https://<domain>/`.
6. Only then deploy the redirects (see `redirects.md`).

## Do not

- Do not enter payment details for the user or ask them to paste card numbers or registrar passwords.
- Do not register on the user's behalf through a browser session unless the user explicitly instructs you to and is present to complete payment.
- Do not register several names "to see what sticks". Register what the user will use.
- Do not tell the user the registered name has any particular ranking potential.
