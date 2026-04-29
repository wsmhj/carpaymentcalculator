# Car Payment Calculator — AdSense 集成与广告策略规划 v1.0

**适用范围**：carpaymentcalculator.app 全站 + 未来所有新页面（50 州落地页、子页、未来工具站、affiliate 页）

**当前阶段**：未提交 AdSense 审核，计划在 50 州落地页全部上线后提交

**核心偏好（已确认）**：
- 优先保证 UX 和计算器互动体验，广告不能干扰
- 手动 ad units 完全控制，不用 Auto Ads
- 全局 JS 加载 + 构建脚本统一管理位置（方案 B）
- 时间线：50 州落地页 → 提交审核 → 通过后部署 ad

**文档结构**：5 个阶段按时间顺序排列，每阶段说清楚做什么 / 不做什么 / 为什么。

---

## 阶段 1：审核前期（50 州落地页发布期间，预估 4-6 周）

### 1.1 严格禁止的行为

在 AdSense 审核通过之前，以下行为**绝对不要**发生：

| 禁止行为 | 原因 |
|---|---|
| 在任何页面插入 AdSense 主 JS（`pagead2.googlesyndication.com`） | 审核员发现"提前占位"会扣分 |
| 在根目录放置 `ads.txt` | 同上 |
| HTML 里写 `<ins class="adsbygoogle">` 标签 | 同上 |
| 在 `<meta>` 标签里加 `google-adsense-account` | 这个标签是审核绑定用的，提交审核时才加 |
| 部署任何 GA4 之外的 Google 服务 | 减少审核员可见的"广告变现意图" |

### 1.2 必须做的预埋（占位但不激活）

**这是方案 B 的核心**——审核前在所有模板里**预埋 ad slot 占位锚点**（HTML 注释，不是 ad 代码），等审核通过后构建脚本自动注入实际 ad 代码。

**占位锚点格式**（在 HTML 模板里）：

```html
<!-- ADSENSE:HEAD -->                  ← <head> 里，主 JS 注入位置
<!-- ADSENSE:TOP-OF-ARTICLE -->         ← article 第一个 H2 之前
<!-- ADSENSE:MID-ARTICLE -->            ← article 中间（H2 之间）
<!-- ADSENSE:END-OF-ARTICLE -->         ← article 末尾
<!-- ADSENSE:AFTER-RELATED -->          ← 邻州导航后
<!-- ADSENSE:SIDEBAR -->                ← 侧栏（仅桌面端，移动端不渲染）
```

**关键设计**：这些锚点**就是 HTML 注释**，浏览器不会显示，对 SEO 无影响，对页面渲染无影响。审核员看页面源码时**完全看不到广告意图**——只看到正常的注释。

### 1.3 创建配置开关（核心）

在 `_src/_data/site-config.json` 里：

```json
{
  "adsense": {
    "enabled": false,
    "publisher_id": "",
    "slots": {
      "top_of_article": "",
      "mid_article": "",
      "end_of_article": "",
      "after_related": "",
      "sidebar": "",
      "footer": ""
    },
    "_comment": "审核通过后改 enabled 为 true，填入真实 publisher_id 和 slot IDs，重跑构建脚本"
  }
}
```

`build.js` 里加一个 `injectAdSense()` 函数：

```javascript
function injectAdSense(html, siteConfig) {
  if (!siteConfig.adsense.enabled) {
    // 审核前/审核中：所有 ADSENSE: 锚点替换为空字符串
    html = html.replace(/<!-- ADSENSE:[A-Z\-]+ -->/g, '');
    return html;
  }
  
  // 审核通过后：注入实际 ad 代码
  const fs = require('fs');
  const adHead = fs.readFileSync('./_src/_shared/adsense-head.html', 'utf8');
  const adTop = fs.readFileSync('./_src/_shared/ad-top-of-article.html', 'utf8');
  const adMid = fs.readFileSync('./_src/_shared/ad-mid-article.html', 'utf8');
  const adEnd = fs.readFileSync('./_src/_shared/ad-end-of-article.html', 'utf8');
  const adRel = fs.readFileSync('./_src/_shared/ad-after-related.html', 'utf8');
  const adSide = fs.readFileSync('./_src/_shared/ad-sidebar.html', 'utf8');
  
  // 占位符替换为真实 publisher ID 和 slot ID
  function fill(template, slotKey) {
    return template
      .split('{{adsense.publisher_id}}').join(siteConfig.adsense.publisher_id)
      .split('{{adsense.slot_id}}').join(siteConfig.adsense.slots[slotKey]);
  }
  
  html = html.replace('<!-- ADSENSE:HEAD -->', fill(adHead, ''));
  html = html.replace('<!-- ADSENSE:TOP-OF-ARTICLE -->', fill(adTop, 'top_of_article'));
  html = html.replace('<!-- ADSENSE:MID-ARTICLE -->', fill(adMid, 'mid_article'));
  html = html.replace('<!-- ADSENSE:END-OF-ARTICLE -->', fill(adEnd, 'end_of_article'));
  html = html.replace('<!-- ADSENSE:AFTER-RELATED -->', fill(adRel, 'after_related'));
  html = html.replace('<!-- ADSENSE:SIDEBAR -->', fill(adSide, 'sidebar'));
  
  return html;
}
```

