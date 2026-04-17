---
name: ai-news-digest
description: Daily AI news digest skill. Searches the web for AI news from the past 24 hours, verifies publication dates, categorizes findings, formats a digest message, and sends it to Telegram. Use this skill whenever the user asks for today's AI news, a daily AI digest, AI news summary, or wants to send a news update to Telegram. Trigger proactively when the user says things like "what happened in AI today", "send the AI digest", "run the news agent", or "check AI news".
---

# AI News Digest

You are a daily AI news agent. Follow these steps in order.

## Step 1: Search for today's AI news

Calculate the 24-hour window:
- CUTOFF = today's date minus 24 hours (YYYY-MM-DD)
- TODAY = today's date (YYYY-MM-DD)

Run ALL of these web searches, appending `after:[CUTOFF]` to every query:

```
Anthropic news today [DATE] after:[CUTOFF]
OpenAI news today [DATE] after:[CUTOFF]
Google DeepMind news today [DATE] after:[CUTOFF]
AI news today site:theverge.com OR site:arstechnica.com OR site:techcrunch.com after:[CUTOFF]
AI tools released today [DATE] after:[CUTOFF]
viral AI tools that work with Claude MCP today [DATE] after:[CUTOFF]
"Claude" new tool OR integration OR workflow [DATE] after:[CUTOFF]
site:youtube.com "Nate Herk" latest after:[CUTOFF]
site:youtube.com "Fireship" OR "Two Minute Papers" latest after:[CUTOFF]
site:reddit.com/r/ClaudeAI OR site:reddit.com/r/ClaudeCode top posts today after:[CUTOFF]
site:reddit.com/r/LocalLLaMA OR site:reddit.com/r/MachineLearning top posts today after:[CUTOFF]
site:x.com AnthropicAI OR OpenAI announcement today after:[CUTOFF]
site:x.com "Sam Altman" OR "Demis Hassabis" AI today after:[CUTOFF]
site:x.com "Nate Herk" latest after:[CUTOFF]
```

## Step 2: Date Verification Gate

For every candidate result from Step 1, BEFORE proceeding:

a. Check the article/post's publication date (byline, URL slug, `og:article:published_time` meta tag, or page content).
b. If the publication date is BEFORE CUTOFF, **discard entirely**. Do not include it in any category. Do not note it as "still in the news cycle."
c. If the publication date **cannot be confirmed** as within the past 24 hours, **discard** (when in doubt, leave it out).
d. Only items that pass this gate proceed to Step 3.

## Step 3: Resolve primary source URLs

For every item that passed the Date Verification Gate:

| Source type | Required URL format | How to find it |
|-------------|--------------------|--------------------|
| Reddit | `reddit.com/r/.../comments/...` | Search `site:reddit.com [post title]` |
| X/Twitter | `x.com/[handle]/status/[id]` | Use search results with direct x.com links |
| YouTube | `youtube.com/watch?v=...` | Search `site:youtube.com [channel] [title]` |
| All others | Original publisher URL (e.g. anthropic.com, techcrunch.com) | Use directly — no aggregators or recaps |

Skip any item where the correct URL cannot be found.

## Step 4: Categorize results

Group findings into these categories. **Skip any category with zero verified items** — do not pad with older stories:

- 🤖 New Models & Releases
- 🛠 AI Tools & Products
- 🔧 Claude Ecosystem — MCP servers, Claude integrations, viral workflows
- 🔬 Research & Papers
- 🏢 Industry & Business
- 💬 Community Buzz (Reddit)
- 🐦 From the Feed (X/Twitter)
- 📺 YouTube Picks

## Step 5: Format the Telegram message

Write the message in this exact format. Each bullet ends with the source URL in parentheses (plain URL — Telegram auto-renders these). Keep each bullet to one line. **Total message under 4000 characters** — trim lower-priority sections if needed (YouTube first, then X, then Reddit):

```
⚡ AI News Digest — [TODAY'S DATE]

🤖 New Models & Releases

[Source] Title — one line summary (https://original-publisher-url.com/article)

🛠 AI Tools & Products

[Source] Title — one line summary (https://original-publisher-url.com/article)

🔧 Claude Ecosystem

[Source] Title — one line summary (https://original-publisher-url.com/article)

🔬 Research & Papers

[Source] Title — one line summary (https://original-publisher-url.com/article)

🏢 Industry & Business

[Source] Title — one line summary (https://original-publisher-url.com/article)

💬 Community Buzz

[r/subreddit] Post title — one line summary (https://reddit.com/r/subreddit/comments/...)

🐦 From the Feed

[@handle] One line summary of what they posted (https://x.com/handle/status/...)

📺 YouTube Picks

[Channel] Video title — one line summary (https://youtube.com/watch?v=...)

Sources: Anthropic · OpenAI · DeepMind · The Verge · TechCrunch · Ars Technica · Reddit · X · YouTube
```

## Step 6: Send to Telegram

Read `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` from the environment, then run:

```bash
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  --data-urlencode "text=YOUR_MESSAGE" \
  -d "chat_id=${TELEGRAM_CHAT_ID}" \
  > /tmp/tg_result.json

cat /tmp/tg_result.json
```

- If response contains `"ok":true` → done.
- If response contains `"ok":false` → trim the lowest-priority category and retry once.
