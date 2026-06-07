# TenantSync Website — Claude Code Context

## What This Repo Is
The LIVE TenantSync marketing site. Static HTML/CSS/JS, no build step.
Files: `index.html` (landing), `privacy.html`, `terms.html`, `README.md`.
Auto-deploys to **www.tenantsync.io** from `main` via Vercel (push = deploy to prod).

## Hard Rules
- **Redesigns/restyles MUST preserve functionality.** Treat the current `index.html`
  as the source of truth. Change layout/CSS/copy only — do NOT drop scripts, links,
  or the checkout flow. (A June 2026 redesign silently deleted the entire Stripe
  checkout and turned every CTA into `href="#"`. Don't repeat it.)
- **Before pushing any redesign:** diff against the prior version and confirm EVERY
  `href`/CTA and the `startCheckout()` flow survived. Then verify on the live deploy
  (`curl https://www.tenantsync.io | grep startCheckout`), not just locally.
- Ask before committing/pushing — pushing to `main` deploys to the live site.
- 100% ASCII in code; inline `onclick` is fine here (this is a normal web page, not
  the MV3 extension — different CSP rules).

## Signup / Payment Flow (critical — do not break)
- ALL signup CTAs (hero, bottom CTA, pricing Free "Get Started Free", pricing Pro
  "Start Pro") call `startCheckout(this)`.
- `startCheckout()` POSTs to `https://singularity-dashboard-alpha.vercel.app/api/billing?action=create-checkout`
  with `{ price_id: TS_PRICE_ID, product: 'tenantsync' }`, then redirects to the
  returned Stripe URL. The server applies a 14-day Pro trial (`trial_period_days=14`).
- **There is no other signup path.** The `register` API is admin-only, so Stripe
  checkout is the ONLY way a user gets an account + API key. Free tier = the
  post-trial downgrade state, reached through the same checkout.
- **"Add to Chrome" buttons** (nav, footer) are a SEPARATE action (install the
  extension) and should point to the Chrome Web Store listing URL once it exists.
  Currently `href="#"` placeholders pending CWS approval.

## Known TODO
- `TS_PRICE_ID` in `index.html` is currently `price_1T9GpF...` — the **TEST-mode $39**
  price (`livemode:false`), while the page advertises **$10/mo**. Swap for the LIVE
  $10/mo price (`STRIPE_PRICE_TS_PRO_MONTHLY`) as part of the Stripe live switch
  before taking real customers. Also update `PRICE_TIER_MAP` in the server `billing.js`.

## Related Repos
- Server/API (Stripe checkout, billing): `JoeG592/singularity-dashboard` -> `api/billing.js`.
- Extension: `JoeG592/tenantsync-extension`.
