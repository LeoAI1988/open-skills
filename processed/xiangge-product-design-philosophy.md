---
name: xiangge-product-design-philosophy
description: "AI 智能体代人产品的设计哲学·class-level技能。涵盖产品理念定位(宏大/头皮发麻)、形态命名偏好(找朋友>找对象)、主人零参与原则、信息流设计、视觉风格。触发:做撮合/匹配/对接/AI Agent代主人的产品时。"
tags: [product, design, philosophy, ai-agent, matching, mandatory]
trigger: [撮合, 匹配, 对接, AI智能体, 代理人, 找朋友, 找对象, 理念, 宏大, 头皮发麻, 起鸡皮, 主人零参与, 主人不能手动, 黑底荧光绿, SVG, 代主人]
version: 2.0.0
author: [Agent名称]
license: MIT
---

# 产品设计哲学·铁律 (xiangge-product-design-philosophy)

> **隐式加载规则**:凡是做"AI 智能体替主人做 XX"类型的产品(撮合/匹配/对接/搜索/谈判/筛选),**默认应用**本 skill 的规则。
> **核心**:理念宏大·主人零参与·视觉极简

## 层1:宏观原则(战略层)

| 原则 | 说明 |
|---|---|
| 理念宏大 | 产品首页第一屏必须传达一个让人震撼的核心洞察——不是"我们有什么功能",而是"这个世界的运行方式可以被改变" |
| 主人零参与 | 人类自己都不一定了解自己,凭什么让主人填自己的性格?智能体读了主人 24h 对话/笔记/决策/情绪,比主人自己还懂主人 |
| 代码对代码 | Agent 之间交流通过 API+JSON,不是"主人读一段说明,手动抄给智能体" |
| 视觉极简 | yizhou 风格 + SVG 自绘,无 emoji 主图标 |
| 用户 OK = 锁定 | 产品迭代时,用户明确说"OK/不用动/这个可以"的元素 = 锁定状态,不许动 |
| Demo 1 步完成 | Demo 阶段主人点 1 次按钮 = 看到最终结果,不要让主人复制粘贴 curl |

## 层2:通用规则(战术层)

### 铁律速查表

| 编号 | 一句话 | 强度 |
|---|---|---|
| 铁律1 | 理念要宏大,要让人"头皮发麻" | S |
| 铁律2 | 形态命名——"找朋友" > "找对象" | S |
| 铁律3 | 主人零参与原则(绝对不可破) | S |
| 铁律4 | 信息获取要"代码对代码",不要"人对人" | S |
| 铁律5 | 视觉风格 — yizhou + SVG 不 emoji | S |
| 铁律6 | 手机竖版优先 + 浮动操作 | S |
| 铁律7 | 海报/分享要"可一键转发" | S |
| 铁律8 | 产品迭代时——用户说 OK 的部分 = 锁定状态,不许动 | S |
| 铁律9 | Demo 模式必须 1 步完成 — 主人零操作 | S |
| 铁律10 | 自测 200 OK ≠ 用户用 OK(自测会污染用户 token) | S |
| 铁律11 | 找朋友需求描述必须基于 Agent 真实读取的 L1 记忆,不能写死模板 | S |

### 铁律1:理念要宏大

**落地要求**:
- Hero 区占满首屏(min-height: 100vh)
- Logo + 主标题 + 理念文案 + 入口,4 个元素足矣
- 不要放任何表单/按钮/细节在第一屏
- 理念文案 ≤ 100 字,但每字千金
- 理念金句必须是**反直觉的洞察**,不能是"我们的产品更好"

**反例**:首页放功能描述 / 第一屏堆注册登录表单 / 文案写成"高效匹配/精准推荐"

### 铁律2:形态命名——"找朋友" > "找对象"

用户**不在乎形态**(恋爱/交友/合作),在乎**"找到懂自己的人"**。名字越宽泛,产品边界越延展。

**命名规则**:
1. 2-3 字动词+宽泛名词:找朋友/找导师/找同行
2. 不限定形态:恋爱/合作/知己/同行都能往里装
3. 子标题澄清:避免宽泛=模糊,加一句"不是XX,是XX"

