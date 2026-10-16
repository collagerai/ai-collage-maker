# Collager.ai REST API v0 - quick reference

Base URL: `https://api.collager.ai/v1` (same API at `https://collager.ai/v1`). Auth: `Authorization: Bearer ck_live_...`
(or `ck_test_...` - watermarked previews only). Keys: https://collager.ai/developers/#request. OpenAPI:
https://collager.ai/developers/openapi.yaml

## Shape collage (seconds)

```bash
curl -s https://api.collager.ai/v1/collages \
  -H "Authorization: Bearer <YOUR_API_KEY>" -H "Content-Type: application/json" -H "Prefer: wait=25" \
  -d '{"shape": {"type": "number", "text": "30"},
       "photo_urls": ["https://example.com/1.jpg", "https://example.com/2.jpg"]}'
```

Answer: `{"id":"clg_...","status":"preview_ready","preview_url":"...","cells":116,...}`. Shapes: `GET /v1/shapes`.
Options: `gap_pct` (0-40, default 6), `size_px` (600-8000, default 3000), `background` (`white`, `cream`, `#rrggbb`),
`metadata` (your strings, returned as is), `webhook_url`. Multipart: `photos[]` files and/or `photos_zip`, JSON in the
`payload` field. `POST /v1/collages/{id}/unlock` -> `hd_url` (live key). `DELETE /v1/collages/{id}` -> 204.

## Photo mosaic (minutes)

```python
import requests, time
KEY = "<YOUR_API_KEY>"                              # your own key from collager.ai/developers - keep it in your config
H = {"Authorization": "Bearer " + KEY, "Idempotency-Key": "order-1042"}
job = requests.post("https://api.collager.ai/v1/mosaics", headers=H, json={
    "main_image_url": "https://example.com/portrait.jpg",
    "tile_image_urls": tiles,                       # 30..10000 links
    "featured_tile_urls": favourites,               # optional: bigger tiles, placed first
    "style": {"layout": "variable", "colorization": "medium"},
    "metadata": {"order": "1042"},
}).json()
while job["status"] not in ("preview_ready", "failed"):
    time.sleep(30)
    job = requests.get("https://api.collager.ai/v1/mosaics/" + job["id"], headers=H).json()
print(job["preview_url"])
```

Statuses: `fetching` (we download your links) -> `queued` -> `processing` -> `preview_ready` -> `unlocked`, or `failed`
(`error.code`). Style: `layout` variable|uniform, `large_tiles` (default 300), `size_levels` 1-4, `colorization`
low|medium|high, `blur_background` 0-100, `clean_subject` true/false. `collect_uploads: true` -> `upload_url` page for
people to add photos, then `POST /v1/mosaics/{id}/start`. `POST /v1/mosaics/{id}/unlock` -> `hd_url`;
`POST /v1/mosaics/{id}/layered` -> `psb_url` (layered file for print) when `psb_status` is `ready`.
`GET /v1/mosaics/{id}/layout` -> every tile: `photo`, `source_url`, `x`, `y`, `w`, `h` (px of the full file), `level`,
`featured`, `avg_color`; `?format=xml` for XML.

## Webhooks

`POST` to your `webhook_url` with `{"event": "mosaic.preview_ready" | "mosaic.failed" | "mosaic.unlocked" |
"mosaic.layered_ready" | "collage.preview_ready" | ..., "mosaic"|"collage": {...}}`, headers `X-Collager-Event`,
`X-Collager-Delivery`, `X-Collager-Signature: sha256=<hex HMAC-SHA256 of the raw body with your webhook secret>`.
Answer 2xx within 10 s; otherwise we retry after 1 m, 5 m, 30 m, 2 h, 6 h, 24 h.

## Errors and limits

`{"error": "code", "message": "..."}`: 400 invalid_request / invalid_shape / invalid_style / too_few_photos, 401
unauthorized / invalid_key, 402 insufficient_credits, 403 test_key, 404 not_found, 409 not_ready, 429 rate_limited /
too_many_jobs / daily_limit / busy (with `Retry-After`). Per key: 60 requests/min, 3 mosaics in progress.
