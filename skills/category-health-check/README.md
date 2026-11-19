# Category Health Check 🩺

**Status:** prototype · **Works with:** Claude, ChatGPT/Gemini (paste as instructions) · **Needs:** [Tiller MCP connector](https://community.tiller.com/t/ai-for-personal-finance-starts-with-a-spreadsheet-you-own-introducing-tillers-claude-connector/35195)

A read-only checkup of your Tiller categories over the last few years: unused and catch-all categories, misfiled payees, money in the wrong place, hidden spending, unbalanced transfers and budget drift. It also shows how your transactions get categorized (rules vs. AI) and suggests AutoCat rules to add, with sample transactions so you can spot-check every finding.

## Try It

1. Connect the Tiller connector (MCP: `https://ai-tools.tillermoney.com/mcp`).
2. Add the skill: upload this folder to Claude's skills, or paste [SKILL.md](SKILL.md) into a Project, custom GPT or Gem.
3. Ask: "Run a Category Health Check on my sheet." It starts with up to four quick questions about your goals.

## Notes

- The Tiller MCP (alone) cannot read the AutoCat rules sheet at this time. If you'd like your AI to cross-reference your rules, reading it through a Google Drive or Sheets connector.

## Make It Yours

Edit the Settings table in SKILL.md: review window, what counts as underused or too broad.

## Privacy

Read-only: it recommends changes but never makes them. Your sheet stays exactly as it was.

## Use at Your Own Risk

Experimental and provided as-is. AI can make mistakes: double-check anything important against your sheet and your bank. Back up your spreadsheet before using skills that edit data. Not financial advice. See the [full disclaimer](../../README.md#use-at-your-own-risk).

## Feedback

Open an issue or post in the [Tiller Community](https://community.tiller.com/c/ai-workflows/52).
