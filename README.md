# AI B2B Lead Qualification & Google Maps Scraper

An end-to-end automated lead generation pipeline that scrapes local businesses from Google Maps, qualifies each lead using AI, generates personalized outreach messages, and exports everything to Google Sheets — with zero manual effort.

> Built with **n8n · Apify · Mistral AI · Google Sheets**

---

## Who Is This For?

Any business or freelancer that needs a repeatable, automated way to find and qualify local B2B leads — without paying for expensive CRMs or lead databases.

### Industries This Works For

| Sector | Example Search |
|--------|---------------|
| Construction & BTP | General contractors, roofers, builders |
| Real Estate | Agencies, property developers, brokers |
| Healthcare | Clinics, dental offices, physiotherapy centers |
| Hospitality | Hotels, restaurants, event venues |
| Retail & E-commerce | Boutiques, wholesalers, distributors |
| Legal & Finance | Law firms, accounting offices, notaries |
| Education & Training | Schools, coaching centers, language institutes |
| Logistics & Transport | Freight companies, couriers, warehousing |
| Manufacturing | Factories, suppliers, industrial companies |
| IT & Technology | Software agencies, IT service providers |
| Beauty & Wellness | Salons, spas, fitness studios |
| Automotive | Garages, dealerships, fleet services |

**If it has a Google Maps listing, this automation can find it, qualify it, and prepare your outreach.**

---

## What It Does

```
One click → up to 100 (or 1000+) AI-qualified leads → Google Sheets → Ready for outreach
```

The number of results per run is fully configurable. Set to **25** by default for free-tier testing, but a single parameter change scales it to **100, 500, or 1000+** leads per run — no code changes needed.

For each business found on Google Maps, the workflow:
1. Extracts contact info (name, address, phone, website, category, rating)
2. Sends data to Mistral AI for lead qualification (fit score 1–5)
3. Generates a personalized outreach message per lead
4. Appends results to Google Sheets with deduplication

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        n8n Workflow                             │
│                                                                 │
│  Manual Trigger                                                 │
│       ↓                                                         │
│  Set Variables ──── city, industry keyword, country             │
│       ↓                                                         │
│  HTTP Request ───── Apify Google Maps Actor (sync)              │
│       ↓             Returns 25 businesses with full details     │
│  Code Node ──────── Extract & clean fields                      │
│       ↓                                                         │
│  AI Agent ───────── Mistral AI qualification                    │
│       ↓             fit_score + fit_reason + outreach_message   │
│  Code Node ──────── Merge company data + AI output              │
│       ↓                                                         │
│  Google Sheets ──── Append or Update (no duplicates)            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Output — Google Sheet Columns

| Column | Description |
|--------|-------------|
| `company_name` | Business name from Google Maps |
| `city` | Target city |
| `address` | Full street address |
| `phone` | Phone number |
| `email` | Email (when available) |
| `website` | Company website |
| `category` | Google Maps business category |
| `rating` | Star rating (1–5) |
| `reviews_count` | Number of Google reviews |
| `fit_score` | AI score 1–5 (5 = perfect lead) |
| `fit_reason` | One-line AI explanation of the score |
| `outreach_message` | Personalized message ready to send |
| `status` | New / Contacted / Demo Booked / Closed Won |

---

## Tech Stack

