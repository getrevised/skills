# Vetting a candidate

Revised screens out high-spam, parked-only, trademark-risk and no-signal names before listing. That is a floor. Everything below is your job, per candidate, against the user's site. Record the result of each check; do not summarize as "looks fine".

## 1. Topical match (the gate)

Question: would a reader of the old site find the user's site a natural continuation or a relevant resource?

1. Get the old site's history from the Wayback Machine (`https://web.archive.org/web/<year>/<domain>/` for several years, or the CDX list from `redirects.md`). Look at its homepage across time and its top content pages.
2. Write one line: what the old site was about, in what language, for which country.
3. Compare with the user's brief (topic, audience, language, geography).

| Result | Action |
|---|---|
| Same topic, language and audience | Continue vetting. |
| Adjacent topic, some old pages have honest counterparts | Continue, but expect many 410s. Tell the user. |
| Different topic, or only the homepage could map | Reject. |
| Different language or country the user does not serve | Reject unless the user serves that market. |

Revised's `category` and `snapshots` help find candidates but do not replace reading the archive. The category is inferred from archived pages; it can be broad (`other`), wrong, or missing.

## 2. Linker quality

- Read `linked_from` (and `linked_from_count`). Recognisable editorial linkers (encyclopedias, universities, government, major publishers, community sites) are worth more than a large count of anonymous links.
- Read `agent_citability.basis` and `sources`. `seed_links` names the AI-cited sources that link in, so check that they are relevant to the user's topic. `link_graph` means no such source; the score is graph position only (max 70), so treat it as weak evidence. A null `agent_citability` is absent data, not a bad sign.
- Compare `referring_domains` with `backlinks`. Many backlinks from few referring domains usually means a sitewide footer or blogroll.
- Ask whether the linking pages are about the same subject as the old site. Links from relevant pages justify relevant redirects; links from unrelated pages are not a reason to buy.
- Optionally check that links still exist today with the user's backlink tool. A recorded link may be gone.

Bands are ranges. Do not rank two candidates on a band difference alone.

## 3. Red flags

Look in the archive across all years, not just the last snapshot.

- Casino, gambling, adult, pharma, loan, replica, or dropship/affiliate-farm content at any point.
- Long runs of parked pages, registrar placeholders, or "domain for sale" pages (fine in moderation; a mostly-parked history means thin evidence).
- Scraped, auto-translated or doorway content; thousands of near-identical pages.
- Content in a language that does not match the linking sites.
- A hack or injection period (spam links or pages inserted into a real site).
- Redirect history to unrelated sites or to a different brand.
- `spam: "moderate"`: not an automatic no, but demands a closer archive read.
- `metrics_missing` including most fields: you are judging on little evidence; say so.

## 4. Trademark and impersonation

- If the name contains or resembles a known brand, or is the former name of an identifiable organization, flag it. Do not recommend a name that would let the user pass as that brand.
- Registering a name that happens to be a former business's name is not itself wrong, but presenting the new site as that business is (see `policy.md`).
- You are not a lawyer. Flag trademark risk and recommend the user checks it; do not clear it.

## 5. Freshness and availability

- Read `availability_checked_at`. If it is more than about 48 hours old, say the name may have been registered since, and plan a registrar check (see `registration.md`).
- On an open listing, a first `reveal_domain` re-checks availability for you.

## Reject rules (apply strictly)

Reject the candidate if any is true:

1. No honest topical match to the user's site.
2. The archive shows spam, adult, gambling, pharma or doorway content at any point.
3. Most of the old site's history is parked or empty and the linkers are not relevant.
4. The name infringes or impersonates a brand or organization.
5. The only use you can imagine is redirecting everything to the homepage.
6. The user's stated goal is rankings from link equity alone, with no relevant content to point it at. Apply `policy.md`.

## Report format per candidate

```
name:            <only if open tier, or in-conversation>
id:              <listing id>
old site:        <one line, with years observed>
match to user:   <one line>
linkers:         <top 3 named, count, relevance>
metrics:         RD <band>, score <range>, age <n|null>, snapshots <band>
risks:           <list, or "none found; archive checked for <years>">
unverified:      <what you could not check>
availability:    checked <timestamp>; registrar check needed
verdict:         pursue | pursue with caveats | reject (reason)
```

Never describe a candidate as "vetted", "clean" or "safe". Say what you checked.
