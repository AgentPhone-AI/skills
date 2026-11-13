# AgentPhone MCP Tools

The hosted AgentPhone MCP server exposes 34 tools. Connect it (see [.mcp.json](../.mcp.json), or add `https://mcp.agentphone.ai/mcp` to any MCP client) and sign in through your browser — no API key to paste. Everything here maps onto the [REST API](./rest-api.md), except `watch_messages` and `unsubscribe_events`, which exist only on the hosted MCP server (REST uses a webhook instead); use whichever surface fits your client.

Connect (Claude Code / any Streamable-HTTP client):

```json
{
  "mcpServers": {
    "agentphone": { "type": "http", "url": "https://mcp.agentphone.ai/mcp" }
  }
}
```

On first use it opens a browser to sign in at agentphone.ai (OAuth). No API key needed. (For scripts, the server also accepts `Authorization: Bearer sk_live_...`.)

---

## Account
- **account_overview** — one-call snapshot: agents, numbers, webhook status, usage.
- **get_usage** — usage stats; `breakdown` = `summary` (default) \| `daily` \| `monthly`.

## Phone numbers
- **list_numbers** — list your numbers (`limit`, `offset`).
- **check_capabilities** — which channels a number can receive before you send: iMessage, SMS, RCS, WhatsApp (`phone_number`).
- **check_readiness** — whether a number is ready to send a message or place a call before you try (`number_id`), with the reason if not.
- **buy_number** — purchase a number (`country` default US, `area_code`, `agent_id`). **Paid; requires `confirm: true`** — called without it, nothing is bought and it returns a confirmation prompt echoing the exact args to resend.

## Messages
- **send_message** — send SMS/iMessage, or **react** to a message. Sender: `agent_id` / `number_id` / `from_number`. Send: `to_number`, `body`, `media_url(s)`, `reply_to_message_id`, `send_style` (iMessage effect). React: set `reaction` (a classic tapback — `love`/`like`/`dislike`/`laugh`/`emphasize`/`question` — or a single emoji like `🔥`) + `react_to_message_id`; then `to_number`/`body` aren't needed. Reactions work on iMessage, and on WhatsApp for WhatsApp-bound messages (see the REST reaction endpoint for details).
- **get_messages** — messages for a number (`number_id`, `limit`).
- **watch_messages** — be told when a message arrives on your numbers, instead of calling `get_messages` again and again (`number_ids`: 1 to 50 number IDs, all in one call). This conversation is then told about each new message on a watched number, if this client can be (the tool says if it can't), and you read it with `get_messages`. A watch lasts until 24 hours pass with no new message on that number; calling again extends it; **unsubscribe_events** ends it early. See [Waiting for a reply](../SKILL.md#waiting-for-a-reply).

## Conversations
- **list_conversations** — threads (`agent_id` filter, `limit`, `offset`).
- **get_conversation** — one thread with message history (`conversation_id`, `message_limit`).
- **get_conversation_messages** — page through a conversation's full history (`conversation_id`, cursor pagination) when `get_conversation`'s recent window isn't enough.
- **update_conversation** — set/clear `metadata` on a conversation.

## Contacts
- **list_contacts** — address book (`search`, `limit`, `offset`).
- **manage_contact** — `action` = `create` \| `update` \| `delete` (+ `contact_id`, `phone_number`, `name`, `email`, `notes`).

## Voice calls
Outbound calls placed via MCP always open by identifying the agent as an AI (added automatically; the caller can't disable it via `topic`/`initial_greeting`).
- **list_calls** — recent calls (`agent_id` / `number_id` scope, or `status`/`direction`/`search`).
- **get_call** — details + transcript (`call_id`; `wait` to long-poll until it finishes).
- **end_call** — hang up an in-progress call (`call_id`).
- **make_call** — outbound webhook-driven call (`agent_id`, `to_number`, `initial_greeting`, `from_number_id`, `voice`).
- **make_conversation_call** — outbound autonomous AI call, no webhook (`agent_id`, `to_number`, `topic`, `initial_greeting`, `wait`, `max_wait_seconds`, `from_number_id`, `voice`). Returns the transcript when `wait` (default) is true.

## Agents
- **list_agents** / **get_agent** — list / fetch (with numbers + voice config).
- **create_agent** — create (`name`; `voice_mode`, `system_prompt`, `begin_message`, `voice`, `model_tier`, `transfer_number`, `voicemail_message`, and voice tuning: `voice_speed`, `interruption_sensitivity`, `enable_backchannel`, `stt_mode`, `ambient_sound`, `denoising_mode`, `max_silence_ms`, `language`, `enable_messaging`).
- **update_agent** — partial update (same fields).
- **delete_agent** — delete (numbers kept, unassigned).
- **attach_number** / **detach_number** — assign/remove a number (`agent_id`, `number_id`).
- **list_voices** — available voices (`voice_id` for create/update).

## Webhooks
- **get_webhook** / **set_webhook** / **delete_webhook** — manage the webhook; pass `agent_id` for a per-agent webhook, omit for the project default.
- **test_webhook** — send a synthetic event.
- **list_webhook_deliveries** — recent delivery history.

---

## Notes
- **Auth is per-request** — the MCP server forwards your OAuth token (or API key) to the AgentPhone API; it stores no credentials.
- **REST feature parity:** WhatsApp rich messages (`buttons`/`list`/`cta`/`template`), number lookup, verify, per-call `disableRecording`, call transcript streaming, and conversation capabilities are available on the [REST API](./rest-api.md) and may not all have a dedicated MCP tool yet.
