# obx-owners — Claude Code context

Static frontend for owners.obx.deals (GitHub Pages + Alpine.js + Auth0 PKCE).
API backend at owners-api.obx.deals (Lambda ECR image, rebuilt nightly at 06:15).
Auth: Auth0 tenant dev-4fg3icvrjdrj4o4a.us.auth0.com, SPA client OQKSijQtrT6jcqIDIPNbbzzACig8TdXE.

## Operational standards

**Observe before acting.** If a chart is wrong, data is missing, or a route
returns unexpected results — start with the data source, not the code.
owner.db is a nightly export from rental-intel; it is read-only in the Lambda.
A stale owner.db (not yet rebuilt after a scrape) is the most common cause of
data issues. Check the Lambda's last-modified timestamp and the owner.db export
time before concluding there is a bug.

**Verify before fixing.** Confirm the root cause from data before changing code.
Is the Lambda running the latest image? Is owner.db current? Does the raw API
response match what the frontend shows?

**Separate diagnosis from action.** State the observation and its source.
Confirm it. Then decide on the fix.

**Never write to owner.db directly.** It is a derived export rebuilt by
`scripts/export_owner_db.py` in rental-intel. All source-of-truth data lives
in rental-intel's SQLite DB and DynamoDB tables.

## Architecture

- `index.html` — single-page app. Auth0 PKCE flow; JWT-gated API calls.
- `owner.db` — lean SQLite baked into the Lambda Docker image at deploy time.
  Tables: properties, weekly_rates, comp_percentiles, booking_pace,
  short_stay_rates, sos_enhanced_weekly, owner_booked_weeks.
- Stripe: currently test mode (sk_test_...). Flip to live via Secrets Manager
  — no code change needed. See ADR 0120 Phase 4.

## Key ADRs

- ADR 0120 — Owner Hub auth, hosting, tiering, Stripe
- ADR 0124 — Chart rendering (Chart.js create-once update-in-place)
- ADR 0136 — Multi-property portfolio support
- ADR 0140 — Badge claim in-hub form

## Deployment

Changes to `index.html` go live on push to `main` (GitHub Pages, ~30s).
The Lambda image is rebuilt at 06:15 by `deploy_obx_deals.sh` in rental-intel
via `make update-owners-api`. Lambda code changes in
`rental-intel/terraform/lambda/owners_api/` require a manual `make update-owners-api`
to take effect before the next 06:15 cycle.
