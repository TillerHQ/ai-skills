---
name: good-morning-money
description: "A short, playful daily money brief from a Tiller spreadsheet via the Tiller MCP: big charges, new payees, possible fraud, subscriptions, balances and stale connections, optionally emailed. Use for a daily money brief or a Good Morning, Money update."
---

# Good Morning, Money

A quick, skimmable daily brief built from the user's Tiller spreadsheet using the Tiller MCP tools. It reads; it never edits transactions.

## Settings

Use these defaults unless the user (or a scheduled task prompt) gives different values.

| Setting | Default |
| --- | --- |
| Spreadsheet | The only spreadsheet from `list_spreadsheets`. If there are several and none is named, ask; on an unattended run, use the first and say so. |
| Time zone | The user's time zone, for "yesterday" and "today" |
| Look-back window | Last 24 hours, by the **Date Added** column (fall back to transaction date if the sheet has none) |
| Large charge threshold | $100 |
| Significant balance change | $1,000 or 10%, whichever is larger |
| Stale connection | No aggregator update in 2+ days, but updated within the last 30 days (older ones are treated as abandoned and skipped) |
| Ignore accounts | None. Account names to leave out of connection warnings and balances |
| Balance snapshot | Every checking account, plus credit cards with 5+ purchases (Expense transactions, not payments or returns) in the last 30 days. Or list accounts by name. |
| Upcoming bills window | Next 5 days |
| Uncategorized and tidy-up window | Last 30 days |
| Tone | Playful, emoji-led, brief ("plain" drops emoji and wordplay) |
| Email delivery | Off. To turn on, see Delivery. |

## Steps

1. **Pick the spreadsheet** (see Settings).
2. **New transactions:** `query_transactions` filtered to the look-back window. Use Date Added, not date: banks often post late with an earlier date. Date Added is date-only text like `11/15/2026` (no leading zeros), so filter with `$in` on yesterday's and today's dates.
3. **History for context:** query the past ~13 months by payee or `search` to learn recurring payees, typical amounts and known payees.
4. **Balances:** `query_balance_history` with `latestPerAccount: true`, plus each account's previous snapshot for the change.
5. **Connection health:** `list_accounts`, limited to accounts linked to the spreadsheet (per `list_spreadsheets`) and not ignored. Flag only accounts that went stale or started erroring within the stale window; skip abandoned ones silently. Archived accounts are already excluded.
6. **Uncategorized:** count transactions with `category.type: "Uncategorized"` in the last 30 days.
7. **Sheet tidy-up:** run one query over the last 30 days only and report categories its audit says are missing from the Categories sheet. Ignore such warnings from history queries.
8. **Write the brief** (Format), then deliver it (Delivery). Classify with `category.type`, never by amount sign. Skip transfers between the user's own accounts unless large or odd.

## What to look for

- **Large charges** over the threshold. For known recurring payments use familiar words: "You just paid the mortgage," "Car insurance went out."
- **Money in:** paychecks, refunds, credits ("Your paycheck landed").
- **New payees:** merchants with no prior history in the sheet.
- **Possible fraud** — flag calmly, don't accuse: small "test" charges under $5 from unfamiliar merchants; a new payee with an odd amount; foreign or garbled merchant names; duplicates (same merchant and amount within ~2 days); activity on a rarely used account.
- **The user's notes come first.** If a transaction, or an earlier one from the same merchant and amount, has a Note or Tags, describe it by that note. The sheet is the user's memory; corrections there should change future briefs.
- **Subscriptions:** only if the same merchant charged a similar amount (within ~10%) monthly or yearly (look 11–13 months back). Varying amounts or several charges a month (rentals, app-store purchases) are purchases, not subscriptions. Call out yearly renewals ("🔄 Annual renewal") and price increases.
- **Coming up:** recurring bills likely due in the upcoming window.
- **Balance swings** past the threshold, with a likely reason.

## Format

1. **Summary first, no emoji:** 2–4 short sentences of plain prose, inverted-pyramid style. Most important first (anything needing action, like possible fraud or a broken connection), then the biggest money moves, then everything else in one closing clause. Someone who reads only the first sentence gets the key point. No greeting or date in the summary.
2. **Sections**, in this order, only those with content. Every item under a header is a bullet (`what — amount (account)`), never a bare paragraph:
   🚨 Check these · 🔌 Connection trouble · 💸 Big charges · 💰 Money in · 👋 New faces · 🔁 Subscriptions · 📅 Coming up · 📈📉 Balance moves · 📝 Noted · 💳 Balances · 🏷️ Uncategorized · 🧹 Sheet tidy-up
3. **💳 Balances** is always included, one account per line, using short account names ("Checking", "Visa"). Credit cards as amount owed. If a balance is older than 24 hours, mark it "⏳ as of <Mon D>".
4. Use a fitting emoji per item when it adds meaning: 🏠 mortgage/rent, 🚗 car, ⚡ utilities, 🛒 groceries, 💳 card payment, 🎉 paycheck, ↩️ refund, 🔺 price increase.
5. Balance moves: 📈 for up, 📉 for down; include the change and the new balance. Whole dollars unless cents matter. Light wordplay welcome ("Netflix wants $2 more 👀"), never at the expense of accuracy. No sign-off.
6. **Quiet day:** a one-sentence summary ("All quiet: 4 small transactions, nothing unusual.") plus any connection warning and the Balances.
7. Never invent data. If a tool call fails, say which part of the brief is missing.

## Delivery

**Default:** the brief is the final chat message.

**Email (optional, via Resend):** only when the user has set it up and provided their own values. Never hard-code anyone's key in this skill or in shared prompts.
- Needed: a Resend API key (ideally send-only), a from address (`onboarding@resend.dev` works without a domain, but only to the Resend account's own email), and a to address.
- The run environment must be allowed to reach `api.resend.com`.
- Read the key from where the user put it (an environment variable such as `RESEND_API_KEY`, or their scheduled task's own instructions).
- Build the payload with Python (`json.dumps`) and POST it to `https://api.resend.com/emails` with `Authorization: Bearer <key>`. A success returns an `id`; retry once on failure, then report the error in the final message.
- **Subject:** "☀️ Good Morning, Money: <Wkday, Mon D>"
- **text:** the plain-text brief, "• " bullets, blank lines between sections.
- **html:** responsive, inline styles only, no manual line breaks inside sentences:
  - Outer full-width table; inner centered table `width:100%;max-width:680px`; cell font `-apple-system,'Segoe UI',Helvetica,Arial,sans-serif`, 16px, line-height 1.45.
  - Summary `<p>` at 17px with a 26px bottom margin.
  - Each section: a bold header `<p>` (6px bottom margin), then a `<ul>` (22px left padding, 24px bottom margin) with one `<li>` per item.
  - Balances: a full-width two-column table, same 16px font, thin divider under each row; left cell is the account name (may wrap) with any "⏳ as of" note on a smaller second line; right cell is the amount only, right-aligned, `white-space:nowrap`.
  - Escape &, < and >.
- **Discuss button (optional):** a centered button at the end, 36px top margin, labeled "Discuss My Finances". When running as a scheduled task, link to this run's own session: call `list_triggers`, find this task, take `last_run.session_id` (only if `fired_at` is within the last hour), and link `https://claude.ai/code/session_<id without "cse_">`. Otherwise link `https://claude.ai/new?q=<url-encoded prompt>` asking to discuss today's brief. Keep account numbers out of links.
