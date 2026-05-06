# 计划：创建 `pre-push` Skill（Code Review + Git Commit 编排）

## Context

当前项目有两个独立的 skill：`code-review`（审查暂存代码）和 `git-commit`（生成规范提交信息）。用户希望在推送代码前有一个统一的流程：先做代码审查，根据审查结果调整代码，然后提交。目前需要手动分两步执行，不够便捷。

## 方案

**创建一个新的 `pre-push` skill 作为编排器**，不修改现有两个 skill。

### 为什么不合并？

- 保持单一职责：`code-review` 和 `git-commit` 各自独立使用场景仍然存在
- 避免上下文丢失：不通过 Skill 工具调用子 skill，而是将审查和提交步骤内嵌在同一工作流中，确保审查结果能在提交阶段使用

### 文件结构

```
.claude/skills/pre-push/
├── SKILL.md                              # 主 skill 文件，编排完整工作流
└── references/
    └── review-commit-bridge.md           # 审查结果到提交信息的映射指南
```

### 工作流设计（4 个阶段）

**Phase 0 - 预检**
- 检查是否有暂存文件，无则提示用户先 `git add`
- 询问用户：完整审查后提交 / 跳过审查直接提交

**Phase 1 - 代码审查**
- 执行 `git diff --cached` 获取暂存差异
- 加载现有 code-review 的 4 个检查清单（通过相对路径引用）：
  - `../code-review/references/solid-checklist.md`
  - `../code-review/references/security-checklist.md`
  - `../code-review/references/code-quality-checklist.md`
  - `../code-review/references/removal-plan.md`
- 使用 P0-P3 严重度分级输出审查结果

**Phase 2 - 用户决策与修复**
- 展示审查结果摘要（仅保留关键信息：severity、type、file:line、一句话描述）
- 提供选项：修复全部 / 仅修复 P0-P1 / 选择性修复 / 直接提交 / 中止
- 若用户选择修复，实施修复后重新暂存并二次审查验证
- **不做任何修改，直到用户明确确认**

### 上下文保护策略（摘要传递）

审查阶段完成后，生成一份**精简摘要**替代完整审查结果传递给提交阶段：

```
## Review Summary for Commit
- Total: 5 issues (P0: 1, P1: 2, P2: 1, P3: 1)
- Resolved: P0(auth.ts:42 - XSS), P1(user.ts:15 - SRP)
- Deferred: P2(api.ts:88 - N+1 query), P3(naming)
- Recommended type: fix (security fix applied)
```

**规则**：
- 只保留 severity、类型分类、file:line、一句话描述
- 不保留完整的审查分析过程和检查清单内容
- 标记哪些已修复、哪些被推迟
- 根据审查结果推荐 commit type

**Phase 3 - 生成提交信息并提交**
- 基于审查摘要（而非完整审查结果）分析最终暂存差异
- 参考 `review-commit-bridge.md` 将摘要中的关键信息融入提交信息
- 生成 Conventional Commit 格式消息，展示给用户确认
- 执行 `git commit`
- 遵循 git-commit 的安全协议（不 force push、不跳过 hooks 等）

### 触发方式

- 直接调用：`/pre-push`
- 自然语言：包含"review before commit"、"review and commit"、"pre-push hook" 等关键词
- **不会**与单独的 `/code-review` 或 `/commit` 冲突

## 需要创建的文件

| 文件 | 说明 |
|------|------|
| `.claude/skills/pre-push/SKILL.md` | 主 skill 定义，包含完整 4 阶段工作流 |
| `.claude/skills/pre-push/references/review-commit-bridge.md` | 审查结果到 commit type/body 的映射指南 |

## 不需要修改的文件

- `.claude/skills/code-review/` — 保持不变，pre-push 通过相对路径引用其检查清单
- `.claude/skills/git-commit/` — 保持不变，pre-push 内嵌其提交逻辑
- `skills-lock.json` — 本地创建的 skill 无需更新 lock 文件

## 验证方式

1. 暂存一些文件变更，运行 `/pre-push`，验证完整流程
2. 测试"跳过审查直接提交"路径
3. 测试审查无问题的场景（直接进入提交）
4. 测试审查发现 P0 问题后的修复 → 重新审查 → 提交路径
5. 测试中途中止路径（确认不会产生任何提交）
