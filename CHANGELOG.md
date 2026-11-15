# Changelog

## v1.4 (Phase 3): Telegram channel
- Added Telegram version of the agent: Telegram Trigger, same prompt, memory and lead capture, reply via Telegram node
- Session memory keyed to the Telegram chat ID
- Documented that WhatsApp needs a Meta developer account; the same pattern applies
  
## v1.3 (Phase 2): Website widget
- Switched Chat Trigger to Embedded mode with restricted CORS origin
- Added GitHub Pages demo page with embedded n8n chat widget
- Demo is switched on only during recordings to protect free-tier quotas

## v1.2: Lead capture fix
- Refreshed Google Sheets column mapping after a header change
- Tightened the save_lead rules so the question is saved word for word

## v1.1 (Phase 1): Owner alerts
- Added `Dookuture Lead Alert` workflow: emails the owner when a new lead is added
- Kept alert separate from the chatbot so it runs without AI quota
- Added workflow guide and alert screenshots

## v1.0: FAQ agent
- Facts-only system prompt with fixed fallback and capability limits
- Conversation memory (10 messages per session)
- Lead capture to Google Sheets via `save_lead` tool
- 15-question test suite, 15/15 passed