| Tool | Role | Cost |
|------|------|------|
| [n8n](https://n8n.io) | Workflow automation engine | Free (self-hosted) |
| [Apify](https://apify.com) — `compass/crawler-google-places` | Google Maps scraping | Free tier available |
| [Mistral AI](https://mistral.ai) — `mistral-small-latest` | AI lead qualification | Very low cost |
| [Google Sheets](https://sheets.google.com) | Lead CRM & output | Free |

---

## Setup Guide

### Prerequisites
- n8n running locally (`localhost:5678`) or on a cloud server
- Apify account + API token ([get one here](https://console.apify.com/account/integrations))
- Mistral AI API key ([get one here](https://console.mistral.ai/api-keys/))
- Google account for Sheets OAuth2

### Step 1 — Import the Workflow
1. Open n8n
2. Click the top-right **⋮** menu → **Import from file**
3. Select `workflow.json` from this repo

### Step 2 — Add Your Credentials

**Apify token** — in the HTTP Request node, replace `YOUR_APIFY_API_TOKEN` in the URL:
```
https://api.apify.com/v2/acts/compass~crawler-google-places/run-sync-get-dataset-items?token=YOUR_APIFY_API_TOKEN
```

**Mistral AI** — in n8n Credentials, add a new "Mistral Cloud" credential with your API key.

**Google Sheets** — in n8n Credentials, add a "Google Sheets OAuth2" credential and authenticate.

### Step 3 — Configure Your Target

In the **Edit Fields** node, set:
```
city           →  e.g. "London", "Dubai", "New York"
sector_keyword →  e.g. "dentist", "construction", "logistics"
country        →  e.g. "United Kingdom", "UAE", "United States"
```

In the **HTTP Request** node body, update:
```json
{
  "searchStringsArray": ["YOUR_INDUSTRY_KEYWORD YOUR_TARGET_CITY"],
  "maxCrawledPlacesPerSearch": 25,   ← change this to 100, 500, or 1000+
  "language": "YOUR_LANGUAGE_CODE",
  "countryCode": "YOUR_COUNTRY_CODE"
}
```

### Step 4 — Set Up Google Sheet

Create a new Google Sheet with one tab and these column headers in row 1:
```
company_name | city | address | phone | email | website | category | rating | reviews_count | fit_score | fit_reason | outreach_message | status
```

- Format `phone` and `email` columns as **Plain Text**
- Add a dropdown to `status` column: `New, Contacted, Demo Booked, Not Interested, Closed Won`

In the **Google Sheets** node, select your sheet and tab, then set `company_name` as the column to match on.

### Step 5 — Customize the AI Prompt

In the **AI Agent** node, edit the prompt to describe:
- What product or service you are selling
- What makes a business a good lead for you
- The language and tone for outreach messages

### Step 6 — Run
Click **Execute Workflow** — results appear in your Google Sheet in ~2 minutes.

---

## Scaling Strategy

### Results Per Run
Change `maxCrawledPlacesPerSearch` in the HTTP Request node body to control how many leads you get:

| Value | Use case |
|-------|----------|
| 25 | Free tier / local testing |
| 100 | Standard prospecting run |
| 500 | City-wide sweep |
| 1000+ | Large-scale market coverage |

> Higher values = longer runtime. 100 leads takes ~5–8 minutes. Plan accordingly.

### City Rotation
Change the city each run to expand your lead pool without duplicates:
```
Run 1 → City A
Run 2 → City B
Run 3 → City C
```
Works globally — any city in any country.

### Deduplication
The Google Sheets node uses **Append or Update Row** with `company_name` as the match key. Running the same search twice updates existing rows instead of creating duplicates.

### Targeting by Rating
Leads with higher Google ratings (4.0+) and more reviews typically indicate more established businesses. Use the `fit_score` and `rating` columns to prioritize outreach.

---

## AI Fit Score Guide

| Score | Meaning | Action |
|-------|---------|--------|
| 5 | Perfect fit — high priority | Contact immediately |
| 4 | Strong fit — good prospect | Contact this week |
| 3 | Moderate fit — worth exploring | Contact if capacity allows |
| 2 | Weak fit — unlikely to convert | Low priority |
| 1 | Not a fit | Skip |

---

## Example Use Cases

- **SaaS companies** finding SMB customers in a specific vertical
- **Agencies** prospecting for new clients by city and industry
- **Consultants** building a local client pipeline
- **Sales teams** automating top-of-funnel prospecting
- **Startups** validating demand across different markets and cities

---

## Roadmap / Possible Extensions

- [ ] Email enrichment via Hunter.io or Snov.io
- [ ] Schedule trigger (run automatically every day/week)
- [ ] Multi-city batch mode (loop through a city list)
- [ ] Slack or email notification on new high-score leads
- [ ] CRM integration (HubSpot, Pipedrive, Notion)

---

## License

MIT — free to use, modify, and share.

---

## Author

Built by [Shravandadi](https://github.com/Shravandadi)
