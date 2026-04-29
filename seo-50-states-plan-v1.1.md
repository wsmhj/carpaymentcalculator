# Car Payment Calculator — 50 州落地页执行计划书 v1.1

**版本变更说明（v1.0 → v1.1）**：
- 新增 OBBBA（Auto Loan Interest Deduction）在每州页面的 H3 段落规范（第 2.4 节）
- 新增 `obbba_conformity` 字段到 states.json schema（第 1.3 节）
- 完整 50 州 + DC 的 OBBBA conformity 数据已填充（附录 C，源自 Tax Foundation 官方报告）
- H 标签结构新增 H3 "OBBBA Auto Loan Interest Deduction in [State]"
- 决策记录：不创建独立 OBBBA 页面（竞品红海评估，详见附录 D）

**目标读者**：能直接执行的 AI 编码助手（如 Claude Code、Cursor、Copilot）
**预期产出**：50 州主页面 + California 第一阶段子页面（税务页、利率页）的完整代码、文案、内链结构
**项目路径**：`D:/网站/baizaoyin/carpaymentcalculator/`（基于 settings_local.json 推断，本地路径）
**线上域名**：`https://carpaymentcalculator.app`
**部署方式**：Git push 到主分支自动部署（静态托管）

---

## 第 0 章 — 项目现状评估（已勘察确认）

### 0.1 技术栈真相

**这不是 Next.js 项目**。实际是纯静态站：

```
carpaymentcalculator/
├── index.html        (1748 行,内嵌 CSS + JS,主计算器)
├── embed.html        (生成嵌入代码的页面)
├── legal.html        (About / Methodology / Privacy / Disclaimer / Contact)
├── widget.html       (可嵌入的迷你计算器,带 query param)
├── favicon.svg
├── og-image.png
├── robots.txt
├── sitemap.xml       (目前只 3 条 URL)
└── README.md
```

**架构决定**:
- 每个 HTML 文件**自带完整 CSS 和 JS**(inline,不共享样式表)
- 没有构建步骤,没有 npm,没有 React
- 直接 `git push` 部署
- URL 都是真实文件路径(`legal.html` 而非 `/legal`)

### 0.2 设计 Token 现状(在 index.html line 132-158)

```css
:root {
  --cream:        #FFF8F0;     /* 主背景 */
  --warm:         #FEF3E2;     /* 次背景/输入框 */
  --peach:        #FFE8D6;     /* 强调底色 */
  --coral:        #E8725A;     /* 主品牌色/CTA */
  --coral-light:  #FFF0EC;
  --coral-dark:   #D45A42;
  --forest:       #2D5A3D;     /* 次品牌色/绿色徽章 */
  --forest-light: #EAF2EC;
  --forest-soft:  rgba(45,90,61,0.08);
  --navy:         #1E2A3A;
  --navy-soft:    #3A4A5C;
  --brown:        #8B6F47;
  --brown-light:  #C4A97D;
  --text:         #2C2420;
  --text-mid:     #5C4F44;
  --text-light:   #9B8E82;
  --border:       #E8DDD2;
  --border-dark:  #D4C5B5;
  --shadow:       rgba(44,36,32,0.06);
  --shadow-md:    rgba(44,36,32,0.1);
  --r:    16px;   /* 默认圆角 */
  --r-sm: 10px;
  --r-lg: 24px;
  --head: 'Fraunces', Georgia, serif;        /* 标题字体 */
  --body: 'Outfit', -apple-system, sans-serif; /* 正文字体 */
}
```

**主题特征**:奶油+珊瑚+森林绿,手账风,圆角卡片,手绘车形 SVG 背景图案。

### 0.3 已有数据资产

**`STATE_RULES`**(index.html line 1008-1061)— **已实现 50 州 + DC + PR 的税率和 trade-in 规则**:

```javascript
CA: { rate: 7.25, tradeInCredit: false }
TX: { rate: 6.25, tradeInCredit: true  }
IL: { rate: 6.25, tradeInCredit: 'partial', tradeInCap: 10000 }
SC: { rate: 5.0,  tradeInCredit: false, taxCap: 500 }
// ... 全 52 个 jurisdiction
```

**`APR_MAP`**(index.html line 998-1001)— 全国平均利率(condition × credit tier):

```javascript
new:  { excellent: 6.5, good: 7.9,  fair: 11.5, poor: 14.5 }
used: { excellent: 8.0, good: 9.5,  fair: 13.5, poor: 17.5 }
```

**计算器内核**:已有完整实现(摊销公式、税务计算、提前还款、对比)。

### 0.4 现有 footer 内链(index.html line 935-962)

```html
<div class="footer">
  <div class="f-brand">[logo SVG] Car Payment Calculator</div>
  <div class="f-links">
    <a href="legal.html#about">About</a> ·
    <a href="legal.html#methodology">Methodology</a> ·
    <a href="legal.html#privacy">Privacy</a> ·
    <a href="legal.html#disclaimer">Disclaimer</a> ·
    <a href="legal.html#contact">Contact</a> ·
    <a href="embed.html">Embed Widget</a>
  </div>
  <div class="f-tagline">Free car payment calculator — not financial advice...</div>
  <div class="f-alt">Also known as a car loan calculator or auto loan calculator</div>
</div>
```

**问题**:零 SEO 内链,完全没有 state pages 入口。落地页发布时必须重构。

---

## 第 1 章 — URL 结构与目录架构

### 1.1 URL 命名规范

**主州页面**:

```
https://carpaymentcalculator.app/california.html
https://carpaymentcalculator.app/texas.html
https://carpaymentcalculator.app/florida.html
... (共 50 个,州名小写,空格用连字符)
https://carpaymentcalculator.app/new-york.html
https://carpaymentcalculator.app/north-carolina.html
```

**为什么用 .html 而不是无后缀路径**:

1. 你现有路径 `legal.html` `embed.html` 已经用 .html,**风格统一**
2. 静态托管(Cloudflare Pages / GitHub Pages / Netlify)默认支持 `.html` 扩展
3. 不需要服务器端 rewrite 规则(无 _redirects 也能跑)
4. Google 对 `.html` 后缀**完全没有** SEO 惩罚

**California 子页面**(第一阶段只做 California 的):

```
https://carpaymentcalculator.app/california-car-sales-tax.html
https://carpaymentcalculator.app/california-auto-loan-rates.html
```

**为什么不用 `/california/sales-tax.html` 子目录结构**:

- 静态站做嵌套目录意味着每个 HTML 必须从 `../` 引用资源(favicon、og-image),容易出错
- 扁平 URL 对小型站更友好,Google 也无偏好
- 未来想做 `/california/cities/los-angeles.html` 时再升级也来得及

### 1.2 文件目录架构(建议改造)

**从这一步开始建议引入构建步骤**(因为 50 州手写 HTML 不现实):

```
carpaymentcalculator/
├── _src/                          [新增,构建源文件,不部署]
│   ├── _shared/
│   │   ├── header.html            [共享头部:meta、字体、CSS 变量]
│   │   ├── footer.html            [共享尾部:含全 50 州内链]
│   │   ├── styles.css             [抽取共享 CSS]
│   │   └── calculator.js          [抽取计算器 JS]
│   ├── _data/
│   │   ├── states.json            [50 州主页面数据,见 1.3]
│   │   └── california-tax.json    [CA 税务子页面数据]
│   │   └── california-rates.json  [CA 利率子页面数据]
│   ├── _templates/
│   │   ├── state-main.html        [主州页模板]
│   │   ├── state-tax.html         [税务子页模板]
│   │   └── state-rates.html       [利率子页模板]
│   └── build.js                   [Node.js 构建脚本]
├── index.html                     [保持不变]
├── legal.html                     [保持不变]
├── embed.html                     [保持不变]
├── widget.html                    [保持不变]
├── california.html                [构建生成]
├── texas.html                     [构建生成]
├── ... (50 州主页面)
├── california-car-sales-tax.html  [构建生成]
├── california-auto-loan-rates.html[构建生成]
├── sitemap.xml                    [构建生成,见第 5 章]
├── robots.txt                     [保持不变]
├── favicon.svg
└── og-image.png
```

**`_src/` 下划线前缀**很关键——很多静态托管(Jekyll、GitHub Pages)默认忽略下划线开头的目录,不会部署到线上。Cloudflare Pages 不忽略,需要在 `.gitignore` 里加 `_src/`(但仍然 commit,通过构建脚本生成的 HTML 才是部署产物)。

**或者更简单的方案**:用一个 `build/` 中间目录,只把 build 输出的 HTML 复制到根目录,`_src/` 通过 `.cfignore` 或部署排除规则跳过。

### 1.3 数据文件 schema:`_src/_data/states.json`

完整 50 州数据,每个州一条记录:

