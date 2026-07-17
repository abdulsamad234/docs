# SEO playbook for the Referly docs

Internal notes — excluded from the published site via `.mintignore`.

## What's already configured

In `docs.json`:

| Setting | Value | Why |
|---|---|---|
| `seo.indexing` | `all` | Indexes every page, not just navigable ones. Safe here because every page is in the nav. |
| `seo.organization` | name, url, logo, `sameAs` | Emits schema.org Organization structured data linking the docs to your brand and socials. |
| `seo.metatags` | `og:site_name`, `og:type`, `twitter:card`, `twitter:site` | Social previews. |
| `metadata.timestamp` | `true` | Shows "last updated" — a freshness signal. |

Every page has a unique `title` and `description`. Home pages also carry `keywords`.

## Before launch

### 1. Verify the canonical host

Docs will serve from `www.referly.so/{help-center,developer-documentation,api-reference}`.
Confirm after deploy that `<link rel="canonical">` points at `www.referly.so` and **not** the
`*.mintlify.site` origin. Canonicals don't render in `mint dev` — only in production.

**If the mintlify.site origin is indexable, you get duplicate content and the docs compete with
themselves.** Check `https://<subdomain>.mintlify.site/robots.txt` and ask Mintlify support to
`noindex` the origin if it isn't already.

### 2. Sitemap

Mintlify generates **one** sitemap covering every page across all three subfolders (dub's
`/docs/sitemap.xml` carries 241 URLs spanning both `/docs/*` and `/help/*`).

`www.referly.so/sitemap.xml` already belongs to the marketing site, so **do not** rewrite it away.
Instead:

1. Find the Mintlify sitemap's real path after the dashboard change (likely
   `<subdomain>.mintlify.site/sitemap.xml`).
2. Proxy it to a non-conflicting path, e.g. `www.referly.so/docs-sitemap.xml`.
3. Reference **both** in the marketing site's `robots.txt`:

```
Sitemap: https://www.referly.so/sitemap.xml
Sitemap: https://www.referly.so/docs-sitemap.xml
```

4. Submit both in Google Search Console.

### 3. The 301 from developers.referly.so

Every `developers.referly.so/*` URL 301s to `www.referly.so/developer-documentation`. Consequence
worth accepting knowingly: the existing indexed `/api-reference/*` pages lose their rankings, because
a many-to-one redirect reads as a soft 404 to Google rather than a page move. If those pages have
traffic, redirect `developers.referly.so/api-reference/:path*` to
`www.referly.so/api-reference/:path*` instead and keep the equity.

Keep the old domain's DNS alive indefinitely — Google needs to re-crawl the 301s, which takes months.

## After launch

### Google Search Console
- Add `www.referly.so` as a domain property (covers all subfolders).
- Keep `developers.referly.so` as a property too, and use **Change of Address** if you move the whole
  domain.
- Submit both sitemaps. Watch Coverage for `Crawled – currently not indexed` on stub pages.

### Stub pages are a real risk
There are ~250 stub pages with frontmatter and no body. Thin content at this scale can drag the
whole section down.

**Do not deploy all 281 pages empty.** Either:
- write content before launch, or
- keep only written pages in `docs.json` navigation and set `seo.indexing` back to `navigable` —
  unnavigable pages then stay out of the index until you're ready.

This is the single highest-impact SEO decision here.

## How to actually rank

### Help Center
Target support intent — people searching a problem, not a product:
- "how to set up an affiliate program"
- "affiliate commission structure"
- "pay affiliates with Wise / PayPal"
- Competitor migration: **"Rewardful alternative"**, "migrate from FirstPromoter", "Tapfiliate
  alternative". You support 8 competitor imports and nobody knows. These are the highest-commercial-
  intent terms you own, and dub ranks on exactly this pattern.

### Developer Documentation
Target implementation intent:
- "affiliate tracking script"
- "track affiliate sales server side"
- "affiliate webhook events"
- Per-integration: "Stripe affiliate tracking", "Shopify affiliate app"

The Webhooks section matters most — 15 live event types, zero docs today.

### API Reference
Ranks on named entities: "Referly API", "Referly API authentication". Low volume, high intent. The
win here is **completeness**, not keywords — the OpenAPI spec is missing `/payout-batches` and
`PATCH /sales`.

### Structural wins
1. **Internal linking.** Every stub should link to 2-3 siblings when written. Orphan pages don't
   rank.
2. **One page per query.** Don't merge "Migrate from Rewardful" and "Migrate from Tolt" — separate
   pages rank for separate searches.
3. **Answer in the first paragraph.** AI Overviews and featured snippets extract the opening.
4. **`llms.txt`** is generated automatically and is how ChatGPT/Claude/Perplexity read your docs —
   confirm it's reachable through the proxy after deploy.
5. **Keep descriptions unique.** Duplicate meta descriptions across 281 pages would be a real
   problem; each one here is distinct today — keep it that way.

## Verify after deploy

```bash
curl -s https://www.referly.so/help-center | grep -E 'canonical|og:|description'
curl -sI https://developers.referly.so/api-reference/affiliates/get   # expect 301
curl -s https://www.referly.so/docs-sitemap.xml | grep -c '<loc>'      # expect ~281
curl -s https://<subdomain>.mintlify.site/robots.txt                   # origin should noindex
```