**反例**:找对象/相亲/婚恋(单一形态排斥用户)、找资源/找项目(像 B2B 工具无情感)、智能匹配/AI 撮合(抽象无场景感)

### 铁律3:主人零参与原则(绝对不可破)

主人只需在**两个节点**参与:同意授权 + 最终审批撮合。

**落地流程**:
```
Agent GET /agent-protocol → 读规则
  ↓
Agent 从自己记忆读主人档案 → 自动结构化
  ↓
Agent POST /api/auto-enroll → 提交
  ↓
主人 GET /consent/{id} → 看到摘要,点同意  ← 唯一必参与点
  ↓
Agent 自动发布需求 + 撮合
  ↓
主人审批撮合结果  ← 第二个必参与点
```

**反例(致命)**:任何"主人填写表单"流程 / 上传 MBTI / 填写兴趣价值观家境 / "智能体根据你填写的内容匹配"

### 铁律4:信息获取要"代码对代码"

Agent 读 JSON → 知道要传什么字段 → 自动 POST。每种 Agent 平台单独标注记忆读取路径:
- Hermes: `~/.hermes/MEMORY.md` + USER.md + Get 笔记 API
- Claude Code: `~/.claude/memory/*` + `~/.claude/projects/*/CLAUDE.md`
- Codex/Cursor: `~/.codex/memory/*` 或 `owner_profile.json`
- Generic: Agent 自己决定

❌ **禁止**:写一段"请让你的 Agent 读这段话..."给人看 / 提供"复制此 JSON 到你的 Agent"引导 / 文档 > 200 字

### 铁律5:视觉风格 — yizhou + SVG 不 emoji

| 项 | ❌ 禁用 | ✅ 用 |
|---|---|---|
| 主图标 | emoji (💞🤝📤) | 自绘 SVG(几何抽象 + 微动画) |
| 主色调 | 多色/鲜艳 | 黑底 `#070908` + 荧光绿 `#00F58A` |
| 卡片 | 圆角阴影/渐变 | 极细边框 `#1f2120` + 微妙背景 `#0f1110` |
| 按钮 | 实心色块 | 透明 + 边框 hover 变绿 |
| 字体 | 衬线/装饰 | system-ui sans-serif(中性现代) |
| 间距 | 紧凑 | 大量留白(padding 24-32px) |
| 动画 | 弹跳/旋转 | 脉动/流动粒子/微缩放 |

**SVG 自绘原则**:抽象几何(两个圆+中间流动线)/ 微动画(animateMotion/animate)/ 统一风格(黑底荧光绿 1-2px 描边)/ 拒绝拟物

### 铁律6:手机竖版优先 + 浮动操作

- viewport 锁移动端:`maximum-scale=1.0, user-scalable=no`
- 字体 ≥ 16px(防 iOS 自动放大)
- 按钮高度 ≥ 48px(拇指友好)
- 触摸目标间距 ≥ 8px
- 右下角浮动按钮组:↑ 回到顶部(滚动 > 400px 显示)+ 分享(板块页常驻)
- **测试方法**:写完前端第一件事是用手机看,不是电脑看

### 铁律7:海报/分享要"可一键转发"

- 每条撮合结果旁有"生成海报"按钮(SVG 图标)
- 海报 720x1280(朋友圈最优比例)
- 一键下载 PNG → 长按 → 分享到微信/朋友圈
- 海报内容:分数 + 配对 + 推演 + 网址
- 视觉风格与产品一致(黑底荧光绿)

❌ **不要**:只生成链接让用户复制 / 海报只放文字不放分数 / 海报用模板套娃

### 铁律8:产品迭代时——用户说 OK 的部分 = 锁定状态,不许动

| 用户说 | 应该 |
|---|---|
| "改 A" | 只改 A,B/C/D/E 不动 |
| "重做这个功能" | 用 patch/增量/新按钮,不是整体重写 |
| "整体感觉不好" | 问清楚哪部分不好,只改那部分 |
| 沉默(没说 OK 也没说不行) | 不要擅自替换老方案,追加新选项让用户挑 |

**正确做法(增量更新 5 步)**:
```
① cp 老文件做备份
  ↓
② 只 patch 用户要求的那段(函数/按钮/路由/CSS class)
  ↓
③ 不动其他任何代码
  ↓
④ 验证时只验证 patch 的部分 + 回归测试老功能
  ↓
⑤ 报告时说明"老功能 100% 保留 + 新增 X 在 Y 位置"
```

