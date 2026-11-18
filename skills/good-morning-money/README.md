# Good Morning, Money ☀️

**Status:** prototype · **Works with:** Claude (skill or scheduled task), ChatGPT/Gemini (paste as instructions) · **Needs:** Tiller connector

Similar to Tiller's "Hello Money", "Good Morning, Money" 😉 is an AI-driven version that you can fully customize. 

This skill creates a short daily brief from your Tiller spreadsheet: a plain-English summary first, then big charges, new payees, possible fraud, subscriptions, upcoming bills, balances and bank connections that need attention.

Tell Claude if/how you'd like the content delivered. 

## Try It

1. Connect the Tiller connector (MCP: `https://ai-tools.tillermoney.com/mcp`).
2. Add the skill: upload this folder to Claude's skills, or paste [SKILL.md](SKILL.md) into a Project, custom GPT or Gem.
3. Ask: "Give me my Good Morning, Money brief."

### Optional:  Automated Delivery 

Make this skill a daily scheduled task for delivery by push notification, email or Slack.

I have found Claude's internal email delivery is unreliable so I am using a [Resend](https://resend.com) free/personal account myself. Resend allows up to 100 emails/day on free accounts.

## Make It Yours

Edit the Settings table at the top of SKILL.md: large-charge threshold, which balances to show, accounts to ignore, tone ("plain" drops the emoji).

## Privacy

Read-only: it never edits your transactions. Keep API keys and account numbers out of the skill and out of anything you share.

## Feedback

What worked, what broke, what you changed: open an issue or post in the [Tiller Community](https://community.tiller.com/c/ai-workflows/52).
