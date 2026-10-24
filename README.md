# AutoScout24 Scraper — car listings from every AutoScout24 country site, through the Apify API

Get AutoScout24 car listings into your own code as structured JSON: price, AutoScout24's
price rating, specs, equipment, every photo and the dealer's contact details, from
autoscout24.com, .de, .it, .fr, .es, .nl and the other country sites. The scraper runs on
[Apify](https://apify.com) over plain HTTP, so there's nothing to host and no browser to run.

**▶ Run it on Apify: [AutoScout24 Scraper](https://apify.com/alkausari_mujahid/autoscout24-scraper)**

This repository holds documentation and examples for developers. The scraper itself runs
on the Apify platform.

---

## Scrape AutoScout24 with one HTTP request

Paste any AutoScout24 search URL from your browser, filters included. This call returns the
cars in the same response:

```bash
curl -X POST \
  "https://api.apify.com/v2/acts/alkausari_mujahid~autoscout24-scraper/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "startUrls": [{ "url": "https://www.autoscout24.com/lst/ford/kuga?atype=C&sort=standard" }],
    "maxResults": 20
  }'
```

Your API token is under **Settings → API & Integrations** in [Apify Console](https://console.apify.com).
Add `&format=csv` to the URL for a spreadsheet instead of JSON. The synchronous endpoint
waits up to 300 seconds; for larger runs, start the run and fetch the dataset when it
finishes, as below.

## From Python

```bash
pip install apify-client
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("alkausari_mujahid/autoscout24-scraper").call(run_input={
    "startUrls": [{"url": "https://www.autoscout24.de/lst/bmw?atype=C&cy=D&sort=standard"}],
    "maxResults": 200,
    "forceEnglish": True,
})

for car in client.dataset(run["defaultDatasetId"]).iterate_items():
    rating = (car.get("priceRating") or {}).get("rating")
    print(car["title"], car["price"]["total"]["amount"], rating)
```

## From JavaScript

```bash
npm install apify-client
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

// Search results only: no detail pages, so it's fast and cheap for price tracking.
const run = await client.actor('alkausari_mujahid/autoscout24-scraper').call({
    startUrls: [{ url: 'https://www.autoscout24.com/lst/ford/kuga?atype=C&sort=standard' }],
    maxResults: 500,
    scrapeDetails: false,
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const car of items) console.log(car.title, car.price.total.amount, car.mileage, car.location);
```

---

## Input

| Field | Default | What it does |
|---|---|---|
| `startUrls` | — | AutoScout24 search pages (URLs containing `/lst/`), car detail pages, or a mix, from any country site. |
| `maxResults` | `60` | Maximum cars to return across all start URLs. `0` means no limit. |
| `scrapeDetails` | `true` | Open each car's page for full specs, equipment, every photo, the price rating and dealer contacts. Turn off to keep only what the search results show; direct car URLs are then skipped. |
| `forceEnglish` | `false` | Read each car from autoscout24.com, so values come back in English whatever site the listing is on. |
| `proxyConfiguration` | none | Runs work without a proxy. Turn on Apify Proxy if AutoScout24 starts blocking. |

## Output

One record per car. This one was scraped with full details on; the dealer's phone numbers,
email and WhatsApp are replaced, and long lists shortened:

```json
{
    "id": "6cb47ddd-d97a-495d-984d-65069cbd4c6d",
    "title": "Ford Kuga 1.5 EcoBoost 150 CV 2WD ST-Line",
    "url": "https://www.autoscout24.com/offers/ford-kuga-1-5-ecoboost-150-cv-2wd-st-line-gasoline-black-6cb47ddd-d97a-495d-984d-65069cbd4c6d",
    "vehicleType": "Car",
    "bodyType": "SUV/Off-Road/Pick-Up",
    "brand": "Ford",
    "model": "Kuga",
    "modelVersion": "1.5 EcoBoost 150 CV 2WD ST-Line",
    "price": { "total": { "amount": 22500, "currency": "EUR", "formatted": "€ 22,500" } },
    "priceRating": null,
    "createdDate": "2026-03-10T09:50:03.496Z",
    "attributes": {
        "Vehicle condition": "Used",
        "Mileage": "15,311 km",
        "Power": "110 kW",
        "Fuel": "Gasoline",
        "Transmission": "Manual",
        "First Registration": "01/2023",
        "Colour": "Black",
        "Number of Vehicle Owners": 1,
        "CO₂ emissions": "149 g/km (comb.)"
    },
    "features": ["Air conditioning", "Cruise control", "Navigation system", "Park Distance Control"],
    "images": ["https://prod.pictures.autoscout24.net/listing-images/6cb47ddd-d97a-495d-984d-65069cbd4c6d_ad34af70-1a6b-42b5-8aff-485a9834cf22.jpg/2560x1920.webp"],
    "imageCount": 22,
    "location": "Livorno - LI",
    "rank": 7,
    "searchUrl": "https://www.autoscout24.com/lst/ford/kuga?atype=C&sort=standard",
    "dealerName": "Blubay Spa",
    "dealerRating": "5",
    "dealerRatingsCount": "1086 Ratings",
    "dealerDetails": {
        "id": 15529411,
        "name": "Blubay Spa",
        "sellerType": "Dealer",
        "phones": [{ "type": "Office", "number": "+39 0000 000000" }],
        "email": "dealer@example.com",
        "website": "www.blubayspa.it",
        "whatsapp": "0000 000000",
        "address": "Via Enriques, 40, 57121, Livorno - LI, IT",
        "addressStructured": { "street": "Via Enriques, 40", "city": "Livorno - LI", "countryCode": "IT", "zip": "57121" },
        "rating": { "averageScore": 4.8, "count": 1086 }
    }
}
```

The full record also carries the description, every attribute and feature, and the
dealer's logo, review page and active-since date. With `scrapeDetails` off, each record is
a summary instead: price, mileage, first registration, fuel, transmission, power, location,
country, seller type and dealer name.

### The price rating

When AutoScout24 has enough comparable cars, `priceRating` holds its verdict on the price,
from "Very good price" to "Extremely high price", plus the price boundaries of every band:

```json
{
    "rating": "Very good price",
    "priceRanges": {
        "Very good price": { "start": null, "end": 34300 },
        "Good price": { "start": 34301, "end": 38200 },
        "Fair price": { "start": 38201, "end": 39600 },
        "Increased price": { "start": 39601, "end": 42500 },
        "High price": { "start": 42501, "end": 46000 },
        "Very high price": { "start": 46001, "end": 48900 },
        "Extremely high price": { "start": 48901, "end": null }
    }
}
```

It's `null` when there are too few comparable cars to rate the price, as in the record above.

---

## Things to design around

**Each search URL tops out at 4,000 cars.** AutoScout24 serves 200 pages of 20 per search.
To go further, split one search into narrower ones, for example by model, year or price
range, and pass them all in `startUrls`. A car found by several searches is returned once.

**Attribute keys are always English; values follow the site.** `attributes` uses fixed keys
like `"Mileage"` and `"First Registration"`, so mappings hold across countries, but a German
listing says `"Benzin"` where an English one says `"Gasoline"`. Set `forceEnglish: true` for
English values. Four keys, `HU`, `Climatisation`, `Parking sensors` and `Airbags`, only
appear for cars read in English.

**Search-only records keep the site's language**, even with `forceEnglish` on, since they're
taken straight from the search page.

**Use the numeric dealer rating.** `dealerRating` and `dealerRatingsCount` are display text
(`"5"`, `"1086 Ratings"`); `dealerDetails.rating` has the same as numbers.

**`description` contains HTML**, such as `<strong>`, exactly as the seller wrote it.

**`rank` and `searchUrl` are `null` for cars passed as direct detail URLs.** For cars found
through a search, they give the position and the search it came from.

**Deduplicate on `id`.** It's AutoScout24's own listing ID, stable across runs, so it also
lets you match today's prices against yesterday's.

**Removed listings are skipped.** A car that's no longer online isn't returned.

---

## Pricing

Pay per car returned. Full details adds a per-car charge for opening each car's own page;
the current rates are on the Actor's
[Pricing tab](https://apify.com/alkausari_mujahid/autoscout24-scraper/pricing). Car pages that
can't be read aren't charged.

Set **Maximum cost per run** in the run options and the scraper works out how many cars that
covers, then stops there.

## Use it from an AI assistant

Apify's MCP server lets Claude, Cursor and other MCP clients run this scraper for you. Add
this server URL to your client:

```text
https://mcp.apify.com?tools=alkausari_mujahid/autoscout24-scraper
```

Then ask in plain words, for example "find used Ford Kugas in Italy under €20,000 with a good
price rating".

## Schedules and integrations

On Apify you can run a search on a schedule, get a webhook when a run finishes, and send
results to Google Sheets, Zapier, Make, Slack or your own endpoint. All of it is set up from
the [Actor page](https://apify.com/alkausari_mujahid/autoscout24-scraper).

---

## Support

Questions, bugs or feature requests: open an issue in the
[Actor's Issues tab](https://apify.com/alkausari_mujahid/autoscout24-scraper/issues), or
email [alkausarimujahid@gmail.com](mailto:alkausarimujahid@gmail.com).

*This is an independent tool. It is not affiliated with, endorsed by or sponsored by
AutoScout24. "AutoScout24" is a trademark of its owner and is used here only to describe the
data source.*
