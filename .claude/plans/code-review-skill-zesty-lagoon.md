# 修改 code-review-expert skill

## Context
用户已将 sanyuan0704/code-review-expert skill 下载到 `.claude/skills/code-review-expert/`。需要两处修改：
1. 将审查范围从"未暂存改动"改为"仅检查暂存区改动"
2. 将 skill 调用命令从 `code-review-expert` 更名为 `code-review`

## 修改内容

### 1. 重命名目录
- `.claude/skills/code-review-expert/` → `.claude/skills/code-review/`

### 2. 修改 SKILL.md（位于重命名后的目录中）

**Frontmatter name 字段（第 2 行）：**
- `name: code-review-expert` → `name: code-review`

**Overview 描述（第 10 行）：**
- "current git changes" → "staged git changes"

**Workflow Step 1 Preflight context（第 25-26 行）：**
- `git diff --stat` → `git diff --cached --stat`
- `git diff` → `git diff --cached`

**Edge cases（第 30 行）：**
- 将 "If `git diff` is empty, inform user and ask if they want to review staged changes or a specific commit range."
- 改为 "If `git diff --cached` is empty, inform user that there are no staged changes and suggest using `git add` to stage files first."

**Large diff（第 31 行）：**
- `git diff` → `git diff --cached`（隐含，不需改文字描述）

### 3. 修改 agents/agent.yaml
- `display_name`: "Code Review Expert" → "Code Review"
- `default_prompt`: "Review current git changes" → "Review staged git changes"

### 4. 可选：删除 README.md
- 该文件为 skill 仓库的说明，非 skill 运行所需，可删除

## 不需要修改的文件
- `references/` 下的 4 个检查清单文件（solid-checklist.md、security-checklist.md、code-quality-checklist.md、removal-plan.md）— 与审查范围无关

## 验证
- 确认 skill 重命名后，`/code-review` 命令可正常触发
- 用 `git add` 暂存一些改动后运行 `/code-review`，确认只检查暂存区内容
