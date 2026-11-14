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
- **Plan changes** (Code node): set `PAGE_ID` to the ID of your Notion page (replace `YOUR_NOTION_PAGE_ID`).
- **Every Data table node**: pick `gym_sets` or `gym_pending_question` from the list (the values `SELECT_..._TABLE` are placeholders).

### 5. Run it

Publish the workflow and send your bot a voice note such as "bench press, 3 sets of 10 at 40 kg".
You can also say "undo", "that was 8 reps", or "summary".
