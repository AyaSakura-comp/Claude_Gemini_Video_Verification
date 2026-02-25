---
name: compare-before-after-with-video
description: Use when you need to compare two screen recordings (before and after a change) to identify visual differences, regressions, or improvements. Calls Gemini CLI to analyze both videos and report differences in detail.
---

# Compare Before/After Videos with Gemini

## Overview

Compare two screen recordings side-by-side using Gemini's visual analysis. Use this when you have a "before" and "after" video and need to understand what changed between them.

## When to Use

- After implementing a UI change and wanting to verify what visually changed
- When debugging a regression and comparing behavior before/after a fix
- When reviewing a PR's visual impact by comparing recordings
- When you need a detailed diff of UI behavior across two recordings

## Prerequisites

- Gemini CLI installed (`which gemini`)
- Authenticated via `gemini auth` (OAuth personal account works)

## How to Use

### Step 1: Identify both videos

You need:
- **Before video path**: Absolute path to the "before" recording
- **After video path**: Absolute path to the "after" recording

### Step 2: Call Gemini CLI

Use the `@` file attachment syntax to pass both videos. Run via Bash:

```bash
echo "You are a QA engineer comparing two screen recordings to identify visual differences.

BEFORE video: @BEFORE_VIDEO_PATH
AFTER video: @AFTER_VIDEO_PATH

Please analyze BOTH video files directly at those exact paths.

First, describe EXACTLY what you observe in each video step by step in chronological order:

## BEFORE video observations:
[Describe every visible element, interaction, timing, and behavior]

## AFTER video observations:
[Describe every visible element, interaction, timing, and behavior]

Then produce a detailed DIFFERENCES report:

## Differences
- List every change you noticed, no matter how small
- For each difference: describe what it was before vs what it is after
- Note any timing changes, animation differences, layout shifts, color/style changes
- Note any elements that appeared, disappeared, moved, or changed state
- Note any behavioral changes (e.g., button response, transitions, error states)

Be exhaustive — the detailed difference report is the most important part.

DO nothing after you finish the video analysis.
" | gemini -m gemini-3-pro-preview -y -e ""
```

**Critical:** Do NOT use `-p` flag — it fails with video files.

### Step 3: Handle errors and quota fallback

If the command exits non-zero, inspect the output for the error type:

**Retry first (all non-fatal errors):**
Before switching models or giving up, retry the same command up to **5 times** with the same model. Wait 3 seconds between retries. Only proceed below if all 5 retries fail.

**Quota / rate limit errors** (output contains `quota`, `429`, `rate limit`, or `resource exhausted`):
After 5 retries fail, try the next model in this fallback chain, in order:
1. `gemini-3-pro-preview` (primary)
2. `gemini-2.5-pro`
3. `gemini-3-flash-preview`
4. `gemini-2.5-flash`

For each fallback model, retry up to 5 times before moving to the next. Stop at the first model that succeeds.

**Other errors:**
- Video file not found → check both paths exist before retrying
- Gemini CLI not installed → `npm install -g @google/gemini-cli`
- Not authenticated → run `gemini` interactively to authenticate
- Any other persistent error after 5 retries → report to user with full error output

### Step 4: Relay the result

Relay Gemini's full analysis to the user:
- The per-video observations
- The complete differences report — relay ALL details, no summarizing

## Notes

- `-y` enables YOLO mode (auto-approves tool calls — Gemini may extract frames using Python/cv2 if available)
- `-e ""` disables Gemini CLI extensions to prevent tool conflicts
- The `@` syntax is Gemini CLI's file attachment — both videos are sent as inline multimodal data
- Model fallback chain (quota exhausted): `gemini-3-pro-preview` → `gemini-2.5-pro` → `gemini-3-flash-preview` → `gemini-2.5-flash`