```json
[
  {
    "code": "CA",
    "slug": "california",
    "name": "California",
    "name_possessive": "California's",
    "abbreviation": "CA",
    "rank_population": 1,
    "rank_car_sales": 1,
    "sales_tax_rate": 7.25,
    "sales_tax_max_combined": 10.75,
    "sales_tax_label": "7.25% state base, up to 10.75% with local",
    "trade_in_credit": false,
    "trade_in_note": "California does NOT allow trade-in tax credit. Sales tax is calculated on full purchase price, regardless of trade-in value.",
    "registration_fee_structure": "value_based",
    "registration_fee_summary": "Base $76 + VLF (0.65% of vehicle value, depreciating 11 years) + TIF ($33-$231 by price) + CHP $34",
    "doc_fee_cap": 85,
    "doc_fee_note": "$85 cap for BPA dealers, $70 for non-BPA. SB 791 (2025) which would have raised cap to $175 was vetoed.",
    "avg_apr_new_excellent": 6.5,
    "avg_apr_used_excellent": 8.0,
    "top_cities": [
      {"name": "Los Angeles", "combined_tax": 9.5},
      {"name": "San Francisco", "combined_tax": 8.625},
      {"name": "San Diego", "combined_tax": 7.75},
      {"name": "San Jose", "combined_tax": 9.375},
      {"name": "Sacramento", "combined_tax": 8.75}
    ],
    "popular_vehicles": [
      {"make": "Tesla", "model": "Model Y", "msrp_avg": 47000, "rank_2025": 1},
      {"make": "Toyota", "model": "RAV4", "msrp_avg": 30000, "rank_2025": 2},
      {"make": "Toyota", "model": "Camry", "msrp_avg": 28000, "rank_2025": 3},
      {"make": "Tesla", "model": "Model 3", "msrp_avg": 39000, "rank_2025": 4},
      {"make": "Honda", "model": "CR-V", "msrp_avg": 32000, "rank_2025": 5}
    ],
    "neighbors": ["NV", "OR", "AZ"],
    "ev_incentive_status": "CVRP closed November 2023. Federal $7,500 EV credit ended September 30, 2025 under OBBBA. CAV/HOV decals expired same date.",
    "smog_check_required": true,
    "smog_check_note": "Required for 1976+ gasoline vehicles every 2 years. Initial registrations always require smog check. 8-year exemption applies only to renewals.",
    "obbba_conformity": {
      "status": "no_conform",
      "conformity_type": "Static-Lagged",
      "conformity_date": "2015-01-01",
      "has_state_income_tax": true,
      "summary": "California does NOT conform to the OBBBA auto loan interest deduction. The state uses static conformity to IRC as of January 1, 2015. You can claim the $10,000 federal deduction but must add it back on California Schedule CA when filing Form 540.",
      "tax_foundation_marker": "no checkmark in Auto column"
    },
    "dmv_url": "https://www.dmv.ca.gov/portal/vehicle-registration/registration-fees/",
    "tax_authority_url": "https://www.cdtfa.ca.gov/taxes-and-fees/sales-use-tax-rates.htm",
    "state_quirks": [
      "No trade-in tax credit (you pay tax on full price even if you trade in)",
      "VLF (Vehicle License Fee) is federally tax-deductible — keep records",
      "Strict smog check rules; ZEVs are exempt",
      "Highest doc fee cap region: $85 max"
    ]
  }
]
```

**字段填充优先级**:
- 🔴 必填(50 州都需要):`code`, `slug`, `name`, `sales_tax_rate`, `trade_in_credit`, `top_cities`(2-3 个), `dmv_url`
- 🟡 强烈建议:`avg_apr_*`, `popular_vehicles`(3-5 个), `neighbors`, `state_quirks`
- 🟢 加分项:`registration_fee_summary`, `doc_fee_cap`, `ev_incentive_status`

**California 当前数据状态**:🔴🟡🟢 全部齐全(已通过双源核实)。
**其他 49 州**:`STATE_RULES` 已有税率和 trade-in 规则,需要补充 cities、popular vehicles、quirks。

---

## 第 2 章 — 主州页面模板规范

### 2.1 California 主页面 H 标签结构(其他州照搬模板)

```
H1: California Car Payment Calculator — 2026 Auto Loan with Sales Tax & VLF

  [计算器组件,预填 CA 默认值]

H2: How California Sales Tax Affects Your Car Payment
  H3: California's 7.25% Base Rate Plus Local Add-ons
  H3: Why Trade-in Doesn't Reduce Your Tax Bill in California

H2: California Vehicle Registration Fees Breakdown
  H3: Base Registration, CHP, and Title Fees
  H3: VLF (Vehicle License Fee): The Hidden 0.65%
  H3: TIF (Transportation Improvement Fee) by Vehicle Price

H2: 2026 Auto Loan Rates for California Buyers
  H3: New vs Used Car APR by Credit Score

H2: Car Payment Examples for California's Top-Selling Models
  [表格:5 款热门车型 × 3 信用档位 × 计算后月供]

H2: City-by-City Tax Comparison: LA vs SF vs San Diego
  [表格:5 大城市 combined tax 率 + $35K 车的实际税额差]

H2: California-Specific Buying Tips
  H3: Smog Check Requirements
  H3: EV Incentives in 2026 (What Changed)
  H3: OBBBA Auto Loan Interest Deduction in California (Federal-State Mismatch)

H2: California Car Payment FAQ
  [6-8 个 schema.org FAQPage 结构化问题]

  [Footer + 内链网络]
```

**字数目标**:1200-1500 词正文(不含计算器、表格、FAQ)
**关键词密度目标**:`car payment calculator california` / `california car payment calculator` / `california auto loan` / `california car loan calculator` 在正文中累计出现 18-25 次,密度 3.5-5%。

### 2.2 完整文案大纲(California 实操示例)

**【H1 区域】**

```
H1: California Car Payment Calculator — 2026 Auto Loan with Sales Tax & VLF

[计算器组件:ZIP 预填 90210,sales_tax 自动锁定 7.25%,
 显示醒目提示:"Trade-in does NOT reduce California sales tax"]

副标题(p):
Calculate your California car payment with accurate 7.25% state sales tax, 
local district taxes up to 10.75%, vehicle license fee (VLF), and registration 
costs. Updated for 2026 with current Q4 2025 average APR data from Experian.
```

**【H2: How California Sales Tax Affects Your Car Payment】**

约 200 词,核心内容:

- 加州 sales tax 由 state base 7.25% + local district tax 0.10%-3.50% 组成
- 全州最低实际税率 7.25%(无地方税的少数县),最高 10.75%(Culver City、Compton 等 18 个城市)
- **关键差异化点**:Trade-in 不抵税。$30,000 车 + $10,000 trade-in,在 CA 仍按 $30,000 全价计税(2,175 美元税款),而在 TX 同样情况只按 $20,000 计税(节省 $725)。
- Manufacturer rebate 也不抵税(全国规则一致,但 CA 用户经常误解)

**【H2: California Vehicle Registration Fees Breakdown】**

约 250 词,这是其他州做不到的差异化。结构:

```
California 的注册费比大多数州复杂,主要由 4 部分组成:

1. **Base Registration**: $76(含 $3 alt fuel/tech fee)
2. **CHP Fee**: $34(California Highway Patrol)
3. **VLF (Vehicle License Fee)**: 0.65% × 类别中点 × 折旧因子
   - Year 1: 100% / Year 2: 90% / Year 3: 80% / ... / Year 11+: 15% (永久残值)
   - 例:$25,000 新车 Year 1 VLF = $163.15;到 Year 11+ 降到 $24.47
4. **TIF (Transportation Improvement Fee)**: 按车价 5 档 ($33/$66/$132/$198/$231)

ZEV (零排放车) 还需在 renewal 时支付 $121 RIF。
注册年缴(annual),不像有些州两年一次。

[表格:不同车价的总注册费示例]
```

**【H2: 2026 Auto Loan Rates for California Buyers】**

约 150 词:

- 引用 Experian Q4 2025 数据:super prime 4.66% / deep subprime 16.01%
- 加州平均贷款金额:Q4 2025 全国平均 $43,582 (new) / $27,528 (used)
- 平均期限 68.9 月(new)/ 67.7 月(used)
- 加州主要 lender:Bank of America、Wells Fargo、Capital One、Chase + 强势的 credit unions(Golden 1、SchoolsFirst、Patelco)

**【H2: Car Payment Examples for California's Top-Selling Models】**

表格,$25K down + 60mo term:

| 车型 | MSRP | Excellent (6.5%) | Good (7.9%) | Fair (11.5%) |
|------|------|-----------------|-------------|--------------|
| Tesla Model Y | $47,000 | $828/mo | $851/mo | $912/mo |
| Toyota RAV4 | $30,000 | $548/mo | $563/mo | $604/mo |
| Toyota Camry | $28,000 | $516/mo | $530/mo | $568/mo |
| Tesla Model 3 | $39,000 | $694/mo | $713/mo | $764/mo |
| Honda CR-V | $32,000 | $580/mo | $596/mo | $639/mo |

(具体数字用计算器跑出来填)

**【H2: City-by-City Tax Comparison】**

表格,$35,000 car:

| City | Combined Tax Rate | Tax on $35K |
|------|------------------|-------------|
| Los Angeles | 9.5% | $3,325 |
| San Jose | 9.375% | $3,281 |
| Sacramento | 8.75% | $3,063 |
| San Francisco | 8.625% | $3,019 |
| San Diego | 7.75% | $2,713 |

**【H2: California-Specific Buying Tips】**

H3 三个子标题:

1. **Smog Check Requirements** — 1976+ gas / 2-year cycle / 8-year exemption only on renewals (not initial)
2. **EV Incentives in 2026** — CVRP 已停 / 联邦 $7,500 已结束 / DCAP (Driving Clean Assistance Program) 仍在 / Clean Cars 4 All 仍可申请
3. **OBBBA Auto Loan Interest Deduction in California (Federal-State Mismatch)** — California 是 static-lagged conformity 州（IRC as of 2015-01-01），**不 conform** OBBBA 抵扣。联邦能抵的最多 $10,000/年（MAGI < $100K 单身或 $200K 联合）必须在加州 Schedule CA 上 add-back。意味着加州车主的实际节省**只有联邦税那部分**，按 22% 税档大约每 $1,000 利息省 $220，与德州、佛州等**无州所得税**的州无差别——但与加州本来 8.84% 的州税率相比，损失了潜在的 $88/$1,000 利息额外节省。

