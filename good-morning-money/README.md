# Good Morning, Money ☀️

**Status:** prototype · **Works with:** Claude (skill or scheduled task), ChatGPT/Gemini (paste as instructions) · **Needs:** Tiller connector

A short daily brief from your Tiller spreadsheet: a plain-English summary first, then big charges, new payees, possible fraud, subscriptions, upcoming bills, balances and bank connections that need attention.

## Try it

1. Connect the Tiller connector (MCP: `https://ai-tools.tillermoney.com/mcp`).
2. Add the skill: upload this folder to Claude's skills, or paste [SKILL.md](SKILL.md) into a Project, custom GPT or Gem.
3. Ask: "Give me my Good Morning, Money brief."
4. Optional: make it a daily scheduled task, and email it with your own Resend key (see Delivery in SKILL.md).

## Make it yours

Edit the Settings table at the top of SKILL.md: large-charge threshold, which balances to show, accounts to ignore, tone ("plain" drops the emoji).

## Privacy

Read-only: it never edits your transactions. Keep API keys and account numbers out of the skill and out of anything you share.

## Feedback

What worked, what broke, what you changed: open an issue or post in the [Tiller Community](https://community.tiller.com/c/ai-workflows/52).
