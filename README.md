# obx-owners

Static frontend for [owners.obx.deals](https://owners.obx.deals) — the OBX Deals Owner Hub.

Hosted on GitHub Pages (CNAME: owners.obx.deals). Deployed automatically on push to `main`.

## What it is

A private dashboard for OBX rental owners. Auth0 PKCE login gates access to:
- Comp percentile charts (p25/p50/p75 vs. your rate by week)
- Booking pace (booked vs. open weeks, orange/green dots)
- Rate positioning vs. market
- Gap detection and short-stay opportunities
- Verified view badge claim form

**Free tier:** comp p50 preview only.  
**Pro tier ($199/season):** full p25/p75, own rate overlay, booking pace, booked weeks.

## Auth

Auth0 tenant: `dev-4fg3icvrjdrj4o4a.us.auth0.com`  
SPA client ID: `OQKSijQtrT6jcqIDIPNbbzzACig8TdXE`  
API audience: `https://owners-api.obx.deals`

## How content gets here

`index.html` is edited directly in this repo. The Lambda API is rebuilt and deployed separately:

```bash
# In rental-intel/:
make update-owners-api   # export owner.db → Docker build → ECR → Lambda update
```

## Developer notes

### Stripe (currently test mode)

Test card: `4242 4242 4242 4242`. Keys are in AWS Secrets Manager (`stripe/secret_key`, `stripe/webhook_secret`). To go live: swap to live keys in Secrets Manager + update `STRIPE_PRICE_ID` Lambda env var. No code change needed.

### Manually setting a user's tier (for testing)

```bash
aws dynamodb update-item --table-name owner_hub_users \
  --key '{"user_id":{"S":"auth0|..."}}' \
  --update-expression "SET #t = :v" \
  --expression-attribute-names '{"#t":"tier"}' \
  --expression-attribute-values '{":v":{"S":"pro"}}' --region us-east-1
```

## Related repos

| Repo | Site | Purpose |
|---|---|---|
| [rental-intel](https://github.com/tiddnet/rental-intel) *(private)* | — | Data pipeline, scrapers, Lambda infra, deploy scripts |
| [obx-deals](https://github.com/tiddnet/obx-deals) | obx.deals | Deal listings + verified view badges |
| [obx-search](https://github.com/tiddnet/obx-search) | search.obx.deals | Consumer property search |

See `rental-intel/README.md` for the full system architecture, data flow, and AWS infrastructure.
</content>
</invoke>