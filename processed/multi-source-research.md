---
name: multi-source-research
description: "股市/投资专业搜索 — 腾讯API 主力 + Tavily 必先调 + 多源交叉验证 + stock_api_hub(行情/基金/公告) + Get笔记同步。触发场景:股市场景(股票/基金/代码/复盘/链传/触发价/持仓/财报)。非股市场景用 free-search-fallback(行业/技术/学术/中文文章/八卦)。"
version: 2.0.0
author: [Agent名称]
metadata:
  hermes:
    tags: [search, research, data-collection, getnote, tavily, web-search]
    related_skills: [getnote-api, free-search-fallback]
---

# 高质量多元信息采集 Skill (multi-source-research)

## 层1:宏观原则(战略层)

| 原则 | 说明 |
|---|---|
| 定位 | 为战略决策、赛道分析、行业研究提供高质量、多源验证、时效明确的信息采集服务(系统性采集→验证→标注→存储,不是简单搜一下给答案) |
| 场景路由 | 本 skill **只用于股市场景**;非股市场景(行业/技术/学术/中文文章/八卦)→ `free-search-fallback` |
| Tavily 必先调 | 搜索/复盘类任务第一步必 Tavily,失败再 fallback |
| 多源交叉验证 | 每个关键数据点必须用至少 2 个独立源验证(用户给的信息只是一个信息源,会被误导) |
| LLM 幻觉必 verify | 大模型生成的"事实性结论"必须 Tavily 二次 verify,特别是"未来事件已发生"的幻觉 |
| 用户链接必读 | 用户发的每一条链接都必须读取(S级铁律) |
| 工具状态必实测 | 报告"工具不可用"前必须 5 秒实测验,不凭印象说错 |

### 搜索场景路由铁律

| query 类型 | 用哪个 skill | 主力源 |
|---|---|---|
| 股票/基金/代码/复盘/链传/触发价/持仓/财报/公告/政策/异动 | **本 skill (multi-source-research)** | 腾讯API + Tavily(必先调) |
| 行业研究/技术/学术/概念/中文文章/八卦 | **free-search-fallback**(12 免费源 + Tavily 守门) | 12 源并行 |
| 模糊 | 用 clarify 列 2 个让用户点,不要猜 | - |

**反向搜索铁律(S级)**:每次搜索必须先跑「反向搜索」——搜与当前核心判断相反的方向。不是验证,是挑战。

## 层2:通用规则(战术层)

### 触发必 Tavily 铁律(S级)

**触发词**(任一命中 → 第一步必 Tavily,不调 = 任务未完成):
- "搜索 / 查一下 / 搜素 / 找数据"
- "今日新闻 / 最新消息 / 行情 / 7 维 / 几维"
- "复盘 / 盘后 / 盘中 / 晨报"
- "调研 / 研究 / 行业 / 赛道 / 链传"

**硬规则**:
- 没调 Tavily = 没做搜索,禁止"7 维搜索失败"之类的敷衍汇报
- web-search-prime 限流(429)+ DuckDuckGo ConnectError + web-reader 不可达**都不是借口**——Tavily 永远先试
- 7 维搜索/复盘类任务 = 调本 skill → 调 `tavily_search.py` → 失败再 fallback
- LLM 生成式摘要必须 Tavily 二次 verify 关键事件

### 多源交叉验证铁律

**每个关键数据点必须用至少 2 个独立源验证**,优先级:
1. 官方数据(交易所/公司公告/财报)> 权威媒体(证券时报/财联社/新华网)
2. 多源数据(雪球+investing+东方财富+腾讯API)> 单一源
3. 实时报价(腾讯API/Tavily)> 截图 OCR(可能延迟)
4. 多源冲突时:取多数源一致的数,标注冲突源

**已知多源验证场景**:
- 股价:腾讯API + 雪球 + investing + 东财(至少 2 个)
- 板块资金流:Tavily 搜"主力净流入"找多篇报道对比
- 新闻:Tavily + 浏览器打开源 URL + 财联社
- 业绩:公告原文 + 财经媒体转述 + 雪球数据

❌ **禁止**:仅用用户截图 + OCR 就出报告(收盘价误差教训)

### LLM 幻觉二次 verify 铁律(S级)

大模型生成的"事实性结论"必须 Tavily 二次 verify,特别是"未来事件已发生"的幻觉。

