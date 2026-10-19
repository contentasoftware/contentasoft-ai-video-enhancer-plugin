---
name: contenta-video-enhancer
description: Enhance videos with the AI Video Enhancer Studio CLI (aivideoenhancer) - NVIDIA Video Super Resolution upscaling, RIFE frame interpolation to 30/60 fps, stabilization, rolling-shutter correction, denoise, deinterlace, sharpen, before/after comparison videos, still-frame extraction and highlight-reel remixes. Use when the user asks to upscale or restore a video, fix shaky or noisy footage, raise the frame rate, grab stills or thumbnails from a video, or cut a highlight reel.
allowed-tools: Bash(aivideoenhancer *) PowerShell(aivideoenhancer *) Bash(aivideoenhancer.exe *) PowerShell(aivideoenhancer.exe *) mcp__plugin_contentasoft-ai-video-enhancer_ai-video-enhancer
---

# AI Video Enhancer Studio

Use the `aivideoenhancer` CLI (AI Video Enhancer Studio 2026.7.22+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\AIVideoEnhancerStudio\aivideoenhancer.exe`, on the user PATH. Check with `aivideoenhancer --version` (it prints the build id after a `+`). Full reference: https://github.com/brunojanvier77/ContentaSoft/blob/main/ai-video-enhancer/cli.md

Every command accepts `--json` (one JSON object on stdout), `--plain`, `--quiet`, `-v/--verbose` and `--progress`. Log lines go to stderr (drop them with `2>/dev/null`; in PowerShell `2>$null`); without `--json` every failure prints one `Error: ...` line on stderr before the non-zero exit, so keep stderr when a command fails. Numbers use a `.` decimal point whatever the machine locale. Relative and full paths both work. `aivideoenhancer register <email> <key>` registers a license key.

## Running commands

- **MCP first:** when this app's MCP tools are connected (`enhance_video`, `get_file_info`, `get_status` ...), use them for what they cover and the command line for the rest. They take paths and options as named fields, so there is no quoting or wildcard to get wrong. Give them full paths.
- **One command per call:** run one `aivideoenhancer` command at a time, with no `cd`, `&&`, `;`, pipes, loops, variables or `$(...)`. Anything else on the line can stop the run to ask the user for approval, and the CLI already handles folders and reports its results (below).
- **Let the CLI do the folder work:** `aivideoenhancer batch <folder>` enhances every video in it (`--include "*.mp4"` picks the files, `--skip-existing` makes a re-run do only what is missing), and `-o` creates missing folders.
- **Read the results, don't measure them:** every `--json` result lists the written files in `outputs`, and a `batch` summary has one `files` row per video. For an output's size, resolution, frame rate and duration, run `aivideoenhancer info <output> --json`. No `stat` or `Get-Item` wrapper is needed. To see what is in a folder, use the agent's own file tools.

## First steps

```bash
aivideoenhancer status --json    # GPU, VRAM, Vulkan, encoders, AI upscale yes/NO/unknown, license block
aivideoenhancer info <file> --json   # resolution, fps, duration, codec, audio, HDR
aivideoenhancer presets
aivideoenhancer formats
```

VSR upscaling needs an NVIDIA RTX GPU; `status` prints `AI upscale: NO` on a machine without one, and x2/x3/x4 then use the standard upscaler (Lanczos and sharpening on the CPU, not AI; `--upscale-engine auto|rtx|standard`, default `auto`). Interpolation needs a Vulkan GPU; `Vulkan: Yes` in `status` means the Vulkan runtime is installed, which is necessary but does not prove the device can run it. Stabilize, rolling shutter, denoise, deinterlace and sharpen need no GPU.

## enhance, batch, watch

```bash
aivideoenhancer enhance <file-or-folder> -o <output-folder> [--preset ID] [options]
aivideoenhancer batch <folder> -o <output-folder> [--include "*.mp4"] [--exclude ...] [-r] [-w 1] [--report r.csv] [enhance options]
aivideoenhancer watch <folder> -o <output-folder> [--interval 10] [--include ...] [enhance options]   # until Ctrl+C
```

- `--upscale off|enhance|x2|x3|x4` (`enhance` = same-resolution cleanup)
- `--interpolate off|30|60` (target fps; never lowers the rate)
- `--stabilize`, `--rolling-shutter`, `--denoise`, `--sharpen`: `off|light|medium|strong`; `--denoise-method nlmeans|hqdn3d|chroma`
- `--deinterlace off|yadif|yadifbob|bwdif|bwdifbob`. With `--preset`, interlaced sources (DV, DVD, TV capture) get `yadif` automatically; `--deinterlace off` keeps it off
- `--codec h264|h265|av1|vp9` (default h265), `--crf 1-51` (anything else exits 2)
- `--skip-existing` (skip videos whose output exists) or `--overwrite`, `--temp-dir <dir>`, `--frame-batch 0` (auto)
- `--compare [--compare-labels "BEFORE|AFTER"] [--compare-layout horizontal|vertical] [--compare-seconds 30]`
- `--clip <start>` (`enhance` only; seconds or `m:ss`): one 10-second segment of ONE video, clean and full resolution, named `<name>-clip`. On the trial it does not use a free export (max 30 s of one video per day)

Output: `<name>_enhanced.mp4` (plus a compare video with `--compare`) in the output folder.

| Preset | Pipeline | Use for |
|--------|----------|---------|
| `tape_restoration` | full-motion deinterlace, colour noise and hum removal, 4:3; CPU only | VHS, Video8, Hi8, MiniDV captures |
| `old_video_restoration` | stabilize medium, denoise strong, upscale x2, audio cleanup | old DVD, camcorder |
| `surveillance_enhancement` | denoise medium, upscale x4 | security cameras |
| `content_creation` | denoise light, upscale x2, 30 fps | social media |
| `drone_action_cam` | stabilize strong, denoise light, upscale x2, 30 fps | drone, action cam, handheld |
| `animation_anime` | denoise light, upscale x4 | animation |
| `video_archival` | stabilize light, denoise medium, upscale x2 | archiving |
| `enhance_cleanup` | denoise light, same-resolution cleanup | compression artifacts |
| `smooth_motion` | denoise light, 30 fps | choppy low-fps clips |

## extract-frames

```bash
aivideoenhancer extract-frames <file-or-folder> -o <dir> [--preset storyboard|thumbnails|social|scenes|custom] [--count N] [--timestamps 2,5,10] [--upscale off|enhance|x2|x3|x4] [--format png|jpg|webp] [--quality 90] [--sharpest]
```

Frames always go into `<dir>\<name>_frames\` (one subfolder per video).

## remix

```bash
aivideoenhancer remix suggest <file> [--duration 30] [--clips N] [--min-clip 3] [--max-clip 6] [--threshold 10] [--audio-weight 0.4] [--json]
aivideoenhancer remix render <file> --clips "1-4,8-11" -o <reel.mp4> [--aspect original|9:16|1:1|16:9] [--music <file>] [--music-volume 30] [--duck] [--mute] [--max-duration 30]
```

`remix suggest` prints a ready-to-run `remix render` command. `remix render` writes one H.264 MP4.

## Examples

```bash
aivideoenhancer enhance ./clip.mp4 -o ./enhanced --upscale x2 --denoise light --codec h265 --crf 18
aivideoenhancer enhance ./clip.mp4 -o ./smooth --interpolate 60 --compare
aivideoenhancer batch ./tapes -o ./restored --preset tape_restoration --skip-existing --json --progress
aivideoenhancer enhance ./tapes/tape1.dv -o ./cleanup --preset enhance_cleanup --deinterlace off
aivideoenhancer enhance ./clip.mp4 -o ./clips --upscale x2 --denoise light --clip 0:42
aivideoenhancer watch ./incoming -o ./enhanced --preset enhance_cleanup
aivideoenhancer extract-frames video.mp4 -o ./frames --preset thumbnails
aivideoenhancer extract-frames video.mp4 -o ./frames_up --timestamps 2,5,10 --upscale x2
aivideoenhancer remix render video.mp4 --clips "1-4,8-11,14-17" -o ./reel.mp4 --aspect 9:16 --music music.mp3 --duck
```

## JSON output

With `--json` stdout holds exactly one JSON object, also for errors. Read `success` (boolean), `exitCode`, `error` (one sentence or `null`), `errors[]` (one entry per failed video), `warnings[]` and `outputs[]` (the files written; report these paths). Trial keys appear when they apply: `trialWatermarked`, `trialNotice`, `freeFilesRemaining`, `buyUrl`. With `--progress`, long commands first print `{"type":"progress","file":...,"completed":N,"total":T}` lines; skip them and parse the last line.

## Exit codes

Same table in all four ContentaSoft CLIs: 0 OK · 1 unexpected error (a bug or a broken install) · 2 invalid arguments (including a bad enumerated value, a `--crf` outside 1-51 and a bad `--clip`; nothing was done) · 3 input not found or unreadable (also a folder with no videos, and `info` on a file that is not a video) · 4 partial failure (a batch where some videos failed) · 5 all failed (a failed `enhance`, also a job that stalls: it stops with `Timed out: ...`, writes no output and can be run again) · 6 license or trial refused (also the daily `--clip` limit for this video, and a key the server rejected) · 130 cancelled.

## Guidelines

- Check `status --json` before offering upscaling (over MCP: `get_status`, field `upscale.available`); without an NVIDIA RTX GPU, say that x2/x3/x4 will use the standard (non-AI) upscaler, and offer denoise, stabilize, sharpen and interpolation.
- MCP `enhance_video` also takes `clip` (start in seconds or m:ss; the free 10-second clean clip, same rules as `--clip`), `compare` (bool), `compare_labels` ("Before|After") and `compare_layout` (`horizontal|vertical`); the result lists `clipPath` and `comparePaths`. On the trial the result carries `trialWatermarked` and a `trialNotice`. `get_file_info` matches `info` (`analyze_video` is its old name, still accepted).
- x4 upscaling of long videos takes a long time; say so before starting, and try a short clip first.
- Interlaced sources (old DVD/camcorder footage; `info --json` and the MCP `get_file_info` tool report it) are deinterlaced with `yadif` when you use a preset. Without a preset, add `--deinterlace yadif` or `bwdif` yourself. For VHS and camcorder tapes, prefer `--preset tape_restoration`.
- Trial: no end date. This computer's first 5 full exports (lifetime, plus 10 after the newsletter confirmation in the app) are full resolution without a watermark, then output is watermarked and capped at 1280x720. A remix render and an upscaled frame extraction each use one free file. `--clip` gives free clean 10-second clips. Before a batch or a long job, run `aivideoenhancer status --json` (MCP: `get_license_status`) and read its `license` object: `state` (`Trial` or `Registered`), `freeFilesRemaining`, `nextOutput` (`clean` or `watermark_720p`) and `afterFreeFiles`. When `state` is `Trial`, tell the user before you start how many full exports are left, that later ones will be watermarked and capped at 1280x720, and that a licence removes both: <https://www.contenta-software.com/aivideoenhancer/buy.php>. Nothing stops working; `aivideoenhancer register <email> <key>` removes the limits.
