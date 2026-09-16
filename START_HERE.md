# 🏬 Vendor Portal — Everything You Need (Simple Version)

> The detailed technical guide is in `VENDOR_SETUP.md`. This is the short version.

## 📁 Files you got (all in this folder)

| File | What to do with it |
|---|---|
| `vendor.html` ⭐ | **NEW.** The vendor portal. Upload it beside your other pages. |
| `home.html` ⭐ | **UPDATED.** Your customer app, now with live vendor menus. **Replaces your current `home.html`.** |
| `seed-menus.html` | **TEMPORARY.** Run once as admin, then **delete it**. |
| `firestore.rules` ⭐ | **Complete rules file — replaces your current rules.** Just paste the whole thing (step 2). |
| `VENDOR_SETUP.md` | Long technical guide. Optional reading. |
| `runner.html` ⭐ | **UPDATED.** Runner dashboard now has a 🔔 alerts toggle (ding + popup + vibrate on new orders). Replaces your current `runner.html`. |
| `EMAIL_SETUP.md` | Runner-alert setup: sound alerts work already; Gmail emails need the 10-min EmailJS setup inside. |

Everything else in this folder (`index.html`, `relay.html`, etc.) is just an untouched copy — ignore those.

## 🔑 Stall codenames (the important part)

To give a vendor access, you put one of these codes on their account.
**Must be UPPERCASE** — `SY` works, `sy` does not.

| Code | Stall | Code | Stall |
|---|---|---|---|
| `AZ` | Auntie Zeny's | `FT` | Food Trip |
| `KK` | KK Food Station | `HO` | Healthy Options |
| `SV` | Snackville | `JC` | Jacky Chow |
| `SY` | Syd's Milktea House | `LV` | La Vivienda |
| `KRT` | The KRT Milktea | `MT` | Mang Tinapay |
| `EB` | Easybite | `MC` | Matcha Cafe |
| `WC` | Wingcraft | `PR` | Papito's Restaurant |
| `AF` | Alcuis Food | `RS` | Rosemin's Snack House |
| `BB` | Bread and Beyond | `TK` | Turks |
| `CG` | Chicks n' Gravy | | |
| `CR` | Creamery Rock | | |
| `DT` | Dew's Tea House Takoyaki | | |

## 🚀 Setup — do once (about 10 minutes)

**1. Upload the pages**
Upload `vendor.html` and the new `home.html` to wherever your app is hosted.
Also upload `seed-menus.html` for now (you'll delete it in step 4).

**2. Replace your security rules**
Firebase Console → Firestore Database → Rules tab. Delete everything there,
paste in the entire `firestore.rules` file, and click **Publish**. (It's your
exact current rules + the vendor additions — nothing removed.)

**3. Log in as admin**
Open your normal Re:Quest app and log in with your **admin** account.

**4. Seed the menus**
Open `seed-menus.html` in your browser → click **Seed Menus to Firestore**.
Wait for "DONE ✓". Then **delete `seed-menus.html`** from your hosting.

**5. Give a vendor access**
1. After you've talked to the stall owner, have them log into `vendor.html`
   with their normal Re:Quest account. They'll see "⏳ Waiting for Permission".
2. In Firebase Console → Firestore → `users` → find their account → add a field:
   - field name: `vendorStore`
   - type: **string**
   - value: their stall code (e.g. `SY`)
3. Their dashboard **unlocks automatically** — they don't even refresh. ✅

To **revoke** access: delete the `vendorStore` field from their account.

## 🧑‍🍳 What to tell stall owners

- Log into `vendor.html` with your **normal account** (same Gmail/password).
- First time? Tap **📥 Import** to pull in your stall's existing menu — then edit prices, add photos, mark sold out, or delete whatever you no longer serve.
- Sold out? Tap **Mark sold out** — students see it instantly.
- Restocked? Tap **Mark available**. Start of day? One tap resets everything.
- Closed today? **Close store** button. Prices/photos/description: edit anytime, live immediately.

## 🔔 Runner order alerts (new)

- **Sound + popup:** re-upload the new `runner.html`. Runners tap **🔔 Alerts off**
  once → new orders ding, pop up, and vibrate while the page is open. No setup.
- **Gmail emails:** follow `EMAIL_SETUP.md` (~10 min, free) so every food order
  also emails all runners' Gmails at once.

## ❓ If something's wrong

- **Vendor stuck on "Waiting for Permission"** → check the `vendorStore` field:
  spelled right? UPPERCASE? On the correct account?
- **"Stall Code Not Recognized"** → the code has a typo. Fix it in Firestore.
- **Menus look empty for everyone** → the seed didn't run. Re-upload
  `seed-menus.html`, run it as admin, then delete it.
- **One stall per account.** If someone runs two stalls, they need a second
  Re:Quest account for the other one.
