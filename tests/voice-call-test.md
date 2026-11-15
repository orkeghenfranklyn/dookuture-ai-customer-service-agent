# Voice Call Test Log

One live Vapi web call on 15 November 2026, using the Dookuture Booking Assistant (prompt v1.0) and the `Dookuture Voice Booking` n8n workflow (published for the call only).

## Result summary

| Area | Result |
|---|---|
| Call connects and the assistant greets the caller | Pass |
| Collects name, phone, date, time and outfit | Pass |
| Resolves "next Saturday" to 21 November | Pass |
| Handles an off-slot time (caller said 10:13, assistant offered 10:00 or 10:30) | Pass |
| Reads details back and asks for confirmation | Pass |
| `book_appointment` tool call completes | Pass. Vapi shows "Completed successfully" |
| n8n workflow runs | Pass. Execution #68 succeeded in 3.379 s |
| Booking row written to the `Booking` tab | Pass |
| Phone number validated as 11 digits | **Fail.** An 8-digit number was saved |
| Brand name spoken correctly | **Fail.** A wrong company name appeared in two closing replies |
| Efficient number capture | **Fail.** The number was collected one digit at a time, making the call long |

## Call metrics

| Metric | Value |
|---|---|
| Duration | 3m 33s |
| Turns | 21 |
| Average turn latency | 2,242 ms |
| Total cost | $0.27 (about $0.076 a minute) |

**Cost breakdown**

| Component | Cost | Share |
|---|---|---|
| Vapi platform | $0.18 | 66.5% |
| LLM | $0.04 | 15.3% |
| Text to speech | $0.03 | 12.2% |
| Speech to text | $0.01 | 5.4% |
| Call analysis | $0.00 | under 1% |

**Latency breakdown (average per turn)**

| Stage | Time |
|---|---|
| Transcriber | 485 ms |
| Endpointing | 566 ms |
| LLM | 867 ms |
| Voice | 273 ms |
| Transport (in and out) | 20 ms and 49 ms |

## Issues and fixes

| # | Issue | Fix | Status |
|---|---|---|---|
| 1 | 8-digit phone number accepted | 11-digit check and re-ask rule in [prompt v1.1](../prompts/voice-assistant-prompt.md) | Written, not retested |
| 2 | Wrong company name spoken twice | Explicit brand-name rule, plus a check for leftover template text | Written, not retested |
| 3 | Digit-by-digit phone capture | Ask once and accept the whole number | Written, not retested |
| 4 | Booking endpoint has no authentication | Keep the workflow unpublished except during calls. Add header authentication later | Open |
| 5 | No availability check | Bookings are logged as requests and the team confirms by phone | By design |

## Evidence
- n8n execution list showing execution #68 and the four green nodes: `screenshots/16-voice-n8n-execution.png`, `screenshots/14-voice-booking-workflow.png`
- Booking row in Google Sheets: `screenshots/15-voice-booking-sheet.png`
- Call transcript with the tool call: `screenshots/17-voice-call-transcript.png`
- Vapi call log, cost and latency: `screenshots/18-voice-call-log.png`, `screenshots/19-voice-call-cost.png`, `screenshots/20-voice-call-latency.png`
