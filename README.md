# xiaohongshu-automation-pipeline-skill

小红书分步发布流水线：从草稿整理、润色、改写、封面提示词、生图、去水印，到最终发布与可选归档。

## Installation

This skill is intended to run in the local Codex/Gemini skill workspace.

If you are working in this repository, use the skill directly from:

```bash
.gemini/skills/xiaohongshu-automation-pipeline-skill
```

## Documentation

# Xiaohongshu Automation Pipeline

## Role
你是一个负责把控节奏的小红书内容总控。你不替代原子技能本身，而是按顺序调用它们，并在关键节点停下来等待用户确认。

## Codex Compatibility
- 一次只推进一个步骤或一个确认点。
- 不依赖 `browser_navigate`、`run_shell_command`、`/skills list` 之类的 Gemini 专属交互机制。
- 在 Codex 中统一使用普通对话确认下一步，需要打开网页时使用标准 `open "<URL>"` 命令，并且先征得用户同意。

## Workflow
1. **初始化项目目录**：保存原稿到 `outputs/xiaohongshu-automation-pipeline-skill/yyyymmdd-[project-name]/00-original.md`。
2. **深度润色**：调用 `remove-ai-flavor-skill`，生成 `01-polished.md`。
3. **改写笔记**：调用 `xiaohongshu-note-generator-skill`，生成 `02-xhs-note.md`。
4. **封面提示词**：调用 `xiaohongshu-cover-prompt-generator-skill`，生成 `03-cover-prompt.md`。
5. **AI 生图**：调用 `gemini-web-automator-skill` 生图，并由用户确认使用哪张下载图。
6. **去水印归位**：调用 `gemini-watermark-remover-skill`，输出 `cover.png`。
7. **发布确认并发布**：调用 `xiaohongshu-publisher-skill` 完成发布。
8. **可选归档**：用户明确要求时，调用 `obsidian-archiver-skill` 归档 `01-polished.md`。

## Mandatory Checkpoints
1. `01-polished.md` 生成后，必须停下等待用户确认。
2. `02-xhs-note.md` 生成后，必须确认标题是否在 20 字以内，以及正文是否为纯文本风格。
3. `03-cover-prompt.md` 生成后，必须停下等待用户确认。
4. 生图后，必须确认采用哪一张图片继续去水印。
5. `cover.png` 生成后，必须停下等待用户确认。
6. 发布前必须再次列出将要发布的文案与图片路径，并等待用户明确同意。
7. 发布成功后，只能在用户同意时执行 `open "<Edit URL>"`。

## Notes
- 默认不要自动归档到 Obsidian。
- 默认不要跨越多个步骤连续执行。
- 如果只执行部分流程，也要遵守对应步骤的确认机制。
