---
name: pre-push
description: "Review staged changes for issues, then commit with a conventional commit message. Use when you want to review before committing, or mention '/pre-push', '/review-and-commit', or 'pre-push hook'."
license: MIT
allowed-tools: Bash
---

# Pre-Push: Review then Commit

## Overview

推送前的统一流程：先对暂存代码进行结构化审查，根据审查结果调整代码，再生成规范的 commit message 提交。编排 code-review 和 git-commit 两个 skill 的工作流。

## 严重度等级

| 等级 | 名称 | 描述 | 操作 |
|-------|------|-------------|--------|
| **P0** | Critical | 安全漏洞、数据丢失风险、正确性 bug | 必须阻塞合并 |
| **P1** | High | 逻辑错误、严重 SOLID 违规、性能回退 | 应在合并前修复 |
| **P2** | Medium | 代码异味、可维护性问题 | 本 PR 修复或创建 follow-up |
| **P3** | Low | 风格、命名、小建议 | 可选改进 |

## Workflow

### Phase 0: 预检

1. 运行 `git status -sb` 和 `git diff --cached --stat` 检查暂存状态。
2. **无暂存文件**：告知用户先用 `git add` 暂存文件，流程终止。
3. **有暂存文件**：告知用户暂存了哪些文件，询问：

```
发现 X 个暂存文件待审查并提交。你希望：
1. 先完整审查，再提交
2. 跳过审查，直接提交
```

- 用户选 1 → 进入 Phase 1
- 用户选 2 → 跳到 Phase 3

### Phase 1: 代码审查

1. 运行 `git diff --cached` 获取完整暂存差异。
2. 大 diff（>500 行）按模块/功能分批审查。
3. 加载以下审查清单（通过相对路径引用 code-review skill 的检查清单）：
   - `../code-review/references/solid-checklist.md` — SOLID 违规与架构异味
   - `../code-review/references/security-checklist.md` — 安全与可靠性风险
   - `../code-review/references/code-quality-checklist.md` — 错误处理、性能、边界条件
   - `../code-review/references/removal-plan.md` — 可移除代码
4. 按以下格式输出审查结果：

```markdown
## Code Review Summary

**Files reviewed**: X files, Y lines changed
**Overall assessment**: [APPROVE / REQUEST_CHANGES / COMMENT]

---

## Findings

### P0 - Critical
(none 或列表)

### P1 - High
1. **[file:line]** Brief title
   - Description of issue
   - Suggested fix

### P2 - Medium
...

### P3 - Low
...
```

5. 审查通过（无问题）时，明确说明检查了哪些方面和未覆盖的范围。

### Phase 2: 用户决策与修复

展示审查结果后，询问用户：

```markdown
---

## Next Steps

发现 X 个问题 (P0: _, P1: _, P2: _, P3: _)。

**你希望如何处理？**
1. **全部修复** - 实施所有建议的修复，然后提交
2. **仅修复 P0/P1** - 只处理关键和高优先级问题
3. **选择性修复** - 告诉我要修复哪些问题
4. **直接提交** - 跳过修复，用当前代码提交
5. **中止** - 不提交，我自己处理
```

**重要：在用户明确确认之前，不做任何修改。**

#### 2.5: 实施修复（用户选择 1-3 时）

1. 实施用户要求的修复。
2. 修复完成后重新暂存变更文件：`git add <changed files>`。
3. 对新暂存的 diff 重新审查，验证修复是否解决了问题。
4. 展示更新后的摘要：哪些问题已解决，是否有新发现。
5. 询问用户是否继续提交或需要进一步修改。

#### 审查摘要（上下文保护）

Phase 2 结束后，生成精简摘要传递给 Phase 3，避免完整审查结果占用过多上下文：

```markdown
## Review Summary for Commit
- Total: X issues (P0: _, P1: _, P2: _, P3: _)
- Resolved: P0(file:line - brief), P1(file:line - brief)
- Deferred: P2(file:line - brief), P3(file:line - brief)
- Recommended type: fix/refactor/feat (reason)
```

**规则**：
- 只保留 severity、file:line、一句话描述
- 不保留完整审查分析过程
- 标记已修复和推迟的问题
- 根据审查结果推荐 commit type

### Phase 3: 生成提交信息并提交

1. 运行 `git diff --cached` 分析最终暂存差异。
2. 加载 `references/review-commit-bridge.md` 获取审查结果到提交信息的映射指南。
3. 确定 commit type 和 scope：
   - 分析 diff 的主要变更类型
   - 若有审查修复，根据 bridge 指南调整 type（参考摘要中的推荐）
4. 生成 Conventional Commit 格式的提交信息：

```
<type>[optional scope]: <description>

[optional body — 仅在 P0/P1 修复时包含审查摘要]

[optional footer — BREAKING CHANGE 等]
```

5. 将提交信息展示给用户确认。
6. 用户确认后执行：`git commit -m "<message>"`
7. 若 commit 失败（如 hooks），诊断问题并重试，创建新 commit（不 amend）。

## Git 安全协议

- 不修改 git config
- 不运行破坏性命令（--force, hard reset），除非用户明确要求
- 不跳过 hooks（--no-verify），除非用户要求
- 不 force push 到 main/master
- commit 因 hooks 失败时，修复问题后创建新 commit（不 amend）

## Commit 类型参考

| Type | 用途 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `docs` | 仅文档 |
| `style` | 格式/风格（无逻辑变更） |
| `refactor` | 代码重构（非功能/修复） |
| `perf` | 性能优化 |
| `test` | 添加/更新测试 |
| `build` | 构建系统/依赖 |
| `ci` | CI/配置变更 |
| `chore` | 杂项维护 |
| `revert` | 回滚提交 |

## 最佳实践

- 一次提交一个逻辑变更
- 使用现在时态："add" 而非 "added"
- 使用祈使语气："fix bug" 而非 "fixes bug"
- description 保持在 72 字符以内
- 关联 issue：`Closes #123`, `Refs #456`
