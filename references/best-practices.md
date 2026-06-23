# Multimodal Bridge 使用技巧

## 目录结构建议

在项目中使用多模态桥接时，建议的目录布局：

```
my-project/
├── picture/              # 存放原始图片/文档
│   ├── screenshot.png
│   ├── architecture.jpg
│   └── report.pdf
├── document/             # 存放提取的 Markdown
│   ├── screenshot.md
│   ├── architecture.md
│   ├── report.md
│   └── index.md          # 批量处理时的索引文件
└── ...
```

## 最佳实践

### 图片质量
- 截图时尽量保持清晰，避免模糊或过低分辨率
- 包含文字的图片，确保文字可读
- 大尺寸图片（>10MB）建议先压缩，API 有大小限制

### 文档处理
- PDF 中的扫描件会以图片形式发给 mimo 2.5，效果依赖图片质量
- 表格密集的 Excel 建议先手动筛选需要的 sheet，减少噪音
- 大型文档（>50页）建议分段处理

### 路径规范
- 使用相对路径：`picture/screenshot.png`（推荐）
- 也支持绝对路径：`E:/files/image.png`
- 路径中避免特殊字符和中文空格（中文文件名可以）

### 批量处理
- 超过 5 个文件时，建议生成 `document/index.md` 索引
- 索引文件包含每个文件的名称、类型和一句话摘要

## 常见问题

### Q: API 调用超时
A: 默认超时 120 秒。大图片可能需要更长时间，脚本会自动重试不同的 endpoint 路径。

### Q: 识别效果不好
A: 尝试用更清晰的原图，或者在 `--prompt` 参数中给出更具体的提示词。例如：
```bash
python vision_api.py --file picture/table.png --output document/table.md \
  --prompt "重点识别表格数据，转为精确的 Markdown 表格格式"
```

### Q: 如何使用其他 API
A: 通过 `--config` 参数指定自定义配置文件：
```bash
python vision_api.py --file img.png --output doc/img.md --config ./my_config.json
```

## 输出格式示例

### 图片输出
```markdown
# 图片内容提取

- **源文件**: `picture/screenshot.png`
- **内容类型**: UI 截图

## 视觉描述

这是一个登录页面的截图，包含...

## 文字内容

- 用户名输入框
- 密码输入框
- [登录] 按钮

## 关键信息

- 页面标题：用户登录
- 包含"忘记密码"链接
```

### 文档输出
```markdown
# 文档内容提取

- **源文件**: `report.pdf`
- **页数**: 12 页

## 摘要

本报告介绍了...

## 正文

### 第一章 ...
```
