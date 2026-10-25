# Tech Reviews page for The Siam Times

An improved, fixed version of https://thesiamtimes.com/techReviews.html.

## Why the old page stopped showing news

The old page loaded Google News through three free public "CORS proxy"
services. All three now fail:

| Service | Status (Oct 2026) |
|---|---|
| `api.allorigins.win` | Times out (error 522) |
| `corsproxy.io` | Now requires a paid API key (401/403) |
| `api.rss2json.com` | Blocked by a Cloudflare challenge for many visitors |

So the page kept showing "Unable to reach Google News servers".

## How the new page gets news

1. **`techReviews.json` on your own site (main source).** A GitHub Action
   (`.github/workflows/update-tech-news.yml`) runs every 3 hours. It fetches
   Google News with `scripts/fetch_tech_news.py` and saves the headlines next
   to the page. Because the file comes from the same site, no proxy is needed.
2. **Live proxies (backup).** If the file is missing or older than 8 hours,
   the page tries several proxies at once, each with a time limit.
3. **Saved copy (last resort).** The last good result is saved in the
   browser, so visitors still see news (marked "Saved news from …").

## Installing on thesiamtimes.com

Copy these into the repository that publishes thesiamtimes.com, keeping the
same folder layout:

```
techReviews.html                       → replaces the old page
techReviews.json                       → first copy of the news feed
scripts/fetch_tech_news.py
.github/workflows/update-tech-news.yml
```

Then in that repository on GitHub:

1. Open **Settings → Actions → General → Workflow permissions**, choose
   **Read and write permissions**, and click Save.
2. Open the **Actions** tab, select **Update tech news**, and click
   **Run workflow** once to check that it works.

The page also works without the Action (it falls back to the live proxies),
but news loads far more reliably with it.

Note: GitHub pauses scheduled workflows in repositories with no activity for
60 days. If the news stops updating, re-enable the workflow from the Actions tab.

## What else changed

- **Security:** headlines are inserted as plain text, so a headline containing
  HTML can't inject code into the page. Only `http(s)` links are allowed.
- **Sentiment accuracy:** keywords now match whole words only. Before,
  "re**new**able" and "**new**s" counted as positive and "ur**ban**" as negative.
  "Clean" (one of the search terms) no longer makes every article positive.
- **Honest status:** "Auto-sync Active" was always shown but nothing
  auto-refreshed. The page now refreshes every 30 minutes and when the tab is
  reopened, and the status light shows green (fresh), amber (saved copy) or
  red (offline).
- **Correct charts:** daily counts use calendar days. The donut no longer
  starts with fake 1/1/1 data. The volume chart is split by sentiment.
- **Fixes:** duplicate stories are removed, articles without a date are no
  longer counted as published "just now", "1 hours ago" now reads "1 hour ago",
  and Refresh can't start overlapping loads.
- **New features:** search box, sentiment and topic filters (Solar, EVs,
  Hydrogen, Batteries…), tap a donut slice to filter, "Show more" button,
  topic tags on cards, last-24-hours trend, and a link back to the home page.
- **Mobile and accessibility:** 2-column stat cards on phones, readable chart
  labels, keyboard focus outlines, chart descriptions for screen readers, and
  reduced motion when the device asks for it.
- Chart.js and Tailwind are pinned to fixed versions, so a library update
  can't break the page.
