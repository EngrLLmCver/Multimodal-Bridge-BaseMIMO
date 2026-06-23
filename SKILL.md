---
name: multimodal-bridge
description: "多模态内容桥接技能。当用户输入中包含任何图片或文档的文件路径时，必须立即使用此技能。支持格式：图片（PNG/JPG/GIF/BMP/WebP/TIFF）、文档（PDF/DOCX/XLSX/PPTX）、矢量图（SVG）。触发场景：用户粘贴了文件路径、说'识别图片'、'读取这个文档'、'OCR'、'提取文字'、'看看这个文件'、'帮我看看这个'、'这个文件里写了什么'，或者任何涉及 .png/.jpg/.pdf/.docx/.xlsx/.pptx 等扩展名的请求。即使用户只是随口提到一个文件路径，只要指向的是图片或文档，就应触发。本技能通过独立脚本调用 mimo 2.5 多模态 API，不将图片数据塞入当前模型上下文。重要：不要尝试用 Read 工具直接读取图片文件，那会导致图片数据进入上下文报错。"
---

# Multimodal Bridge — 多模态内容桥接

## 为什么需要这个技能

当前底层模型 mimo 2.5 pro 是纯文本模型，不支持多模态。如果直接把图片数据塞进上下文会报错。本技能的解决方案：通过独立的 Python 脚本调用 mimo 2.5（多模态版本）的 API，将图片/文档内容提取为 Markdown 文本，存入项目的 `document/` 目录。这样 mimo 2.5 pro 只需要读取纯文本就能理解多模态内容。

**重要：不要用 Read 工具直接读取图片！** Read 工具读取图片后，图片数据会进入当前上下文发给底层模型，mimo 2.5 pro 不支持多模态会报错。必须通过本技能的脚本来处理。

## 工作流程

### Step 1: 确认项目目录

检查当前项目根目录下是否有 `picture/` 和 `document/` 目录，没有就创建：

```bash
mkdir -p picture document
```

### Step 2: 运行提取脚本

使用 `D:/anaconda3/python.exe` 调用脚本（该环境已安装所有依赖）。

首先确定脚本的实际路径（二选一，哪个存在用哪个）：
- `C:\Users\Administrator\.claude\skills\multimodal-bridge\scripts\vision_api.py`
- `d:\Claude_Data\skills\multimodal-bridge\scripts\vision_api.py`

然后运行：

```bash
D:/anaconda3/python.exe "<脚本实际路径>" \
  --file "<用户提供的文件路径>" \
  --output "document/<输出文件名>.md"
```

参数说明：
- `--file`：用户提供的原始文件路径（图片或文档）
- `--output`：Markdown 输出路径，放在 `document/` 目录下
- `--config`（可选）：配置文件路径，默认读取 `C:\Users\Administrator\.claude\settings.json`
- `--prompt`（可可选）：自定义提示词，覆盖默认

脚本会自动：
1. 从 settings.json 读取 API 地址和密钥
2. 判断文件类型（图片 vs 文档）
3. 图片：base64 编码后调用 mimo 2.5 vision API
4. 文档：先用对应 Python 库提取内容，再发给 mimo 2.5 整理
5. 将结果写入指定的 .md 文件

### Step 3: 读取结果并回复

脚本运行后，用 Read 工具读取生成的 Markdown 文件，基于内容回答用户问题。

回复末尾加上提示：`[多模态内容已提取至 document/<filename>.md]`

## 文件命名规则

| 源文件 | 输出文件 |
|--------|----------|
| `picture/screenshot.png` | `document/screenshot.md` |
| `report.pdf` | `document/report.md` |
| `data.xlsx` | `document/data.md` |
| `icon.svg` | `document/icon.md` |

同名文件已存在时，添加数字后缀：`screenshot_2.md`、`screenshot_3.md`。

## 批量处理

用户一次提供多个文件时：
1. 逐个调用脚本处理
2. 每个文件独立保存为一个 `.md`
3. 额外生成 `document/index.md` 索引，列出所有已处理文件及摘要

## 错误处理

- 脚本报错时，将错误信息展示给用户
- API 调用失败时，提示用户检查网络或 API 配置
- 文件不存在时，提示用户确认路径

## 配置说明

脚本从 `C:\Users\Administrator\.claude\settings.json` 读取：
- `env.ANTHROPIC_BASE_URL`：API 地址
- `env.ANTHROPIC_AUTH_TOKEN`：API 密钥

如需使用其他配置，通过 `--config` 参数指定。

详细使用技巧见 `references/best-practices.md`。
