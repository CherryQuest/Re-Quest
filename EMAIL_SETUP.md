# 🔔 Runner Alerts — Setup Guide

Two parts: **(A) sound + popup alerts** work already, nothing to do.
**(B) Gmail emails to runners** need the ~10-minute setup below.

## A. Sound + popup alerts (done — no setup)

Re-upload `runner.html`. Each runner taps **🔔 Alerts off** (top bar) once →
it becomes **🔔 Alerts on**. From then on, every new food order plays a ding,
pops an OS notification, and vibrates their phone while the runner page is open.
The choice is remembered per device.

- Only **new, unassigned** orders alert — no noise for delivered/taken ones,
  and opening the page never dings for old orders.
- iPhone runners: Safari → Share → **Add to Home Screen**, then use that icon
  for popups to work. (Android/PC work in the normal browser.)

## B. Email every runner's Gmail on each order (needs setup)

Your app uses **EmailJS** (free tier: **200 emails/month**, and each order
sends exactly 1 email to all runners at once — so ~200 orders/month free).
Outgrow it later? Tell me and I'll switch you to a no-monthly-cap relay.

### Step 1 — Create the free account
1. Go to **emailjs.com** → **Sign Up** (free) → verify your email.

### Step 2 — Connect a Gmail (this address SENDS the alerts)
1. Left sidebar → **Email Services** → **Add New Service** → **Gmail**.
2. Click **Connect Account**, pick the Gmail the alerts come from
   (your own, or a dedicated one like `request.alerts@gmail.com`).
3. Click **Create Service**. Copy the **Service ID** (looks like `service_ab12cd`).

### Step 3 — Make the "New order" template
1. Left sidebar → **Email Templates** → **Create New Template**.
2. **Settings tab:**
   - Template Name: `ReQuest New Order` (anything)
   - **To Email:** `{{to_email}}`
   - **Subject:** `🍔 New order #{{order_id}} — {{stall_list}} ({{total}})`
   - From Name: `Re:Quest Alerts` (anything)
3. **Content tab** — delete everything, paste this:
   ```
   A customer just placed an order — a runner is needed! 🚴

   Order #{{order_id}} · {{item_count}} items · {{total}}
   Placed {{placed_at}}

   {{items_text}}

   ------------------------------
   Deliver to: {{location}}
   Customer: {{customer_name}} ({{phone}})

   Open the runner portal to accept:
   {{runner_url}}

   — Re:Quest auto-alert (1 email per order to all runners)
   ```
4. Click **Save**. Copy the **Template ID** (looks like `template_xy34zt`).

### Step 4 — Get your Public Key
1. Click your avatar (top right) → **Account** → **General**.
2. Copy the **Public Key** (looks like `AbCdEfGhIjKlMnOp`).

### Step 5 — Plug 3 values into home.html
Open `home.html`, search for `PASTE_EMAILJS_PUBLIC_KEY`, and fill in:

```js
const EMAIL_ALERTS = {
  enabled: true,               // ← flip to true
  publicKey:  'AbCdEfGhIjKlMnOp',   // ← Step 4
  serviceId:  'service_ab12cd',     // ← Step 2
  templateId: 'template_xy34zt',    // ← Step 3
};
```

(The runner link in emails is auto-detected from your site, so there's
no URL to configure. All your pages just need to sit side by side on
your hosting, which is the normal setup.)

Re-upload `home.html`. **Or just send me the 3 IDs and I'll paste them for you.**

### Step 6 — Test it
1. Place a real test order from a customer account.
2. Check every runner's Gmail inbox. First email may land in **Spam** →
   open it → **Report not spam**. (Optional: in Gmail, create a filter for
   `from:(your-sender-gmail)` → **Never send to Spam** + **Always mark important**.)
3. Runners should also turn **Gmail app notifications ON** so the phone buzzes.

## ❓ If emails don't arrive

- `enabled: true` set? All 3 IDs pasted with no extra spaces?
- EmailJS dashboard → **History** shows every send + errors — check there first.
- "The recipients address is empty"? No users with `role == 'runner'` have an
  `email` field — approve/promote runners first.
- Free quota used up (200/month)? Dashboard shows usage. Ask me about options.

## 🔒 Notes

- The Public Key is meant to be public (it's in the page code by design).
  EmailJS rate-limits your account, and the template can only send your
  fixed order format — nobody can use it to send arbitrary spam.
- Only users whose `users` doc has `role == 'runner'` get emailed, max 100.
  Admins are NOT emailed (promote yourself to runner too if you want them —
  note the app reads one role, so use a second account for that if needed).
