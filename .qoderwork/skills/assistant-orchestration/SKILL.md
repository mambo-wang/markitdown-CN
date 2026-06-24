---
name: assistant-orchestration
description: 使用 markitdown MCP 的 analyze_document 工具分析文档，并用助手自身视觉能力完成图片 OCR/描述。当用户要求分析文档、提取文档内容、OCR 识别时使用此技能。
version: 1.0.0
---

# AI 助手编排：文档分析与图片 OCR

## 概述

此技能让 AI 助手（QoderWork / CodeBuddy / Qoder）在**无需配置外部 LLM**的情况下，充分利用自身的视觉能力完成文档分析。

传统流程需要单独配置视觉模型来做 OCR，而本技能将流程拆为两阶段：
1. **MCP Server** 负责文档解析，提取文本骨架和图片文件
2. **AI 助手**利用自身的视觉能力直接读取图片并完成 OCR 或内容描述

这样既降低了部署复杂度，又能获得高质量的图片理解结果。

## 前置条件

在使用此技能前，请确认：

- **markitdown MCP 服务器已配置并可用**：检查 MCP 工具列表中是否包含 `analyze_document`
- **助手具备视觉能力**：QoderWork、CodeBuddy、Qoder 均支持通过 Read 工具读取图片文件

验证方式：在 MCP 工具列表中搜索 `analyze_document`，若存在则说明 MCP 服务器已就绪。

## 两阶段工作流

### Phase 1：文档分析

调用 `analyze_document` MCP 工具，传入目标文件路径：

```
工具: mcp__markitdown__analyze_document
参数: { "file_path": "/path/to/document.docx" }
```

工具返回三部分数据：

| 字段 | 说明 |
|---|---|
| `text_skeleton` | Markdown 格式的文本骨架，图片位置用 `![image](<path>)` 占位 |
| `images` | 提取出的图片文件列表，每项含 `path`、`size_bytes`、`width`、`height` |
| `metadata` | 文档元信息（标题、作者、页数等） |

**判断分支**：若 `images` 为空列表，说明文档不含图片，直接返回 `text_skeleton` 即可，无需进入 Phase 2。

### Phase 2：图片处理

遍历 `images` 列表，对每张图片执行以下步骤：

**Step 1 — 读取图片**

使用 Read 工具直接读取图片文件（助手会自动以视觉方式解析图片内容）：

```
工具: Read
参数: { "file_path": "<图片的绝对路径>" }
```

**Step 2 — 根据内容选择处理方式**

观察图片内容，决定采用哪种处理策略：

| 图片类型 | 处理方式 | 输出格式 |
|---|---|---|
| 含文字的图片（扫描件、截图、表格） | OCR 提取全部文字 | 用 `> ` 引用块包裹提取的文字 |
| 图表 / 流程图 / 示意图 | 描述图表内容和关键信息 | 用文字描述，保留图表标题 |
| 装饰性图片（logo、分隔线、背景） | 简短描述 | `[装饰性图片：<简短描述>]` |

**Step 3 — 替换占位符**

将 `text_skeleton` 中对应的 `![image](<path>)` 占位符替换为 Step 2 的输出内容。

示例替换：

```markdown
# 替换前（text_skeleton 中的占位符）
![image](/tmp/extracted/image_001.png)

# 替换后（OCR 结果）
> 第一段：本季度销售额同比增长 23%，其中线上渠道贡献了 65% 的增量。
> 第二段：新客户获取成本降低至 45 元/人。

# 替换后（图表描述）
**[柱状图]** 展示了 2024 年 Q1-Q4 各季度销售额对比，Q4 达到峰值 1200 万元。
```

## 智能图片选择策略

为了在处理速度和质量之间取得平衡，遵循以下策略：

### 按大小筛选

- **优先处理**：`size_bytes > 10240`（10KB 以上）的图片，这类图片通常包含有意义的文字或图表
- **跳过**：`size_bytes < 2048`（2KB 以下）的图片，多为小图标或装饰元素，替换为 `[图标]` 即可
- **按需处理**：2KB - 10KB 之间的图片，根据 `width` 和 `height` 判断是否值得 OCR

### 分批处理

