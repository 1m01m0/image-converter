# AI Skills — 格式转换工具集

适用于 Claude Code / Claude Cowork / OpenCode / Codex 的格式转换 Skill 集合。

## Skills 列表

### 1. 图片格式转换 (`image-converter`)

PNG / JPEG / WebP 互转，支持质量调节、缩放、批量处理。

### 2. 文档格式转换 (`document-converter`)

DOCX / PPTX / XLSX ↔ PDF 互转。Office → PDF 用 LibreOffice，PDF → Office 用 Python 库。

| 方向 | 引擎 | 质量 |
|------|------|------|
| DOCX/PPTX/XLSX → PDF | LibreOffice | 优秀 |
| PDF → DOCX | pdf2docx | 中等，需手动检查 |
| PDF → XLSX（表格） | pdfplumber | 中等 |
| PDF → PPTX | fitz 提取文本 | 低，仅提取文字 |

---

## 安装方法（按平台）

### Claude Code（命令行终端）

先克隆仓库，然后在**终端**中执行：

```bash
git clone https://github.com/1m01m0/format-converter.git
cd format-converter

# 安装图片转换 skill
claude skills install ./image-converter.skill

# 安装文档转换 skill
claude skills install ./document-converter.skill
```

也可以直接通过 URL 安装（无需克隆）：

```bash
claude skills install https://github.com/1m01m0/format-converter/blob/main/image-converter.skill
claude skills install https://github.com/1m01m0/format-converter/blob/main/document-converter.skill
```

安装后在 **Claude Code 对话中**输入 `/image-converter` 或 `/document-converter` 触发。

### Claude Cowork（桌面应用）

在 **Cowork 聊天框**中直接拖入 `.skill` 文件即可安装。安装后在对话中输入 `/image-converter` 或 `/document-converter` 触发。

### OpenCode

```bash
# 在终端执行，两个 skill 都要装就两条都跑
unzip image-converter.skill -d ~/.opencode/skills/image-converter/
unzip document-converter.skill -d ~/.opencode/skills/document-converter/
```

### Codex

```bash
# 在终端执行
unzip image-converter.skill -d ~/.codex/skills/image-converter/
unzip document-converter.skill -d ~/.codex/skills/document-converter/
```

---

## 文件说明

| 文件 | 用途 |
|------|------|
| `image-converter.skill` | 图片转换 Skill 安装包 |
| `image-converter.html` | 图片转换浏览器版工具（双击即用） |
| `document-converter.skill` | 文档转换 Skill 安装包 |
| `skill/SKILL.md` | 图片转换 Skill 定义（可单独阅读） |
| `skill-docs/SKILL.md` | 文档转换 Skill 定义（可单独阅读） |
