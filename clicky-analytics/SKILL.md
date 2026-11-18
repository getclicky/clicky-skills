---
name: clicky-analytics
description: Analyze a user's website traffic with Clicky (clicky.com) - visitors, page views, referrers, search keywords, goals, conversions, ecommerce revenue, and uptime. Use when the user asks about their site traffic or analytics data, mentions Clicky, or wants a report or dashboard answer. Read-only. Works over the Clicky MCP server (preferred) or its plain HTTP API.
---

# Clicky analytics

Clicky (https://clicky.com) is a real-time web analytics service. Its data is reachable two ways:

1. **MCP server** at `https://clicky.com/mcp` - two tools, `sites` and `analytics`. Preferred; connect first.
2. **HTTP API** at `https://clicky.com/api/stats/4` and `https://clicky.com/api/account/sites` - plain query params via POST (recommended) or GET, no SDK. Use when no MCP connection exists.

Authentication is a single per-account **userkey** (found or generated at https://clicky.com/user/edit). Over MCP it goes in as a bearer token or a `userkey` URL param; in the API it's a `userkey` param on the account call. Never store or echo the key in output. If a tool call 401s, ask the user for their key; do not guess.

## Quick start

1. Call the `sites` tool (or `POST https://clicky.com/api/account/sites` with the userkey; add `stats=1` to get today's summary numbers for every site in the same call). Returns every site on the account: `site_id`, `sitekey`, hostname, nickname, timezone, visitors online now and today.
2. Pick the site the user means (match by hostname or nickname; ask if ambiguous).
3. Call `analytics` with that site's `site_id` + `sitekey`, a `type`, and a `date`.

Results come back as markdown tables by default (cheapest in tokens). Pass `output=json` when you need to parse values programmatically.

## Report types

`type` accepts one or several comma-separated values (batch related reports in one call):

- **Totals:** `summary` (visitors + actions + times + bounce in one), `visitors`, `actions`, `visitors-online`, `visitors-unique`, `visitors-new`, `actions-pageviews`, `bounce-rate`, `time-average`, `time-total`, `goals-tally`, `goals-revenue`
- **Ranked:** `pages`, `pages-entrance`, `pages-exit`, `traffic-sources`, `searches-engines`, `searches-keywords`, `campaigns`, `goals`, `countries`, `cities`, `regions`, `web-browsers`, `operating-systems`, `links`, `links-outbound`, `downloads`, `clicks`, `events`, `custom`, `splits`
- **Chronological:** `visitors-list` (live per-visitor log), `actions-list`, `searches-recent`, `links-recent`, `uptime-list`
- **Segmentation:** `type=segmentation` + `segments=` (comma-separated report types to compute for a matched set of visitors) + `filter=` (see below)

## Dates

`date=today` (default), `yesterday`, `last-7-days`, `last-30-days`, `YYYY-MM-DD`, or a range `YYYY-MM-DD,YYYY-MM-DD`. Day boundaries follow each site's own timezone. A range that ends before yesterday is stable and cached for an hour; ranges touching today are cached 60s (paid plans) or 15 min (free). Don't poll faster than the cache window - it can't return fresher data.

## Filtering and segments

Any report narrows to matching visitors via `filter`. Plain text = match items within the report. A value containing `=` is a query tail appended to the call verbatim (URL-encode `&`, `=`, and spaces):

- `filter="country=us&href=%2Fpricing"` - visitors from the US on /pricing
- Filter params: `country, region, city, domain, link, href, search, campaign, goal, source, org, hostname, uid, ip_address, session_id, browser, os, hardware, language, num_actions, time, action_type, custom=KEY:VALUE`
- Compare cohorts with `type=segmentation` + `segments=summary,pages` + the tail.

## Etiquette and gotchas

- **Read-only.** Nothing here can change settings; if asked to configure Clicky, tell the user what to click instead (https://clicky.com/help/agents/limits).
- Empty results usually mean *no data in that range*, not an error. Say so; don't retry in a loop.
- An `ERROR:` line or a fail response is the failure shape (e.g. `ERROR: Invalid sitekey.`).
- `visitors-list`, `actions-list`, and `segmentation` are capped: 31-day ranges, 1,000 items/request max, page with `limit` + `page`.
- Premium-only report types return an upgrade link instead of data on free plans.
- AI visitors (from chatbots and `.ai` referrers) are categorized separately in `traffic-sources`.

## More

- API reference (all types + params): https://clicky.com/help/api
- Agent procedures: https://clicky.com/help/agents
- Machine-readable index: https://clicky.com/llms.txt
- Installing the tracking code is a separate skill: `clicky-install-tracking`
