# AgentiLoop skills

[Agent Skills](https://agentskills.io) for AI coding agents (Claude Code, Codex, Cursor, OpenCode, Gemini CLI, …), built around [qrcode.pub](https://qrcode.pub/?ref=skills).

[![skills.sh](https://skills.sh/b/AgentiLoop/skills)](https://skills.sh/AgentiLoop/skills)

```bash
npx skills add AgentiLoop/skills            # pick skills interactively
npx skills add AgentiLoop/skills --all      # install every skill for every detected agent
npx skills add qrcode.pub                   # same skills via /.well-known/agent-skills
```

| Skill | What it teaches the agent |
|---|---|
| [`qr-code-generator`](skills/qr-code-generator/SKILL.md) | QR code images (PNG/SVG) for any URL or text with one HTTP GET — no key, no library; Markdown/HTML embeds, bulk, static vs dynamic codes. |
| [`x402-web-tools`](skills/x402-web-tools/SKILL.md) | Pay-per-call web tools for agents with an x402 wallet (USDC on Base/Solana): page → Markdown, page metadata, 30-day file hosting, QR codes; also a remote MCP server. |

Each skill is a single `SKILL.md` (YAML front matter + instructions) under `skills/<name>/`. No code runs on install.

Live docs: https://qrcode.pub/qr-code-api · Issues and ideas: open one here. MIT licensed.
