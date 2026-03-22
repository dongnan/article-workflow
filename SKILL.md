# Article Workflow Skill

## 描述

文章分析工作流自动化 Skill。提供从文章抓取、分析、归档到质量评分的完整流程，支持 URL 去重、Heartbeat 自动触发、监控日志等功能。

> **注意：** 本 Skill 已合并原 `article-analyzer` 功能，推荐使用本 Skill。

## 🔒 安全说明

**数据边界：**
- ✅ 所有数据文件在 `skills/article-workflow/data/` 内
- ✅ 所有日志文件在 `skills/article-workflow/logs/` 内
- ✅ 不访问工作区外文件
- ✅ 路径验证确保所有操作在 Skill 目录内

**凭证要求：**
- ✅ 需要 `BITABLE_APP_TOKEN` 和 `BITABLE_TABLE_ID`
- ✅ 凭证通过 `config.json` 或环境变量提供
- ✅ 敏感信息不会被提交到版本控制（.gitignore）

**运行模式：**
- **独立模式** - 仅 CLI 功能（去重检查、状态查看）
- **集成模式** - 需要 OpenClaw 环境（完整功能）

## 使用场景

当用户需要：
- 分析文章 URL（微信、知乎、GitHub 等）
- 自动生成摘要、标签、分析报告
- 将分析结果归档到飞书多维表格和文档
- 对文章进行质量评分（S/A/B/C/D）
- 自动检查重复 URL
- 定时自动处理群聊中的文章链接
- **批量分析多篇文章**（并发执行）

## 🚀 智能路由模式

### 单篇模式（默认）

**触发：** 单篇文章 URL

```
分析这篇文章：https://example.com/article
```

**执行方式：** 主 Agent 一次性执行（1 次流式请求）

```
主 Agent（机器人）
  ├─ 流式请求开始 ─────┐
  │  1. web_fetch 抓取   │
  │  2. 自己分析内容     │  ← 同一次流式请求
  │  3. 生成完整报告     │
  │  4. feishu_create_doc│
  │  5. feishu_bitable   │
  └─ 流式请求结束 ─────┘
  
模型请求次数：1 次 ✅
```

### 批量模式（自动）

**触发：** 多篇文章 URL

```
批量分析这些文章：
- https://example.com/article1
- https://example.com/article2
- https://example.com/article3
```

**执行方式：** SubAgent 并发执行（N 次但并行）

```
主 Agent（机器人）
  ├─ 创建 3 个 SubAgent（并发）
  │   ├─ SubAgent-1: 分析 url1  →  1 次流式请求
  │   ├─ SubAgent-2: 分析 url2  →  1 次流式请求
  │   └─ SubAgent-3: 分析 url3  →  1 次流式请求
  └─ 汇总结果
  
模型请求次数：3 次（但并发执行，总时间 ≈ 1 次）✅
```

### 并发控制

- **最大并发数：** 5 个 SubAgent
- **超过限制：** 自动分批处理
- **流式优化：** 工具调用不中断流式，算 1 次请求

---

## 工作流程

### 单篇模式（完整版）

```
输入：文章 URL
  ↓
主 Agent 智能路由（单篇模式）
  ├─ 1. 图片抓取（如有配图）：
  │   ├─ browser 打开页面 + 分析 HTML 获取图片 URL
  │   ├─ curl 下载原图到本地
  │   └─ 保存到 docs/article/yyyy-mm/images/
  ├─ 2. 内容抓取：
  │   ├─ web_fetch 抓取文章内容
  │   └─ browser 补充抓取（如需要）
  ├─ 3. 内容分析：
  │   ├─ 提取核心观点
  │   ├─ 生成简短摘要
  │   └─ 质量评分（S/A/B/C/D）
  ├─ 4. 本地保存（新规则）：
  │   ├─ 创建 docs/article/yyyy-mm/yyyy-mm-dd_文章标题.md
  │   └─ 写入完整分析报告
  ├─ 5. 飞书文档同步：
  │   ├─ feishu_create_doc 创建飞书文档
  │   ├─ 图片上传飞书云盘（feishu_drive_file）
  │   ├─ 图片插入飞书文档（feishu_doc_media）
  │   └─ feishu_update_doc 更新飞书文档内容
  ├─ 6. 多维表格归档：
  │   ├─ feishu_bitable 创建记录
  │   └─ 填写标题/摘要/标签/链接等字段
  └─ 7. Git 提交：
      ├─ git add docs/article/
      ├─ git commit -m "docs: 分析文章《标题》"
      └─ git push
  ↓
输出：摘要 + 本地路径 + 飞书文档链接 + 评分
```

