# Vasool architecture notes

## Components

- **Web app (React, built in Google AI Studio):** six screens: Overview, Follow-ups, Replies, Collected, Activity, Settings, plus a System status tab.
- **Server (server.ts):** holds the Gemini key as a server-side secret and makes the model calls. The browser never sees the key.
- **Data:** a synthetic ledger (payers, invoices, test replies) loaded into the app, with a "Reset demo data" button. Dates are offset from today's date so the demo stays current.
- **Model (Gemini Flash family):** two jobs only: reading a payer's reply into JSON (intent, confidence, amount, date) and drafting reminder messages.
- **Payments:** one Razorpay test-mode payment link for INV-1016 (balance Rs 98,000, partial payments on). All other invoices show placeholder text.

## Flow

1. Detect: classify each open invoice (open, due soon, overdue, part-paid, disputed) from due date and payer history.
2. Plan: choose channel, timing, tone and offer per payer (preferred channel, language, best contact time, past disputes).
3. Draft: write the reminder from invoice fields; amounts come from invoice data only.
4. Approve: the merchant sends, edits, holds or skips. Nothing is sent without approval.
5. Read reply: the model returns intent, confidence, amount and date as JSON.
6. Respond: paid (check the bank statement first), split or reschedule (new link for the right amount), dispute (pause reminders, notify the owner), resend invoice, opt-out (do not contact), unclear (ask the owner).
7. Collect: Razorpay payment link.
8. Learn: log outcomes per payer.
9. Dashboard: overdue, due soon, outstanding, collected within 14 days (synthetic).

## Guardrails

- Amounts, dates and links come from invoice data, never from the model. A number is treated as an amount only with a currency marker.
- A dispute pauses reminders and goes to the owner.
- An opt-out marks the payer do-not-contact.
- Accounts flagged "never auto-contact" are skipped.
- No pressure tactics, fake deadlines or threats.
- If the model is unavailable (503 or 429), the app retries, then falls back to a clearly labelled "Example result, live call unavailable" with confidence hidden.

## Health check

GET /api/health makes one tiny live model call, measures latency, retries on 503 or 429, and returns healthy, degraded or down. The System status tab shows this live result so it cannot show green while the model fails.

## Not built

Payment webhook, WhatsApp or SMS capture of replies, real sending, voice calls, reconciliation of NEFT or cheque receipts, per-invoice link generation through Razorpay's API.
