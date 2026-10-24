# user agent header for scraping: how to set it, rotate it, and when to let your scraper API handle headers instead

The user agent header is the single cheapest thing you can fix when a scraper starts getting blocked, and in most cases you should send a current, common browser string — a recent Chrome on Windows or macOS is the safest default. But there's a catch most tutorials skip: if you route requests through a scraping API like ScraperAPI, sending your own user agent can actually override the provider's optimized header system and lower your success rate. This guide covers both paths — setting and rotating the header yourself, and letting the API do it.

> **Quick take:** ScraperAPI includes desktop and mobile user agents, custom header support, and rotating residential IPs on every paid plan, starting at **$49/month for 100,000 API credits** (about $44/month billed annually). There's a free plan with 1,000 credits per month and a 7-day trial with 5,000 credits, no credit card required — 👉 [start the free trial and test it against your real targets](https://www.scraperapi.com/?fp_ref=coupons)

## What the user agent header actually does

It's one line of text that tells the server which client is asking. When Chrome loads a page, the request carries something like `User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...`, and the server reads that string to log traffic, serve the right layout, and — this is the part scrapers care about — decide whether the visitor looks human.

The string packs several facts into one line: browser engine, browser version, operating system, sometimes the device type. That makes it the first and cheapest signal a site can check. It's also the cheapest one you control, which is why it's usually the first thing worth changing when requests start failing.

## Why default user agents get blocked instantly

Because they announce the tool by name. If you call `requests.get(url)` in Python without setting headers, the library sends `User-Agent: python-requests/2.34.2`. cURL sends `curl/8.x`. A site can filter that traffic with a single substring match — no fingerprinting, no machine learning, just a text filter.

The fix is straightforward: put a real browser string in a headers dictionary and pass it with the request.

python
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/120.0.0.0 Safari/537.36"
}
r = requests.get("https://example.com", headers=headers)


The same `headers` argument works whether you parse the response with BeautifulSoup or feed it into a Scrapy spider, and cURL uses the same idea: `curl -H "User-Agent: Mozilla/5.0 ..." https://example.com`.

One habit worth picking up early: use a current browser version. A Chrome string from two years ago is itself a signal — real users update their browsers, and stale versions show up disproportionately in bot traffic.

## Rotating user agents: what it does and what it doesn't

Rotation means keeping a small pool of current browser strings and picking a different one per request, usually with `random.choice` from a list. This breaks up a uniform fingerprint — hundreds of identical requests from one machine look obviously automated, even if the string itself is a valid browser.

But rotation alone only gets you past basic checks. Sites that fingerprint traffic look at much more than one header:

| Signal | What gives a scraper away | Typical mitigation |
| --- | --- | --- |
| Other headers | Missing `Accept`, `Accept-Language`, or `Referer` that real browsers always send | Send a full, consistent header set |
| IP address | Many requests from one IP, or a known datacenter range | Rotate residential or mobile IPs |
| Request rate | Hundreds of evenly spaced hits per minute | Add delays and jitter |
| TLS fingerprint | The TLS handshake identifies Python, not Chrome | Use a client that mimics browser TLS |
| JavaScript | Page needs JS to render and your client runs none | Headless browser or a rendering API |

The consistency point matters more than most guides make it sound. If your User-Agent claims to be Chrome on Windows, your `Accept-Language` and `Sec-CH-UA` headers should match what a real Chrome request would send. A Chrome user agent paired with a Python TLS handshake is a contradiction that fingerprinting systems catch easily. Each additional signal you fake correctly means maintaining more logic yourself — which is exactly the point where many people decide to hand the whole problem to a scraping API.

## When you shouldn't send your own user agent header at all

If you're using a scraping API, sending your own user agent can backfire. This is the piece that the generic "rotate your UA" tutorials don't cover, and it's directly relevant to how ScraperAPI works.

ScraperAPI maintains a header management system internally, tuned to raise success rates on difficult sites. If you pass `keep_headers=true` and attach your own headers, the API forwards your headers to the target site — but in doing so you override that internal system. The official documentation is blunt about this: "When you send your own custom headers you override our header system, which oftentimes lowers your success rates." Their advice is to use custom headers only when you genuinely need a specific header value to get the data you want, such as a session cookie or a particular Referer.

Two limitations worth knowing before you write any code:

- Custom headers are **completely unsupported** when `ultra_premium=true` is enabled. Even with `keep_headers=true` set, all custom headers get discarded on ultra-premium requests — the parameter is silently ignored.
- If you want mobile results, don't construct a mobile user agent manually. Use the `device_type=mobile` parameter instead, and ScraperAPI sets the mobile user agent for you as part of its normal header handling.

