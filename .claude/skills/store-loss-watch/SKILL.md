---
name: store-loss-watch
description: Loss-prevention watch for one FavTrip store. Daily, log Modisoft sales by tender and the deposits they should produce, and flag shrink signals. Weekly, log into the bank in Chrome (read-only) and match every deposit, card settlement, delivery payout and ACH bill against what was expected. Trigger on "run loss watch", "daily sales check", "weekly bank check", "did the deposits clear", or "are the bills paid right".
---

# Store Loss Watch

You watch ONE store for lost money. You do not move money, pay bills, or change anything in the bank or Modisoft. Read, log, flag.

## Setup (fill once)

- Store: `<STORE NAME>`
- Google Sheet log: `<SHEET URL>` (tabs below)
- Report email: babir@favtrip.com
- Bank: user is already logged in in Chrome. If the session has expired, stop and ask. Never ask for or store a password.
- Expected bills list: tab `Bills` (vendor, amount or range, due day, ACH or check)

## Sheet tabs

| Tab | Columns |
|---|---|
| `Daily` | Date, Gross sales, Cash, Credit, Debit, EBT, DoorDash, UberEats, Grubhub, Other delivery, Lottery sales, Lottery payouts, Paid-outs, Refunds, Voids, No-sales, Cash over/short, Expected cash deposit, Flags |
| `Expected` | Date expected, Source (cash / card batch / delivery app), Expected amount, Sales date(s) covered, Status (open / matched / missing / off) |
| `Bank` | Post date, Description, Amount, Type (deposit / ACH / check / fee), Matched to, Variance, Note |
| `Bills` | Vendor, Expected amount, Due day, Method, Last paid, Status |
| `Flags` | Date, Severity (red / yellow), What, Amount, Status |

## Daily check (every morning, previous day)

1. Pull yesterday's Modisoft daily sales / shift report for the store.
2. Write one row to `Daily`.
3. Add rows to `Expected`:
   - Cash deposit = cash sales minus paid-outs minus lottery payouts, expected next bank day.
   - Card batch = credit + debit, expected in 1 to 2 bank days, net of processor fees.
   - Each delivery app = that day's delivery sales, expected on the app's payout day net of commission.
   - EBT, expected in 1 to 2 bank days.
4. Flag (write to `Flags`):
   - RED: cash over/short worse than $20, or any day short 3 days in a row.
   - RED: refunds or voids more than 2x the 30-day average, or above $100.
   - YELLOW: no-sales above 15 in a day, or one cashier with most of them.
   - YELLOW: lottery sales vs lottery report mismatch.
   - YELLOW: gross sales down more than 25% vs same weekday average.
5. Email only if there is a RED flag. Subject: `[Loss Watch] <STORE> <date> RED`.

## Weekly check (Monday, prior Mon to Sun)

1. Open the bank in Chrome. Read the last 10 days of transactions. Log each to `Bank`.
2. Match each open `Expected` row:
   - Cash deposit: matched if within $1.
   - Card batch: matched if within the normal fee range (record the fee %). Flag if the fee % jumps.
   - Delivery payouts: matched to that week's delivery sales minus commission. Flag if payout is short more than 5% beyond normal commission.
3. Match ACH debits and checks to `Bills`:
   - Paid, right amount: mark paid.
   - Paid twice, wrong amount (off more than $5 or 2%), or unknown payee: RED.
   - Due and not paid: YELLOW.
4. Any `Expected` row open more than 3 bank days: mark `missing`, RED.
5. Balance check: starting balance + deposits − debits = ending balance. Flag any gap.
6. Email the weekly summary. Subject: `[Loss Watch] <STORE> week of <date>`. Body:
   - Sales total vs deposits total, and the gap
   - Missing or short deposits (list)
   - Bill problems (list)
   - Shrink signals from the week's daily flags
   - Open items carried to next week

## Rules

- Read-only in the bank. Never click pay, transfer, or edit.
- Never put account numbers or full card numbers in the sheet or email. Last 4 only.
- If a number cannot be found, write "not found" and flag it. Do not guess.
- Keep the email short: numbers and lists, no padding.
