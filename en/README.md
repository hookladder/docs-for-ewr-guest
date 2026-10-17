# Manager Manual

Everything the East Winds team needs to run the guest app from WordPress: approving bookings, answering guests, keeping stays, experiences and menus up to date, and posting alerts.

*Version 0.1 (beta) · October 2026 · For the **Resort Manager** role*

The guest app is the little website each guest opens on their phone during their stay. Everything a guest sees in it comes from WordPress, and you control it from the WordPress admin (also called **wp-admin**). You never need to touch any code.

?> **This is a beta manual** The app is still growing, and so is this guide. If something on your screen looks different from the pictures here, or you cannot find a feature, write to [support](/en/support).

## Signing in

1. Open `https://wordpress-1587086-6643085.cloudwaysapps.com/wp-admin/` on a computer or tablet.
2. Type the **username** (or email) and **password** you received from Hook + Ladder.
3. Press **Log In**. You land in the admin, with your menu on the left.

<figure><img src="assets/img/login.webp" alt="The WordPress login screen"><figcaption>The login screen.</figcaption></figure>

!> **Keep your password to yourself** Every manager has their own account, so the admin can show who approved or declined each booking. Never share an account. If you forget your password, use *Lost your password?* on the login screen.

?> **The address will change** The site still runs on a temporary address. When the resort moves to its final domain, you will receive the new login address.

## Your menu

As a Resort Manager you only see what you need. Developer tools are hidden, and if you open a page you are not allowed to use, WordPress sends you back.

<figure><img src="assets/img/sidebar.webp" alt="The Resort Manager menu on the left of the admin"><figcaption>The Resort Manager menu. The numbers in circles are things waiting for you.</figcaption></figure>

| Menu item | What it is for | Section |
|---|---|---|
| **Dashboard** | The start page after you sign in | — |
| **Media** | All pictures and PDFs used in the app | [Pictures & files](/en/media) |
| **Pages** | The app screens. Look, but do not edit | [Pages](/en/pages) |
| **Exc. Bookings** | Requests for experiences and spa treatments. The number in a circle counts requests waiting for you | [Bookings](/en/bookings) |
| **Desk Messages** | Messages guests send from the app. The number counts unread messages | [Desk messages](/en/messages) |
| **Offers** | Promotions and seasonal cards | [Offers](/en/offers) |
| **Alerts** | Short notices such as “Pool closed this morning” | [Alerts](/en/alerts) |
| **Daily Content** | Breakfast, lunch and dinner menus for each day | [Menus](/en/menus) |
| **Excursions** | Every activity, tour, event and spa treatment | [Experiences](/en/experiences) |
| **Stays** | One record per guest booking at the resort, with the guest's private link | [Stays](/en/stays) |
| **Rooms** | Room numbers and types | [Rooms](/en/rooms) |
| **Resort Settings** | Resort-wide content: hosts, services, hours, map, island guide, contact email | [Resort Settings](/en/resort-settings) |
| **Profile** | Your own name, email and password | — |

## Golden rules

1. **Always save.** Nothing reaches the app until you press the blue button on the right: **Update** on most screens, **Save** in Excursions, **Publish** for something new.
2. **Change, don't copy.** To keep menus and events current, change the date of an existing item instead of creating a copy. Fewer items means fewer mistakes.
3. **Move to Trash, never delete for good.** The Trash keeps items for 30 days, so mistakes can be undone. See [Restoring from the Trash](/en/troubleshooting?id=i-deleted-something-by-mistake).
4. **Saint Lucia time.** All dates and times in the admin are Saint Lucia time (AST). Midnight is when “today” changes in the app.
5. **Write for guests.** Descriptions, dishes, alert messages and decline notes are read by guests. Use British English and full sentences. Some titles are only for the team: each section says which.
6. **When unsure, ask.** Writing to [support](/en/support) is always better than guessing.

## How the guest app works

A few ideas explain almost everything you will see in this manual.

- **No passwords for guests.** Each stay has a *private link*. Opening it signs the guest in on that phone. Anyone with the link can see that stay, so only send it to the guest.
- **Dates decide what the guest sees.** The app reads the check-in and check-out dates of the stay:

| Guest state | When | What the guest sees |
|---|---|---|
| **Pre-arrival** | Before the check-in date | A welcome screen with the arrival date. Booking is closed |
| **On property** | From the check-in date until the check-out date | The whole app. First a short check-in form (email and consents), then everything opens, including bookings |
| **No stay** | The link is wrong, or the stay was removed | A neutral welcome asking for the personal link |

- **Requests, not payments.** Guests *request* experiences and spa treatments; you *approve* or *decline*. The app never takes money. Paid experiences are settled at the resort.
- **Changes are live.** When you press Update, guests see the change the next time their screen loads.

<figure class="phone"><img src="assets/img/app-today.webp" alt="The Today screen of the guest app on a phone"><figcaption>What a guest sees: the Today screen.</figcaption></figure>