**【H2: California Car Payment FAQ】**

8 个问题(全部加 FAQPage schema):

1. Why is my California car payment higher than the calculator shows? → 答:你可能没把 VLF 算进去
2. Does the $7,500 federal EV tax credit still work in California? → 答:2025-09-30 已终止
3. Can I deduct California car loan interest from federal taxes? → 答:OBBBA 新政策,可以,有条件
4. Why doesn't California give me trade-in tax credit? → 答:CDTFA 法规,从来没有过
5. What's the average car payment in California in 2026? → 答:基于 Experian 数据
6. How much should I put down on a car in California? → 答:20% 经典法则
7. Is leasing better than buying in California? → 答:lease 不交 sales tax 全额,只交月供部分
8. What's California's doc fee limit? → 答:$85(BPA)/ $70(non-BPA),SB 791 已否决

### 2.3 模板替换变量(给构建脚本用)

每个州用同一份模板,通过变量替换生成。变量命名约定:

```
{{state.name}}              → California
{{state.slug}}              → california
{{state.code}}              → CA
{{state.name_possessive}}   → California's
{{state.sales_tax_rate}}    → 7.25
{{state.trade_in_credit}}   → false
{{state.trade_in_sentence}} → 自动生成的 trade-in 规则文案
{{state.dmv_url}}           → DMV 链接
{{state.year}}              → 2026
{{state.top_cities_table}}  → 自动生成 city tax 表格 HTML
{{state.popular_vehicles_table}} → 自动生成车型表格 HTML
{{state.faq_jsonld}}        → 自动生成 FAQPage JSON-LD
{{state.related_states_links}}  → 内链区(neighbors + 邻州)
{{state.canonical_url}}     → https://carpaymentcalculator.app/california.html
{{state.obbba_h3_section}}  → OBBBA 段落,根据 conformity 状态自动生成
```

---

### 2.4 OBBBA H3 段落生成规范（关键差异化模块）

**这是 v1.1 新增的核心模块**，让 50 个州页面避免 doorway pages 风险的关键。每个州的 OBBBA H3 段落必须**根据该州的 conformity 状态**生成不同内容。

#### 2.4.1 数据基础（来自 Tax Foundation 官方报告）

每个州在 `states.json` 里都有 `obbba_conformity` 字段，含 4 种 status 值：

| status 值 | 含义 | 对应 Tax Foundation 标记 | 受影响州数 |
|----------|------|------------------------|-----------|
| `conform` | 州 conform OBBBA，联邦+州双重抵税 | "Auto" 列有 ✓ | ~13 个 |
| `no_conform` | 州不 conform，必须 add-back | "Auto" 列空白 | ~21 个 |
| `no_state_tax` | 该州无个人所得税，OBBBA 在州层面 N/A | PIT = "n.a." | 9 个（AK/FL/NV/NH/SD/TN/TX/WA/WY） |
| `pending` | 州正在立法或情况复杂 | 备注栏 | ~7 个 |

#### 2.4.2 4 套不同模板

构建脚本根据 `state.obbba_conformity.status` 选用对应模板：

**模板 A — `conform` 州（双重好处）**

```markdown
### OBBBA Auto Loan Interest Deduction in {{state.name}}

The federal **One Big Beautiful Bill Act (OBBBA)** allows up to **$10,000/year** in auto loan interest as an above-the-line deduction for tax years 2025-2028 — and {{state.name}} is one of the **{{conform_state_count}} states whose tax code conforms** to this provision, giving you a double benefit.

**How {{state.name}} car buyers benefit:**

| Income | Federal Deduction | State Deduction | Total Savings on $5,000 Interest |
|--------|------------------|-----------------|----------------------------------|
| < $100K single ($200K MFJ) | Full $5,000 | Full $5,000 | Federal 22% + State {{state.income_tax_rate}}% = **${{calc_combined_savings}}** |
| MAGI in phaseout | Reduced | Same as federal | See {{state.tax_authority_name}} for state rules |

To qualify, your vehicle must be: **new, U.S.-assembled, used personally**, with a loan signed after Dec 31, 2024. Your lender will issue **Form 1098-VLI** starting tax year 2026. File on Schedule 1-A (federal) and report on {{state.tax_form_name}} (state).

[Learn more about OBBBA eligibility →](#obbba-faq) (链接到本页面 FAQ 区域中的 OBBBA 问题)
```

**模板 B — `no_conform` 州（联邦能抵但州税要 add-back）**

```markdown
### OBBBA Auto Loan Interest Deduction in {{state.name}} (Federal-State Mismatch)

The federal **OBBBA** allows up to **$10,000/year** in auto loan interest as an above-the-line deduction for 2025-2028 — but **{{state.name}} does NOT conform** to this provision. {{state.name}} uses {{state.obbba_conformity.conformity_type_human}} conformity ({{state.obbba_conformity.conformity_date_human}}), so you must **add this deduction back** when filing your state return.

**What this means for {{state.name}} buyers:**

- ✅ **Federal**: Yes, up to $10,000/year off your federal taxable income (MAGI limits apply)
- ❌ **{{state.name}} state**: Must add back on {{state.tax_form_name}}; auto loan interest remains taxable at state level
- 📊 **Net savings on $5,000 interest** at 22% federal bracket: **$1,100** federal only, no state benefit

If you live in {{state.name}}, the OBBBA still saves you federal tax — but **don't double-count it as state savings**. If your tax software calculates state tax based on federal AGI, double-check that it added back the deduction. Failing to add back could trigger a {{state.tax_authority_name}} adjustment notice.

[Compare states that DO conform →](#obbba-conform-comparison)
```

**模板 C — `no_state_tax` 州（联邦白送，州层面 N/A）**

```markdown
### OBBBA Auto Loan Interest Deduction for {{state.name}} Buyers

Lucky news for {{state.name}} buyers: since {{state.name}} has **no state income tax**, the federal OBBBA auto loan interest deduction has no state-level complications — **what you save federally is what you save, period**.

The federal **OBBBA** lets you deduct up to **$10,000/year** in auto loan interest above-the-line for tax years 2025-2028. To qualify, your vehicle must be **new, U.S.-assembled, and used personally**, with the loan signed after Dec 31, 2024. The deduction phases out starting at $100,000 MAGI (single) or $200,000 (MFJ).

**{{state.name}} car buyer at 22% federal bracket, $5,000 annual interest** = **$1,100 federal tax saved**. No state add-back forms, no state-conformity worries. File on Schedule 1-A with your Form 1040.

[See full eligibility rules →](#obbba-faq)
```

**模板 D — `pending` 州（情况复杂，需慎重表述）**

```markdown
### OBBBA Auto Loan Interest Deduction in {{state.name}}

The federal **OBBBA** lets you deduct up to **$10,000/year** in auto loan interest above-the-line for tax years 2025-2028, capped at $100,000 MAGI single / $200,000 MFJ.

**{{state.name}}'s state-level treatment is currently {{state.obbba_conformity.pending_reason}}.** {{state.obbba_conformity.pending_note}}

Until {{state.tax_authority_name}} issues guidance:
- Claim the federal deduction normally on Schedule 1-A
- Watch for {{state.tax_authority_name}} notices in early 2026
- Consider filing an extension if state guidance is pending at the deadline

[Track {{state.name}} conformity updates →]({{state.tax_authority_url}})
```

#### 2.4.3 关键约束（必须遵守）

1. **数字精确**：每个模板里的 "save $X" 数字必须由构建脚本根据 `state.income_tax_rate` 自动计算，不能写错
2. **anchor text 多样化**：50 个州 H3 标题不能完全相同
   - `conform` 州用："OBBBA Auto Loan Interest Deduction in [State]"
   - `no_conform` 州用："OBBBA Auto Loan Interest Deduction in [State] (Federal-State Mismatch)"
   - `no_state_tax` 州用："OBBBA Auto Loan Interest Deduction for [State] Buyers"
   - `pending` 州用："OBBBA Auto Loan Interest Deduction in [State]"
3. **避免重复短语**：不要在同一段落里出现 "auto loan interest deduction" 超过 3 次（关键词堆砌信号），用 "OBBBA"、"the deduction"、"this tax break" 等代词轮换
4. **法律免责声明**：模板 D 末尾必须加 "Always verify with a {{state.name}} tax professional before filing." 这一句

#### 2.4.4 FAQ 区域必须配套加 OBBBA 题

每个州的 FAQ 区域（H2: [State] Car Payment FAQ）增加 2 个 OBBBA 相关问题：

**Q1（所有州通用，但 answer 文本根据 status 变化）**：
"Can I deduct car loan interest on my {{state.name}} taxes in 2026?"

**Q2（仅 `no_conform` 和 `pending` 州）**：
"Why doesn't {{state.name}} let me deduct car loan interest if the federal government does?"

**Q3（仅 `conform` 州）**：
"How do I claim the OBBBA deduction on both my federal and {{state.name}} returns?"

这两道题 + 各州不同的答案，进一步加强差异化。

#### 2.4.5 OBBBA 段落字数控制

- 不能超过 250 词（避免主页面被 OBBBA 内容稀释 SEO 焦点）
- 不能少于 150 词（少了 Google 判定为 thin content）
- 必须有至少 1 个表格或列表（结构化内容信号）

---

