---
name: category-health-check
description: "A read-only review of a Tiller spreadsheet's categories over the past 1–3 years: fit with the user's goals, unused or overly broad categories, misfiled payees, hidden spending, transfers and categorization coverage, with suggested AutoCat rules and recommendations the user can spot-check. Use when asked for a category review, category checkup or Category Health Check."
---

# Category Health Check

A user-invoked, **read-only** review of how well a Tiller user's categories serve them. It recommends; it never changes anything. Every finding comes with sample transactions so the user can spot-check it in their sheet.

## Ground rules

- **Read-only.** Never call add_category, update_category, rename_category, rename_category_group, delete_category or update_transactions. If the user asks for a fix, describe the exact steps instead, or, only if they explicitly ask in the conversation, make the specific change they name.
- **Tiller does the math.** Use `query_transactions` aggregations (count, sum) for every number. Never estimate totals in your head. Classify with `category.type`, never by amount sign.
- **Fresh data only.** Re-query the sheet every time the skill runs, even if earlier results are in the conversation: the user may have fixed things since. Every number, example and sample transaction in the report must come from this run's queries, never from memory or an earlier report.
- **Show your evidence.** Each finding cites the query numbers behind it and up to 3 sample transactions (date, description, amount, account).
- **Be humble about intent.** The user knows their life; frame unclear findings as questions ("Is Household meant to include insurance?").

## Settings

| Setting | Default |
| --- | --- |
| Spreadsheet | The only one from `list_spreadsheets`; if several and none named, ask |
| Review window | Last 3 years (use 1–2 if the sheet is younger or the user asks) |
| Recent window | Last 12 months, for "unused" checks |
| Coverage windows | Last 90 days and last 12 months |
| Underused | Fewer than 5 transactions in 12 months |
| Too broad | 15%+ of expense dollars, or 40+ distinct payees of unrelated kinds (many restaurants in Eating Out is fine; restaurants, hardware and pharmacies together is not) |
| Large type mismatch | Over $500 |
| Clear payee pattern | 2+ past transactions with 75%+ in one category |
| Rule candidate | 5+ transactions in 12 months, 95%+ in one category, categorized by AI Suggest or Description Match |

## Step 1: Context (short interview)

Unless the user has already said, ask up to 4 quick questions in one message:
1. What do you mainly use Tiller for? (budgeting, tracking, taxes, planning, a business)
2. Who else uses or relies on the sheet?
3. What questions do you wish it answered better?
4. Anything big now or ahead? (kids in college, retirement, a move, a property)

If the user says "just run it," proceed, infer what you can from the data, and list your assumptions at the top of the report.

## Step 2: Gather (read-only)

1. **Categories:** `list_categories` for names, types, groups, budgets and hide-from-reports flags.
2. **Usage:** aggregate `query_transactions` by `category` (count, sum) over the review window, then the recent window. Don't add `category.type` to these: if orphan categories exist, a type-based query refuses to run.
3. **Audit block:** note categories used in transactions but missing from the Categories sheet ("unmatched"), categories present without a Type ("untyped"), and uncategorized counts.
4. **Hidden categories:** aggregations skip hide-from-reports categories and there is no switch to include them. For each hidden category (and every Transfer-type category), list rows with a filter and limit, and read the row count from the audit's filter step.
5. **Type mismatches:** expense-type categories with inflows over the threshold, and income-type categories with outflows. If orphans make the query refuse, filter by the expense or income category names instead of `category.type`.
6. **Inside big categories:** for the largest 3–5 expense categories, aggregate by `description` to count distinct payees and see what's inside.
7. **Payees:** group transactions by a normalized payee: lowercase, strip processor prefixes (`tst*`, `sq *`, `sp `, `fsp*`, `bkg*`, `py *`, `pp*`), store numbers and trailing location text. Use this for recurring payees, inconsistent payees and rule candidates.
8. **Coverage:** aggregate by `Categorized By` over both coverage windows. Values include Rules, Description Match and AI Suggest; blank with a category means set by hand or imported; blank category means uncategorized.
9. **Transfers:** aggregate Transfer-type transactions by month; each month should net close to zero.

## Step 3: Diagnose

Score each category against these checks. A category is **healthy** if it passes all of them.

| Check | Signal |
| --- | --- |
| Unused | 0 transactions in the recent window |
| Underused | Below the underused threshold |
| Too broad | Above the too-broad threshold; unrelated payees mixed |
| Catch-all | Misc, Other, uncertain or placeholder names; many uncategorized |
| Inconsistent | A payee with a clear pattern, but some transactions in other categories |
| Type mismatch | Inflows in expense categories or outflows in income categories over the threshold |
| Orphan | Used in transactions but missing from the Categories sheet |
| Case variant | Same name differing only by case ("fun" vs "Fun"); the tool matches them anyway, but reports may split them |
| Untyped or misgrouped | Category without a Type, or a group that contradicts its use (e.g. purchases in a Transfer-type category) |
| Budget drift | Budget set but unused, heavy spend with no budget, or budgets that stop at an old month |
| Hidden money | Hidden or Transfer-type categories holding real purchases |
| Bulk-import residue | Many rows sharing one Date Added with a blank "Categorized By", often carrying another app's category names |
| Poor fit | Recurring payees whose category name doesn't describe them |
| Suspect rule | Transactions categorized by Rules whose category disagrees with that payee's clear pattern: a sign of an overly broad or outdated AutoCat rule |
| Unbalanced transfers | Months where transfers don't net near zero, or one-sided transfers: a missing account or a payment filed as spending |
| Job-to-be-done gap | Something the user cares about that no category isolates |

Treat **reimbursement netting** (refunds or paybacks filed in the category they offset, e.g. a tax refund in Taxes) as a valid choice, not an error: mention it once, and flag only inflows that don't look like paybacks.

Also look for **trends**: spending shifting between categories, categories that faded, and new kinds of spending with no good home.

**Rule candidates:** payees meeting the rule-candidate setting. Propose one AutoCat row each (Description Contains + Category), using a distinctive keyword that won't match unrelated merchants. This skill can't read the AutoCat sheet, so tell the user to check for an existing rule first and to place new rows above broader rules (AutoCat applies the first match).

## Step 4: Report

Succinct and skimmable. Lead with the answer.

```
🩺 Category Health Check — <spreadsheet>, <window>

Summary (2–4 sentences, plain prose, most important first)

Assumptions (only if no interview)

Scorecard: <N> categories · <N> healthy · <N> need attention
Coverage (last 90 days): Rules <x>% · Description Match <x>% · AI Suggest <x>% · by hand <x>% · uncategorized <x>%

👀 Check these 3 first  (the three findings most worth verifying, one line each)

🔴 Fix first            (type mismatches, orphans, hidden money, unbalanced transfers, catch-alls hiding real money)
🟡 Worth a look         (too broad, inconsistent payees, poor fit, suspect rules)
⚪ Tidy up              (unused, underused, case variants, budget drift)
📈 Trends
🎯 Fit with your goals  (job-to-be-done gaps)
🤖 Rules to add         (AutoCat rows: keyword → category, with match count)
✅ Recommended changes  (numbered, smallest effort first)
❓ Questions for you
```

Before sending, check every cited transaction against this run's query results and drop any whose category no longer matches the finding.

For each finding: one line on what and why, the numbers, and up to 3 sample transactions. For each recommended change: what to do, why, effort (quick / moderate / bigger), and side effects (history, budgets, templates, AutoCat rules). Prefer forward-looking fixes (new AutoCat rules, a new category going forward) over rewriting history.
