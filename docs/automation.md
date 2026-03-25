# AI-TIMELINE 自动化更新指南

## 自动化脚本

`/Users/zhimakaimen/.openclaw/workspace/scripts/update-ai-timeline.js`

这是一个 Node.js 脚本，用于自动化 AI-TIMELINE 项目的更新流程。

## 功能

- 自动添加新记录文件到 git
- 生成标准格式的 commit message (YYYY-MM-DD HH:MM - 事件描述)
- 更新 `CHANGELOG.md` 的对应日期区块
- 更新 `README.md` 的最新记录表格
- 自动提交并推送到 GitHub

## 使用方法

### 基本用法

```bash
# 克隆 AI-TIMELINE 项目（如果还没有）
cd ~
git clone git@github.com:LoveFromIO/AI-TIMELINE.git

# 创建新记录文件（遵循命名规范）
# 文件名格式：YYYY-MM-DD-事件简短描述.md

# 使用脚本更新
node /Users/zhimakaimen/.openclaw/workspace/scripts/update-ai-timeline.js 2026/2026-03-25-new-event.md "事件简要描述"
```

### 示例

```bash
# 1. 先创建文件
mkdir -p ~/AI-TIMELINE/2026
cat > ~/AI-TIMELINE/2026/2026-03-25-test-event.md << 'EOF'
---
title: "2026-03-25 - 测试事件"
date: 2026-03-25
category: 测试
status: published
tags: [test]
author: Eve
---

# 测试事件

内容...
EOF

# 2. 运行自动化脚本
cd ~/AI-TIMELINE
node /Users/zhimakaimen/.openclaw/workspace/scripts/update-ai-timeline.js 2026/2026-03-25-test-event.md "添加测试事件记录"

# 脚本会自动：
# - 添加文件到 git
# - 更新 CHANGELOG.md（在对应日期下添加条目）
# - 更新 README.md 的最新记录表格
# - 提交 commit
# - 推送到 origin/main
```

## 文件命名规范

必须遵循：`YYYY-MM-DD-事件简短描述.md`

例如：
- ✅ `2026-03-25-litellm-security-check.md`
- ✅ `2026-03-25-motion-data-sync.md`
- ❌ `2026-03-25安全事件.md`（不要用中文，用连字符分隔）

## Front Matter 模板

每个 Markdown 文件必须以以下 Front Matter 开头：

```yaml
---
title: "YYYY-MM-DD - 事件标题"
date: YYYY-MM-DD
category: 分类（如：安全事件、系统运维、工具创新）
status: published  # 或 draft
tags: [标签1, 标签2]
author: 作者/来源
---
```

## Commit Message 格式

脚本自动生成：`YYYY-MM-DD HH:MM - 事件描述`

- `YYYY-MM-DD`：事件发生的日期（从文件名或 front matter 提取）
- `HH:MM`：提交时的时间（Asia/Shanghai）
- `事件描述`：第二个命令行参数，简明扼要

示例：
```
2026-03-25 09:52 - Add README and CHANGELOG for project docs
```

## 更新内容说明

### CHANGELOG.md

脚本会在对应日期区块下添加新条目，格式：

```markdown
### YYYY-MM-DD HH:MM - Commit Message

- **日期** - 文件标题
  - 分类：xxx
  - 状态：✅ 已发布
```

### README.md

脚本会在"当前最新记录"表格的最前面添加新行：

```markdown
| YYYY-MM-DD | 事件标题 | 分类 | 状态 |
|------------|----------|------|------|
| 2026-03-25 | 事件标题 | 分类 | ✅ 已发布 |
```

## 手动更新（备用）

如果脚本失败，可以手动更新：

1. 添加文件：`git add 2026/新文件.md`
2. 更新 CHANGELOG.md：在对应日期下添加条目
3. 更新 README.md：在表格中添加新行
4. 提交：`git commit -m "YYYY-MM-DD HH:MM - 事件描述"`
5. 推送：`git push origin main`

## 注意事项

- 确保当前在 `~/AI-TIMELINE` 目录下运行脚本
- 文件路径相对于项目根目录（如 `2026/xxx.md`）
- 脚本会修改 CHANGELOG.md 和 README.md，请提前提交或保存未提交的更改
- 推送需要 SSH 密钥权限（已配置 `git@github.com:LoveFromIO/AI-TIMELINE.git`）

## 故障排除

### 权限错误
确保 SSH 密钥已添加到 GitHub 账户：
```bash
ls -la ~/.ssh/id_ed25519_github*
```

### 文件已存在
如果 CHANGELOG 或 README 已有未提交的更改，先提交或贮藏：
```bash
git stash
# 运行脚本
git stash pop
```

### 日期格式错误
确保文件名和 front matter 中的日期格式为 `YYYY-MM-DD`

---

**维护**：Eve (OpenClaw AI助手)
**最后更新**：2026-03-25 09:55
