# Workflows

| File | Purpose | Trigger | Uses AI? |
|---|---|---|---|
| `dookuture-faq-bot.json` | Web chat agent: answers from approved facts, remembers context, captures leads | Chat message (embedded widget) | Yes (Gemini) |
| `dookuture-lead-alert.json` | Emails the owner whenever a new lead is added | New row in the leads sheet | No |
| `dookuture-telegram-bot.json` | The same agent inside Telegram | Telegram message | Yes (Gemini) |
| `dookuture-voice-booking.json` | Receives booking tool calls from the Vapi voice assistant and saves them | Webhook (POST from Vapi) | No |

## Why separate workflows?
- **Separation of concerns:** answering customers, notifying staff and taking bookings are different jobs.
- **Resilience:** the alert and the booking path keep working even if the AI model hits a quota limit.
- **Speed:** the voice booking path has no AI step, so it answers Vapi in about 3 seconds.
- **Reuse:** every channel writes to the same sheet, and one alert covers them all.

## Before you import
1. Create an n8n account (Cloud trial or self-hosted).
2. Create a Google Sheet with a tab named `Sheet1` (headers `Name`, `Phone`, `Question`) and a tab named `Booking` (headers `Name`, `Phone`, `Date`, `Time`, `Outfit`). Keep headers free of trailing spaces.
3. Have your own credentials ready: Google Gemini API key, Google Sheets, Gmail, and a Telegram bot token from BotFather.

## Import
In n8n: workflow menu, then **Import from file**. Pick one JSON file at a time, then:
1. Select your own credentials on each node that shows a warning.
2. Re-pick the Google document and sheet in the Sheets nodes, and refresh the column list.
3. Replace `owner@example.com` in the Gmail node with your own address.
4. Paste the matching system prompt from [`prompts/`](../prompts/) into the AI Agent node.

**No keys are stored in these files.** Placeholders you will see: `YOUR_SHEET_ID`, `YOUR-WEBHOOK-PATH`, `owner@example.com`.

## Voice booking: how it connects to Vapi

**1. Vapi tool.** Create a Function tool named `book_appointment` with five required string parameters: `name`, `phone`, `date` (YYYY-MM-DD), `time` (24-hour HH:MM) and `outfit`. Set its Server URL to the n8n **Production** webhook URL (it contains `/webhook/`, not `/webhook-test/`). Leave async off.

**2. Extract the fields.** n8n's expression engine refuses to read a field named `arguments`, which is where Vapi puts the parameters. A Code node reads them instead:

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

**3. Reply in the exact shape Vapi expects.** HTTP 200, a top-level `results` array, the same `toolCallId`, and a single-line string result:

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

**4. Publish only during calls.** A new Vapi server URL has no authentication by default, so keep the workflow unpublished except while testing.

## Security checklist before uploading an export
- Search the file for `AIza`, `apiKey`, your real email and any real phone numbers
- Replace the webhook path and Google document IDs with the placeholders above
- Never commit a bot token, API key or call recording link
