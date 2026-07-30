---
name: memory-sync
description: L1/memory tool/L1.5/L2四层记忆同步铁律 — 定位清晰，各司其职
tags: [memory, sync, L1, L1.5, L2, cron, 铁律]
trigger: [同步记忆, 更新持仓, 更新体系, memory更新, L1.5更新]
version: 2.0.0
author: [Agent名称]
---

# 四层记忆同步铁律 (memory-sync)

## 层1:宏观原则(战略层)

| 原则 | 说明 |
|---|---|
| 四层各司其职 | L1/memory tool/L1.5/L2 定位清晰,不串写 |
| 写入必验证 | 任何写入后必 GET 回验证,`success:true` ≠ 真写入 |
| 动手前必 skill_view | 任何任务动手前必先 skill_view,禁凭印象工作流 |
| 假成功=S级事件 | 改代码/配置后必走 4 步真验证才能说"已修" |
| L2 每日唯一 | 当日 L2 nid 有且仅有一个,禁硬编码失效 nid |
| 业务专题必 recall | 建任何业务专题笔记前必 recall 找已有,选对笔记本 |

### 四层记忆路径(每次动手前 5 秒看一眼)

```
L1     = ~/.hermes/memories/MEMORY.md    ← memory tool add() 也写这里
L1.5   = ~/.hermes/MEMORY.md             ← 根目录那个, 不是 memories 子目录
L2     = Get 笔记(云端·每天新 nid, 无固定 nid)
L3     = Get 笔记(云端)
```

**自检**(任何 L1/L1.5 写入前必走):
```bash
ls -la ~/.hermes/memories/MEMORY.md   # L1 (有 memories/ 子目录)
ls -la ~/.hermes/MEMORY.md            # L1.5 (根目录)
```

**memory tool add() 必走**:`add` 接口写的是 L1 (`memories/`),**不是** L1.5 (`./`)。L1.5 是 `write_file`/`patch` 工具写根目录,跟 memory tool 没关系。

## 层2:通用规则(战术层)

### 同步铁律速查表

| 编号 | 一句话 | 触发场景 | 强度 |
|---|---|---|---|
| #0 | 本地 skill ≠ 云端笔记 | 改完本地 skill 必同步云端 | S |
| #1 | 写入 L2/L3 必须 GET 回验证 | `success:true` ≠ 真写入 | S |
| #2 | 防 endpoint 和认证头陷阱 | Get 笔记 API 401/404 | S |
| #3 | L1 精简 ≠ memory tool 精简 | L1 ≤3000 / L1.5 无上限 / memory tool ~3KB | S |
| #4 | 持仓三源同步 | 6 步:复盘+脚本+POSITIONS+三脚本+三源对账+三层同步 | S |
| #5 | 5 段 base64 自检 | key 拼装前必跑 3 步断言 | S |
| #6 | 持仓清零/空仓期同步 | 5 只全清走 7 步(含 3 脚本 POSITIONS) | S |
| #7 | 写入前 grep 全层 | 防 L2 / L1.5 / memory tool 重复搬运 | S |
| #8 | L1 字符数 + 关键词自动检测 | mem_sync_hourly 必跑,违规硬报警 | S |
| #9 | 假成功=S级事件铁律 | 改代码/配置 → 必走 4 步真验证才能说"已修" | S |
| #10 | 业务专题建笔记前必 recall + 选对笔记本 | 防重复建·防塞错笔记本 | S |
| #11 | cron 故障排查先看时间线·禁凭 L1 记忆瞎说 | 看 created_at/last_run_at 再下结论 | S |
| #12 | L3 极简原则·错误直接改对·不在后面打补丁追加新章节 | 错误直接改对,不追加新章节 | S |
| #13 | 沉默型 cron 反 iLink 限流铁律 | 改 local + 降频,禁补丁思维 | S |
| #14 | L2 工作上下文 = 每天 1 篇新 nid(`note/create` 不 `note/update`) | 当天建立当天上下文 | S |
| #15 | 任何任务动手前必 skill_view·禁凭印象工作流 | 涉及外部 API/工具必 load skill | S |
| #16 | L2 每日唯一 nid 不变式 + 禁硬编码失效 nid | 多 cron 共享同一当日 nid | S |

### #9 假成功=S级事件铁律

改任何代码后,**绝不许**说"已修"——必走 4 步真验证:

