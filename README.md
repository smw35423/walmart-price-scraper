# Walmart Price Scraper: Why Search Pages Come Back Without Prices, What 1,000 Priced Products Really Costs, and How to Size a Plan (Full ScraperAPI Plan Table)

A Walmart price scraper gets billed in two parts: the page that lists products, and the product page that carries the price. Walmart's search results don't include prices, so every priced row in your spreadsheet is a second fetch. That one fact is why a plain listing job and a real price-monitoring job differ in cost by around 4x, and it's the number you need before you pick a plan size.

Below: the full ScraperAPI plan table, the credit arithmetic for a Walmart request, five field-level traps that quietly corrupt price reports, and the sizing rule that works for a daily repricing run.

> **Size by requests, not by rows.** A Walmart request costs 5 credits, so Hobby's 100,000 credits cover roughly 20,000 fetches a month. A 1,000-product daily check needs about 30,000 requests, which puts you on Startup. The tier ScraperAPI labels most popular, Scaling, is **$475/month, or $427.50/month billed annually** ($5,130/year) for 5,000,000 credits and 200 concurrent threads.
> 👉 [See the current ScraperAPI plan prices](https://scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans, side by side

ScraperAPI lists seven paid tiers plus a free plan. For a Walmart job the differences that matter are monthly credits, concurrent threads and how precise your geotargeting can get. Credits reset when the subscription renews and don't roll over on any tier.

| Plan | Credits / month | Concurrent threads | Geotargeting | Price & billing | Link |
| --- | --- | --- | --- | --- | --- |
| Free | 1,000 | 5 | — | $0 | [Free plan details](https://scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | 100,000 | 20 | US & EU | Monthly or annual, 10% off annual | [Check price](https://scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | US & EU | Monthly or annual, 10% off annual | [Check price](https://scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Country-level | Monthly or annual, 10% off annual | [Check price](https://scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Country-level | $475/mo, or $427.50/mo billed annually | [Get Scaling pricing](https://scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Country-level | Monthly or annual, 10% off annual | [Check price](https://scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Country-level | Monthly or annual, 10% off annual | [Check price](https://scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Country-level | Custom quoted | [Check price](https://scraperapi.com/pricing?fp_ref=coupons) |

Scaling is the one tier whose dollar figure is spelled out in the page copy; the others are described by credit allowance and concurrency, with the number sitting on the same pricing page. I've left those as listed rather than estimate them.

Two billing rules that change how a monitoring job behaves. Pay-as-you-go, which keeps requests flowing past your monthly allowance at a fixed per-credit rate up to a cap you set, is available on Scaling, Professional, Advanced and Enterprise only. On Hobby, Startup and Business, running out of credits mid-month means HTTP 429 responses until renewal, unless you've opted into auto-upgrade. There's no surcharge for going over: the request is turned away rather than queued. Analytics history is another split worth noting, capped at 30 days on Hobby and Startup and unlimited from Business up.

## What a Walmart price scraper has to get past

Four things make Walmart harder than a generic e-commerce target, and each of them shapes what you pay.

**Search results don't carry prices.** Apify's Walmart actor documentation states it plainly: search results return names, ratings and review counts but not price, UPC or category path, because those live on the individual product page. The actor has a `fetch_prices` switch that reads each product page to fill in `price`, `was_price`, `upc`, `brand`, `model` and `category_path`, and it reports `detail_fetched` so you know whether the page was actually read. Some category pages may render a listing price directly, so treat this as a rule about search result pages rather than about every Walmart list view.

**Prices are localized by where the request comes from.** Walmart serves different prices and availability depending on the source IP. In practice that means two runs from different egress points produce two numbers you can't compare, which is fatal for repricing. Oxylabs' request example pins `geo_location: 'United States'`; Apify stamps a `country` field on every row. Same lesson from both: fix the country and keep it fixed. On ScraperAPI geotargeting costs no extra credits but it is country-level only from Business upward, with Hobby and Startup limited to US and EU. If your baseline is Walmart Canada, check that against the tier you're buying.

**Bot protection sits in the path.** Walmart's defences include Akamai and PerimeterX, and store-level pricing generally needs JavaScript rendering to resolve. ScraperAPI prices that in: 10 extra credits per request when it bypasses Cloudflare, Turnstile, DataDome, PerimeterX/HUMAN or Akamai, and another 10 for `render=true`. A worst-case Walmart fetch therefore costs 25 credits, not 5.

**Walmart's own developer APIs won't help.** They're built for marketplace sellers, affiliates and suppliers and don't expose public search, review or best-seller data.

What ScraperAPI gives you for Walmart specifically is a structured product endpoint. You pass `product_id` (the number in the `/ip/` URL), optionally `country_code`, and `tld` for `com` or `ca`, and get JSON or CSV back without writing a parser. The async version accepts an array of `productIds` plus a webhook callback, which is the shape you want for a nightly batch of a few thousand SKUs.

## What a Walmart request costs in credits

ScraperAPI's credit table prices a standard page at 1 credit and e-commerce sites, Walmart included, at 5 credits per request. That converts cleanly:

- 100,000 credits (Hobby) = 20,000 Walmart fetches
- 1,000,000 credits (Startup) = 200,000 fetches
- 5,000,000 credits (Scaling) = 1,000,000 fetches

A daily check of 1,000 products is 30,000 requests a month, or 150,000 credits at 5 each. That clears Hobby and fits inside Startup with room to spare. Add bypass and rendering on every request and each fetch costs 25 credits, which pushes the same job to 750,000 credits a month, still inside Startup.

At the Scaling rate the per-request cost works out to about **$0.48 per 1,000 Walmart requests** on monthly billing, and about $0.43 on annual. Those figures assume the 5-credit rate and no add-ons, so verify against your own URLs: every response carries an `sa-credit-cost` header showing what the request actually consumed, the dashboard has a Domain Cost Estimator that quotes a URL before you scrape it, and `max_cost` caps any single request.

For a cost benchmark from a different billing model: Apify's Walmart actor charges $0.001 per run start, $0.001 per delivered row, and $0.003 per product page actually read, which its own example prices at $4.00 per 1,000 products with prices versus $1.00 per 1,000 without. Rows and requests aren't the same unit, so read that as an order-of-magnitude check rather than a quote.

One warning about testing: the free ScraperAPI plan is 1,000 credits, or 200 Walmart fetches at 5 credits each. Apify's free plan gives 10 rows per run. Either is enough to confirm the response shape and field names; neither will tell you what a production month costs.

👉 [Compare ScraperAPI's credit allowances against your request volume](https://scraperapi.com/pricing?fp_ref=coupons)

## Five field-level traps that turn a price report into fiction

The field names below come from Apify's published Walmart schema, the most explicit one I found, but the semantics apply to anything you build or buy.

**`was_price` only appears when the markdown is real.** It holds a value when the current price sits below a genuine original, and nothing otherwise. That makes its presence a usable discount signal, and its absence meaningful. Don't carry a stale `was_price` forward from a previous run and compute a discount off it.

**`price: null` is deliberate.** When a product page offers no price that clearly belongs to that product, a good scraper returns null instead of borrowing a nearby variant, bundle or "similar items" number. If your report averages a price column and treats null as zero, every average comes out systematically low.

**`sponsored` has to be filtered.** Paid placements are flagged so they don't blend into organic results. Leave them in and your "market price" for a keyword is partly an ad price.

**Best-seller rank isn't always a sales rank.** A chart run reports `sortApplied`; when it's false, a `chartNote` explains what the order actually is and `rank` is just shelf position. A merchandising shelf passed off as a sales ranking will send your assortment research in the wrong direction.

**Review pulls are capped.** The same actor reads up to 20 pages per run and reports `reviewsScanned` and `truncatedByPageBudget`. Without checking those, a 200-review sample looks like a product's entire review history.

One field worth using more: the UPC/GTIN from the product page is an exact join key against Amazon, Target or your own catalogue. Matching on titles is guesswork by comparison.

## Writing the parser yourself: what the tutorial leaves out

Oxylabs publishes a Walmart price tracker tutorial, and it's a fair picture of the DIY route. The parsing instructions are three XPaths against a category page: a title from `//span[@data-automation-id='product-title']`, a price from the `product-price` div's `span[@class='w_iUH7'][1]`, and `a/@href` for links, with `amount_from_string` converting the price text to a number.

Both of those attribute values are Walmart's own markup, hard-coded into your script. When the page structure changes, the script returns empty lists and you find out from a flat report. The repository behind the tutorial was created in January 2024 and carries 2 stars, 1 watcher, 0 forks and no releases; its published parse sample is dated November 2022 and ships with warnings that the parser couldn't determine shipping or seller info. Parsing successfully and getting complete fields are different things.

Then there's everything around the parse. The tutorial keeps history in a local JSON file keyed by product URL, diffs each run into `new_products` and `price_changed`, and sends the alert through `smtplib` with STARTTLS on port 587 and a Gmail app password. That's maybe 60 lines, all of which you own, monitor and fix. Budget for it before you compare a hosted per-request price against "free".

## Questions that come up before buying

**Why does my scraper return products without prices?** Because you're hitting a search URL. Names, ratings and review counts are on the results page; price, UPC and category path are on the product page. Turn on price fetching, or fetch each `/ip/` URL through the structured product endpoint.

**Why is the same product priced differently on two runs?** The request came from a different location, and Walmart localizes price and availability. Pin `country_code` on the structured endpoint or geo on the proxy, and keep the same value across runs.

**Can I buy reviews only?** ScraperAPI's published Walmart endpoint is product-level; reviews would need parsing from the product page. If reviews are the entire job, a tool with a dedicated reviews mode is the shorter route: Apify's takes a `product_id`, offers all six Walmart sorts plus a star filter, and returns up to 200 reviews per run within a 20-page budget.

**What does a null mean in the output?** For price, it means no price was attributable to that exact product. For a field like verified purchase, it can mean Walmart gave no evidence either way, which is not the same as false. Check the field definition before you filter on it.

**Will the free tier tell me if this works?** It'll tell you whether the endpoint returns the fields you need. It won't tell you what a month of daily runs costs.

## How to pick a tier

Match the plan to the job, not to the biggest number on the page.

**You just want to know what Walmart stocks for a keyword.** Search-page requests at 5 credits each, no product-detail reads. Hobby's 100,000 credits is 20,000 fetches, enough for a broad weekly sweep.

**You're repricing daily and need prices.** Count requests as products × runs. 1,000 products a day is 150,000 credits a month, so Startup. If bypass and rendering fire on every fetch, expect 750,000 and stay on Startup.

**You want review or best-seller history.** Different pipeline, different per-item pricing. Size it separately from the price job so one doesn't eat the other's credits.

Two things to confirm at checkout. First, whether your country scope matches the tier's geotargeting, since Hobby and Startup stop at US and EU. Second, whether your plan can burst past its allowance mid-month, because pay-as-you-go starts at Scaling and credits don't roll over. A repricing job that dies on the 28th of the month costs more in missed markdowns than the upgrade would.

👉 [Start with the ScraperAPI plan that matches your Walmart request volume](https://scraperapi.com/pricing?fp_ref=coupons)
