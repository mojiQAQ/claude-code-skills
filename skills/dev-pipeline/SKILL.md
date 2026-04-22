---
name: dev-pipeline
description: 启用 7-agent 开发流水线（product-manager → feature-decomposer → frontend-page-designer → module-coder → design-parity-inspector → qa-inspector）。当用户有产品构思、功能需求、重构任务或 bug 修复需要系统化执行时触发；也可以通过 /dev-pipeline 主动调用以确认本会话按流水线执行。
license: MIT
---

# 开发流水线 Skill

这个 skill 是 `claude-code-skills` plugin 的"编排大脑"。当它被激活时，Claude 必须按以下规则把用户的开发任务派发给 plugin 提供的 7 个专职 agent，而不是自己直接动手。

> 如果用户明确说"不用走流水线"、"直接改一下就行"、"只是问个问题"，则不激活本 skill 的约束，按普通对话回答即可。

---

## 流水线总览

```
用户想法/需求
    │
    ▼
┌─────────────────────┐
│ product-manager      │  需求模糊 / 需要调研
│ (产品经理 Agent)     │  → docs/PRD-{产品名}-{日期}.md
└────────┬────────────┘
         ▼
┌─────────────────────┐
│ feature-decomposer   │  需求明确后从此开始
│ (功能拆解 Agent)     │  → docs/product-spec.md
│                      │  → docs/requirements-traceability.md
└────────┬────────────┘
         ▼
┌─────────────────────┐
│ frontend-page-designer  有 UI 时启动
│ (页面设计 Agent)     │  → docs/page-design-spec.md
└────────┬────────────┘
         ▼
┌─────────────────────┐
│ module-coder         │  按功能清单连续编码
│ (编码 Agent)         │  → 代码 + docs/milestone-N-report.md
└────────┬────────────┘
         ▼
┌─────────────────────┐
│ design-parity-inspector 有 UI 时启动
│ (设计还原审核)       │  → docs/milestone-N-design-review.md
└────────┬────────────┘
         ▼
┌─────────────────────┐
│ qa-inspector         │  里程碑节点验收
│ (QA Agent)           │  → docs/milestone-N-qa-report.md
└─────────────────────┘
         │
    🟢 通过 → module-coder 执行 git commit → 继续下一批
    🔴 不通过 → module-coder 修复 → 重新 QA
```

---

## 各阶段触发规则

| 阶段 | Agent | 何时触发 | 输入 | 输出 |
|------|-------|---------|------|------|
| 0 | **product-manager** | 需求模糊、需要竞品分析、"调研一下"、从零做新产品 | 用户的模糊想法 | `docs/PRD-*.md` |
| 1 | **feature-decomposer** | 有明确 PRD 或需求描述 | PRD 或用户需求 | `docs/product-spec.md` + `docs/requirements-traceability.md` |
| 2 | **frontend-page-designer** | 任何需要 UI 的功能 | 功能清单 | `docs/page-design-spec.md` |
| 3 | **module-coder** | 功能清单已确认 | 功能清单 + 设计规格 | 代码 + milestone 报告 |
| 4 | **design-parity-inspector** | coder 实现了有 UI 的功能 | 代码 + 设计规格 | 设计一致性报告 |
| 5 | **qa-inspector** | coder 到达里程碑节点 | 代码 + 追踪清单 + 设计审核 | QA 报告 |

## 四种使用模式

| 模式 | 适用 | 路径 |
|------|------|------|
| **完整流水线** | 新产品、需求模糊 | 0 → 1 → 2 → 3 → 4 → 5 |
| **简化流水线** | 需求已明确、有 UI | 1 → 2 → 3 → 4 → 5 |
| **纯后端** | 无 UI 项目 | 1 → 3 → 5 |
| **快速修复** | Bug、小改动 | 3 → 5 |

## 协作规范

- 每个 agent 的输出必须写入 `docs/`，作为下一个 agent 的输入
- `docs/requirements-traceability.md` 是核心对账底本；coder/QA 每完成一项都要更新状态
- 上游 agent 完成后，**主 Claude 自动调用下一个 agent，不等用户指令**
- 如果下游发现上游信息缺失（例如 PRD 漏了关键字段），回溯到对应 agent 补充，而不是自己编
- 所有 agent 输出使用中文

## 判断起点的简易算法

读完用户输入后按顺序判断：

1. 用户给的是"想法 / 方向 / 调研需求"？ → 从 **product-manager** 开始
2. 用户给的是"明确的功能描述 / PRD"？ → 从 **feature-decomposer** 开始
3. 用户给的是"修 bug / 小改一下"？ → 直接 **module-coder** → **qa-inspector**
4. 项目没有 UI？ → 跳过 frontend-page-designer 和 design-parity-inspector

**不要跳过这个流程，不要等用户提醒。**
