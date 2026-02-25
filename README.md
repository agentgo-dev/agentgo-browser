# agentgo-browser

AgentGo cloud browser skill for AI coding agents (Claude Code, GitHub Copilot, etc.).

Connect to [AgentGo](https://app.agentgo.live/)'s distributed browser cluster using `playwright@1.51.0` — no local browser needed.

## Get an API key

Register at **https://app.agentgo.live/** — free credits included, no credit card required.

## Install

```bash
npm install playwright@1.51.0
```

## Quick start

```typescript
import { chromium } from "playwright"; // must be playwright@1.51.0

const options = { _apikey: process.env.AGENTGO_API_KEY };
const serverUrl = `wss://app.browsers.live?launch-options=${encodeURIComponent(JSON.stringify(options))}`;

const browser = await chromium.connect(serverUrl);
const page = await browser.newPage();

await page.goto("https://example.com");
console.log(await page.title());

await browser.close();
```

## Skill for AI agents

The `skills/agentgo-browser/` directory contains a skill compatible with Claude Code and other AI coding agents.

### Install skill (Claude Code)

```bash
cp -r skills/agentgo-browser .claude/skills/agentgo-browser
```

Once installed, your AI agent will automatically use AgentGo's cloud browsers for browser automation tasks.

## Contents

```
skills/agentgo-browser/
├── SKILL.md                      # main skill entry point
└── references/
    ├── connection.md             # connection & auth details
    ├── session-management.md    # managing browser sessions
    └── running-code.md          # playwright@1.51.0 API examples
```
