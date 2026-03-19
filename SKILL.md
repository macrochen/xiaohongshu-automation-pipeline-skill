---
name: xiaohongshu-automation-pipeline-skill
description: 小红书分步发布流水线。负责串联润色、改写、封面提示词、生图、去水印、发布与可选归档，并在关键节点等待用户确认。
---

# Xiaohongshu Automation Pipeline

本技能是小红书发布任务的**总指挥**。它不直接代替各个原子技能产出内容，而是负责按顺序调用相关技能并管理每一步的确认节奏。

## Codex / Gemini 兼容执行规则

本技能可以运行在不同的 Agent 环境中，但**一律按“分步执行 + 关键节点确认”**处理，严禁一口气跑完整条流水线。

### 通用规则
- **一次只推进一个 Step 或一个确认点**。完成当前节点后必须停下，等待用户明确确认。
- **不能依赖特定运行时的交互函数**。即使运行环境没有 `/skills list`、`AskUserQuestion`、`run_shell_command`、`browser_navigate` 等机制，也必须改用普通对话或标准命令显式确认后再继续。
- **所有可选步骤默认关闭**。除非用户明确要求，否则不要自动执行可选节点。

### Codex 专用约束
- 在 Codex 中，不要把本技能理解为“全自动脚本”，而要理解为“人工可控的编排说明书”。
- Codex 执行时，必须手动补齐这些停顿点，不能因为工具链支持连续执行就跳过确认。
- 若用户只说“发这篇小红书”，默认也只能推进到下一个确认节点，不能自动跨越多个关键选择。

## 核心原则
1. **顺序强制**：必须按照 Step 0-7 顺序执行。
2. **确认机制**：每一个关键节点（如润色稿确认、标题确认、封面确认、发布确认）都必须由用户确认后才能进入下一步。**严禁自动连续执行多个 Step。**
3. **标题约束**：小红书标题必须控制在 20 字以内；若候选标题超限，必须在 Step 2 内修正，不能带病进入发布。
4. **纯文本约束**：`02-xhs-note.md` 必须是可直接发布的正文，默认使用纯文本表达，不要保留 Markdown 标题、列表语法或多余结构标记。
5. **可选归档**：Step 7 的 Obsidian 归档默认不执行。只有当用户明确说“归档”“保存到 Obsidian”“同步到知识库”时才允许执行。
6. **状态追溯**：每一步生成的中间产物（润色稿、笔记稿、提示词、封面图）必须明确告知用户存放路径。

## 文件与项目管理规范
所有发布任务必须在独立的项目目录下进行：`content/xiaohongshu/[project-name]/`
*   `content/xiaohongshu/[project-name]/00-original.md`: 原始草稿。
*   `content/xiaohongshu/[project-name]/01-polished.md`: 深度润色后的稿件。
*   `content/xiaohongshu/[project-name]/02-xhs-note.md`: 最终可发布的小红书笔记。
*   `content/xiaohongshu/[project-name]/03-cover-prompt.md`: 封面图生成提示词。
*   `content/xiaohongshu/[project-name]/cover.png`: 最终清洗后的封面图。

## 发布流水线 (Checklist)

```
小红书发布进度：
- [ ] Step 0: 初始化项目目录 (00-original.md)
- [ ] Step 1: 内容深度润色 (01-polished.md)
- [ ] Step 2: 改写为小红书笔记 (02-xhs-note.md)
- [ ] Step 3: 封面图提示词生成 (03-cover-prompt.md)
- [ ] Step 4: 封面图自动化生成 (原始下载图)
- [ ] Step 5: 去水印并归档封面 (cover.png)
- [ ] Step 6: 发布至小红书 (Success/Fail + Edit URL)
- [ ] Step 7: 归档至 Obsidian 知识库 (01-polished.md)
```

## 强制确认点 (Mandatory Checkpoints)

以下节点必须显式停顿，等待用户确认：

1. **Step 1 完成后**：展示 `01-polished.md` 的结果摘要与保存路径，等待用户确认润色稿。
2. **Step 2 完成后**：展示 `02-xhs-note.md` 的标题与正文摘要，要求用户确认标题方案与正文是否可继续。
3. **Step 3 完成后**：展示 `03-cover-prompt.md` 的提示词，等待用户确认或修改。
4. **Step 4 完成后**：说明已完成生图，并要求用户确认使用哪一张下载图片继续去水印。
5. **Step 5 完成后**：展示 `cover.png` 或说明其路径，等待用户确认封面。
6. **Step 6 之前**：明确列出将要发布的文案文件和图片文件；只有用户明确确认“继续发布”后，才能真正执行发布。
7. **Step 6 成功后**：告知已发布成功并展示 `Edit URL`；若用户同意，再执行 `open "<URL>"`。
8. **Step 7**：永远视为可选，不得默认执行。

如果用户在任意确认点提出修改意见，必须停留在当前节点迭代，直到用户明确说可以继续。

## 详细步骤指令

