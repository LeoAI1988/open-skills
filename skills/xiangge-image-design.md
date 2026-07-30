---
name: xiangge-image-design
description: 公众号文章配图设计系统 — 黑底荧光绿终端风格，GPT Image 2 / Nano Banana 2 / HTML+CSS 多方案。精确控制排版/字体/配色/布局。
tags: [image-design, 公众号, 配图, HTML, CSS, AI生图]
trigger: [配图, 作图, 公众号图, 设计图, 横版图, 竖版图, 图文卡片]
version: 2.0
---

# 公众号配图设计 Skill

## 一、用途

为公众号/小红书/视频号内容生成配图。核心风格：**黑底荧光绿终端风**，精确控制排版/字体/配色/布局。

## 二、工具优先级

按效果+稳定性排序，依次尝试：

| 优先级 | 工具 | 适用 | 限制 |
|--------|------|------|------|
| 1 | **GPT Image 2**（Evolink） | 默认，效果好 | 需 API key |
| 2 | **Nano Banana 2** | GPT 不行时备选 | 中文渲染一般 |
| 3 | **HTML+CSS 截图** | 需 100% 精准排版时 | 无 AI 生成感 |

**关键：** 用户没说用什么工具 → 默认 GPT Image 2。

## 三、画面比例

| 比例 | 适用场景 | 尺寸 |
|------|---------|------|
| **5:3 横版** | 公众号正文配图（默认） | 1200×720 |
| **3:4 竖版** | 小红书/公众号封面 | 1080×1440 |
| **9:16 竖版** | 视频号/抖音 | 1080×1920 |
| **1:1 方形** | Instagram | 1080×1080 |

**铁律：** 默认横版。只有用户明确说"竖版""公众号封面""小红书"才切换。

## 四、设计规范

### 配色（终端风）

```css
--bg: #000000;
--green: #00F58A;
--white: #FFFFFF;
--gray: #666666;
```

### 字体

- **主标题：** 160px+ 超大（封面）/ 80-120px（内页）
- **副标题：** 40-60px
- **正文：** 24-32px
- **签名：** 20px

### 品牌签名（右下角，必须）

```
⌘ 用户名 · AI助手名
```

**顺序铁律：** 用户名在前，AI 助手名在后。

### 标题颜色规则

- 封面第一行：白色
- 封面第二行及内页：荧光绿（每张换色系）

## 五、工作流

### Step 1：确认规格

- 用途（公众号/小红书/视频号）
- 比例（横版/竖版/方形）
- 文字内容（标题/副标题/正文）
- 数量（1 张还是系列）

### Step 2：文字定稿

**铁律：** 文字必须先定稿，再做图。

- 用户说"定了"/"OK" → 进入作图
- 用户还在改文字 → 不作图
- 不要并发讨论图片规格（等文字定了再说）

### Step 3：选择工具

按"二、工具优先级"选择。

### Step 4：生成

**GPT Image 2 prompt 模板：**

```
Black background, neon green (#00F58A) terminal aesthetic.
Main title: "[标题]" in white, 160px bold.
Subtitle: "[副标题]" in neon green, 60px.
Brand signature bottom-right: "⌘ [用户] · [AI助手]" in 20px gray.
Layout: [横版 5:3 / 竖版 3:4].
Style: minimalist, high contrast, tech aesthetic.
```

### Step 5：交付

- 生成后发用户审稿
- 不满意 → 调整 prompt 重生
- 满意 → 归档到笔记系统

## 六、HTML+CSS 备选方案

当 AI 生图效果不稳定（中文乱码/排版错位）时，用 HTML+CSS 手写 + 浏览器截图。

```html
<!DOCTYPE html>
<html>
<head>
<style>
  body { margin:0; background:#000; font-family: -apple-system, sans-serif; }
  .container { width: 1200px; height: 720px; position: relative; }
  .title { color: #FFF; font-size: 160px; font-weight: 900; }
  .subtitle { color: #00F58A; font-size: 60px; }
  .signature { position: absolute; bottom: 30px; right: 30px; color: #666; font-size: 20px; }
</style>
</head>
<body>
<div class="container">
  <div class="title">[标题]</div>
  <div class="subtitle">[副标题]</div>
  <div class="signature">⌘ [用户] · [AI助手]</div>
</div>
</body>
</html>
```

用浏览器打开 → 截图 → 交付。

## 七、小红书图文卡片

**触发：** "做图文版""小红书图文""1000 字内"

### 规格

- 竖版 3:4（1080×1440）
- 每张一个核心观点
- 5-8 张成系列
- 封面+内容页结构

### 流程

1. 拆解文章为 5-8 个核心观点
2. 每个观点生成一张卡片
3. 封面用大标题+钩子
4. 内容页用观点+解释+例子
5. **所有图一次性统一发**（不分批）

## 八、已知坑

| 坑 | 后果 | 规避 |
|----|------|------|
| 文字没定稿就作图 | 改文字=全部重做 | 先定文字再作图 |
| 用 9:16 做公众号配图 | 比例不对 | 默认 5:3 横版 |
| 签名顺序写反 | 用户打回 | 用户名在前 |
| 标题字号太小 | 看不清 | 封面 160px+ |
| 并发讨论规格 | 干扰用户文字校对 | 等文字定了再说 |
| AI 生图中文乱码 | 不可用 | 切 HTML+CSS |
| 每张图上传笔记 | 啰嗦 | 等用户说"定了"再统一上传 |
| 自己改用户定稿文字 | 用户怒 | 一字不改 |

## 九、文章托管（可选）

生成的配图+文章可托管为 HTML 页面，方便分享：

1. 生成单文件 HTML（含所有图）
2. 上传到 paste.rs 或类似服务
3. 用 htmlpreview 包装链接
4. 发给用户预览

**注意：** 国内微信内置浏览器可能打不开 htmlpreview，需提示用户用外部浏览器。