**已知幻觉模式**:

| 模型输出 | 幻觉类型 | Tavily 验证实际 |
|---|---|---|
| "央行决议(已落地,边际偏鸽)" | 未来事件当成已发生 | 实际是首次加息鹰派 |
| "两国两三天达成协议" | 未来事件当成过去讲 | 实际互袭 + 油价上涨 |
| "纳指 ETF T+0" | 事实错误 | QDII 是 T+1 |
| "取消某 ETF" | 擅自修改用户指令 | 用户只"加入",没"取消" |

**必 Tavily 验证的 LLM 输出类型**:
1. 任何带"具体数字+未来时态"的预测("X 将/预计/即将")→ 调 Tavily 验证当前实际
2. 任何"已发生的事件"("已落地/已公布/已通过")→ 调 Tavily verify 时间戳
3. 任何"评分结论"("X 强烈看多")→ 调 Tavily 验证 catalyst 是否真实
4. 任何"对手方观点"("分析师认为/央票定价/机构上调")→ 调 Tavily 查原文

### 用户链接处理铁律(S级)

用户发的每一条链接都必须读取。用户发链接 = 搜索/复盘/分析的输入素材,不是装饰。

**SPA H5 标准读取流程**:
```
① browser_navigate(url)
  ↓
② 等待 3-5 秒 JS 渲染(SPA 是空壳,需 JS 跑完才出 DOM)
  ↓
③ 提取渲染后 DOM 文本(关键:不要 querySelectorAll('p'),要 innerText)
  → browser_console("document.body.innerText.substring(0, 5000)")
  ↓
④ 读完后提取关键数据点,与搜索数据合并输出
```

**要点**:
- H5 页面(SPA)用 `browser_navigate` → `browser_console` 提取 `document.body.innerText`(SPA 用 div+span,p 选择器拿不到)
- "空 snapshot ≠ 空 body":browser_navigate 返回 (empty page) 不代表 body 是空,等几秒再跑 innerText
- 链接确实打不开(被墙/需登录)→ 如实告知,不能假装没看到

### 工具状态"实测 vs 凭印象"铁律

| 规则 | 内容 |
|---|---|
| ✅ 报告"工具不可用"前必须实测验 | 5 秒 `urllib.request.urlopen` 一次,比凭印象说错节省 10 倍后续代价 |
| ✅ 真实测状态 | 5 秒内 curl/urllib/requests 实测 → 报告说"实测 HTTP 状态码 X" |
| ✅ 凭记忆状态 | 上次失败/类比推断 → 报告说"上次 X 报 Y 错,这次可能也是" |
| ❌ 禁止凭印象断言不可用 | 不查证就说不可用 = 严重违纪 |

**5 个工具实测姿势**:

| 工具 | 实测命令 | 期望 |
|---|---|---|
| 腾讯 API | `urllib.request.urlopen("http://qt.gtimg.cn/q=sz[股票代码]", timeout=5)` | 200 + 511 字节 |
| Tavily | `requests.post("https://api.tavily.com/search", json={...})` | 200 + results 数组 |
| 雪球 | `requests.get("https://xueqiu.com/S/SH[股票代码]", headers={...})` | 经常 403 |
| 新浪美股 | `urllib.request.urlopen(..., headers={"Referer": "https://finance.sina.com.cn"})` | 经常 403 |
| web_search_prime | `mcp__web_search_prime__web_search_prime(query="test")` | 429 = 月配额耗尽 |

### Tavily 诊断铁律

| 现象 | 误判 | 真相 |
|---|---|---|
| Tavily 返回 400/422 | "限流/坏" | **请求格式/参数错** → 改姿势重试 |
| Tavily 返回 429 | - | 真限流 → 等或换 key |
| Tavily 返回 200 但 `results=[]` | "工具坏" | 真配额耗尽 → 降级到 web-search-prime/browser |
| `TAVILY_API_KEY` 不存在(.env 找不到) | "Tavily 坏了" | **key 缺失** = 改 web_search_prime |
| `import tavily_search` ImportError | "Tavily 不可用" | 改 sys.path.insert stock-surveillance/scripts |
| web_search_prime 返回 `[]` | "工具坏" | 月配额耗尽,等重置 |

### Tavily 不可用时改 web_search_prime 铁律(S级)

