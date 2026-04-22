# Claude Code Agents & Skills

<div align="center">

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Claude Code](https://img.shields.io/badge/Claude%20Code-Agents-orange.svg)

**一套面向 [Claude Code](https://claude.ai/code) 的自定义 Agent 子系统 + 自动化开发流水线**

将产品构思到代码上线的完整流程，编排为 7 个专职 Agent 的自动化流水线。

[Agent 总览](#agent-总览) | [流水线架构](#流水线架构) | [安装指南](#安装) | [CLAUDE.md 配置](#在-claudemd-中启用流水线) | [Skills](#附赠-skills)

</div>

---

## 项目简介

这个仓库包含：

1. **7 个自定义 Agent 定义文件** — 放入 `~/.claude/agents/` 即可被 Claude Code 识别和调度
2. **Agent 流水线工作流配置** — 通过 `CLAUDE.md` 让 Agent 之间自动协作，形成完整的软件开发流水线
3. **2 个自定义 Skills** — 小红书智能发布助手 + Cursor 报销自动化（附赠）

## Agent 总览

| Agent | 角色 | 职责 | 输出 |
|-------|------|------|------|
| [`product-manager`](agents/product-manager.md) | 产品经理 | 竞品调研、用户分析、需求提炼 | `docs/PRD-*.md` |
| [`feature-decomposer`](agents/feature-decomposer.md) | 功能拆解师 | 将 PRD 拆解为功能组 + 里程碑 + 验收标准 | `docs/product-spec.md` + `docs/requirements-traceability.md` |
| [`frontend-page-designer`](agents/frontend-page-designer.md) | 页面设计师 | 基于功能规格产出 UI/交互设计规格 | `docs/page-design-spec.md` |
| [`designer`](agents/designer.md) | 视觉实现师 | 将设计规格转化为生产级 UI 代码 | 可工作的前端组件代码 |
| [`module-coder`](agents/module-coder.md) | 全栈工程师 | 按功能清单连续编码，里程碑处暂停 | 代码 + `docs/milestone-N-report.md` |
| [`design-parity-inspector`](agents/design-parity-inspector.md) | 设计还原审核 | 对比实现与设计规格的一致性 | `docs/design-parity-report.md` |
| [`qa-inspector`](agents/qa-inspector.md) | QA 工程师 | 里程碑质量验收，4 维度评分 + 硬阈值 | `docs/milestone-N-qa-report.md` |

### Agent 设计亮点

- **product-manager**: 强制深度竞品调研（至少 3-5 个直接竞品），输出带数据来源的 PRD，不接受"拍脑袋"需求
- **feature-decomposer**: 自动生成需求追踪清单（`requirements-traceability.md`），作为全链路的"对账底本"
- **module-coder**: 里程碑前强制需求对账，防止"实现了但漏了需求"；QA 通过后自动 git commit
- **qa-inspector**: 类似 GAN 判别器的对抗设计，内置"抗宽容校准"机制，强制 E2E 浏览器测试
- **design-parity-inspector**: 5 维度评分（还原度/统一性/业务适配/高级感/交互完整性），拒绝"AI 味"设计

---

## 流水线架构

```
用户想法/需求
    │
    ▼
┌─────────────────┐
│ product-manager  │  需求模糊时启动
│ (产品经理)       │  竞品调研 → 用户画像 → PRD
└────────┬────────┘
         │ docs/PRD-*.md
         ▼
┌─────────────────────┐
│ feature-decomposer   │  需求明确时直接从这里开始
│ (功能拆解)           │  PRD → 功能组 + 里程碑 + 验收标准
└────────┬────────────┘
         │ docs/product-spec.md
         │ docs/requirements-traceability.md
         ▼
┌─────────────────────────┐
│ frontend-page-designer   │  有 UI 时启动
│ (页面设计)               │  功能规格 → UI/交互设计规格
└────────┬────────────────┘
         │ docs/page-design-spec.md
         ▼
┌─────────────────┐
│ module-coder     │  按功能清单连续编码
│ (编码实现)       │  里程碑处暂停 → 交付报告
└────────┬────────┘
         │ 代码 + docs/milestone-N-report.md
         ▼
┌────────────────────────────┐
│ design-parity-inspector     │  有 UI 时启动
│ (设计还原审核)              │  对比实现 vs 设计规格
└────────┬───────────────────┘
         │ docs/design-parity-report.md
         ▼
┌─────────────────┐
│ qa-inspector     │  里程碑节点质量验收
│ (QA 验收)        │  4 维度评分 + E2E 测试
└────────┬────────┘
         │ docs/milestone-N-qa-report.md
         ▼
    🟢 通过 → module-coder 执行 git commit → 继续下一批功能
    🔴 不通过 → module-coder 修复 → 重新提交 QA
```

### 三种使用模式

| 模式 | 适用场景 | 流水线 |
|------|---------|--------|
| **完整流水线** | 新产品、需求模糊 | PM → Decomposer → Designer → Coder → Design Review → QA |
| **简化流水线** | 需求已明确 | Decomposer → Designer → Coder → Design Review → QA |
| **快速修复** | Bug 修复、小改动 | Coder → QA |
| **纯后端** | 无 UI 项目 | Decomposer → Coder → QA |

### 文档流转机制

Agent 之间通过 `docs/` 目录下的文档进行协作：

```
docs/
├── PRD-{产品名}-{日期}.md          ← product-manager 输出
├── product-spec.md                 ← feature-decomposer 输出
├── requirements-traceability.md    ← decomposer 生成，coder/QA 持续更新
├── page-design-spec.md             ← frontend-page-designer 输出
├── milestone-1-report.md           ← module-coder 输出
├── milestone-1-design-review.md    ← design-parity-inspector 输出
└── milestone-1-qa-report.md        ← qa-inspector 输出
```

`requirements-traceability.md` 是核心对账文档，状态流转：

```
🔲 待实现 (decomposer)
    → 🚧 开发中 (coder)
        → ✅ 已实现 (coder)
            → ✔️ QA 确认 (qa-inspector)
            → 🔴 QA 驳回 (qa-inspector)
```

---

## 安装

本仓库已按 **Claude Code Plugin** 规范组织，一次安装即可得到全部 agents + skills + slash commands。

### 方式一：作为 Plugin 安装（推荐）

把本仓库作为一个 marketplace 加入 Claude Code，然后安装 plugin：

```bash
# 1) 添加本仓库为 marketplace
/plugin marketplace add mojiQAQ/claude-code-skills

# 2) 安装 plugin
/plugin install claude-code-skills@claude-code-skills
```

安装后立刻生效：

- 7 个 agents 自动注册：`product-manager`、`feature-decomposer`、`frontend-page-designer`、`designer`、`module-coder`、`design-parity-inspector`、`qa-inspector`
- 1 个编排 skill：`/dev-pipeline` —— 激活流水线工作规则
- 1 个工具 skill：`/cursor-billing` —— Cursor 订阅报销自动化
- 1 个工具 skill：`/xiaohongshu-publish` —— 小红书内容发布
- 1 个 slash command：`/pipeline-init` —— 一键在当前项目 `CLAUDE.md` 写入流水线配置

### 方式二：本地目录安装（开发调试）

```bash
git clone https://github.com/mojiQAQ/claude-code-skills.git
# 在 Claude Code 中把本地路径加为 marketplace
/plugin marketplace add /path/to/claude-code-skills
/plugin install claude-code-skills@claude-code-skills
```

### 方式三：手工复制（不使用 plugin 机制时的后备方案）

```bash
git clone https://github.com/mojiQAQ/claude-code-skills.git
cd claude-code-skills

cp agents/*.md        ~/.claude/agents/
cp -r skills/*        ~/.claude/skills/
cat examples/CLAUDE.md.example >> ~/.claude/CLAUDE.md
```

### 在项目中启用流水线

Plugin 装好后，进入任意项目运行：

```
/pipeline-init
```

它会把流水线触发规则写入当前项目的 `CLAUDE.md`，之后该项目的所有开发任务都会自动走 7-agent 流水线。
也可以在任意会话里输入 `/dev-pipeline` 主动激活本会话的流水线约束。

### 前置依赖

| 依赖 | 用途 | 必需？ |
|------|------|--------|
| [Claude Code](https://claude.ai/code) | AI 编程助手 | 必需 |
| [Playwright MCP](https://github.com/anthropics/mcp) | E2E 测试 + 浏览器自动化 | QA 测试 / Skills 需要 |

Playwright MCP 配置（在 Claude Code 的 settings.json 中添加）：

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright"]
    }
  }
}
```

---

## 使用示例

### 场景 1：从零开始做一个新产品

```
你: 我想做一个 AI 面试助手

Claude Code 自动执行:
1. product-manager → 竞品调研 (面试相关产品)、用户画像、输出 PRD
2. feature-decomposer → 拆解功能清单、设定里程碑、生成追踪清单
3. frontend-page-designer → 设计页面和交互规格
4. module-coder → 连续编码实现
5. design-parity-inspector → 审核 UI 还原度
6. qa-inspector → 质量验收 (E2E 测试 + 4 维度评分)
```

### 场景 2：给现有项目加功能

```
你: 给这个项目添加用户认证功能，要支持 OAuth 和邮箱登录

Claude Code 自动执行:
1. feature-decomposer → 拆解认证功能、设定里程碑
2. frontend-page-designer → 设计登录/注册页面
3. module-coder → 实现认证系统
4. design-parity-inspector → 审核 UI
5. qa-inspector → 安全测试 + 功能验收
```

### 场景 3：修 Bug

```
你: 修复登录页面的样式错位问题

Claude Code 自动执行:
1. module-coder → 定位并修复问题
2. qa-inspector → 回归测试
```

---

## 各 Agent 详细说明

### product-manager（产品经理）

**核心理念**: "好的 PRD 是基于数据和调研的，不是基于拍脑袋的臆想"

- 强制深度竞品分析：至少 3-5 个直接竞品 + 2-3 个间接竞品
- 多角度搜索策略（英文/中文/Reddit/GitHub）
- 输出带数据来源的完整 PRD（竞品矩阵 + 用户画像 + MoSCoW 需求分级）
- 质量门控：每条数据必须标注来源，不能凭空编造

### feature-decomposer（功能拆解师）

**核心理念**: "定义 WHAT 和 WHY，不是 HOW"

- 将 PRD 拆解为有序功能组 + QA 里程碑
- 自动生成 `requirements-traceability.md` 需求追踪清单
- 每个功能有清晰的用户故事和可测试的验收标准
- 里程碑设在"方向性风险最高"的节点

### frontend-page-designer（页面设计师）

**核心理念**: "你不是通用的视觉装饰师"

- 4 层设计目标：统一性 → 业务适配 → 高级感/去 AI 化 → 交互完整性
- 明确 hover/active/focus/disabled/loading/empty/error/success 等全状态
- 输出可让 module-coder 实现的设计规格文档

### designer（视觉实现师）

**核心理念**: "平庸的界面会侵蚀用户信任"

- 自动检测项目前端框架，匹配已有代码风格
- 拒绝通用字体（Inter/Roboto/Arial）和 AI 味设计（紫色渐变白底）
- 产出生产级、有设计辨识度的可工作代码

### module-coder（全栈工程师）

**核心理念**: "连续实现，里程碑暂停"

- 按功能清单顺序连续编码，不需要为每个功能单独确认
- 里程碑前**强制需求对账**：逐条比对追踪清单，防止遗漏
- 自动更新追踪清单状态
- QA 通过后自动执行 git commit

### design-parity-inspector（设计还原审核）

**核心理念**: "缺少证据不等于通过评审"

- 5 维度评分：设计还原度 / 规范统一性 / 业务适配 / 高级感 / 交互完整性
- 问题分级：🔴 严重 / 🟡 中等 / 🟢 轻微
- 每个结论必须有截图/代码证据支撑

### qa-inspector（QA 工程师）

**核心理念**: "你和 coder 是对抗关系，你的职责是找出问题"

- 4 维度评分 + 硬阈值（功能完整性≥7 / 可靠性≥6 / 用户体验≥6 / 代码质量≥5）
- 强制 E2E 浏览器测试（Playwright MCP）
- 内置"抗宽容校准"：如果第一反应给 8 分，先找 3 个问题再决定
- 独立验证 coder 的自标记，不信任 ✅ 直到自己验证

---

## 项目结构

```
claude-code-skills/
├── README.md                               # 本文件
├── LICENSE                                  # MIT 许可证
├── agents/                                  # Agent 定义文件（核心）
│   ├── product-manager.md                   # 产品经理 Agent
│   ├── feature-decomposer.md                # 功能拆解 Agent
│   ├── frontend-page-designer.md            # 页面设计 Agent
│   ├── designer.md                          # 视觉实现 Agent
│   ├── module-coder.md                      # 编码实现 Agent
│   ├── design-parity-inspector.md           # 设计还原审核 Agent
│   └── qa-inspector.md                      # QA 验收 Agent
├── workflow/
│   └── agent-pipeline.md                    # 流水线详细文档
├── examples/
│   └── CLAUDE.md.example                    # CLAUDE.md 配置模板
└── skills/                                  # 附赠 Skills
    ├── xiaohongshu-publish/                 # 小红书智能发布助手
    │   ├── SKILL.md
    │   ├── skill.json
    │   └── LICENSE
    └── cursor-billing/                      # Cursor 报销自动化
        ├── SKILL.md
        └── skill.json
```

---

## 自定义与扩展

### 创建自己的 Agent

Agent 定义文件是一个带 YAML frontmatter 的 Markdown 文件，放入 `~/.claude/agents/` 即可：

```markdown
---
name: "my-agent"
description: "Agent 的用途描述，Claude Code 据此决定何时调用"
model: opus
color: blue
memory: user
---

你是 [角色]，负责 [职责]...

## 工作流程
...

## 输出要求
...
```

**关键字段说明:**

| 字段 | 说明 |
|------|------|
| `name` | Agent 唯一标识，用于在 CLAUDE.md 中引用 |
| `description` | Claude Code 用来判断何时调用此 Agent 的描述 |
| `model` | 使用的模型（`opus` / `sonnet` / `haiku`） |
| `color` | 在 CLI 中显示的颜色标识 |
| `memory` | 记忆范围（`user` = 用户级） |

### 调整流水线

你可以根据项目需要调整流水线行为，在 `CLAUDE.md` 中自定义：

```markdown
## 项目特殊规则

- 本项目是纯后端 API，跳过 frontend-page-designer 和 design-parity-inspector
- 里程碑 QA 不需要 E2E 浏览器测试，用 curl 测试 API 即可
- module-coder 每完成 3 个功能做一次 QA，不等里程碑
```

---

## 附赠 Skills

### xiaohongshu-publish（小红书智能发布助手）

智能化小红书内容发布：视频/图文/长文，爆款标题生成，话题推荐，一键发布。

**触发**: `/xhs` `/xiaohongshu-publish` `/小红书发布`

### cursor-billing（Cursor 报销自动化）

Cursor 订阅月度报销全流程：多账户登录 → 收据下载 → OCR 识别 → 智能匹配 → 金额汇总。

**触发**: `/cursor-billing` `/cursor报销` `/报销处理`

---

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件

## 作者

**Moji** — [GitHub](https://github.com/mojiQAQ)

---

<div align="center">

如果这个项目对你有帮助，欢迎 Star 支持一下！

</div>
