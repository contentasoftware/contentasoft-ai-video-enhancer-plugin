# AI Video Enhancer Studio plugin

Use **AI Video Enhancer Studio** from your AI agent on your Windows PC: upscaling, stabilizing, denoising and frame-rate conversion of videos on an NVIDIA RTX GPU. You ask in plain words and the
agent does the work on files on your computer. It works with any agent that runs local MCP servers: Cursor,
Claude Code, Codex, VS Code, Windsurf and others.

## Ask your agent

- "Clean up and upscale the old family videos I copied from VHS tapes."
- "Steady this shaky GoPro clip."
- "Make a before-and-after video so I can see the difference."

AI Video Enhancer Studio also works without an agent: open it from the Start menu and work in its own window.

## Price and trial

- $129, a one-time purchase. No subscription needed. Prices: https://www.contenta-software.com/aivideoenhancer/buy.php
- The free trial has no end date: the first 5 full exports per PC are clean and full resolution, later ones are watermarked and capped at 1280x720; 10-second clips (`--clip`) stay free. `aivideoenhancer status` shows how many full exports are left. Nothing stops working; a licence removes the limits.

## Install

**Claude Code** (Windows): two commands, typed inside Claude Code. This repository is its own one-plugin
marketplace, named after the repository:

```
/plugin marketplace add contentasoftware/contentasoft-ai-video-enhancer-plugin
/plugin install contentasoft-ai-video-enhancer@contentasoft-ai-video-enhancer-plugin
```

From a terminal the same is `claude plugin marketplace add contentasoftware/contentasoft-ai-video-enhancer-plugin` and `claude plugin install contentasoft-ai-video-enhancer@contentasoft-ai-video-enhancer-plugin`.
Cowork and the Claude.ai directory: install **AI Video Enhancer Studio** from the directory once it is listed.

**Cursor, Claude Desktop, VS Code, Codex and other MCP clients** (`mcpServers` JSON):
`"ai-video-enhancer": { "command": "cmd", "args": ["/c", "npx", "-y", "@contentasoft/ai-video-enhancer-mcp"] }` (needs Node.js 18+), or,
once the app is installed, `"ai-video-enhancer": { "command": "aivideoenhancer", "args": ["serve"] }` with no Node.js at all.
Client-by-client instructions: https://www.npmjs.com/package/@contentasoft/ai-video-enhancer-mcp

## What you need

- Windows 10 or 11 with **AI Video Enhancer Studio** installed. It has a free trial: https://www.contenta-software.com/aivideoenhancer/download.php
  (if it is not installed yet, the `get_started` tool gives the agent the download link and the steps).
- Nothing else for the Claude Code plugin: its launcher is a Windows batch file (`server/launch.cmd`) that starts the
  app's own MCP server. Node.js is not required; only the `npx` route for other clients needs it.
- An agent that runs on that computer. Browser chat apps cannot start local programs, so they cannot use these
  tools.

## What is included

- **MCP server** `ai-video-enhancer` with the tools `enhance_video`, `analyze_video`, `get_status`, `list_presets`. Each tool
  says whether it only reads files, writes new files, may overwrite files, or uses the internet.
- **Skill** `contenta-video-enhancer`: how and when the agent should use those tools and the `aivideoenhancer` command line.

## For developers

- Windows 10 and 11, 64-bit only. There is no macOS or Linux version.
- `aivideoenhancer` never waits for input or shows a dialog, so it runs unattended from Task Scheduler or a script.
  A nightly run with `--skip-existing` enhances only the videos that have no output yet:

  ```
  schtasks /create /tn "AI Video Enhancer Studio nightly" /sc daily /st 02:00 /tr "aivideoenhancer enhance D:\Tapes -o D:\Restored --preset old_video_restoration --skip-existing"
  ```
- Results are plain text on stdout and log lines go to stderr; a failure prints one `Error: ...` line on stderr. Only `remix suggest` has `--json`, so check the exit code.
- Exit codes: 0 done · 1 general error · 2 invalid arguments · 3 daily clip limit reached · 4 enhancement failed or timed out · 5 file not found · 6 tools missing.
- The full command reference is the skill: `skills/contenta-video-enhancer/SKILL.md`.

## Privacy

- Videos are enhanced on this PC, on its own GPU.
- The app sends usage statistics (which tools ran, which MCP client connected, trial state) and crash reports to
  ContentaSoft, tagged with a hashed machine ID. They never contain your files; a crash report can contain a file
  name, but folder paths are removed. To turn them off, open the app and
  go to Settings, Privacy: Send usage statistics and crash reports. The same switch covers the command line and the
  MCP server. On a PC where nobody opens the app, set `"AllowTelemetry": false` in `%APPDATA%\AIVideoEnhancerStudio\settings.json`.
- Privacy policy, with the full list of what is sent: https://www.contenta-software.com/aivideoenhancer/privacy.php

## What runs

- As a Claude plugin it runs `server/launch.cmd` from the plugin folder, which looks for `aivideoenhancer.exe` in the app's
  install folder (`%LOCALAPPDATA%\Programs\AIVideoEnhancerStudio`), on your `PATH` or in `Program Files`, and runs
  `aivideoenhancer serve`. Nothing is downloaded. If the app is missing, the bundled Node launcher (`server/index.js`, the same
  bytes as the npm package `@contentasoft/ai-video-enhancer-mcp`, MIT, no dependencies; source: https://github.com/contentasoftware/mcp-launcher)
  or, without Node.js, the Windows PowerShell stub `server/stub.ps1` serves one tool, `get_started`. Neither sends
  anything over the network.
- The skill may only run the app's own command-line tool (`allowed-tools: Bash(aivideoenhancer:*)`).

## License

This plugin and the launcher are MIT-licensed (see LICENSE). AI Video Enhancer Studio itself is commercial software by
ContentaSoft AB.