```
① 重启 gateway 加载新代码 → systemctl --user restart hermes-gateway
  ↓
② 手触发 cron(不要等 next_run)→ hermes cron run <job_id>
  ↓
③ 看输出文件 KB 对比历史 → ls -la ~/.hermes/cron/output/<job_id>/今日.md
  ↓
④ grep errors.log 确认无相关报错 → grep "ErrorType" ~/.hermes/logs/errors.log
```

4 步全过才能说"修好"。

### #10 业务专题建笔记前必 recall + 选对笔记本

**建任何业务专题笔记前 5 步必走**(无法跳过):

```
① Step 1: recall 找"是否已有同类笔记"(主题词+业务词)
  → 命中已有 → 立即停笔,改"append 到该篇"而非新建
  ↓
② Step 2: 确认目标笔记本 ID(业务类不写量化笔记)
  ↓
③ Step 3: 写入时显式带 topic_id
  ↓
④ Step 4: 写入后 GET 验证(沿用 #1)
  ↓
⑤ Step 5: 用 topics 字段确认笔记本归属
```

**业务类去向**:
- 业务类体系规则 → memory tool(蒸馏精华)
- 业务类具体内容 → Get 笔记**业务笔记本**
- 业务类当日操作 → L2 业务类专属笔记(新建业务类 L2)
- 业务类方法论/技术 → L3 或 skill

### #15 动手前必 skill_view·禁凭印象工作流

**任何任务动手前 30 秒必走**(无法跳过):

```
① Step 1: 收到任务 → 立即想"我之前做过类似的吗?"
  → 是 → 该类 skill 必有 → 必 skill_view
  → 否 → 扫 skills_list 找相关 skill
  ↓
② Step 2: skill_view 后必读:端点/API 完整路径 + Headers + 返回路径 + 已知坑点 + 实战模板
  ↓
③ Step 3: 第一次调用前必跑"连通性测试"(GET /note/list 等无副作用端点)
  → 200 + 真实数据 = key 有效 + 路径对
  → 401/404 = 立即查 skill 别继续盲试
```

❌ **禁止**:凭印象猜路径/猜 Header/没 skill_view 就写 fallback 试错脚本/"API 不可用"收尾(实际是路径错了)

**预防自检 4 问**(任何涉及外部 API/工具的任务前必答):
- [ ] 这类 API/工具我之前用过吗?→ 用过 → **必 load**
- [ ] 完整 endpoint 路径我现在能背出来吗?→ 不能 → **必 load**
- [ ] Headers 我记得清吗?→ 不清 → **必 load**
- [ ] 返回路径我会读吗?→ 不会 → **必 load**

任一为"不能/不清/不会" → 立即 skill_view,不要凭印象。

**#15 vs #11 vs #9 区别**:
- #9 假成功 → 改代码/配置后没真验证就报"已修"
- #11 凭 L1 瞎说 → 故障排查没查 created_at/last_run_at 就下结论
- #15 凭印象工作流 → 动手前没 load skill,凭印象猜路径/Header/返回结构
- 共同点:都是"用记忆/印象代替查证",都进 S 级事件表

### #16 L2 每日唯一 nid 不变式 + 严禁硬编码失效 nid

**不变式**(每天必满足·任何 cron/任何人工操作都适用):
```
✅ 当日 L2 nid 有且仅有一个: recall "工作上下文_YYYY-MM-DD" 只能返回 1 个未被标废弃的命中
❌ 禁止: 同日 L2 有 ≥2 个未被标废弃的笔记(多篇 = bug 已发生)
```

**铁律**(3 条·无法跳过):

1. 🚫 **严禁硬编码任何 L2 nid** — L2 没有"固定 nid"概念,每天都是新 nid。任何 cron prompt/skill/L1 里出现的硬编码 nid 立即删除,改成统一模板

2. ✅ **L2 写入必须用「recall 找当日 nid → 找不到再 create」统一路径**(见层3)

3. ✅ **每日收盘后做一次「L2 唯一性审计」**(可在周日复盘 skill 里挂个 cron)

**配套检查**(patch 任何 cron/skill 后必跑·5 分钟可完成):
```bash
# 1. grep jobs.json 看所有引用失效 nid 的 cron
grep -F "<失效nid>" ~/.hermes/cron/jobs.json
# 2. grep skills 看所有引用失效 nid 的 skill
grep -rFl "<失效nid>" ~/.hermes/skills/
# 3. 任一命中 → 立即 patch 替换为统一模板
```

### #14 L2 工作上下文 = 每天 1 篇新 nid

