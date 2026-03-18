# AGENTS.md

## What is this project?

Swim Day is a PWA that shows live swim schedules for six East Bay (California) public pools. The schedule data is scraped automatically from pool websites using Playwright and pdf-parse, then served as a static site via GitHub Pages.

**Live site:** https://ainsleys.github.io/swimday/

## How the scraper works

`npm run fetch` runs `scripts/fetch-schedules.js`, which uses a headless Chromium browser to:

1. **El Cerrito Swim Center** — navigates to elcerrito.gov, finds the weekly PDF link, downloads and parses it
2. **Albany Aquatic Center** — navigates to albanyaquaticcenter.com (Wix site), intercepts the PDF network request, downloads and parses it
3. **Temescal Pool** — navigates to oaklandca.gov, scrapes schedule text from the DOM
4. **Strawberry Canyon** / **Roberts Pool** — seasonal outdoor pools, currently closed. Data is hardcoded with reopening dates.
5. **King Pool** — see below

Output goes to `data/schedules.json`, which is copied to `public/data/schedules.json` before deploy.

## Help needed: King Pool schedule

**King Pool** (City of Berkeley, 1700 Hopkins St) is the one pool our scraper cannot reach. The schedule lives on `berkeleyca.gov`, which uses a **FortiWeb WAF that blocks automated requests via JA3 TLS fingerprinting**. This blocks both curl and Playwright's bundled Chromium. We were unable to bypass it with stealth plugins (they patch JS, not the TLS layer).

The current King Pool data was manually extracted from this PDF:

> https://berkeleyca.gov/sites/default/files/2026-02/King%20Pool%20Schedule%20Spring%202026%203.2-6.7.pdf

**This schedule expires June 7, 2026.** After that date, the data will be stale unless someone submits an update.

### How you can help

If you are located in California (or can access `berkeleyca.gov` via a California-based connection), you can help keep King Pool's data current:

1. Visit https://berkeleyca.gov/community-services/parks-recreation and find the current King Pool schedule PDF
2. Open an issue or PR on this repo with:
   - The PDF URL
   - The date range it covers
   - Any holiday/closure dates mentioned in the PDF
3. Alternatively, paste the schedule text or a screenshot — we can parse it from there

The schedule PDF is typically found at a URL like:
```
https://berkeleyca.gov/sites/default/files/YYYY-MM/King%20Pool%20Schedule%20[Season]%20[Year]%20[dates].pdf
```

### Data format

King Pool's entry in `data/schedules.json` looks like this:

```json
{
  "king-pool": {
    "dataSource": "live",
    "sourceUrl": "https://berkeleyca.gov/...",
    "dateRange": "March 2 – June 7, 2026",
    "closedDates": [
      { "start": "2026-03-30", "reason": "City holiday" },
      { "start": "2026-06-06", "end": "2026-06-07", "reason": "Pre-Summer staff training" }
    ],
    "schedule": {
      "monday": [
        { "label": "Lap Swim", "type": "lap", "open": "7:30am", "close": "12:30pm" },
        ...
      ],
      ...
    }
  }
}
```

Each day has an array of sessions with `label`, `type`, `open`, and `close` fields. `closedDates` is an array of `{ start, end?, reason }` for holidays or special closures mentioned in the PDF.

## Other contributions

PRs welcome for:
- Improving the scraper reliability
- Adding new East Bay pools
- Fixing schedule parsing bugs
- Any WAF bypass ideas for berkeleyca.gov (we've tried `playwright-extra` stealth — it doesn't work because the block is at the TLS level, not JS)
