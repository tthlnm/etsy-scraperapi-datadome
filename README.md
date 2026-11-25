# Etsy Product Scraper: Can ScraperAPI Get Past DataDome, What Do You Actually Get Back, and How Many Credits Does 10,000 Listings Burn? (Plus How the Dedicated Etsy Tools Compare)

ScraperAPI will fetch Etsy product pages and deal with the bot protection on the way in, but what comes back is HTML, not a finished product record. You hand it a listing, search or category URL, it returns the page, and your code pulls the title, price, images and shop name out of the markup. ScraperAPI's own published Etsy walkthrough works exactly that way: send the request through the API, get raw HTML, parse it with BeautifulSoup. The parser is yours to write.

Two numbers decide whether that trade is worth it. The free plan is **1,000 credits with 5 concurrent connections**. Etsy's pages sit behind DataDome, and ScraperAPI adds **10 credits per request** when it has to break through Cloudflare, DataDome or PerimeterX, on top of the 1 credit a standard page costs. Budget around 11 credits per Etsy product page and the free allowance covers roughly 90 pages of testing before you have to pay.

> The short version for most readers: if you can write or maintain a parser, start on the free plan to confirm what an Etsy URL actually costs you, then buy the smallest tier that covers your monthly credit burn. If you want columns in a spreadsheet without touching code, a dedicated Etsy scraper is usually cheaper in engineering time, and the comparison table below lays out what those cost.
>
> 👉 [See the current ScraperAPI plans and credit allowances](https://scraperapi.com/pricing?fp_ref=coupons)

## The plan ladder, and what each tier actually changes

ScraperAPI runs eight tiers: Free, Hobby, Startup, Business, Scaling, Professional, Advanced and Enterprise. The pricing page quotes the credits and the dollar figure for each one, and those numbers move with billing cycle and promotions, so treat the page as the only live source. What stays put is which tiers get the two features that decide how a month ends: auto-upgrade and pay-as-you-go.

| Plan | Allowance / price | Auto-upgrade when credits run out | Pay-as-you-go overage | Get it |
| --- | --- | --- | --- | --- |
| Free | 1,000 credits, 5 concurrent connections | No | No | [Start here](https://scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | Set on the pricing page | Yes | No | [See Hobby](https://scraperapi.com/pricing?fp_ref=coupons) |
| Startup | Set on the pricing page | Yes | No | [See Startup](https://scraperapi.com/pricing?fp_ref=coupons) |
| Business | Set on the pricing page | Yes | No | [See Business](https://scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | Set on the pricing page | Yes | Yes | [See Scaling](https://scraperapi.com/pricing?fp_ref=coupons) |
| Professional | Set on the pricing page | Yes | Yes | [See Professional](https://scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | Set on the pricing page | No | Yes | [See Advanced](https://scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Set on the pricing page | No | Yes | [See Enterprise](https://scraperapi.com/pricing?fp_ref=coupons) |

Every tier bills monthly or annually, and annual plans can be renewed early to get a fresh batch of credits.

The detail that catches people is what happens at the end of the month. By default, scraping pauses until credits renew. On Hobby through Professional you can switch on auto-upgrade, which bumps you to the next plan when you run dry; that is convenient and also the fastest way to spend more than you planned if a scraper goes into a retry loop. On Scaling, Professional, Advanced and Enterprise you get pay-as-you-go instead, billed at a fixed per-credit rate up to a monthly cap you set yourself. For a fixed monthly volume of Etsy pages, auto-upgrade on a mid tier is fine. For spiky work, like a one-week competitor price sweep, the cap on pay-as-you-go is the safer shape.

👉 [Compare which tiers include auto-upgrade and overage billing](https://scraperapi.com/pricing?fp_ref=coupons)

## What scraping Etsy product data actually involves

Etsy is not a site you can loop with plain `requests`. Three separate problems show up, and only the first one is about proxies.

**Bot protection.** Etsy front-loads DataDome, rate limiting and CAPTCHAs, which is the layer ScraperAPI's bypass credits pay for. Independent Etsy scrapers document the same wall: Scrape.do's Etsy toolkit exists specifically to get around DataDome, rate limits and CAPTCHAs, and it only reads publicly accessible HTML with no login. Apify's Xtracto listing recommends residential proxies and ships with a default concurrency of 5 and a retry budget of 5 per request, which tells you how often a single request fails on the first attempt.

**Region and currency.** Prices and language change with where the request appears to come from. Scrape.do's scripts pin `geoCode=us` and add a custom locale header (`sd-x-detected-locale: USD|en-US|US`) so prices come back in dollars instead of whatever the proxy's exit node suggests. Oxylabs takes `geo_location: 'United States'` in the request body. With ScraperAPI, geo targeting and custom headers are both listed features, and getting them wrong is the usual reason a price column comes back in the wrong currency or a listing title comes back translated.

**Parsing.** Etsy product pages carry JSON-LD structured data with name, description, price, images and rating, which is the most stable thing to read. Scrape.do builds its single-product scraper on exactly that, with fallbacks for single-offer and array-offer formats and several image formats. Search and category pages are messier: card layouts and class names change, and the count of cards on a page can shift because Etsy injects a "recently viewed" block.

**Reviews are a separate job.** They do not come from the product page HTML. Scrape.do's review script calls Etsy's internal GraphQL API and has to lift the listing ID, shop ID and a CSRF token out of the product page first, then page through results sorted by Relevancy, MostRecent or MostHelpful. Nothing in ScraperAPI's published material describes an equivalent review endpoint, so with ScraperAPI you would be reproducing that token dance yourself inside the HTML you get back. That is the single biggest reason to pick a dedicated Etsy tool if reviews are the point of the project.

## What you can pull off an Etsy listing

With ScraperAPI the field list is whatever your parser can find. With dedicated tools it is a fixed schema, and the schemas overlap heavily. Across the verified outputs of Oxylabs, Scrape.do and Apify's Xtracto actor, these are the fields that are reliably present on a product page:

- Listing ID, title, full description, category breadcrumb
- Price and currency, plus original price and discount rate on search pages
- Image URLs, including full-resolution versions
- Shop name and shop URL, shop rating, shop review count, Star Seller badge
- Product rating and review count
- Availability or stock status, and remaining quantity on some listings
- Variation count, materials list, seller tags
- Shipping origin country, free-shipping flag

Oxylabs' parsed output, for example, returns `stock_status`, `variation_count`, `shipping.from` and a seller object with rating and review count inside one JSON record. Xtracto returns `materials` and `tags` as arrays plus full-resolution image URLs. Search-page schemas add the merchandising fields: discount rate, "only X left", free shipping, Star Seller.

If your project needs materials, tags or variation-level pricing, check that the tool exposes them before you buy, because the raw HTML contains them but not always in a parseable place.

## What 1,000 and 10,000 Etsy pages cost in credits

This is where the search results you have probably read get vague. ScraperAPI bills credits per request, and the credit cost of a request depends on the domain and the parameters you attach. Its pricing FAQ is explicit that Amazon costs 5 credits, Google and Bing 25, LinkedIn 30, and bot-protected sites add 10 when the bypass runs. A standard page is 1.

Do the arithmetic with the DataDome surcharge included:

| Etsy pages per month | Credits at roughly 11 per page | What that means |
| --- | --- | --- |
| 100 | ~1,100 | Just past the free allowance |
| 1,000 | ~11,000 | Needs a paid tier, still small |
| 10,000 | ~110,000 | Where per-page cost starts to matter |

Two adjustments to that estimate. Retries are automatic in ScraperAPI, and a page that fails twice is not one page of work; leave headroom of 10–30% on top. Reviews, if you go after them, do not appear in that column at all, since they come from a different endpoint you have to call separately for each listing.

You can pin the real number down before committing: the dashboard has a Domain Cost Estimator that returns the exact credit cost for any URL including your parameters, and there is a `max_cost` setting that caps what a single scrape is allowed to spend. Run one real Etsy listing through the estimator rather than trusting a blog post's estimate, this one included.

👉 [Check the current ScraperAPI credit allowances against your monthly volume](https://scraperapi.com/pricing?fp_ref=coupons)

## How the dedicated Etsy scrapers compare

The three tools below solve the same problem with different shapes, and their billing units are not interchangeable with ScraperAPI's credits. Use the numbers to compare within each row, not across rows.

| Tool | Shape | Verified pricing | Verified Etsy coverage | Watch out for |
| --- | --- | --- | --- | --- |
| Scrape.do | API plus Python scripts you run and maintain | 1,000 free API credits per month | Search/category listings, single product via JSON-LD, reviews via Etsy's internal GraphQL with CSRF token and three sort orders | You manage the token, the scripts and the pacing; the scripts carry no publication date, so Etsy layout changes may have broken them |
| Oxylabs | Universal endpoint: send a URL, get HTML, or parsed JSON/Markdown for product pages | Not stated in its public README | Product pages with seller object, stock status, variation count, shipping origin, categories | Parsed output is per-URL; search and shop pages come back as HTML you parse |
| Apify, ParseForge Etsy Shop Products Scraper | Hosted actor, export to CSV/JSON/Excel/XML | **$0.15066 per result**; $15.07 per 100, $150.66 per 1,000, $1,506.60 per 10,000; $5 free credit for new accounts | All products from a named shop, up to 1M items per paid run; free plan previews 10 results | Shop input only, no category or keyword filter inside a shop; private or deactivated shops return nothing; opt-in extras since 3 Oct 2026 at $0.04 per shop for reviews and $0.02 per shop for About/policies, plus $0.005 per GB at run start |
| Apify, Xtracto Etsy Product Scraper | Hosted actor, one record per listing URL | Not stated on its listing page | Full listing detail including materials, tags, full-resolution images, shop name and rating | Residential proxies recommended; defaults are 5 concurrent requests and 5 retries, which sets the pace |

The billing basis differs enough to change the decision. Apify charges per result written to the dataset, so a shop with 400 products costs about $60 at the ParseForge rate whether those pages were easy or hard. ScraperAPI charges per request, so a hard page costs more than an easy one, and a page you retry three times costs more again. Neither model is wrong; they just fail differently. Per-result pricing protects you from Etsy's difficulty. Per-request pricing protects you when you are fetching pages that are cheap to get.

## Who should not buy ScraperAPI for this

If you need 50 listings for a one-off pricing check, you do not need an API. Apify's free plan previews 10 results per run, and a spreadsheet plus a browser gets you the rest.

If the deliverable is a review corpus rather than product records, ScraperAPI is the wrong starting point. You would be rebuilding Etsy's internal review call by hand, CSRF token included, and maintaining it. Scrape.do ships that as a script.

If nobody on the team wants to own a parser, buy a tool with a fixed schema. Class names on Etsy search pages change, page layouts get A/B tested, and the JSON-LD block occasionally ships with an array of offers instead of a single one. Every one of those is a small fix for someone who owns the code and a broken pipeline for someone who does not.

## FAQ

**Do I need an Etsy account or API key?**
No. Both ScraperAPI and the dedicated Etsy scrapers read publicly visible pages. The scrapers that document their access method are explicit that no login or authentication with Etsy is involved.

**Can ScraperAPI scrape Etsy reviews?**
There is no documented review endpoint. Reviews live behind Etsy's internal GraphQL API and need a listing ID, shop ID and CSRF token pulled from the product page first. You can attempt it against the HTML ScraperAPI returns, but you are building and maintaining that integration yourself.

**Is pricing per request or per result?**
ScraperAPI bills credits per request, with the cost set by the domain and the parameters; bot-protected sites add 10 credits. Apify's Etsy actors bill per result. A 10,000-product job is roughly 110,000 ScraperAPI credits or $1,506.60 at the ParseForge rate, and those two figures are not equivalent purchases.

**What happens if an Etsy shop has been deactivated?**
Public-page scrapers return empty data rather than an error. Apify documents this for private and deactivated shops. Build a check for empty result sets so you do not silently lose a shop from a recurring report.

**Does scraping Etsy break their terms?**
ScraperAPI's and the other vendors' documentation all point the same way: collect public product and review data, keep request volume sane, and stay within Etsy's terms. Review author names are personal data, so if you store them, GDPR, CCPA or PIPL obligations attach depending on where your users are.

## Before you pick a tier

Match the tier to the shape of the work, not to the size of the number in the plan name.

- A few hundred Etsy pages a month for testing or a small price watch: free plan first, then the entry paid tier. Confirm the per-URL credit cost in the Domain Cost Estimator before you upgrade.
- One to five thousand product pages a month on a steady schedule: a mid tier with auto-upgrade turned on, so a spike does not stall the pipeline mid-month.
- Tens of thousands of pages, or unpredictable bursts: a tier with pay-as-you-go and a cap you set, which is the only configuration where a runaway loop cannot quietly double your bill.
- Reviews as a core deliverable, or no appetite for maintaining a parser: a dedicated Etsy scraper, and budget by the result.

Two things to confirm on the live page before you enter a card: the credit allowance that actually ships with the tier you are looking at, and whether the billing cycle you choose can be renewed early for fresh credits. Both change the effective cost per Etsy page more than the headline number does.

👉 [Check ScraperAPI's current tiers, credit allowances and overage rules](https://scraperapi.com/pricing?fp_ref=coupons)