**铁律**(L2 工作上下文写入必走 3 步·无法跳过):
```
✅ Step 1: 用 note/create 创建当天新 nid(不是 update 老 nid)
   - title 固定: 工作上下文_YYYY-MM-DD(当次触发简版描述)
   - topic_ids: 量化笔记库
   - 当天日期用 date "+%Y-%m-%d" 实测(不要凭印象)
✅ Step 2: GET 回验字符数一致才算成功(沿用 #1)
❌ 禁止: note/update 老 nid(覆盖原始内容,不可逆)
```

### Get 笔记 API key 安全写法

**踩坑**:`点+星号` 组合被 sandbox 渲染成 `***`,脚本 SyntaxError。

**唯一安全写法**(base64 拼装 + `write_file` + `python3` 调):

```python
# /tmp/xxx.py (用 write_file 工具写入, 不走 execute_code 也不走 -c)
import requests, base64
# base64 段从 references/keys.md 复制 (一次性生成, 永久复用)
ak = base64.b64decode('[API_KEY_BASE64]').decode()
h = {'Authorization': ak, 'X-Client-ID': '[CLIENT_ID]'}
r = requests.get('[API_BASE_URL]/open/api/v1/resource/note/detail',
                 params={'id': 'NOTE_ID'}, headers=h, timeout=15)
print(r.json())
```

**禁用**:
- ❌ `python3 -c "import requests; requests.get(..., headers={'Authorization': '[API_KEY]'})"`
- ❌ 任何 `点+星号` 在同一行连续出现

## 层3:操作流程(执行层)

### L2 写入统一模板(recall 找当日 nid → 找不到再 create)

```python
import requests, datetime
key = "[API_KEY]"
cid = "[CLIENT_ID]"
base = "[API_BASE_URL]"
today = datetime.date.today().isoformat()

# Step 1: recall 找当日 L2 nid
r = requests.post(f"{base}/open/api/v1/resource/recall",
    headers={"Authorization": key, "X-Client-ID": cid, "Content-Type": "application/json"},
    json={"query": f"工作上下文_{today}", "cursor": "0"}, timeout=15).json()

target_nid = None
for x in r.get('data', {}).get('results', []):
    t = x.get('title', '')
    if t.startswith(f'工作上下文_{today}') and '[废弃' not in t:
        target_nid = x.get('note_id')
        break

# Step 2: 找到 → update 追加本时段 / 没找到 → create 新建
if target_nid:
    d = requests.get(f"{base}/open/api/v1/resource/note/detail",
        params={"id": target_nid},
        headers={"Authorization": key, "X-Client-ID": cid}, timeout=15).json()
    old = d['data']['note']['content']
    new_content = old + f"\n\n---\n\n## <时段> {today}\n{<本段内容>}"
    if len(new_content) <= 15000:  # update 上限 15K
        requests.post(f"{base}/open/api/v1/resource/note/update",
            headers={"Authorization": key, "X-Client-ID": cid, "Content-Type": "application/json"},
            json={"id": target_nid, "title": d['data']['note']['title'], "content": new_content}, timeout=30)
    else:  # 超 15K → 单时段独立 create, 标题加时段后缀
        target_nid = requests.post(f"{base}/open/api/v1/resource/note/create",
            headers={"Authorization": key, "X-Client-ID": cid, "Content-Type": "application/json"},
            json={"title": f"工作上下文_{today}({时段}·独立段)", "content": "<本段内容>",
                  "topic_id": "<量化笔记库ID>"}, timeout=30).json()['data']['id']
else:  # 第一次触发 (如预采集) → create
    target_nid = requests.post(f"{base}/open/api/v1/resource/note/create",
        headers={"Authorization": key, "X-Client-ID": cid, "Content-Type": "application/json"},
        json={"title": f"工作上下文_{today}(本 cron 触发·<简版描述>)", "content": "<L2 内容>",
              "topic_id": "<量化笔记库ID>"}, timeout=30).json()['data']['id']

# Step 3: GET 回验 (沿用 #1)
v = requests.get(f"{base}/open/api/v1/resource/note/detail",
    params={"id": target_nid},
    headers={"Authorization": key, "X-Client-ID": cid}, timeout=15).json()
# 末尾输出: 📌 L2 写入: nid={target_nid} 字符={N} | 验证:✅
```

### mem_sync_hourly 必走 4 步

