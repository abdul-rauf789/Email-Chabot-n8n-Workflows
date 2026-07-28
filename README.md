# AI Email Auto-Reply Chatbot (n8n)

An AI-powered email assistant that automatically replies to incoming Gmail messages using an LLM, built as an n8n workflow.

## Stack
- **n8n** — workflow orchestration
- **Gmail API (OAuth2)** — trigger on new emails, send replies, apply labels
- **OpenRouter** — LLM API for generating email replies

## How it works
1. **Gmail Trigger** polls the inbox every minute for new messages.
2. **Filter (If node)** skips messages that:
   - Have already been replied to (checked via a `bot-replied` label)
   - Come from `noreply`, `no-reply`, or `mailer-daemon` addresses
   - Have subjects containing "out of office", "automatic reply", or "auto-reply"
3. **HTTP Request** sends the email subject + snippet to an LLM (via OpenRouter) with a system prompt instructing it to write a concise, professional reply — no placeholders, no invented details, asks a clarifying question if there isn't enough context.
4. **Reply to a message** sends the AI-generated reply back through Gmail, threaded as a reply.
5. **Add label to message** tags the email with `bot-replied` so it's never replied to twice.

## Setup

1. Import `n8n_email_chatbot(generic).json` into your n8n instance (**Workflows → Import from File**).
2. Recreate these credentials in n8n (names must match, or re-link them after import):
   - `Gmail account` — Gmail OAuth2 credentials (needs Gmail read/send/label scopes)
   - `Header Auth account 2` — Bearer token / API key for your LLM provider (e.g., OpenRouter API key)
3. Create a Gmail label called `bot-replied` in your inbox, and update the `labelIds` value in the **Add label to message** node with your label's actual ID.
4. Customize the system prompt in the **HTTP Request** node — replace the sign-off name/company and tone instructions to match your use case.
5. Activate the workflow.

## Environment / Credentials (NOT included in this repo)
No secrets are stored in this exported JSON file — only credential *names/IDs* are referenced. You must configure each credential fresh in your own n8n instance.

## Customization notes
- Swap the LLM model in the `HTTP Request` node's JSON body (`model` field) to any OpenRouter-supported model.
- Adjust the `If` node's filter conditions to match your own spam/auto-reply exclusion rules.
- The system prompt controls tone, sign-off, and reply style — edit it to fit your brand voice.
