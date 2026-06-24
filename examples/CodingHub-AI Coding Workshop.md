# CodingHub — AI Coding Workshop

> 来源: `D:\CodingHub-AI Coding Workshop.pptx`
> 转换方式: MCP `analyze_document` + AI 助手多模态视觉 OCR（路径二）

---

## Slide 1 — CodingHub 编码经验分享

**PRACTICE**

### CodingHub 编码经验分享
**AI Coding 工具组合 × 问题定位 × 实战经验**

#### CodingHub 平台介绍

> *[AI OCR: slide_1_4 — CodingHub 网站截图]*
>
> 导航栏: 工具广场 | 论坛 | 热榜 | 快速开始 | 关于 | 登录 | 注册
>
> **发现 AI 的无限可能**
> 探索、分享、协作 — 找到最适合你的 AI 助手
>
> 分类标签: 全部 | Skill | MCP | 插件 | Prompt | 其他
>
> 推荐工具:
> - **CodeGraph** (Skill, by wangbao, 2021/7/6)
> - **harness-creator** (Skill, by wangbao, 2021/7/6)
> - **gitlab-mcp** (Skill, by wangbao, 2021/7/5)

**Spec-kit · OpenSpec · Superpowers**
**CodeBuddy · MCP · Skills**

Created by ima copilot, Edit by wangbao

---

## Slide 2 — 端到端开发测试概览

### 端到端开发测试概览

**Spec-kit | OpenSpec Superpowers + UI-UX-Pro-Max + OpenCLI 完整链路**

开发流程六阶段:

| 阶段 | 英文 |
|------|------|
| 规格定义 | Specify |
| 方案规划 | Plan |
| 代码实现 | Implement |
| 验证确认 | Validate |
| 测试 | Test |
| 问题修复验证 | Repair |

SDD/TDD

#### 工具组合映射

> *[AI OCR: slide_2_30 — 工具组合映射表]*

| 阶段 | Spec-Kit | OpenSpec | Superpowers | OpenSpec-Superpowers |
|------|----------|----------|-------------|---------------------|
| 规格定义 | Specify → clarify | explore → propose | brainstorming | explore → propose |
| 方案规划 | Plan → tasks | propose | write-plan | write-plan |
| 代码实现 | implements | apply | executing-plans | executing-plans |
| 验证确认 | Checklist/analyze | verify | requesting-code-review / finish-branch | verify |
| 问题修复 | — | — | — | systematic-debugging |
| 测试验证 | — | — | — | opencli / pytest / playwright-cli |

---

## Slide 3 — 方案设计编码

### 方案设计编码

**Spec-kit / OpenSpec + Superpowers 组合 — 二选一即可**

- Spec-kit 核心工作流（以 feature 为粒度）
- OpenSpec + Superpowers 组合核心工作流（以 change 为粒度）

#### 步骤 1: 探索与规格定义

| | Spec-kit | OpenSpec + Superpowers |
|---|---|---|
| 命令 | `/speckit.specify` | `/opsx.explore` |
| 说明 | 探索 思考 | 探索 |
| 输出 | 自然语言 → spec.md，聚焦 WHAT 和 WHY | — |

#### 步骤 2: 方案细化与规划

| | Spec-kit | OpenSpec + Superpowers |
|---|---|---|
| 命令 | `/speckit.clarify` | `/opsx.propose` |
| 说明 | 自然语言 → spec.md + proposal.md + design.md + tasks.md | — |
| | 识别并解决功能规格中的模糊或不完整之处 | — |

| | Spec-kit | OpenSpec + Superpowers |
|---|---|---|
| 命令 | `/speckit.plan` | `/opsx.write-plan` |
| 输出 | spec.md → plan.md + data-model + api-contracts | tasks.md → plan.md |

#### 步骤 3: 任务拆分与实现

| | Spec-kit | OpenSpec + Superpowers |
|---|---|---|
| 命令 | `/speckit.tasks` | `/opsx.executing-plans` |
| 说明 | 按用户故事拆分为原子任务（US1~US5） | 按 phase 顺序执行，标记完成，失败暂停 |
| | SDD TDD 开发，红绿测试循环 | — |

| | Spec-kit | OpenSpec + Superpowers |
|---|---|---|
| 命令 | `/speckit.implement` | `/opsx.archive` |
| 说明 | — | 归档、同步到主规格 |
| 辅助命令 | constitution / checklist / analyze / tasks-to-issues | apply / verify / sync / new / continue / ff |

---

## 转换说明

本次转换完整演示了 **AI 助手驱动（路径二）** 的工作流:

1. **MCP `analyze_document`** — 提取文本骨架（text_skeleton）+ 57 张图片到磁盘
2. **AI 视觉读取** — 用 Read 工具读取 5 张关键大图，识别出 2 张有效内容图（slide_1_4 CodingHub 截图、slide_2_30 工具映射表）+ 3 张空白占位图
3. **组装输出** — 将 OCR 识别的文字内容嵌入文本骨架，过滤掉装饰性图片和空白占位图，生成结构化 Markdown

与 **路径一（markitdown 内置 LLM）** 的区别: 路径二不依赖外部 LLM API，由 AI 助手自身的多模态能力完成图片理解。