1. Tavily key 实测不可用时(5 秒实测),改用 `mcp__web_search_prime__web_search_prime` = 主力 fallback
2. 5 维搜索必跑全(不能用"时间紧迫"借口跳过)
3. 报告必出 5 维搜索实测结果(不是"工具失败"),每个维度标"✅ 已跑 / ❌ 失败"

### 禁止行为

- ❌ 搜索失败后不汇报,直接用预训练知识冒充搜索结果
- ❌ 用"搜索到了"开头但实际没搜索
- ❌ 信息没有标注时效和来源
- ❌ 连续搜索超过 10 次不暂停(会触发封锁)
- ❌ 把训练数据当成实时数据呈现
- ❌ 跳过用户发的链接不读取
- ❌ 凭印象断言工具不可用(必须实测)
- ❌ 借口"时间紧迫"5 维全跳

## 层3:操作流程(执行层)

### 搜索执行流程

```
① Step 0:明确搜索目标(数据/案例/趋势/观点?多新?多深?用途?)
  ↓
② Step 0.5:触发必 Tavily 铁律(触发词命中 → 第一步必 Tavily)
  ↓
③ Step 1:多源并行搜索(Tavily 优先) → Tavily 调通后才考虑其他源作补充
  ↓
④ Step 2:深度读取(对最相关 3-5 个 URL 用浏览器/curl 读全文)
  ↓
⑤ Step 3:数据验证与时效标注(每条数据标注时效+来源可信度)
  ↓
⑥ Step 4:同步到 Get笔记(结构化笔记)
```

### Tavily 调用姿势(3 种)

```python
# 姿势 1:脚本命令行 — 只传 query + time_range
python3 /home/ubuntu/.hermes/skills/stock-surveillance/scripts/tavily_search.py "2026年6月12日 美股 收盘" "d"
# time_range 可选: "d" (24h) / "w" (week) / "m" (month) / "y" (year)

# 姿势 2:Python import
import sys
sys.path.insert(0, '/home/ubuntu/.hermes/skills/stock-surveillance/scripts')
from tavily_search import search
r = search("query", max_results=4, time_range="d", search_depth="advanced")
results = r.get('results', [])

# 姿势 3:直接 requests.post — key 已硬编码在脚本里
import requests
r = requests.post("https://api.tavily.com/search", json={
    "api_key": "[API_KEY]",
    "query": "query",
    "max_results": 4,
    "search_depth": "advanced",
    "include_answer": True,
    "time_range": "d"
}, timeout=30)
```

**调用变体**:
- `search_depth=basic`:快速摘要(默认)
- `search_depth=advanced`:全文+相关问题(更慢,消耗 2x 配额)
- `include_answer=True`:返回 AI 总结(实战很有用,一句话给结论)
- `time_range="d"`:限定近 24 小时(7 维搜索实战必备)
- `max_results=3-5`:够用即可,避免配额浪费

**❌ 错误姿势**:
- 传 `"-n"` 当 time_range → 400(脚本不支持 `-n` flag)
- urllib 用 `urlencode`(form 编码)→ 422(Tavily 要 JSON body)

### LLM 幻觉 verify 流程

```python
# V4 Pro 输出关键事件 → Tavily 反向 verify
events_to_verify = [
    ("事件1", "query1"),
    ("事件2", "query2"),
]
for label, q in events_to_verify:
    r = search(q, max_results=3, time_range="d", search_depth="advanced")
    # 对比 LLM 输出 vs Tavily answer,冲突=幻觉,必须纠正
```

### 7 维搜索实战模板

```python
# 每维度一个独立 query,search_depth='advanced' + time_range='d'
dims = [
    ("个股_股价", "[股票代码] stock price news"),
    ("个股_业绩", "[公司] Q3 2026 revenue guidance"),
    ("宏观_政策", "central bank rate decision hawkish dovish"),
    ("行业_链传", "industry chain upstream downstream news"),
    # ... 每维度一个独立 query
]
for name, q in dims:
    r = search(q, max_results=4, time_range="d", search_depth="advanced")
```

### web_search_prime 5 维搜索模板(Tavily 不可用时)

