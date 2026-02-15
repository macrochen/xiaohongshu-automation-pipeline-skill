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
*   `obsidian-archiver-skill`

## File Management Convention
All project assets must be stored in a dedicated folder: `content/xiaohongshu/[topic-name]/`
*   `content/xiaohongshu/[topic-name]/00-original.md`: The raw draft.
*   `content/xiaohongshu/[topic-name]/01-polished.md`: The elevated draft.
*   `content/xiaohongshu/[topic-name]/02-xhs-note.md`: The final note content (for Obsidian).
*   `content/xiaohongshu/[topic-name]/03-cover-prompt.md`: The image generation prompt.
*   `content/xiaohongshu/[topic-name]/cover.png`: The final cleaned cover image.

## Step-by-Step Instructions

### Step 0: Initialization
**Goal**: Set up the workspace for the new article.
1.  **Action**: Create the directory `content/xiaohongshu/[topic-name]/`.
2.  **Input**: The raw draft text provided by the user.
3.  **Output**: Save the raw draft to `content/xiaohongshu/[topic-name]/00-original.md`.

### Step 1: Content Polishing
**Goal**: Elevate the quality of the input draft.
1.  **Input**: `content/xiaohongshu/[topic-name]/00-original.md`.
2.  **Action**: Activate/Use the `content-polisher-skill` skill.
    *   Instruct it to polish the input.
3.  **Output**: Save the result to `content/xiaohongshu/[topic-name]/01-polished.md`.
4.  **🛑 INTERACTION**: Output "✅ Polished draft saved to: content/xiaohongshu/[topic-name]/01-polished.md. Please review or edit it. Type 'next' to generate the note."
    *   **WAIT** for user input.

### Step 2: Note Generation
**Goal**: Convert the article into a vertical XHS-style note.
1.  **Input**: `content/xiaohongshu/[topic-name]/01-polished.md` (from Step 1).
2.  **Action**: Activate/Use the `xiaohongshu-note-generator-skill` skill.
    *   **CRITICAL**: Remind the skill that the **Title MUST be under 20 characters** and the content **MUST be plain text (No Markdown)**.
    *   Feed it the polished content.
    *   **Interactive**: The skill will ask you to select a title. Follow its instructions.
3.  **Output**: Save the result to `content/xiaohongshu/[topic-name]/02-xhs-note.md`.
4.  **🛑 INTERACTION**: Output "✅ XHS Note saved to: content/xiaohongshu/[topic-name]/02-xhs-note.md. Please review or edit it. Type 'next' to generate the cover prompt."
    *   **WAIT** for user input.

### Step 3: Cover Prompt Generation
**Goal**: Create a prompt for the cover image.
1.  **Input**: `content/xiaohongshu/[topic-name]/02-xhs-note.md` (from Step 2).
2.  **Action**: Activate/Use the `xiaohongshu-cover-prompt-generator-skill` skill.
    *   Feed it the note content.
3.  **Output**: Save the result to `content/xiaohongshu/[topic-name]/03-cover-prompt.md`.
4.  **🛑 INTERACTION**: Output "✅ Cover Prompt saved to: content/xiaohongshu/[topic-name]/03-cover-prompt.md. Please review or edit it. Type 'next' to generate the image."
    *   **WAIT** for user input.

### Step 4: Image Generation (Human-in-the-Loop)
**Goal**: Generate the physical image file.
1.  **Input**: `content/xiaohongshu/[topic-name]/03-cover-prompt.md` (from Step 3).
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
2.  **Output**: Save the cleaned image to `content/xiaohongshu/[topic-name]/cover.png`.

### Step 6: Publishing
**Goal**: Post to Xiaohongshu.
1.  **Input**:
    *   Note Content: `content/xiaohongshu/[topic-name]/02-xhs-note.md` (from Step 2).
    *   Image: `content/xiaohongshu/[topic-name]/cover.png` (from Step 5).
2.  **🛑 INTERACTION**: Output "Ready to publish!\n- Note: content/xiaohongshu/[topic-name]/02-xhs-note.md\n- Image: content/xiaohongshu/[topic-name]/cover.png\nType 'publish' to confirm and post to Xiaohongshu."
    *   **WAIT** for user input.
3.  **Action**: Activate `xiaohongshu-publisher-skill`.
    *   Command: `python .gemini/skills/xiaohongshu-publisher-skill/publish.py --file "content/xiaohongshu/[topic-name]/02-xhs-note.md" --images "content/xiaohongshu/[topic-name]/cover.png"`
4.  **Final Step**: After success, extract the `Edit URL` from the output.
    *   **🛑 INTERACTION**: Ask the user: "Publish successful! Would you like to open the edit page to check or adjust? (y/n)"
    *   If **'y'**: Use `run_shell_command` with the system's open command (e.g., `open "<URL>"` on macOS).
    *   If **'n'**: Proceed to Step 7.
5.  **Verification**: Confirm success message.

### Step 7: Archive to Obsidian
**Goal**: Save the detailed polished version to the long-term knowledge base.
1.  **Input**: `content/xiaohongshu/[topic-name]/01-polished.md`.
2.  **Action**: Activate `obsidian-archiver-skill`.
    *   Command: `python .gemini/skills/obsidian-archiver-skill/scripts/archive.py --title "placeholder" --type "Xiaohongshu" --content_file "content/xiaohongshu/[topic-name]/01-polished.md"`
3.  **Output**: Confirm the file is saved in `Obsidian-Knowledge-Base/`.