### 1.4 阶段 1 输出物清单

完成阶段 1 时，仓库应有：

- ✅ `_src/_data/site-config.json`（`adsense.enabled = false`）
- ✅ `_src/_shared/` 里 6 个 ad 模板文件（占位状态，含 `{{adsense.publisher_id}}` 等占位符）
- ✅ `_src/build.js` 含 `injectAdSense()` 函数
- ✅ 50 州落地页模板里所有 `<!-- ADSENSE:* -->` 锚点已预埋
- ✅ 现有 index.html / legal.html / embed.html 也要预埋锚点（在合适位置）
- ✅ widget.html **不要**预埋任何 ADSENSE 锚点
- ❌ 没有任何 AdSense JS 代码、`ads.txt`、`<ins>` 标签

---

## 阶段 2：审核期（提交审核 → 通过/拒绝，预估 1-4 周）

### 2.1 提交审核的最佳时机

**不要 50 州一上线就立刻提交**。建议等：

1. **50 州全部上线 + 4-8 周**（让 Google 索引、累积自然流量、GSC 有 impression 数据）
2. **GSC 显示至少 50% 的州页面已被索引**
3. **总日均 PV ≥ 100**（虽然 AdSense 没明确门槛，但 100+ 是经验数据，能避免审核员判定"无流量站点"）

### 2.2 提交审核的具体步骤

1. 在 AdSense 后台添加站点 `carpaymentcalculator.app`
2. 拿到一段验证 JS（约 1KB）
3. **只在 index.html 的 `<head>` 加这段验证 JS**（构建脚本里临时改一下，不用改所有页面）
4. 不要加 `ads.txt`、不要改任何其他文件
5. 等审核结果（通常 1-2 周，最长 4 周）

### 2.3 审核期间禁忌

| 禁止 | 原因 |
|---|---|
| 大量发布新页面 | 审核员看到"内容不稳定"会扣分 |
| 修改主页面布局 | 同上 |
| 关闭/重定向任何页面 | 同上 |
| 加任何其他广告网络代码 | 多重广告身份会触发拒绝 |
| 加 affiliate 链接（如果还没有的话） | 与 AdSense 政策可能冲突 |

### 2.4 应对拒绝（如果发生）

最常见的拒绝理由 + 应对：

| 拒绝理由 | 应对 |
|---|---|
| "Low value content" | 检查 50 州页面字数（是否 1200+ 词）、关键词密度、原创性。补 200-300 词差异化内容，重新提交 |
| "Insufficient content" | 等更多页面被索引、流量累积 1-2 个月再提交 |
| "Site not yet ready" | 检查导航完整性、隐私政策、联系信息（你 legal.html 已有，应该不会触发） |
| "Difficult navigation" | 检查 footer details 50 州导航是否在所有页面都生效 |
| "Duplicate content" | 加强 50 州页面差异化（OBBBA conformity 模板已经做了，但可能需要更多） |

**拒绝后多久能再提交**：AdSense 政策是任何时候都能再提交，但经验是**至少等 4 周 + 修改具体问题后再提交**，否则容易再次拒绝且降低审核员对站点的整体评分。

---

## 阶段 3：审核通过即刻部署（D-Day）

### 3.1 一键激活流程

审核通过那一刻，按这个顺序操作：

**Step 1**：编辑 `_src/_data/site-config.json`：

```json
{
  "adsense": {
    "enabled": true,
    "publisher_id": "ca-pub-1234567890123456",
    "slots": {
      "top_of_article": "1111111111",
      "mid_article": "2222222222",
      "end_of_article": "3333333333",
      "after_related": "4444444444",
      "sidebar": "5555555555",
      "footer": "6666666666"
    }
  }
}
```

