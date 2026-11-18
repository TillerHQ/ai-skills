# Tiller AI Skills

Skills that put AI to work on your Tiller spreadsheet, the financial record you own.

| Skill | What it does | Status |
| --- | --- | --- |
| [Good Morning, Money](skills/good-morning-money/) | A short daily brief: big charges, new payees, possible fraud, subscriptions, balances and stale connections. Optional email delivery. | Prototype |

## Use a skill

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

## Feedback and edits

These are early prototypes. Tell us what works, what breaks and what you changed: open an issue, suggest an edit with a pull request, or post in the [Tiller Community](https://community.tiller.com/c/ai-workflows/52).

## Privacy

Skills read your sheet through the Tiller connector. Category Health Check never changes anything. Never put API keys or account numbers in a skill or a shared prompt.