❌ **错误做法**:重新设计整套界面/后端 + 用自己的"更好方案"替换用户认可的方案 + 把用户没说 OK 的部分也"顺便优化"

### 铁律9:Demo 模式必须 1 步完成

Demo 阶段(给用户第一次看效果),**主人点 1 次按钮 = 看到最终结果**。不要让主人在 demo 阶段也复制粘贴 curl、粘 verify_code、点同意等中间步骤。

| 模式 | 主人操作 | 后端代跑 |
|---|---|---|
| Demo 模式(第一次体验/演示) | **0 操作** | ✅ submit + verify 都代跑 |
| 生产模式(真实使用) | 复制 curl 给 Agent → 粘 verify_code | ❌ 不代跑,Agent 自己跑 curl |

**何时切到生产模式**:用户说"我自己跑 curl" → 切生产模式;真实 Agent 接入 → 切生产模式;Demo 演示 → 永远 demo 模式

### 铁律10:自测 200 OK ≠ 用户用 OK(自测会污染用户 token)

**正解:测试隔离 3 步**:
1. 自测用专属测试 token(前缀 `test_`,永远不复用生产 token)
2. 自测完立刻清掉 token 状态(调 `/api/handshake/clear` 或重启后端)
3. 给用户 token 时,明确说"这是给你的新 token,我不会用"

❌ **绝对不要**:用用户的 token 自测 / 给用户 token 后还在自测里碰到

### 铁律11:找朋友需求描述必须基于 Agent 真实读取的 L1 记忆

任何"AI 智能体代人"产品里,自动生成的需求/画像/小传/海报,**不能写死模板**。必须从 Agent 真实读取的主人记忆(L1/L2/笔记)动态生成。

**铁律(强制)**:
- ✅ 类型写宽泛("找朋友")但描述/城市/需求必须从真值生成
- ✅ 不写"不限性别/不限恋爱"这类话术(看着就是假的)
- ✅ 允许"城市范围 = 不限"作为兜底,但不写死 5 个城市让主人挑
- ✅ 小传/海报/档案摘要/分享文案 = 全部基于 stats + profile_text 真实数据
- ❌ 不要写死任何"通用需求模板"

## 层3:操作流程(执行层)

### 3 步握手流程(OAuth Device Flow 简化版·生产模式)

```
① 主人点 [一键复制连接命令] → 网站生成 token(5 分钟有效)+ curl 命令
  ↓
② 主人把 curl 命令粘给 Agent → Agent 自己跑 curl POST 到 /api/handshake/{token}/submit
  ↓
③ 网站返回 6 位验证码 → 主人复制验证码 → 粘到网站 → 点 [同意同步]
  ↓
④ 档案入库,Agent 拿到 API Key
```

### Demo 模式 1 步完成代码

```javascript
async function startDemo() {
  var platform = document.getElementById("agent-platform").value;
  // Step 1: start (生成 token)
  var r1 = await fetch("/api/handshake/start", {
    method: "POST", body: JSON.stringify({ agent_platform: platform })
  });
  var d1 = await r1.json();
  // Step 2: 后端代跑 submit (模拟 Agent 跑 curl)
  var r2 = await fetch("/api/handshake/" + d1.token + "/submit", {
    method: "POST",
    body: JSON.stringify({ agent_platform: platform, mtime_stats: {...}, display_name: "用户" })
  });
  var d2 = await r2.json();
  // Step 3: 后端代跑 verify
  var r3 = await fetch("/api/handshake/" + d1.token + "/verify", {
    method: "POST",
    body: JSON.stringify({ verify_code: d2.verify_code, owner_consent: true, display_name: "用户", platform: platform })
  });
  var d3 = await r3.json();
  showResult(d3);  // 直接跳到档案页
}
```

### 后端 3 路由(FastAPI + 内存 HANDSHAKES dict)

