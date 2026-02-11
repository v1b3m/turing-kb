# Connecting Stripe (Connect OAuth Setup)

## Stripe Dashboard Setup

1. Go to **Stripe Dashboard** → **Settings** → **Connect** → **Onboarding options** → **OAuth**
   - Direct URL: `https://dashboard.stripe.com/settings/connect/onboarding-options/oauth`
   - Test mode: `https://dashboard.stripe.com/test/settings/connect/onboarding-options/oauth`
   - NOTE: The OAuth settings are NOT on the main Connect overview page. They're under Settings → Connect → Onboarding options → OAuth specifically.
   - NOTE: The "Created apps" tab under API keys is for **Stripe Apps** (marketplace), NOT Connect OAuth. Don't confuse the two.

2. **Enable OAuth** toggle

3. Copy the **Client ID** (`ca_xxx`) — this is your `STRIPE_CONNECT_CLIENT_ID`

4. Add **Redirect URIs**:
   - Local dev: `http://localhost:3000/api/oauth/stripe/callback`
   - Local dev (desktop): `http://localhost:3000/desktop/stripe/callback`
   - Production: `https://bfloat.dev/api/oauth/stripe/callback`
   - Production (desktop): `https://bfloat.dev/desktop/stripe/callback`

## Environment Variables

Set in `.env` (app-engineer):

```
STRIPE_CONNECT_CLIENT_ID=ca_xxx        # from step 3 above
STRIPE_SECRET_KEY=sk_test_xxx          # platform's secret key (used as client_secret in token exchange)
STRIPE_WEBHOOK_SECRET=whsec_xxx        # for webhook signature verification (optional for Connect OAuth)
```

Optional (defaults to localhost:3000 if not set):
```
STRIPE_CONNECT_REDIRECT_URI=https://bfloat.dev/api/oauth/stripe/callback
STRIPE_CONNECT_DESKTOP_REDIRECT_URI=https://bfloat.dev/desktop/stripe/callback
```

## Troubleshooting

### "No application matches the supplied client identifier"
- `STRIPE_CONNECT_CLIENT_ID` is wrong or not set. Get it from the OAuth settings page (step 3 above).

### "Invalid redirect URI"
- The redirect URI sent during OAuth doesn't match what's configured in Stripe.
- The app uses `STRIPE_CONNECT_REDIRECT_URI` / `STRIPE_CONNECT_DESKTOP_REDIRECT_URI` env vars (falls back to `http://localhost:3000/...`).
- Make sure the URI in your env matches exactly what's listed in Stripe's OAuth redirect URI settings.

### "Authorization code provided does not belong to you"
- **The `STRIPE_SECRET_KEY` doesn't match the account that owns the `STRIPE_CONNECT_CLIENT_ID`.**
- Both must belong to the same Stripe account (e.g., `acct_1Q0rBpFWPtXFqtnw`).
- If you're in **test/sandbox mode**, the secret key must be `sk_test_xxx` (not `sk_live_xxx`).
- If you're in **live mode**, the secret key must be `sk_live_xxx`.
- The mode of the secret key must match the mode the OAuth flow was initiated in.

## How It Works

- Auth URL: `https://connect.stripe.com/oauth/authorize`
- Token URL: `https://connect.stripe.com/oauth/token` (uses `STRIPE_SECRET_KEY` as `client_secret`)
- Deauthorize URL: `https://connect.stripe.com/oauth/deauthorize`
- Returns `stripe_user_id`, `access_token`, `stripe_publishable_key` (permanent, no refresh token)
- Tokens are valid until the connected account deauthorizes

## Notes

- Stripe Connect OAuth is **not deprecated** but is discouraged for new platforms. It still works for connecting existing Stripe accounts.
- There are separate `ca_xxx` Client IDs for test mode and live mode.
- When exchanging the auth code, the API key mode (test/live) must match the authorization code mode.
- The deep link callback (`bfloat://stripe/callback`) requires `onStripeCallback` in `lib/conveyor/api/app-api.ts` and a handler in `lib/main/main.ts`.
