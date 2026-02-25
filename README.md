# Claude Gemini Video Verification Skills

[Claude Code](https://claude.ai/claude-code) skills that use Gemini CLI to analyze screen recordings — either verifying a single recording against expected behavior, or comparing two recordings to identify differences.

## Skills

### `verify-video`

Give Claude a video file path and a description of expected behavior — Claude will call Gemini to analyze the recording and return a detailed **PASS / FAIL** verdict with reasoning.

### `compare-before-after-with-video`

Give Claude two video file paths (before and after a change) — Claude will call Gemini to analyze both recordings and produce an exhaustive **differences report** covering layout, timing, animations, element states, and behavioral changes.

## Installation

Copy the skills into your Claude Code personal skills directory:

```bash
# verify-video
mkdir -p ~/.claude/skills/verify-video
cp skills/verify-video/SKILL.md ~/.claude/skills/verify-video/SKILL.md

# compare-before-after-with-video
mkdir -p ~/.claude/skills/compare-before-after-with-video
cp skills/compare-before-after-with-video/SKILL.md ~/.claude/skills/compare-before-after-with-video/SKILL.md
```

Claude Code loads skills from `~/.claude/skills/` automatically on startup.

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) installed
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) installed and authenticated

```bash
npm install -g @google/gemini-cli
gemini  # authenticate on first run
```

## Usage

### Verify a single recording

> "The Playwright test recorded a video at `/path/to/video.webm`. Verify that the login flow works — user should type credentials, click Login, and land on the dashboard."

Claude invokes `verify-video`, calls Gemini, and reports back the full PASS/FAIL verdict.

### Compare before and after

> "Compare `/path/to/before.webm` and `/path/to/after.webm` and tell me what changed."

Claude invokes `compare-before-after-with-video`, sends both videos to Gemini, and reports every visual and behavioral difference found.

## How it works

```bash
echo "...@/absolute/path/to/video.webm..." | gemini -m gemini-3-pro-preview -y -e ""
```

Key details:
- Uses `@file` attachment syntax via stdin pipe (the only reliable way to pass video in headless mode — `-p` flag fails with video files)
- Run Gemini from the same directory as the video files so the workspace restriction doesn't block access
- `-y` auto-approves Gemini tool calls (frame extraction via Python/cv2 if available)
- `-e ""` disables extensions to prevent tool conflicts

## Model fallback chain

If a model hits quota limits, the skills automatically fall back:

`gemini-3-pro-preview` → `gemini-2.5-pro` → `gemini-3-flash-preview` → `gemini-2.5-flash`

Each model is retried up to 5 times before moving to the next.

## Example output

**verify-video (FAIL case):**
```
VERDICT: FAIL

Detailed Variance Report:
1. Divergence Point: 00:00
2. Expected: Login form with email/password fields
   Actual: Documentation page with no form elements
3. Missing: Login button, loading spinner, dashboard redirect
Root cause: Test may have navigated to wrong URL before recording started
```

**compare-before-after-with-video:**
```
## Differences
- Final score: 13 (before) → 2 (after)
- Video duration: 14 sec (before) → 48 sec (after)
- Game-over behavior: cuts off immediately (before) → lingers 42 sec on failure screen (after)
- Background animations continue running during failure state (observable only in after video)
- No changes to visual assets, colors, UI styling, or layout
```
