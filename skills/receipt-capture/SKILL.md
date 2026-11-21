---
name: receipt-capture
description: "Snap a photo of a receipt and file it into a Tiller spreadsheet: read the receipt, match the card to a Tiller account, store the image in Google Drive, and either annotate the bank transaction that already posted or add a placeholder row that Tiller's matcher merges when the charge arrives. Use when the user shares a receipt photo or asks to capture or file a receipt."
---

# Receipt Capture

Turns a receipt photo into a linked, categorized entry in the user's Tiller sheet without creating duplicates.

## Needs

- **Tiller connector** (MCP): find transactions and accounts; write plain-text cells such as Category and Note text. It writes values as text, so it **can't** create a working link.
- **Google Sheets connector:** write the `=HYPERLINK(...)` formula (`update_formulas`), read existing cells and links (`get_spreadsheet` with `userEnteredValue` and `hyperlink`), and add placeholder rows (`insert_dimension` + `update_formulas`).
- **Google Drive connector:** store the receipt image and get its link.
- Placeholder rows rely on Tiller's transaction matcher (Google Sheets only). If the user's sheet doesn't have it, use the attach-later fallback.
- If the Sheets connector is missing, ask the user to connect it. Don't fall back to a plain-text link.

## Settings

| Setting | Default |
| --- | --- |
| Spreadsheet | The only one from `list_spreadsheets`; if several and none named, ask |
| Drive folder | "Tiller Receipts" (create if missing) |
| Match window | ±5 days from the receipt date |
| Tip range | Final charge from the receipt total up to 130% of it (tip-likely businesses only) |
| Link label | 🧾 Receipt |
| Confirm before writing | On: show what will be written and wait for "yes" |

## Steps

1. **Read the receipt.** Extract merchant, date, time, subtotal, tax, printed tip, handwritten tip, total, card brand and last 4 digits, payment type (card, cash, check), and line items. Note anything unreadable rather than guessing.
2. **Settle the tip.**
   - If a handwritten tip and/or total is legible, use the handwritten total.
   - If the business is one where people usually tip (restaurant, café, bar, salon, spa, taxi or rideshare, delivery) and the tip can't be read confidently, ask: "Did you add a tip? If so, how much?" Use the answer to set the final amount.
   - Otherwise use the printed total.
3. **Match the account.** Compare the receipt's last 4 digits with the `Account #` of accounts linked to the spreadsheet (`list_spreadsheets`, `list_accounts`). One match: use it. None or several: ask which account. Cash: ask whether to record it, and in which (manual) account.
4. **Look for the bank transaction first.** `query_transactions` on that account within the match window, with an amount equal to the final amount (or within the tip range if the tip is still unknown). Use merchant-name similarity to break ties.
   - **One clear match:** annotate it (steps 6–7). Don't add a row.
   - **Several possible matches:** show them and ask.
   - **No match:** add a placeholder row (step 8).
5. **Store the image.** Save the photo to the Drive folder named `YYYY-MM-DD Merchant $Amount`, sharing restricted to the user. Put the line items and tip in the file's description, so itemization stays searchable without new sheet columns. Get the file's link.
6. **Find the row** (Sheets connector).
   - Read the header row (`Transactions!1:1`) to get the column letters for Transaction ID, Note, and any column named Link, Receipt or Attachment (case-insensitive).
   - Read the Transaction ID column in chunks (rows 2–500, then 501–1000, …) until the ID is found. Recent rows are usually near the top.
   - Right before writing, re-read that row with `get_spreadsheet` (`userEnteredValue`, `hyperlink`). Confirm the Transaction ID still matches, since sorting or a fill can shift rows, and check what the Note and link cells already hold.