### 批量模式

```
输入：多篇文章 URL
  ↓
主 Agent 智能路由（批量模式）
  ├─ 创建 SubAgent #1 ─→ 分析文章 1（含图片处理）
  ├─ 创建 SubAgent #2 ─→ 分析文章 2（含图片处理）
  ├─ 创建 SubAgent #3 ─→ 分析文章 3（含图片处理）
  └─ 等待所有 SubAgent 完成
      ↓
    汇总所有结果
  ↓
输出：汇总报告 + 各文章链接（含图）
```

## 🔧 配置管理

### 配置保护机制

**重要：** `config.json` 包含敏感信息，已加入 `.gitignore`，不会被 Git 提交。

**在修改/升级 Skill 前：**
```bash
cd skills/article-workflow

# 1. 备份配置（修改前必做）
python3 scripts/config_manager.py backup

# 或使用脚本
./scripts/backup-config.sh
```

**修改/升级后：**
```bash
# 恢复配置
python3 scripts/config_manager.py restore

# 或使用脚本
./scripts/restore-config.sh
```

### 首次使用配置

**方式 1：配置向导（推荐）**
```bash
python3 scripts/config_manager.py guide
```

**方式 2：手动创建**
```bash
# 复制示例配置
cp config.example.json config.json

# 编辑配置
vim config.json
```

**方式 3：环境变量**
```bash
export BITABLE_APP_TOKEN=your_token
export BITABLE_TABLE_ID=your_table_id
```

### 配置参数

在 `config.json` 中配置：

```json
{
  "bitable": {
    "app_token": "YOUR_BITABLE_TOKEN",
    "table_id": "YOUR_TABLE_ID"
  },
  "workflow": {
    "check_interval_hours": 6,
    "batch_limit": 10,
    "enable_quality_score": true,
    "enable_url_dedup": true
  },
  "paths": {
    "data": "./data",
    "logs": "./logs"
  }
}
```

## 命令

### 分析单篇文章

**飞书单聊（推荐）：**

```
分析这篇文章：https://example.com/article
```

**飞书群聊：**

```
分析这篇文章：https://example.com/article
```

单聊时无需@机器人，直接发送即可。

### 查看状态

```bash
cd skills/article-workflow
./scripts/monitor.sh status
```

### 生成周报

```bash
./scripts/monitor.sh report
```

### 清理数据

```bash
./scripts/monitor.sh cleanup
```

## 输出格式

### 群聊消息

```
✅ 文章分析完成

📌 [文章标题]
🔗 [URL]

📝 简短摘要
[3-5 句摘要]

🏷️ 标签：[标签 1] [标签 2] [标签 3]
⭐ 重要程度：[高/中/低]
📊 质量评分：[85/100] (A 级)

📄 详细报告：[飞书文档链接]
📊 已录入：[Bitable 链接]
```

---

## 🖼️ 图片处理流程（新增）

当文章包含重要配图时，按以下流程处理：

### Step 1: 分析文章 HTML 获取图片路径（推荐）

```bash
# 使用 browser 工具打开文章
browser(action="open", url="文章 URL")

# 提取所有图片的真实路径
browser(action="act", kind="evaluate", fn="""
() => {
  const imgs = document.querySelectorAll('img');
  const imgUrls = [];
  imgs.forEach((img, i) => {
    if(img.src && img.src.includes('mmbiz')) {
      imgUrls.push({index: i, src: img.src, alt: img.alt || ''});
    }
  });
  return JSON.stringify(imgUrls, null, 2);
}
""")
# 返回所有图片的真实 URL 列表

# 使用 curl 直接下载图片（推荐，避免截图失真）
curl -o docs/reference/yyyy-mm/images/{filename}.jpg "图片 URL"
```

