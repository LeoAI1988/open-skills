---
name: free-search-fallback
description: "免费搜索兜底集成 — 12 个免费源(无 key)全纳入,智能匹配场景 + 每次调用 ≥6 源并行 + 强制读全文。非股市场景用(行业/技术/学术/中文文章/概念/八卦/教育/香港身份/任何)。Tavily 完全留给股票:本 skill 100% 走 12 源,12 源全挂 = 标失败·不调 Tavily 抢救。股市场景走 multi-source-research。"
version: 2.0.0
author: [Agent名称]
metadata:
  hermes:
    tags: [search, free, fallback, multi-source, tavily-saver, no-key]
    related_skills: [multi-source-research, duckduckgo-search, tavily-search]
---

# 免费搜索兜底集成 Skill (free-search-fallback)

## 层1:宏观原则(战略层)

| 原则 | 说明 |
|---|---|
| 保护稀缺搜索额度 | Tavily 是稀缺资源,额度全部留给股票投资场景 |
| 非股市一律免费 | 12 个免费源覆盖所有非股市场景,Tavily 在本 skill 调用路径上 100% 不调 |
| 官方源优先 | 涉及官方权威渠道(政府/学校/交易所)时,先走官方直访,不依赖二手聚合 |
| 真发才算发 | 限流状态必须真发一次验证拿到 message_id,不靠"我以为恢复了"判断 |

**与 multi-source-research 的区别**:
- multi-source-research → 股市/投资专业搜索(腾讯API + Tavily 为主)
- free-search-fallback → 非股市日常搜索(免费源为主,Tavily 守门)

## 层2:通用规则(战术层)

### Tavily 守门铁律(终版·无例外条款)

| 规则 | 内容 |
|---|---|
| ❌ 禁止调 Tavily | 任何时候、任何形式、任何"兜底"借口 |
| ❌ 禁止留口子 | "12 源全挂才 Tavily 解封"等任何兜底设计一律禁止 |
| ❌ 禁止例外条款 | 不存在"以 X skill 为准,跳过守门"的例外 |
| ❌ 禁止 cron 自动调 | cron agent 在非股市场景禁止调 tavily-search tool |
| ❌ 禁止写解封章节 | prompt/SKILL.md 任何地方不准写"启用 Tavily 兜底"等章节 |
| ✅ 必须走 search_hub | search_hub.search() (12 源并行 + P0 官方源直访) |
| ✅ 全挂即标失败 | 12 源全 timeout/拒服 → 标"⚠️ 数据源失败,待修复" → 完(不抢救) |
| ✅ 拉不到不顶替 | 拉不到精确数据 → 标失败 → 不编、不顶替、不调 Tavily |

**核心动机**:非股市场景(世界杯/教育/百科/身份/技术/八卦)= 不是必须知道的数据,漏一天不影响决策;真重要数据(股票)用 Tavily 才值得。12 源全挂 = "今天运气不好",标失败等明天,不是"必须抢救"。

**实施要点**:
- Tavily KEY 仍存在于 search_hub.py(以备 multi-source-research 等股市场景用)
- 本 skill 调用入口(search_hub.search())永远不调 Tavily
- 股市场景(腾讯 API 拉不到时)= 单独调 tavily_search.py,不走本 skill

### 失败处理规则

| 场景 | 处理 |
|---|---|
| 单源失败 | 记录 `❌ {源} 失败 {原因}`,不影响其他源 |
| 多源失败(>50%) | 降级到"剩余源 top 5 全文读取" + 标注"低置信" |
| 12 源全 fail | 标"⚠️ 数据源失败,待修复" → 完(不调 Tavily) |
| 大量无关结果 | 12 源全通但 0 条有效新闻 → 标"12 源返大量噪声",改走 P0 官方源 curl 直连,不触发 Tavily |
| 返回结构 web 字段空 | 不要凭 `web` 为空就判定失败,先看完整结构 `r.keys()`;放弃 debug 直接走备用源,不反复重试 |

### 同源稳定性规则

| 规则 | 说明 |
|---|---|
| 6 源并行 = 正确架构 | 即使 5 源挂 1 源也够 |
| 4-5 源挂 = 正常波动 | 不要报警 |
| 6 源全挂才提示 | 触发失败标注 |
| 以单次实测为准 | 同一天同一源都能通→死,不被"上午通下午死"骗到 |
| 多日多时段验证 | 真验证不能拿单次跑通就宣告"成功" |

### 能力边界(搜不到的场景)