slot ID 从 AdSense 后台逐个创建（详见 4.3 节，每个位置的 ad 类型不同）。

**Step 2**：在仓库根目录添加 `ads.txt`：

```
google.com, pub-1234567890123456, DIRECT, f08c47fec0942fa0
```

把 `pub-1234567890123456` 换成你的真实 publisher ID（注意去掉 `ca-` 前缀）。

**Step 3**：在 `_src/_data/site-config.json` 里加上 `<head>` meta 验证标签的开关（如果 AdSense 要求）。

**Step 4**：运行构建脚本：

```bash
node _src/build.js
```

预期输出：

```
✓ AdSense enabled, publisher: ca-pub-1234567890123456
✓ Generated alabama.html (with 6 ad slots)
✓ Generated alaska.html (with 6 ad slots)
... (50 个州)
✓ Patched index.html (with 4 ad slots)
✓ Patched legal.html (with 2 ad slots)
✓ Patched embed.html (with 1 ad slot)
✓ Skipped widget.html (no ads in iframe)
Done.
```

**Step 5**：

```bash
git add .
git commit -m "feat: activate AdSense across all pages"
git push
```

部署后 24-48 小时内广告会开始展示。

### 3.2 部署当天的视觉检查

部署后立即在浏览器开 **3 个不同页面 + 移动端模拟**，检查：

- [ ] index.html：广告位是否出现在合理位置
- [ ] california.html：6 个广告位是否都加载
- [ ] california-car-sales-tax.html：子页广告是否正常
- [ ] widget.html：**不应该有任何广告**（核心政策合规检查）
- [ ] 移动端 375px 宽度：广告是否响应式（不会撑破布局）
- [ ] 计算器组件附近：**不应该有广告**（UX 偏好）
- [ ] 输入框聚焦时：广告不应该 reflow 推动输入框位置（CLS 检查）

---

## 阶段 4：精确广告位置规范（这是核心干货）

按你"UX 优先"的偏好，下面是每个页面类型的精确广告位置 + 完整 HTML 代码。

### 4.1 设计原则（贯穿所有页面类型）

**禁区（绝对不放广告）**：

1. Hero 区域上方、Hero 区域内（H1 标题 + 副标题）
2. 计算器组件本身上下 100px 内
3. 输入框、按钮、结果显示框附近
4. Sticky bar 区域
5. mode-toggle（forward/reverse 切换）附近
6. 任何用户主动交互的元素附近（slider、tabs 等）
7. widget.html 全部（iframe 嵌入禁区）

**允许区（可以放广告）**：

1. 计算器结果框**之后** + 第一个 H2 **之前**（top-of-article）
2. 长 article 中间，H2 与 H2 之间（mid-article）
3. Article 末尾，FAQ 之后（end-of-article）
4. Related States 邻州导航之后（after-related）
5. 侧栏（仅桌面端 ≥900px，移动端不渲染）
6. Footer 上方（end-of-page）

**密度上限**：

- 任何页面**最多 4 个 ad slot**（AdSense 政策上无硬上限，但 4 个是 UX 与 RPM 的甜蜜点）
- 移动端**最多 3 个 ad slot**（移动屏幕小，超过 3 个广告导致跳出率激增）

### 4.2 各页面类型的精确广告位地图

#### 类型 A：50 州主页面（california.html、texas.html 等 50 个）

```
[Breadcrumb]
[Hero with H1]
[计算器 iframe]
[Subpage nav]
─────────────────── ← ★ AD #1 (top-of-article)
[H2: Sales Tax]
[200 words]
─────────────────── ← ★ AD #2 (mid-article)
[H2: Registration Fees]
[250 words]
[H2: Auto Loan Rates]
[150 words]
[H2: Car Payment Examples (table)]
[H2: City-by-City Comparison (table)]
[H2: California-Specific Tips]
[3 H3 子段(包括 OBBBA)]
─────────────────── ← ★ AD #3 (end-of-article)
[H2: California FAQ]
[8 FAQ items]
─────────────────── ← ★ AD #4 (after-related)
[Related States section]
[Footer]
```

**桌面端 4 个 ad slot，移动端隐藏 #2 mid-article 留 3 个**（移动端文字密度小，3 个广告够了）。

