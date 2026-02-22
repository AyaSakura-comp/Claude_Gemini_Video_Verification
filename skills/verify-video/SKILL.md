---
name: verify-video
description: Use when you need to verify whether a screen recording (video) matches expected UI behavior. Calls Gemini CLI to analyze the video and provides detailed pass/fail reasoning. Typically used after Playwright tests produce screen recordings.
---

# Verify Video with Gemini

## Overview

Verify that a screen recording matches expected behavior by sending it to Gemini for visual analysis. Use this whenever you have a video file (`.webm`, `.mp4`) and need to confirm the UI behaved as expected.

## When to Use

- After a Playwright test produces a screen recording
- When you need to visually verify UI behavior that can't be asserted programmatically
- When a test passes functionally but you need to confirm the visual experience
- When debugging a UI issue and you have a recording to analyze

## Prerequisites

- Gemini CLI installed (`which gemini`)
- Authenticated via `gemini auth` (OAuth personal account works)

## How to Use

### Step 1: Identify the video and expectation

You need:
- **Video path**: Absolute path to the video file
- **Expected behavior**: A clear description of what the video should show

### Step 2: Call Gemini CLI

Use the `@` file attachment syntax to pass the video. Run via Bash:

```bash
echo "You are a QA engineer reviewing a screen recording from an automated UI test.

Describe EXACTLY what you observe in this video @VIDEO_PATH step by step in chronological order.

Then compare against this EXPECTED BEHAVIOR:
EXPECTATION_DESCRIPTION

Give a verdict: PASS or FAIL.

If FAIL, explain exhaustively:
- What specifically was expected vs what actually happened
- At what point in the video the behavior diverged from expectations
- What the actual behavior was instead
- Any visual anomalies, timing issues, missing elements, or incorrect states you noticed
- Possible root causes based on what you observed

The detailed reasoning is the most important part — describe every discrepancy, no matter how small." | gemini -m gemini-3-flash-preview -y -e ""
```

**Critical:** Use `@/absolute/path` (with `@` prefix). Do NOT use `-p` flag — it fails with video files.

### Step 3: Handle errors and quota fallback

If the command exits non-zero, inspect the output for the error type and fallback accordingly:

**Quota / rate limit errors** (output contains `quota`, `429`, `rate limit`, or `resource exhausted`):
Try the next model in this fallback chain, in order:
1. `gemini-3-flash-preview` (primary)
2. `gemini-3-pro-preview`
3. `gemini-2.5-pro`
4. `gemini-2.5-flash`

Replace `-m gemini-3-flash-preview` with the next model and retry the exact same command. Stop at the first model that succeeds.

**Other errors:**
- Video file not found → check path exists before retrying
- Gemini CLI not installed → `npm install -g @google/gemini-cli`
- Not authenticated → run `gemini` interactively to authenticate
- Any other error → report to user with full error output, do not retry

### Step 4: Interpret the result

- **PASS**: Report success to the user with Gemini's observations as confirmation
- **FAIL**: Report the failure with Gemini's full detailed reasoning — relay ALL mismatch details to the user
- **All models exhausted**: Report that all quota fallbacks failed and ask the user to check their Gemini quota

## Example

```bash
echo "You are a QA engineer reviewing a screen recording from an automated UI test.

Describe EXACTLY what you observe in this video @/home/user/test-results/login-test/video.webm step by step in chronological order.

Then compare against this EXPECTED BEHAVIOR:
User types email and password, clicks Login, and is redirected to the dashboard. The dashboard shows a welcome message with the user's name.

Give a verdict: PASS or FAIL.

If FAIL, explain exhaustively:
- What specifically was expected vs what actually happened
- At what point in the video the behavior diverged from expectations
- What the actual behavior was instead
- Any visual anomalies, timing issues, missing elements, or incorrect states you noticed
- Possible root causes based on what you observed

The detailed reasoning is the most important part — describe every discrepancy, no matter how small." | gemini -m gemini-3-flash-preview -y -e ""
```

## Notes

- `-y` enables YOLO mode (auto-approves tool calls — Gemini may extract frames using Python/cv2 if available)
- `-e ""` disables Gemini CLI extensions to prevent tool conflicts
- The `@` syntax is Gemini CLI's file attachment — the model receives the video as inline multimodal data
- Model fallback chain (quota exhausted): `gemini-3-flash-preview` → `gemini-3-pro-preview` → `gemini-2.5-pro` → `gemini-2.5-flash`