| 场景 | 根因 | 处理 |
|---|---|---|
| 24h 内官方人事/排班 | 开赛前 1-2 周才公布,只在内部系统/个人微博,不上网或不索引 | 不反复 retry,告诉用户"搜不到,等信息源自己公布" |
| 股票当日买卖盘口 | 走 L2 行情推送,搜索引擎抓不到 | 走 multi-source-research + 腾讯 API |
| 未公开内部信息 | 不公开发布,搜索引擎无索引 | 直接说"搜不到,等官方发布" |

**搜不到铁律**:搜不到 = 立刻汇报"搜不到" + 给出"等用户主动告知"或"去 X 看一眼"方案;❌ 禁反复重试 12 源;❌ 禁用预训练知识编造;❌ 禁列"我会继续搜/等会儿再试"。

### 限流判错铁律(S级)

| 规则 | 内容 |
|---|---|
| ❌ 禁止靠"我以为恢复了"判断限流状态 | |
| ✅ 必真发一次验证 | 拿到 message_id 才算"发出去";没 message_id = 限流中,没发出去 |
| ✅ 限流时等 5 分钟再补发 | 不连续触发 |
| ✅ 限流期间手动补发最多 1 次 | 失败就标"⏳ 限流冷却中,明早 9 点补发" |
| ✅ 高频推送严守 ≤200 字 | 不主动补发(被限就让用户去聊天记录翻) |
| ❌ 禁在 final response 写"已发微信"等措辞 | 系统会自己发,agent 不要替系统承诺 |

### 模块名铁律

| 规则 | 内容 |
|---|---|
| ✅ 真实文件 | `search_hub.py`(在 `scripts/` 下) |
| ✅ 真实模块 | `search_hub` |
| ✅ 正确 import | `from search_hub import search` |
| ❌ 错误 import | `from free_search_hub import search`(文件不存在) |
| ✅ 写 import 前先 ls | 确认文件名,禁止凭"语义名应该是"推断模块名 |

## 层3:操作流程(执行层)

### 主调用流程

```
① 解析 query → 判断场景类型
  ↓
② Step 0: 智能识别是否涉及官方源 → 是 → 浏览器直访 (P0)
  ↓
③ Step 1: 12 源并行(≥6 源) → 多源交叉验证
  ↓
④ Step 2: 强制读全文(浏览器 + innerText,不能只读标题+摘要)
  ↓
⑤ Step 3: 去重(按 URL 标准化 + title 相似度)
  ↓
⑥ Step 4: 交叉验证(同一信息出现 ≥2 源 = 高置信)
  ↓
⑦ 输出:每条结果标注 [来源1, 来源2, 来源3] + 置信度
```

**铁律**:实际调用源数 = max(6, 匹配源数)。匹配出的源 < 6 时,按权重从 general 源补足。

### P0 官方源工作流

```
search("query", min_sources=6)  →  P0 URL列表 + 12源标题列表
  ↓
browser_navigate(P0 列表中具体子页)  →  真实新闻标题/日期
  ↓
过滤 7 天内 + 关键词匹配  →  写入简报
```

**P0 通道要点**:
- P0 是"该去哪个官网"信号,不是"该看哪条新闻"信号
- 拿到 P0 URL 后必须再 navigate 进具体子页才能拿真实新闻
- P0 拿到结果时仍跑 12 源作多源验证(避免单一源风险 + 补充二手转述)
- ❌ 反模式:只跑 P0 不跑 12 源 → 漏掉多源验证
- ❌ 反模式:只跑 12 源不跑 P0 → 12 源全挂时就裸奔

### 生产环境备用链路优先级

1. 浏览器直访 / curl 官方源(EDB/HKEAA/JUPAS/info.gov.hk) ← 实际验证有效
2. 36kr RSS(curl 永远可拉) ← 实际验证有效
3. HN Algolia API ← skill 已记录
4. Crossref / arxiv / openalex(学术 fallback) ← 实际验证有效(但数据不是新闻)
5. Tavily(守门,全挂时才开,本 skill 不调)

### 投递限流实战工作流

```
① cron 跑 → Tavily 401/iLink 拒发(限流)
  ↓
② 等 5 分钟 → 真发一次 → 拿到 message_id?
  ↓ 是 → 已发出去
  ↓ 否 → 标"⏳ 限流冷却中,明早 9 点补发"
```

## 层4:数据来源(验证层)

### 12 个免费源清单(实测可用)

