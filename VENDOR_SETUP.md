# 🏬 Vendor Portal — Setup Guide

Vendors log in with their **existing Re:Quest account** (same Gmail +
password — no second account). You grant access by putting a **stall
codename** on their `users` doc. Every change they make goes **live to
students in `home.html` instantly** — no refresh needed.

## Files

| File | What it is |
|---|---|
| `vendor.html` | The vendor portal (login → waiting room → dashboard). **New.** |
| `home.html` | Customer app. **Updated** — menus load live from Firestore with the built-in menu as fallback. |
| `seed-menus.html` | One-time admin utility. Seeds menus + stall codenames. **Delete after running.** |
| `firestore.rules` ⭐ | **Complete merged rules file.** Paste the whole thing over your current rules. |

## New Firestore data

- **`users/{uid}.vendorStore`** — the stall codename you set (e.g. `"SY"`).
  This single field IS the vendor grant. No code = no dashboard.
- **`menu_items/{autoId}`** — live menu items
  (`{ storeId, storeCode, name, price, desc, emoji, photoURL, available, … }`).
- **`stores/{storeId}`** — now also holds `{ code, name, emoji, desc, photoURL }`
  alongside the existing `{ closed, closedBy, closedAt }`.

## Stall codenames

| Code | Stall | ID | Code | Stall | ID |
|---|---|---|---|---|---|
| `AZ` | Auntie Zeny's | 1 | `FT` | Food Trip | 13 |
| `KK` | KK Food Station | 2 | `HO` | Healthy Options | 14 |
| `SV` | Snackville | 3 | `JC` | Jacky Chow | 15 |
| `SY` | Syd's Milktea House | 4 | `LV` | La Vivienda | 16 |
| `KRT` | The KRT Milktea | 5 | `MT` | Mang Tinapay | 17 |
| `EB` | Easybite | 6 | `MC` | Matcha Cafe | 18 |
| `WC` | Wingcraft | 7 | `PR` | Papito's Restaurant | 19 |
| `AF` | Alcuis Food | 8 | `RS` | Rosemin's Snack House | 20 |
| `BB` | Bread and Beyond | 9 | `TK` | Turks | 21 |
| `CG` | Chicks n' Gravy | 10 | | | |
| `CR` | Creamery Rock | 11 | | | |
| `DT` | Dew's Tea House Takoyaki | 12 | | | |

> ⚠️ Codes are **UPPERCASE** (`SY`, not `sy`). Firestore rules compare
> them exactly — a lowercase code will not grant access.

## Setup (do once, in order)

### 1. Upload the files
Deploy `vendor.html` and the updated `home.html` alongside your other pages.
Also upload `seed-menus.html` temporarily.

### 2. Replace your Firestore rules
Firebase Console → Firestore → Rules. Select-all, delete, and paste the
entire `firestore.rules` file. Publish. It's your exact current rules plus
the vendor additions (helpers, `menu_items` block, vendor access on `stores`,
and `vendorStore` protection on `users`) — nothing removed.
*(If you deployed any earlier vendor rules snippet, `firestore.rules` replaces it completely.)*

### 3. Seed the menus (as admin)
1. Log in to the main app as an **admin** account.
2. Open `seed-menus.html` and click **Seed Menus to Firestore**.
3. It writes ~280 items (with stall codes) + stamps every `stores/{id}`
   doc with its codename. Skips the "Menu not yet available" placeholders.
4. It refuses to run twice unless you tick "force".
5. **Delete `seed-menus.html`** when done.
6. *Seeded before the codename update?* Click **Backfill stall codes**
   instead — it stamps codes onto your existing data without duplicating.

### 4. Grant a vendor access
1. After you've coordinated with the stall owner offline, have them log into
   `vendor.html` with their normal Re:Quest account — they'll see a
   "Waiting for Permission" page.
2. In Firestore, open their `users/{uid}` doc and add/update the field:
   - field: `vendorStore`
   - type: **string**
   - value: the stall's code, e.g. `SY`
3. Their dashboard unlocks **automatically**, even mid-session. ✅

To revoke: delete the `vendorStore` field (or set it to `null`).

## Daily vendor workflow (what to tell stall owners)

- **Sold out?** Tap `Mark sold out` on the item — students instantly see it
  greyed out with a badge and can't add it to cart.
- **Restocked?** Tap `Mark available`.
- **Start of day:** `☀️ Start of day — all available` resets everything in one tap.
- **Closed today?** `🔒 Close store` — same flag admins use; the stall shows
  as Closed to everyone.
- Prices, descriptions, photos: edit anytime, live immediately.

## ⚠️ Things to know

1. **Vendors use their normal account.** Being a vendor doesn't change
   anything else — they can still order food, run, chat, etc. with the
   same login.
2. **One stall per account.** Each account holds one codename. If a person
   runs two stalls, they'd need a second Re:Quest account for the other one.
3. **Stalls without a vendor are unaffected.** If a store has no live items
   in Firestore, `home.html` silently shows the original built-in menu —
   onboard stalls one at a time.

## Suggested next features (when you're ready)

- Vendor order view (incoming orders per stall).
- In-app admin approvals (grant codenames without opening the console).
- Vendor badge in Campus Directory / chat.
- Vendor sales history / daily summary.
