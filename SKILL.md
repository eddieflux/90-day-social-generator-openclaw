---
name: "90-day-social-generator"
description: "Generate a 90-day social media calendar (HL Social Planner CSV): AI posts + images per client, with or without HighLevel connection. Requires HighLevel to import, AI model for content, image API + your own FTP/SSH host for image URLs."
---

# 90-Day Social Media Calendar Generator

Generate a full 90-day social media calendar for any client: post copy, one
linking URL per post (from the client's live sitemap), and one AI-generated
image per post. Output is a CSV in HighLevel Social Planner import format.

## Requirements

- **HighLevel CRM** - the finished CSV is imported into a client's HighLevel
  Social Planner. You need a HighLevel account to use the output.
- **An AI model API key** for post content and image prompts (DeepSeek,
  OpenAI, ChatGPT, or any OpenAI-compatible provider)
- **An image API key** (Gemini free tier, FAL, or Kling) for images
- **FTP or SSH access to a web host** if you want generated images publicly
  accessible (the package saves images locally; you host them and pass
  `--image-base-url` for real CSV image URLs)
- Optional: a HighLevel API key, ONLY for Mode A (auto-fetch client details)

## Quick start

```bash
./run.sh --details "Company Name|website.com|City|ST|Business Category|Company description" \
  --llm-key sk-...
```

Output: `output/<domain>-social-90d.csv` (HighLevel Social Planner import).

Using a ChatGPT/OpenAI key instead of DeepSeek:

```bash
./run.sh --details "Company Name|website.com|City|ST|Business Category|Company description" \
  --llm-key sk-you…-key \
  --llm-model gpt-4o-mini \
  --llm-base https://api.openai.com/v1/chat/completions
```

## Two ways to provide client details

- **Mode A (HL-connected):** `./run.sh --email client@example.com` - pulls the
  contact + custom fields from your HighLevel subaccount. The ONLY env var in
  the package, `HIGHLEVEL_ACCESS_TOKEN`, is used here.
- **Mode B (no HighLevel):** `./run.sh --details "..."` or
  `./run.sh --client-json file.json` - hand over client details directly and
  still get the importable CSV. No env vars needed.

## Pipeline

1. Client details (either mode) -> normalized client JSON
2. Sitemap -> filtered page list from the client's site
3. Posts -> LLM writes the calendar (45 posts, every other day, 9am, one URL
   per post, no emojis, hashtags)
4. Images (optional) -> one local image per post; pass `--image-base-url` to
   fill the CSV `imageUrls` column with your hosted URLs
5. Import the CSV into the client's HighLevel Social Planner

See README.md for the full flag reference.