### Step 2: 浏览器截图（备用方案）

```bash
# 如果无法直接下载，使用 browser 工具截取完整页面
browser(action="screenshot", fullPage=true, type="png")
# 返回：MEDIA:/Users/dongnan/.openclaw/media/browser/{uuid}.png

# 如果文章有图片轮播/分页，点击下一页继续截图
browser(action="act", kind="click", ref="e32")  # 点击第 2 张图
browser(action="screenshot", fullPage=true, type="png")  # 截图

# 重复直到所有图片都截取完成
```

### Step 2: 创建本地目录

```bash
# 创建文档和圖片目录
mkdir -p docs/reference/{yyyy-mm}/images/
```

### Step 3: 上传飞书云盘

```bash
# 复制图片到临时目录（飞书允许的路径）
cp /Users/dongnan/.openclaw/media/browser/{uuid}.png /var/folders/.../T/{filename}.png

# 使用 feishu_drive_file 上传
feishu_drive_file(action="upload", file_path="/var/folders/.../T/{filename}.png")
# 返回：file_token, url
```

### Step 4: 插入飞书文档

```bash
# 使用 feishu_doc_media 插入图片到文档
feishu_doc_media(
  action="insert",
  doc_id="{doc_id}",
  file_path="/var/folders/.../T/{filename}.png",
  type="image"
)
# 返回：block_id, file_token
```

### Step 5: 本地保存

```bash
# 复制图片到工作区目录
cp /Users/dongnan/.openclaw/media/browser/{uuid}.png docs/reference/{yyyy-mm}/images/{filename}.png

# Git 提交
git add docs/reference/{yyyy-mm}/
git commit -m "docs: 新增 {文章标题} 分析报告（含配图）"
git push
```

### Step 6: 图片内容识别（新增）

```bash
# 使用 browser 工具截图后，分析图片中的文字内容
# 对于微信公众号文章，通常有图片轮播/分页，需要：
# 1. 点击每张图片进行截图
# 2. 识别图片中的文字内容
# 3. 整理到分析报告中

# 示例：微信公众号图片轮播
browser(action="act", kind="click", ref="e32")  # 点击第 1 张
browser(action="screenshot", fullPage=true)     # 截图
browser(action="act", kind="click", ref="e33")  # 点击第 2 张
browser(action="screenshot", fullPage=true)     # 截图
# ... 重复直到所有图片都截取
```

### Step 7: 图片内容分析（新增）

```bash
# 对于每张截图，需要分析图片中的文字内容
# 方法 1：使用 browser snapshot 获取页面文本
# 方法 2：使用 OCR 工具识别图片文字
# 方法 3：人工查看截图并整理内容

# 分析内容应包括：
# - 图片标题
# - 关键信息列表
# - 数据表格
# - 技能名称和说明
# - 其他重要文字内容

# 将分析结果整理到 Markdown 报告中
```

### 目录结构

```
docs/reference/2026-03/
├── 2026-03-22_文章标题_分析报告.md  ← Markdown 文档（含相对路径图片引用）
└── images/
    ├── 2026-03-22_文章主图.png      ← 主图
    ├── 2026-03-22_图片 1.png        ← 第 1 张详细内容图
    ├── 2026-03-22_图片 2.png        ← 第 2 张详细内容图
    └── ...
```

### Markdown 图片引用

```markdown
## 📊 文章主图

![文章主图说明](images/2026-03-22_文章主图.png)

> 配图说明：...

## 📊 详细内容 - 图片 1

![图片 1 说明](images/2026-03-22_图片 1.png)

**图片内容分析：**
- 标题：...
- 关键信息：...
- 技能列表：...
```

### 注意事项

1. **飞书文档图片** — 必须通过 `feishu_doc_media` 插入，不能使用本地路径
2. **本地 Markdown** — 使用相对路径引用 `images/xxx.png`
3. **临时文件** — 上传云盘后需清理 `/var/folders/.../T/` 下的临时文件
4. **Git 提交** — 图片和 Markdown 文档一起提交，确保版本同步
5. **文件命名** — 使用 `{日期}_{文章标题}_{图片类型}.png` 格式，避免中文乱码
6. **多图抓取** — 微信公众号文章通常有图片轮播，需要点击每张图片进行截图
7. **图片内容识别** — 截图后需要人工或 AI 识别图片中的文字内容，整理到报告中

