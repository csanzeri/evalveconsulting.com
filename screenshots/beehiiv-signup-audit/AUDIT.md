# Beehiiv Signup Flow — Audit

_Date: 2026-06-04 · Site: evalveconsulting.com · Publication: **Evalve Out Loud**
(`pub_9d491709-6d3b-4d14-81c5-4cbe594d26c0`) · Form ID `ee144ccc-5793-4359-9bdf-3aa50b263a88`_

## Summary
The site is a static HTML page (GitHub Pages). Newsletter signup is two embedded Beehiiv
iframes on `index.html`. The audit found the email form was being **mis-sold as the
podcast feed**, the podcast platform "badges" were **dead (non-clickable)**, and the
embeds had several small reliability/accessibility gaps. This pass fixes the
**website-side** issues only; Beehiiv-dashboard gaps are documented but not changed.

## How the flow works today
1. `index.html:15` loads `https://subscribe-forms.beehiiv.com/embed.js` (async).
2. Two `<iframe>` embeds of the same Beehiiv form:
   - `#subscribe` "email-cta" section (top of page).
   - `.bottom-cta` "Stay in the Loop" section (above the contact form).
3. The Beehiiv iframe handles the entire submit + confirmation; the site owns no
   thank-you page and fires no analytics event.

## Findings

### Fixed in this PR (website-side)
| # | Finding | Fix |
|---|---------|-----|
| 1 | **Email form mislabeled as the podcast** — top heading read "...podcast delivered to your inbox"; bottom heading read "Don't Miss an Episode". The form delivers a *newsletter*, not episodes. | Reframed both sections as the newsletter ("Get weekly insights ... straight to your inbox" / "Get the Newsletter"). |
| 2 | **"Subscribe for Episodes" button pointed at the email form** (`href="#subscribe"`) — mixing podcast and newsletter. | Repointed to a podcast-platform placeholder, relabeled **"Listen to the Podcast"**, with a "coming soon" note. |
| 3 | **Podcast platform badges were inert `<span>`s** — Apple Podcasts / Spotify / YouTube were not links, so the site had no path to the actual podcast. | Converted to `<a>` elements (the `.platform-badge` CSS already had `text-decoration:none` + a hover color, i.e. it was designed to be a link). |
| 4 | **Fixed `height:400px` + `scrolling="no"`** on both iframes — Beehiiv forms vary in height; the taller success state can be clipped. | Switched to `min-height:420px` and removed `scrolling="no"` so `embed.js` can auto-resize. |
| 5 | **No `title` on the iframes** (accessibility/SEO). | Added `title="Subscribe to the Evalve Out Loud newsletter"`. |
| 6 | **No fallback if the embed fails to load** (blank space). | Added a visible link under each embed to the hosted form `https://evalveconsulting.beehiiv.com/subscribe`. |
| 7 | **No source attribution** — website signups indistinguishable in Beehiiv reporting. | Appended `?utm_source=website&utm_medium=embed&utm_campaign=email_cta_top|bottom` to each iframe `src`. |

### Documented only — NOT changed (Beehiiv dashboard, owner action)
- **No welcome/onboarding automation.** `list_automations` returns 0 — new subscribers
  get no welcome email. Recommend adding a `form_submission`/`signup` automation in Beehiiv.
- **Referral program enabled but empty.** 0 milestones/rewards configured, so referral
  links do nothing. Configure milestones or disable.
- **Double opt-in is off** (`require_subscriber_approval: false`) — likely intentional.

## TODO for the owner (placeholders to fill)
When the podcast is live on platforms, replace the placeholder `href="#"` values in
`index.html` (search for `TODO`):
- "Listen to the Podcast" button → primary show URL (e.g. Spotify).
- The three `.platform-badge` links → Apple Podcasts / Spotify / YouTube show URLs.

## Screenshots
Captured headless (Chromium) against the **local** `index.html`.

> ⚠️ **Network note:** this environment's policy blocks outbound to `evalveconsulting.com`
> and `beehiiv.com` ("Host not in allowlist"), so the Beehiiv iframe could **not** render
> here — the empty area in the embed shots is where the form loads in production. The
> embed configuration was verified from source and via the Beehiiv API instead.

**Before**
- `before-01-homepage-full.png` — full homepage (desktop).
- `before-02-newsletter-top-embed.png` — top embed; heading mislabels it as podcast delivery.
- `before-03-podcast-cta-dead-badges.png` — "Subscribe for Episodes" + inert badges.
- `before-04-newsletter-bottom-embed.png` — bottom "Don't Miss an Episode" embed.
- `before-05-mobile-full.png` — full homepage (390px mobile).

**After**
- `after-01-newsletter-top-embed.png` — newsletter framing + fallback link.
- `after-02-podcast-cta-linked.png` — "Listen to the Podcast" + clickable platform links + "coming soon" note.
- `after-03-newsletter-bottom-embed.png` — "Get the Newsletter" + fallback link.
- `after-04-homepage-full.png` — full homepage after changes.

## How to verify in production (once merged)
1. Visit https://evalveconsulting.com — confirm both embeds render the Beehiiv form, no
   clipping of the success state, at desktop and mobile widths.
2. Confirm "Listen to the Podcast" + badges are present (filled with real URLs once live).
3. Submit a test email; confirm it appears via Beehiiv `list_subscriptions` for the
   publication, attributed to `utm_source=website`.
</content>