#### 类型 B：California 子页面（california-car-sales-tax.html、california-auto-loan-rates.html）

```
[Breadcrumb]
[Hero with H1]
[Subpage nav]
─────────────────── ← ★ AD #1 (top-of-article)
[H2 #1] [content]
─────────────────── ← ★ AD #2 (mid-article)
[H2 #2] [content]
[H2 #3] [content]
─────────────────── ← ★ AD #3 (end-of-article)
[Related Pages cluster]
[Footer]
```

**桌面端 + 移动端均 3 个**。

#### 类型 C：首页 index.html

这个页面用户主要互动计算器，**广告位极度克制**。

```
[Sticky bar]
[Hero with H1]
[Mode toggle]
[Container: 计算器 + 侧栏]
   ├ 主区: 计算器 + 结果展示
   └ 侧栏: ─────────── ← ★ AD #1 (sidebar，仅桌面端 ≥900px)
[长内容区: How it works / Tips / FAQ]
─────────────────── ← ★ AD #2 (mid-article)
[更多 SEO 内容]
─────────────────── ← ★ AD #3 (end-of-article)
[Footer]
```

**桌面端 3 个 ad slot，移动端 2 个**（移动端不展示 sidebar 广告）。

#### 类型 D：legal.html（About / Methodology / Privacy / Disclaimer / Contact）

这个页面是工具用户偶尔访问的，可以放 2 个 ad。

```
[Hero]
[H2: About]
[H2: Methodology]
─────────────────── ← ★ AD #1 (mid-article)
[H2: Privacy]
[H2: Disclaimer]
[H2: Contact]
─────────────────── ← ★ AD #2 (end-of-article)
[Footer]
```

#### 类型 E：embed.html（嵌入代码生成页）

这个页面访问量低，1 个 ad 足够。

```
[Hero]
[Embed code generator UI]
[Examples]
─────────────────── ← ★ AD #1 (end-of-article)
[Footer]
```

#### 类型 F：widget.html（iframe 嵌入小部件）

**绝对零广告**。

### 4.3 每个 ad slot 的 AdSense 类型推荐

在 AdSense 后台创建 ad units 时，每个 slot 选择对应类型：

| Slot 名称 | AdSense 类型 | 推荐尺寸 | 原因 |
|---|---|---|---|
| top_of_article | Display ad → Responsive | 自适应 | 头部要响应式，桌面端是大横幅，移动端是矩形 |
| mid_article | **In-article ad** | fluid | AdSense 专为正文中间设计的类型，自动匹配排版风格 |
| end_of_article | Display ad → Responsive | 自适应 | 与 top_of_article 一致 |
| after_related | Display ad → Responsive | 自适应 | 同上 |
| sidebar | Display ad → Square | 300×250 或 300×600 | 侧栏固定宽度，不需要响应式 |
| footer | **Multiplex ad**（matched content） | 自适应 | 末尾用 multiplex（推荐内容样式）转化率高 |

**为什么不用 In-feed ad**：In-feed ad 是给"列表/卡片流"用的，你的州页面是 article 结构，用 in-article 更合适。

### 4.4 完整 HTML 模板（6 个 shared snippets）

#### `_src/_shared/adsense-head.html`

```html
<!-- AdSense Global JS -->
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client={{adsense.publisher_id}}"
     crossorigin="anonymous"></script>
<meta name="google-adsense-account" content="{{adsense.publisher_id}}">
```

#### `_src/_shared/ad-top-of-article.html`

```html
<div class="ad-container ad-top-of-article" data-ad-position="top-of-article">
  <span class="ad-label">Advertisement</span>
  <ins class="adsbygoogle"
       style="display:block"
       data-ad-client="{{adsense.publisher_id}}"
       data-ad-slot="{{adsense.slot_id}}"
       data-ad-format="auto"
       data-full-width-responsive="true"></ins>
  <script>(adsbygoogle = window.adsbygoogle || []).push({});</script>
</div>
```

#### `_src/_shared/ad-mid-article.html`

```html
<div class="ad-container ad-mid-article" data-ad-position="mid-article">
  <span class="ad-label">Advertisement</span>
  <ins class="adsbygoogle"
       style="display:block; text-align:center;"
       data-ad-layout="in-article"
       data-ad-format="fluid"
       data-ad-client="{{adsense.publisher_id}}"
       data-ad-slot="{{adsense.slot_id}}"></ins>
  <script>(adsbygoogle = window.adsbygoogle || []).push({});</script>
</div>
```