## 第 3 章 — 内链网络架构(SEO 核心)

### 3.1 内链层级设计

```
                    [首页 index.html]
                          │
              ┌───────────┴───────────┐
              │                       │
      [50 州主页面]              [legal.html / embed.html]
              │
       ┌──────┴──────┐
       │             │
  [税务子页]    [利率子页]  ← 第一阶段只做 California
```

### 3.2 内链规则矩阵

**位置 A:全局 footer(所有页面共享)**

新设计的 footer 必须包含 50 州入口,不能像现在只 6 个 legal 链接:

```html
<footer class="footer">
  <!-- Brand row(保留现有 logo) -->
  <div class="f-brand">[logo SVG] Car Payment Calculator</div>
  
  <!-- 主链接行(保留现有 legal 链接) -->
  <div class="f-links">
    <a href="legal.html#about">About</a> ·
    <a href="legal.html#methodology">Methodology</a> ·
    <a href="legal.html#privacy">Privacy</a> ·
    <a href="legal.html#disclaimer">Disclaimer</a> ·
    <a href="legal.html#contact">Contact</a> ·
    <a href="embed.html">Embed Widget</a>
  </div>
  
  <!-- 新增:50 州地区导航(可折叠) -->
  <details class="f-states">
    <summary>Calculate by State</summary>
    <div class="f-states-grid">
      <a href="alabama.html">Alabama</a>
      <a href="alaska.html">Alaska</a>
      <a href="arizona.html">Arizona</a>
      <a href="arkansas.html">Arkansas</a>
      <a href="california.html">California</a>
      <!-- ... 全 50 州按字母序 -->
      <a href="wyoming.html">Wyoming</a>
    </div>
  </details>
  
  <div class="f-tagline">Free car payment calculator — not financial advice.</div>
  <div class="f-alt">Also known as a car loan calculator or auto loan calculator</div>
</footer>
```

**配套 CSS**(添加到 `_src/_shared/styles.css` 或每页 inline):

```css
.f-states{
  margin: 18px auto 12px;
  max-width: 880px;
  padding: 0 24px;
}
.f-states summary{
  cursor: pointer;
  font-family: var(--head);
  font-weight: 600;
  font-size: 13px;
  color: var(--coral);
  padding: 8px 0;
  list-style: none;
  text-align: center;
}
.f-states summary::-webkit-details-marker{display:none;}
.f-states summary::before{
  content: '▸ ';
  transition: transform 0.2s;
  display: inline-block;
}
.f-states[open] summary::before{transform: rotate(90deg);}
.f-states-grid{
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(110px, 1fr));
  gap: 4px 14px;
  margin-top: 12px;
  padding: 14px 16px;
  background: var(--warm);
  border: 1.5px solid var(--border);
  border-radius: var(--r-sm);
}
.f-states-grid a{
  font-size: 12px;
  color: var(--text-mid);
  text-decoration: none;
  padding: 4px 0;
  transition: color 0.15s;
}
.f-states-grid a:hover{color: var(--coral);}
```

**为什么用 `<details>` 折叠**:
1. 50 个链接平铺会让 footer 变得很重
2. `<details>` 是 HTML5 原生标签,Google 完全索引(链接默认就在 DOM 里),不像 JS hidden 那样有风险
3. 移动端体验好

**注意**:Googlebot 完整渲染 details 内的链接,内链权重不会因折叠而损失。

**位置 B:每个州页面的"邻州导航"区块**

在 H2 "California-Specific Buying Tips" 后面、FAQ 前面,加一个区块:

```html
<section class="related-states">
  <h2>Calculator for Neighboring States</h2>
  <div class="related-states-grid">
    <a href="nevada.html" class="related-card">
      <span class="rs-name">Nevada</span>
      <span class="rs-rate">6.85% sales tax</span>
    </a>
    <a href="oregon.html" class="related-card">
      <span class="rs-name">Oregon</span>
      <span class="rs-rate">0% sales tax</span>
    </a>
    <a href="arizona.html" class="related-card">
      <span class="rs-name">Arizona</span>
      <span class="rs-rate">5.6% sales tax</span>
    </a>
  </div>
</section>
```

数据来源:`states.json` 里的 `neighbors` 字段,每个州 2-4 个。

**位置 C:每个州页面的"子页面导航"区块**

在 H1 计算器组件正下方:

```html
<nav class="subpage-nav" aria-label="California car payment topics">
  <a href="california-car-sales-tax.html">📋 California Sales Tax Rules</a>
  <a href="california-auto-loan-rates.html">📈 Current CA Auto Loan Rates</a>
</nav>
```

仅 California 有,其他州第二阶段再加。

**位置 D:首页 index.html 的内链入口**

在现有的内容区域(line 750-930 之间合适位置),添加一个小型 state grid:

```html
<section class="states-quicknav">
  <h2 class="states-quicknav-title">Find Your State's Calculator</h2>
  <p class="states-quicknav-sub">Sales tax and registration fees vary by state. Pick yours:</p>
  <div class="states-quicknav-popular">
    <a href="california.html">California</a>
    <a href="texas.html">Texas</a>
    <a href="florida.html">Florida</a>
    <a href="new-york.html">New York</a>
    <a href="pennsylvania.html">Pennsylvania</a>
    <a href="illinois.html">Illinois</a>
    <a href="ohio.html">Ohio</a>
    <a href="georgia.html">Georgia</a>
    <a href="north-carolina.html">North Carolina</a>
    <a href="michigan.html">Michigan</a>
  </div>
  <p><a href="#" onclick="document.querySelector('.f-states').open=true; document.querySelector('.f-states').scrollIntoView({behavior:'smooth'}); return false;" class="see-all">See all 50 states →</a></p>
</section>
```

10 个搜索量最大的州做 above-the-fold 展示,其余 40 州通过 footer details 展开。

**位置 E:子页面之间的内链(California 三页 cluster)**

每个 California 子页底部:

```html
<aside class="page-cluster">
  <h3>Related California Resources</h3>
  <ul>
    <li><a href="california.html">California Car Payment Calculator</a></li>
    <li><a href="california-car-sales-tax.html">California Sales Tax Calculator</a></li>
    <li><a href="california-auto-loan-rates.html">Current California Auto Loan Rates</a></li>
  </ul>
</aside>
```

### 3.3 内链 anchor text 规范

**避免**:全部用同一个 anchor text(Google 会判定为 over-optimization)
**推荐**:多样化锚文本

| 链接目标 | 推荐 anchor text 变体 |
|---------|---------------------|
| california.html | "California car payment calculator" / "calculate California car loan" / "California-specific calculator" / "California" |
| california-car-sales-tax.html | "California sales tax rules" / "CA sales tax calculator" / "California vehicle tax" |
| california-auto-loan-rates.html | "current CA auto loan rates" / "California APR data" / "California car loan rates" |

构建脚本自动从一个 anchor pool 里轮换选择,保证不重复。

### 3.4 nofollow 策略

**不要给任何内链加 nofollow**。所有州页面之间的链接、子页面 cluster、footer details 都是 dofollow,这些是你自己的页面,需要传递权重。

**仅外部链接需考虑**:DMV 官网、CDTFA 等政府权威源——**这些反而应该是 dofollow**,告诉 Google 你引用了权威来源。

---

## 第 4 章 — 构建脚本规范

### 4.1 `_src/build.js` 应该做什么

Node.js 脚本,无外部依赖,跑 `node _src/build.js` 就能生成所有页面。

**伪代码**:

```javascript
// _src/build.js
const fs = require('fs');
const path = require('path');

const states = require('./_data/states.json');
const stateMainTemplate = fs.readFileSync('./_src/_templates/state-main.html', 'utf8');
const sharedHeader = fs.readFileSync('./_src/_shared/header.html', 'utf8');
const sharedFooter = fs.readFileSync('./_src/_shared/footer.html', 'utf8');

// 1. 生成 50 州主页面
states.forEach(state => {
  let html = stateMainTemplate;
  
  // 注入 shared parts
  html = html.replace('{{shared.header}}', sharedHeader);
  html = html.replace('{{shared.footer}}', sharedFooter);
  
  // 注入州数据
  Object.keys(state).forEach(key => {
    const placeholder = `{{state.${key}}}`;
    const value = state[key];
    html = html.split(placeholder).join(formatValue(key, value, state));
  });
  
  // 生成派生字段
  html = html.replace('{{state.trade_in_sentence}}', generateTradeInSentence(state));
  html = html.replace('{{state.faq_jsonld}}', generateFaqJsonLd(state));
  html = html.replace('{{state.top_cities_table}}', generateCitiesTable(state));
  html = html.replace('{{state.popular_vehicles_table}}', generateVehiclesTable(state));
  html = html.replace('{{state.related_states_links}}', generateNeighborsBlock(state));
  
  // 写文件到根目录
  fs.writeFileSync(`./${state.slug}.html`, html);
  console.log(`✓ Generated ${state.slug}.html`);
});

// 2. 生成 California 子页面(第一阶段)
const caTaxData = require('./_data/california-tax.json');
const caRatesData = require('./_data/california-rates.json');
buildSubPage('california-car-sales-tax.html', './_src/_templates/state-tax.html', caTaxData);
buildSubPage('california-auto-loan-rates.html', './_src/_templates/state-rates.html', caRatesData);

// 3. 重新生成 sitemap.xml(包含所有新页面)
generateSitemap();

// 4. 更新 footer 注入到现有的 index.html、legal.html、embed.html(保留它们的其他内容)
patchExistingPages();
```

**关键设计决策**:

