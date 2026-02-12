---
name: video-frames
description: Extract frames or short clips from videos using ffmpeg.
homepage: https://ffmpeg.org
metadata:
  {
    "openclaw":
      {
        "emoji": "🎞️",
        "requires": { "bins": ["ffmpeg"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "ffmpeg",
              "bins": ["ffmpeg"],
              "label": "Install ffmpeg (brew)",
            },
          ],
      },
  }
---

# Video Frames (ffmpeg)

Extract a single frame from a video, or create quick thumbnails for inspection.

## When to Use This Skill

**Use when:**

- Extracting a single frame from a video file
- Creating thumbnails for video inspection
- Need a frame at a specific timestamp

**Don't use when:**

- Need video editing (cutting, merging) → use ffmpeg directly
- Need video transcription → use `openai-whisper` or `openai-whisper-api`
- Need to record video → use other tools

**Success Criteria:**

- Frame extracted at correct timestamp
- Output image is viewable (JPG or PNG)

## Quick start

First frame:

```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --out /tmp/frame.jpg
```

At a timestamp:

```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --time 00:00:10 --out /tmp/frame-10s.jpg
```

## Notes

- Prefer `--time` for "what is happening around here?".
- Use a `.jpg` for quick share; use `.png` for crisp UI frames.

### Common Pitfalls

**What NOT to do:**

- Using wrong timestamp format: use HH:MM:SS format
- Choosing PNG for quick sharing: JPG is faster/smaller for previews
- Not checking if input video exists before running: fails silently
