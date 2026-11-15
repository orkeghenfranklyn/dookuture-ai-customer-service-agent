# Changelog

## v1.5 (Phase 4): Voice booking agent
- Added `Dookuture Voice Booking` n8n workflow: Webhook, Code node, Google Sheets append, Respond to Webhook
- Added a Code node to extract booking fields, because n8n blocks expressions that read a field named `arguments`
- Created a Vapi assistant and a `book_appointment` function tool pointing at the n8n production webhook
- Verified one live web call end to end: booking saved to the `Booking` tab, n8n execution succeeded in 3.4 s
- Call cost $0.27 for 3m33s, 21 turns, 2.2 s average turn latency
- Found three prompt issues in the first call (8-digit phone accepted, wrong company name spoken twice, digit-by-digit number capture) and wrote fixes in voice prompt v1.1. Retest pending
- Added voice prompt and voice call test log

<details>
<summary>Code and settings for v1.5</summary>

**1. Webhook node**
- HTTP method: POST
- Respond: Using "Respond to Webhook" Node
- Path: the random default (published only during calls, because the endpoint has no authentication)

**2. Code node (`Extract booking`)**

n8n blocks expressions that read a field named `arguments`, which is where Vapi puts the tool parameters, so this node reads them instead.

```javascript
const body = $input.first().json.body || {};
const message = body.message || {};
const calls = message.toolCalls || message.toolCallList || [];
const call = calls[0] || {};
const fn = call.function || {};
let args = fn.arguments || call.arguments || call.parameters || {};
if (typeof args === 'string') {
  try { args = JSON.parse(args); } catch (e) { args = {}; }
}
return [{
  json: {
    toolCallId: call.id || '',
    name: args.name || '',
    phone: String(args.phone || ''),
    date: args.date || '',
    time: args.time || '',
    outfit: args.outfit || ''
  }
}];
```

**3. Google Sheets node (`Append row in sheet`)**

Operation: Append Row. Tab: `Booking`. Mapping mode: Map Each Column Manually.

| Column | Expression |
|---|---|
| Name | `{{ $json.name }}` |
| Phone | `{{ $json.phone }}` |
| Date | `{{ $json.date }}` |
| Time | `{{ $json.time }}` |
| Outfit | `{{ $json.outfit }}` |

**4. Respond to Webhook node**

Respond With: JSON. Vapi needs HTTP 200, a top-level `results` array and a matching `toolCallId`.

```json
{
  "results": [
    {
      "toolCallId": "{{ $('Extract booking').item.json.toolCallId }}",
      "result": "Booking request saved. Tell the caller the Dookuture team will confirm the appointment by phone."
    }
  ]
}
```

**5. Vapi tool `book_appointment`**

Function tool, async off, Server URL set to the n8n Production webhook URL.

```json
{
  "type": "object",
  "properties": {
    "name": { "type": "string", "description": "Caller's full name" },
    "phone": { "type": "string", "description": "Caller's phone number, digits only" },
    "date": { "type": "string", "description": "Appointment date in YYYY-MM-DD format" },
    "time": { "type": "string", "description": "Appointment time in 24-hour HH:MM format, between 09:00 and 17:30" },
    "outfit": { "type": "string", "description": "The outfit the caller wants measured for, for example senator set" }
  },
  "required": ["name", "phone", "date", "time", "outfit"]
}
```

The assistant prompt is in [`prompts/voice-assistant-prompt.md`](prompts/voice-assistant-prompt.md).

</details>

## v1.4 (Phase 3): Telegram channel
- Added Telegram version of the agent: Telegram Trigger, same prompt, memory and lead capture, reply via the Telegram node
- Memory keyed to the Telegram chat ID
- Documented that WhatsApp needs a Meta developer account and that the same pattern applies

## v1.3 (Phase 2): Website widget
- Switched the Chat Trigger to Embedded mode with a restricted CORS origin
- Added a GitHub Pages demo page with the embedded n8n chat widget
- Demo is switched on only during recordings to protect free-tier quotas

## v1.2: Lead capture fix
- Refreshed the Google Sheets column mapping after a header change
- Tightened the `save_lead` rules so the question is saved word for word

## v1.1 (Phase 1): Owner alerts
- Added the `Dookuture Lead Alert` workflow: emails the owner when a new lead is added
- Kept the alert separate from the chatbot so it runs without AI quota
- Added workflow guide and alert screenshots

## v1.0: FAQ agent
- Facts-only system prompt with a fixed fallback and stated capability limits
- Conversation memory (10 messages per session)
- Lead capture to Google Sheets through the `save_lead` tool
- 15-question test suite, 15 of 15 passed
