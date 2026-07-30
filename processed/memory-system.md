---
name: memory-system
description: 记忆系统 — L1/L2/L3架构 + 模型调度策略v3.0 + 所有cron任务速查 + iLink限流兜底铁律 + 触发规则
tags: [memory, knowledge-base, getnote, system, model-strategy, cron]
version: 2.0.0
author: [Agent名称]
---

# 记忆系统 (memory-system)

## 层1:宏观原则(战略层)

| 原则 | 说明 |
|---|---|
| 三层架构 | L1(本地精简)/L2(云端工作上下文)/L3(云端固化知识)各司其职 |
| 指定文件更新 | 永远在指定文件中更新,不新建 |
| L3 改对不打补丁 | L3 升级不是在后面打补丁,而是直接改对,错误的不要了 |
| 服务器信息必实测 | 不要 100% 信 L1 静态信息,服务器/网络是动态的,报信息前必实测一次 |
| API 路径必含 /resource/ | Get笔记 API 所有 note 操作端点都在 `/open/api/v1/resource/note/*` 路径下 |

### 指定文件路径(任何操作前必读)

| 用途 | 唯一指定位置 |
|---|---|
| L3 固化知识 | Get笔记「记忆系统·技术配置与完整备份」(note_id 见云端) |
| L2 工作上下文 | Get笔记 量化笔记库「工作上下文_YYYY-MM-DD」(当日一篇) |
| L1.5 本地 | `~/.hermes/MEMORY.md` |

## 层2:通用规则(战术层)

### L1 服务器/网络信息自检铁律(S级)

任何 L1 / USER PROFILE 服务器/网络/IP/端口信息被引用前必走:

| 规则 | 内容 |
|---|---|
| ❌ 不要 100% 信 L1 静态信息 | 服务器/网络是动态的(L1 写"无公网IP"实测已有;L1 写"22 端口"实测没监听) |
| ✅ 报信息前必实测一次 | 网络类用 `curl ifconfig.me` / `curl ip.cn`;端口用 `ss -tlnp`;服务用 `systemctl status` |
| ✅ L1 与实测冲突 = 必报差异 | 列两个值让用户拍板,不"L1 写啥就报啥",也不"实测是啥就推翻 L1" |
| ✅ L1 错误修正 = 必 patch L1 | `memory` 工具更新 USER PROFILE,加"PATCH DATE"标注,留修改痕迹 |
| ✅ 不可反解的字段 | `/etc/shadow` 哈希(`$y$` yescrypt / `$6$` SHA-512 / `$1$` MD5)单向,无法反解明文;必报"密码不可见,需控制台重置" |

**适用范围**:服务器 IP / SSH 端口 / 防火墙规则 / 操作系统版本 / 内核版本 / cron 任务 ID / deliver 配置 / API endpoint / base_url / model 名称 / 任何"曾经写过"的硬件/服务/路径信息

**L1 PATCH 流程**:
```yaml
# memory 工具更新 USER PROFILE 段
old: "服务器 无公网IP(公网IP要单独买)"
new: "服务器 [PATCH DATE] 公网IP [IP] (实测,旧 L1 写'无公网IP'已废) | 内网 [IP] | OS [版本] | SSH 端口 [端口] | 密码登录"
```

### Get笔记 API 路径铁律(S级)

Get笔记 API 所有 note 操作端点都在 `/open/api/v1/resource/note/*` 路径下。缺 `/resource/` 段 → 路由不匹配 → 返回 HTML 404 页面(非 JSON)。

**正确端点表**:

| 操作 | 方法 | 完整路径 |
|---|---|---|
| 读笔记 | GET | `/open/api/v1/resource/note/detail?id=NOTE_ID` |
| 创建笔记 | POST | `/open/api/v1/resource/note/create` |
| 更新笔记 | POST | `/open/api/v1/resource/note/update` |
| 语义搜索 | POST | `/open/api/v1/resource/recall` |

| 规则 | 内容 |
|---|---|
| ✅ 任何 note 操作路径必须含 `/resource/` | `/open/api/v1/resource/note/{create,update,detail}` |
| ✅ recall 路径 | `/open/api/v1/resource/recall`(不受影响,本身就在 resource 下) |
| ❌ 禁止 `/open/api/v1/note/{create,update,get,detail}` | 全部 404 |
| ✅ API 404 时第一反应 | 检查路径是否含 `/resource/`,而非假设"API挂了" |

### 搜索架构拆分(非股市/股市)

| Skill | 场景 | 主力源 | 守门 |
|---|---|---|---|
| multi-source-research | 股市/投资专业 | 腾讯API + 天天基金 + 东财公告 + Tavily | 无 |
| free-search-fallback | 非股市日常 | 12 个免费源(≥6 源) | Tavily 守门(平时禁) |

### iLink 限流根治方案

iLink 平台官方无文档,无具体阈值。唯一确定的事实是**触发器是条数,不是字符**(4 字符 [SILENT] 也限流)。

| 方案 | 内容 |
|---|---|
| 详简双轨投递 | 微信发简版(≤200 字)+ Get笔记 nid 索引 + agent 写 Get笔记详版 |
| jobs.json 触达量控制 | ≤17 次/天,≤5 次/小时 |
| 关键任务分档 | L1 软限 → L2 中限 → L3 硬限 |

## 层3:操作流程(执行层)

### 盘后复盘 cron + ETF盯盘 cron 已彻底移除