### Step 0: 初始化项目
- **目标**：准备工作空间。
- **动作**：创建目录 `content/xiaohongshu/[project-name]/`。
- **输入**：用户提供的原始草稿。
- **输出**：保存至 `content/xiaohongshu/[project-name]/00-original.md`。

### Step 1: 内容深度润色
- **目标**：提升原稿质量，得到适合进一步改写的素材。
- **输入**：`content/xiaohongshu/[project-name]/00-original.md`。
- **调用**：`activate_skill content-polisher-skill`。
- **输出**：保存为 `content/xiaohongshu/[project-name]/01-polished.md`。
- **停止要求**：生成后必须展示结果摘要与保存路径，并等待用户确认后再进入 Step 2。

### Step 2: 改写为小红书笔记
- **目标**：将长文改写为适合移动端竖屏阅读的小红书笔记。
- **输入**：`content/xiaohongshu/[project-name]/01-polished.md`。
- **调用**：`activate_skill xiaohongshu-note-generator-skill`。
- **关键约束**：
  - 必须提醒下游技能：**标题必须不超过 20 字**。
  - 正文必须为**可直接发布的纯文本风格**，不要保留 Markdown 标题和列表语法。
  - 若技能生成多个标题候选，必须先让用户选择或确认推荐方案，再保存最终版本。
- **输出**：保存至 `content/xiaohongshu/[project-name]/02-xhs-note.md`。
- **停止要求**：展示最终标题、正文摘要与保存路径，等待用户确认后再进入 Step 3。

### Step 3: 封面图提示词生成
- **目标**：为封面图生成可用提示词。
- **输入**：`content/xiaohongshu/[project-name]/02-xhs-note.md`。
- **调用**：`activate_skill xiaohongshu-cover-prompt-generator-skill`。
- **输出**：保存至 `content/xiaohongshu/[project-name]/03-cover-prompt.md`。
- **停止要求**：必须展示提示词内容或摘要与保存路径，等待用户确认后再进入 Step 4。

### Step 4: 封面图自动化生成
- **目标**：通过自动化生图拿到原始封面。
- **输入**：`content/xiaohongshu/[project-name]/03-cover-prompt.md`。
- **调用**：`activate_skill gemini-web-automator-skill`。
- **交互方式**：
  - 生图结束后，不要假设运行时支持特殊交互函数。
  - 直接用普通对话询问用户：是使用 `~/Downloads` 中最新图片，还是提供明确路径。
- **停止要求**：在确定具体图片路径后，才允许进入 Step 5。

### Step 5: 去水印并归档封面
- **目标**：清洗最终发布用封面。
- **输入来源**：
  - 如果用户提供了路径，直接使用该路径。
  - 如果用户要求自动识别，使用 `ls -t ~/Downloads/*.{png,jpg,jpeg,webp} | head -n 1` 之类的方法选择最新图片；若命令不可用，改用等价方式。
- **调用**：`activate_skill gemini-watermark-remover-skill`。
- **输出**：保存清洗后的图片到 `content/xiaohongshu/[project-name]/cover.png`。
- **停止要求**：生成后必须展示封面或其路径，等待用户确认后再进入 Step 6。

### Step 6: 发布至小红书
- **目标**：将笔记与封面图发布到小红书。
- **输入**：
  - 文案：`content/xiaohongshu/[project-name]/02-xhs-note.md`
  - 图片：`content/xiaohongshu/[project-name]/cover.png`
- **发布前确认**：必须先列出将要发布的文件路径，并等待用户明确确认“继续发布”。
- **调用**：`activate_skill xiaohongshu-publisher-skill`。
- **执行命令**：优先使用当前 Skill 或项目约定的虚拟环境；如果没有独立 `.venv`，再根据现有环境执行发布脚本。不要写死系统 Python。
- **结果处理**：
  - 发布成功后，提取并展示 `Edit URL`。
  - 只有用户同意继续时，才执行 `open "<URL>"`。
  - 若发布失败，停留在当前步骤排查，不得自动进入归档。

### Step 7: 归档至 Obsidian (可选)
- **目标**：将较完整的稿件保存至知识库。
- **输入**：`content/xiaohongshu/[project-name]/01-polished.md`。
- **动作**：激活 `obsidian-archiver-skill`。
- **执行**：`./.venv/bin/python .gemini/skills/obsidian-archiver-skill/scripts/archive.py --title "placeholder" --type "Xiaohongshu" --content_file "content/xiaohongshu/[project-name]/01-polished.md"`
- **默认行为**：跳过。
- **仅在以下情况下执行**：用户明确要求归档到 Obsidian / 知识库。

## 异常处理
- 若任一步骤输出文件不存在，必须先补齐或重跑当前步骤，不能跳步。
- 若标题超出 20 字、正文仍含 Markdown 结构、封面图缺失或发布脚本报错，必须停留在当前步骤修正。
- 若用户仅要求执行部分流程，例如“只生成小红书笔记”或“只发布现有笔记”，也必须遵守当前步骤的确认机制，但可以从对应步骤开始。
