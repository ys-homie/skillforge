# SkillForge

> Modular AI agent skill templates for mobile dev workflows — from requirement analysis to code implementation.

模块化的 AI Agent 技能模板，覆盖需求拆解 → Figma 解析 → Plan → 代码实现全流程。  
适用于 Cursor / Codex / Claude Code 或任何 LLM 驱动的编码助手。

## 为什么需要 SkillForge

大多数 AI 编码助手缺少**结构化的工作流约束**——它们知道怎么写代码，但不知道你的项目该怎么做需求拆解、怎么读 Figma、Plan 要不要人工确认。

SkillForge 把这些「做事方法」拆成独立的 skill 文件，你只需复制到项目里、填上项目配置，AI 就能按流程工作。

## 目录结构

```
skills/
├── requirement-analysis/    # 需求拆解 + 工时预估
├── figma-parsing/           # Figma 设计稿解析
├── generate-plan/           # 生成实现 Plan
├── code-implementation/     # 按 Plan 代码实现
├── bugfix/                  # Bug 修复（轻量流程）
├── sync-i18n/               # 多语言词条同步
├── generate-efficiency-record/  # 生成提效跟踪记录
└── sync-efficiency-summary/     # 同步提效总表
```

## 快速上手

### 1. 复制

```bash
# 将 skills/ 复制到目标项目
cp -r skills/ your-project/.agents/skills/
```

### 2. 定制

每个 skill 文件中都有 `<!-- [项目特定] -->` 注释，替换为你的项目配置：

```
<!-- [项目特定] -->
<!-- iOS: AutoLayout 约束 / SwiftUI 自适应 -->
<!-- Android: dp/sp + ConstraintLayout / Compose Modifier -->
<!-- Flutter: .w / .h / .sp (ScreenUtil) -->
```

→ 替换为项目实际方案即可。

### 3. 引用

在编辑器规则（`.cursor/rules/workflow.mdc`）或系统提示词中引用 skill：

```markdown
## 需求开发流程

| 步骤 | Skill | 说明 |
|---|---|---|
| 1 | `requirement-analysis` | 需求拆解 + 传统工时预估 |
| 2 | `figma-parsing` | Figma 设计稿解析 |
| 3 | `generate-plan` | 生成实现 Plan（**确认后才进入下一步**） |
| 4 | `code-implementation` | 按 Plan 代码实现（**自测通过后才进入收尾**） |
```

> **提示：** 收尾阶段（sync-i18n / efficiency-record / efficiency-summary）涉及提效数据收集，团队暂不需要可整体跳过。

## Skill 总览

### 开发阶段

| Skill | 触发条件 | 核心输出 |
|---|---|---|
| `requirement-analysis` | 收到新功能/新页面需求 | 需求分析 + 工时预估（含风险分类） |
| `figma-parsing` | 有 Figma 链接 | UI 结构 + 字体/颜色映射 + 切图清单 |
| `generate-plan` | 需求 + Figma 解析完成 | 结构化实现 Plan（含复杂度评估） |
| `code-implementation` | Plan 经人工确认 | 代码 + 变更文件清单 + 自检清单 |
| `bugfix` | Bug / 异常 / 报错 | 定位 + 修复 + 影响范围 + 回归验证 |

### 收尾阶段

| Skill | 触发条件 | 核心输出 |
|---|---|---|
| `sync-i18n` | 有新增多语言词条 | 批量同步到翻译平台 |
| `generate-efficiency-record` | 自测通过 | 提效跟踪记录文档 |
| `sync-efficiency-summary` | 提效记录完成 | 更新提效总表 |

## 工作流概览

```
新需求 ──→ ① requirement-analysis
           ② figma-parsing
           ③ generate-plan ──→ 🔒 人工确认
           ④ code-implementation ──→ 🔒 人工自测
           ⑤ sync-i18n（可选）
           ⑥ generate-efficiency-record（可选）
           ⑦ sync-efficiency-summary（可选）

Bug ──→ bugfix（轻量流程，不走 Plan）
小改动 ──→ 直接实现（不走 Plan）
```

## 适配指南

| 项目类型 | 需定制内容 |
|---|---|
| **iOS** (Swift / UIKit / SwiftUI) | 项目约束、AutoLayout/SwiftUI 布局、Assets.xcassets 资源管理 |
| **Android** (Kotlin / Compose) | 项目约束、Compose/XML 布局、res/ 资源管理 |
| **Flutter** | 项目约束、ScreenUtil、Provider、ARB 国际化 |
| **React Native** | 项目约束、StyleSheet、Redux/Zustand、i18n |

## 核心原则

1. **任务分流 > 一刀切** — Bug 和小改动不走完整 Plan
2. **Skill 模块化** — 每个步骤独立，按需跳过
3. **传统工时事前记录** — 提效数据可信度的基石
4. **Plan 必须人工确认** — Step 3 → Step 4 是硬卡点
5. **Figma 链接全程保留** — UI 还原度验证的基础

## 兼容工具

- [Cursor](https://cursor.sh) — `.cursor/rules/` 引用 skill
- [OpenAI Codex](https://openai.com/codex) — 系统提示词引用
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — `CLAUDE.md` 引用
- 任何支持自定义 system prompt 的 LLM 工具

## License

[MIT](LICENSE)
