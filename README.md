# Scrape Amazon Reviews: How Many You Can Actually Get, Why amazon.com Now Needs a Login, and Which ScraperAPI Plan Fits (Plan Table + Field List)

Anonymous visitors to Amazon see a sample of **8 to 13 reviews per product**. That is the real ceiling for any tool that doesn't log in, and on amazon.com even that sample closed in October 2026, when reviews became visible only to signed-in shoppers.

ScraperAPI's own documentation confirms the consequence. Its structured Amazon Reviews endpoint is listed as unavailable, because Amazon review pages now require a login and ScraperAPI doesn't scrape behind logins. The recommendation is to use the Amazon Product Page API and collect whatever reviews Amazon still shows publicly.

So the useful question isn't "which tool scrapes all the reviews." It's "how many public reviews exist for my ASINs, what does each Amazon request cost in credits, and which plan do I need for the volume I'm actually running." That's what this breaks down.

> **Quick take:** for a recurring weekly pull across a list of ASINs, run the Amazon Product Page API on a paid tier with pay-as-you-go overage, so a heavy week doesn't stall the pipeline. If you're still testing, the free plan's **1,000 credits and 5 concurrent connections** buy 200 Amazon page fetches, enough to see what your own ASINs expose.
> 👉 [Start free and price your own ASINs](https://scraperapi.com/pricing?fp_ref=coupons)

## How many Amazon reviews can you actually get?

Between 8 and 13 per product on a single storefront with default settings. That's Amazon's public sample for logged-out visitors, and it holds across products and storefronts with some variation. The complete review list is reserved for signed-in shoppers.

Two things follow from that, and both catch people out.

**The rating count is not the review count.** A product showing 51,884 ratings may return a dozen readable reviews. `product_rating_count` includes every star-only rating, including the ones where nobody wrote anything. If you divide total ratings by page count and plan storage around the result, you'll be off by orders of magnitude.

**amazon.com is closed to anonymous requests.** Since October 2026, reviews on the US storefront display only to signed-in users. The workaround compliant tools use is to read the same product on Amazon Canada or Amazon UK and tag every record with the storefront it came from, so you know a "review" in your dataset is really a Canadian or British one. That matters for sentiment work: a complaint about shipping to Ontario tells you nothing about US fulfilment.

This wall isn't new, only tighter. ScrapeHero retired its Amazon review scrapers and API back in October 2023, saying Amazon had begun requiring authentication to view full review data. Anyone selling you "the complete review history of a product" in 2026 is either logging in against Amazon's terms or misdescribing what they deliver.

Widening the sample is legitimate, just limited. Pulling the same ASIN from `co.uk` and `ca` alongside `com`, plus a couple of colour or size variants, surfaces different reviews. How much overlap you get varies by product, and every filter — star range, verified-only, date window — cuts the pool further.

## What ScraperAPI does and doesn't do for Amazon reviews

There is no working dedicated review endpoint, and the docs say why in plain terms: review pages require a login, ScraperAPI doesn't do logins, so the endpoint isn't available. What you get instead is the Amazon Product Page API, plus the async structured-data route for batch work.

ScraperAPI's own guide to Amazon reviews shows the shape of that output: average rating, total rating count, the star-by-star distribution as absolute numbers and percentages, product metadata, and a single top positive review with its star rating, date, verified-purchase flag, reviewer name, title and body. That's a rating summary with one representative review attached — useful for monitoring a catalogue, not a substitute for a review corpus.

The billing detail that decides your budget: **a standard page costs 1 credit, Amazon costs 5.** Google and Bing cost 25, LinkedIn 30, and sites behind Cloudflare, DataDome or PerimeterX add 10 credits per request when ScraperAPI bypasses them. Every Amazon page you fetch is five times a normal one.

Two controls keep that from running away. `max_cost` caps what any single scrape can spend, and the Domain Cost Estimator in the dashboard prices a specific URL before you commit. Amazon being a five-credit domain is exactly the case that estimator exists for.

👉 [Check the current per-plan credit allowances](https://scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans, compared

The tiers differ in more than size. Auto-upgrade and pay-as-you-go are distributed unevenly across the ladder, and that's the part that affects whether a busy week breaks your pipeline or just costs more.

| Plan | Who it fits | When credits run out | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Testing what your ASINs expose before paying anything | Hard stop: 1,000 credits, 5 concurrent connections | $0 | [Open a free account](https://scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | First paid step, solo projects | Auto-upgrade to the next tier (opt-in) | Monthly or annual | [See Hobby pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Startup | Small teams running regular pulls | Auto-upgrade to the next tier (opt-in) | Monthly or annual | [See Startup pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Business | Production jobs that can't pause mid-month | Auto-upgrade to the next tier (opt-in) | Monthly or annual | [See Business pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | Variable volume that needs overflow without a plan change | Auto-upgrade **or** pay-as-you-go up to a monthly cap you set | Monthly or annual | [See Scaling pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Professional | Same as Scaling, at higher baseline volume | Auto-upgrade **or** pay-as-you-go up to your cap | Monthly or annual | [See Professional pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | High sustained volume | Pay-as-you-go up to your cap | Monthly or annual | [See Advanced pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom terms and volume | Pay-as-you-go up to your cap | Custom | [Discuss Enterprise terms](https://scraperapi.com/pricing?fp_ref=coupons) |

I've left dollar figures out of that table deliberately. ScraperAPI's per-plan rates and credit allowances change, and a number I quote today can be stale by the time you read it. The pricing page carries the live rate, annual plans are billed at a different rate than monthly, and annual subscribers can renew early for a fresh batch of credits when a project front-loads its usage.

The distinction worth remembering: pay-as-you-go only appears from Scaling upward. On Hobby, Startup or Business, running dry means auto-upgrade moves you to the next tier, which is a bigger commitment than paying for a few extra thousand credits. If your Amazon volume spikes unpredictably, that alone justifies starting at Scaling.

## The fields worth storing

Whichever route you use, the record structure matters more than the scraper. This is the field set a well-built Amazon review export carries, and why each one earns its column:

| Field | Why it matters |
| --- | --- |
| `review_id` | Amazon's own review ID, stable across runs and storefronts. The dedupe key for any recurring job. |
| `rating` | 1–5 as a number, not "5.0 out of 5 stars" — sortable and aggregatable |
| `title`, `body` | Headline plus full text with the reviewer's paragraph breaks preserved |
| `date` | ISO `YYYY-MM-DD`; keep `date_raw` too if you need the original string |
| `verified_purchase` | True when Amazon shows the Verified Purchase badge |
| `is_vine` | Flags Vine Customer Review of Free Product — worth excluding from sentiment averages |
| `helpful_votes` | A crude weight for how much a complaint resonated |
| `variant`, `variant_asin` | "Color: Large \| Copper" style strings; reviews often attach to one variant only |
| `country`, `language` | Where the review was written and in what language, e.g. `en-US` |
| `images` | Full-size reviewer photo URLs, first one duplicated as `image_url` |
| `marketplace` | Which Amazon site the record came from — the field that tells you a review is really from `amazon.co.uk` |
| `product_rating`, `product_rating_count`, `product_rating_distribution` | Product context, with Amazon's displayed percentage for each star level |

`review_id` is the one people skip and later regret. Because it's stable across runs *and* across marketplaces, you can append new pulls to an existing table and drop duplicates on that single column. Without it you're matching on author name plus date plus the first forty characters of body text, which fails the moment someone edits a review.

`product_rating_distribution` deserves a mention too. It gives you the 5-to-1 star split as Amazon displays it, which is often the fastest signal of a quality slide — a shift from 78/13/5/2/2 toward 60/15/8/7/10 shows up in the distribution long before it shows up in the average.

## Three things that produce an empty CSV

**The review widget is lazy-loaded.** It isn't in the DOM when the page first renders. A working sequence is: wait 2000ms, scroll to the bottom of the page, wait another 2000ms, then run extraction. If your results are empty or partial, raise both waits to 3500ms — slow pages routinely need it.

**`country_code` has to match the domain in the URL.** A US proxy hitting `amazon.de` is a fingerprint Amazon spots immediately, and it's one of the two most common causes of empty results. ScraperAPI handles this with paired `tld` and `country` parameters on Amazon routes: if you need `amazon.com` content as seen from Canada, both must be set. Supported TLDs run from `com`, `co.uk` and `ca` through `de`, `fr`, `it`, `es`, `jp`, `in`, `com.mx`, `com.au`, `com.br`, `nl`, `sg`, `ae`, `sa`, `se`, `pl`, `tr` and `cn`.

**Request pacing and retries.** Amazon defends itself with CAPTCHAs, IP bans and user-agent checks, which is the whole reason a managed API exists. Start at 5 concurrent requests, which is the free plan's ceiling and a sane starting point on paid plans too. Retry only transient failures — twice is enough — and never retry a 4xx, because that means the request was wrong rather than unlucky. Cache locally: reviews barely change inside 24 hours, so re-scraping the same ASIN daily is wasted credits.

The selectors, if you're extracting client-side or debugging an empty response:

css
#cm-cr-dp-review-list > li        /* review list items */
.a-profile-name                   /* reviewer */
.review-rating > span             /* star rating */
.review-date                      /* review date */
.review-title span:not([class])   /* headline */
.review-text                      /* body */


## Per-request vs per-review pricing

Two billing models dominate this niche and they punish different mistakes.

ScraperAPI charges **per request**, at 5 credits for an Amazon page. Your cost scales with how many pages you fetch, not with how many reviews come back — so a page that yields four reviews costs the same as one that yields twelve. That model rewards knowing your ASINs and punishes blind crawling.

Apify's Amazon review actors charge **per review**: one of them lists $1.75 per 1,000 reviews plus a $0.02 run fee, with the per-product cap defaulting to 10 and maxing at 50. ScrapingBee charges **per request** too, at 25 credits for a JS-rendered page, doubling with `premium_proxy` and higher with `stealth_proxy`; its 1,000 free credits work out to roughly 40 such requests.

Neither model is cheaper in the abstract. If you need review text at volume across a narrow set of products, per-review pricing is easier to forecast. If you need product context, ratings and review text together across a wide ASIN list, per-request pricing at 5 credits per Amazon page is straightforward arithmetic — 1,000 free credits is 200 Amazon fetches, and what those 200 fetches yield depends entirely on how much each of your product pages exposes.

## Is scraping Amazon reviews legal?

In many jurisdictions, scraping publicly visible data is generally lawful. Amazon's terms of service separately restrict automated access, and those are two different questions — a site's ToS can prohibit something the law permits.

The practical rules that follow: collect only public, non-authenticated content; keep request rates reasonable; and treat reviewer names and opinions as personal data. Under GDPR and CCPA that means minimising what you collect, storing it securely, and honouring deletion requests. Logging in to reach the full review list is against Amazon's terms and outside what compliant tools will do for you — which is precisely why the review endpoint at ScraperAPI doesn't exist.

None of this is legal advice. Check Amazon's terms and your own jurisdiction's rules before you put anything into production.

## When you shouldn't buy anything

You need a plan if you're monitoring a set of ASINs on a schedule and want structured output without maintaining proxies. You don't if any of these describes you:

- **You want the complete review history of one product.** No compliant vendor sells that, at any price. The public sample is the public sample.
- **You need thirty reviews for a one-off qualitative read.** The free tier covers 200 Amazon fetches. Use it and stop.
- **You need reviews from amazon.com specifically, for US shoppers.** Since October 2026 that requires a login. Canadian or UK reviews of the same product are not a substitute for US sentiment, and no amount of credits changes that.

## Picking a tier

Running a weekly pull across a few dozen ASINs with unpredictable spikes? Start at **Scaling** — it's the cheapest tier where pay-as-you-go overage exists, so a heavy week costs extra credits instead of forcing a plan upgrade.

Steady, predictable volume inside your allowance? **Hobby** or **Startup** is fine, and auto-upgrade is a safety net rather than a routine event.

Testing whether the Product Page API returns enough for your ASINs at all? The free plan's 1,000 credits and 5 concurrent connections will tell you in an afternoon.

Two things to confirm on the pricing page before you pay: the credit allowance for the tier you're choosing, and whether that tier offers pay-as-you-go or only auto-upgrade. Those two answers decide what happens on your worst traffic month, which is the month you'll care about.

👉 [Compare ScraperAPI plans and start with 1,000 free credits](https://scraperapi.com/pricing?fp_ref=coupons)
