# kindle-niche-snapshot

Daily snapshot of Kindle nonfiction niche data, consumed by the `clawcreek-howto-author` skill.
The skill runs in a sandbox and downloads these files via their `raw.githubusercontent.com` URLs,
e.g. `https://raw.githubusercontent.com/Aoshine-T/kindle-niche-snapshot/main/snapshot/manifest.json`.

## Update frequency

Generated every day on ClawCreek's server (the KDP workspace of the automation executor, as a zero-token cron job)
and pushed here by the server host every 15 minutes when something changed (`data: snapshot YYYY-MM-DD`):

- **Hot list**: checked daily at 02:00 UTC, refetched when the last successful fetch is 7 days old (about 48 list
  pages of Best Sellers / Hot New Releases in 16 nonfiction Kindle categories). A failed week retries the next day.
- **Deep checks**: daily from 02:30 UTC, one keyword at a time in queue order until the day's request budget is
  spent (default 120 Amazon requests a day for the server's one outbound IP, including the server's own product-page
  checks; about 21 requests per keyword, 8–12 s between pages, 60 s between keywords).
- **Queue**: hot-list topics with no card first, then topics whose card is 7 days old, each by heat. `queue` in the
  manifest lists them with an estimated date (`eta_date`).
- A CAPTCHA or bot check stops the day at once; nothing is retried or worked around. The next day continues
  from the keyword it stopped on.
- Cards accumulate across days; `card_index` in the manifest gives each card's fetch date. The manifest (and its
  `generated_at`) is rewritten only when the hot list or a card changed.

## Data source

Parsed summaries of Amazon's public Kindle list pages (Best Sellers / Hot New Releases) and the search and
product pages for each deep-checked keyword. The data is produced by the step-01 script of
`clawcreek-howto-author` (`niche_research.py`), which is **not** in this repository (`scripts/` here is the
retired local batch, no longer run); `manifest.json` records
the skill branch and commit used. This repository only holds the parsed output.

## Layout

This is the proposed layout for skill 1.2.0; the skill side may still adjust it.

```text
snapshot/
├── manifest.json                         # snapshot metadata (see below)
├── topics.json                           # aggregated hot-list result; same as topics.json from the skill's step 01
└── cards/
    └── <keyword-slug>/
        ├── card.md                       # topic card for one keyword
        └── competitors.json              # competitor data for that keyword
```

`snapshot/manifest.json` fields:

| Field          | Meaning                                                        |
|----------------|----------------------------------------------------------------|
| `schema`       | Schema version, e.g. `kindle-niche-snapshot/0.1`               |
| `generated_at` | Generation time, UTC ISO 8601 (e.g. `2026-10-05T00:00:00Z`); `null` before the first snapshot |
| `topics`       | Number of topics in `topics.json`                              |
| `cards`        | Number of keyword directories under `cards/`                   |
| `files`        | Map of path (relative to `snapshot/`) → sha256 hex digest      |
| `skill`        | Repo, script path, branch and commit of the generating skill   |
| `requests`     | Amazon requests made by this run                               |
| `elapsed_seconds` | Run time of this run                                        |
| `steps`        | Requests and seconds per step of this run (`topics`, `deep`)   |
| `new_cards`    | Card slugs added by this run                                   |
| `stopped`      | `null`, or why this run's batch ended early and the next topic (`{"reason": "captcha", "next": "…"}`) |
| `card_index`   | slug → `{keyword, fetched_at}` for every card in `cards/`      |
| `queue`        | Topics still to deep-check, in order: `[{slug, keyword, position, eta_date}]`; `eta_date` is an estimate from the daily budget (`null` = no budget) |
| `budget`       | Amazon requests of the generating day: `{date, used, limit}` (UTC date; `used` includes the server's product-page checks) |

## Rules

- Only parsed results. No raw Amazon HTML, ever.
- No credentials, cookies, or personal data.
- Every file must be 5 MB or smaller.

## Usage

The data in this repository is provided solely for use by the `clawcreek-howto-author` skill.
No open-source license is granted.
