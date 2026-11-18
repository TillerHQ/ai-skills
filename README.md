# Tiller AI Skills

Skills that put AI to work on your Tiller spreadsheet, the financial record you own.

| Skill | What it does | Status |
| --- | --- | --- |
| [Good Morning, Money](skills/good-morning-money/) | A short daily brief: big charges, new payees, possible fraud, subscriptions, balances and stale connections. Optional email delivery. | Prototype |

## Use a Skill

Each skill lives in its own folder:

```
skills/<skill-name>/
  README.md   what it does and how to try it (for people)
  SKILL.md    the instructions the AI follows (plain English, edit freely)
  examples/   sample output (optional)
```

You'll need the Tiller connector (MCP: `https://ai-tools.tillermoney.com/mcp`) in your AI tool.

- **Claude:** download the skill folder as a zip and upload it in Claude's skills settings.
- **ChatGPT, Gemini and others:** copy the SKILL.md text into a Project, custom GPT or Gem as instructions.

## Feedback & Edits

These are early prototypes. Tell us what works, what breaks and what you changed: open an issue, suggest an edit with a pull request, or post in the [Tiller Community](https://community.tiller.com/c/ai-workflows/52).

## Use at your own risk

These skills are experimental prototypes, shared as-is with no warranty of any kind. They give an AI assistant instructions to read, and in some cases change, your Tiller spreadsheet and other connected data.

- **AI makes mistakes.** It can misread transactions, miscalculate, misclassify, or miss things entirely. Check anything important against your spreadsheet and your bank before acting on it.
- **Back up first.** Before using any skill that can edit your data, make a copy of your spreadsheet (File > Make a copy). Google Sheets also keeps version history (File > Version history) if you need to roll back.
- **Not financial, tax or legal advice.** The output is informational only.
- **You're responsible for what you connect and run.** That includes which AI tool you use, what data you share with it, any API keys you add, and any changes the AI makes at your direction.

Tiller is not liable for data loss, errors, or decisions made based on these skills' output.

## Privacy

Skills read your sheet through the Tiller connector. Category Health Check never changes anything. Never put API keys or account numbers in a skill or a shared prompt.

