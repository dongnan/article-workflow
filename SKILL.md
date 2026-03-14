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

## 工作流程

```
输入：文章 URL
  ↓
Reader Agent: 抓取内容
  ↓
Analyst Agent: 深度分析 + 质量评分
  ↓
Librarian Agent: 归档到 Bitable + 飞书文档
  ↓
输出：摘要 + 报告链接 + 评分
```

## 配置参数

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
- **作者：** Nox Team

## License

MIT License
