# Deploying Stripe payments — do this once Stripe verification clears

This code is ready now, but three things have to be true before it will actually run:

1. Your Stripe account shows **fully verified** (not "pending") in the Stripe dashboard.
2. Your Firebase project is upgraded from Spark (free) to **Blaze** (pay-as-you-go) —
   Cloud Functions require it. Blaze still has a generous free tier; you're not
   signing up for a big bill, just enabling billing to exist.
3. You have Node.js installed locally, and the Firebase CLI (`npm install -g firebase-tools`).

## 1. Get your Stripe keys
In the Stripe dashboard: Developers → API keys. Copy the **secret key** (starts with `sk_`).
You'll get the **webhook signing secret** in step 4, after the webhook exists.

## 2. Set the keys in Firebase
From this project's root folder (where `firebase.json` lives), run:

```
firebase login
firebase use --add          # pick your dominioncity-30d77 project
firebase functions:config:set stripe.secret_key="sk_live_..."
```

## 3. Install dependencies and deploy the functions

```
cd functions
npm install
cd ..
firebase deploy --only functions
```

This gives you three live function URLs — note the `stripeWebhook` one, you need it next.

## 4. Register the webhook in Stripe
Stripe dashboard → Developers → Webhooks → Add endpoint.
- URL: the `stripeWebhook` URL from step 3
- Event to listen for: `checkout.session.completed`

Stripe will show you a **signing secret** (starts with `whsec_`). Set it:

```
firebase functions:config:set stripe.webhook_secret="whsec_..."
firebase deploy --only functions
```

## 5. Connect the frontend
Two small additions are still needed once you reach this point (not built yet,
since there was nothing to test against until now):

- A **"Connect payout account"** button in `dashboard.html`'s Profile tab that
  calls `createSellerOnboardingLink` and redirects the seller to the returned URL.
- A **"Pay now"** button in `buyer-dashboard.html`, shown on offers with
  `status: 'accepted'`, that calls `createCheckoutSession` with the `offerId`
  and redirects the buyer to the returned Stripe Checkout URL.

Say the word once you're at this stage and I'll wire both of those in — they're
small, but need the live function URLs to test against, which don't exist until
you've deployed.

## In the meantime
The admin dashboard's **"Completed sales"** panel already lets you confirm a
sale manually (fee split calculated the same way, `method: 'manual'` instead
of `'stripe'` on the transaction record). Reviews work off that already — you
don't have to wait on Stripe to start collecting them.
