# 运动数据获取优先级策略

## 更新日期

2026-03-25 10:14 (Asia/Shanghai)

## 优先级定义

### Level 1: Garmin Connect（国际版）- 最高优先级

**账号信息**：
- 用户名：pqqq@me.com
- 密码：XSxs366496（存储在 `secrets/秘密/garmin-password.txt`）
- API 端点：Garmin Connect 国际版

**特征**：
- 最完整、最准确的运动数据
- 支持多种运动类型（跑步、骑行、游泳、力量训练等）
- 包含详细指标（心率、配速、爬升、训练效果等）
- 数据实时性高（同步延迟通常小于1小时）

**获取策略**：
- 所有运动数据查询必须首先尝试 Garmin
- 即使 Garmin 返回临时错误（如503），也不立即切换到 Strava
- 实现指数退避重试机制（最多3次，延迟2s、4s、8s）
- 只有在确认 Garmin 长期不可用（如超过24小时）时才考虑降级

### Level 2: Strava - 降级方案

**账号信息**：
- 用户名：dingxiaoshan
- Access Token: 2f3a272e2b22e3b7fe4170c7b973b8f7553d3ef4（6小时有效）
- Refresh Token: 2dec756c4e9e50f751eea97c20ab8c1b77971cb5

**特征**：
- 数据相对完整，但依赖于第三方同步
- API 有速率限制
- Token 需要定期刷新

**获取策略**：
- 仅在 Garmin 无任何数据记录（超过7天）时作为补充
- 不处理 Garmin 临时故障的情况
- 实现自动 token 刷新机制

### Level 3: Manual - 手动记录

**数据源**：
- Apple Notes
- 临时文本记录
- 用户主动输入的回忆

**特征**：
- 非实时，依赖人工操作
- 数据格式不固定，需要解析

**获取策略**：
- 当所有API都失败时启用
- 提供简单的 CLI 工具供手动录入
- 后续自动同步到 Garmin 作为主数据源

## 实施规则

1. **强制优先级**：任何运动数据获取脚本必须遵循 Garmin → Strava → Manual 的顺序
2. **错误处理**：Garmin 临时错误（503、timeout）应重试，不降级
3. **状态追踪**：每次获取都记录到 `memory/motion-sync-status.json`
4. **告警机制**：连续3次 Garmin 失败 → 发送通知
5. **数据验证**：获取的数据需校验字段完整性（日期、距离、时长必须）

## 脚本实现

### motion-data-fetcher.js

**位置**：`/Users/zhimakaimen/.openclaw/workspace/scripts/motion-data-fetcher.js`

**功能**：
- 统一运动数据获取入口
- 实现优先级策略
- 自动重试机制
- 生成状态报告

**用法**：
```bash
cd ~/.openclaw/workspace/scripts
node motion-data-fetcher.js
```

**输出**：
- 控制台：详细的获取日志
- 文件：`../memory/motion-data-YYYY-MM-DD.json`（数据）
- 文件：`../memory/motion-sync-status.json`（最新状态）

### garmin-sync-intl.js

**位置**：`/Users/zhimakaimen/.openclaw/workspace/scripts/garmin-sync-intl.js`

**功能**：
- 专门从 Garmin 获取过去7天数据
- 生成周报汇总
- 供 cron 定时任务使用

**用法**：
```bash
cd ~/.openclaw/workspace/scripts
GARMIN_USERNAME=pqqq@me.com GARMIN_PASSWORD=XSxs366496 node garmin-sync-intl.js
```

### garmin-today.js

**位置**：`/Users/zhimakaimen/.openclaw/workspace/scripts/garmin-today.js`

**功能**：
- 获取今日最新活动
- 快速检查

**用法**：
```bash
cd ~/.openclaw/workspace/scripts
node garmin-today.js
```

## 定时任务（cron）

当前配置（HEARTBEAT.md）：

- **每周日 09:00**：自动同步过去一周数据（Garmin）
- **每日 09:00**：检查今日运动情况（Garmin）

## 监控指标

| 指标 | 目标 | 告警阈值 |
|------|------|----------|
| Garmin 同步成功率 | ≥ 95% | < 80% |
| 数据延迟 | < 1 小时 | > 6 小时 |
| 重试次数 | ≤ 1 次/天 | > 3 次/天 |
| 手动记录频率 | 0 次/周 | > 1 次/周 |

## 故障处理

### Garmin 服务不可用（503）

1. 自动重试3次（指数退避）
2. 记录错误到 `motion-sync-status.json`
3. 不切换 Strava（除非确认长期故障）
4. 通知用户检查服务状态

### Strava Token 失效

1. 自动使用 refresh_token 刷新
2. 如果刷新失败，记录错误并停止尝试
3. 依赖 Garmin 数据源

### 数据不一致

如果 Garmin 和 Strava 数据差异超过20%，记录到 `motion-data-discrepancy.md` 供人工审核。

## 相关文档

- MEMORY.md "运动数据查询优先级"
- HEARTBEAT.md "Garmin 周同步"
- scripts/garmin-sync-intl.js
- scripts/motion-data-fetcher.js

---

**维护者**：Eve (OpenClaw AI助手)
**最后更新**：2026-03-25 10:15
