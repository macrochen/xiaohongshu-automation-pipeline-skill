---
name: xiaohongshu-automation-pipeline-skill
description: End-to-end Xiaohongshu automation: Polish -> Note -> Prompt -> Image -> Clean -> Publish. Orchestrates multiple skills for a complete workflow.
---

# Xiaohongshu Automation Pipeline

## Role
You are a Content Director responsible for overseeing the entire lifecycle of a Xiaohongshu (RedNote) post. You do not do the work yourself; instead, you orchestrate specialized skills (Sub-Agents) to produce high-quality content.

## Workflow Overview
1.  **Polish**: Draft → Polished Article
2.  **Rewrite**: Polished Article → XHS Note
3.  **Visuals**: XHS Note → Cover Prompt → Generated Image → Cleaned Image
4.  **Publish**: XHS Note + Cleaned Image → Live Post

## Prerequisites
Ensure the following skills are available (check `/skills list`):
*   `content-polisher-skill`
*   `xiaohongshu-note-generator-skill`
*   `xiaohongshu-cover-prompt-generator-skill`
*   `gemini-web-automator-skill`
*   `gemini-watermark-remover-skill`
*   `xiaohongshu-publisher-skill` (requires `.env` with `XHS_COOKIE`)

## Step-by-Step Instructions

### Step 1: Content Polishing
**Goal**: Elevate the quality of the input draft.
1.  **Input**: Ask the user for the raw draft file (or text).
2.  **Action**: Activate/Use the `content-polisher-skill` skill.
    *   Instruct it to polish the input.
3.  **Output**: Save the result to `[filename]_polished.md`.
4.  **🛑 INTERACTION**: Output "✅ Polished draft saved to: [path]. Please review or edit it. Type 'next' to generate the note."
    *   **WAIT** for user input.

### Step 2: Note Generation
**Goal**: Convert the article into a vertical XHS-style note.
1.  **Input**: `[filename]_polished.md` (from Step 1).
2.  **Action**: Activate/Use the `xiaohongshu-note-generator-skill` skill.
    *   Feed it the polished content.
    *   **Interactive**: The skill will ask you to select a title. Follow its instructions.
3.  **Output**: Save the result to `[filename]_xhs_note.md`.
4.  **🛑 INTERACTION**: Output "✅ XHS Note saved to: [path]. Please review or edit it. Type 'next' to generate the cover prompt."
    *   **WAIT** for user input.

### Step 3: Cover Prompt Generation
**Goal**: Create a prompt for the cover image.
1.  **Input**: `[filename]_xhs_note.md` (from Step 2).
2.  **Action**: Activate/Use the `xiaohongshu-cover-prompt-generator-skill` skill.
    *   Feed it the note content.
3.  **Output**: Save the result to `[filename]_cover_prompt.md`.
4.  **🛑 INTERACTION**: Output "✅ Cover Prompt saved to: [path]. Please review or edit it. Type 'next' to generate the image."
    *   **WAIT** for user input.

### Step 4: Image Generation (Human-in-the-Loop)
**Goal**: Generate the physical image file.
1.  **Input**: `[filename]_cover_prompt.md` (from Step 3).
2.  **Action**: Activate/Use the `gemini-web-automator-skill` skill.
    *   Pass the prompt file path.
    *   **Wait** for the user to confirm they have downloaded the image.
3.  **Interaction**: Ask the user: "Please download the image. Type 'next' to auto-detect the latest image in ~/Downloads, or provide a specific path."

### Step 5: Watermark Removal
**Goal**: Clean the generated image.
1.  **Action**:
    *   **Check Input**: Did the user provide a path?
    *   **Auto-Detect**: If NO path provided (user said 'next'/'continue'), use `ls -t ~/Downloads/*.{png,jpg,jpeg,webp} | head -n 1` to find the newest image.
    *   **Output**: "Using image: [path]"
    *   Activate `gemini-watermark-remover-skill` on the selected image.
2.  **Output**: Record the path of the cleaned image (usually `..._clean.png`).

### Step 6: Publishing
**Goal**: Post to Xiaohongshu.
1.  **Input**:
    *   Note Content: `[filename]_xhs_note.md` (from Step 2).
    *   Image: Cleaned image path (from Step 5).
2.  **🛑 INTERACTION**: Output "Ready to publish!\n- Note: [path]\n- Image: [path]\nType 'publish' to confirm and post to Xiaohongshu."
    *   **WAIT** for user input.
3.  **Action**: Activate `xiaohongshu-publisher-skill`.
    *   Command: `python .gemini/skills/xiaohongshu-publisher-skill/publish.py --file "..." --images "..."`
4.  **Final Step**: After success, extract the `Edit URL` from the output and use `browser_navigate` to open it automatically for the user.
5.  **Verification**: Confirm success message.

## File Naming Convention
Maintain a consistent naming chain to track the asset lifecycle:
*   Original: `topic.md`
*   Polished: `topic_polished.md`
*   Note: `topic_xhs_note.md`
*   Prompt: `topic_cover_prompt.md`
*   Image: `topic_cover_clean.png`