| ID | 名称 | 类型 | 语言 | 格式 |
|---|---|---|---|---|
| bing_rss | Bing RSS | 综合 | zh/en | XML |
| bing_academic | Bing 学术 | 学术 | en/zh | XML |
| ecosia | Ecosia | 综合 | en | HTML |
| yandex | Yandex | 综合 | en/ru | HTML |
| arxiv | arXiv API | 学术 | en | Atom |
| openalex | OpenAlex | 学术 | en | JSON |
| crossref | Crossref | 学术 | en | JSON |
| baidu_baike | 百度百科 API | 中文知识 | zh | JSON |
| wechat_sogou | 搜狗微信 | 中文公众号 | zh | HTML |
| github_api | GitHub API | 代码 | en | JSON |
| stackoverflow | StackOverflow | 技术问答 | en | JSON |
| npm | npm search | 包搜索 | en | JSON |

❌ **不可用**(被墙/反爬):DuckDuckGo, Brave, SearXNG, Mojeek, Qwant, OpenLibrary, Wikidata, 维基百科(中/英), Google Scholar, Twitter, X, YouTube

**生产可用性修正**:"实测可用"≠"生产可用"。真实可用源数应假设是 `max(2, 实测数 × 0.4)`。免费源最大价值不是搜索结果,是路径发现。

### P0 官方源识别规则

| query 涉及 | 官方源 | URL 模式 |
|---|---|---|
| 香港教育(DSE/JUPAS/教育局/考评局) | 教育局 EDB / 考评局 HKEAA / JUPAS 联招 | `https://www.edb.gov.hk/` / `https://www.hkeaa.edu.hk/` / `https://www.jupas.edu.hk/` |
| 香港入境/身份 | 入境处 ImmD / 高才通 | `https://www.immd.gov.hk/` |
| 中国教育 | 教育部 MOE | `http://www.moe.gov.cn/` |
| 科技/AI | 36kr RSS / 华尔街见闻 | `https://36kr.com/feed` / `https://wallstreetcn.com` |
| 中国新闻/政府 | 国务院 / info.gov.hk | `https://www.info.gov.hk/` |
| 开发者/开源 | GitHub | `https://api.github.com/` |
| 学术论文 | arXiv / OpenAlex | (已在 12 源里) |

### 智能匹配规则(每次 ≥6 个源)

| 场景关键词 | 必中源 | 兜底补充 |
|---|---|---|
| 学术/论文/研究/arxiv/citation | arxiv, openalex, crossref, bing_academic | + bing_rss, ecosia |
| 代码/库/github/框架/包/语言 | github_api, stackoverflow, npm | + bing_rss, ecosia, openalex |
| 中文/中国/微信/公众号/中文文章 | wechat_sogou, baidu_baike | + bing_rss, yandex, wechat_sogou |
| 知识/概念/百科/谁是谁/定义 | baidu_baike, bing_rss | + ecosia, yandex, github_api |
| 英文/海外/国际新闻/技术博客 | ecosia, yandex, bing_rss | + bing_academic, openalex, github_api |
| 八卦/新闻/最新/最近/突发 | bing_rss, wechat_sogou, ecosia | + yandex, baidu_baike, bing_academic |
| 默认(无明确场景) | bing_rss, ecosia, yandex, baidu_baike, github_api, arxiv | (全 6 个) |

### 官方源已知坑(验证沉淀)

| 坑 | 现象 | 修复 |
|---|---|---|
| HKEAA 是 SPA shell 页 | curl 各子页都返 78KB 同一模板 | 不要尝试 HKEAA,改走 EDB PR + info.gov.hk |
| ImmD Press Release 列表页 404 | `eng/press/press_releases/index.html` 返 404 | 改走 info.gov.hk 政府新闻聚合 |
| JUPAS News 页 browser_navigate 抓不到 | 返回菜单不是新闻 | 改走 query 命中 P0(JUPAS 官网首页)+ bing_rss |
| 百度百科 API 必须 Referer 头 | 不带 Referer 返 `{"errno":2}` 11 字节 | http_get 检测 `baike.baidu.com` 自动加 `Referer: https://baike.baidu.com/` |
| 36kr RSS link 字段 CDATA 包裹 | `<link><![CDATA[...]]></link>` | 解析代码兼容两种格式(CDATA + 裸 URL) |
| site: operator 不触发 P0 | `detect_official_source()` 只匹配关键词,不识别 Bing operator | 别用 `site:` operator,直接用中文关键词让 P0 命中更稳 |

## 层5:快速参考(检查清单/铁律表)

### 禁止行为清单

