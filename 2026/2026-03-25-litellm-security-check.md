---
title: "2026-03-25 - litellm 恶意版本安全检查"
date: 2026-03-25
category: 安全事件
status: published
tags: [litellm, 安全审计, 后门检测, Python包]
author: 丁小山
---

# litellm 恶意版本安全检查

> 检查日期：2026-03-25 09:48 (Asia/Shanghai)
> 检查人：Eve (OpenClaw AI助手)
> 类型：安全审计 / 恶意软件检测

## 背景

近期 litellm 包出现供应链攻击，恶意版本 1.82.7 和 1.82.8 包含后门程序 `sysmon`，会窃取环境变量中的API密钥等敏感信息。

## 检查范围

- 所有 Python 环境：系统级 pip、pip3、venv、conda
- uv 缓存：`~/.cache/uv` 中的 litellm_init.pth
- 后门文件：`~/.config/sysmon/` 和 `~/.config/systemd/user/sysmon.service`

## 检查结果

| 检查项 | 详情 | 状态 |
|--------|------|------|
| `pip show litellm` | 版本 1.82.6，位置 `/usr/local/lib/python3.11/site-packages` | ✅ 安全 |
| `pip3 show litellm` | 版本 1.82.6，位置 `/usr/local/lib/python3.11/site-packages` | ✅ 安全 |
| uv cache `litellm_init.pth` | 未找到 | ✅ 安全 |
| conda 环境 | conda 未安装 | ✅ 安全 |
| 后门目录 `~/.config/sysmon/` | 不存在 | ✅ 安全 |
| 后门服务 `~/.config/systemd/user/sysmon.service` | 不存在 | ✅ 安全 |

## 关键发现

- **检测版本**：1.82.6（安全版本）
- **恶意版本**：1.82.7、1.82.8（未检测到）
- **后门特征**：未发现 `sysmon` 相关文件或服务
- **凭据风险**：当前无已知凭据泄露风险

## 结论

系统未感染已知的 litellm 恶意后门。当前安装的 1.82.6 版本在安全范围内，无需执行卸载和凭据轮换操作。

## 建议

1. **定期检查**：建立 pip 包版本定期检查机制
2. **版本锁定**：使用 `pip freeze > requirements.txt` 锁定关键依赖版本
3. **升级监控**：升级 litellm 前验证版本安全性
4. **凭据管理**：继续使用环境变量管理敏感信息，避免硬编码

## 参考信息

- **受影响版本**：>= 1.82.7，<= 1.82.8（已从 PyPI 移除）
- **后门行为**：窃取环境变量、系统信息、API密钥
- **修复建议**：立即卸载恶意版本并轮换所有凭据

---