# Image Spec Check

[简体中文](README_CN.md)

A skill for Claude Code and Codex. It keeps image requirements in a checklist file and checks each generated image against that list before the user sees it.

Use it when an image has specific requirements, such as appearance, costume, accessories, which hand holds what, left and right, prop counts, lighting or camera position. It also fits revising one image over several rounds, and making first/last frames or keyframes for video.

## Workflow

1. Write the user's requirements to `清单.md` as items you can judge by looking at the image. Each item records its source (user or AI guess) and which part of the image to check.
2. Keep only the current version. When the user corrects or withdraws something, rewrite or delete that item before the next generation, so old wording can't leak back into prompts.
3. State left and right both as the character's own and as seen in the frame. Recompute both when the camera moves.
4. Write style terms in three parts: what must be there, what must not be there, and where to look.
5. For local changes, edit the current base image. Regenerate when the camera, framing or pose changes a lot, or when the same fix fails twice. The base image then serves only as a reference.
6. After each image, a checker that hasn't seen the prompt crops and enlarges each area, then marks every item pass, fail or can't tell. A pass must name the place and what was seen there.
7. If the same item fails twice in a row, stop and hand it to the user instead of rewording the prompt.

Natural motion, fitting expressions and overall look are for the user to judge. The skill doesn't decide them.

## Install

Claude Code:

```bash
git clone https://github.com/chipfighter/image-spec-check ~/.claude/skills/image-spec-check
```

Codex:

```bash
git clone https://github.com/chipfighter/image-spec-check ~/.codex/skills/image-spec-check
```

Cropping uses `ffmpeg` and `ffprobe`. The skill text is in Chinese.

## Test notes

In October 2026 the skill ran in Codex (gpt-6.1-sol) through five rounds of revisions to one illustration of a Song-dynasty woman.

- **Worked:** changed requirements left no stale items. Left/right mapping was right for both front and back camera angles. Local changes were made as edits, and the camera change as a regeneration. Unclear areas and failed generations were reported as such.
- **Missed:** the checker passed the wrong art style four rounds in a row, until the user pointed it out. Two rules were added after that and have not been retested: style items list "must have / must not have / where to look", and a pass must name the place and what was seen there.
- **Not exercised:** the stop-after-two-failures rule, and checks that first and last frames connect.
- **Cost:** each round took about 10–20 minutes, partly from network retries. The independent checker adds time.

Status: experimental.
