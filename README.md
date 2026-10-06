# How to Scrape Greenhouse, Lever, Ashby & Workable Job Boards (One Schema, Four ATS)

> **Run it now, no code: [multi-ats-jobs-scraper on the Apify Store](https://apify.com/bibian/multi-ats-jobs-scraper)** &nbsp;|&nbsp; $1.20 per 1,000 job postings &nbsp;|&nbsp; no browser, no proxy

Greenhouse, Lever, Ashby and Workable are the four applicant-tracking systems (ATS) behind most tech company career pages. Each one publishes its own public job board API, with its own shape. This guide shows how to read all four in a single run and get one normalized schema: title, department, location, remote policy, employment type and full description.

Searches this page answers: greenhouse scraper, lever jobs scraper, ashby scraper, workable scraper, ATS job board scraper, company career page scraper.

## What you get from the Multi-ATS scraper

- Every open posting for the company boards you list, across Greenhouse, Lever, Ashby and Workable, normalized into one schema.
- Department, location, remote policy (`remote`) and employment type (`employmentType`) where the platform publishes them.
- Publication and update timestamps, and the canonical posting URL.
- Optional full job description as plain text.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, long descriptions truncated) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id` | `platform:company:postingId`, unique across all four platforms |
| `platform` | `greenhouse`, `lever`, `ashby` or `workable` |
| `company` | Board token/slug as given in the input |
| `title`, `department` | Job title and department |
| `location` | Location label as published by the platform |
| `remote` | Remote policy where the platform exposes it; always `null` on Greenhouse (not published consistently there) |
| `employmentType` | Full-time, contract, etc., where published |
| `publishedAt`, `updatedAt` | ISO timestamps (`updatedAt` is Greenhouse-only) |
| `url` | Canonical link to the posting |
| `description` | Plain text, when `includeDescriptions` is enabled |

## Use cases

- **Monitor several companies' hiring at once**, whatever ATS each one happens to use, from one run instead of four separate tools.
- **Feed a job alert or an aggregator.** Run it daily and dedupe against your own store by `id`.
- **Track a sector's hiring.** List every open role for a group of companies you follow.
- **Benchmark remote policy and employment types** across companies and platforms.

## Input

| Field | What it does |
|---|---|
| `greenhouseBoards` | Greenhouse board tokens, for example `airbnb` from `boards.greenhouse.io/airbnb`. |
| `leverBoards` | Lever company slugs, for example `palantir` from `jobs.lever.co/palantir`. |
| `ashbyBoards` | Ashby job board names, for example `ramp` from `jobs.ashbyhq.com/ramp`. |
| `workableBoards` | Workable account shortcodes, for example `huggingface` from `apply.workable.com/huggingface`. |
| `maxItems` | The run stops once this many postings are saved, across all boards combined (default 300). |
| `includeDescriptions` | Adds the full job description as plain text. |

Leave every board list empty to run a small default set spanning all four platforms.

Example input:

```json
{
  "greenhouseBoards": ["webflow"],
  "leverBoards": ["palantir"],
  "ashbyBoards": ["ramp"],
  "workableBoards": ["huggingface"],
  "maxItems": 300,
  "includeDescriptions": true
}
```

## Three ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/multi-ats-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the board tokens for whichever platforms you need and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new postings to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/multi-ats-jobs-scraper').call({
    "greenhouseBoards": ["webflow"],
    "leverBoards": ["palantir"],
    "ashbyBoards": ["ramp"],
    "workableBoards": ["huggingface"],
    "maxItems": 300,
    "includeDescriptions": true
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.company} (${item.platform}) | ${item.location ?? '-'}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/multi-ats-jobs-scraper").call(run_input={
    "greenhouseBoards": ["webflow"],
    "leverBoards": ["palantir"],
    "ashbyBoards": ["ramp"],
    "workableBoards": ["huggingface"],
    "maxItems": 300,
    "includeDescriptions": True,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('company')} ({item.get('platform')}) | {item.get('location')}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Pricing

Pay per result: **$1.20 per 1,000 job postings** (that is $0.0012 per job posting), plus a tiny start fee of $0.00005 per run. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/multi-ats-jobs-scraper) when this guide was written; the Store page is the reference.

## Limits and honest notes

- A board slug or token that does not exist on its platform is skipped with a warning in the log, not a failure.
- `remote` is read from each platform's own field (Lever's `workplaceType`, Ashby's `isRemote`, Workable's `telecommuting`); Greenhouse does not expose it consistently, so it is always `null` there. This is not a bug, it reflects what Greenhouse publishes.
- `updatedAt` is only published by Greenhouse; it is `null` on the other three platforms.
- The dataset contains job postings only. The hiring company is named, never an individual recruiter or candidate.
- This is an unofficial tool, not affiliated with Greenhouse, Lever, Ashby or Workable. Check each platform's terms and the data protection rules that apply to your use.

## FAQ

**Does it need a browser or a proxy?**  
No. Every platform publishes a public job board API; this Actor only reads it, so runs are fast and cheap.

**What if I only use one of the four ATS?**  
Fill in only that platform's board list and leave the others empty; the Actor only queries what you give it.

**How do I only get new postings every day?**  
Schedule the Actor once a day and dedupe the results against your own store by `id`.

**Can I get the data in Excel or Google Sheets?**  
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

## Try it

**[Open multi-ats-jobs-scraper on the Apify Store](https://apify.com/bibian/multi-ats-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
