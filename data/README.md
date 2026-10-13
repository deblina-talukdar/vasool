# Synthetic dataset

Entirely synthetic: fictional businesses and amounts, for prototype demonstration only. Reference date: 9 Oct 2026 (the app offsets dates from today).

- vasool_payers.csv: 10 payers with preferred channel, language, best contact time, usual days to pay, key-account and never-auto-contact flags, past disputes.
- vasool_invoices.csv: 34 invoices (INV-1001 to INV-1034) with status (open, overdue, partially_paid, disputed, paid_on_time, paid_late), balances, reminders sent and notes.
- vasool_test_replies.csv: 15 labelled payer replies (10 English, 5 Hinglish) with expected intent: paid, split, reschedule, dispute, resend_invoice, opt_out, unclear.
- vasool_data.json: all three tables plus metadata.
- vasool_synthetic_data.xlsx: the same data as a three-tab workbook.

Edge cases built in: a part-paid invoice (INV-1016, balance Rs 98,000), a disputed invoice (INV-1014), a payer who must never be auto-contacted, a chronically late payer who only answers calls, a new payer with no history.