#### `_src/_shared/ad-end-of-article.html`

```html
<div class="ad-container ad-end-of-article" data-ad-position="end-of-article">
  <span class="ad-label">Advertisement</span>
  <ins class="adsbygoogle"
       style="display:block"
       data-ad-client="{{adsense.publisher_id}}"
       data-ad-slot="{{adsense.slot_id}}"
       data-ad-format="auto"
       data-full-width-responsive="true"></ins>
  <script>(adsbygoogle = window.adsbygoogle || []).push({});</script>
</div>
```

#### `_src/_shared/ad-after-related.html`

```html
<div class="ad-container ad-after-related" data-ad-position="after-related">
  <span class="ad-label">Advertisement</span>
  <ins class="adsbygoogle"
       style="display:block"
       data-ad-client="{{adsense.publisher_id}}"
       data-ad-slot="{{adsense.slot_id}}"
       data-ad-format="auto"
       data-full-width-responsive="true"></ins>
  <script>(adsbygoogle = window.adsbygoogle || []).push({});</script>
</div>
```

#### `_src/_shared/ad-sidebar.html`

```html
<div class="ad-container ad-sidebar" data-ad-position="sidebar">
  <span class="ad-label">Advertisement</span>
  <ins class="adsbygoogle"
       style="display:inline-block;width:300px;height:600px"
       data-ad-client="{{adsense.publisher_id}}"
       data-ad-slot="{{adsense.slot_id}}"></ins>
  <script>(adsbygoogle = window.adsbygoogle || []).push({});</script>
</div>
```

### 4.5 配套 CSS（添加到所有页面共享样式）

这套 CSS 实现：
1. 广告位有明显视觉边界（"Advertisement" 标签是 AdSense 政策强烈推荐的）
2. 移动端隐藏 `mid-article` 和 `sidebar`
3. 防止 CLS（Cumulative Layout Shift）：广告未加载时也保留空间

```css
/* ===== AdSense Containers ===== */
.ad-container {
  margin: 32px auto;
  max-width: 760px;
  text-align: center;
  background: var(--warm);
  border: 1px solid var(--border);
  border-radius: var(--r-sm);
  padding: 12px 16px 16px;
  position: relative;
  /* 防 CLS：保留最小高度 */
  min-height: 120px;
}

.ad-container .ad-label {
  display: block;
  font-size: 10px;
  font-weight: 500;
  color: var(--text-light);
  text-transform: uppercase;
  letter-spacing: 1px;
  margin-bottom: 8px;
  text-align: left;
}

.ad-container .adsbygoogle {
  display: block;
  margin: 0 auto;
}

/* In-article 类型不需要外框，与正文融合 */
.ad-mid-article {
  background: transparent;
  border: none;
  padding: 8px 0;
  min-height: 250px;
}

/* Sidebar 广告样式 */
.ad-sidebar {
  margin: 0;
  max-width: 300px;
  min-height: 600px;
  position: sticky;
  top: 100px;
}

/* ===== 移动端响应式 ===== */
@media (max-width: 900px) {
  /* 移动端隐藏 sidebar 广告 */
  .ad-sidebar {
    display: none !important;
  }
}

@media (max-width: 680px) {
  /* 移动端 mid-article 仍然显示，但简化样式 */
  .ad-container {
    margin: 24px auto;
    padding: 8px 12px 12px;
  }
  /* 移动端隐藏 50 州主页面的 mid-article（保持总数 3 个） */
  /* 在主页面模板里给该 ad 加 .hide-mobile 类来控制 */
  .ad-mid-article.hide-mobile {
    display: none !important;
  }
}

/* ===== 防止广告挤压关键交互 ===== */
.ad-container + .calculator,
.calculator + .ad-container {
  margin-top: 48px !important;
}
```

### 4.6 模板修改示例（在 state-main.html 里）

修改 `_src/_templates/state-main.html`，在第 6.1 节的位置插入 ad 锚点：

