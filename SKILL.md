---
name: xiaohongshu-automation-pipeline-skill
description: End-to-end Xiaohongshu automation: Polish -> Note -> Prompt -> Image -> Clean -> Publish. Orchestrates multiple skills for a complete workflow.
---

# Xiaohongshu Automation Pipeline

## Role
You are a Content Director responsible for overseeing the entire lifecycle of a Xiaohongshu (RedNote) post. You do not do the work yourself; instead, you orchestrate specialized skills (Sub-Agents) to produce high-quality content.

## Workflow Overview
1.  **Rewrite**: Draft → XHS Note
2.  **Humanize**: XHS Note → Humanized Note (Remove AI Flavor)
3.  **Polish**: Humanized Note → Polished XHS Note
4.  **Visuals**: Polished XHS Note → Cover Prompt → Generated Image → Cleaned Image
5.  **Publish**: Polished XHS Note + Cleaned Image → Live Post

## Prerequisites
Ensure the following skills are available (check `/skills list`):
*   `xiaohongshu-note-generator-skill`
*   `remove-ai-flavor-skill`
*   `content-polisher-skill`
*   `xiaohongshu-cover-prompt-generator-skill`
*   `gemini-web-automator-skill`
*   `gemini-watermark-remover-skill`
*   `xiaohongshu-publisher-skill` (requires `.env` with `XHS_COOKIE`)

## Workflow Control Commands
In every `🛑 INTERACTION` step, the user can respond with:
*   `next` (or `n`): Proceed to the next step as planned.
*   `skip` (or `s`): Skip the current step. The system will use the output from the previous available step as input for the next.
*   `end` (or `e`): Terminate the pipeline immediately and keep all generated files.

## Step-by-Step Instructions

### Step 1: Note Generation
**Goal**: Convert raw draft into XHS structure.
1.  **Input**: Raw draft file/text.
2.  **Action**: Use `xiaohongshu-note-generator-skill`.
3.  **Output**: `[filename]_xhs_note_raw.md`.
4.  **🛑 INTERACTION**: Output "✅ Rough XHS Note saved.
    - Type 'next' to humanize (remove AI flavor).
    - Type 'skip' to use this raw version for polishing.
    - Type 'end' to stop here."
    - **WAIT** for user input.

### Step 2: Humanize (Remove AI Flavor)
**Goal**: Inject soul and remove "plastic smell".
1.  **Decision**:
    - If `skip`: Copy `[filename]_xhs_note_raw.md` to `[filename]_xhs_note_humanized.md` (pass-through).
    - If `next`: Use `remove-ai-flavor-skill`.
2.  **Output**: `[filename]_xhs_note_humanized.md`.
3.  **🛑 INTERACTION**: Output "✅ Humanized Note prepared.
    - Type 'next' for final polishing (Zinsser style).
    - Type 'skip' to use this version for image generation.
    - Type 'end' to stop here."
    - **WAIT** for user input.

### Step 3: Content Polishing
**Goal**: Final rhythm and clarity check.
1.  **Decision**:
    - If `skip`: Copy `[filename]_xhs_note_humanized.md` to `[filename]_xhs_note.md`.
    - If `next`: Use `content-polisher-skill`.
2.  **Output**: `[filename]_xhs_note.md`.
3.  **🛑 INTERACTION**: Output "✅ Content finalized.
    - Type 'next' to generate cover prompt.
    - Type 'skip' to skip visuals and go straight to publishing (text only).
    - Type 'end' to stop here."
    - **WAIT** for user input.

### Step 4: Cover Prompt Generation
**Goal**: Visual brainstorming.
1.  **Decision**:
    - If `skip`: Proceed to Step 7 (Publishing) without image assets.
    - If `next`: Use `xiaohongshu-cover-prompt-generator-skill`.
2.  **Output**: `[filename]_cover_prompt.md`.
3.  **🛑 INTERACTION**: Output "✅ Prompt ready.
    - Type 'next' to generate image via Browser.
    - Type 'skip' to use your own local image (provide path in next step).
    - Type 'end' to stop."

### Step 5 & 6: Image Pipeline
**Goal**: Generate and clean the cover.
1.  **Decision**:
    - If user provided a path during `skip` in Step 4: Use that image.
    - If `next`: Use `gemini-web-automator-skill` then `gemini-watermark-remover-skill`.
2.  **Output**: `[filename]_cover_clean.png`.
3.  **🛑 INTERACTION**: Output "✅ Visuals ready. Type 'next' to review and publish, or 'end' to stop."

### Step 7: Publishing
**Goal**: Final Review & Post.
1.  **Action**: Check if `[filename]_cover_clean.png` exists. If not, prepare for a text-only post.
2.  **🛑 INTERACTION**: Output "Ready to publish!\n- Note: [path]\n- Image: [Image Path or 'None (Text Only)']\nType 'publish' to confirm, or 'end' to cancel."
3.  **Action**: Use `xiaohongshu-publisher-skill`.


## File Naming & Storage Convention
All generated assets MUST be stored in a subfolder under `content/` named after the topic.
*   **Root Directory**: `content/[topic]/`
*   Original Draft: `content/[topic]/[topic].md`
*   Rough Note: `content/[topic]/[topic]_xhs_note_raw.md`
*   Humanized Note: `content/[topic]/[topic]_xhs_note_humanized.md`
*   Polished Note: `content/[topic]/[topic]_xhs_note.md`
*   Prompt: `content/[topic]/[topic]_cover_prompt.md`
*   Image: `content/[topic]/[topic]_cover_clean.png`


