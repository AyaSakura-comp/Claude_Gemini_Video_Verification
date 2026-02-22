# Claude Gemini Video Verification Skill

A [Claude Code](https://claude.ai/claude-code) skill that uses Gemini CLI to verify screen recordings from automated UI tests (e.g. Playwright) against expected behavior.

## What it does

Give Claude a video file path and a description of expected behavior — Claude will call Gemini to analyze the recording frame-by-frame and return a detailed **PASS / FAIL** verdict with reasoning.

## Installation

Copy the skill into your Claude Code personal skills directory:

```bash
mkdir -p ~/.claude/skills/verify-video
cp skills/verify-video/SKILL.md ~/.claude/skills/verify-video/SKILL.md
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

Just describe what you want verified naturally in Claude — no slash command needed:

> "The Playwright test recorded a video at `/path/to/video.webm`. Verify that the login flow works — user should type credentials, click Login, and land on the dashboard."

Claude will automatically invoke this skill, call Gemini, and report back the full verdict.

## How it works

```bash
echo "...@/absolute/path/to/video.webm..." | gemini -m gemini-3-flash-preview -y -e ""
```

Key details:
- Uses `@file` attachment syntax via stdin pipe (the only reliable way to pass video in headless mode — `-p` flag fails with video files)
- `-y` auto-approves Gemini tool calls (frame extraction via Python/cv2 if available)
- `-e ""` disables extensions to prevent tool conflicts

## Model fallback chain

If a model hits quota limits, the skill automatically falls back:

`gemini-3-flash-preview` → `gemini-3-pro-preview` → `gemini-2.5-pro` → `gemini-2.5-flash`

## Example output (FAIL case)

```
VERDICT: FAIL

Detailed Variance Report:
1. Divergence Point: 00:00
2. Expected: Login form with email/password fields
   Actual: Documentation page with no form elements
3. Missing: Login button, loading spinner, dashboard redirect
Root cause: Test may have navigated to wrong URL before recording started
```
