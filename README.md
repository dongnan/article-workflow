# Article Workflow Skill - 完整使用指南

## 📖 简介

Article Workflow 是一个完整的文章分析自动化 Skill，提供从文章抓取、分析、归档到质量评分的一站式解决方案。

## ✨ 核心功能

1. **文章分析** - 自动抓取、总结、标签、生成详细报告
2. **质量评分** - 4 维度评分系统（内容价值、技术深度、适配性、可读性）
3. **URL 去重** - 智能检查重复文章，避免重复处理
4. **自动触发** - Heartbeat 定时检查群聊消息
5. **监控日志** - 完整的日志记录和周报生成
6. **知识归档** - 自动写入飞书多维表格和文档

## 🚀 快速开始

### 1. 安装

```bash
cd ~/.openclaw/workspace/skills
./install.sh  # 或手动复制
```

### 2. 配置

复制配置模板：

```bash
cd skills/article-workflow
cp config.example.json config.json
```

编辑 `config.json`：

```json
{
  "bitable": {
    "app_token": "你的多维表格 Token",
    "table_id": "表 ID"
  },
  "workflow": {
    "check_interval_hours": 6,
    "batch_limit": 10,
    "enable_quality_score": true,
    "enable_url_dedup": true
  }
}
```

### 3. 使用

**飞书单聊（推荐）：**

```
分析这篇文章：https://mp.weixin.qq.com/s/xxx
```

**飞书群聊：**

```
分析这篇文章：https://mp.weixin.qq.com/s/xxx
```

单聊时无需@机器人，直接发送即可。群聊时根据机器人配置可能需要@。

## 📊 质量评分标准

### 评分维度（100 分）

| 维度 | 权重 | 说明 |
|------|------|------|
| 内容价值 | 40 分 | 信息密度、原创性、时效性、权威性 |
| 技术深度 | 30 分 | 代码质量、原理分析、数据支撑 |
| 适配性 | 20 分 | 相关性、技术栈匹配、成本 |
| 可读性 | 10 分 | 结构、排版、图表、语言 |

### 评分等级

| 总分 | 等级 | 重要程度 | 处理策略 |
|------|------|---------|---------|
| 85-100 | S 级 | 极高 | 立即处理 + 团队分享 |
| 70-84 | A 级 | 高 | 优先处理 + 详细分析 |
| 55-69 | B 级 | 中 | 正常处理 + 标准报告 |
| 40-54 | C 级 | 低 | 简略处理 + 基础摘要 |
| 0-39 | D 级 | 极低 | 跳过或仅存档 |

## 🔧 高级配置

### Heartbeat 自动触发

编辑 `~/.openclaw/workspace/HEARTBEAT.md`：

```markdown
## 定期任务清单

### 每 6 小时
- [ ] 检查群聊未处理文章链接 → article-workflow
```

### 监控命令

```bash
# 查看状态
./scripts/monitor.sh status

# 生成周报
./scripts/monitor.sh report

# 清理数据
./scripts/monitor.sh cleanup
```

### URL 去重

```bash
# 检查 URL
python3 scripts/check_url_dup.py "https://example.com/article"

# 添加到缓存
python3 scripts/check_url_dup.py "https://example.com/article" \
  --add "标题" "record_id" "https://doc.url"
```

## 📁 文件结构

```
article-workflow/
├── SKILL.md                    # Skill 定义
├── README.md                   # 本文档
├── install.sh                  # 安装脚本
├── config.example.json         # 配置模板
├── scripts/                    # 脚本
│   ├── check_url_dup.py        # URL 去重
│   ├── monitor.sh              # 监控
│   └── workflow.py             # 主流程
├── docs/                       # 文档
│   ├── config.md               # 配置说明
│   ├── quality-score.md        # 评分标准
│   └── automation.md           # 自动化
├── data/                       # 数据（.gitignore）
│   └── url_cache.json
└── logs/                       # 日志（.gitignore）
    ├── workflow.log
    └── error.log
```

## 🔒 安全说明

**环境变量配置：**

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件，填入实际值
vi .env
```

**注意：**
- ✅ `.env` 文件已在 `.gitignore` 中，不会被提交
- ✅ `config.json` 包含敏感信息，请勿提交到版本控制
- ✅ 生产环境请使用环境变量而非硬编码

## 🐛 故障排查

### 常见问题

**Q: URL 去重不生效？**
```bash
# 检查缓存文件
ls -la data/url_cache.json

# 验证脚本权限
chmod +x scripts/check_url_dup.py
```

**Q: 监控脚本报错？**
```bash
# 检查 Bash 版本
bash --version  # 需要 4.0+

# 检查 Python 版本
python3 --version  # 需要 3.7+
```

**Q: 飞书 API 调用失败？**
- 检查 OAuth 授权
- 验证 app_token 和 table_id
- 确认配置文件路径正确

## 📈 性能指标

| 指标 | 目标值 |
|------|-------|
| 处理时间 | < 2 分钟/篇 |
| 成功率 | > 95% |
| 重复率 | < 5% |

## 🔄 更新日志

### v1.0.0 (2026-03-14)

- ✅ 初始版本
- ✅ 文章分析工作流
- ✅ 质量评分系统
- ✅ URL 去重
- ✅ 监控日志

## 📝 License

MIT License
