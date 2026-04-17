# Claude Code Skills & Agent Pipeline

<div align="center">

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Claude Code](https://img.shields.io/badge/Claude%20Code-Skills-orange.svg)

**一套面向 [Claude Code](https://claude.ai/code) 的自定义 Skills 和 Agent 流水线工作流**

[Skills 列表](#-skills-列表) | [Agent 工作流](#-agent-流水线工作流) | [安装指南](#-安装) | [使用说明](#-使用)

</div>

---

## 项目简介

这个仓库包含两部分内容：

1. **自定义 Skills** — 可直接安装到 Claude Code 的功能扩展插件
2. **Agent 流水线工作流** — 一套将 Claude Code 内置 Agent 类型编排为自动化开发流水线的 `CLAUDE.md` 配置方案

两者可以独立使用，也可以组合使用。

## Skills 列表

### xiaohongshu-publish（小红书智能发布助手）

智能化小红书内容发布工具，支持视频、图文、长文多种形式，提供爆款优化建议。

| 特性 | 说明 |
|------|------|
| 多格式支持 | 视频、图片上传、文字配图、长文 |
| 爆款标题生成 | 基于 5 大公式自动生成 3-5 个标题方案 |
| 话题推荐 | 覆盖 10 大垂直领域的官方话题库 |
| 发布时间建议 | 根据内容类型推荐最佳发布时段 |
| 一键发布 | 通过 Playwright 自动完成发布流程 |

**触发命令**: `/xhs` `/xiaohongshu-publish` `/小红书发布`

**依赖**: Playwright MCP

> 详细文档: [skills/xiaohongshu-publish/SKILL.md](skills/xiaohongshu-publish/SKILL.md)

---

### cursor-billing（Cursor 订阅报销自动化）

自动完成 Cursor 订阅的月度报销全流程：登录账户 → 下载收据 → OCR 识别交易截图 → 智能匹配 → 文件重命名 → 金额汇总。

| 特性 | 说明 |
|------|------|
| 多账户批量处理 | 自动登录并切换多个 Cursor 账户 |
| 收据自动下载 | 从 Cursor Billing 页面下载 PDF 收据 |
| OCR 交易识别 | 使用 macOS Vision 框架识别银行交易截图 |
| 智能匹配 | 自动将收据与交易记录配对（金额+日期+卡号） |
| 统一命名 | 按规范格式重命名所有报销文件 |
| 金额汇总 | 输出完整的匹配明细表和汇总表 |

**触发命令**: `/cursor-billing` `/cursor报销` `/报销处理`

**依赖**: Playwright MCP, macOS（OCR 依赖 Vision 框架）

> 详细文档: [skills/cursor-billing/SKILL.md](skills/cursor-billing/SKILL.md)

---

## Agent 流水线工作流

这是一套通过 `CLAUDE.md` 配置实现的 Agent 自动化工作流，将 Claude Code 内置的 6 种 Agent 类型编排为完整的软件开发流水线。

### 流水线架构

```
用户想法/需求
    │
    ▼
┌─────────────────┐
│ product-manager  │  产品经理：需求调研、竞品分析、输出 PRD
└────────┬────────┘
         ▼
┌─────────────────────┐
│ feature-decomposer   │  功能拆解：PRD → 功能组 + 里程碑 + 验收标准
└────────┬────────────┘
         ▼
┌─────────────────────────┐
│ frontend-page-designer   │  页面设计：功能规格 → UI/交互设计规格
└────────┬────────────────┘
         ▼
┌─────────────────┐
│ module-coder     │  编码实现：按功能清单持续编码
└────────┬────────┘
         ▼
┌────────────────────────────┐
│ design-parity-inspector     │  设计还原审核：对比实现与设计规格
└────────┬───────────────────┘
         ▼
┌─────────────────┐
│ qa-inspector     │  质量验收：里程碑节点做质量检查
└─────────────────┘
```

### Agent 角色说明

| Agent | 角色 | 触发条件 | 输出文档 |
|-------|------|---------|---------|
| `product-manager` | 产品经理 | 需求模糊、需要竞品分析 | `docs/PRD-*.md` |
| `feature-decomposer` | 功能拆解 | 有明确的 PRD 或需求 | `docs/product-spec.md` |
| `frontend-page-designer` | 页面设计 | 有 UI 相关功能 | 设计规格文档 |
| `module-coder` | 编码实现 | 功能清单已确认 | 代码 + 里程碑报告 |
| `design-parity-inspector` | 设计审核 | UI 功能实现完成 | 设计一致性报告 |
| `qa-inspector` | 质量验收 | 到达里程碑节点 | QA 报告 |

### 三种使用模式

**完整流水线**（适用于新功能、新产品）:
```
product-manager → feature-decomposer → frontend-page-designer → module-coder → design-parity-inspector → qa-inspector
```

**简化流水线**（适用于需求已明确的任务）:
```
feature-decomposer → frontend-page-designer → module-coder → design-parity-inspector → qa-inspector
```

**快速修复**（适用于 bug 修复、小改动）:
```
module-coder → qa-inspector
```

> 详细配置: [workflow/agent-pipeline.md](workflow/agent-pipeline.md)

---

## 安装

### 方法一：克隆整个仓库到 skills 目录

```bash
# 克隆仓库
git clone https://github.com/mojiQAQ/claude-code-skills.git

# 将 skills 复制到 Claude Code 的工作目录
cp -r claude-code-skills/skills/xiaohongshu-publish ~/.claude/skills/
cp -r claude-code-skills/skills/cursor-billing ~/.claude/skills/
```

### 方法二：只安装单个 Skill

```bash
# 只安装小红书发布助手
cd ~/.claude/skills
git clone https://github.com/mojiQAQ/claude-code-skills.git
ln -s claude-code-skills/skills/xiaohongshu-publish ./xiaohongshu-publish

# 或直接复制
cp -r claude-code-skills/skills/xiaohongshu-publish ./
```

### 方法三：作为项目级 Skill

将 Skill 目录放入你的项目根目录：

```bash
cd your-project
mkdir -p .claude/skills
cp -r /path/to/claude-code-skills/skills/xiaohongshu-publish .claude/skills/
```

### 安装 Agent 工作流

将工作流配置添加到你项目的 `CLAUDE.md` 文件中：

```bash
# 方法 1：直接复制示例配置
cat claude-code-skills/examples/CLAUDE.md.example >> your-project/CLAUDE.md

# 方法 2：手动添加
# 参考 workflow/agent-pipeline.md 中的配置，复制到你的 CLAUDE.md
```

---

## 使用

### 使用 Skills

在 Claude Code 中直接输入触发命令：

```bash
# 小红书发布
/xhs

# Cursor 报销
/cursor-billing
```

### 使用 Agent 工作流

配置好 `CLAUDE.md` 后，直接向 Claude Code 描述你的需求即可。Agent 会根据需求自动选择流水线模式：

```bash
# 需求模糊 → 自动从 product-manager 开始
"我想做一个类似 Notion 的笔记产品"

# 需求明确 → 自动从 feature-decomposer 开始
"给这个项目添加用户认证功能，要支持 OAuth 和邮箱登录"

# 小任务 → 自动走 module-coder + qa-inspector
"修复登录页面的样式错位问题"
```

### 在 CLAUDE.md 中配置 Skills + 工作流

你可以在同一个 `CLAUDE.md` 中同时配置工作流和 Skill 的使用说明：

```markdown
# CLAUDE.md

## 项目信息
- 项目名称：MyApp
- 技术栈：Vue 3 + TypeScript + FastAPI

## 开发工作流规则
<!-- 粘贴 examples/CLAUDE.md.example 的内容 -->

## 常用 Skills
- 发布小红书内容时使用 /xhs
- 每月报销时使用 /cursor-billing
```

---

## 前置依赖

| 依赖 | 用途 | 安装方式 |
|------|------|---------|
| [Claude Code](https://claude.ai/code) | AI 编程助手 | `npm install -g @anthropic-ai/claude-code` |
| [Playwright MCP](https://github.com/anthropics/mcp) | 浏览器自动化 | 在 Claude Code 设置中启用 |

### Playwright MCP 配置

在 Claude Code 的 settings.json 中添加：

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

## 项目结构

```
claude-code-skills/
├── README.md                           # 本文件
├── LICENSE                             # MIT 许可证
├── skills/
│   ├── xiaohongshu-publish/            # 小红书智能发布助手
│   │   ├── SKILL.md                    # Skill 定义文件（Claude Code 读取）
│   │   ├── skill.json                  # Skill 元数据
│   │   └── LICENSE                     # MIT 许可证
│   └── cursor-billing/                 # Cursor 报销自动化
│       ├── SKILL.md                    # Skill 定义文件
│       └── skill.json                  # Skill 元数据
├── workflow/
│   └── agent-pipeline.md               # Agent 流水线详细文档
└── examples/
    └── CLAUDE.md.example               # CLAUDE.md 工作流配置示例
```

---

## 自己开发 Skill

一个最简单的 Skill 只需要一个 `SKILL.md` 文件：

```markdown
---
name: my-skill
description: 这个 skill 做什么
license: MIT
triggers:
  - /my-skill
  - /ms
---

# My Skill

这里写 skill 的详细指令...
```

可选添加 `skill.json` 提供结构化元数据：

```json
{
  "name": "my-skill",
  "version": "1.0.0",
  "description": "这个 skill 做什么",
  "author": "your-name",
  "license": "MIT",
  "triggers": ["/my-skill", "/ms"],
  "dependencies": {
    "mcp": ["playwright"]
  },
  "tags": ["自动化"],
  "metadata": {
    "language": "zh-CN"
  }
}
```

---

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件

## 作者

**Moji** — [GitHub](https://github.com/mojiQAQ)

---

<div align="center">

如果这个项目对你有帮助，欢迎 Star 支持一下！

</div>
