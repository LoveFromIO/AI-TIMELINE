---
title: "2026-03-25 - 运动数据同步服务故障与降级处理"
date: 2026-03-25
category: 系统运维
status: published
tags: [Garmin, Strava, API故障, 服务降级, 运动追踪]
author: 丁小山
---

# 运动数据同步服务故障记录

> 记录时间：2026-03-25 09:46 (Asia/Shanghai)
> 操作人：Eve (OpenClaw AI助手)
> 类型：API故障 / 服务降级

## 目标

将运动数据更新至 2026-03-25（当前日期），检查过去一周（2026-03-18 至 2026-03-25）的运动情况。

## 当前数据状态

- **最新记录**：2026-03-22
- **过去一周总结**（2026-03-15 至 2026-03-22）：
  - 总次数：3 次
  - 总距离：11.65 km
  - 总时长：191 分钟
  - 活动类型：trail_running, treadmill_running, strength_training

## 同步尝试

### 1. Garmin Connect API

**脚本**：`/Users/zhimakaimen/.openclaw/workspace/scripts/garmin-sync-intl.js`

**凭证**：
- 用户名：pqqq@me.com（国际版）
- 密码：XSxs366496

**结果**：❌ 失败

```
Error: Request failed with status code 503
```

**分析**：Garmin 服务暂时不可用（Service Unavailable），可能是临时维护或网络问题。

### 2. Strava API

**方法**：使用 Access Token 直接调用 REST API

**Token**：2f3a272e2b22e3b7fe4170c7b973b8f7553d3ef4

**结果**：❌ 失败

```
{"message":"Authorization Error","errors":[{"resource":"Athlete","field":"access_token","code":"invalid"}]}
```

**分析**：Access Token 已过期（6小时有效期），需要 refresh_token 刷新。

## 降级措施

1. **保持现有数据**：继续使用 2026-03-22 的最新记录
2. **记录同步失败**：在 MEMORY.md 中添加本次失败记录
3. **建议手动记录**：今日运动先记录到 Apple Notes
4. **计划重试**：晚些时候再次尝试同步

## 待办事项

- [ ] 检查 Strava refresh_token 机制，自动刷新 token
- [ ] 设置 Garmin 服务不可用时的重试策略（指数退避）
- [ ] 添加运动数据的手动录入界面或脚本
- [ ] 建立"API优先，手动降级"的数据同步流程

## 系统优化建议

### 短期（本周）
- 实现 Strava token 自动刷新
- 添加 Garmin 服务健康检查，503 时延迟重试
- 提供手动录入的便捷方式（如 Apple Notes → 自动解析）

### 中期
- 建立多数据源优先级：Garmin（主）→ Strava（备）→ 手动记录（降级）
- 设计数据同步状态监控面板
- 实现同步失败自动告警机制

### 长期
- 构建离线-first 的运动数据管理架构
- 本地缓存 + 云端同步的双向同步机制
- 支持多平台数据源（Apple Health、Google Fit等）

## 相关配置

- **Garmin 同步脚本**：`/Users/zhimakaimen/.openclaw/workspace/scripts/garmin-sync-intl.js`
- **数据存储**：`/Users/zhimakaimen/.openclaw/workspace/memory/garmin-weekly-*.json`
- **优先级规则**：先查 Garmin，无记录再查 Strava（MEMORY.md 中定义）

## 参考

- **运动健康报告偏好**：必须使用图表格式，结合WHO/ACSM等权威研究
- **数据分类**：非涉密-核心（个人健康数据）
- **同步频率**：每周日 09:00 自动同步

---