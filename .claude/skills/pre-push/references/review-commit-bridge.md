# Review-to-Commit Bridge

## 审查结果到提交类型的映射

| 审查结果 | 推荐 commit type | 说明 |
|---|---|---|
| 审查通过，新功能 | `feat` | 标准功能提交 |
| 审查通过，修复 bug | `fix` | 标准 bug 修复 |
| 修复了 P0 安全问题 | `fix` | 必要时加 `BREAKING CHANGE` footer |
| 修复了 P1 SOLID 违规 | `refactor` | body 中说明重构原因 |
| 修复了 P1/P2 代码质量 | `fix` 或 `refactor` | 正确性问题用 `fix`，结构问题用 `refactor` |
| 仅修复 P3 样式问题 | `style` | 无逻辑变更 |
| 功能 + 审查修复混合 | `feat` | body 中提及修复内容 |

## 何时在 commit body 中包含审查上下文

**包含**（审查发现 P0/P1 且用户选择修复）：
- 修复了什么安全问题
- 重构了什么架构问题
- 推迟了哪些问题（附 follow-up 建议

**不包含**（以下情况不写审查信息）：
- 审查完全通过，无发现问题
- 仅 P3 建议，且未被实施
- 审查信息与 diff 本身相比没有额外价值

## Commit body 模板（有审查修复时）

```
修复了审查中发现的 N 个问题：
- P0 [file:line] 一句话描述
- P1 [file:line] 一句话描述

推迟的问题：
- P2 [file:line] 一句话描述（建议 follow-up: #xxx）
```

保持简洁，body 用于提供 diff 之外的上下文，不是完整审查报告。

## 示例

**审查发现并修复了安全问题：**
```
fix(api): validate input schema before processing

修复了审查发现的 1 个问题：
- P0 [api.ts:42] 用户输入未校验，存在注入风险

Refs: security audit 2026-Q2
```

**审查触发了架构重构：**
```
refactor(store): extract user preferences into composable

修复了审查发现的 1 个问题：
- P1 [store.ts:88] SRP 违规，用户偏好与全局状态混合

推迟的问题：
- P2 [store.ts:120] N+1 查询问题（建议 follow-up）
```