```python
# 1. 启动握手:生成 token + curl 命令
@app.post("/api/handshake/start")
async def handshake_start():
    token = secrets.token_urlsafe(8)
    curl_cmd = f'curl -sS -X POST "{endpoint}/api/handshake/{token}/submit" ...'
    HANDSHAKES[token] = {"verify_code": None, "expires_at": now+5min}
    return {"token": token, "curl_command": curl_cmd}

# 2. Agent 提交:返回 6 位验证码
@app.post("/api/handshake/{token}/submit")
async def handshake_submit(token: str, req: AgentSubmitRequest):
    if token not in HANDSHAKES:
        raise HTTPException(404, "Invalid or expired token")
    # 验证码字符集去掉容易混淆的字符 (0/O, 1/I)
    verify_code = "".join([secrets.choice("0123456789ABCDEFGHJKLMNPQRSTUVWXYZ") for _ in range(6)])
    HANDSHAKES[token]["verify_code"] = verify_code
    return {"verify_code": verify_code}

# 3. 主人验证:粘验证码 + 同意 → 档案入库
@app.post("/api/handshake/{token}/verify")
async def handshake_verify(token: str, req: HandshakeVerifyRequest):
    if req.verify_code.upper() != HANDSHAKES[token]["verify_code"]:
        raise HTTPException(401, "Wrong verify code")
    if not req.owner_consent:
        return {"success": False, "message": "主人拒绝同步"}
    # 写数据库 → 生成 API Key → 返回 agent_id
```

### 颗粒度 B(中等)方案 mtime-only

不读 memory 内容,**只统计文件 mtime**(最早/最晚/总文件数/记忆条数/项目数)。隐私 100% 安全。

```python
import os, time, json
files = []
for root, dirs, fns in os.walk(os.path.expanduser("~/.claude")):
    for fn in fns:
        fp = os.path.join(root, fn)
        files.append((fp, os.path.getmtime(fp)))
files.sort(key=lambda x: x[1])
mtime_stats = {
    "file_count": len(files),
    "oldest_mtime": datetime.fromtimestamp(files[0][1]).isoformat(),
    "newest_mtime": datetime.fromtimestamp(files[-1][1]).isoformat(),
    "memory_count": sum(1 for f in files if "/memory/" in f[0]),
    "project_count": sum(1 for f in files if "/projects/" in f[0]),
}
# POST 这个 dict 到 /api/handshake/{token}/submit
```

### 长任务汇报节奏

长任务(>30 分钟)**不能闷头干**,必须每 10 分钟或到关键节点时主动汇报:

```
✅ 进度: [已完成 N 项,共 M 项]
⏳ 当前: [正在做 XXX]
🆘 卡住: [如有,无则省]
```

**关键节点(必汇报)**:后端核心 API 写完 / 前端原型完成可看效果 / 测试通过可发链接 / 遇到失败需要决策 / 全部完成

## 层4:数据来源(验证层)

### 何时该用握手模式 vs 自动模式

| 场景 | 推荐模式 |
|---|---|
| 跨平台(不同 Agent CLI) | 握手模式(curl 通用) |
| 单一平台 + Agent SDK 完善 | 自动模式(更省事) |
| 用户对隐私敏感 | 握手模式(主人看到命令跑什么) |
| 营销演示/一次性活动 | 自动模式(主人操作少) |
| 真实生产环境 | 握手模式(鉴权更严,token 5 分钟过期) |

**默认用握手模式**:用户拍板的方向,后续同类产品都用这个模式。

### 已知坑点(验证沉淀)

| 坑 | 现象 | 修复 |
|---|---|---|
| SPA 按钮 inline onclick 在移动 WebView 不可靠 | 手机点按钮毫无反应 | 所有交互按钮必须用 `addEventListener` 绑定,禁用 inline onclick |
| 前端交付前没真实点击测试 | 看代码 OK 实际跑不起来 | 交付前必走 4 步:代码层 grep + HTTP 层 curl + 浏览器层 browser_click + 手机层 tunnel 测试 |
| 本地服务发现端点全错 | 凭印象改路径浪费 30 分钟 | 任何"API 找不到"先 skill_view 加载完整 skill,不凭印象猜路径 |
| JS 字符串双引号嵌套 | 整个 script 块 SyntaxError,按钮无反应 | JS 字符串用单引号或模板字符串,HTML 属性用双引号 |
| SPA 把内容塞进 hidden div | 后端 200 OK 用户看不到 | 塞内容前先确认目标 div 是 visible(`display !== 'none'`) |
| 函数改名要全栈同步 | 按钮调老函数名静默失败 | 改函数名 = 全栈 grep + 改(HTML onclick + JS 定义 + 后端路由) |

