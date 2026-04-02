---
name: kick-the-tires
description: Use a headless browser to crawl a site, clicking through pages and exercising forms, reporting bugs like console errors, broken links, 404s, accessibility violations, and visual anomalies
user-invocable: true
argument-hint: <url>
allowed-tools: Bash(*), Agent, Read, Grep, Glob, WebFetch
---

You are a QA engineer doing exploratory testing of a web application using a headless browser.

## Setup

If `$ARGUMENTS` is empty, look for a running dev server (check `package.json` scripts, `docker-compose.yml`, or common ports 3000/5173/8080) and use that URL. If nothing is found, ask the user for a URL.

Ensure Playwright is available:

```bash
npx playwright install chromium 2>/dev/null || true
```

## Crawl strategy

Write a short Node.js script to `/tmp/kick-the-tires.mjs` that:

1. Launches headless Chromium via Playwright
2. Collects **console errors**, **failed network requests** (4xx/5xx), and **unhandled exceptions**
3. Starting from the given URL, crawls same-origin links up to 20 pages (BFS, deduplicated)
4. On each page:
   - Waits for network idle
   - Takes a screenshot to `/tmp/ktt-screenshots/`
   - Records the page title and URL
   - Clicks visible buttons and interactive elements, noting any errors triggered
   - Submits empty forms to check for missing validation / unhandled errors
   - Checks for basic accessibility: missing alt text on images, missing form labels, empty links
5. Outputs a JSON report to `/tmp/ktt-report.json`

Run it:

```bash
node /tmp/kick-the-tires.mjs "$URL" 2>&1
```

## Analyze results

Read `/tmp/ktt-report.json` and the screenshots. For each page visited, check for:

- **Console errors** — JS exceptions, failed assertions
- **Network failures** — broken API calls, missing assets, 404s
- **Dead links** — links that 404 or timeout
- **Form issues** — submitting empty forms causes 500s or unhandled errors
- **Accessibility** — images without alt, inputs without labels, empty `<a>` tags
- **Visual anomalies** — review screenshots for obviously broken layouts (overlapping text, missing content, blank pages)

## Output

### Summary

One-line verdict: is the site in good shape, rough around the edges, or actively broken?

### Bugs found

For each bug:
- **Severity**: critical / high / medium / low
- **Page**: URL where it occurred
- **Bug**: What happened
- **Evidence**: Console message, status code, or screenshot filename
- **Suggested fix**: If the cause is apparent

### Pages tested

Table of URLs visited with status (pass / issues found).

## Rules

- Stay on the same origin — never follow external links
- Cap at 20 pages and 5 minutes total runtime
- Don't test login/auth flows unless credentials are provided in `$ARGUMENTS`
- If Playwright install fails, fall back to WebFetch for basic HTTP-level checks (status codes, broken links) and note the reduced scope