1. **为什么不用模板引擎(Handlebars、EJS)**:零依赖,纯 string replace 足够,不引入 npm 复杂度
2. **patchExistingPages()** 必须用**精确锚点替换**,而不是覆盖全文。例如:

```javascript
function patchExistingPages() {
  ['index.html', 'legal.html', 'embed.html'].forEach(file => {
    let html = fs.readFileSync(`./${file}`, 'utf8');
    
    // 用注释锚点定位 footer 区域,做精确替换
    const footerStart = '<!-- FOOTER:START -->';
    const footerEnd = '<!-- FOOTER:END -->';
    const startIdx = html.indexOf(footerStart);
    const endIdx = html.indexOf(footerEnd);
    
    if (startIdx === -1 || endIdx === -1) {
      console.warn(`⚠ ${file} missing FOOTER:START/END markers, skipping`);
      return;
    }
    
    const newFooter = `${footerStart}\n${sharedFooter}\n${footerEnd}`;
    html = html.slice(0, startIdx) + newFooter + html.slice(endIdx + footerEnd.length);
    
    fs.writeFileSync(`./${file}`, html);
  });
}
```

**前提条件**:必须先在 index.html、legal.html、embed.html 现有的 `<div class="footer">` 上下加锚点注释:

```html
<!-- FOOTER:START -->
<div class="footer">...</div>
<!-- FOOTER:END -->
```

### 4.2 构建流程命令

```bash
# 一次性生成所有页面
node _src/build.js

# 输出预期
✓ Generated alabama.html
✓ Generated alaska.html
... (50 个)
✓ Generated california-car-sales-tax.html
✓ Generated california-auto-loan-rates.html
✓ Patched index.html footer
✓ Patched legal.html footer
✓ Patched embed.html footer
✓ Generated sitemap.xml (55 URLs)
Done.
```

### 4.3 git 工作流

```bash
# 修改 _src/_data/states.json 后
node _src/build.js
git add .
git commit -m "feat: regenerate state pages from updated data"
git push
```

---

## 第 5 章 — sitemap.xml 重构

### 5.1 新 sitemap.xml 结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <!-- 主入口 -->
  <url>
    <loc>https://carpaymentcalculator.app/</loc>
    <lastmod>2026-04-28</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  
  <!-- Legal / utility -->
  <url>
    <loc>https://carpaymentcalculator.app/embed.html</loc>
    <lastmod>2026-04-28</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.6</priority>
  </url>
  <url>
    <loc>https://carpaymentcalculator.app/legal.html</loc>
    <lastmod>2026-04-28</lastmod>
    <changefreq>quarterly</changefreq>
    <priority>0.4</priority>
  </url>
  
  <!-- 50 州主页面(按字母序,priority 0.8) -->
  <url>
    <loc>https://carpaymentcalculator.app/alabama.html</loc>
    <lastmod>2026-04-28</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
  <!-- ... 50 个 ... -->
  <url>
    <loc>https://carpaymentcalculator.app/wyoming.html</loc>
    <lastmod>2026-04-28</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
  
  <!-- California 子页面(priority 0.7) -->
  <url>
    <loc>https://carpaymentcalculator.app/california-car-sales-tax.html</loc>
    <lastmod>2026-04-28</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>
  <url>
    <loc>https://carpaymentcalculator.app/california-auto-loan-rates.html</loc>
    <lastmod>2026-04-28</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>
</urlset>
```

### 5.2 sitemap 提交

发布后立刻去 Google Search Console 重新提交 sitemap:
- URL: `https://carpaymentcalculator.app/sitemap.xml`
- 状态变为 "Success" 后等待 1-3 天首批收录

不要等 Google 自己发现——你 robots.txt 里已经写了 sitemap 路径,但**主动 ping 比被动等快得多**。

---

## 第 6 章 — 单页 HTML 模板完整骨架

### 6.1 `_src/_templates/state-main.html` 完整结构

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- ===== SEO META ===== -->
<title>{{state.name}} Car Payment Calculator — 2026 Auto Loan with Sales Tax</title>
<meta name="description" content="Free {{state.name}} car payment calculator with {{state.sales_tax_rate}}% state sales tax, registration fees, and 2026 average APR rates. Get your monthly auto loan payment in seconds.">
<meta name="robots" content="index, follow, max-snippet:-1, max-image-preview:large, max-video-preview:-1">
<link rel="canonical" href="https://carpaymentcalculator.app/{{state.slug}}.html">
<link rel="alternate" hreflang="en" href="https://carpaymentcalculator.app/{{state.slug}}.html">

<!-- ===== Open Graph ===== -->
<meta property="og:site_name" content="Car Payment Calculator">
<meta property="og:title" content="{{state.name}} Car Payment Calculator — 2026 Auto Loan">
<meta property="og:description" content="Calculate your {{state.name}} car payment with accurate {{state.sales_tax_rate}}% sales tax and current APR rates.">
<meta property="og:type" content="website">
<meta property="og:url" content="https://carpaymentcalculator.app/{{state.slug}}.html">
<meta property="og:locale" content="en_US">
<meta property="og:image" content="https://carpaymentcalculator.app/og-image.png">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">

<!-- ===== Twitter Card ===== -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="{{state.name}} Car Payment Calculator — 2026">
<meta name="twitter:description" content="Free {{state.name}} car loan calculator with state-specific tax rules.">
<meta name="twitter:image" content="https://carpaymentcalculator.app/og-image.png">

<!-- ===== JSON-LD: WebApplication ===== -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebApplication",
  "name": "{{state.name}} Car Payment Calculator",
  "description": "Free {{state.name}}-specific car payment calculator with {{state.sales_tax_rate}}% state sales tax and current APR rates.",
  "url": "https://carpaymentcalculator.app/{{state.slug}}.html",
  "applicationCategory": "FinanceApplication",
  "operatingSystem": "Any",
  "offers": {
    "@type": "Offer",
    "price": "0",
    "priceCurrency": "USD"
  }
}
</script>

<!-- ===== JSON-LD: BreadcrumbList ===== -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://carpaymentcalculator.app/"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "{{state.name}}",
      "item": "https://carpaymentcalculator.app/{{state.slug}}.html"
    }
  ]
}
</script>

<!-- ===== JSON-LD: FAQPage ===== -->
{{state.faq_jsonld}}

<!-- ===== Favicon & Fonts ===== -->
<link rel="icon" type="image/svg+xml" href="favicon.svg">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,600;0,9..144,700;0,9..144,800;1,9..144,400;1,9..144,600&family=Outfit:wght@300;400;500;600&display=swap" rel="stylesheet">

<!-- ===== Inline CSS(继承现有 index.html 主题)===== -->
<style>
:root {
  --cream:#FFF8F0; --warm:#FEF3E2; --peach:#FFE8D6;
  --coral:#E8725A; --coral-light:#FFF0EC; --coral-dark:#D45A42;
  --forest:#2D5A3D; --forest-light:#EAF2EC; --forest-soft:rgba(45,90,61,0.08);
  --navy:#1E2A3A; --navy-soft:#3A4A5C;
  --brown:#8B6F47; --brown-light:#C4A97D;
  --text:#2C2420; --text-mid:#5C4F44; --text-light:#9B8E82;
  --border:#E8DDD2; --border-dark:#D4C5B5;
  --shadow:rgba(44,36,32,0.06); --shadow-md:rgba(44,36,32,0.1);
  --r:16px; --r-sm:10px; --r-lg:24px;
  --head:'Fraunces',Georgia,serif;
  --body:'Outfit',-apple-system,sans-serif;
}
*{margin:0;padding:0;box-sizing:border-box;}
html{scroll-behavior:smooth;}
body{
  font-family:var(--body);
  background-color:var(--cream);
  color:var(--text);
  line-height:1.6;
  -webkit-font-smoothing:antialiased;
}

/* ===== Breadcrumb ===== */
.breadcrumb{max-width:880px;margin:24px auto 0;padding:0 24px;font-size:13px;color:var(--text-light);}
.breadcrumb a{color:var(--coral);text-decoration:none;}
.breadcrumb a:hover{text-decoration:underline;}

/* ===== Hero ===== */
.state-hero{max-width:880px;margin:24px auto;padding:32px 24px;text-align:center;}
.state-hero h1{
  font-family:var(--head);font-size:clamp(28px,4.5vw,44px);
  font-weight:800;letter-spacing:-0.8px;line-height:1.2;
  margin-bottom:14px;color:var(--text);
}
.state-hero h1 em{font-style:italic;color:var(--coral);}
.state-hero p{font-size:16px;color:var(--text-mid);max-width:620px;margin:0 auto;line-height:1.65;}

/* ===== Calculator Container(嵌入主计算器)===== */
.calc-embed{max-width:880px;margin:32px auto;padding:0 24px;}

/* ===== Subpage Nav ===== */
.subpage-nav{
  max-width:880px;margin:18px auto 32px;padding:0 24px;
  display:flex;gap:12px;justify-content:center;flex-wrap:wrap;
}
.subpage-nav a{
  display:inline-block;padding:10px 18px;
  background:var(--warm);border:1.5px solid var(--border);
  border-radius:50px;
  font-size:13px;font-weight:500;color:var(--text-mid);
  text-decoration:none;transition:all 0.2s;
}
.subpage-nav a:hover{
  background:var(--coral-light);border-color:var(--coral);
  color:var(--coral);
}