- ❌ 平时悄悄调 Tavily(必须显式触发,本 skill 永远不调)
- ❌ 6 个源读了不读全文(铁律)
- ❌ 跨场景使用本 skill 去搜股市(multi-source-research 才是)
- ❌ Tavily 自动启用而不提示
- ❌ "搜索失败"不汇报直接用训练数据补
- ❌ 凭"语义名应该是"推断模块名
- ❌ 靠"我以为恢复了"判断限流状态
- ❌ 反复重试 12 源(浪费时间)
- ❌ 用预训练知识编造

### 质量检查清单

- [ ] 是否走 search_hub.search()(12 源并行 + P0 官方源直访)?
- [ ] 是否强制读全文(browser_navigate + innerText)?
- [ ] 是否多源交叉验证(≥2 源 = 高置信)?
- [ ] 是否 100% 不调 Tavily?
- [ ] 搜不到时是否立刻汇报 + 给方案,不反复 retry?
- [ ] 限流时是否真发验证 + 等 5 分钟,不连续触发?
- [ ] import 是否用 `from search_hub import search`?

### Tavily KEY 位置(股市场景用·本 skill 不用)

```python
# 仅 multi-source-research 场景用
TAVILY_KEY = "[API_KEY]"
# 位置: ~/.hermes/skills/custom/free-search-fallback/scripts/search_hub.py 内部
```

### 智能识别官方源代码骨架

```python
OFFICIAL_SOURCE_RULES = {
    "education_hk": {
        "keywords": ["DSE", "JUPAS", "教育局", "EDB", "考评局", "HKEAA", "香港教育", "派位", "放榜"],
        "urls": ["https://www.edb.gov.hk/", "https://www.hkeaa.edu.hk/", "https://www.jupas.edu.hk/"]
    },
    "immigration_hk": {
        "keywords": ["入境处", "ImmD", "高才通", "优才", "IANG", "身份", "签证"],
        "urls": ["https://www.immd.gov.hk/"]
    },
}

def detect_official_source(query: str) -> List[str]:
    """返回该 query 应该直访的官方源 URL 列表"""
    q = query.lower()
    matches = []
    for rule in OFFICIAL_SOURCE_RULES.values():
        if any(kw.lower() in q for kw in rule["keywords"]):
            matches.extend(rule["urls"])
    # site: operator 补盲
    for domain in re.findall(r'site:([\w\.\-]+)', q):
        for rule in OFFICIAL_SOURCE_RULES.values():
            if any(domain in url for url in rule["urls"]):
                matches.extend(rule["urls"])
    return list(set(matches))
```

### Tavily 守门实现

```python
TAVILY_ENABLED = False  # 默认关
TAVILY_KEY = "[API_KEY]"

def maybe_enable_tavily(reason: str) -> bool:
    """守门函数:任何 Tavily 调用前必走这个"""
    global TAVILY_ENABLED
    if TAVILY_ENABLED:
        return True
    raise TavilyGateError(
        f"🚫 Tavily 已禁用(保护额度)。\n"
        f"触发原因: {reason}\n"
        f"如要启用,告诉AI助手:'用 Tavily 搜' 或确认启用。"
    )
```

### 调用示例

```python
import sys
sys.path.insert(0, "/home/ubuntu/.hermes/skills/custom/free-search-fallback/scripts")
from search_hub import search  # ✅ 正确模块名(不是 free_search_hub)

# 默认 ≥6 源并行
results = search("transformer attention mechanism 原理", min_sources=6)
# 自动匹配:学术场景 → arxiv + openalex + crossref + bing_academic + bing_rss + ecosia

results = search("2026 高考志愿填报", min_sources=6)
# 自动匹配:中文场景 → wechat_sogou + baidu_baike + bing_rss + yandex + ecosia + github_api
```

## 相关 Skill

- **multi-source-research**: 股市/投资专业搜索(腾讯API + Tavily 为主)
- **duckduckgo-search**: 单 DuckDuckGo 源(本 skill 已包含)
- **tavily-search**: Tavily 单独调用(本 skill 守门,需手动启用)

## 文件结构

```
free-search-fallback/
├── SKILL.md                    # 本文件
├── scripts/
│   ├── search_hub.py           # 12 源并行 + 智能匹配主入口
│   ├── parsers.py              # HTML/JSON/XML/RSS 各源解析
│   └── tavily_gate.py          # Tavily 守门(禁/启用/提示)
└── references/
    ├── source-validation.md    # 12 源实测报告
    └── p0-official-source-design.md  # P0 官方源直访设计
```