```python
# 维度 1: AI 半导体
mcp__web_search_prime__web_search_prime(
    search_query="AI 半导体 龙头股 业绩 涨停",
    search_recency_filter="oneDay"
)
# 维度 2: 持仓涨跌 / 维度 3: 板块资金流 / 维度 4: 北向资金 / 维度 5: 中报行情
```

### 盘后快速核对持仓

```python
from stock_api_hub import quote_multi
holdings = ["[股票代码1]", "[股票代码2]", "[股票代码3]"]
results = quote_multi(holdings)
for r in results:
    print(f"{r['name']}: {r['change_pct']:+.2f}%  现价 ¥{r['price']}")
```

### 失败处理流程

```
① 测试所有可用搜索渠道
  ↓ 全部失败
② 列出已尝试的渠道和失败原因
  ↓
③ 告知用户"搜索工具不可用,以下信息基于已有素材+训练数据(截止时间XX)"
  ↓
④ 请用户提供信息源或建议替代方案
```

❌ **禁止**:搜索失败后不汇报,直接出答案

### 同步到 Get笔记

```python
import requests
API_KEY = '[API_KEY]'
CLIENT_ID = '[CLIENT_ID]'
BASE = '[API_BASE_URL]'
HEADERS = {'Authorization': API_KEY, 'X-Client-ID': CLIENT_ID, 'Content-Type': 'application/json'}

resp = requests.post(f'{BASE}/open/api/v1/resource/note/create',
    headers=HEADERS,
    json={'title': '标题', 'content': '内容', 'note_type': 'plain_text'},
    timeout=15)
note_id = resp.json()['data']['id']
```

**笔记格式要求**:
- 标题含日期:`{主题}·信息采集({YYYY-MM-DD})`
- 开头标注:数据来源、采集时间、时效性说明、采集工具、采集人
- 每条数据必须包含**具体数字**(ARR、营收、门店数、利润率、补贴金额等),不能只写框架/标题
- 每条数据标注来源 URL 和时效(✅最新/⚠️较新/📌历史参考)
- 结尾附数据来源 URL 汇总表
- **内容完整度标准**:拿到的数据全部写入,宁多勿少(只写标题没写数据 = 没法复用)
- ❌ 禁止:只写结论不写数据、只写框架不填内容、省略具体数字

## 层4:数据来源(验证层)

### 工具链优先级

| 优先级 | 工具 | 用途 | 何时用 |
|:---:|---|---|---|
| 1 | **Tavily REST** ⭐必先调 | 搜索/查新闻/查数据 | 必先调,主力 |
| 2 | **腾讯API**(qt.gtimg.cn) | 行情报价 | 实时股价/指数/成交量 |
| 3 | **浏览器**+browser_navigate+innerText | 读 H5/SPA/任何 URL | 用户给链接,或 Tavily 搜到的 URL 需要全文 |
| 4 | **Get笔记 API**(recall) | 查历史笔记/持仓/复盘 | 引用之前的数据 |
| 5 | **session_search** | 查历史对话 | 找之前怎么做的 |
| 6 | **web-search-prime MCP** | Tavily 失败后 | 偶尔备用(429 限流) |
| 7 | **ddgs** | Tavily 失败后 | 限流严重,中文差 |
| 8 | **Bing/36kr 浏览器** | Tavily 失败后 | 中文搜索质量差 |

### 股市专业 API 集(stock_api_hub.py)

```python
from stock_api_hub import quote, quote_multi, fund_nav, announcements

# 实时行情(主力,腾讯 API)
q = quote("[股票代码]")
# q = {name, price, change, change_pct, open, high, low, volume_ratio,
#      turnover_pct, pe, total_mcap, limit_up, limit_down, time, source}

# 批量行情(一次 HTTP 拉多只)
qs = quote_multi(["[股票代码1]", "[股票代码2]", "[股票代码3]"])

# 基金净值(天天基金)
fn = fund_nav("[基金代码]")
# fn = {name, nav, nav_date, estimate, estimate_pct, estimate_time}

# 公告(东财)
ann = announcements("[股票代码]", count=5)
# ann = [{title, code, notice_date, art_code}, ...]
```

### stock_api_hub 函数可用性