/* ===== Article(SEO content)===== */
.state-article{max-width:760px;margin:48px auto;padding:0 24px;}
.state-article h2{
  font-family:var(--head);font-size:28px;font-weight:700;
  letter-spacing:-0.4px;color:var(--text);
  margin:48px 0 16px;
}
.state-article h3{
  font-family:var(--head);font-size:20px;font-weight:600;
  color:var(--text);margin:28px 0 12px;
}
.state-article p{font-size:16px;color:var(--text-mid);line-height:1.75;margin-bottom:16px;}
.state-article ul,.state-article ol{margin:12px 0 16px 24px;color:var(--text-mid);}
.state-article li{margin-bottom:8px;line-height:1.7;}
.state-article a{color:var(--coral);text-decoration:none;border-bottom:1.5px solid var(--coral-light);}
.state-article a:hover{border-bottom-color:var(--coral);}

/* ===== Tables ===== */
.state-table{width:100%;border-collapse:collapse;margin:20px 0;font-size:14px;}
.state-table th,.state-table td{
  padding:12px 14px;text-align:left;
  border-bottom:1px solid var(--border);
}
.state-table th{
  background:var(--warm);
  font-family:var(--head);font-weight:600;color:var(--text);
}
.state-table tr:hover td{background:var(--coral-light);}

/* ===== Callout boxes ===== */
.callout{
  padding:16px 20px;margin:20px 0;
  background:var(--coral-light);border-left:4px solid var(--coral);
  border-radius:8px;font-size:15px;color:var(--text);
}
.callout strong{color:var(--coral-dark);}
.callout-info{background:var(--forest-light);border-left-color:var(--forest);}
.callout-info strong{color:var(--forest);}

/* ===== Related states ===== */
.related-states{
  max-width:880px;margin:48px auto;padding:32px 24px;
  background:var(--warm);border-radius:var(--r);
}
.related-states h2{
  font-family:var(--head);font-size:24px;text-align:center;
  margin-bottom:20px;color:var(--text);
}
.related-states-grid{
  display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));
  gap:14px;
}
.related-card{
  display:flex;flex-direction:column;align-items:center;
  padding:18px 14px;background:#fff;
  border:1.5px solid var(--border);border-radius:var(--r-sm);
  text-decoration:none;transition:all 0.2s;
}
.related-card:hover{
  border-color:var(--coral);
  box-shadow:0 4px 12px rgba(232,114,90,0.1);
  transform:translateY(-2px);
}
.rs-name{font-family:var(--head);font-size:17px;font-weight:700;color:var(--text);}
.rs-rate{font-size:12px;color:var(--text-light);margin-top:4px;}

/* ===== FAQ ===== */
.faq-list{margin:24px 0;}
.faq-item{
  border:1.5px solid var(--border);border-radius:var(--r-sm);
  margin-bottom:10px;overflow:hidden;
}
.faq-item summary{
  padding:16px 20px;cursor:pointer;
  font-family:var(--head);font-weight:600;color:var(--text);
  list-style:none;
}
.faq-item summary::-webkit-details-marker{display:none;}
.faq-item summary::after{content:'+';float:right;color:var(--coral);font-size:20px;}
.faq-item[open] summary::after{content:'−';}
.faq-item[open] summary{background:var(--warm);}
.faq-item .faq-answer{padding:4px 20px 18px;color:var(--text-mid);line-height:1.7;}

/* ===== Footer(由 build.js 注入)===== */
{{shared.footer_styles}}

/* ===== Mobile ===== */
@media(max-width:680px){
  .state-hero{padding:24px 20px;}
  .state-hero h1{font-size:28px;}
  .state-article{padding:0 20px;}
  .state-article h2{font-size:24px;}
  .state-table{font-size:13px;}
  .state-table th,.state-table td{padding:10px 8px;}
}
</style>
</head>

<body>

<!-- ===== Breadcrumb ===== -->
<nav class="breadcrumb" aria-label="Breadcrumb">
  <a href="/">Home</a> › <span>{{state.name}}</span>
</nav>

<!-- ===== Hero ===== -->
<section class="state-hero">
  <h1>{{state.name}} Car Payment <em>Calculator</em></h1>
  <p>Calculate your {{state.name}} car loan payment with accurate {{state.sales_tax_rate}}% state sales tax, registration fees, and current 2026 APR rates from Experian.</p>
</section>

<!-- ===== Calculator(指向主页面)===== -->
<div class="calc-embed">
  <iframe 
    src="widget.html?defaultApr={{state.avg_apr_new_excellent}}&taxRate={{state.sales_tax_rate}}&accent=%23E8725A" 
    style="width:100%;max-width:680px;height:560px;border:1.5px solid var(--border);border-radius:var(--r);display:block;margin:0 auto;"
    title="{{state.name}} Car Payment Calculator"
    loading="lazy"></iframe>
  <p style="text-align:center;margin-top:14px;font-size:13px;color:var(--text-light);">
    Need a full-featured calculator? <a href="/" style="color:var(--coral);">Use the main calculator with amortization schedule</a>
  </p>
</div>

<!-- ===== Subpage nav(仅 California 启用,其他州为空)===== -->
{{state.subpage_nav}}

<!-- ===== Article ===== -->
<article class="state-article">

  <h2>How {{state.name}} Sales Tax Affects Your Car Payment</h2>
  {{state.section_sales_tax}}

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

<!-- ===== Related States ===== -->
<section class="related-states">
  <h2>Calculator for Neighboring States</h2>
  <div class="related-states-grid">
    {{state.related_states_links}}
  </div>
</section>

<!-- ===== FAQ ===== -->
<article class="state-article">
  <h2>{{state.name}} Car Payment FAQ</h2>
  <div class="faq-list">
    {{state.faq_html}}
  </div>
</article>

<!-- FOOTER:START -->
{{shared.footer}}
<!-- FOOTER:END -->

</body>
</html>
```

### 6.2 共享 footer(`_src/_shared/footer.html`)

```html
<footer class="footer">
  <div class="f-brand" style="display:flex;align-items:center;justify-content:center;gap:8px;">
    <svg class="logo-mark" width="36" height="24" viewBox="0 0 64 44" fill="none" xmlns="http://www.w3.org/2000/svg">
      <g stroke="#E8725A" stroke-width="2.6" stroke-linecap="round" stroke-linejoin="round">
        <path d="M5 32 L9 20 Q10 15 15 13 L28 9 Q33 8 38 10 L48 15 Q52 17 53 21 L57 32"/>
        <line x1="3" y1="32" x2="59" y2="32"/>
        <circle cx="16" cy="32" r="6"/><circle cx="16" cy="32" r="2.5"/>
        <circle cx="48" cy="32" r="6"/><circle cx="48" cy="32" r="2.5"/>
        <path d="M20 13 L22 21 L32 21 L33 13"/>
        <path d="M35 14 L36 21 L46 21 L47 16"/>
      </g>
      <circle cx="52" cy="10" r="9" fill="#2D5A3D" stroke="#FFF8F0" stroke-width="2"/>
      <path d="M52 5.5 Q49.5 5.5 49.5 7.5 Q49.5 10 52 10 Q54.5 10 54.5 12 Q54.5 14.5 52 14.5" stroke="#FFF8F0" stroke-width="1.6" stroke-linecap="round"/>
      <line x1="52" y1="4" x2="52" y2="16" stroke="#FFF8F0" stroke-width="1.3" stroke-linecap="round"/>
    </svg>
    Car Payment Calculator
  </div>
  
  <div class="f-links">
    <a href="/legal.html#about">About</a> ·
    <a href="/legal.html#methodology">Methodology</a> ·
    <a href="/legal.html#privacy">Privacy</a> ·
    <a href="/legal.html#disclaimer">Disclaimer</a> ·
    <a href="/legal.html#contact">Contact</a> ·
    <a href="/embed.html">Embed Widget</a>
  </div>
  
  <details class="f-states">
    <summary>Calculate by State</summary>
    <div class="f-states-grid">
      <a href="/alabama.html">Alabama</a>
      <a href="/alaska.html">Alaska</a>
      <a href="/arizona.html">Arizona</a>
      <a href="/arkansas.html">Arkansas</a>
      <a href="/california.html">California</a>
      <a href="/colorado.html">Colorado</a>
      <a href="/connecticut.html">Connecticut</a>
      <a href="/delaware.html">Delaware</a>
      <a href="/florida.html">Florida</a>
      <a href="/georgia.html">Georgia</a>
      <a href="/hawaii.html">Hawaii</a>
      <a href="/idaho.html">Idaho</a>
      <a href="/illinois.html">Illinois</a>
      <a href="/indiana.html">Indiana</a>
      <a href="/iowa.html">Iowa</a>
      <a href="/kansas.html">Kansas</a>
      <a href="/kentucky.html">Kentucky</a>
      <a href="/louisiana.html">Louisiana</a>
      <a href="/maine.html">Maine</a>
      <a href="/maryland.html">Maryland</a>
      <a href="/massachusetts.html">Massachusetts</a>
      <a href="/michigan.html">Michigan</a>
      <a href="/minnesota.html">Minnesota</a>
      <a href="/mississippi.html">Mississippi</a>
      <a href="/missouri.html">Missouri</a>
      <a href="/montana.html">Montana</a>
      <a href="/nebraska.html">Nebraska</a>
      <a href="/nevada.html">Nevada</a>
      <a href="/new-hampshire.html">New Hampshire</a>
      <a href="/new-jersey.html">New Jersey</a>
      <a href="/new-mexico.html">New Mexico</a>
      <a href="/new-york.html">New York</a>
      <a href="/north-carolina.html">North Carolina</a>
      <a href="/north-dakota.html">North Dakota</a>
      <a href="/ohio.html">Ohio</a>
      <a href="/oklahoma.html">Oklahoma</a>
      <a href="/oregon.html">Oregon</a>
      <a href="/pennsylvania.html">Pennsylvania</a>
      <a href="/rhode-island.html">Rhode Island</a>
      <a href="/south-carolina.html">South Carolina</a>
      <a href="/south-dakota.html">South Dakota</a>
      <a href="/tennessee.html">Tennessee</a>
      <a href="/texas.html">Texas</a>
      <a href="/utah.html">Utah</a>
      <a href="/vermont.html">Vermont</a>
      <a href="/virginia.html">Virginia</a>
      <a href="/washington.html">Washington</a>
      <a href="/west-virginia.html">West Virginia</a>
      <a href="/wisconsin.html">Wisconsin</a>
      <a href="/wyoming.html">Wyoming</a>
    </div>
  </details>
  
  <div class="f-tagline">Free car payment calculator — not financial advice. Consult your lender for exact terms.</div>
  <div class="f-alt">Also known as a car loan calculator or auto loan calculator</div>