```html
<!-- ===== Inline CSS ===== -->
<style>
  /* ... 现有 CSS ... */
  
  /* AdSense styles 注入位置 */
  {{adsense_styles}}
</style>

<!-- ADSENSE:HEAD -->

</head>

<body>
<!-- Breadcrumb / Hero / Calculator / Subpage nav -->
<!-- ...（现有内容） -->

<!-- ADSENSE:TOP-OF-ARTICLE -->

<article class="state-article">
  <h2>How {{state.name}} Sales Tax Affects Your Car Payment</h2>
  {{state.section_sales_tax}}

  <!-- ADSENSE:MID-ARTICLE -->

  <h2>{{state.name}} Vehicle Registration Fees Breakdown</h2>
  {{state.section_registration}}

  <h2>2026 Auto Loan Rates for {{state.name}} Buyers</h2>
  {{state.section_rates}}

  <h2>Car Payment Examples for {{state.name}}'s Top-Selling Models</h2>
  {{state.popular_vehicles_table}}

  <h2>City-by-City Tax Comparison</h2>
  {{state.top_cities_table}}

  <h2>{{state.name}}-Specific Buying Tips</h2>
  {{state.section_tips}}
</article>

<!-- ADSENSE:END-OF-ARTICLE -->

<section class="related-states">
  <!-- 现有内容 -->
</section>

<!-- ADSENSE:AFTER-RELATED -->

<article class="state-article">
  <h2>{{state.name}} Car Payment FAQ</h2>
  {{state.faq_html}}
</article>

<!-- 现有 footer -->
```

---

## 阶段 5：上线后优化（持续 3-12 个月）

### 5.1 第 1 周监控指标

部署后 7 天内每天检查 AdSense 后台：

| 指标 | 健康范围 | 异常处理 |
|---|---|---|
| Page RPM | $0.50 - $5（前期偏低正常） | < $0.30 持续 3 天检查广告填充率 |
| Ad Impressions | 与 GA 的 PV 数比例 ~80% | < 50% 说明 ad 加载有问题 |
| Click-through Rate | 0.5% - 3% | > 5% 警惕 invalid clicks（可能账号被封） |
| Coverage（填充率） | > 85% | < 70% 是正常的（部分位置无 ad 库存） |

### 5.2 第 1 个月：A/B 测试位置（手动）

AdSense 没有原生 A/B 测试，但你可以：

1. 把某个低 RPM 的州（比如 Wyoming）作为对照组，**移除其 mid-article 广告**
2. 持续 2 周观察该州 vs 其他州的 RPM 差异
3. 如果 Wyoming RPM 几乎不变（说明 mid-article 广告对该类州贡献低），考虑全局移除某些位置

### 5.3 第 2-3 个月：策略迭代决策点

根据数据决定是否启用以下优化：

#### 优化 1：是否启用 Auto Ads 部分类型

如果数据显示 RPM 低于行业平均（汽车金融垂直 $5-10），可以**有选择地**启用 Auto Ads 的某些类型：

| Auto Ads 类型 | 是否启用 | 原因 |
|---|---|---|
| In-page ads | ❌ 不启用 | 与你手动 ad 重复 |
| Anchor ads（底部强制条） | ❌ 不启用 | 严重干扰移动端 UX |
| Vignette ads（全屏过渡） | ❌ 不启用 | 同上，极度干扰 |
| Side rail ads（侧边浮动） | ❌ 不启用 | 移动端会变成 anchor，桌面端干扰阅读 |
| Multiplex ads（推荐内容） | 🟡 可考虑在 footer | 形似"相关阅读"，UX 友好 |

**默认所有类型都关闭**（这是方案 B 的承诺），除非数据明显证明启用某项的价值。

#### 优化 2：是否做 Header Bidding

只有当月广告收入 > $500 时考虑（说明流量已规模化）。Header Bidding 接入 Prebid.js 等中介，让多个广告网络竞价。技术成本高，初期不必。

#### 优化 3：是否引入其他广告网络

如 Mediavine（要求月 PV ≥ 50,000）、Ezoic、Raptive 等。门槛 + 政策严格，等流量稳定后再考虑。

### 5.4 第 6-12 个月：评估变现多元化

到这个阶段，你应该有：
- 月 PV ≥ 30,000
- 月 AdSense 收入 ≥ $200
- 50 州 + OBBBA 子页都已上线

可以开始考虑：

1. **Affiliate 整合**（基于你已有的 affiliate 品牌池研究，比如 LendingTree、Capital One Auto Navigator 这类汽车金融 affiliate）
2. **Sponsored content**（直接接洽汽车金融 SaaS 做内容合作）
3. **付费工具升级版**（如果有用户调研支持）

这一阶段的规划不在本文档，留给未来 v2.0。

---

## 附录 A — Widget.html 政策合规性

### A.1 为什么 widget.html 绝对不能放广告

