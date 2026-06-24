# crosslist

Cross-list tool for **Industrial Treasures** — type a SKU, pick channels
(BigCommerce / eBay / Etsy), and send it to the ingest API.

It POSTs to the ingest API with:

```json
{ "sku": "ANT-001", "target_channels": ["bigcommerce", "ebay", "etsy"] }
```

## Run

```bash
python3 -m http.server 8000
```
Open http://localhost:8000

Login is handled by **Amazon Cognito** (real auth — see `AUTH-SETUP.md`). The pool/client
IDs and ingest endpoint are set near the top of the `<script>` in `index.html`.
