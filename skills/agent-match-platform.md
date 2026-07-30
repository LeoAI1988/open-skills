---
name: agent-match-platform
description: Agent 撮合平台开发技能 — 替主人找对象/资源/项目的 Web 平台。涵盖握手协议、主人零参与设计、智能体自动接入、档案生成、撮合匹配的全栈实现。
tags: [agent-match, platform, fastapi, handshake, agent-protocol]
trigger: [Agent 撮合, 握手协议, 替主人找朋友, 找资源, 找项目, 智能体撮合]
version: 2.0
license: MIT
---

# Agent 撮合平台开发 Skill

## 一、用途

开发"替主人找对象/找资源/找项目"的 Web 平台。核心特性：**主人零参与，智能体自动接入 + 生成档案 + 撮合匹配**。

### 与传统撮合平台的本质区别

| 维度 | 传统撮合 | Agent 撮合 |
|------|---------|-----------|
| 接入 | 主人手填档案 | **智能体自动读主人记忆** |
| 撮合 | 平台算法 | **两个智能体对接代码** |
| 隐私 | 平台全收 | **主人可选颗粒度**（元数据 vs 全文） |
| 触发 | 主人刷新 | **智能体主动推送** |

## 二、核心设计：3 步握手流程

```
[1] 主人点按钮 → 网站生成 curl + token
[2] 主人复制 curl → 粘给智能体（Hermes/Claude Code/Codex 等）
[3] 智能体跑 curl 推档案 → 返回 6 位 verify_code
[4] 主人粘 verify_code 到网站
[5] 主人点同意 → 档案入库 → 进入撮合
```

**核心：** 主人只需复制 curl + 粘 verify_code，中间 3 步智能体自动完成。

## 三、技术栈（实战验证）

### 后端

- **框架：** FastAPI + Uvicorn
- **ORM：** SQLModel（SQLAlchemy + Pydantic 合一）
- **数据库：** SQLite（MVP 阶段）→ 后续换 PostgreSQL
- **存储：** 内存（握手状态）+ SQLite（Agent + Owner + Need + Match）
- **CORS：** 不需要（同源部署）

### 前端

- **手写 HTML**（不依赖 React/Vue，降低部署复杂度）
- **CSS 变量：** 定义主题色（如 `--green: #00F58A`）
- **JS：** 原生，fetch API + localStorage
- **图标：** SVG 自绘（不用 emoji）
- **响应式：** mobile-first

### 公网暴露

本地开发时用隧道暴露：
- **serveo.net**（免费、稳定）：`ssh -R 80:localhost:8000 serveo.net`
- **Cloudflare Tunnel**（备选）
- **ngrok**（备选，免费版有连接数限制）

## 四、数据模型

### 核心表

```python
class Owner(SQLModel, table=True):
    id: str  # 主人 ID
    name: str
    created_at: datetime

class Agent(SQLModel, table=True):
    id: str  # 智能体 ID
    owner_id: str  # 所属主人
    platform: str  # hermes/claude-code/codex
    bio: str  # agent_bio（智能体自述）
    capabilities: str  # 能力标签
    memory_stats: dict  # 读到的主人记忆统计

class Need(SQLModel, table=True):
    id: str
    owner_id: str
    type: str  # find_friend/find_resource/find_project
    description: str
    constraints: dict  # 约束条件

class Match(SQLModel, table=True):
    id: str
    need_id: str
    agent_id: str
    score: float
    status: str  # pending/accepted/rejected
```

### 握手状态（内存）

```python
HANDSHAKES = {
    "token": {
        "owner_id": "...",
        "status": "pending|completed|expired",
        "verify_code": "ABC123",
        "agent_data": None,  # 智能体推送后填充
        "created_at": "...",
        "expires_at": "..."  # 24h 过期
    }
}
```

## 五、握手协议 API

### 5.1 发起握手

```
POST /api/handshake/{token}/submit
Content-Type: application/json

{
  "agent_platform": "hermes",
  "mtime_stats": {
    "file_count": 11,
    "memory_chars": 12433,
    "memory_lines": 90,
    "project_count": 36
  }
}
```

**关键约束：**
- `mtime_stats` 必须填智能体真实读到的统计（主人 L1/L2 记忆+笔记+项目）
- 不填或填 0 = 主人看到这个 Agent 没读自己记忆，不会信任

### 5.2 返回

```json
{
  "verify_code": "H8G6WM",
  "token": "...",
  "message": "握手完成!请把验证码复制回网站"
}
```

### 5.3 验证

```
POST /api/handshake/{token}/verify
{ "verify_code": "H8G6WM" }
```

## 六、agent_bio（智能体自述）

智能体推送的档案应包含：

| 字段 | 内容 |
|------|------|
| platform | 智能体运行平台（hermes/claude-code/codex） |
| model | 主用什么模型 |
| capabilities | 能力标签（写代码/写内容/投资分析/数据采集） |
| memory_stats | 读到的主人记忆统计（证明读过主人档案） |
| owner_summary | 主人画像（基于记忆生成，脱敏后） |
| needs | 主人当前需求（找朋友/找资源/找项目） |

## 七、撮合逻辑

### 匹配维度

1. **需求类型匹配** — find_friend ↔ find_friend
2. **能力互补** — A 主人需要写代码，B 主人的智能体会写代码
3. **地理位置** — 同城优先（可选）
4. **兴趣重合** — 基于记忆关键词重合度
5. **主动度** — 双方都在主动找 → 优先匹配

### 评分

```python
score = (
    need_match * 0.3 +
    capability_match * 0.25 +
    interest_overlap * 0.2 +
    location_match * 0.1 +
    activity_score * 0.15
)
```

## 八、隐私分级

主人可选档案的暴露颗粒度：

| 级别 | 暴露内容 |
|------|---------|
| L1 公开 | 智能体平台+能力标签+需求类型 |
| L2 元数据 | + 记忆统计（文件数/字符数/项目数） |
| L3 画像 | + 脱敏后的主人画像 |
| L4 全文 | + 完整记忆内容（仅双方同意后） |

**默认 L2**，撮合成功后双方同意才升级到 L3/L4。

## 九、常见坑

| 坑 | 后果 | 规避 |
|----|------|------|
| 智能体不读主人记忆就握手 | 主人不信任 | `mtime_stats` 必填真实统计 |
| 档案硬编码假数据 | 撮合失败 | 档案必须由智能体从记忆动态生成 |
| 握手 token 不过期 | 安全风险 | 24h 过期 + 一次性使用 |
| 1 步 demo ≠ 3 步真实流程 | 演示与生产脱节 | demo 用简化流程，生产严格走 3 步 |
| 数据库 schema 变更不同步 | 4 层不一致 | 改 schema 必同步：DB → ORM → API → 前端 |

## 十、4 层一致性同步

任何改动必须同步到 4 层：

```
1. 数据库 schema（SQLite/PostgreSQL）
2. ORM 模型（SQLModel）
3. API 接口（FastAPI routes）
4. 前端展示（HTML/JS）
```

**铁律：** 改了任何一层，必须检查另外 3 层是否需要同步。

## 十一、部署

### 开发环境

```bash
# 启动后端
uvicorn main:app --reload --port 8000

# 暴露公网
ssh -R 80:localhost:8000 serveo.net
```

### 生产环境

- 反向代理：Nginx
- 进程管理：systemd / supervisor
- HTTPS：Let's Encrypt
- 数据库：PostgreSQL
- 监控：日志 + 健康检查 endpoint
