---
name: markitdown-convert
description: Converts documents (PDF/PPTX/DOCX/XLSX) and images to structured Markdown using markitdown-mcp MCP server + AI vision OCR. Use when the user asks to convert any document or image file to markdown, or mentions markitdown.
version: 1.0.0
---

# MarkItDown 文档转 Markdown 工作流

## 前置条件

- markitdown-mcp MCP Server 已配置并运行（STDIO mode）
- MCP 三工具：`convert_to_markdown`、`analyze_document`、`ocr_image`
- 路径一（LLM）可选：需配 `MARKITDOWN_LLM_API_KEY` / `MARKITDOWN_LLM_BASE_URL` / `MARKITDOWN_LLM_MODEL` 环境变量
- 路径二（AI助手驱动）：无需额外配置，用 AI 助手自身的 Read 视觉能力做 OCR

## Step 1: 格式分流

拿到文件后，先判断文件类型，选择对应路径：

| 文件类型 | 扩展名 | 处理方式 |
|---|---|---|
| 文档文件 | .pdf .pptx .docx .xlsx .html .csv | 走 MCP 工具 |
| 裸图片 | .png .jpg .jpeg .gif .bmp .webp | **绕过 MCP**，直接用 Read 工具视觉 OCR |
| 其他 | 未知格式 | 先尝试 convert_to_markdown，失败则提示用户 |

**关键限制**：MCP 三工具仅接受文档格式，不支持裸图片输入。

## Step 2: 判断文档类型（文档文件走此步）

调用 `analyze_document` 工具，检查返回的 `text_skeleton`：

- **text_skeleton 有丰富文本** → 纯文本/半结构化文档（如文字型 PDF、DOCX）
  - 直接用 `convert_to_markdown` 一步到位即可
  - 如果文档含图片且需要 OCR 图片内容，继续 Step 3

- **text_skeleton 仅有图片引用或几乎为空** → 扫描文档（如扫描 PDF、图片型 PPT）
  - 必须走 Step 3 的 OCR 流程

## Step 3: 图片 OCR 流程

### 3.1 收集图片

`analyze_document` 返回结果中包含图片列表，每个图片有路径和尺寸信息。图片通常保存在临时目录下。

### 3.2 过滤装饰性图片

**不是所有图片都需要 OCR**。以下类型应跳过：

| 过滤条件 | 说明 |
|---|---|
| 任一维度 >= 2000px | 通常是全页背景或装饰性底图 |
| 任一维度 <= 15px | 分割线、细线装饰 |
| 宽 <= 72px **且** 高 <= 72px **且** 文件大小 < 2KB | 小图标、装饰元素 |

**注意**：不要用 <= 120px 作为阈值，会误删有意义的小图（如 220x64 的 "PRACTICE" 徽章、112x112 的步骤图标）。72px + 2KB 双条件是经验证的安全阈值。

### 3.3 逐页 OCR

对过滤后保留的每张图片，使用 **Read 工具**读取：

```
Read(file_path="<图片绝对路径>")
```

AI 助手通过视觉能力直接"看"图片内容，提取文本。

**批量处理建议**：每次并行读取 5 张图片，避免单次请求过多导致上下文过载。

### 3.4 OCR 提取要点

读取每张图片时，确保提取以下内容：

- **正文段落**：完整还原文本，不要遗漏
- **标题/小标题**：识别层次关系，后续映射为 `#`/`##`/`###`
- **数据表格**：还原为 Markdown table 格式（`| col1 | col2 |`）
- **脚注/注释**：以 `*` 标记，放在相关段落之后
- **签名和日期**：保留原文
- **列表**：还原为有序/无序列表

## Step 4: 组装最终 Markdown

将所有 OCR 结果与 text_skeleton 的结构信息合并：

1. 用 text_skeleton 中的章节标题作为骨架
2. 将 OCR 提取的正文内容嵌入对应章节
3. 表格用标准 Markdown table 语法
4. 脚注用 `*` 标记并紧跟相关内容
5. 保持原文档的章节层次（一级标题用 `#`，二级用 `##`，以此类推）
6. 签名和日期放在文档末尾

**输出位置**：保存到与源文件相同的目录（如 `D:\` ），文件名取源文件名加 `.md` 后缀。

## Step 5: 验证

- 确认所有章节都有内容（不是空的图片引用）
- 确认表格格式正确（能渲染）
- 确认没有遗漏页面
- 用 `present_files` 展示给用户

## 常见坑

1. **Windows 短路径**：MCP 提取的图片路径可能是 8.3 短名（如 `ADMINI~1`），Read 工具无法读取。markitdown-mcp 已用 `os.path.realpath()` 修复，但如果遇到路径问题需检查。

2. **扫描 PDF 的 text_skeleton 为空是正常的**：这不是报错，说明 PDF 每页都是图片。直接走 OCR 流程即可。

3. **图片过滤阈值不能太激进**：曾经用 < 120px & < 3KB 做阈值，结果误删了有意义的内容图。严格用 72px + 2KB 双条件。

4. **MCP Server 修改依赖后必须重启**：旧进程不会加载新安装的 Python 包。

5. **裸图片不要用 MCP**：MCP 工具不支持 PNG/JPG 等裸图片输入，直接用 Read 视觉即可。

## 路径一 vs 路径二 选择指南

| 场景 | 推荐路径 |
|---|---|
| 用户已配置 LLM 环境变量 | 路径一：全自动，MCP 内部调用 LLM 描述图片 |
| 用户未配置 LLM（默认情况） | 路径二：AI 助手 Read 视觉 OCR |
| 纯文本文档（无图片） | 直接 convert_to_markdown，两条路都不需要 |
| 批量文档处理 | 路径一更高效（无需逐页 Read） |
| 高质量要求（表格、脚注、签名） | 路径二更灵活，AI 助手能理解文档结构 |
