# 图片格式转换工具

一个纯浏览器端的图片格式转换工具，支持 PNG / JPEG / WebP 互转。另提供 AI 编码助手 Skill，可在 Claude Code / OpenCode / Codex 中安装使用。

## 浏览器版

直接在浏览器中打开 `image-converter.html` 即可使用，无需安装任何依赖。

**功能：**
- 拖拽或点击上传图片
- 输出格式：PNG、JPEG、WebP
- 批量转换
- 可调节输出质量（JPEG / WebP）
- 按比例缩放或自定义尺寸
- 转换后预览对比，显示体积变化

## AI Skill 版

### 安装

**Claude Code / Claude Cowork：**
```bash
# 方式一：直接安装 .skill 文件
claude skills install ./image-converter.skill

# 方式二：从 GitHub 安装
claude skills install https://github.com/1m01m0/image-converter/blob/main/image-converter.skill
```

**OpenCode / Codex（手动安装）：**
```bash
# 解压 .skill 文件到 skills 目录
unzip image-converter.skill -d ~/.opencode/skills/image-converter/
# 或
unzip image-converter.skill -d ~/.codex/skills/image-converter/
```

### 使用

安装后，在对话中输入 `/image-converter`，然后描述需求：

```
/ 把这张图转成 JPEG，质量调到 80%
/ 把这个文件夹里的 PNG 批量转成 WebP
/ 把 screenshot.png 缩放到 50% 再转成 JPEG
```

Skill 会自动使用 Python Pillow 完成转换，处理透明度、输出对比信息。

## 文件说明

| 文件 | 用途 |
|------|------|
| `image-converter.html` | 浏览器版独立工具，双击打开即用 |
| `image-converter.skill` | AI Skill 安装包（zip 格式） |
| `skill/SKILL.md` | Skill 定义文件（可单独查看） |
| `skill/image-converter.html` | Skill 附带的浏览器工具 |
