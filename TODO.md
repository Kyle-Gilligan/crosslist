# TODO

## Security (before launching on app.industrialtreasures.com)
- [x] **Replace the front-end login with real authentication.**
  Done — Amazon Cognito user pool (`us-east-2_As1F7dFY3`) + API Gateway Cognito
  authorizer on `POST /ingest`. The front end signs in via `amazon-cognito-identity-js`
  and sends the ID token on every request. See `AUTH-SETUP.md`.
  - [ ] **Verify the lockdown:** `curl -X POST …/dev/ingest` with no token must return 401.
  - [ ] **Add CORS to API Gateway 4XX gateway responses** (UNAUTHORIZED / ACCESS_DENIED /
    DEFAULT_4XX) so the browser can read a 401 — otherwise an expired-token response is
    blocked by CORS and the UI shows a misleading message.

## Backend correctness (CORS)
- [ ] **Add `Access-Control-Allow-Origin` to the Lambda's error return paths.**
  Success (200) responses include it, but the 404 ("SKU not found") and 400 paths do
  not — so the browser blocks them and the frontend can't read the real status.
  Add the header to every `return` in the Lambda, then redeploy.
- [ ] **Preflight allowed-methods.** `OPTIONS` response currently returns
  `Access-Control-Allow-Methods: OPTIONS` only — should be `OPTIONS,POST`. Update the
  OPTIONS method's Integration Response header mapping in API Gateway and redeploy.

## Features (nice to have)
- [ ] Item preview (name/price/image) before sending, to confirm the right SKU.
- [ ] Per-channel result feedback (BigCommerce ✓ / eBay ✓ / Etsy ✗) instead of one combined message.
- [ ] Recent activity log of recently listed SKUs + status.
- [ ] Add Chairish and Facebook Marketplace channels (in the backend spec, not yet in UI).
- [ ] Status view backed by the DynamoDB cross-reference matrix (DRAFT / LIVE / SOLD).
- [ ] Mark-sold / delist button.
