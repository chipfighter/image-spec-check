# Image Spec Check

[简体中文](README_CN.md)

A skill for Claude Code and Codex. It keeps image requirements in a checklist file and checks each generated or existing image against that list before the user sees it.

Use it when an image has specific requirements, such as appearance, costume, accessories, which hand holds what, left and right, prop counts, lighting, camera position or art style. It also fits revising one image over several rounds, checking an existing image or a set of images for consistency, and making first/last frames for video.

## Workflow

1. Write the user's requirements to `清单.md` as items you can judge by looking at the image. Each item records its source (user or AI guess) and which part of the image to check.
2. Keep only the current version. When the user corrects or withdraws something, rewrite or delete that item before the next generation, so old wording can't leak back into prompts. Every problem the user points out becomes an item that is checked from then on, so the same mistake doesn't reach the user twice.
3. State left and right both as the character's own and as seen in the frame. Recompute both when the camera moves.
4. Write style terms in three parts: what must be there, what must not be there, and where to look.
5. For local changes, edit the current base image. Regenerate when the camera, framing or pose changes a lot, or when the same fix fails twice. The base image then serves only as a reference.
6. After each image, a checker that hasn't seen the prompt crops each area at its original resolution and marks every item pass, fail or can't tell. A pass must name the place and what was seen there. Crops are not upscaled, because upscaling adds blur and tokens, not detail.
7. Check every new image automatically, without being asked. Only when the user says an image is a draft or needs no check, take a quick look at the error-prone items: hands, held objects, left/right and counts. After a local edit, check the changed items and the error-prone ones in detail, and compare the rest against the previous version. A regenerated image gets every item checked.
8. The checker can run as a subagent at low reasoning effort, since the job is mostly looking. A model one tier below the main one is fine, as long as it can still see small details.
9. If the same item fails twice in a row, stop and hand it to the user instead of rewording the prompt.
10. Save tokens without asking anything of the user. Leave viewing the crops to the subagent, so images don't pile up in the main conversation. Merge steps where possible, and append to the check log instead of re-reading it.

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
