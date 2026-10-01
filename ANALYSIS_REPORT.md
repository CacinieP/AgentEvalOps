# AgentBench 代码分析报告

> 分析日期：2026-05-25
> 仓库来源：https://github.com/CacinieP/agentbench

---

## 项目概述

**AgentBench** 是一个面向 AI Agent 的回归测试、漂移检测与质量保障平台，完全使用 Claude Code 构建。

### 核心定位
与清华大学 THUDM/AgentBench 学术基准测试**无关**，本项目是面向工程团队的 **EvalOps Dashboard**，用于回归测试企业自己的 AI Agent。

### 技术栈

| 组件 | 技术 |
|------|------|
| 框架 | Next.js 16 (App Router + Turbopack) |
| UI | React 19 + Tailwind CSS 4 |
| 语言 | TypeScript 5 (strict 模式) |
| 数据存储 | localStorage (useSyncExternalStore) |
| AI 集成 | 纯 fetch，无 SDK，支持多 Provider |

---

## 项目结构

```
src/
├── app/                    # Next.js App Router 页面
│   ├── page.tsx           # Dashboard
│   ├── suites/page.tsx    # 测试套件管理
│   ├── compare/page.tsx   # 版本对比
│   ├── run/[id]/page.tsx  # 运行详情
│   ├── settings/page.tsx  # AI Provider 配置
│   └── api/               # Server Routes
│       ├── analyze/      # AI 分析端点
│       └── execute/      # Agent 执行端点
├── components/            # 6 个自定义组件
│   ├── CreateSuiteModal.tsx
│   ├── RunSimulation.tsx
│   ├── ScoreRing.tsx
│   ├── MetricCard.tsx
│   ├── Sidebar.tsx
│   └── StatusBadge.tsx
└── lib/                   # 核心逻辑
    ├── ai-provider.ts     # 多 Provider AI 抽象层
    ├── agent-adapter.ts   # Agent HTTP 适配器
    ├── evaluator.ts       # 评估器 (Contains/Regex/JSON/LLM Judge)
    ├── data-context.tsx   # localStorage 数据层
    ├── settings-context.tsx
    ├── types.ts
    ├── utils.ts
    ├── seed-data.ts       # 示例数据
    └── demo-data.ts       # AI 分析降级数据
```

---

## 核心功能

### 1. 测试套件管理
- 创建包含多个测试用例的套件
- 支持 4 种 Agent 类型：Chatbot / Code Review / Data Extraction / Other
- 每个用例定义 Input + Expected Output + Evaluator

### 2. 回归测试
两种运行模式：
- **Live Mode**: 真实调用配置的 Agent Endpoint
- **Simulation Mode**: 确定性哈希生成模拟分数（无 Agent 时演示用）

### 3. 版本对比 (Compare)
- 选择 Baseline 和 Candidate 两次运行
- 四维对比：Score / Pass Rate / Latency / Cost
- 自动分类：Improved / Regression / Flaky / Unchanged
- 触发阈值：|delta| > 0.05

### 4. AI 分析
- 支持多 Provider：Anthropic / OpenAI / Custom (DeepSeek, Mistral, Groq, Together, OpenRouter, SiliconFlow)
- Server Route 转发请求，保护 API Key
- 未配置时降级为内置演示分析

### 5. 评估器系统

| 评估器 | 评分逻辑 |
|--------|----------|
| Contains | 期望短语匹配率 |
| Exact Match | 完全匹配 + Jaccard 相似度 |
| Regex | 正则表达式匹配 |
| JSON Schema | JSON 结构验证 |
| LLM Judge | AI 语义评判 |

---

## 技术亮点

### 1. useSyncExternalStore 响应式 localStorage
- 替代 useEffect + setState，符合 React Compiler 严格模式
- 零依赖，React 原生外部存储同步
- SSR 安全 (getServerSnapshot 返回空数组)
- 实时响应，`emitChange()` 触发所有订阅者重新读取

### 2. 多 Provider AI 抽象
- 统一 `callAI()` 接口，自动路由到 Anthropic Messages API 或 OpenAI Chat Completions
- 零 SDK 依赖，纯 fetch 实现
- 请求/响应格式自适应

### 3. 数据模型

```
TestSuite  1───*  TestCase
    │
TestRun  1───*  TestResult
```

所有数据存储在 localStorage，API Key 仅保存在客户端。

---

## 代码规模

| 指标 | 数据 |
|------|------|
| 总行数 | ~6,500 行 TypeScript/TSX |
| 组件数 | 6 个自定义组件 |
| 页面数 | 5 个主页面 + 2 个 API Route |
| 测试文件 | 3 个 (evaluator, utils, agent-adapter) |

---

## 产品定位

| 角色 | 使用场景 |
|------|----------|
| AI 产品经理 | 定义"好的 Agent 回答"，量化迭代效果 |
| Prompt 工程师 | 改完 prompt 后一键回归测试 |
| 后端工程师 | 换模型前对比分数变化 |
| DevOps/SRE | 接入 CI 流水线，低于阈值阻断部署 |

---

## 快速开始

```bash
cd ~/agentbench
npm install
npm run dev
# 访问 http://localhost:3000
```

首次访问自动加载示例数据（3 个套件 + 4 次运行）。

---

## 与 THUDM/AgentBench 的区别

| | THUDM/AgentBench | 本项目 |
|---|---|---|
| **目的** | 学术基准测试，对比 LLMs | 面向团队的 EvalOps，回归测试自己的 Agent |
| **目标用户** | AI 研究人员 | 工程团队 & 产品经理 |
| **测试内容** | 8 个通用环境 | 你自己定义的测试套件 |
| **输出** | 排行榜 & 研究数据 | 回归报告 & 质量趋势 |
| **技术栈** | Python + Docker | Next.js + React + TypeScript |

---

## 扩展路线

### 短期（1-2 周）
- [ ] 真实 Agent 执行（连接实际 AI Agent）
- [ ] 自定义评分规则（正则匹配 / 相似度 / LLM-as-Judge）

### 中期（1-2 月）
- [ ] 后端持久化（SQLite / PostgreSQL）
- [ ] CI/CD 集成（CLI 工具 + GitHub Action）
- [ ] 回归告警（Webhook / 邮件通知）

### 长期（3-6 月）
- [ ] 团队协作（多用户、权限管理）
- [ ] Agent 市场（共享测试套件模板）
- [ ] 多模态测试（图片、语音 Agent）