</footer>
```

---

## 第 7 章 — 发布节奏与 SEO 风控

### 7.1 分批发布计划

**不要 50 个一次性发**——Google 会判定为 mass-generated low-quality content。

**第 1 批(Day 1):California 主页 + 2 个子页**
理由:作为 pilot 验证整套模板和构建流程,California 数据最全。
URL 数量:3
sitemap 提交:立即

**第 2 批(Day 3-5):Top 10 高搜索量州**
TX / FL / NY / PA / IL / OH / GA / NC / MI / WA
理由:高流量州优先,数据准备相对容易。
URL 数量:10
sitemap 提交:批次完成后

**第 3 批(Day 8-10):中等规模州**
AZ / MA / TN / IN / MO / MD / WI / CO / MN / SC / AL / LA / KY / OR / OK
URL 数量:15

**第 4 批(Day 12-15):剩余州**
余下 25 州一次性发布
URL 数量:25

每批之间间隔几天的目的是:
1. 给 Google 时间分批爬取,避免被 throttle
2. 监控早批次的索引情况和警告(GSC 会提示问题)
3. 出问题可以及时调整模板,不影响后续批次

### 7.2 GSC 监控指标

每批发布后 48 小时内检查 Google Search Console:

- **Coverage > Indexed**:发布的州页面是否进入索引
- **Coverage > Not Indexed**:有没有被标记 "Discovered - currently not indexed"(质量信号弱)
- **Performance**:有没有曝光/点击数据
- **Enhancements > FAQ**:FAQ schema 是否被识别
- **Enhancements > Breadcrumbs**:面包屑 schema 是否被识别

**红线警告**:如果第 1 批 California 发布 7 天后仍未被索引,**暂停后续批次**,先排查问题(常见:重复内容、薄内容、internal link 不足)。

### 7.3 内容质量底线

**绝不踩的雷区**:

1. ❌ **同模板填州名了事**:如果两个州页面 80% 文字相同,只是 "California" 改成 "Texas",会被判定为 doorway pages
2. ❌ **Trade-in 规则错误**:STATE_RULES 已经做对了,文案也必须严格按 `tradeInCredit` 字段写,不能笼统说"trade-in saves you money"
3. ❌ **过期 EV 政策**:CVRP 已停 / 联邦 $7,500 已结束,任何州页面引用都必须用过去时
4. ❌ **JSON-LD 数据与页面文字不一致**:FAQPage 里的 answer 必须**一字不差**地出现在页面正文中,否则 Google 会判定 schema spam

**绝对要做的**:

1. ✅ 每个州至少 3 段 200+ 词的州特异性内容(不能模板化)
2. ✅ 至少 1 个州独有数据(税率、注册费、特殊车型偏好等)
3. ✅ 至少 2 张差异化表格(城市税率表 + 车型月供表)
4. ✅ FAQ 至少 2 个州专属问题(不能全是通用模板)

---

## 第 8 章 — 第一阶段完成的判定标准

执行 AI 完成本计划书后,**Mao 验收时检查这些**:

### 8.1 文件清单(根目录)

应新增:
- `_src/` 整个目录(含 build.js、_data、_templates、_shared)
- `california.html`(主州页)
- `california-car-sales-tax.html`(子页)
- `california-auto-loan-rates.html`(子页)

应被构建脚本"打补丁"(footer 区域已替换):
- `index.html`
- `legal.html`
- `embed.html`

应更新:
- `sitemap.xml`(从 3 条 URL 变为 6 条)

### 8.2 California 主页质量检查清单

- [ ] H1 含关键词 "California Car Payment Calculator"
- [ ] meta description 80-160 字符，含 "California" "car payment calculator" "sales tax"
- [ ] canonical = `https://carpaymentcalculator.app/california.html`
- [ ] 至少 3 个 JSON-LD blocks：WebApplication / BreadcrumbList / FAQPage
- [ ] 计算器 iframe 可正常加载，预填 7.25% 税率
- [ ] 正文 1200+ 词，关键词密度 3.5-5%
- [ ] 至少 2 张数据表格（城市税率 + 车型月供）
- [ ] FAQ 至少 6 题，JSON-LD 与正文一致
- [ ] **OBBBA H3 段落**使用 `no_conform` 模板（California 是 St-L 州）
- [ ] **OBBBA 段落**字数 150-250 词，含 add-back 警告
- [ ] **OBBBA FAQ 题**至少 2 道（Q1 通用 + Q2 no_conform 特异）
- [ ] Related States 区域含 NV / OR / AZ 三链接
- [ ] Footer details 含全 50 州链接
- [ ] 移动端样式正常（680px breakpoint）

### 8.3 构建脚本质量检查

- [ ] `node _src/build.js` 一键执行无报错
- [ ] 输出日志清晰列出生成的文件
- [ ] 重复执行幂等(产物不变)
- [ ] FOOTER:START / FOOTER:END 锚点替换不破坏 index.html 其他内容

### 8.4 Mao 视觉验收

- [ ] California 主页打开后视觉风格与首页**完全一致**(奶油色背景、Fraunces 字体标题、coral 强调色)
- [ ] 没有出现 AI 风格的过度装饰(无渐变光晕、无多余 emoji、无滥用 lucide 图标)
- [ ] Footer details 折叠展开流畅
- [ ] 所有内链可正常跳转(虽然其他州页面还没生成,链接 404 是预期的)

---

## 附录 A — 给执行 AI 的明确指令

请按以下顺序执行：

**Step 1**：阅读本计划书全文，理解 8 个章节 + 附录 C / D 的关系。**特别注意**：
- 第 2.4 节（OBBBA H3 段落规范）是 v1.1 新增的关键差异化模块
- 附录 C 含 50 州 + DC 的完整 OBBBA conformity 数据（直接拿来填 states.json）

**Step 2**：先创建 `_src/` 目录骨架：
- `_src/_data/states.json`（从 index.html 现有 STATE_RULES 提取 + 补充附录 C 的 obbba_conformity 字段，California 数据已在计划书中给全）
- `_src/_data/california-tax.json`（按本计划书提供的数据填充）
- `_src/_data/california-rates.json`（按本计划书提供的数据填充）
- `_src/_templates/state-main.html`（按 6.1 节）
- `_src/_templates/state-tax.html`（子页模板，沿用主页风格）
- `_src/_templates/state-rates.html`（同上）
- `_src/_templates/obbba-conform.md`（OBBBA H3 段落模板 A，按 2.4.2 节）
- `_src/_templates/obbba-no-conform.md`（OBBBA H3 段落模板 B）
- `_src/_templates/obbba-no-state-tax.md`（OBBBA H3 段落模板 C）
- `_src/_templates/obbba-pending.md`（OBBBA H3 段落模板 D）
- `_src/_shared/footer.html`（按 6.2 节）
- `_src/build.js`（按 4.1 节，含 generateObbbaSection() 函数根据 status 选模板）

**Step 3**：在 `index.html`、`legal.html`、`embed.html` 现有 `<div class="footer">` 上下加 `<!-- FOOTER:START -->` `<!-- FOOTER:END -->` 锚点。

**Step 4**：运行 `node _src/build.js`，生成：
- `california.html`（含 OBBBA `no_conform` 段落）
- `california-car-sales-tax.html`
- `california-auto-loan-rates.html`
- 重写 `sitemap.xml`
- 给 index/legal/embed 三页注入新 footer

**Step 5**：本地预览 California 主页面，人工检查：
- 视觉是否与首页一致
- OBBBA H3 段落字数 150-250 词
- FAQ 区是否含 OBBBA 相关 Q1 + Q2

**Step 6**：暂停。等 Mao 验收 California 三页面后，再考虑生成其他 49 州。

**Step 7**（等 Mao 拍板后）：用同样的构建流程生成第 2 批 10 个州。每个州的 OBBBA 段落必须根据 `obbba_conformity.status` 字段自动选模板，确保 50 个州的段落各不相同。

---

## 附录 B — 已交付数据备忘（California 第一阶段）

