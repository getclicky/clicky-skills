---
name: clicky-custom
description: Set up, customize, or debug Clicky custom tracking beyond the default pageviews. Use when the user wants goals/conversions with revenue, custom events, ecommerce data, usernames or custom fields on visitors, single-page-app route tracking, link tagging, or logging visits from their own backend scripts (PHP etc). Covers the clicky_custom object, clicky.log()/clicky.goal() functions, utm_custom URL params, CSS tagging classes, and the server-side in.php endpoint.
---

# Clicky custom tracking

Goal: the default one-line install logs page views; this skill covers everything beyond that: goals and revenue, events, custom visitor data, and manual logging from code.

## 0. Ground rules

- The standard tracking code must already be installed (see the `clicky-install` skill).
- Get the site's `site_id` + `sitekey` first: MCP `sites` tool, or `POST https://clicky.com/api/account/sites` with the user's userkey (never echo the userkey back).
- Most of this is premium-only (goals, custom data, event logging). If a call comes back with an upgrade link, tell the user.
- `clicky_custom` variables must be declared **before** the tracking script tag on the page. Define the object safely, then set one property at a time:

```html
<script>
	var clicky_custom = clicky_custom || {};
	clicky_custom.timeout = 20;
</script>
<!-- then the tracking code -->
```

## 1. Goals & revenue

Two ways, and they are NOT interchangeable:

- `clicky_custom.goal = { id: "10", revenue: "49.95" };` declared in the page (before the tracking code) fires the goal on page load. Use for thank-you/confirmation pages.
- `clicky.goal( "10", "49.95" );` called from a javascript event (button click etc). Never call it on page load.
- Predefined goal IDs are listed on the site's goal management page. Dynamic goals skip setup entirely: pass a name instead (`clicky.goal( "Clicked email" )` or `clicky_custom.goal = { name: "New user" }`). Unlimited dynamic goals; downside is no renaming, icons, or funnels.
- Varying revenue (ecommerce) has to come from the user's cart code; if they can't surface the value in JS, they need server-side logging (§5).

## 2. Events

```html
<a href="#menu/home" onclick="menu('home'); clicky.log('#menu/home','Home'); return false;">...</a>
```

`clicky.log( href, title, type )`: href required, must start with `/`, `#` (virtual/event URL, recommended for events), or http(s) (outbound). type is optional: `click` (default), `download`, `outbound`, `pageview`. For bulk tagging without inline JS, use the CSS classes on any link: `clicky_log_download`, `clicky_log_outbound`, `clicky_log` (as click), and `clicky_ignore` to exclude a link that would otherwise auto-count.

## 3. Custom visitor data

- `clicky_custom.visitor = { username: 'bobjones', email: 'bob@jones.com' };` attaches key/value pairs to the session. A `username` key gets special treatment: it replaces the IP/hostname in the visitors list.
- `username`, `name`, and `email` are also saved in a 30-day cookie so they stick across visits (unless visitor privacy is on or cookies are disabled).
- `utm_custom` does the same from a URL, no code needed: `?utm_custom%5Busername%5D=Bob+Jones` (URL-encode the brackets; email tools eat raw `[ ]`). Works on any page, not just landing pages.
- Privacy: logging personal data needs disclosure in the site's privacy policy, and possibly consent. `clicky_custom.visitor_consent = 1` flags consent given and re-enables full IPs/cookies for that visitor.

## 4. SPAs, overrides, and turning things off

- SPAs: `history.pushState` is tracked automatically (`history_disable = 1` to opt out if they log route changes manually); `clicky.pageview()` takes no parameters; `pageview_disable = 1` means nothing logs until they call it.
- `href` / `title` override the logged URL/title for page views (e.g. strip cart IDs from a query string).
- Common toggles: `timeout` (ping minutes, 5-240, 0 disables), `outbound_disable`, `outbound_pattern` (for `/go/`-style redirects), `html_media_track` (careful: autoplaying video = traffic spike), `heatmap_disable` / `osa_disable` (per-page), `cookies_disable`, `sticky_data_disable`, `cookie_domain`, `split` (goal split tests).
- Several of these also exist in the site's preferences; javascript wins when both are set.

## 5. Server-side logging (PHP etc)

Clicky documents its ingest endpoint, so their own scripts can log data: talk to `https://in.getclicky.com/in.php` with:

- Always: `site_id`, `sitekey_admin` (the *write* key, from the site's preferences - treat it like a password, server-side only), and `ip_address` OR `session_id`. Forgetting both logs their server's IP, not the visitor's.
- An action: `href` (required), plus `title`, `type` (pageview/download/outbound/click), `ref`, `ua`.
- Update an old session (even 30 days later, e.g. delayed ecommerce revenue): `type=goal&goal[id]=123&goal[revenue]=49.95` or `type=custom&custom[username]=billy`, keyed by `session_id`.
- URL-encode every value. A ready-made PHP function to do the encoding and assembling: https://clicky.com/help/code/php-clicky-log.phps
- Internal-script logging is Pro-and-higher.

## 6. Verify

Load the page / fire the event, then within ~30 seconds read it back (MCP `analytics` tool, or `POST https://clicky.com/api/stats/4` with site_id + sitekey): `type=goals` after a goal fires, `type=events` for clicks, `type=visitors-list` (or filter with `custom=KEY:VALUE`) to confirm custom data landed.

## More

The complete reference, every property with examples: https://clicky.com/help/custom and https://clicky.com/help/custom/manual (append `?md` for the markdown version). Media/video tracking: https://clicky.com/help/custom/media