- 图片数量 **20 张以内**：逐一处理，一次性完成所有图片的 OCR / 描述
- 图片数量 **20 - 100 张**：分批处理，每批 5 - 10 张，完成一批后汇总中间结果，再继续下一批
- 图片数量 **100 张以上**：
  - 仅对尺寸较大（`size_bytes > 20480`，即 20KB 以上）的图片做 OCR
  - 其余图片统一标注为 `[图片]`
  - 处理前向用户说明策略，询问是否需要调整

### 优先级排序

按以下顺序处理图片，确保最有价值的内容优先被识别：

1. `size_bytes` 最大的图片（通常包含最多信息）
2. `width * height` 像素面积最大的图片
3. 按在文档中出现的顺序处理剩余图片

## 输出格式

最终输出为一份完整的 Markdown 文档，遵循以下规范：

- **文本部分**：保留 `text_skeleton` 中的原始 Markdown 格式（标题、列表、表格等）
- **OCR 文字**：使用 `> ` 引用块标记，与正文区分
- **图表描述**：用 `**[图表类型]**` 前缀标注，后接描述文字
- **装饰性图片**：替换为 `[装饰性图片：<简短描述>]`
- **未处理的图片**：保留原始占位符 `![image](<path>)` 或替换为 `[图片]`
- **文档元信息**：在文档开头以 YAML frontmatter 形式附上 `metadata` 中的关键字段

完整输出模板：

```markdown
---
title: 文档标题
author: 作者
pages: 页数
---

# 文档标题

正文段落内容...

> OCR 提取的文字内容，来自文档中的第一张图片。
> 第二行 OCR 内容。

继续正文...

**[柱状图]** 图表描述内容...

> 表格 OCR 结果：
> | 列1 | 列2 | 列3 |
> |---|---|---|
> | 数据 | 数据 | 数据 |

[装饰性图片：公司 logo]
```

## 示例流程

假设用户要求分析一份名为 `report.docx` 的文档：

**Step 1 — 调用 analyze_document**

```
调用: mcp__markitdown__analyze_document
参数: { "file_path": "D:/docs/report.docx" }
```

返回结果（简化）：

```json
{
  "text_skeleton": "# Q4 季度报告\n\n本季度整体表现良好...\n\n![image](D:/tmp/markitdown/img_001.png)\n\n销售数据如上所示。\n\n![image](D:/tmp/markitdown/img_002.png)\n\n![image](D:/tmp/markitdown/img_003.png)\n\n总结：...\n",
  "images": [
    { "path": "D:/tmp/markitdown/img_001.png", "size_bytes": 45200, "width": 800, "height": 600 },
    { "path": "D:/tmp/markitdown/img_002.png", "size_bytes": 1200, "width": 32, "height": 32 },
    { "path": "D:/tmp/markitdown/img_003.png", "size_bytes": 28500, "width": 1200, "height": 400 }
  ],
  "metadata": { "title": "Q4 季度报告", "author": "市场部", "pages": 5 }
}
```

**Step 2 — 筛选图片**

- `img_001.png`：45KB，大图片 → 做 OCR / 描述
- `img_002.png`：1.2KB，小图标 → 跳过，标记为 `[图标]`
- `img_003.png`：28KB，大图片 → 做 OCR

**Step 3 — 读取并处理图片**

读取 `img_001.png`，识别为柱状图：

```
工具: Read
参数: { "file_path": "D:/tmp/markitdown/img_001.png" }
→ 视觉识别结果：这是一张柱状图，展示了 10-12 月销售额...
```

读取 `img_003.png`，识别为包含文字的表格：

```
工具: Read
参数: { "file_path": "D:/tmp/markitdown/img_003.png" }
→ 视觉识别结果：这是一张数据表格...
```

**Step 4 — 组装最终输出**

```markdown
---
title: Q4 季度报告
author: 市场部
pages: 5
---

# Q4 季度报告

本季度整体表现良好...

**[柱状图]** 展示了 10-12 月销售额对比：10 月 320 万元，11 月 410 万元，12 月 520 万元，呈逐月上升趋势。

销售数据如上所示。

> | 产品线 | Q3 销售额 | Q4 销售额 | 增长率 |
> |---|---|---|---|
> | A 产品 | 280 万 | 350 万 | +25% |
> | B 产品 | 190 万 | 220 万 | +16% |
> | C 产品 | 150 万 | 180 万 | +20% |

[图标]

总结：...
```

将此完整 Markdown 文档作为最终结果返回给用户。