| 规则 | 内容 |
|---|---|
| ✅ 早盘启动检查 | `hermes cron list` 中 grep 已移除的 cron ID → 不在 = 正常,报"已移除"即可 |
| ✅ 不要尝试恢复 | 不要浪费工具调用去"暂停"一个不存在的 cron |
| ✅ 盘后复盘 cron 永远不恢复 | 除非用户明确要求 |
| ❌ 不要报"cron API unavailable / cron not found"作为错误 | 不在列表 = 已删除 = 预期行为 |

## 层4:数据来源(验证层)

### 参考文档索引(按需查阅)

| 文档 | 路径 | 何时读 |
|---|---|---|
| 搜索架构(重构) | `references/search-architecture.md` | 任何"搜索/查一下"场景 |
| 🔴 L3 改对不打补丁铁律(S级) | `references/l3-update-iron-law.md` | 任何"升级 L3"前必读 |
| 搜索 fallback 链历史 | `references/search-fallback-chain.md` | 历史参考(旧规范) |
| Vision 失败硬性处理 | `references/vision-fallback.md` | vision_analyze/browser_vision 失败 |
| API 401/Key 调试 | `references/deepseek-401-debug.md` | API 401/403/timeout |
| 模型路由 | `references/hermes-model-routing.md` | 模型调用失败/换模型 |
| Gateway 重启恢复 | `references/gateway-restart-recovery.md` | 网关挂/服务挂 |
| 记忆系统同步流程 | `references/memory-updater.md` | L1/L2/L3 同步 |
| 晨盘 cron 漏跑 | `references/morning-startup-cron-missed.md` | cron 没跑 |
| Cron 三层失败 | `references/cron-three-layer-failure.md` | 早盘崩复盘 |
| Cron Model dict 错 | `references/cron-model-dict-attribute-error.md` | Cron model 字段问题 |
| Cron Debug 通用 | `references/cron-debug.md` | Cron 任何异常 |
| Get笔记 API 路径 404(缺 `/resource/`) | `references/getnote-note-get-404.md` | API 返回 HTML 404 / 写入失败 |
| Cron 投递设计 | `~/.hermes/skills/devops/cron-delivery-design/` | jobs.json 顶层结构/schedule 表达式精度/deliver 模式/微信触达容量/iLink 限流对策/详简双轨协议 |
| Cron Model Config 坑 | `references/cron-model-config-pitfall.md` | Cron model 配置踩坑 |
| 模型迁移历史 | `references/model-migration.md` | 模型切换历史 |
| 持仓截图同步 | `references/screenshot-portfolio-sync.md` | OCR 持仓同步 |
| 代理设置 | `references/proxy-setup.md` | VPN/代理配置 |
| 记忆强化 | `references/memory-boost.md` | 记忆系统增强 |
| 已知混淆 | `references/known-confusions.md` | 易混淆概念 |

## 层5:快速参考(检查清单/铁律表)

### 服务器信息汇报检查清单

- [ ] 是否实测了网络/端口/服务(`curl ifconfig.me` / `ss -tlnp` / `systemctl status`)?
- [ ] L1 与实测冲突时是否列了两个值让用户拍板?
- [ ] L1 错误是否 patch 了(加 PATCH DATE 标注)?
- [ ] 不可反解的字段(密码哈希)是否报"不可见,需控制台重置"?

### API 路径检查清单

- [ ] note 操作路径是否含 `/resource/`(`/open/api/v1/resource/note/*`)?
- [ ] API 404 时是否先检查路径而非假设"API挂了"?
- [ ] 是否 skill_view 了 getnote-api 看完整端点表?

### 禁止行为清单

- ❌ 100% 信 L1 静态信息(服务器/网络是动态的)
- ❌ 用 `/open/api/v1/note/*`(缺 `/resource/`,全部 404)
- ❌ L3 升级在后面打补丁(直接改对,错误的不要了)
- ❌ 浪费工具调用去"暂停"一个不存在的 cron
- ❌ 报"cron API unavailable / cron not found"作为错误(不在列表 = 已删除 = 预期行为)
- ❌ 只改 L1 不告诉用户"刚 patch 了"(信任丢失)

## 文件结构

```
memory-system/
├── SKILL.md                    # 本文件
└── references/
    ├── search-architecture.md          # 搜索架构(重构)
    ├── l3-update-iron-law.md           # L3 改对不打补丁铁律(S级)
    ├── search-fallback-chain.md        # 搜索 fallback 链历史
    ├── vision-fallback.md              # Vision 失败硬性处理
    ├── deepseek-401-debug.md           # API 401/Key 调试
    ├── hermes-model-routing.md         # 模型路由
    ├── gateway-restart-recovery.md     # Gateway 重启恢复
    ├── memory-updater.md               # 记忆系统同步流程
    ├── morning-startup-cron-missed.md  # 晨盘 cron 漏跑
    ├── cron-three-layer-failure.md     # Cron 三层失败
    ├── cron-model-dict-attribute-error.md  # Cron Model dict 错
    ├── cron-debug.md                   # Cron Debug 通用
    ├── getnote-note-get-404.md         # Get笔记 API 路径 404
    ├── cron-model-config-pitfall.md    # Cron Model Config 坑
    ├── model-migration.md              # 模型迁移历史
    ├── screenshot-portfolio-sync.md    # 持仓截图同步
    ├── proxy-setup.md                  # 代理设置
    ├── memory-boost.md                 # 记忆强化
    └── known-confusions.md             # 已知混淆
```
