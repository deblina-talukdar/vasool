# Vasool: AI build log

**Project:** Vasool, an AI collections agent for Indian MSME merchants who sell to other businesses on 30 to 60 day credit.
**Challenge:** Razorpay AI x PM Build Challenge, Track 2 "Recover and Grow with AI" (AI Product Manager role).
**Author:** individual submission.

This log covers what AI did and what I did, what went wrong, and what is still unverified. It contains no API keys, credentials or private data. Everything in the prototype's dataset is synthetic.

---

## 1. AI tools and models used

| Tool | What I used it for |
|---|---|
| **Claude** (Anthropic, used through the Claude app in Cowork mode; Sonnet-class models) | Idea shortlisting against Razorpay's shipped agents, problem statement, build plan, synthetic data generation (Python script), prompt writing for the app builder, browser-driven testing of the prototype, the one-page product note (PDF), demo animations, and this log. |
| **Claude in Chrome** (browser extension) | Operating my own browser tabs to test the prototype screen by screen, read Razorpay's site for colours and pricing, and create the Razorpay test payment link after I approved it. |
| **Google AI Studio, Build mode** (Gemini-based app builder) | Generated and edited the prototype itself (a React app plus a small `server.ts`). Each edit took roughly 7 to 11 minutes. |
| **Gemini Flash family** (inside the app, called server-side) | Two jobs only: reading a customer's free-text reply into JSON (intent, confidence, amount, date), and drafting reminder messages. The exact model id is read from one shared constant in the app and shown live on the app's "System status" tab, so it is not hard-coded here. |
| **Lovable** | First attempt at the prototype. Abandoned after the free credits ran out (see section 5). |
| **Razorpay Dashboard in Test Mode** | One real test-mode payment link, used for the INV-1016 balance (₹98,000, partial payments on). |
| **Google Drive / Sheets** | Held the synthetic data (payers, invoices, test replies). |
| **Python (reportlab, pandas), ffmpeg, headless Chromium (Playwright)** | The product-note PDF, data analysis, and checking the animated demos. |
| **Powtoon, Animaker, Canva** | Explored for the 90-second demo video. I drafted prompts for Animaker and Canva; the final demo assets were self-contained HTML animations. |
| **Web Speech API with Microsoft Neerja (en-IN) voice in Edge** | Indian female voiceover for the Vasool explainer and WhatsApp simulation. |
| **Web search** | Market figures for the product note (Udyam registrations, delayed payments, Recordent receivables report, Razorpay's listed pricing). |

## 2. Key prompts, workflows and agent setup

### The workflow, in order

1. **Idea selection.** I gave Claude the Track 2 brief and asked for agentic flows that (a) are not already shipped in Razorpay Agent Studio, (b) do not eat into Razorpay's revenue, and (c) give merchants a measurable growth or recovery opportunity. I told it MDR was not required. Claude mapped the existing agents (Cart Recovery, COD Confirmation, EMI Recovery, Subscription Recovery, Payment Retry and others) so overlap could be checked and suggested alternatives to compare. The dues agent for B2B collections was my idea, and I named it Vasool.
2. **Problem statement.** 5 to 6 sentences, then a hard cap of 125 words.
3. **Synthetic data.** A script generated 10 payers, 34 invoices and 15 labelled test replies (10 English, 5 Hinglish), with deliberate edge cases: a part-paid invoice, a disputed invoice, a payer who must never be auto-contacted, a habitual late payer, a payer who only answers calls.
4. **Prompt-driven build.** One master prompt to AI Studio Build mode, then a test, fix-prompt, retest loop (section 3).
5. **Razorpay test link** for INV-1016 only, added to the app as a "Pay balance" button.
6. **Theme and status.** A Razorpay-inspired colour theme (palette read from razorpay.com), a Razorpay logo in the header, a dynamic "today" date, a real health-check "System status" tab, and later a dark theme.
7. **Demo assets.** A dark WhatsApp simulation (Vasool and a payer, split payment, the Razorpay payment page), and a 24-second animated explainer with an Indian female voice.
8. **Submission documents.** One-page product note (PDF) with sourced market figures, and this log.

### Agent setup (what the product does)

Vasool runs this loop on every open invoice:

1. Detect overdue or at-risk invoices.
2. Decide channel, timing, tone and offer per payer.
3. Draft the reminder from invoice data only.
4. Wait for the merchant's approval.
5. Read the customer's reply (paid, split, reschedule, dispute, resend invoice, opt-out, unclear).
6. Draft the right response, for example "₹49,000 now and ₹49,000 on 21 Oct".
7. Collect through a Razorpay payment link.
8. Learn from outcomes.
9. Show a dashboard.

Hard rules: a dispute pauses reminders and goes to the owner; an opt-out is honoured; key accounts flagged "never auto-contact" are skipped; no pressure tactics or fake deadlines; amounts come only from invoice data, never from the model; the merchant approves before anything is sent.

### Prompts that mattered (shortened)

**Master prompt to AI Studio (design intent):** plain accounting-software look (system font, 14px body, one blue accent, no gradients, no emoji), Indian number and date formats (₹6,82,700, "9 Oct 2026"), six screens (Overview, Follow-ups, Replies, Collected, Activity, Settings), and Gemini limited to two jobs: reply-to-JSON and message drafting, with a clear fallback if a call fails. I asked for this look specifically because the first version "screamed AI".

**Amount-extraction fix:**
> Only treat a number as an amount if it has a currency marker (₹, Rs, rupees, INR) or the customer is clearly quoting a payment figure. Quantities like "2 boxes", "5 days" or "10 pieces" are NOT amounts. If no currency amount is present, set amount to null and show "None mentioned".

**Model overload handling:**
> If the Flash-Lite call fails with 503 or 429, retry once on the main Flash model before using the fallback.

**Honest labelling:**
> On the Collected screen add: "Illustrative figures from synthetic sample data. Not real merchant results." Under the Replies box add: "Prototype: replies are pasted here. In production they would arrive from WhatsApp or SMS."

**Real health check, not a static badge:** a `/api/health` endpoint that makes one tiny live Gemini call, measures latency, retries on 503/429, and returns healthy, degraded or down, so the status tab cannot show green while the model is failing.

## 3. One AI output I corrected: the "₹2" amount

**What the AI produced.** The first working version of the Replies screen read the customer message "Goods short by 2 boxes. Will not pay until credit note." correctly as a dispute (98% confidence) but displayed **"Amount mentioned: ₹2"**. The model had treated the quantity "2 boxes" as money.

**Why it mattered.** Vasool's central promise is that amounts come only from invoice data. A reply reader that invents a rupee figure from a quantity breaks that promise in the one place a merchant is most sensitive: a dispute. A merchant who sees "₹2" would stop trusting every other number on screen.

**How it was caught.** Not by reading the code. At my request, Claude ran each sample reply (dispute, opt-out, already-paid, unclear) through the live app in my browser, one at a time, and read the result card. It reported this bug and a second one from the same pass (the Replies screen opened on a paid ₹0 invoice). I then pasted the fix prompt into AI Studio myself.

**The fix.** The prompt in section 2, plus a rule to filter unpaid invoices in the dropdown and default to the most overdue one. On retest the dispute reply showed "None mentioned" for the amount and 100% confidence on the dispute.

**A second, smaller correction.** After the first fix round, the Activity log said "Reminder queued for INV-1016 (₹1,88,000)" when only ₹98,000 was outstanding. The app was showing the full invoice value for a part-paid invoice. A further fix prompt required one shared balance calculation everywhere; the retest showed ₹98,000.

## 4. What AI did versus what I decided or did myself

**AI did:**
- Mapped Razorpay's existing agents so the idea could be checked for overlap, and suggested alternatives to compare it with.
- Wrote the problem statement and the free-tool build plan.
- Generated the synthetic dataset (script, JSON, CSV, Excel).
- Wrote the master prompt and every fix prompt for the app builder.
- Generated the prototype code (Gemini in AI Studio).
- Ran browser tests, read screens, and reported bugs.
- Wrote the product note, the animated explainer, the WhatsApp simulation, and the drafts of this log.

**I decided or did myself:**
- **The idea and its constraints.** The idea of building Vasool as a dues agent for B2B collections was mine. I also set the rules for the search (not already shipped, not eating Razorpay's revenue, measurable merchant benefit, MDR not required) and named it Vasool.
- **Product scope and guardrails.** Human approval before any send, dispute pause, opt-out, key-account protection, amounts only from invoice data. I also decided what to leave out (auto-send, webhooks, voice calls, lending, any MDR feature).
- **The account steps and the money-adjacent steps.** I handled the sign-ups and logins myself (including Razorpay), supplied the Gemini key as a server-side secret, and approved the creation of the test payment link before it was made. Claude was set up not to enter passwords or financial details for me.
- **Tool switches.** I moved from Lovable to Google AI Studio when credits ran out and asked for the interface to look less "AI".
- **Look and feel.** I asked for the Razorpay colour theme and logo, a live health tab, a dynamic date, and a dark theme. (Claude had advised against using the Razorpay logo because it could look like an official Razorpay screen; I chose to include it as a small mark in the header, so the decision and its risk are mine.)
- **Voice and pacing of the demo.** I chose the Indian female voice (Microsoft Neerja), the dark WhatsApp style, and rejected the first animated versions as rushed, so the explainer grew from 20 to 24 seconds.
- **What gets published and submitted.** Nothing was published, shared or sent without my explicit yes.

## 5. Other things that went wrong, and what I did

| Problem | What happened | Resolution |
|---|---|---|
| Lovable key warning and credit limit | The Gemini call returned a 404 because the model name was not available to a new key; then free credits ran out and an upgrade was offered | Declined the paid upgrade and rebuilt in Google AI Studio |
| Gemini 503 "high demand" | The live call sometimes failed during testing | Retries (up to 3, 2-second pause), a lighter Flash-Lite model first with the main Flash model as backup, and a clearly labelled "Example result, live call unavailable" fallback with the confidence hidden |
| Publish button greyed out | The auto-generated App URL failed a reserved-terms check | Renamed the URL; Publish was not clicked without my permission |
| Same voice for both speakers in the WhatsApp demo | Voice list was limited to English and relied on pitch, which Natural voices ignore | Included Hindi voices, auto-picked different ones, added a warning when both match |
| Voiceover cut off mid-line | Speech was started without waiting for the previous line to end | Made speech promise-based so each line finishes before the scene changes |
| Indian male voice for an existing MP4 | The cloud workspace has no usable voice service | Told that this could not be produced there; left as a manual step in a voice tool |
| Public sources for market figures | Search results gave headline numbers but not always the primary document | See limitations below; only numbers I could open are in the note |

## 6. Known limitations, risks and verification steps

### Limitations of the prototype

- **All data is synthetic.** 10 payers, 34 invoices, 15 test replies. It was modelled on a realistic Hyderabad auto-trade merchant but has not been validated with a single real merchant.
- **No payment webhook.** A payment made through the link is not detected by the app. The Collected figures are synthetic and labelled as illustrative.
- **Replies are pasted in.** In production they would arrive from WhatsApp or SMS; that integration is not built.
- **Only INV-1016 has a real payment link** (Razorpay Test Mode). Every other invoice shows placeholder text.
- **Nothing is actually sent.** The Send action is labelled "prototype: nothing is actually sent".
- **Reply-reading accuracy is not measured.** I tested a handful of replies by hand (dispute, opt-out, already paid, unclear, and the amount edge case), not the full 15-reply set, and I have not run any statistical evaluation. Split, reschedule and resend-invoice handling in particular deserve a proper test.
- **Model availability.** The app depends on Gemini's free tier and on shared quota if the published link is used by many people. The fallback keeps the app usable, but fallback output is not a model result.
- **Demo audio.** The voiceover relies on the viewer's browser voices. I could not verify the real Microsoft Neerja voice in the cloud workspace, only that the code selects it when present.
- **Published link may lag the editor.** The theme, logo, dynamic date and dark-theme edits were made in the AI Studio editor; the live published URL only shows them after I republish. Check this before submitting.

### Risks

- **Untrusted input.** A payer's reply is untrusted text that goes into a model. The design reduces the risk (amounts come from invoice data, a human approves every message, the model returns structured JSON only), but a determined prompt-injection test has not been run.
- **Tone and compliance.** Collections messages can harm relationships or breach consent rules. The guardrails (no pressure, honour opt-outs, dispute pause) are enforced by prompts and logic I tested by hand, not by a legal review.
- **Revenue estimates are arithmetic, not forecasts.** The product note estimates Razorpay's fee upside from collections moved onto payment links, using 1% to 2% as a planning range. Razorpay lists 2% + GST (Payment Links carry no separate fee) and says the 2% platform fee also applies to standard UPI despite zero MDR. From 15 Oct 2026 select UPI merchant payments above ₹2,000 carry a 0.4% MDR (capped at ₹300 from ₹75,000), paid by the merchant to its acquiring bank. Real yield depends on payment mix and any custom pricing, which I have not confirmed, so 1% is not a proven conservative case. These fees also cost the merchant more than a free bank transfer, which could slow adoption.
- **Public app.** Anyone with the link can use the published app; the Gemini key stays on the server and is not in the browser.

### Verification steps I took, and ones still to do

Done:
- Had each sample reply run through the live app and every result card read (Claude, in my browser).
- Re-tested after every fix prompt I pasted, including after "Reset demo data".
- Checked that the Follow-ups badge and list header match and that the INV-1016 balance shows ₹98,000 in the Activity log and on its card.
- Opened the market source pages (not just search snippets) and recorded where each number came from.

Still to do:
- Confirm by eye on the final app: the 4-second Send confirmation, a live "None mentioned" date (instead of "null"), the System status tab's live model call, and every screen in the dark theme.
- Pilot with 5 to 10 real MSME merchants against their manual chasing.
- Measure reply-intent accuracy on real Hinglish and Telugu messages, treating a missed dispute as the critical error.
- Add a payment webhook so collected amounts come from Razorpay, not synthetic data.
- Run a prompt-injection and consent/opt-out review before any real sending.
- Republish the app and re-verify every screen on the live link.

### Sources for the market figures in the product note

- 8.84 crore Udyam and Udyam Assist registrations as of 30 Jun 2026: KNN India, citing the Minister of State for MSME.
- ₹8.1 lakh crore of delayed payments: reported by Ascendants (30 Jan 2026) and MSME Sampark, both attributing it to the Economic Survey. I could not open the Survey itself, so this is a reported figure.
- ₹3.83 crore average receivables unpaid for over 360 days per SME, and 73 days average to settle an invoice: Recordent Indian SME Receivables Report 2026 (about 1.1 lakh MSMEs), via MediaBrief and Telangana Today. The sample skews to larger SMEs.
- 2% + GST listed rate and the UPI platform-fee wording: razorpay.com/pricing.
- UPI MDR from 15 Oct 2026 (0.4% above ₹2,000, capped at ₹300): Ministry of Finance (PIB) release and the Department of Financial Services FAQ, Sep 2026. The FAQ does not say how this applies to payment aggregators or B2B invoice payments.
