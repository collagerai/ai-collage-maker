# Photo Mosaic & AI Collage Maker - Collager.ai

A photo mosaic maker and AI photo collage maker for Claude, ChatGPT and any MCP-capable agent. Turn your own photos into:

- **a photo mosaic** - one picture (a portrait, a couple, a pet) recreated from hundreds or thousands of your small
  photos. Zoom in and every tile is a real photo;
- **a photo collage in seconds** - a heart, a number (birthday, anniversary, year), a word, "I love you", a star or a
  flower made of photos.

Website: **https://collager.ai/** · for developers and AI agents: **https://collager.ai/developers/**

## What is inside

| Part | What it does |
|---|---|
| MCP server `https://collager.ai/mcp` | tools `make_shape_collage`, `get_shape_collage`, `list_collage_shapes`, `list_mosaic_styles`, `start_photo_mosaic`; no sign-in needed |
| Skill `photo-collage-maker` | tells the assistant when and how to use the tools (or the REST API with a key, or the website) |
| REST API `https://api.collager.ai/v1` | for developers with a key - OpenAPI: https://collager.ai/developers/openapi.yaml |

## Install

**Claude Code**

```
/plugin marketplace add collagerai/ai-collage-maker
/plugin install ai-collage-maker@collager-ai
```

or only the MCP server: `claude mcp add --transport http collager-ai https://collager.ai/mcp`

**Any agent that reads Agent Skills** (Codex CLI, Gemini CLI, VS Code, ...): `npx skills add collagerai/ai-collage-maker`

**ChatGPT / Claude.ai**: add the connector `https://collager.ai/mcp` (no authentication).

## Try

- "Make a heart collage from these 30 photos"
- "Make the number 50 out of our family photos for my parents' anniversary"
- "I want a photo mosaic of my dog made of all his photos"

The chat shows a watermarked preview; the full-resolution file is downloaded on collager.ai.

## Privacy

Photos are used only to build your collage or mosaic - never published, sold or used to train AI models. They are
deleted automatically 30 days after the job is created. Files are stored in Amazon S3, EU (Frankfurt). Details:
https://collager.ai/developers/privacy/ · Support: support@collager.ai

## License

MIT for this repository (skill text and configuration). The Collager.ai service has its own terms:
https://collager.ai/terms-of-service.php
