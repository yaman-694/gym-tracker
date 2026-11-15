# Speak It, Log It: AI That Writes Your Journal

An n8n workflow that turns a Telegram voice note like "bench, 10 at 50" into a logged gym set in an n8n data table and a Notion page.
Built for the Build with AI talk at GDG Gwalior.

## How it works

1. **Telegram** receives a voice or text message.
2. **Data tables** give the AI context: today's sets and any open question the bot asked.
3. **deAPI** transcribes the voice note (speech to text, Whisper Large V3).
4. **Groq** (free tier, `openai/gpt-oss-20b`) turns the text into JSON: exercise, reps, weight.
5. **n8n** saves the set to the data table, rebuilds today's toggle in Notion, and replies on Telegram.

## What is in this repo

| Path | What it is |
| --- | --- |
| `workflow/log-gym-sets-telegram-to-notion.json` | The n8n workflow, ready to import |
| `data-tables/gym_sets.csv` | Schema plus sample rows for the `gym_sets` table |
| `data-tables/gym_pending_question.csv` | Schema only (empty) for the `gym_pending_question` table |

## Accounts and links you need

| Service | Link | What to do there |
| --- | --- | --- |
| PikaPods | [pikapods.com](https://www.pikapods.com/) | Host your own n8n instance (pick the n8n app) |
| Telegram | [@BotFather](https://t.me/BotFather) | Create a bot and copy its token |
| deAPI | [Dashboard](https://app.deapi.ai/dashboard) | Create an API key for speech to text |
| deAPI | [Webhook settings](https://app.deapi.ai/settings/webhooks) | Webhook settings, used when deAPI sends the finished transcript back to n8n |
| Groq | [API keys](https://console.groq.com/keys) | Create a free API key for the LLM |
| Notion | [Integration tokens](https://app.notion.com/developers/tokens) | Create an integration token, then share your Workout Log page with that integration |

## Setup

### 1. Create the data tables

In n8n, open **Overview > Data tables > Create data table > Import CSV**.
Import each CSV and keep the file name as the table name (`gym_sets`, `gym_pending_question`).
`gym_pending_question` starts empty; the workflow adds and clears its row by itself.
You can delete the sample rows in `gym_sets` after importing; they only exist so the number columns are detected.
In `gym_sets`, `setNo`, `reps`, `weight` and `telegramMessageId` must be **number** columns, and all others **string**.
In `gym_pending_question`, all three columns are **string**.

### 2. Import the workflow

Create a new workflow, open the **...** menu, choose **Import from File**, and pick the JSON file.

### 3. Add credentials

| Node | Credential |
| --- | --- |
| On Telegram message, Download voice note, Send reply | Telegram API (bot token from @BotFather) |
| Transcribe an audio | deAPI (install the `n8n-nodes-deapi` community node first) |
| Chat model | OpenAI credential with base URL `https://api.groq.com/openai/v1` and your Groq API key |
| Find day block, Delete old day block, Create day block, Fill day block | Notion API (internal integration, shared with your page) |

### 4. Replace the placeholders

- **On Telegram message**: set the chat ID filter to your own Telegram chat ID (replace `YOUR_TELEGRAM_CHAT_ID`).
  To find it, clear the **Restrict to Chat IDs** field, click **Execute step**, and send your bot any message.
  The output shows it at `message.chat.id`; paste that number back into the field so only you can use the bot.
- **Plan changes** (Code node): set `PAGE_ID` to the ID of your Notion page (replace `YOUR_NOTION_PAGE_ID`).
- **Every Data table node**: pick `gym_sets` or `gym_pending_question` from the list (the values `SELECT_..._TABLE` are placeholders).

### 5. Run it

Publish the workflow and send your bot a voice note such as "bench press, 3 sets of 10 at 40 kg".
You can also say "undo", "that was 8 reps", or "summary".
