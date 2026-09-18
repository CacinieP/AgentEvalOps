# AgentEvalOps — AI Agent 可观测性与回归测试平台

[English](README.md) | [中文](README.zh.md)

[![CI](https://github.com/CacinieP/AgentEvalOps/actions/workflows/ci.yml/badge.svg)](https://github.com/CacinieP/AgentEvalOps/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)

> 面向 AI Agent 的回归测试、漂移检测与质量保障平台。
> **完全由 Claude Code 构建** — AI 编程既是手段，也是产品本身。

> ⚠️ **命名声明**：本项目原名 AgentBench，现已改名为 **AgentEvalOps**，以避免与清华大学 [THUDM/AgentBench](https://github.com/THUDM/AgentBench) 学术基准测试项目混淆。

## 核心能力

AgentEvalOps 是一个 EvalOps 平台，帮助团队交付可靠的 AI Agent，提供以下核心能力：

- **测试套件管理** — 为 AI Agent 定义测试用例，包含输入与预期输出
- **回归测试** — 运行评估并在不同 Agent 版本间追踪质量变化
- **版本对比** — 并排对比两次运行结果，在部署前捕获退化
- **AI 智能分析** — 多 Provider AI 自动分析失败原因并给出修复建议
- **多 Provider 支持** — 支持 Anthropic、OpenAI 及任何 OpenAI 兼容 API（DeepSeek、Mistral、Groq、Together、OpenRouter、SiliconFlow 等）
- **本地优先** — 所有数据存储在浏览器 localStorage，无需后端数据库
- **API Key 透明机制** — Key 存储在 localStorage，仅在你触发测试或分析时发送到本应用的 server route，由服务端转发至配置的 provider。如需完全本地控制，请自行部署。

## 截图预览

| 仪表盘 | 测试套件 | 版本对比 |
|---|---|---|
| ![仪表盘](screenshots/dashboard.png) | ![套件](screenshots/suites.png) | ![对比](screenshots/compare.png) |

## 技术栈

- **Next.js 16**（App Router、Turbopack）
- **React 19** + TypeScript
- **Tailwind CSS 4** — 暗色主题仪表盘 UI
- **多 Provider AI** — Anthropic / OpenAI / OpenAI 兼容 API

## 快速开始

```bash
npm install
npm run dev
```

打开 [http://localhost:3000](http://localhost:3000) 查看仪表盘。

首次访问会自动加载示例数据（3 个测试套件、4 次运行记录）。

### AI Provider 配置

前往 **设置**（`/settings`）配置用于智能分析的 AI Provider：

| Provider | 模型 | 说明 |
|----------|--------|-------|
| **Anthropic** | Claude Sonnet、Opus、Haiku | 原生 Messages API |
| **OpenAI** | GPT-4o、GPT-4.1、o4-mini | Chat Completions API |
| **自定义** | 任意模型 | DeepSeek、Mistral、Groq、Together、OpenRouter、SiliconFlow 等 |

未配置 API Key 时，AI 智能分析将回退到内置演示数据。

## 页面

| 路由 | 说明 |
|------|------|
| `/` | 仪表盘 — 指标概览、套件总览、分数趋势、最近运行 |
| `/suites` | 测试套件管理 — 创建、展开、运行测试套件 |
| `/compare` | 并排回归分析 — 对比两个 Agent 版本 |
| `/run/[id]` | 运行详情 — 可展开查看单条用例分析 |
| `/settings` | AI Provider 配置与数据管理 |

## 架构

```
src/
├── app/
│   ├── page.tsx              # 仪表盘
│   ├── layout.tsx            # 根布局（SettingsProvider + DataProvider）
│   ├── globals.css           # 暗色主题（CSS 自定义属性）
│   ├── settings/page.tsx     # AI Provider 配置 + 数据管理
│   ├── suites/page.tsx       # 测试套件 CRUD + 运行模拟
│   ├── compare/page.tsx      # 版本对比 + AI 分析
│   ├── run/[id]/page.tsx     # 运行详情，用例可展开
│   ├── api/execute/route.ts  # Agent 端点执行端点
│   └── api/analyze/route.ts  # 多 Provider AI 分析端点
├── components/
│   ├── CreateSuiteModal.tsx  # 新建套件表单
│   ├── MetricCard.tsx        # 仪表盘指标卡片
│   ├── RunSimulation.tsx     # 测试执行动画
│   ├── ScoreRing.tsx         # SVG 环形分数图
│   ├── Sidebar.tsx           # 导航 + 最近运行
│   └── StatusBadge.tsx       # 通过/失败/警告状态标签
└── lib/
    ├── agent-adapter.ts      # Agent 端点适配器（OpenAI/Anthropic/自定义 HTTP）
    ├── ai-provider.ts        # 多 Provider AI 抽象
    ├── data-context.tsx      # localStorage 数据仓库（套件 + 运行）
    ├── demo-data.ts          # AI 分析回退演示数据
    ├── seed-data.ts          # 首次加载示例数据
    ├── settings-context.tsx  # AI Provider 设置（localStorage）
    ├── types.ts              # TypeScript 类型定义
    └── utils.ts              # 工具函数
```

## 数据流

- **套件与运行** → 经 `useSyncExternalStore`（响应式 context）存于 localStorage
- **AI 设置** → 单独存放于 localStorage，与测试数据隔离
- **示例数据** → 首次访问自动填充；可在设置页清除
- **AI 分析** → 客户端将 provider 配置（含 API Key）发送至 `/api/analyze`；服务端仅转发调用，不落盘

## 为什么做这个项目？

本项目是从 45+ 份每日机会报告中筛选出的、最具验证价值的黑客松创意：

1. **EvalOps / Agent 回归测试** 在 46 天的机会分析中出现了 **12 次以上** — 是出现频率最高的主题
2. 它完美体现了"AI 编程既是手段也是产品"：我们用 AI（Claude Code）构建了一个测试 AI Agent 的平台
3. "用 AI 测试 AI" 的范式在技术上有趣，在商业上也极具价值，因为 Agent 正在进入生产环境

> **演讲稿一句话**：部署 AI Agent 的团队在修改提示词、模型或工具时，缺乏可靠手段来检测质量退化。AgentEvalOps 提供面向 AI Agent 的、CI 友好的回归测试框架 —— 定义黄金测试用例，每次变更都运行评估，在问题进入生产环境前捕获退化。

## 常见问题

### 这个项目以前叫 AgentBench 吗？

**是。** 现在改名为 AgentEvalOps，既能更准确表达 EvalOps 定位，也避免与 THUDM/AgentBench 混淆。

| | THUDM/AgentBench | AgentEvalOps |
|---|---|---|
| **目的** | 学术基准测试，横向对比不同 LLM 的 Agent 能力 | EvalOps 仪表盘，帮助团队对自身 Agent 做回归测试 |
| **目标用户** | AI 研究者、模型开发者 | 工程团队、产品经理 |
| **测试内容** | 8 个通用环境（操作系统、数据库、网购等） | 你自己的 Agent 测试套件（客服、代码审查、数据提取等） |
| **产出物** | 排行榜、学术论文数据 | 回归报告、质量趋势图 |
| **技术栈** | Python + Docker + conda | Next.js + React + TypeScript |

如果你需要的是学术基准测试，请前往 [github.com/THUDM/AgentBench](https://github.com/THUDM/AgentBench)。

## 路线图

- **近期** — 套件与运行结果的导入/导出（JSON）；更多内置评测器
- **中期** — localStorage 之外的可选后端存储；CLI / CI 集成，在流水线中运行评估
- **远期** — 团队协作与权限；跨版本质量趋势报表

## 参与贡献

欢迎 issue 与 PR！请先阅读[贡献指南](./CONTRIBUTING.md)，其中包含本地开发、分支命名与 issue → PR 工作流程。

## 许可证

MIT — 详见 [LICENSE](./LICENSE)。
