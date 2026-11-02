# Redirect plan

Goal: send each old URL that had inbound links to the closest genuinely relevant page on the user's site, and say Gone for the rest. A redirect is a claim that the destination is the same content or its replacement. Make only honest claims.

## 1. Extract the old URLs

Wayback CDX API, one line per unique URL that returned 200:

```
https://web.archive.org/cdx/search/cdx?url=<domain>/*&output=json&fl=original,statuscode&filter=statuscode:200&collapse=urlkey
```

`<domain>` is the registered domain without a scheme (use the real one; `example.com` here only as an illustration). Notes:

- Output is JSON: first row is the header, the rest are `[original, statuscode]`.
- Add `&limit=5000` for big sites; use `&from=YYYY&to=YYYY` to bound years.
- Normalize: strip `:80`, lower-case the host, decide whether to treat `www.` and bare the same, drop fragments.
- Be polite to archive.org: one request, no tight loops.
- CDX tells you which URLs existed, not which had inbound links. Use `linked_from` and the user's backlink tool where available to prioritize; keep content pages, drop obvious noise (assets, `feed`, `wp-json`, `?s=`, tracking parameters, pagination, tag/date archives).

## 2. Map each URL

For each old URL, read the archived page (or its title/slug) and the user's site page inventory. Decide:

| Situation | Status |
|---|---|
| A page on the user's site covers the same subject | `301` to it |
| A section or hub page genuinely covers the topic | `301` to the section |
| Homepage of old site and the user's homepage are on the same topic | `301` to the user's homepage |
| No relevant counterpart | `410` |
| Old URL was an asset, feed or junk | omit (let it 404) |

Rules:

- **Never blanket-redirect everything to the homepage.** Google treats mass irrelevant redirects as soft 404s, and it is the signature of expired-domain abuse.
- Prefer 410 over a stretched 301. If the destination would surprise a reader who clicked the old link, use 410.
- One hop only: source straight to final destination. No chains, no loops.
- Keep path and query handling explicit; do not use wildcards that send unrelated paths to one target.
- If fewer than about half of the content URLs have an honest counterpart, say the domain is not a topical fit and recommend dropping it (see `policy.md`).
- Do not create new thin pages just to receive redirects.

## 3. Redirect map file

Write `redirects.csv` with a header and one row per rule:

```
source,target,status,reason
https://example.com/guides/widget-basics,https://example.org/learn/widgets,301,same topic: widget fundamentals
https://example.com/old-promo,,410,no counterpart
```

Include `reason` so the user can review it. JSON is fine if the target tool prefers it. Show the user the whole map and any counts: total rows, 301s, 410s, omitted. Get approval before deploying.

## 4. Implementation snippets

Pick the one that fits the user's stack. Replace placeholders; keep `example.com`-style names in drafts you share.

**Cloudflare Bulk Redirects** (zone on Cloudflare, domain proxied): create a Bulk Redirect List from the CSV (columns `source_url`, `target_url`, `status_code`), then a Bulk Redirect Rule that enables the list. Bulk Redirects support 301 and similar status codes only; handle 410 with a Worker or a custom rule that returns 410.

**Tiny Worker** (also serves 410):

```js
const REDIRECTS = {
  "/guides/widget-basics": "https://example.org/learn/widgets",
};
const GONE = new Set(["/old-promo"]);

export default {
  async fetch(request) {
    const url = new URL(request.url);
    if (url.protocol === "http:") {
      url.protocol = "https:";
      return Response.redirect(url.toString(), 301);
    }
    const path = url.pathname.replace(/\/$/, "") || "/";
    const target = REDIRECTS[path];
    if (target) return Response.redirect(target, 301);
    if (GONE.has(path)) return new Response("Gone", { status: 410 });
    return new Response("Not found", { status: 404 });
  },
};
```

**`_redirects`** (Cloudflare Pages / Netlify):

```
/guides/widget-basics  https://example.org/learn/widgets  301
/old-promo             /410.html                           410
```

(Netlify honours `410` via a status rule; Pages does not support 410, use a Worker there.)

**nginx**:

```nginx
map $uri $redirect_target {
    default "";
    /guides/widget-basics https://example.org/learn/widgets;
}
server {
    listen 443 ssl;
    server_name example.com www.example.com;
    # ssl_* omitted
    location = /old-promo { return 410; }
    location / {
        if ($redirect_target) { return 301 $redirect_target; }
        return 404;
    }
}
server { listen 80; server_name example.com www.example.com; return 301 https://$host$request_uri; }
```

Redirect both `www` and apex to the same rules.

## 5. Verify before announcing

For a sample (all rows if under 100):

```
curl -sI -o /dev/null -w '%{http_code} %{redirect_url}\n' https://example.com/guides/widget-basics
```

Check: expected status, `Location` equals the mapped target, no chain (the target itself is 200, not another redirect), http redirects to https.

## 6. Keep and monitor

- Keep redirects for at least a year, longer if links keep arriving.
- Watch Google Search Console and Bing Webmaster Tools (add the old domain as a property, using DNS verification) for soft 404s, redirect errors and crawl anomalies. Fix what they report; do not add blanket redirects to silence them.
- Do not repurpose the old domain for new unrelated content later without re-reading `policy.md`.
- Never state or imply that redirects will transfer ranking. They may help users and crawlers find the right page; outcomes vary.