| 函数 | 状态 | 备注 |
|---|---|---|
| `quote(code)` | ✅ 用 | 腾讯API 主力,GBK 解析 |
| `quote_multi(codes)` | ✅ 用 | 一次 HTTP 拉多只 |
| `parse_ticker(code)` | ✅ 用 | 自动识别 sh/sz/hk/us 前缀 |
| `fund_nav(code)` | ✅ 用 | 天天基金实测返完整数据 |
| `announcements(code, count)` | ✅ 用 | 东财公告实测拿到 |
| `kline(code, period, count)` | ⚠️ 返 [] | push2his 404,EAST_KLINE_DISABLED=True |
| `fundflow(code, days)` | ⚠️ 返 [] | 同上,所有调用返空 |
| `sector_rank(count)` | ⚠️ 返 [] | push2 clist 404 |
| `fundflow_rank(count)` | ⚠️ 返 [] | 同上 |
| `get_full_quote(code)` | ⚠️ 部分可用 | quote OK,kline/fundflow 返空 |

**调用注意**:
- `kline/fundflow/sector_rank/fundflow_rank` 都返 `[]`(不是报错),调用方要按"空列表"判断"该维度数据不可用"
- 想**真**用 K线/资金流 → 等东财 datacenter-web 新端点找到后,移除 `EAST_KLINE_DISABLED=True`
- 实战 7 维搜索/复盘要走这些维度时 → **用 Tavily 补**(7 维搜主力本来就在)

### 8 个 API 端点实测

| API | 端点 | 状态 | 用途 |
|---|---|---|---|
| 腾讯API 实时行情 | `http://qt.gtimg.cn/q=sz[股票代码]` | ✅ 主力 | 51 字段实时行情 |
| 腾讯API 批量 | `http://qt.gtimg.cn/q=sz[代码1],sh[代码2],hk[代码3]` | ✅ | 一次拉多只 |
| 天天基金净值 | `http://fundgz.1234567.com.cn/js/[基金代码].js` | ✅ | ETF/基金净值+估值 |
| 东财公告 | `https://np-anotice-stock.eastmoney.com/api/security/ann` | ✅ | 个股公告/研报 |
| 东财 K线/资金流/板块 | push2his/push2 clist | ❌ 端点下线 | datacenter-web 待探索 |
| 新浪 hq.sinajs | - | ❌ 403 | 反爬 |
| 雪球 xueqiu.com | - | ❌ 403 | 反爬 |
| Yahoo Finance / Investing.com | - | ❌ 403 | 反爬 |

### 实战集成路径

- **盘前/盘中/盘后行情**:腾讯API quote/quote_multi(主力,毫秒级返回)
- **基金(纳指 ETF)**:天天基金 fund_nav
- **公告/研报**:东财 announcements
- **消息面/新闻**:Tavily 主力(7 维搜)+ 浏览器读全文
- **K线/资金流/板块**:⚠️ 暂缺,等东财 datacenter-web 新端点(可探索 `reportName` 报表系统)

### 本机出口环境(GFW 限制)

**铁律**:国际搜索源在本机基本不可用 — 不要列为默认选项;默认走国内直连源;Tavily 走 AWS 段(本机仍可用,不受影响)。

**✅ 国内直连可用**(12 个免费源,已用于 free-search-fallback skill):
Bing 中国/国际、Ecosia、Yandex、arXiv、OpenAlex、Crossref、百度百科、搜狗微信、GitHub、StackOverflow、npm

**❌ 完全不可用**(被墙/反爬):DuckDuckGo, Brave, SearXNG, Mojeek, Qwant, OpenLibrary, Wikidata, Wikipedia(中/英), Google Scholar, Twitter/X, YouTube, Semantic Scholar

**GFW 规律**:可用 IP 段 = AWS / 微软 / Fastly / 部分 Cloudflare / 国内 CDN

### 数据时效标注规则

每条数据必须标注:
- ✅最新(当年数据)
- ⚠️较新(上年数据)
- 📌历史参考(更早)
- ❓未验证(无法确认时间)

来源可信度:官方财报 > 权威媒体 > 行业报告 > 自媒体 > 训练数据

### 已知坑点