---

## 🔧 配置管理

### Bitable 字段映射

**必填字段：**
| 字段名 | 说明 | 来源 |
|--------|------|------|
| **文章标题（主）** | 主字段（显示用） | 文章标题 |
| **标题** | 副标题（兼容用） | 文章标题（与主字段相同） |
| **简短摘要** | 3-5 句摘要 | LLM 生成 |
| **阅读日期** | 分析日期 | 当前时间戳 |
| **来源** | 文章来源 | 微信/GitHub/知乎等 |
| **关键词标签** | 多选标签 | LLM 提取 + 匹配现有选项 |
| **重要程度** | 高/中 | 根据质量评分 |
| **状态** | 已完成 | 固定值 |
| **创建方式** | 手动触发/自动分析 | 根据触发方式 |
| **URL 链接** | 原文链接 | 用户提供的 URL |
| **详细报告链接** | 飞书文档链接 | feishu_create_doc 返回 |
| **封面图片** | 文章主图（可选） | 飞书云盘 file_token |

**注意：**
- "文章标题（主）"和"标题"都需要填写，确保数据完整性
- 如有配图，需上传飞书云盘后记录 file_token 到"封面图片"字段

### 质量评分

| 总分 | 等级 | 重要程度 | 处理策略 |
|------|------|---------|---------|
| 85-100 | S 级 | 极高 | 立即处理 + 团队分享 |
| 70-84 | A 级 | 高 | 优先处理 + 详细分析 |
| 55-69 | B 级 | 中 | 正常处理 + 标准报告 |
| 40-54 | C 级 | 低 | 简略处理 + 基础摘要 |
| 0-39 | D 级 | 极低 | 跳过或仅存档 |

## 依赖

- Python 3.7+
- 飞书开放平台 API
- OpenClaw 框架

## 安装

```bash
cd ~/.openclaw/workspace/skills
git clone <repo_url> article-workflow
cd article-workflow
./install.sh
```

## 文件结构

```
article-workflow/
├── SKILL.md                    # 本文件
├── README.md                   # 详细文档
├── install.sh                  # 安装脚本
├── config.example.json         # 配置模板
├── scripts/                    # 可执行脚本
│   ├── check_url_dup.py        # URL 去重
│   ├── monitor.sh              # 监控
│   └── workflow.py             # 主流程
├── docs/                       # 文档
│   ├── config.md               # 配置说明
│   ├── quality-score.md        # 评分标准
│   └── automation.md           # 自动化
├── data/                       # 运行时数据
│   └── url_cache.json
└── logs/                       # 日志
    ├── workflow.log
    └── error.log
```

## 注意事项

1. **首次使用**需要配置 `config.json`
2. **飞书授权**需要完成 OAuth 流程
3. **Heartbeat 自动触发**需要在 HEARTBEAT.md 中配置
4. **日志文件**建议定期清理（>30 天）
5. **URL 缓存**保留最近 1000 条记录
6. **图片处理**（新增）：
   - 飞书文档图片必须通过 `feishu_doc_media` 插入，不能使用本地路径
   - 本地 Markdown 使用相对路径引用 `images/xxx.png`
   - 上传云盘前需复制图片到 `/var/folders/.../T/` 临时目录
   - 图片和文档一起 Git 提交，确保版本同步
7. **文件命名**：
   - 使用 `{日期}_{文章标题}_{图片类型}.png` 格式
   - 避免特殊字符和过长文件名
   - 中文标题建议使用拼音或英文缩写

## 故障排查

### URL 去重不生效
- 检查 `data/url_cache.json` 是否存在
- 验证 Python 脚本权限：`chmod +x scripts/check_url_dup.py`

### 监控脚本报错
- 检查 Bash 版本：需要 4.0+
- 验证路径配置

### 质量评分偏差
- 调整 `docs/quality-score.md` 中的权重
- 优化 LLM 提示词

## 版本

- **当前版本：** 1.0.0
- **最后更新：** 2026-03-14
- **作者：** Nox（DongNan 的 AI 助理）

## License

MIT License