### Agent 平台接入协议模板

```python
# GET /agent-protocol 返回的协议结构
{
  "supported_platforms": [
    {"id": "hermes", "memory_locations": ["~/.hermes/MEMORY.md", "Get 笔记 API"]},
    {"id": "claude_code", "memory_locations": ["~/.claude/memory/MEMORY.md", "~/.claude/projects/*/CLAUDE.md"]},
    {"id": "codex", "memory_locations": ["~/.codex/memory/*", "owner_profile.json"]},
    {"id": "generic", "memory_locations": ["Agent 自维护 JSON"]}
  ],
  "required_owner_fields": ["age", "gender", "city", "personality", "work", "family", "friend_pref"],
  "consent_flow": "Agent auto-enroll → 主人只点同意按钮"
}
```

### 4 步流程卡片(主人在第 2 步和第 4 步才出现)

1. 选择 Agent 平台(Hermes / Claude Code / Codex / Generic)
2. 主人确认授权 ← 主人参与,只点 [同意]/[拒绝]
3. 智能体自动发布需求 + 撮合(主人无需参与)
4. 撮合结果 + 海报分享 ← 主人参与,审批 + 转发

## 层5:快速参考(检查清单/铁律表)

### 产品交付前自检清单

- [ ] 首页第一屏有"头皮发麻"级别的理念文案吗?
- [ ] 形态命名够宽泛吗(找朋友 > 找对象)?
- [ ] 主人是否只需在 2 个节点参与(同意授权 + 审批结果)?
- [ ] Agent 之间通过 API+JSON 通信,不是人对人?
- [ ] 视觉是 SVG 自绘 + yizhou 风格,无 emoji 主图标?
- [ ] 手机竖版友好,有"回到顶部"浮动按钮?
- [ ] 撮合结果可一键生成海报分享?
- [ ] Demo 模式 = 1 步完成,不要让主人在 demo 阶段也复制粘贴?
- [ ] 自测用专属测试 token,不污染用户 token?
- [ ] 需求描述基于 Agent 真实读取的 L1 记忆,不写死模板?
- [ ] 用户说 OK 的部分 = 锁定状态,没有动?
- [ ] 所有交互按钮用 addEventListener 绑定,无 inline onclick?
- [ ] 前端交付前真实点击每个按钮测试?

### 增量更新反问自检(动手前 30 秒)

- [ ] 用户要改 X,我有没有同时改 Y/Z?
- [ ] 老代码/老界面整体保留了?只在用户指定的部分加了 patch?
- [ ] 如果要"重写",是不是该用追加/增量/按钮+入口,而不是覆盖?
- [ ] 用户说"OK"的部分 = 锁定状态,有没有动?
- [ ] 报告里有没有说明"老功能保留 + 新增在哪"?

### 禁止行为清单

- ❌ 任何"主人填写表单"流程
- ❌ 首页第一屏堆注册/登录/表单
- ❌ 文案写成"高效匹配/精准推荐"营销话术
- ❌ 用 emoji 作主图标
- ❌ 整体重写用户认可的方案(只改用户要求的部分)
- ❌ Demo 阶段让主人复制粘贴 curl
- ❌ 用用户的 token 自测
- ❌ 写死"通用需求模板"(必须从真实记忆生成)
- ❌ inline onclick 在移动 WebView
- ❌ JS 字符串双引号嵌套
- ❌ 把内容塞到 hidden div
- ❌ 只改函数定义不改调用点
- ❌ 凭印象猜 API 路径(先 skill_view)

## 与其他 skill 的关系

- **xiangge-communication-style**:管对话风格(简洁/不发链接),本 skill 管**产品设计**。两者都是 class-level,隐式加载。
- **xiangge-image-design**:管公众号文章配图,与本 skill 互补(本 skill 是产品 UI,那是文章配图)。
- **memory-system**:产品偏好可沉淀到 USER.md "业务版图"段。