| 坑 | 现象 | 修复 |
|---|---|---|
| ddgs 安装与 Python 版本 | ddgs 装在系统 Python 3.12,但 venv 用 3.11 | 必须用 `python3 -m pip install ddgs` 安装到 venv(不能用 `pip install`) |
| ddgs 限流 | 连续搜索 5-8 次触发封锁 | 每个查询间隔 2-3 秒,连续 5 次后暂停 60 秒;优先英文搜索 |
| ddgs 中文差 | 中文搜索走 Yahoo/Startpage,极不稳定经常超时 | 优先英文搜索 |
| Bing 中文搜索质量极差 | 搜中文返完全无关结果 | 不要依赖 Bing 做中文搜索,只适合英文 |
| Tavily 配额 | 免费计划有用量上限,用完返 ForbiddenError | 返回空/results=[] 说明配额用完,降级 |
| web-search-prime 配额 | 有周/月配额限制,429 时跳过 | 配额重置时间在错误信息中显示 |
| 36kr 搜索 | 索引不全 | 适合科技/创投领域中文内容,不要指望覆盖所有关键词 |
| 网站访问限制 | Medium 被墙;PandaYoo 有 CF 保护;财联社/澎湃是 SPA | SPA 需浏览器渲染才能拿到内容 |

## 层5:快速参考(检查清单/铁律表)

### Tavily KEY 位置

```python
# key 实际在 2 个脚本里硬编码(无需查笔记·无需 .env):
# /home/ubuntu/.hermes/skills/stock-surveillance/scripts/tavily_search.py
# /home/ubuntu/.hermes/skills/daily-brief/scripts/tavily_search.py
# 两者 key 相同,直接 from tavily_search import search 即可用
TAVILY_KEY = "[API_KEY]"
```

### 实战组合

| 场景 | 组合 |
|---|---|
| 盘后复盘 | Tavily 必先调(7+7 维)+ 腾讯API(行情)+ 浏览器(读 H5)+ Get笔记(L2) |
| 战略研究 | Tavily 多轮搜索 + 浏览器读源 URL + 多源交叉验证 |
| 持仓核对 | 券商 app 截图 + Tavily(雪球/investing)+ 腾讯 API |

### 质量检查清单

- [ ] 是否先判 query 类型(股市用本 skill,非股市用 free-search-fallback)?
- [ ] 是否第一步必调 Tavily(触发词命中)?
- [ ] 是否每个关键数据点用至少 2 个独立源验证?
- [ ] LLM 输出的"事实性结论"是否 Tavily 二次 verify?
- [ ] 用户发的链接是否全部读取(SPA 用 innerText)?
- [ ] 报告"工具不可用"前是否 5 秒实测?
- [ ] 5 维/7 维搜索是否跑全(不跳过)?
- [ ] 每条数据是否标注时效和来源?
- [ ] 搜索失败是否如实汇报(不用预训练知识冒充)?
- [ ] 是否跑反向搜索(挑战核心判断)?
- [ ] 同步到 Get笔记是否含具体数字(不只写框架)?

### 禁止行为清单(汇总)

- ❌ 搜索失败后不汇报,直接用预训练知识冒充搜索结果
- ❌ 用"搜索到了"开头但实际没搜索
- ❌ 信息没有标注时效和来源
- ❌ 连续搜索超过 10 次不暂停
- ❌ 把训练数据当成实时数据呈现
- ❌ 跳过用户发的链接不读取
- ❌ 凭印象断言工具不可用(必须实测)
- ❌ 借口"时间紧迫"5 维全跳
- ❌ 仅用用户截图+OCR 就出报告
- ❌ 传 `-n` 当 time_range 给 Tavily
- ❌ 用 urlencode(form 编码)调 Tavily

## 相关 Skill

- **free-search-fallback**:非股市日常搜索(12 免费源 + P0 官方源 + Tavily 守门)
- **getnote-api**:Get笔记 API 操作
- **stock-chain-trading**:反向搜索详细规则(§17.28)

## 文件结构

```
multi-source-research/
├── SKILL.md                    # 本文件
├── scripts/
│   ├── stock_api_hub.py        # 腾讯API+天天基金+东财公告
│   └── tavily_search.py        # Tavily 调用(key 已硬编码)
└── references/
    ├── tavily-first-search.md          # Tavily 必先调实战
    ├── tavily-usage.md                # Tavily 调用铁律(3 步诊断流程)
    ├── tool-access.md                 # 5 工具实测姿势 + fallback 链
    ├── h5-spa-reading.md              # 13 条 H5 链接 + 4 步标准流程
    └── stock-api-hub-upgrade.md       # stock_api_hub 升级报告
```