For most scraping work, the API's default header behavior — including desktop and mobile user agents, which come with every plan — is the better path. If that matches what you need, 👉 [compare the plans and see what's included at each tier](https://www.scraperapi.com/pricing?fp_ref=coupons).

If you do need to send a specific header, the mechanics are simple. In API mode:


curl --request GET \
  --header "X-MyHeader: 123" \
  --url 'http://api.scraperapi.com/?api_key=API_KEY&keep_headers=true&url=http://example.com'


In proxy mode, you pass the option through the proxy username: `scraperapi.keep_headers=true:APIKEY@proxy.scraperapi.com:8001`.

## ScraperAPI plans: full comparison

Every paid plan includes the same core feature set: JS rendering, premium residential and mobile IPs, custom header support, CAPTCHA and anti-bot detection, desktop and mobile user agents, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. The plans differ only in four things — credits, concurrent threads, geotargeting granularity, and how long the analytics dashboard keeps data.

| Plan | API credits | Concurrent threads | Monthly price | Annual (per month) | Get started |
| --- | --- | --- | --- | --- | --- |
| Free | 1,000/mo | 5 | $0 | — | [Free signup](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | 100,000 | 20 | $49 | $44.10 | [Start Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | $149 | $134.10 | [Start Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | $299 | $269.10 | [Start Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | $475 | $427.50 | [Start Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | $975 | $877.50 | [Start Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | $1,975 | $1,777.50 | [Start Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ (custom) | Custom quote | — | [Contact sales](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Details the table can't show: Hobby and Startup are limited to US & EU geotargeting, while Business and up support country-level targeting. The analytics dashboard keeps 30 days of history on Hobby and Startup, unlimited from Business up. Annual billing takes 10% off every tier.

## What a "credit" actually costs you

Here's something the pricing page headline hides and most reviews never mention: one API credit is the cost of a standard page request, but many popular targets cost more. A standard page costs 1 credit. Amazon costs 5. Google and Bing, including their subdomains, cost 25. LinkedIn costs 30. And sites behind bot protection like Cloudflare, Datadome, or PerimeterX add 10 credits per request when ScraperAPI bypasses them.

The practical consequence: your real request count depends heavily on what you're scraping. A million "credits" is a million standard pages, but only 40,000 Google requests. ScraperAPI provides a Domain Cost Estimator in the dashboard so you can check the exact cost for any URL before committing, and you can set a `max_cost` per scrape to cap how much any single request consumes.

This also means you can estimate your plan needs before paying. If you know you're scraping 50,000 product pages a month on Amazon, that's 250,000 credits — comfortably within Hobby territory but well past the free plan. 👉 [Use the pricing page to match your monthly volume against each tier](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Running out of credits, refunds, and other things that affect the bill

A few billing details only appear in the FAQ, and they're worth knowing before you pick a plan:

- **Monthly plans (Student through Professional):** if you hit 100% of credits before renewal, you can turn on auto-upgrade, which moves you to the next tier automatically — usually at a better price per credit — or contact support for a custom plan.
- **Annual plans (Hobby through Professional):** if you burn through your annual credits early, you can re-subscribe on your next billing date for a fresh batch, or switch to a higher monthly plan.
- **Scaling, Professional, Advanced, Enterprise:** these include Pay-As-You-Go, letting you keep scraping past your limit at a fixed per-credit rate. You can set a monthly spending cap, and the service will not exceed it unless you change the setting yourself. It's a spending limit, not a pre-charge.
- **Refunds:** there's a 7-day, no-questions-asked refund policy. You cancel anytime from the dashboard at no charge.

The refund policy removes most of the risk from an annual commitment: pay annually for the 10% discount, and if the service doesn't work against your specific targets within a week, you get the money back.

## Does changing the user agent header still matter if the API handles headers?

Yes, but in a different role. When ScraperAPI manages headers, IPs, retries, and CAPTCHA handling, the user agent stops being something you engineer and becomes something the system handles — which is generally what you want for sites with real anti-bot protection, since those sites check TLS fingerprints and header consistency that hand-rolled rotation can't fake anyway.

The manual approach still makes sense for lighter targets: a site with no meaningful bot protection doesn't need residential IPs or JS rendering, and a simple headers dictionary costs nothing. Paying for a scraping API to fetch pages that respond fine to plain `requests` calls is wasted spend.

## FAQ

**What should I put in the user agent header for scraping?**
A current, common real browser string. A recent Chrome on Windows or macOS is the safest default because it blends into the largest share of live traffic. Avoid stale browser versions and obscure browsers, and pair the string with the full header set a real browser would send.

**Is rotating the user agent enough to avoid blocks?**
No. It breaks up a uniform fingerprint but only gets past basic checks. Sites that fingerprint traffic also examine missing browser headers, IP reputation, request timing, and TLS handshakes.

**How do I send a custom user agent through ScraperAPI?**
Add `keep_headers=true` to your request and attach the headers you want. The API forwards them to the target site. Note that custom headers are discarded entirely on ultra-premium requests, and the docs recommend avoiding custom headers unless you need a specific value.

**Is there a free option to test before paying?**
Yes. ScraperAPI has a permanent free plan with 1,000 credits per month and 5 concurrent connections, plus a 7-day trial with 5,000 credits at signup, no credit card required. Support can issue additional testing credits if you ask.

## Which route should you take

If your targets are plain pages with no serious bot protection, a browser user agent string in a headers dictionary is all you need — spend nothing, done.

If you're scraping Amazon, Google, LinkedIn, or anything sitting behind Cloudflare or similar systems, hand-rolled header rotation won't be enough; you need residential IPs, JS rendering, and consistent header management, and ScraperAPI bundles all of that into every plan. Start on the free tier to validate against your real targets — the Domain Cost Estimator will tell you what your monthly volume actually costs in credits. If it works, the annual rate saves 10% and the 7-day refund policy covers the first week either way: 👉 [sign up and claim your 5,000 free trial credits](https://www.scraperapi.com/?fp_ref=coupons)
