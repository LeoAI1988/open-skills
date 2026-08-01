# Open Skills

面向公开使用的通用 Skill 集合。仓库与网站下载区保持同一份 Skill 清单和内容。

## Skill 清单

### 创作与发布

| Skill | 用途 |
|---|---|
| `content-writing` | 中文写作、合规改写与润色 |
| `topic-selection` | 内容选题池与标题策划 |
| `wechat-writing` | 公众号文章写作 |
| `article-image-design` | 封面、配图与系列卡片 |
| `cinematic-emotional-short-drama` | 原创电影感情绪短剧 |
| `hk-identity-classic-bridge` | 香港身份服务电影感广告创意 |
| `social-media-publishing` | 社交平台内容发布与授权素材采集 |

### 投资与研究

| Skill | 用途 |
|---|---|
| `daily-review` | 每日投资复盘 |
| `investment-upgrade-workflow` | 投资规则体系版本升级 |
| `multi-source-research` | 多源投资研究 |
| `performance-analysis` | 投资绩效分析 |
| `stock-chain-trading` | 产业链研究与风险情景 |

### 商业与策略

| Skill | 用途 |
|---|---|
| `strategic-thinking-system` | 复杂商业决策分析 |
| `tencent-creator-account` | 创作者账号与经营主体规划 |

### 通用工具

| Skill | 用途 |
|---|---|
| `free-search-fallback` | 免费公开搜索兜底 |
| `skill-management` | Skill 创建、审查、打包与安装 |

## 目录结构

```text
skills/
└── skill-name/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

复制整个 Skill 目录到当前 Codex 配置的 Skill 目录即可安装。网站 ZIP 下载包使用相同目录内容。

## 公开安全要求

所有 Skill 在发布前检查：

- 姓名、昵称、联系方式、账户、客户与家庭信息；
- API 密钥、令牌、Cookie、内部 ID 和私人链接；
- 本地路径、服务器地址和环境专属配置；
- 私人投资数据、未授权案例和可识别经历；
- 过时政策断言、绝对化收益承诺和未经确认的外部操作。

如发现以上内容，应先替换为通用占位或删除，再公开发布。