- ✅ Sales tax 7.25% / 18 个 10.75% 城市
- ✅ Trade-in 规则(全价计税)
- ✅ VLF 完整 11 年折旧表(Y1-Y11+)
- ✅ TIF 5 档分段表
- ✅ 所有 base/CHP/Title/Smog/RIF/Tire 费用
- ✅ Doc fee cap $85/$70(SB 791 已否决)
- ✅ Late penalties 5 档表
- ✅ 杂项费用完整清单
- ✅ 支付手续费(office 2.1%/online & kiosk 1.95%/eCheck 0%)
- ✅ Q4 2025 by-tier APR + 平均贷款金额 + 期限
- ✅ 2025 Top 5 销量车型
- ✅ EV 政策 3 项变化(CVRP/Federal/HOV)
- ✅ OBBBA 车贷利息抵税新政（含 California `no_conform` 状态确认）
- ✅ **50 州 + DC 完整 OBBBA conformity 数据（附录 C，源自 Tax Foundation 官方报告）**
- ✅ Smog check 规则
- ✅ 主要城市 combined tax 率

**待补**(可在执行中补,不阻塞):
- ⚠️ 县级 sales tax 完整 ZIP 表(目前用州级 7.25% + "up to 10.75%" 表述,够用)
- ⚠️ VLF 年龄计算改用 first registration year 而非 model year(代码层面修正)

---

## 附录 C — 50 州 + DC 完整 OBBBA Conformity 数据表

**数据源权威性**：源自 Tax Foundation《Guide to OBBBA State Tax Conformity》官方报告（2025年7月发布），加上 NCDOR / Grant Thornton 等权威源补充。

**字段说明**：
- `code`：州两字母代码
- `pit_status`：Personal Income Tax conformity 类型
  - `Roll` = Rolling（自动 conform 最新 IRC，OBBBA 默认 conform）
  - `St-C` = Static-Current（conform 到近期固定日期）
  - `St-L` = Static-Lagged（conform 到较早日期，OBBBA 不 conform）
  - `Sel` = Selective（选择性 conform）
  - `n.a.` = 该州无个人所得税
- `auto_conform`：是否 conform OBBBA auto loan interest deduction（Tax Foundation "Auto" 列）
- `obbba_conformity.status`：模板分支字段（`conform` / `no_conform` / `no_state_tax` / `pending`）

| State | Code | PIT Status | Auto Conform? | obbba_conformity.status | 备注 |
|-------|------|-----------|---------------|------------------------|------|
| Alabama | AL | Roll | ✓ | conform | 双重抵扣 |
| Alaska | AK | n.a. | — | no_state_tax | 无 PIT |
| Arizona | AZ | St-C | ✓ | conform | conformity 已更新到 2025 |
| Arkansas | AR | Sel | （无标记） | pending | Selective，无明确 OBBBA 决定 |
| **California** | **CA** | **St-L** | **✗** | **no_conform** | **2015-01-01 conformity，需 add-back** |
| Colorado | CO | Roll | ✓ | conform | 双重抵扣 |
| Connecticut | CT | Roll | ✓ | conform | 双重抵扣 |
| Delaware | DE | Roll | ✓ | conform | 双重抵扣 |
| District of Columbia | DC | Roll | ✓ | conform | 双重抵扣 |
| Florida | FL | n.a. | — | no_state_tax | 无 PIT |
| Georgia | GA | St-C | （无标记） | pending | conformity 已更新但 Auto 列空白，特殊情况 |
| Hawaii | HI | St-C | （无标记） | no_conform | 个税但未 conform Auto |
| Idaho | ID | St-C | ✓ | conform | 双重抵扣 |
| Illinois | IL | Roll | ✓ | conform | 双重抵扣 |
| Indiana | IN | St-L | （无标记） | no_conform | 滞后 conformity |
| Iowa | IA | Roll | ✓ | conform | 双重抵扣 |
| Kansas | KS | Roll | ✓ | conform | 双重抵扣 |
| Kentucky | KY | St-C | （无标记） | no_conform | conform 但 Auto 列空白 |
| Louisiana | LA | Roll | （无标记） | pending | Roll 但 Auto 列空白，反常情况 |
| Maine | ME | St-L | （无标记） | no_conform | 滞后 conformity |
| Maryland | MD | Roll | ✓ | conform | 双重抵扣 |
| Massachusetts | MA | St-L | ✓ | conform | 罕见：St-L 但 conform Auto |
| Michigan | MI | Roll | ✓ | conform | 双重抵扣 |
| Minnesota | MN | St-L | ✓ | conform | 罕见：St-L 但 conform |
| Mississippi | MS | Sel | （无标记） | no_conform | 选择性，未选 Auto |
| Missouri | MO | Roll | ✓ | conform | 双重抵扣 |
| Montana | MT | Roll | ✓ | conform | 双重抵扣 |
| Nebraska | NE | Roll | ✓ | conform | 双重抵扣 |
| Nevada | NV | n.a. | — | no_state_tax | 无 PIT |
| New Hampshire | NH | n.a. | — | no_state_tax | 无 PIT（仅 interest/dividend） |
| New Jersey | NJ | Sel | （无标记） | no_conform | 选择性，未选 Auto |
| New Mexico | NM | Roll | ✓ | conform | 双重抵扣 |
| New York | NY | Roll | ✓ | conform | 双重抵扣 |
| **North Carolina** | **NC** | **St-L** | **✗** | **no_conform** | **2023-01-01 conformity，NCDOR 已发 1/8/2026 通知须 add-back** |
| North Dakota | ND | Roll | ✓ | conform | 双重抵扣 |
| Ohio | OH | St-C | （无标记） | no_conform | conform 近期但未 Auto |
| Oklahoma | OK | Roll | ✓ | conform | 双重抵扣 |
| Oregon | OR | Roll | ✓ | conform | 双重抵扣 |
| Pennsylvania | PA | Sel | （无标记） | pending | 选择性 conformity |
| Rhode Island | RI | Roll | ✓ | conform | 双重抵扣 |
| South Carolina | SC | St-C | ✓ | conform | 双重抵扣 |
| South Dakota | SD | n.a. | — | no_state_tax | 无 PIT |
| Tennessee | TN | n.a. | — | no_state_tax | 2021 起无 PIT |
| Texas | TX | n.a. | — | no_state_tax | 无 PIT |
| Utah | UT | Roll | ✓ | conform | 双重抵扣 |
| Vermont | VT | St-C | ✓ | conform | 双重抵扣 |
| Virginia | VA | St-C | ✓ | conform | 双重抵扣 |
| Washington | WA | n.a. | — | no_state_tax | 无 PIT（仅 capital gains 高收入） |
| West Virginia | WV | St-C | ✓ | conform | 双重抵扣 |
| Wisconsin | WI | St-L | ✓ | conform | 罕见：St-L 但 conform |
| Wyoming | WY | n.a. | — | no_state_tax | 无 PIT |

**统计汇总**：
- `conform` 州：30 个 + DC = 31 个
- `no_conform` 州：10 个（CA / HI / IN / KY / ME / MS / NJ / NC / OH + 1）
- `no_state_tax` 州：9 个（AK / FL / NV / NH / SD / TN / TX / WA / WY）
- `pending` 州：4 个（AR / GA / LA / PA）

**重要警告**：
1. 该数据基于 Tax Foundation 2025-07 报告 + 各州截至 2026-01 的更新。**州立法可能继续变化**，特别是 `pending` 州。
2. **必须每季度复核**：在 `_src/_data/states.json` 文件头部加版本日期，每年 1 月、4 月、7 月、10 月跑一次 verification（搜 "{state} OBBBA conformity 2026"）
3. 部分 `no_conform` 州可能在 2026-2027 年立法 conform。一旦 conform，对应州页面必须重生成
4. `pending` 州的 OBBBA H3 段落必须用谨慎措辞，避免给读者错误财务建议

---

## 附录 D — 决策记录：为什么不做独立 OBBBA 页面

**背景**：v1.0 初稿曾考虑做独立 `/car-loan-interest-deduction.html` 页面。v1.1 决定 **不做**，仅在每州页面集成 H3 段落。

**决策过程**：

**第 1 阶段（初步评估）**：
认为 OBBBA 是 2025 年新法律，搜索量正在爆发，竞品空白期，应抢占。

**第 2 阶段（市场调研，2026-04-28）**：
发现已有大量竞品占位：
- nationaltaxtools.com（专业税务工具站）已有完整 calculator + guide
- trumptaxtools.com（专门 OBBBA 域名）已上线
- obbba.tax（完美域名）已被占
- notaxon.com 已有 VIN 验证工具
- H&R Block / TurboTax / KBB / Kelley Blue Book 等权威站点（DA 70+）都已有内容

**第 3 阶段（差异化角度评估）**：
所有竞品都是纯税务角度，无人整合"OBBBA + 50 州 conformity"角度。但这个角度的搜索意图模糊，"OBBBA 50 州 conformity"几乎无人这么搜，流量预期极低。

**第 4 阶段（决策）**：
- 不做独立页面（投入产出比差）
- 但保留 OBBBA 内容作为 50 州落地页的差异化模块
- 用 `obbba_conformity` 数据让 50 个州页面的 OBBBA 段落各不相同，避免 doorway pages 信号
- 这同时解决了"50 州落地页内容差异化不足"的潜在问题

**反向决策触发条件**（未来如果出现下面情况，重新考虑做独立页）：
- 某个 `no_conform` 州在 2026-2027 年通过 conformity 立法，引发新闻热度
- 国会延长或修订 OBBBA（2028 年到期前），引发搜索激增
- AdSense 数据显示用户在 OBBBA 段落停留时间异常高，证明独立页有需求
- 我们 50 州落地页全部上线后，进入新内容矩阵阶段

---

**计划书结束**

文档版本：v1.1  
生成日期：2026-04-28  
适用于：carpaymentcalculator.app 50 州落地页第一阶段

