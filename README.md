# Vasool

An AI collections agent for Indian MSME merchants who sell to other businesses on 30 to 60 day credit.
Built for the Razorpay AI x PM Build Challenge, Track 2: Recover and Grow with AI.

Vasool spots overdue invoices, decides channel, timing, tone and offer per payer, drafts a reminder from invoice data only, waits for the merchant's approval, reads the payer's reply (paid, split, reschedule, dispute, resend invoice, opt-out, unclear), drafts the right response, and collects through a Razorpay payment link.

**Prototype:** https://vasool-dues-agent-2027.ai.studio/ (built in Google AI Studio; it may lag the latest edits)

## What is in this repository

| Path | What it is |
|---|---|
| `docs/Vasool_one_page_product_note.pdf` | One-page product note: problem, evidence, what was built and left out, decisions, metric |
| `docs/AI_BUILD_LOG.md` | AI tools used, key prompts, one corrected AI output, AI versus human work, limitations |
| `docs/ARCHITECTURE.md` | How the prototype works, guardrails and data flow |
| `prompts/` | The prompts used to build and fix the app |
| `data/` | Synthetic dataset: 10 payers, 34 invoices, 15 labelled test replies (JSON, CSV, XLSX) |
| `demo/` | Self-contained HTML demos: dark WhatsApp simulation and a 24-second animated explainer |

## Honest limits

All data is synthetic. There is no payment webhook, customer replies are pasted into the app rather than captured from WhatsApp, only invoice INV-1016 has a real (Razorpay test-mode) payment link, and the "Collected" figures are illustrative. Nothing is actually sent. See `docs/AI_BUILD_LOG.md` for the full list.

## Source code note

The prototype's React and `server.ts` source lives in Google AI Studio and was generated from the prompts in `prompts/`. It is not copied here. No API keys or secrets are in this repository; the Gemini key is held server-side in AI Studio's Secrets.

## Disclaimer

Razorpay is a trademark of its owner. This is an independent challenge entry and is not an official Razorpay product.