```
① Step 1.5: 跑 #8 检测(scripts/l1_self_check.py),违规就立即报警 + 列在报告顶部
  ↓
② Step 2: 跑 #7 grep 全层(防 L1.5 / L2 镜像重复)
  ↓
③ Step 3-4: 按"一周后还会准吗"分 4 层(投资/规则/索引/固化)
  ↓
④ Step 5: 写入后必 GET 回验证(#1) + grep 全层(#7)
```

### L2 唯一性审计流程

```python
# 审计当日 L2 是否唯一
today = datetime.date.today().isoformat()
r = requests.post(f"{base}/open/api/v1/resource/recall",
    json={"query": f"工作上下文_{today}", "cursor": "0"}, headers=h, timeout=15).json()
hits = [x for x in r.get('data', {}).get('results', [])
        if x.get('title', '').startswith(f'工作上下文_{today}')
        and '[废弃' not in x.get('title', '')]
if len(hits) > 1:
    # 🚨 S 级事件: 当日 L2 出现多篇, 触发 5 步合并流程
    ...
```

## 层4:数据来源(验证层)

### 支持文件索引

| 文件 | 内容 |
|---|---|
| `references/business-topic-creation-pitfalls.md` | #10 详细复盘 |
| `references/notebook-chooser.md` | 笔记本选择速查表(笔记本地图 + 9 步判断流程) |
| `references/l2-app-visibility.md` | #14 配套 App 可见性陷阱 |
| `references/keys.md` | API key base64 段(防 sandbox 渲染坑) |
| `references/mem-sync-hourly-cron-template.md` | mem_sync_hourly cron 模板 |
| `references/self-improvement-flywheel.md` | "skill 写规则但没人执行"类问题复盘 |
| `references/l2-daily-uniqueness.md` | #16 配套 L2 每日唯一 nid 不变式 |

## 层5:快速参考(检查清单/铁律表)

### 写 L2 / patch cron 前 30 秒必答

- [ ] 我要写的 cron prompt / skill 里有没有硬编码 nid?有 → 立即删
- [ ] 我要写的 cron prompt / skill 里有没有任何**硬编码 nid**?有 → 改成「recall 找当日 nid」模板
- [ ] 这个 cron 跟其他 L2 cron 是不是会产生「同日多 nid」?是 → 共享同一 nid
- [ ] 我跑完这次写入后,当日 L2 是不是仍唯一?recall 一次确认

### 每次 L2 写入前 30 秒自检

- [ ] 我是要写 L2 还是 L4 复盘?L2 = 持仓/操作/盘面,L4 = 完整 13 板块
- [ ] 我是要 create 新 nid 还是 update 老 nid?**默认 create**(除非明确说续写)
- [ ] title 是否按 `工作上下文_YYYY-MM-DD(<简版>)` 格式?
- [ ] topic_ids 是否带量化笔记库?
- [ ] 写入后 GET 回验字符数一致?

### 任务完成后自检

- [ ] 我这次任务 load 了相关 skill 吗?
- [ ] 我引用了 skill 里的端点路径 / Header / 坑点吗?
- [ ] 还是凭印象写的?(凭印象 → S 级事件)

### 禁止行为清单

- ❌ 凭印象猜路径/猜 Header/没 skill_view 就写 fallback 试错脚本
- ❌ "API 不可用"收尾汇报(实际是路径错了不是 API 坏了)
- ❌ 改代码/配置后没真验证就报"已修"(假成功 = S 级事件)
- ❌ 故障排查没查 created_at/last_run_at 就下结论
- ❌ 硬编码任何 L2 nid(L2 没有固定 nid,每天都是新 nid)
- ❌ `note/update` 老 nid(覆盖原始内容,不可逆)
- ❌ 业务类塞错笔记本(量化笔记不写业务)
- ❌ 没 recall 就新建笔记(防重复建)
- ❌ 错误后打补丁追加新章节(错误直接改对)

## 文件结构

```
memory-sync/
├── SKILL.md                    # 本文件
├── scripts/
│   ├── l1_self_check.py        # #8 L1 字符数+关键词检测
│   └── verify_l2_create.py     # L2 create 验证脚本
└── references/
    ├── business-topic-creation-pitfalls.md  # #10 详细复盘
    ├── notebook-chooser.md                  # 笔记本选择速查表
    ├── l2-app-visibility.md                 # #14 App 可见性陷阱
    ├── keys.md                              # API key base64 段
    ├── mem-sync-hourly-cron-template.md     # mem_sync_hourly cron 模板
    ├── self-improvement-flywheel.md         # 自我改进飞轮
    └── l2-daily-uniqueness.md               # #16 L2 每日唯一 nid
```
