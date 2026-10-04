---
name: contenta-video-enhancer
description: Enhance videos with the AI Video Enhancer Studio CLI (aivideoenhancer) - NVIDIA Video Super Resolution upscaling, RIFE frame interpolation to 30/60 fps, stabilization, rolling-shutter correction, denoise, deinterlace, sharpen, before/after comparison videos, still-frame extraction and highlight-reel remixes. Use when the user asks to upscale or restore a video, fix shaky or noisy footage, raise the frame rate, grab stills or thumbnails from a video, or cut a highlight reel.
allowed-tools: Bash(aivideoenhancer:*)
---

# AI Video Enhancer Studio

Use the `aivideoenhancer` CLI (AI Video Enhancer Studio 2026.7.17+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\AIVideoEnhancerStudio\aivideoenhancer.exe`, on the user PATH. Check with `aivideoenhancer --version` (it prints the build id after a `+`).

Results are human-readable text on stdout; log lines go to stderr (drop them with `2>/dev/null`; in PowerShell `2>$null`), and every failure prints one `Error: ...` line on stderr before the non-zero exit, so keep stderr when a command fails. When stdout is not a terminal, progress is one line per phase (`enhance --quiet` silences it). Numbers use a `.` decimal point whatever the machine locale. Only `remix suggest` has `--json`. Relative and full paths both work. `aivideoenhancer register <email> <key>` registers a license key.

## First steps

```bash
aivideoenhancer status          # GPU, VRAM, Vulkan, encoders, AI upscale yes/NO/unknown, license
aivideoenhancer analyze <file>  # resolution, fps, duration, codec
aivideoenhancer presets
```

VSR upscaling needs an NVIDIA RTX GPU (no CPU fallback); `status` prints `AI upscale: NO` on a machine without one, and every upscale job would fail there. Interpolation needs a Vulkan GPU. Stabilize, rolling shutter, denoise, deinterlace and sharpen need no GPU.

## enhance

```bash
aivideoenhancer enhance <file-or-folder> -o <output-folder> [--preset ID] [options]
```

- `--upscale off|enhance|x2|x3|x4` (`enhance` = same-resolution cleanup)
- `--interpolate off|30|60` (target fps; never lowers the rate)
- `--stabilize`, `--rolling-shutter`, `--denoise`, `--sharpen`: `off|light|medium|strong`; `--denoise-method nlmeans|hqdn3d`
- `--deinterlace off|yadif|yadifbob|bwdif`. With `--preset`, interlaced sources (DV, DVD, TV capture) get `yadif` automatically; `--deinterlace off` keeps it off
- `--codec h264|h265|av1|vp9` (default h265), `--crf 1-51` (anything else exits 2)
- `--skip-existing` (skip videos whose output exists), `--temp-dir <dir>`, `--frame-batch 0` (auto)
- `--compare [--compare-labels "BEFORE|AFTER"] [--compare-layout horizontal|vertical]`
- `--clip <start>` (seconds or `m:ss`): one 10-second segment of ONE video, clean and full resolution, named `<name>-clip`. On the trial it does not use a free export (max 30 s of one video per day)

Output: `<name>_enhanced.mp4` (and `<name>_enhanced_compare.mp4`) in the output folder.

| Preset | Pipeline | Use for |
|--------|----------|---------|
| `old_video_restoration` | stabilize medium, denoise strong, upscale x2, audio cleanup | VHS, DVD, camcorder |
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

## Examples (verified on 2026.7.17)

```bash
aivideoenhancer enhance ./clip.mp4 -o ./enhanced --upscale x2 --denoise light --codec h265 --crf 18
aivideoenhancer enhance ./clip.mp4 -o ./smooth --interpolate 60 --compare
aivideoenhancer enhance ./tapes -o ./restored --preset old_video_restoration --skip-existing
aivideoenhancer enhance ./tapes/tape1.dv -o ./cleanup --preset enhance_cleanup --deinterlace off
aivideoenhancer enhance ./clip.mp4 -o ./clips --upscale x2 --denoise light --clip 0:42
aivideoenhancer extract-frames video.mp4 -o ./frames --preset thumbnails
aivideoenhancer extract-frames video.mp4 -o ./frames_up --timestamps 2,5,10 --upscale x2
aivideoenhancer remix suggest video.mp4 --duration 10
aivideoenhancer remix render video.mp4 --clips "1-4,8-11,14-17" -o ./reel.mp4 --aspect 9:16 --music music.mp3 --duck
```

## Exit codes

0 success · 1 general error (also `analyze` on a file that is not a video) · 2 invalid arguments (including a bad enumerated value, a `--crf` outside 1-51 and a bad `--clip`) · 3 daily `--clip` limit reached for this video · 4 enhancement failed · 5 file not found (also a folder with no videos) · 6 tools missing.

## Guidelines

- Check `status` before offering upscaling (over MCP: `get_status`, field `upscale.available`); without an NVIDIA RTX GPU, offer denoise, stabilize, sharpen and interpolation instead.
- MCP `enhance_video` also takes `clip` (start in seconds or m:ss; the free 10-second clean clip, same rules as `--clip`), `compare` (bool), `compare_labels` ("Before|After") and `compare_layout` (`horizontal|vertical`); the result lists `clipPath` and `comparePaths`. On the trial the result carries `trialWatermarked` and a `trialNotice`.
- x4 upscaling of long videos takes a long time; say so before starting, and try a short clip first.
- Interlaced sources (old DVD/camcorder footage; the MCP `analyze_video` tool reports `isInterlaced`) are deinterlaced with `yadif` when you use a preset. Without a preset, add `--deinterlace yadif` or `bwdif` yourself.
- Trial: no end date. This computer's first 5 full exports (lifetime, plus 10 after the newsletter confirmation in the app) are full resolution without a watermark, then output is watermarked and capped at 1280x720. A remix render and an upscaled frame extraction each use one free file. `--clip` gives free clean 10-second clips. Nothing stops working; `aivideoenhancer register <email> <key>` removes the limits.
