# East Winds Resort — Guest App: Project Context

**Client:** East Winds Resort, St Lucia · **Built by:** Hook & Ladder, inc. (H&L Digital)
**Status:** Staging, hardened for a soft release; the resort team is training with the staff manual · **Last updated:** 27 October 2026
**Staging (this work):** `https://wordpress-1587086-6643085.cloudwaysapps.com` (Cloudways)
**Production (from launch):** `https://guest.eastwinds.com` (today it still points to a different Cloudways app, `…-6637015`; the live version will be served there)
**Staff manual:** <https://staffdocs.eastwinds.com> (this repository, `hookladder/docs-for-ewr-guest`)

This document is the starting point for anyone (person or AI agent) picking up the project. It explains why the app exists, how guests and staff use it, how it is built, and the constraints that are easy to miss. It deliberately stays at product and architecture level; implementation detail lives in the docblocks of each file referenced below.

---

## 1. Why the app exists

East Winds is a small, intimate all-inclusive resort. Before the app, everything a guest needed during a stay (the day's activities, menus, excursion bookings, housekeeping requests, information about the resort and the island) lived on paper, at reception, or in emails. The app replaces that with one place on the guest's phone.

Goals that shaped every decision:

1. **No accounts, no passwords.** A guest receives a private link and is in. Nobody registers or logs in.
2. **Feels like a native app.** Installable on the home screen (PWA), full screen, bottom tab bar, smooth screen changes, no hover effects on touch.
3. **Always current.** Menus, activities and availability change daily, so content is never served stale from a cache.
4. **Run by resort staff, not developers.** Everything guests see is edited in WordPress (custom post types and a settings page), and requests arrive in the admin with clear queues and counters.
5. **Requests, not transactions.** Excursions and spa treatments are *requested* in the app and *approved* by the front desk. Payment, when there is any, is settled at the resort; the app never takes money.

---

## 2. How guests use it

### 2.1 Getting in
- The front desk creates a **Stay** in WordPress (guest name, room, dates, party size). A unique token is generated automatically.
- The guest receives the **private link** `…/my-stay/{token}/` (by email or at check-in).
- Opening the link starts a session (signed cookie). If the cookie is lost (private browsing, iOS home-screen app, aggressive cache), the token travels in the URL as `?ews=<token>` and is threaded onto every internal link, so the session self-heals.

### 2.2 Guest states
The app adapts to three states, computed **from the stay dates** (never from a manual flag):

| State | When | What the guest sees |
|---|---|---|
| **Pre-arrival** | Before check-in date | Welcome screen, check-in date, limited content |
| **On property** | Check-in → check-out | The full app; bookings enabled |
| **No stay** | No valid session | Neutral welcome asking for the personal link |

### 2.3 Check-in and consent
On the first visit while on property, a **blocking check-in gate** collects contact email, a few opt-ins (travel agent, Family Foundation interest, water-sports waiver, privacy notice) and records the consent with a timestamp and version. Until it is completed, the rest of the app (and the install button) stays locked.

### 2.4 The tabs

| Tab | Content |
|---|---|
| **Today** | Greeting, weather, today's menus (the meal for the current time is highlighted), today's activities, offers and alerts |
| **Explore** | *Dining* (meal times and menus), *Experiences* (all activities and spa treatments, filterable by date, type and category), *Island* (editorial guide cards) |
| **Book** | The guest's own bookings with live status, and the bookable catalogue |
| **My Stay** | Stay summary, check-in summary, housekeeping preferences (two-way synced with the admin), install-the-app button |
| **Resort** | Hosts, services, map, guest directory and documents, opening hours, feedback, messages to the front desk |

### 2.5 Booking
- **Weekly activities** (yoga, pilates, cocktails, tours…) show their *next* session date, which is recalculated daily from a weekly timetable.
- **Standing / by-request activities** (water sports, pool, spa) have no fixed date and are always requestable.
- **Spa treatments** are individual items grouped in sub-categories; a guest may request the same treatment more than once per stay, for up to 4 people. The spa team contacts the guest to confirm the time before approving.
- After requesting, the guest sees *Requested → Confirmed / Not available*. A decline always carries a reason, which the guest sees in **My Bookings**.

### 2.6 Installing the app
- **Android / Chrome:** the "Get East Winds on your device" button (My Stay, after check-in) opens the native install prompt.
- **iPhone / iPad (Safari):** the same button shows *Share → Add to Home Screen* instructions and pins the session to the URL first. This matters because iOS home-screen apps do not share Safari's cookies. Recommended browsers: **Safari on iOS, Chrome on Android**.

---

## 3. How staff use it (wp-admin)

| Area | Purpose |
|---|---|
| **Stays** | Create and edit guest stays; the private link is derived from the token |
| **Rooms** | Room types and numbers used by stays |
| **Excursions** | Activities and spa treatments: schedule (weekly / one-off / by request), spots (*Spots Available*), price, meet point, image, category |
| **Exc. Bookings** | Booking queue. Sidebar bubble = pending requests. Row actions *Approve* / *Decline* (decline opens a dialog requiring a reason). A **Spa** view isolates spa requests |
| **Desk Messages** | Messages from guests ("Say hello", "Message the desk", "Plan this with us"). Sidebar bubble = unread; opening a message marks it read |
| **Daily Content** | Menus per date and meal (breakfast / lunch / afternoon tea / dinner). The title is internal (restaurant and meal); guests see the meal and its dishes. No Preview: menus are checked in the app |
| **Offers / Alerts** | Promotions and operational notices shown in the app |
| **Resort Settings** | One options page for resort-wide content: hosts, services, hours, map, directory, feedback links, **Island guide cards** |
| **PWA WP Router** | App name, icon, colours and install-button styling |
| **Role-GenX** | Admin sidebar order and custom roles |

**Resort Manager role** (Role-GenX): sees only the resort's content types, Media and Resort Settings, with the URL guard blocking everything else. Anyone who cannot install plugins is also treated as staff by `dev-tools-guard.php`, which hides the Novamira switch and refuses its actions. One account per person, never shared; an H&L test Manager account is used for the manual's screenshots. Usernames and passwords are never written in this repository, which is public.

**Staff manual:** <https://staffdocs.eastwinds.com>, in English and Saint Lucian Kwéyòl (draft, pending native review). Docsify site published by GitHub Pages from `main` of this repository; work goes to `new-version-incoming` and reaches `main` through a reviewed pull request. Commits are authored by Ali Leon only (no AI co-author line), so a person stays accountable at handover. See the root `README.md` for editing and publishing.

**Island guide cards** (Resort Settings → *Island things to do*): title, image and a description that accepts a tiny markdown subset (blank line = new paragraph, `**bold**`, `- ` bullet). Order in the repeater is the order in the app. Every card ends with a "Plan this with us" button that opens a pre-filled message to the front desk.

Notifications for new booking requests and guest messages are emailed to the front desk, so nothing waits unseen in the admin.

---

## 4. Architecture

### 4.1 Stack
- **WordPress** (7.x) with the **Hello Elementor** theme, **Elementor + Elementor Pro** (v3 widgets and v4 atomic elements coexist).
- **Crocoblock suite:** JetEngine (CPTs, meta boxes, options page, Query Builder, listings), JetSmartFilters, JetTabs, JetElements, JetPopup.
- **WP Rocket** for page caching and asset optimisation.
- **Novamira (+ Pro):** MCP bridge used by AI agents to operate the site.
- Three in-house plugins carry all custom behaviour (below). No custom theme code.

### 4.2 Custom plugins

| Plugin | Role | Notes |
|---|---|---|
| **EW Stay Access** | The guest app's core: identity and sessions, state gating, check-in, booking engine, all app shortcodes, date logic, Explore/Book/Resort support, page transition, and the single app stylesheet `assets/ew-app.css` | Lives only in production. Each `includes/*.php` opens with a docblock explaining its responsibility; read those first |
| **PWA WP Router** (H&L) | Installable PWA: dynamic manifest, session-safe service worker, install button, kill switch | Generic and reusable. Source of truth is the H&L local copy with its README and e2e suite |
| **Role-GenX** (H&L) | Global admin-menu order and whitelisted custom roles, enforced on direct URLs and on `options.php` posts | Generic and reusable. H&L local copy with README and e2e suite |

### 4.3 EW Stay Access modules (`includes/`)

| File | Responsibility |
|---|---|
| `identity.php` | Stay token, signed cookie, `ew_get_current_stay()`, state helpers, Elementor dynamic tags, "Update Stay" form action (form field ID = stay meta key) and two-way preference prefill |
| `session-token.php` | Token-first session (`?ews=`), link threading, no-cache headers for guest pages, PWA start-URL pinning |
| `checkin-gate.php` | Consent gate, check-in summary, `[ew_checked_in]` wrapper |
| `booking-engine.php` | Booking CPT, capacity, request handler, approve/decline (with reasons), spa rules, admin queue (columns, bubble, Spa view) |
| `excursion-recurrence.php` | Weekly timetable → derived next `schedule_date`, recalculated daily |
| `queries.php` | Date-aware Query Builder filters (Explore and Today), listing macros, ordering of standing activities and spa |
| `explore.php` | Explore support: nested-template CSS, category filter pills and card category data |
| `book.php` | My Bookings (status and decline reason) |
| `resort.php` | Resort page sections and Island guide cards, all from Resort Settings |
| `hosts.php` | Hosts page, "Say hello" messaging, Desk Messages unread tracking |
| `offer.php`, `shortcodes-extra.php` | Offers, menus, weather, small app shortcodes |
| `page-transition.php` | "Loading East Winds…" screen between page loads |
| `admin-tweaks.php` | Removes Preview / View (button, permalink box, row action, admin-bar item) for menus, alerts and offers, which have no page of their own and are only shown inside other app screens |
| `dev-tools-guard.php` | Treats anyone without `install_plugins` as staff. Hides the Novamira admin-bar switch, refuses Novamira's admin-post / AJAX actions, and refuses staff REST writes to developer tools: Novamira, MCP and the core Abilities API (which run PHP), AI proxies, the JetEngine builder, `/wp/v2/settings`, and Elementor or plugin settings. Role-GenX leaves admin-post, AJAX and REST alone, so this file covers them |
| `security.php` | Keeps stays unlisted (out of REST, search and sitemaps; an old stay slug never redirects to a rotated token), hides usernames from visitors (REST users, author archives), sends Referrer-Policy, nosniff, X-Frame-Options and HSTS, and disables the file editors |

### 4.4 Data model (JetEngine)
- **CPTs:** `stay`, `room`, `excursion`, `ew_exc_booking` (private), `ew_desk_message` (private), `daily-content`, `offer`, `alert`.
- **Taxonomy:** `excursion_category`, **hierarchical**. *Spa & Wellness* has the treatment sub-categories; being in one of those children is what makes an item a "spa treatment".
- **Options page:** `resort-settings` (all resort-wide content).
- **Key meta:** stay dates and token on `stay`; `recurrence_mode`, `schedule_date`, `spots_available` (the capacity staff edit; legacy hidden `capacity` is only a fallback), `price`, `excursion_type` on `excursion`; `status`, `guests_count`, `decline_reason` on bookings; `content_date`, `meal` on daily content (`breakfast`, `lunch`, `afternoon_tea`, `dinner`; labels and service windows in `ew_meals()` / `ew_meals_now()` in `shortcodes-extra.php`). One menu per meal per day; Afternoon Tea shows from 11:00 alongside lunch and is served at the time set in Resort Settings.

### 4.5 Front-end composition
- Pages: Today (front page), Explore, Book, My Stay, Resort, Menu, Hosts.
- Content is assembled from **Elementor templates** (state-specific My Stay templates, Explore tab templates, listing items) plus **shortcodes from EW Stay Access**. Shortcode markup is styled by `ew-app.css`, because Elementor never compiles Global Class CSS for PHP-echoed markup.
- Experiences filters run **client-side** (no reload), reading hidden data carriers on each card.

---

## 5. Engineering conventions (non-obvious)

1. **A stay URL is a key.** Tokens are 16 random characters; stays must never appear in REST, search, sitemaps or any listing, and a rotated token must stop working (no old-slug redirect). See `security.php`.
2. **Guest pages are never page-cached.** Any response with a session sends `no-store` + `DONOTCACHEPAGE`; per-guest data must never reach a shared cache.
3. **Use the plugin that owns the data.** Content types, fields and the options page belong to JetEngine. Don't register CPTs, taxonomies or meta in PHP for content JetEngine models.
4. **Nested Elementor templates need their CSS enqueued manually** (JetTabs panels, `[elementor-template]`); see `explore.php` and `book.php`.
5. **Elementor Pro popups don't fire on this site** (they conflict with JetPopup). Modals are hand-rolled.
6. **No hover, focus or tap colour change on any button.** Hello Elementor's reset paints buttons pink on hover; a dedicated block at the end of `ew-app.css` pins every button's states to its base colours. Keyboard `:focus-visible` outlines stay for accessibility.
7. **Re-date, don't duplicate.** To keep menus or stays current, move existing items to new dates and trash what is past. Daily content holds only today onwards, one item per meal and day, alternating two menu sets.
8. **Backups before edits.** Production files and settings are backed up into non-autoloaded `ew_backup_*` options (never as `.bak` files in the web root, which would expose source).
9. **Language:** code comments and H&L documentation in English; guest-facing copy in British English.

---

## 6. Operations

### 6.1 Hosting and security (Cloudways)
- **Shared server: 2 CPUs and ~4 GB RAM** hosting other applications. Every uncached WordPress boot costs ~1.2 s of CPU, so bursts of admin traffic can saturate it.
- **Imunify360 Bot Protection** is active. It blocks automation (the Novamira MCP) from IPs that are not whitelisted, and its JavaScript challenge **drops the private-link path on redirect**, so a guest opening their link for the first time from a non-whitelisted network can land without a session. Recommendation: **disable Bot Protection for this app**, or have Cloudways exclude the app's routes and `/wp-json/mcp/novamira`.
- **Hardening (27 October 2026):** search engines are discouraged (Settings > Reading, plus `noindex` on every page), sitemaps and the REST users endpoint are off for visitors, security headers are set, and the old `.bak` copies inside the plugin were removed (they were downloadable as source; their content is kept in the non-autoloaded option `ew_backup_bak_files_20261027`).
- **Login:** WPS Hide Login moves the login to `/903ewr-staff-login/`; `/wp-login.php` and `/wp-admin` (signed out) return 404. By decision (27 October 2026) the team address is published in the manual (`https://guest.eastwinds.com/903ewr-staff-login/`), so the live site must use the same slug. Novamira is unaffected (it authenticates through REST).
- **Manual gate:** only `resortmanager@eastwinds.com` and `dev-admin@hldigital.ca` pass the courtesy form of the staff manual (stored as SHA-256 fingerprints). It is not access control: the manual's files are public.
- **Activity log:** Simple History records sign-ins and changes, visible to administrators only.
- **Desk messages:** only a guest with a valid stay session can send one, at most 10 per hour, so the form cannot be used to spam the front desk mailbox.
- **WP-Cron** runs on page loads (~150 times per hour). Recommendation: a server cron every 5 minutes in Cloudways, then `DISABLE_WP_CRON`.

### 6.2 Caching layers
- **WP Rocket:** page cache for anonymous pages only. Guest pages bypass it.
- **Service worker (PWA WP Router):** cache-first for static assets, network-first for HTML and REST. The cache version bumps automatically on Rocket purges, Elementor saves and settings saves, so devices refresh silently.
- `ew-app.css` is versioned by file modification time.

### 6.3 Working on the site with an AI agent
- The site is operated through **Novamira**: either the MCP (stdio proxy `@automattic/mcp-wordpress-remote`, credentials as environment variables only) or the **Novamira CLI** (`novamira --site east-winds …`, OAuth login approved in the browser).
- If the MCP connects with **0 tools**, or the CLI reports *"OAuth metadata is malformed or unavailable"*, Imunify is blocking the current public IP (the site answers with its "One moment, please…" challenge page). Whitelist that IP in Cloudways (preferably the whole /24) and retry. Never send bursts of parallel requests: they trigger a hard block.
- Novamira keeps a **memory** inside WordPress with project notes, product rules and open items. Agents should read it at the start of a session.

---

## 7. Known limitations and open items

| Item | Impact | Suggested direction |
|---|---|---|
| Capacity counts **all-time** bookings, not per session | Weekly activities gradually show as full (staff can raise *Spots Available*, which since 13 October is the field the booking engine reads) | Count only bookings for the next session date |
| *Today* / *This week* filters hide by-request items (incl. spa) | Standing activities only appear under *All upcoming* | Always show standing items, or label them separately |
| Card meta line starts with "·" on items without a time | Cosmetic | Skip the separator when the time is empty |
| My Stay renders empty after check-out | Past guests see a blank screen | Add a post-stay state |
| Island cards edited inside Resort Settings | No drafts, scheduling or revisions | Optional move to an "Island Guide" CPT |
| Resort Manager role keeps `manage_options` **by decision** (the Manager runs the whole app) | Plugin AJAX handlers that check only that capability, and any plugin installed later, are reachable by staff | Known developer routes are refused in `dev-tools-guard.php`; check every new plugin's AJAX and REST routes against the Manager role before installing it |
| No 2FA and no login-attempt limit | Password guessing relies on the private login address and Imunify | Add a login limiter or 2FA before the public launch |
| Guest email on decline | On stand-by (not discarded) | Reuse the decline-reason text when it comes back |
| Temporary Cloudways domain | Private links, and the QR codes East Winds prints in welcome letters, will change | Move to the final domain before any letter for real guests is printed |
| Media placeholders still in use | Some items share demo photos | Replace with real photography |

### 7.1 Scope: V1 and V2 (agreed 16 October 2026)

After the resort's first round of feedback (Judith, 16 October), the line between the current version and the next phase is:

| Request | Where it lands | Notes |
|---|---|---|
| Rename menus to *Bird of Paradise Restaurant*; remove the confusing Preview | **V1, done** | `admin-tweaks.php`; titles are internal |
| *Afternoon Tea* as a meal | **V1, done** | Shows 11:00–17:00; time read from Resort Settings |
| QR code for each stay | **Client side** | East Winds turns the stay permalink into a QR code with any generator |
| Welcome letter | **Client side** | Built by hand by East Winds |
| Team training | **Manual only** | No live session; support by email |
| Reply to desk messages inside the app | **V2, development** | Together with guest emails, including the decline email on stand-by |
| Printable daily sheet of spa and excursion bookings | **V2, separate plugin** | A booking-management (CRM-type) feature |
| Guest instruction card, Spa & Excursion flyers, generated welcome letter with QR | **V2, design team** | Needs a designed template plus a printable-format build |

The commercial line for V1 is not closed yet; anything outside the V1 rows needs a proposal before work starts.

---

## 8. Where to look next
- **EW Stay Access:** the docblock at the top of each `includes/*.php`, and the comment sections in `assets/ew-app.css`.
- **PWA WP Router / Role-GenX:** their `README.md` (installation, configuration, security model, tests, changelog).
- **Novamira memory** in WordPress: the current product rules and open items.
- **Staff manual** source: `en/` and `kw/` in this repository; the root `README.md` explains how to edit and publish it.
