# 90-Day Social Media Calendar Generator

Generate a full 90-day social media calendar for any client: post copy, one
linking URL per post (from the client's live sitemap), and one AI-generated
image per post. Output is a CSV in HighLevel Social Planner import format.

## Requirements

- **HighLevel CRM** - the finished CSV is designed to be imported into a
  client's HighLevel Social Planner. You need a HighLevel account (yours or
  the client's) to actually use the output.
- **An AI model API key** for creating post content and image prompts
  (DeepSeek, OpenAI, ChatGPT, or any OpenAI-compatible provider)
- **An image API key** (Gemini free tier, FAL, or Kling) if you want images
  generated
- **FTP or SSH access to a web host** if you want the generated images to be
  publicly accessible. The image step saves images locally; to put real
  image URLs in the CSV (the `imageUrls` column), you host them yourself and
  pass `--image-base-url`. The package does not upload for you.
- Optional: a HighLevel API key, ONLY if you want Mode A (auto-fetch client
  details from your HighLevel subaccount)

## Quick start

```bash
cp .env.example .env   # optional: add your HighLevel key here (Mode A only)
./run.sh --details "Company Name|website.com|City|ST|Business Category|Company description" \
  --llm-key sk-...
```

Using a ChatGPT/OpenAI key instead of DeepSeek:

```bash
./run.sh --details "Company Name|website.com|City|ST|Business Category|Company description" \
  --llm-key sk-your-openai-key \
  --llm-model gpt-4o-mini \
  --llm-base https://api.openai.com/v1/chat/completions
```

Example:

```bash
./run.sh --details "Example Dental Group|exampledental.com|Orlando|FL|dentist|Family dentist offering cleanings and same-day crowns."
```

Output: `output/exampledental.com-social-90d.csv` ready to import into that
client's HighLevel Social Planner (you import it in HighLevel, under the
client's Social Planner).

Note: the CSV is a HighLevel Social Planner import file. Without HighLevel
you can still generate it, but you need HighLevel to schedule/publish the
posts from it.

## Two ways to provide client details

### Mode A: Auto-fetch from HighLevel (needs API key)

```bash
# set HIGHLEVEL_ACCESS_TOKEN in .env or your environment
./run.sh --email client@example.com
```

Pulls the contact + custom fields (Business Category, Company Description,
website, city, state) from your HighLevel subaccount. This is the ONLY mode
that uses an env var, and it is the only env var the package requires.

### Mode B: Provide details directly (no HighLevel needed)

```bash
./run.sh --details "Company|website|City|ST|Business Category|Description"
# or from a JSON file:
./run.sh --client-json examples/client-example.json
```

Handy when you already have the client info in front of you (for example, you
pulled it from HighLevel yourself) and you do not want to connect the tool to
HighLevel. Same CSV output either way.

## Options

| Flag | Default | Purpose |
|---|---|---|
| `--email` | | Mode A: fetch client from HighLevel by email |
| `--details` | | Mode B: client details as `Company\|site\|City\|ST\|Category\|Desc` |
| `--client-json` | | Mode B: normalized client JSON file |
| `--count` | 45 | Number of posts (90 days at 1 every 2 days) |
| `--start-date` | today | First post date (YYYY-MM-DD) |
| `--no-images` | | Skip the image step |
| `--llm-key` | | LLM API key (required for post generation; flag only, never an env var). Works with DeepSeek, OpenAI, ChatGPT, or any OpenAI-compatible provider |
| `--llm-model` | | Model name (default `deepseek-chat`; use e.g. `gpt-4o-mini` for OpenAI/ChatGPT) |
| `--llm-base` | | API endpoint (default DeepSeek; use `https://api.openai.com/v1/chat/completions` for OpenAI/ChatGPT) |
| `--image-base-url` | | Public URL prefix so the CSV imageUrls column is filled |
| `--gemini-key` / `--fal-key` / `--kling-ak` + `--kling-sk` | | Image provider keys (flags, never env vars) |

## Environment variables

The ONLY env var this package requires is `HIGHLEVEL_ACCESS_TOKEN`, and only
for Mode A. Everything else (LLM key, image keys) is passed as a CLI flag.
See `.env.example`.

## Pipeline

1. **Client details** (`fetch_client.py`) - normalized client JSON (either mode)
2. **Sitemap** (`fetch_sitemap.py`) - pulls and filters the client sitemap
3. **Posts** (`generate_posts.py`) - LLM writes the posts, one URL each,
   balanced entertaining/informative/promotional, no emojis, hashtags, and a
   09:00 schedule every other day
4. **Images** (`generate_images.py`, optional) - one local image per post,
   saved under `output/<domain>-images/`; fill `imageUrls` in the CSV only if
   you pass `--image-base-url`
5. **Import** - upload the CSV to the client's HighLevel Social Planner

## Output

- `output/<domain>-social-90d.csv` - HighLevel Social Planner import file
- `output/<domain>-social-90d-posts.json` - raw posts (image prompts + URLs)
- `output/<domain>-images/` - generated images (local, when enabled)

## Custom field names

Mode A maps these HighLevel custom field names (case-insensitive):
`Business Category` and `Company Description`. If your subaccount uses
different field names, edit `fetch_client.py` where it reads
`customFields`, or just use Mode B.
