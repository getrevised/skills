# Policy guardrail: expired domain abuse

Read this before shortlisting for a goal and again before finalizing a redirect plan.

## What the policy says (paraphrased)

Google's spam policies list **expired domain abuse**: buying an expired domain and using it primarily to manipulate search rankings by hosting content with little or no value to users, typically content unrelated to what the domain was before. Sites that do this can be demoted or removed from results. Google's examples include affiliate or low-value content on the former site of a government agency or non-profit, and similar repurposing. Read the current text, which may change: https://developers.google.com/search/docs/essentials/spam-policies

Related policies that apply to the same plans: scaled content abuse, site reputation abuse, and link spam (including link schemes that exist to pass ranking signals). Bing and other engines have comparable rules. This file is not legal advice or a statement of how any engine will treat a given site.

## Legitimate uses

- The domain is **topically related** to the user's real business or publication, and the user serves the same audience.
- **Relevant 301s**: each old page that had links goes to a page on the user's site that really covers the same subject, and old URLs with no counterpart return 410.
- **Rebuilding genuinely useful content** on the domain: real pages that serve visitors, honestly branded as the user's, not as the previous owner.
- Registering the former domain of the user's own brand or a business the user acquired, then redirecting it to the current site.
- Using the domain as a product's or project's own site, with original content.

## Refuse or flag and stop

If the plan, or the user's stated goal, is any of the following, say so plainly, explain why, and do not continue it. Offer the legitimate alternative where one exists.

1. **Redirecting an unrelated domain to a site just to pass link equity.** Includes homepage-only blanket 301s from a domain on a different topic.
2. **Building a PBN**: registering several expired domains to host thin or templated sites that link to a money site.
3. **Impersonating the previous owner or brand**: copying the old site's design, name, logo or contact details, claiming its history, reviving a defunct organization's identity (especially a government body, school, charity, healthcare provider or news outlet) to borrow trust.
4. **Thin or unrelated content on the old domain** (spun articles, affiliate or casino/adult/pharma pages, mass generated pages) to exploit the old domain's reputation.
5. **Hiding the redirect** from users or crawlers (cloaking, conditional redirects based on user agent).
6. Using the history as bait for a scam or phishing page.
7. Anything the user frames as "so Google won't notice" or "to get around" a penalty.

When refusing, be brief and constructive: "I won't build this plan, because it redirects <topic A> pages to a <topic B> site only for link equity, which is the pattern Google's expired domain abuse policy targets. I can instead look for domains in <topic B>, or help you plan relevant content."

## Gray areas: ask, do not assume

- Partial topical overlap (e.g. a general hobby site redirecting to a niche shop in the same hobby). Map only the honest pages; 410 the rest; tell the user the risk.
- A domain with a strong history but a different language or country. Only proceed if the user truly serves that market.
- A user who wants to "just test" what happens. Recommend a relevant, original use or no use.

## Things you must not say

- That any redirect, domain or metric will improve rankings, traffic, authority or revenue.
- That a domain is "safe", "penalty-free", "clean" or "vetted" (see `vetting.md`).
- That Revised or any tool guarantees compliance. Compliance is about how the user uses the domain.

## Checklist before handing over a plan

- [ ] Every redirect target is a page a user following the old link would find relevant.
- [ ] There is a 410 (or omission) for every old URL without a relevant counterpart.
- [ ] No blanket redirect to the homepage unless the topics truly match.
- [ ] No content on the old domain that borrows the previous owner's identity.
- [ ] The user has been told this is their decision and risk, with no ranking promise.
- [ ] Monitoring plan in place (Search Console / Bing Webmaster Tools).