AdSense 政策原文：
> "Publishers are not permitted to place Google ads on sites that are accessed via iframes that don't use parent frame display"

你的 widget.html 设计目的就是**让其他网站通过 iframe 嵌入**。在它里面放 AdSense = 在第三方网站的 iframe 里展示广告 = **触发 AdSense 账号封禁**。

### A.2 widget.html 的合规设计

保持现状即可：
- `noindex, follow`（已设置）
- 无任何 ad 代码
- 仅在 attribution 里链接到主站 `https://carpaymentcalculator.app/`

主站才是变现入口，widget 是流量入口。

### A.3 如果未来要给 widget 变现

唯一合规方式是：**让嵌入者付费购买无广告版本**。
- Free widget：你的样式，链接回主站，**永远无广告**
- Premium widget（未来 SaaS 化方向）：嵌入者付月费，可以自定义颜色、移除 attribution

这是另一条变现路径，与 AdSense 不冲突。

---

## 附录 B — 未来新页面接入标准

任何未来新页面（OBBBA 独立页、新工具站、新 affiliate 页），按以下 3 步接入 AdSense：

**Step 1**：判断页面类型，从 4.2 节选对应模板（A/B/C/D/E）

**Step 2**：在新页面 HTML 里加锚点：

```html
<head>
<!-- ... -->
<!-- ADSENSE:HEAD -->
</head>
<body>
<!-- ... -->
<!-- ADSENSE:TOP-OF-ARTICLE -->
<!-- ... -->
<!-- ADSENSE:MID-ARTICLE -->
<!-- ... -->
<!-- ADSENSE:END-OF-ARTICLE -->
</body>
```

**Step 3**：跑 `node _src/build.js`，自动注入实际 ad 代码。

**就这样**——这就是"全局 AdSense"的真正含义：新页面不用单独配置 AdSense，加锚点就行，构建脚本统一处理。

---

## 附录 C — Affiliate 与 AdSense 共存规则

如果未来加 affiliate 链接（如 LendingTree、汽车金融 SaaS），要注意：

1. **affiliate 链接不能放在 ad container 里**（会被识别为伪装广告，违反政策）
2. **affiliate 链接必须有 `rel="sponsored"`**（Google 2020 起的强制要求）
3. **同一段落里不能同时有 ad 和 affiliate 链接**（用户混淆来源）
4. **页面顶部 above-the-fold 区域**：要么 ad、要么 affiliate，不能两者都有

推荐布局：

```
[Hero] [Calculator]
─── ★ AD #1 ───
[H2 with affiliate link 可以]
─── ★ AD #2 ───
[H2 with affiliate link 可以]
─── ★ AD #3 ───
[Footer]
```

Ad 占据"通用变现位"，affiliate 内嵌在内容中（更高单 RPM）。

---

## 附录 D — 检查清单（执行 AI 用）

### D.1 阶段 1 完成检查

- [ ] `_src/_data/site-config.json` 已创建，`adsense.enabled = false`
- [ ] `_src/_shared/` 含 6 个 ad 模板文件，使用 `{{adsense.publisher_id}}` 占位
- [ ] `_src/build.js` 含 `injectAdSense()` 函数，处理 `enabled = false` 时清空锚点
- [ ] 所有模板（state-main、state-tax、state-rates）含 `<!-- ADSENSE:* -->` 锚点
- [ ] index.html、legal.html、embed.html 在合适位置加锚点
- [ ] widget.html **没有**任何 ADSENSE 锚点
- [ ] 浏览器查看任意页面 DOM，**应该完全看不到** AdSense JS 代码、`<ins>` 标签、`ads.txt`

### D.2 阶段 3 部署后检查

- [ ] `site-config.json` 中 `enabled = true`，所有 slot ID 已填
- [ ] `ads.txt` 已加到根目录
- [ ] 跑构建脚本无报错，输出显示所有页面已注入 ad
- [ ] 浏览器开 5 个不同页面 + 移动端模拟，6 个 ad slot 类型都能加载
- [ ] widget.html **完全无广告**
- [ ] 计算器组件附近 100px 内**无任何广告**
- [ ] 移动端 sidebar 广告**不显示**

---

**文档结束**

文档版本：v1.0
生成日期：2026-04-28
适用于：carpaymentcalculator.app + 未来全部新页面
依赖文档：seo-50-states-plan-v1.1.md（共享同一构建脚本架构）
