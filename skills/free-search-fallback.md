---
name: free-search-fallback
description: 免费搜索兜底集成 — 12 个免费源（无需 API key）全纳入，智能匹配场景 + 每次调用 ≥6 源并行 + 强制读全文。非股市场景用本 skill。
tags: [search, free, fallback, multi-source]
trigger: [搜索, 查一下, 帮我查, 行业研究, 技术调研, 学术, 中文文章]
---

# 免费搜索兜底 Skill

## 一、用途

为非股市场景提供免费的多源搜索。**与股市搜索 skill 严格分流：**
- 本 skill：行业/技术/学术/中文文章/概念/八卦/教育/任何非股票场景
- 股市搜索 skill：股票/基金/ETF/复盘/行情

**铁律：** 本 skill 100% 走 12 个免费源，**不调付费 API（如 Tavily）**。12 源全挂 = 标失败，不抢付费额度。

## 二、12 个免费源

| # | 源 | 类型 | 中文质量 | 接入方式 |
|---|-----|------|---------|---------|
| 1 | DuckDuckGo | 综合 | 中 | `ddgs -b yandex` CLI |
| 2 | DuckDuckGo Python | 综合 | 中 | DDGS library |
| 3 | Brave Search | 综合 | 中 | 免费额度 |
| 4 | SearXNG 公开实例 | 元搜索 | 中 | 爬取 |
| 5 | Wikipedia | 百科 | 高 | API |
| 6 | 百度百科 | 中文百科 | 高 | 爬取 |
| 7 | 知乎搜索 | 中文问答 | 高 | 爬取 |
| 8 | 微博搜索 | 中文社交 | 高 | 爬取 |
| 9 | B 站搜索 | 中文视频 | 中 | 爬取 |
| 10 | arXiv | 学术 | N/A | API |
| 11 | GitHub | 代码 | N/A | API |
| 12 | Google Scholar | 学术 | 中 | 爬取 |

## 三、智能匹配

根据 query 关键词自动选择最合适的 6+ 源并行：

| Query 特征 | 优先源 |
|-----------|--------|
| 中文 + 人物/事件 | 知乎 + 微博 + 百度 + DDG |
| 中文 + 技术概念 | 百度 + 知乎 + GitHub + DDG |
| 学术/论文 | arXiv + Scholar + DDG |
| 代码/开源 | GitHub + DDG |
| 视频/教程 | B 站 + DDG |
| 通用/不确定 | 全部 12 源并行 |

## 四、执行流程

### Step 1：分析 query

判断类型（中文/英文/学术/代码/视频），选择优先源。

### Step 2：并行调用

至少 6 个源并行，不等单个源返回。

```python
import concurrent.futures

def search_all(query, sources):
    with concurrent.futures.ThreadPoolExecutor(max_workers=12) as executor:
        futures = {executor.submit(search, src, query): src for src in sources}
        results = []
        for future in concurrent.futures.as_completed(futures, timeout=15):
            try:
                results.append((futures[future], future.result()))
            except:
                pass
    return results
```

### Step 3：读全文

**不能只看 snippet**（摘要经常断章取义）。对每个有用的结果，用 `web_extract` 或 `browser_navigate` 读全文。

### Step 4：去重 + 排序

- 按内容相似度去重
- 按权威性排序（官方 > 媒体 > 自媒体）
- 按时效性排序（新闻优先）

### Step 5：输出

```markdown
## 搜索结果

**Query：** XXX
**源命中：** 6/12

### Top 5 结果
1. **[标题](URL)** — 来源/时间
   - 关键信息 1
   - 关键信息 2
2. ...
```

## 五、强制读全文铁律

**坑：** snippet 经常断章取义，导致结论错误。

**正确做法：**
- 对每个 Top 5 结果，用 `web_extract(urls=[url])` 读全文
- 如果全文 > 15000 字符，取 head+tail
- 读完后用自己的话总结，不直接复制 snippet

## 六、常见坑

| 坑 | 后果 | 规避 |
|----|------|------|
| 只看 snippet | 断章取义 | 强制读全文 |
| 单源失败就放弃 | 漏数据 | 12 源并行，单个失败不影响 |
| 中文 query 用英文源 | 质量差 | 自动匹配中文优先源 |
| DDG 不加 `-b yandex` | 中文质量差 | 必加 |
| 跑去调 Tavily | 浪费付费额度 | 本 skill 禁用付费源 |

## 七、失败处理

| 情况 | 处理 |
|------|------|
| 12 源全挂 | 标失败，提示用户手动查 |
| 6 源以下命中 | 降低置信度，标注"数据有限" |
| 单源超时 | 跳过，不等 |
| 全文读不到 | 用 snippet 兜底，标注"未读全文" |
