# AI Skills — 格式转换工具集

适用于 Claude Code / Claude Cowork / OpenCode / Codex 的格式转换 Skill 集合。

## Skills 列表

### 1. 图片格式转换 (`image-converter`)

PNG / JPEG / WebP 互转，支持质量调节、缩放、批量处理。

**安装：**
```bash
claude skills install ./image-converter.skill
# 或
claude skills install https://github.com/1m01m0/image-converter/blob/main/image-converter.skill
```

**使用：** `/image-converter` — 把这张图转成 JPEG 质量 80% / 把文件夹里的 PNG 批量转 WebP

**另附：** `image-converter.html` 浏览器版独立工具，双击打开即用。

### 2. 文档格式转换 (`document-converter`)

DOCX / PPTX / XLSX ↔ PDF 互转。Office → PDF 用 LibreOffice，PDF → Office 用 Python 库。

**安装：**
```bash
claude skills install ./document-converter.skill
# 或
claude skills install https://github.com/1m01m0/image-converter/blob/main/document-converter.skill
```

**使用：** `/document-converter` — 把这个 Word 导出成 PDF / 把这个 PDF 转成可编辑的 DOCX

**转换方向：**

| 方向 | 引擎 | 质量 |
|------|------|------|
| DOCX/PPTX/XLSX → PDF | LibreOffice | 优秀 |
| PDF → DOCX | pdf2docx | 中等，需手动检查 |
| PDF → XLSX（表格） | pdfplumber | 中等 |
| PDF → PPTX | fitz 提取文本 | 低，仅提取文字 |

## 手动安装（OpenCode / Codex）

```bash
unzip image-converter.skill -d ~/.opencode/skills/image-converter/
unzip document-converter.skill -d ~/.opencode/skills/document-converter/
```

## 文件说明

| 文件 | 用途 |
|------|------|
| `image-converter.skill` | 图片转换 Skill 安装包 |
| `image-converter.html` | 图片转换浏览器版工具 |
| `document-converter.skill` | 文档转换 Skill 安装包 |
| `skill/SKILL.md` | 图片转换 Skill 定义 |
| `skill-docs/SKILL.md` | 文档转换 Skill 定义 |
