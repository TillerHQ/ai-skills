# Receipt Capture 🧾

**Status:** prototype · **Works with:** Claude (tested) · **Needs:** Tiller, Google Sheets and Google Drive connectors

Snap a photo of a receipt and file it into your Tiller sheet. The AI reads the receipt, matches the card to your Tiller account, saves the image to Google Drive, and adds a 🧾 Receipt link to the matching transaction. If the charge hasn't posted yet, it adds a placeholder row that Tiller merges with the bank transaction when it arrives, so you don't get duplicates.

## What it does

- **Reads the receipt:** merchant, date, total, tax, tip, card last 4 and line items. Line items go in the Drive file's description, so your sheet doesn't need new columns.
- **Asks about tips:** at restaurants, salons, rideshare and other places where people tip, it asks if the tip isn't clear.
- **Finds the transaction first:** searches the matching account within ±5 days. One clear match gets the link. Several possible matches: it asks.
- **Adds a placeholder** when nothing has posted: a row at the top of Transactions that Tiller's matcher merges when the bank charge arrives.
- **Shows a preview** and waits for your "yes" before writing anything.

## Connectors

| Connector | Why |
| --- | --- |
| **Tiller** (MCP: `https://ai-tools.tillermoney.com/mcp`) | Find transactions and accounts, set categories |
| **Google Sheets** | Write the 🧾 Receipt link (the Tiller connector writes plain text only, so links need Sheets) and add placeholder rows |
| **Google Drive** | Store the receipt images |

**Let the connector create the Tiller Receipts folder.** Claude's Google Drive connector can only see files and folders it created itself, not the rest of your Drive. Let the skill create "Tiller Receipts" on its first run. A folder you make yourself is invisible to the connector. This keeps the AI's Drive access limited to your receipts.

Placeholder rows need Tiller's transaction matcher, which works only in Google Sheets. Without it, the skill keeps the receipt in Drive and links it once the charge posts. Excel isn't supported.

## Optional columns

The skill never adds columns. It uses these if your Transactions sheet already has them:

| Column | What it gets |
| --- | --- |
| **Link**, **Receipt** or **Attachment** | The 🧾 Receipt link, with a short description in Note (e.g. "Latte, scone + $2 tip"). Without one of these columns, the link goes in Note by itself. |
| **Month** | Formula: the first of the month, from Date |
| **Week** | Formula: the Sunday starting the week, from Date |
| **Source** | "Receipt skill" |

Placeholder rows also fill Full Description, Date Added, Categorized By ("Receipt skill") and Categorized Date when those columns exist.

## Try it

1. Connect the Tiller, Google Sheets and Google Drive connectors.
2. Add the skill: upload this folder to Claude's skills.
3. Share a receipt photo and say "File this receipt."

## Make it yours

Edit the Settings table in SKILL.md: Drive folder name, match window, tip range, link label.

## Privacy

Receipt images stay private in your own Drive and are never shared by link. Only the last 4 digits of a card are used. It never changes the amount, date or description of a bank transaction. It writes only the link, Note, and (with your OK) the category.

## Use at your own risk

Experimental and provided as-is. This skill edits your spreadsheet: back it up first (File > Make a copy). AI can misread receipts, so check the preview before saying yes. Not financial advice. See the [full disclaimer](../../README.md#use-at-your-own-risk).

## Feedback

Open an issue or post in the [Tiller Community](https://community.tiller.com/c/ai-workflows/52).
