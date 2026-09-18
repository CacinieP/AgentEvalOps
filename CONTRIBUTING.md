# 贡献指南

感谢关注 AgentEvalOps！本文档描述如何有序地参与贡献。

## 本地开发

环境要求：Node 20+，npm 10+。

```bash
npm install        # 安装依赖
npm run dev        # 启动开发服务器 http://localhost:3000
npm run lint       # ESLint 检查
npm test           # Vitest 单元测试
npm run build      # 生产构建（含 TypeScript 检查）
```

提交 PR 前，请确保 lint、test、build 三项全部通过（CI 会跑同样的检查）。

## 分支命名

从 `main` 切出功能分支，命名格式为 **`<类型>/<功能短词>`**：

| 前缀 | 用途 | 示例 |
|---|---|---|
| `feat/` | 新功能 | `feat/export-report` |
| `fix/` | 缺陷修复 | `fix/compare-picker` |
| `chore/` | 工程与治理 | `chore/governance` |
| `docs/` | 文档 | `docs/readme-polish` |

## 工作流程

1. **先开 issue**：描述问题或提案，达成一致后再动手，避免无效投入。
2. **分支实现**：从最新 `main` 切出分支，保持每个分支只关注一件事。
3. **提交 PR**：标题遵循 `类型: 摘要`；正文使用 PR 模板，用 `Fixes #N` 关联 issue。
4. **CI 通过后合并**：CI 会自动运行 lint + test + build。合并采用 squash，保持 main 历史线性。

## 代码风格

- TypeScript strict 模式；避免 `any`，必要时用注释说明原因
- 组件与工具函数需有对应测试（Vitest，放在同目录 `*.test.ts`）
- UI 文案使用中文（与现有页面保持一致），代码标识符使用英文
- 提交信息使用祈使句，如 `fix: 修复对比页下拉框溢出`

## 报告缺陷

请使用 issue 模板中的 Bug 报告模板，附上复现步骤、期望行为与实际行为，以及浏览器/Node 版本。

## 许可

提交即表示你同意以 [MIT License](./LICENSE) 授权你的贡献。
