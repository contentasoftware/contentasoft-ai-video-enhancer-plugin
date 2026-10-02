# AI Video Enhancer Studio for Claude

Use **AI Video Enhancer Studio** from Claude on your Windows PC: upscaling, stabilizing, denoising and frame-rate conversion of videos on an NVIDIA RTX GPU. Claude calls the app's tools on files on
your computer; nothing is uploaded to Anthropic or to ContentaSoft to do the work.

## What you need

- Windows 10 or 11 with **AI Video Enhancer Studio** installed. It has a free trial: https://www.contenta-software.com/aivideoenhancer/download.php
  (if it is not installed yet, the plugin's `get_started` tool gives Claude the download link and the steps).
- Node.js 18 or later, which runs the small launcher in `server/index.js` (MIT, no dependencies, the same code
  as the npm package `@contentasoft/ai-video-enhancer-mcp`; source: https://github.com/contentasoftware/mcp-launcher). The launcher
  starts the app's own MCP server (`aivideoenhancer serve`) and passes its messages through; it sends nothing over the
  network itself.
- Claude Code or Cowork on that computer. Chat on claude.ai cannot start local programs, so it does not load
  this plugin's tools.

## What is included

- **MCP server** `ai-video-enhancer` with the tools `enhance_video`, `analyze_video`, `get_status`, `list_presets`. Each tool
  says whether it only reads files, writes new files, may overwrite files, or uses the internet.
- **Skill** `contenta-video-enhancer`: how and when Claude should use those tools.

## What runs and what is sent

- The plugin runs `node server/index.js` from the plugin folder; nothing is downloaded. The launcher looks for
  `aivideoenhancer.exe` in the app's install folder (`%LOCALAPPDATA%\Programs\AIVideoEnhancerStudio`), on your `PATH`
  or in `Program Files`, and runs `aivideoenhancer serve`. If the app is missing it serves one tool, `get_started`.
- The skill may only run the app's own command-line tool (`allowed-tools: Bash(aivideoenhancer:*)`).
- The app processes local files only. It sends anonymous usage telemetry (which tools ran, which MCP client
  connected, trial state) to ContentaSoft; turn it off in the app's settings. Privacy policy: https://www.contenta-software.com/aivideoenhancer/privacy.php
- During the trial some outputs carry a watermark or other limits; the tool results say so and link to the
  license.

## License

This plugin and the launcher are MIT-licensed (see LICENSE). AI Video Enhancer Studio itself is commercial software by
ContentaSoft AB.
