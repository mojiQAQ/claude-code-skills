---
description: 在当前项目的 CLAUDE.md 中启用 claude-code-skills 开发流水线
---

把以下流水线配置片段追加到当前项目根目录的 `CLAUDE.md`（如果文件不存在就创建）：

```markdown
## 开发工作流规则（必须遵守）

本项目启用 claude-code-skills plugin 的 7-agent 开发流水线。所有开发任务自动按下列规则派发，
详细规格见 skill `dev-pipeline`（可用 `/dev-pipeline` 主动激活）。

### 各阶段触发规则

| 阶段 | Agent | 何时触发 |
|------|-------|---------|
| 0 | product-manager | 需求模糊、需要竞品调研、从零做新产品 |
| 1 | feature-decomposer | 有明确 PRD 或需求描述 |
| 2a | code-reviewer（架构评审） | feature-decomposer 出方案后、coder 动手前 |
| 2b | frontend-page-designer | 需要 UI/页面设计 |
| 3 | module-coder | 架构评审通过 + 功能清单已确认 |
| 4a | code-reviewer（代码评审） | coder 到达里程碑后、QA 介入前 |
| 4b | design-parity-inspector | coder 实现了有 UI 的功能 |
| 5 | qa-inspector | code-reviewer 阶段二通过后、里程碑节点 |

### 协作规范

- 每个 agent 的输出必须写入 `docs/` 子目录（`docs/architecture/`、`docs/test/`）
- `docs/test/requirements-traceability.md` 是全链路对账底本
- 上游完成后自动调用下游，不等用户指令
- code-reviewer 报 P0 → coder 立即修复；P1/P2 → coder 修后必须重新走 QA；仅 P3 → 放行
- 小任务（bug 修复等）：module-coder → code-reviewer（涉及安全/数据库/分布式逻辑必跑） → qa-inspector

**不要跳过这个流程。**
```

操作步骤：
1. 检查当前目录是否有 `CLAUDE.md`，没有就创建
2. 如果已存在且已经包含"开发工作流规则"字样，不要重复追加，提示用户"已配置"
3. 否则追加到文件末尾
4. 写入完成后输出一行：`✅ 已在 <path>/CLAUDE.md 启用 claude-code-skills 开发流水线`
