[Agoda Scraper](https://apify.com/solidcode/agoda-scraper?fpr=data)

Pull hotel and accommodation listings from Agoda at scale — names, addresses, coordinates, star ratings, guest scores, nightly prices, photos, amenities, descriptions, and check-in windows for any destination across 200+ countries. Built for travel aggregators, OTA market researchers, hospitality analysts, and travel-content teams who need fresh Agoda inventory and pricing without wrestling with city IDs, locale-prefixed URLs, and date-bound pagination.

## Why This Scraper?

- **Full Agoda catalog across 200+ countries** — search by destination keyword in any of **20 site languages** and **28 currencies**, including USD, EUR, GBP, JPY, KRW, IDR, AED, and THB.
- **Six sort modes** — Recommended, Price (low/high), Review score (high/low), and Distance from city center, so you can build a Cheapest-First feed or a Highest-Rated comparison without re-fetching.
- **5-tier star rating filter (1★–5★)** plus **8 property types** — Hotels, Apartments, Hostels, Resorts, Villas, Bed & Breakfasts, Guest Houses, and Vacation Homes — pick any subset, leave empty for everything.
- **Multi-occupancy with child ages** — rooms, adults, children, and per-child age list surface the same family-aware nightly pricing the Agoda website shows for that exact party.
- **Output `url` and `propertyId` chain straight into Agoda Reviews Scraper** — every row carries the canonical hotel URL plus numeric property ID, so a search → reviews pipeline is one input wire.
- **Optional `includeHotelDetails` enrichment** — flip one toggle to add full hotel description, named amenity categories, and check-in/check-out windows to every row in the same run.
- **Per-row guest-score band** with `reviewScore`, `reviewScoreLabel` (Exceptional / Excellent / Very good / Good / Fair / Below average), `reviewCount`, and a separate `minReviewScore` / `maxReviewScore` gate on input.
- **Discount detection** — when Agoda shows a strikethrough price, the row gets a calculated `discountPercent` so you can sort or alert on deals without parsing the UI.
- **Search by destination string OR by direct Agoda URL** — paste `https://www.agoda.com/en-gb/search?city=5085...` or any individual hotel page URL and the scraper auto-detects which endpoint to hit.

## Use Cases

**Travel Comparison & Aggregator Sites**

- Build city-by-city hotel inventories with current nightly rates
- Surface deals by sorting on `discountPercent` and `pricePerRoomPerNight`
- Power "best of" landing pages with real ratings, photos, and amenity lists

**Hospitality Market Research**

- Track hotel supply by star tier and property type across markets
- Compare average nightly rates by city, neighborhood, or country
- Monitor seasonal pricing curves with repeated date-window runs

**Competitive Pricing & Revenue Management**

- Benchmark your property against neighbors on Agoda for the same dates
- Detect competitor discounting by watching `discountPercent` changes
- Calibrate your own pricing windows by sorting on `price_low_to_high`

**Travel Content & Destination Guides**

- Generate "Top 25 hotels in Bangkok under $100" listicles with verified data
- Pull full descriptions, amenities, and photos for content-rich destination pages
- Cross-reference rooms with the **Agoda Reviews Scraper** to add guest sentiment

**Lead Generation & Sales**

- Build hotel sales prospect lists filtered by star rating and review count
- Target boutique-hotel chains by property type (Villas, Bed & Breakfasts, Guest Houses)
- Identify under-rated or under-reviewed properties for marketing-services pitches

## Getting Started

### Simple — One destination, one date window

```
{
    "search": ["hotels in Tokyo"],
    "checkIn": "2026-06-15",
    "checkOut": "2026-06-18",
    "maxResults": 100
}
```

### Filtered — 4★ and 5★ only, sorted cheapest first

```
{
    "search": ["Bangkok"],
    "checkIn": "2026-07-10",
    "checkOut": "2026-07-12",
    "starsCountFilter": ["4", "5"],
    "priceMin": 80,
    "priceMax": 250,
    "sortBy": "price_low_to_high",
    "minReviewScore": 8,
    "maxResults": 200
}
```

### Advanced — Multi-destination, family of four, full hotel detail enrichment, JPY pricing in Japanese

```
{
    "search": ["Kyoto", "Osaka", "Hakone"],
    "checkIn": "2026-09-20",
    "checkOut": "2026-09-25",
    "rooms": 1,
    "adults": 2,
    "children": 2,
    "childrenAges": ["5", "8"],
    "propertyType": ["hotels", "resorts"],
    "starsCountFilter": ["4", "5"],
    "sortBy": "review_high_to_low",
    "language": "ja-jp",
    "currency": "JPY",
    "includeHotelDetails": true,
    "maxResults": 300
}
```

## Input Reference

### What to Scrape

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `search` | string[] | `["hotels in Tokyo"]` | Destination keywords. Each entry runs as its own Agoda search. Use this OR `startUrls`. |
| `startUrls` | string[] | `[]` | Paste Agoda search-result URLs or individual hotel page URLs. Auto-detects the URL type. Same field shape as **Agoda Reviews Scraper** for easy chaining. |

### Dates & Guests

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `checkIn` | string (YYYY-MM-DD) | empty | Check-in date. Required to fetch live nightly prices — leave empty for catalog data only. |
| `checkOut` | string (YYYY-MM-DD) | empty | Check-out date. Must be after `checkIn`. |
| `rooms` | integer | `1` | Number of rooms required (1–9). |
| `adults` | integer | `2` | Number of adult guests (1–20). Agoda counts age 18+ as an adult. |
| `children` | integer | `0` | Number of child guests (0–10), ages 0–17. |
| `childrenAges` | string[] | `[]` | Age of each child guest. Required when `children` > 0 — Agoda prices family rooms based on these ages. Example: `["5", "8"]`. |

### Filters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `propertyType` | select[] | empty | Restrict to one or more accommodation types: Hotels, Apartments, Hostels, Resorts, Villas, Bed & Breakfasts, Guest Houses, Vacation Homes. |
| `starsCountFilter` | select[] | empty | Only include properties with these official star ratings (1 star – 5 stars). Empty = any. |
| `minReviewScore` | integer | `0` | Minimum guest review score on Agoda's 0–10 scale. |
| `maxReviewScore` | integer | `10` | Maximum guest review score on Agoda's 0–10 scale. |
| `priceMin` | integer | empty | Minimum nightly price in the chosen currency. Empty = no minimum. |
| `priceMax` | integer | empty | Maximum nightly price in the chosen currency. Empty = no maximum (recommended for high-denomination currencies like JPY, KRW, IDR). |

### Sort & Localization

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `sortBy` | select | `Recommended` | Recommended, Price (lowest first), Price (highest first), Review score (highest first), Review score (lowest first), Distance from city center. |
| `language` | select | `English (UK)` | Agoda site language (locale). Affects descriptions and amenity labels. 20 options: English (UK/US), German, French, Spanish, Italian, Dutch, Portuguese, Polish, Russian, Japanese, Korean, Chinese (Simplified/Traditional), Thai, Vietnamese, Indonesian, Malay, Arabic, Turkish. |
| `currency` | select | `US Dollar` | Currency code (ISO 4217). 28 options including USD, EUR, GBP, JPY, KRW, IDR, AED, THB. Forces all prices into this currency regardless of the locale default. |

### Options

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `maxResults` | integer | `100` | Maximum hotels to return per destination/URL. Set to `0` for all available. The last page is kept whole — the scraper stops asking for new pages once the cap is reached but never truncates mid-page. A single Agoda search holds anywhere from a few hundred to several thousand listings depending on the destination (Bangkok ≈ 4,000, Tokyo ≈ 900). For very large destinations, split the query by neighborhood, star tier, or property type to go beyond what one search can return. |
| `includeHotelDetails` | boolean | `false` | Fetch the hotel detail page for every result and add full description, amenities, and check-in/check-out times to each row. Roughly doubles the row cost — leave off for fast catalog runs. |

## Output

### Hotel row (always)

```
{
    "propertyId": 1234567,
    "name": "The Peninsula Tokyo",
    "url": "https://www.agoda.com/en-gb/the-peninsula-tokyo/hotel/tokyo-jp.html",
    "propertyType": "Hotel",
    "image": "https://q-xx.bstatic.com/xdata/images/hotel/max1024x768/12345.jpg",
    "address": {
        "country": "Japan",
        "countryCode": "JP",
        "city": "Tokyo",
        "area": "Marunouchi",
        "street": "",
        "postalCode": ""
    },
    "location": { "lat": 35.6762, "lng": 139.7649 },
    "rating": 5,
    "reviewCount": 1842,
    "reviewScore": 9.2,
    "reviewScoreLabel": "Exceptional",
    "pricePerRoomPerNight": 920,
    "pricePerBook": 2760,
    "priceCurrency": "USD",
    "requestedCurrency": "USD",
    "discountPercent": 15,
    "freeCancellation": true,
    "breakfastIncluded": null,
    "amenities": [],
    "description": null,
    "checkInTime": null,
    "checkOutTime": null,
    "searchQuery": "hotels in Tokyo",
    "searchUrl": "https://www.agoda.com/en-gb/search?city=14266"
}
```

### Core Fields

| Field | Type | Description |
| --- | --- | --- |
| `propertyId` | number | Numeric Agoda hotel ID. Drops directly into the **Agoda Reviews Scraper** `hotelIds` input. |
| `name` | string | Hotel name in the requested locale. |
| `url` | string | Canonical Agoda hotel URL. Drops directly into the **Agoda Reviews Scraper** `startUrls` input. |
| `propertyType` | string | Property type label (Hotel, Resort, Apartment, etc.). |
| `image` | string | Primary hotel photo URL (https-normalized). |
| `rating` | number | Official star rating (1–5). |
| `searchQuery` | string | The destination string that produced this row. |
| `searchUrl` | string | The Agoda search URL that produced this row. |

### Address & Location

| Field | Type | Description |
| --- | --- | --- |
| `address.country` | string | Country name. |
| `address.countryCode` | string | ISO country code. |
| `address.city` | string | City name. |
| `address.area` | string | Neighborhood / district name. |
| `address.street` | string | Street address (populated when `includeHotelDetails` is on). |
| `address.postalCode` | string | Postal code (populated when `includeHotelDetails` is on). |
| `address.full` | string | Full formatted address (populated when `includeHotelDetails` is on). |
| `location.lat` | number | Latitude. |
| `location.lng` | number | Longitude. |

### Ratings & Pricing

| Field | Type | Description |
| --- | --- | --- |
| `reviewCount` | number | Total guest reviews on Agoda. |
| `reviewScore` | number | Guest review score on Agoda's 0–10 scale. |
| `reviewScoreLabel` | string | Bucketed label: Exceptional, Excellent, Very good, Good, Fair, Below average. |
| `pricePerRoomPerNight` | number | Nightly rate in `priceCurrency`. |
| `pricePerBook` | number | Total stay price in `priceCurrency`. |
| `priceCurrency` | string | Currency Agoda actually returned (some locales force a substitution). |
| `requestedCurrency` | string | Currency the input asked for. Compare with `priceCurrency` to detect overrides. |
| `discountPercent` | number | Percent off the strikethrough price, when Agoda displays one. |
| `freeCancellation` | boolean | Whether the surfaced offer includes free cancellation. |

### Detail Enrichment (only when `includeHotelDetails: true`)

| Field | Type | Description |
| --- | --- | --- |
| `description` | string | Full hotel description in the requested locale. |
| `amenities` | string[] | Named amenity categories (e.g. "Wellness", "Food and drink", "Outdoors"). |
| `images` | string[] | Up to 12 gallery photo URLs for the hotel (https-normalized). |
| `checkInTime` | string | Stated check-in window (e.g. "From 15:00"). |
| `checkOutTime` | string | Stated check-out window (e.g. "Until 11:00"). |

## Tips for Best Results

- **Pair this scraper with [Agoda Reviews Scraper](https://apify.com/solidcode/agoda-reviews-scraper)** — feed each row's `url` straight into the reviews scraper's `startUrls` (or `propertyId` into `hotelIds`) for end-to-end hotel + guest-feedback pipelines in two clicks.
- **Always pass `checkIn` and `checkOut` when you need prices** — without dates the catalog data still comes through (names, ratings, addresses, photos) but `pricePerRoomPerNight` will be empty, since Agoda only quotes rates against a real date window.
- **Run multiple date windows for maximum city coverage** — Agoda's available inventory shifts with seasonality and last-minute releases, so three runs across three weekends will pick up properties a single run misses.
- **Match `currency` to your audience, not the destination** — set `currency: "USD"` for a US-facing dashboard even when scraping Tokyo or Bangkok; Agoda will convert and surface the rate.
- **Leave `priceMax` empty for high-denomination currencies** (JPY, KRW, IDR, VND) unless you really mean to filter — a `priceMax: 1000` against JPY will exclude almost everything.
- **Use `sortBy: "review_high_to_low"` with `minReviewScore: 8`** to build a "best in city" list without sifting through low-rated properties.
- **Children pricing requires `childrenAges`** — Agoda quotes family rooms based on each child's age. A `children: 2` input without ages will fall back to adult-priced occupancy.

## Pricing

**$2.00 per 1,000 results** — flat. No compute charges — you only pay per result returned.

| Results | Estimated Cost |
| --- | --- |
| 100 | $0.20 |
| 1,000 | $2.00 |
| 10,000 | $20.00 |
| 100,000 | $200.00 |

A "result" is one hotel row in the output dataset. Enabling `includeHotelDetails` does not change the per-row price — it only changes how many fields each row contains.

## Integrations

Export data in JSON, CSV, Excel, XML, or RSS. Connect to 1,500+ apps via:

- **Zapier** / **Make** / **n8n** — Workflow automation
- **Google Sheets** — Direct spreadsheet export
- **Slack** / **Email** — Notifications on new results
- **Webhooks** — Trigger custom APIs on run completion
- **Apify API** — Full programmatic access
- **Agoda Reviews Scraper** — Pipe each `url` straight in for a complete hotel + reviews dataset

## Legal & Ethical Use

This actor is designed for legitimate travel research, market analysis, content publishing, and competitive benchmarking. Users are responsible for complying with applicable laws and Agoda's Terms of Service. Do not use extracted data for spam, price manipulation, or any illegal purpose. Respect Agoda's published rate parity rules when republishing pricing data.