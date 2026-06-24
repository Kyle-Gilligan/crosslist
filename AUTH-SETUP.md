# Authentication setup (Amazon Cognito)

Real auth for the cross-list tool. The security boundary is the **API**, not the UI —
the front-end login only matters because the API now rejects requests without a valid
Cognito token.

## What's deployed

- **Cognito User Pool:** `us-east-2_As1F7dFY3` (region `us-east-2`)
- **App client (public SPA, no secret):** `251ifcg5qrk0v5lc02n1r2fn29`
- **Shared user:** one login that all staff use
- **API:** existing `POST /ingest` on the `dev` stage, protected by a Cognito authorizer

These IDs live near the top of the `<script>` in `index.html`.

## How it works

1. Staff sign in on the antique login card → `amazon-cognito-identity-js` authenticates
   against the user pool (SRP, password never sent in the clear).
2. On success the browser holds a short-lived **ID token** (in `sessionStorage`, so
   closing the tab signs out). The library auto-refreshes it using the refresh token.
3. Every `POST /ingest` sends the raw ID token in the `Authorization` header.
4. The API Gateway **Cognito authorizer** validates it; no token / bad token → **401**.

## Managing the shared login

Set or reset the password (permanent, so there's no forced change on next sign-in):

```bash
aws cognito-idp admin-set-user-password \
  --user-pool-id us-east-2_As1F7dFY3 \
  --username industrial \
  --password 'YourStrongSharedPassword' \
  --permanent --region us-east-2
```

## Still to verify / finish

- [ ] **Confirm the lockdown:** `curl -X POST https://…/dev/ingest -d '{}'` with no
      `Authorization` header must return **401 Unauthorized**.
- [ ] **CORS on the authorizer's rejections:** in API Gateway → **Gateway Responses**,
      add `Access-Control-Allow-Origin` (and `Access-Control-Allow-Headers: Authorization,Content-Type`)
      to **UNAUTHORIZED**, **ACCESS_DENIED**, and **DEFAULT_4XX**, then redeploy.
      Without this the browser can't read a 401 and the UI shows a misleading message.
- [ ] **Preflight:** the `OPTIONS` method on `/ingest` must stay Authorization **NONE**,
      and `Access-Control-Allow-Headers` must include `Authorization`.

## Rotating credentials / offline note

- The page loads `amazon-cognito-identity-js` from jsDelivr CDN. To keep the page fully
  offline-capable, download that file into `fonts/`-style local vendoring and point the
  `<script src>` at it.
