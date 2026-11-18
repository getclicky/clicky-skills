---
name: clicky-install-tracking
description: Install, fix, or verify Clicky web analytics tracking on a website. Use when the user wants to add Clicky to a site, sees zero or wrong visitors, or asks whether tracking works. Covers the one-line async script, tag managers, CSP, SPA/framework setups, and the ad-blocker proxy.
---

# Install Clicky tracking

Goal: every page view fires exactly one request to Clicky, and the dashboard shows it within ~30 seconds.

## 1. Get the site's ID (and sitekey if you need API verification)

With the user's userkey (from https://clicky.com/user/edit - never echo it back):

```
POST https://clicky.com/api/account/sites  (userkey=USERKEY)
```

Each site comes back with `site_id`, `sitekey`, `hostname`, and `timezone`. Confirm which site maps to the codebase you're editing.

## 2. Add the tracking code

One line, site-wide, just before `</body>`. Replace `SITE_ID`:

```html
<script async data-id="SITE_ID" src="//static.getclicky.com/js"></script>
```

- Paste it in the shared layout/template include, not per-page.
- **Do not add a second copy** to pages that already have it - duplicate tracking double-counts. Grep for `getclicky` or `data-id` first.
- WordPress/Joomla/Drupal/etc: use the plugin (https://clicky.com/help/plugins) instead of editing code.
- Google Tag Manager or other tag manager: add a Custom HTML tag with the script. The anti-adblock proxy option and other variants live on the site's tracking-code page inside Clicky's site preferences.
- Single-page apps / frameworks (React, Vue, Next, ...): the async script tracks the first page load; later route changes need a manual pageview call via the JS API once the tracker is loaded (see https://clicky.com/help/custom).

## 3. Gotchas

- **CSP:** the site's Content-Security-Policy must allow `static.getclicky.com` in `script-src` and `connect-src`.
- **Ad blockers** block the default beacon. If numbers look low, enable Clicky's anti-adblock proxy (on the tracking-code page) and paste the variant script it gives you.
- **Cookie consent** (GDPR): Clicky doesn't use cookies for tracking by default; no banner needed for Clicky itself, but check the site's own policy.
- If the site answers on more than one domain, add the others as mirrors in the site's preferences or those hits won't count.

## 4. Verify

1. Load a page of the site in a browser (disable the cache; make sure an ad blocker isn't blocking the beacon for this test).
2. Within ~30 seconds:

```
POST https://clicky.com/api/stats/4  (site_id=SITE_ID, sitekey=SITEKEY, type=visitors-online, output=json)
```

A non-zero count = tracking works. Zero: check the browser Network tab for the beacon request (blocked = CSP/ad blocker; absent = script didn't load or isn't on the page). If the beacon fires but the count stays 0, the hostname probably doesn't match the site's domain/mirrors.

## More

- Full procedure: https://clicky.com/help/agents/install
- Custom tracking (events, custom vars, ecommerce): https://clicky.com/help/custom
- Reading the data afterwards: the `clicky-analytics` skill