7. **Annotate the transaction.** Change only these cells:
   - **Link column exists:** write `=HYPERLINK("<drive link>","🧾 Receipt")` there (`update_formulas`), and a short description in Note (e.g. "Latte, scone + $2 tip") with the Tiller connector.
   - **No link column:** write only `=HYPERLINK("<drive link>","🧾 Receipt")` in Note (`update_formulas`), with no other text.
   - **Never overwrite** an existing Note or link; if either already has content, ask first.
   - Set a category (Tiller connector) only if the transaction is uncategorized and the user agrees. When you do, also set Categorized By and Categorized Date (below).
   - **Verify:** re-read the cell and confirm `hyperlink` is the Drive link. If it came back as text, report that and stop.
8. **Add a placeholder row** (Sheets connector), when the charge hasn't posted:
   - **Always put it in row 2, directly below the header.** Get the Transactions sheet's `sheetId` (`get_spreadsheet`, `sheets.properties`), then `insert_dimension` with ROWS, startIndex 1, endIndex 2, `inheritFromBefore: false`, so the new row takes the data rows' formatting instead of the header's. Never append at the bottom.
   - Write the whole row to `Transactions!2:2` in one `update_formulas` call, in header order, so Sheets parses it as if typed: dates become dates, numbers stay numbers, formulas work. If any text value starts with `=`, `+`, `-` or `@`, prefix it with `'` so it stays text. The row holds:
     - **Transaction ID:** `match:` + a unique ID, so Tiller's matcher merges the bank charge into this row instead of adding a duplicate. For cash with no bank side, use `manual:` + a unique ID.
     - **Date** as M/D/YYYY, so Sheets stores it as a date. **Description:** the merchant. **Amount:** a negative number, the final amount including tip.
     - **Category:** from the merchant's past transactions when there's a clear pattern; otherwise ask or leave blank.
     - **Account fields** from step 3: Account, Account #, Account ID, Institution, so the matcher can use them.
     - The bookkeeping columns (below) and the link and Note as in step 7.
   - Re-read row 2 and confirm the Transaction ID is the new one, the link works, and Date, Amount, Month and Week read back as dates and numbers.
   - Tell the user the row is a placeholder until the bank charge arrives.
9. **Attach-later fallback** (no matcher): don't add a row. Keep the receipt in Drive and, on a later run (or in a daily brief), match it to the posted charge and annotate as in steps 6–7.

## Bookkeeping columns

Fill these on every placeholder row, but only for columns the sheet already has (match headers case-insensitively). Never add columns.

| Column | Value |
| --- | --- |
| Month | Formula off the row's Date cell: `=EOMONTH(A5,-1)+1` (first of the month) |
| Week | Formula off the row's Date cell: `=A5-WEEKDAY(A5)+1` (Sunday starting the week) |
| Full Description | Same text as Description |
| Date Added | Current date and time |
| Categorized By | `Receipt skill` |
| Categorized Date | Current date and time (only when a category is set) |
| Source | `Receipt skill` |

- In the formulas, replace `A5` with the row's actual Date cell: the Date column from the header row, in row 2 (e.g. `=EOMONTH(A2,-1)+1`). Use plain per-row formulas, never ARRAYFORMULA.
- Write formulas with `update_formulas`; other write tools may store them as text.
- Write timestamps as fixed values in the user's time zone (e.g. `11/21/2026 14:05:00`), never `=NOW()`, which would keep changing.
- On an existing bank transaction, change only Categorized By and Categorized Date, and only when the skill sets the category. Leave the other columns as Tiller filled them.

## Confirm, then report

Before writing, show a compact preview:

```
🧾 Blue Door Café — Nov 20, $14.63 incl. $2.00 tip
Card: Visa ••1234 → Atmos Visa
Action: annotate the posted charge (Nov 20, -$14.63)   or   add placeholder row
Category: Eating Out
```

After writing, confirm in one line what changed and where.

## Guardrails

- Change only the cells listed above. Never change amounts, dates or descriptions on bank transactions.
- Never guess an account, tip or amount: ask.
- Keep receipt images private; never share them by link with "anyone."
- Card numbers beyond the last 4 digits never go into the sheet or file names.
