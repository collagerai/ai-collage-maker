---
name: photo-collage-maker
description: Photo mosaic maker and AI photo collage maker - make photo mosaics and photo collages from the user's own photos with Collager.ai - a heart, a number (birthday, anniversary, year), a word, a star or a flower made of photos in seconds, or a photo mosaic (one big picture recreated from hundreds or thousands of small photos). Use when the user asks for a photo collage, heart photo collage, collage in the shape of a number or word, "make a picture out of these photos", photo mosaic, mosaic portrait, memorial or anniversary gift from family photos, or wants to turn many photos into one image. Works through the Collager.ai MCP tools (no sign-in, no key); otherwise guides the user to collager.ai. For developers it points to the Collager.ai REST API documentation.
license: MIT
metadata:
  homepage: https://collager.ai/developers/
  version: "0.1.0"
---

# Photo mosaic & AI photo collage maker (Collager.ai)

Two products, pick by what the user wants:

| The user wants | Product | Time | How many photos |
|---|---|---|---|
| A heart / number / word / star / flower **made of photos** | **Shape collage** | seconds | 1-300 (fewer photos than cells = photos repeat) |
| **One picture** (a portrait, a couple, a pet) **recreated from many small photos** | **Photo mosaic** | minutes | 100-2000 is best (minimum 30) |

If unsure, ask one question: "Should the photos form a shape (like a heart or the number 30), or should they recreate one main photo?"

## Route 1 - Collager.ai MCP tools are connected (preferred)

Tools of the `collager-ai` server: `make_shape_collage`, `get_shape_collage`, `list_collage_shapes`,
`list_mosaic_styles`, `start_photo_mosaic`.

1. **Shape collage:** call `make_shape_collage` with `shape` and the photos:
   - photos the user attached in the chat -> `photos` (ChatGPT passes file objects); photos on the web -> `photo_urls`;
   - shape examples: `{"type":"heart"}` (size chosen from the photo count), `{"type":"heart","cells":70}`,
     `{"type":"number","text":"30"}`, `{"type":"text","text":"LOVE"}`, `{"type":"love"}` (the words "I love you"),
     `{"type":"star","size":"large"}`, `{"type":"grid","n":4}`. Call `list_collage_shapes` when the user asks what is possible.
   - If the answer says `processing`, call `get_shape_collage` with the `id` a few seconds later.
   - Show the preview image and give the `full_resolution_page` link: the full-resolution file without the watermark
     is downloaded there.
2. **Photo mosaic:** call `list_mosaic_styles` if the user wants to choose a look, then `start_photo_mosaic` and give
   the link with its steps. The mosaic is built on collager.ai from the photos the user uploads there.

## Route 2 - a developer integrating Collager.ai into their own product

Only when the user is writing their own code (a print shop, a photo app, an agent). Do not look for API keys on the
user's machine and do not ask them to paste a key into the chat: the developer puts their own key into their own code.
Base URL `https://api.collager.ai/v1` (also `https://collager.ai/v1`), header `Authorization: Bearer <YOUR_API_KEY>`,
keys at https://collager.ai/developers/#request. Full contract: https://collager.ai/developers/openapi.yaml.
Details and examples: [references/api.md](references/api.md).

- Shape collage: `POST /v1/collages` with `{"shape": {...}, "photo_urls": [...]}` and `Prefer: wait=25` -> usually
  `preview_ready` in the answer; `POST /v1/collages/{id}/unlock` -> `hd_url` (live keys).
- Photo mosaic: `POST /v1/mosaics` with `main_image_url` + `tile_image_urls` (30-10000) -> poll `GET /v1/mosaics/{id}`
  every 20-30 s (or pass `webhook_url`) until `preview_ready` -> `POST /v1/mosaics/{id}/unlock` -> `hd_url`;
  `layout_url` gives every tile with its position (JSON, or XML with `?format=xml`).
- Test keys (`ck_test_`) give watermarked previews only and cannot unlock.

## Route 3 - nothing connected

Send the user to https://collager.ai/ and explain the steps: create an account, upload the main photo and the small
photos (mark favourites with a star - they get bigger tiles), press Make Collage, the watermarked preview is ready in
minutes, download the full-resolution file on the site. For a heart / number collage the same site has the shape
tools. Developers and AI agents: https://collager.ai/developers/

## Good results - tips to pass on

- **Mosaic main photo:** sharp, well lit, the face(s) large in the frame; avoid tiny faces in a group shot.
- **Tiles:** more and more varied photos = a better mosaic; 300+ is good, duplicates are fine.
- **Styles:** soft classic (default, faces blend evenly), clean subject (the person stays a clean photo, mosaic around),
  stronger mosaic (the tiles form the picture), uniform tiles (a neat grid).
- **Shape collage:** pick the heart size close to the number of photos (27, 34, 70, 79, 91, 164 cells).

## Privacy

Photos are used only to build the collage or mosaic - never published, sold or used to train AI. They are deleted
automatically 30 days after the job is created (`DELETE` removes them at once). Storage: Amazon S3, EU (Frankfurt).
Do not upload photos of people without their consent.
