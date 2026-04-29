# Project Context — carpaymentcalculator.app

> **For future AI sessions**: read this file first. It encodes architectural decisions, conventions, common pitfalls, and what was already tried so you don't re-litigate them.

**Last updated**: 2026-04-29

---

## 1. What this site is

A **free, privacy-respecting, browser-based U.S. auto-loan calculator**. Real-time monthly payment estimates with credit-tier APR, state-aware vehicle tax, full amortization, and a side-by-side term comparison. The user-facing brand: hand-drawn style (Fraunces + Outfit fonts, cream/coral/forest palette).

**Live**: https://carpaymentcalculator.app/
**GitHub**: https://github.com/wsmhj/carpaymentcalculator
**Owner email**: hello@carpaymentcalculator.app
**Hosting**: Cloudflare Pages (auto-deploy on git push to `main`)

**Owner profile**: solo developer in China. Native Chinese speaker. Strong product instinct; not a deep SEO/legal expert. Verifies data manually via Google. Cannot access `.ca.gov` / `.gov` sites without a VPN (Akamai blocks Chinese IPs). Asks for honest pushback, not yes-man answers.

---

## 2. Tech stack & architecture (load-bearing decisions)

- **Pure HTML / inline CSS / inline vanilla JS**. No frameworks. No build step. No npm. No TypeScript.
- **Deployment**: `git push` to GitHub → Cloudflare Pages auto-deploys static files.
- **Fonts**: Google Fonts (Fraunces for headings/numerals, Outfit for body). Only third-party dependency.
- **No analytics, no cookies, no tracking, no ads** (as of 2026-04). Privacy is a deliberate selling point.
- **CSS variables**: `--cream`, `--warm`, `--coral` (#E8725A primary), `--forest` (#2D5A3D secondary), etc. Defined in `:root` of every page.

**Don't suggest**: React/Next/Vue rewrites, build tooling, CSS extraction, npm dependencies — the user has explicitly chosen single-file static for deploy simplicity. Pushing back on that is wasted effort.

### Responsive breakpoints (load-bearing — copy these exactly into new state pages)

Three breakpoints used consistently across all pages:
- **`900px`** — calculator container switches from 2-column grid (form left + result-box right) to single column. Sticky right column also disabled. Footer 50-state grid auto-flows.
- **`680px`** — mobile typography scale-down: hero h1 → 28px, article h2 → 24px, panel padding 28→20px, result-box font 48→40px, tables font 14→13px.
- **`480px`** — city quickpicks grid switches from 4 columns to 2 columns. Used inside the `tax-rate-section` block.

Don't invent new breakpoints. Copy these from `california/index.html` as-is.

### Animation classes

All pages use a single fade-up animation:
- `.fi` — applies `animation: fadeUp 0.5s ease forwards`
- `.fi1` / `.fi2` / `.fi3` — adds `animation-delay: 0.08s / 0.18s / 0.28s` and `opacity: 0` (so element is invisible until animated in)

Apply `class="fi fi1"` to first content block, `fi fi2` to second, `fi fi3` to third. New state pages should match this pattern for visual continuity.

---

## 3. URL conventions (CRITICAL — get this wrong and you 301-loop the whole site)

### Cloudflare Pages strips trailing slashes

`https://carpaymentcalculator.app/california/` automatically 301-redirects to `https://carpaymentcalculator.app/california`. Therefore the **canonical form is no trailing slash**.

### State pages

- **File path**: `california/index.html` (directory + index file)
- **Canonical URL**: `https://carpaymentcalculator.app/california` (NO trailing slash, NO `.html`)
- **All references** (canonical link, og:url, JSON-LD url, internal links, sitemap, footer links) must use `/california` form.

When adding a new state, the pattern is:
```
texas/index.html
new-york/index.html
south-carolina/index.html
```
URLs: `/texas`, `/new-york`, `/south-carolina`.

### Other pages

- Index: `/`
- Trust pages: `/legal.html` (kept `.html` for now since it's standalone, not a directory)
- Embed marketing: `/embed.html`
- Embeddable widget: `/widget.html`

### Relative path rule inside `state/index.html` files

Because state files live one level deep, all asset and cross-page references **must be absolute paths**:
- `/favicon.svg` not `favicon.svg`
- `/legal.html#about` not `legal.html#about`
- `/embed.html` not `embed.html`

### Coming-soon / span placeholder pattern (anti-404 strategy)

When a state page doesn't exist yet, **never link to it**. Two specific patterns:

**For "Calculator for Neighboring States" cards** (3 cards on each state page):
```html
<span class="related-card coming-soon" aria-disabled="true">
  <span class="rs-name">Nevada</span>
  <span class="rs-rate">6.85% sales tax · trade-in credit</span>
  <span class="rs-status">Page coming soon</span>
</span>
```
CSS handles opacity + `cursor: not-allowed` + disabled hover.

**For footer 50-state grid** (the "Calculate by State" `<details>` block):
```html
<span>Texas</span>          <!-- not built yet -->
<a href="/california">California</a>  <!-- built -->
```
CSS rule: `.f-states-grid span { opacity: 0.55; cursor: not-allowed; }`

When you finish a new state page, convert that one `<span>` to `<a href="/slug">` in **every page that has the f-states grid** (see Section 6 sync responsibilities).

**Never** link to a state page that doesn't exist. Cloudflare returns 404 → Google penalizes the linking page → entire site authority drops.

---

## 4. SEO conventions

### TDK (Title / Description / Keywords)

| Field | Rule | Why |
|-------|------|-----|
| Title | 30-60 chars | Google truncates >60 |
| Description | 100-160 chars | Google truncates >160 |
| Keywords | **Do not add** (no `<meta name="keywords">`) | Google ignored since 2009; adding it looks dated |
| og:title | Aligned with `<title>` | Brand consistency in social shares |
| og:description / twitter:description | Aligned with `<meta description>` | Same |

### JSON-LD structured data

Each indexable page should have:
- `WebSite` (homepage only)
- `WebApplication` (all calculator pages)
- `Organization` (homepage; included in legal.html)
- `BreadcrumbList` (state pages)
- `FAQPage` (pages with visible FAQ — content **must match** the JSON-LD verbatim)

### External links rel attributes

| Type | rel value |
|------|-----------|
| External authoritative source (DMV, IRS, Bankrate, Tax Foundation, etc.) | `rel="nofollow noopener"` + `target="_blank"` |
| GitHub / third-party platforms | `rel="nofollow noopener"` |
| Self-references to `carpaymentcalculator.app` | `rel="noopener"` only — **never nofollow** (these are juice paths back to us, especially the embed widget attribution) |
| Internal links (relative or `/`-prefixed) | No rel needed |

### widget.html is `noindex, follow`

Don't try to make it rank. It's an iframe payload. Its inside-iframe attribution is for end-user clicks. The juice-passing attribution lives **outside the iframe** in the embed snippet shown on `embed.html`.

---

## 5. Tax engine (the core differentiator)

The state-by-state vehicle tax engine in `index.html` (`STATE_RULES`) handles 51 jurisdictions correctly. Key non-obvious behaviors:

- **No trade-in credit**: CA, HI, MD, VA, DC, PR — tax full purchase price
- **Partial credit (capped)**: IL caps credited trade-in at $10,000
- **Flat tax with cap**: SC charges 5% IMF capped at $500 (no trade-in credit)
- **Boundary ZIP overrides**: 5 ZIP3 ranges that cross state lines were misrouted by simpler 2-digit prefix lookup. Now corrected for RI / DE / MS / HI / AK.
- **Manual rate inheritance**: When user types a manual tax rate AND has a known ZIP, the synthetic rule inherits that state's `tradeInCredit` and `taxCap` semantics. Strips them when ZIP is cleared.

**Math invariants** (must hold under all inputs):
- `Total cost displayed` = `down + Σ monthly_payment + trade_in` (i.e., user's actual out-of-pocket)
- Equivalently: `price + tax + total_interest`
- **Common bug**: forgetting to include tax in displayed total cost. There was a bug like this on `california/index.html` already fixed.

---

## 6. State landing page template

When the owner adds a new state (Texas, Illinois, etc.), follow the **California pattern** at `california/index.html`:

### Structure
1. SEO meta block (title 30-60 chars, description 100-160 chars, OG/Twitter aligned)
2. JSON-LD: WebApplication + BreadcrumbList + FAQPage
3. Breadcrumb nav
4. Hero section with H1 and short paragraph (free + key differentiator)
5. Calculator widget (state's specific defaults)
6. Article body with H2/H3 hierarchy:
   - "How [State] Sales Tax Affects Your Car Payment"
   - "[State] Vehicle Registration Fees Breakdown"
   - "2026 Auto Loan Rates for [State] Buyers"
   - "Car Payment Examples for [State]'s Top-Selling Models"
   - "City-by-City Tax Comparison" (top 20 + searchable all-cities table if applicable)
   - "[State]-Specific Buying Tips" (smog/registration/EV incentives)
   - "OBBBA Auto Loan Interest Deduction in [State]" — H3 inside Buying Tips
   - "[State] Car Payment FAQ" (6-8 Q&A, JSON-LD must mirror exactly)
7. Related States section (3 neighbors as `<span class="related-card coming-soon">` until those pages exist)
8. Footer: f-brand, f-links, **f-states 50-state grid** (49 spans + 1 active link to current state)

### Pacing rule

**One state per day, 5/week, 10 weeks total**. Do NOT batch-publish 50 in 2 weeks (Google flags as mass-generated content). The state grid in footers uses spans for non-existent pages; convert to `<a>` only when the page is built.

### Required data per state (owner must research)

`STATE_RULES` already encodes tax + trade-in rules. For the article content, owner researches per state:
- Top 5 best-selling vehicles + CNCDA-equivalent registration data
- Top 20 cities + combined tax rates from state revenue authority
- State-specific quirks (smog laws, EV incentives, doc fee caps, OBBBA conformity)
- 6-8 unique FAQ questions

**Owner does data research manually via Google.** AI does not fact-check tax law claims (training cutoff + can't access .gov sites). AI's job: assemble, format, do math, ensure consistency.

### Tax UI complexity decision (per state)

California uses a **3-layer tax rate UI**:
- Layer 1: 8 city quickpick buttons (LA / SD / SF / San Jose / Sacto / Long Beach / Oakland / Fresno)
- Layer 2: Searchable dropdown with all 322 incorporated cities
- Layer 3: Manual rate input (always present, syncs with the other two)

**When to use full 3-layer (CA / TX / NY / IL / FL)**: state has >100 cities AND city-level tax variation >2% from state base. This justifies the UI complexity.

**When to use Layer 3 only (most smaller states)**: just manual rate input + a brief "look up your local rate" hint. Quickpicks add UI noise without value when the state has uniform combined tax.

Decision rule: ask "Does this state have at least one city where combined tax is >2% above state base?" If no, simplify to manual.

### f-states grid is duplicated across pages — sync responsibility

The "Calculate by State" footer grid is **physically copied** into:
- `index.html`
- `california/index.html` (and every future `[state]/index.html`)

There is no shared template. When you build a new state page (e.g., `texas/index.html`):

1. **Edit `index.html`**: change `<span>Texas</span>` → `<a href="/texas">Texas</a>` in the grid
2. **Edit existing state pages** (`california/index.html`, etc.): same change in their grids
3. **In the new `texas/index.html`**: it has its own grid where Texas itself is bolded as the active page (`<a href="/texas"><strong>Texas</strong></a>`), and California is a real link (`<a href="/california">California</a>`), and the other 48 are spans

Failing to sync = inconsistent footer = mild SEO penalty + users clicking "California" on Texas page get 404 (because that state's grid wasn't updated).

`legal.html` and `embed.html` deliberately do NOT have the f-states grid (low-priority for SEO juice flow).

### Brand voice & writing style

Write copy that sounds like a thoughtful independent developer talking to a smart buyer — not a corporate marketing department.

- **Direct, slightly informal**: "we give yours" not "we provide accurate calculations". Use "we" and "you", not "users" or third person.
- **Admit limitations openly**: disclaimer says "estimates only", legal page declares "independent developer", privacy page admits Google Fonts is the one third-party dependency. This honesty is a competitive advantage.
- **No corporate language**: never "Welcome to..." / "Our comprehensive..." / "Trusted by thousands..."
- **No fake metrics or social proof**: zero counter-fakery. If we don't have a number, we don't fabricate one.
- **No emoji** unless the owner explicitly asks. The hand-drawn SVGs do the visual warmth instead.
- **Differentiation > generic claims**: instead of "free car payment calculator", say "free California car payment calculator with all 322 city tax rates" — name the specific thing competitors don't have.
- **Numbers when possible**: "8.875% in NYC vs 4% state base" beats "varies by location significantly".

When writing a new state page hero or FAQ, mirror the rhythm of `california/index.html`. Don't drift into corporate-speak.

---

## 7. Calculator math (don't let me drift)

Standard amortization formula (Truth in Lending Act compliant):
```
M = P × r(1+r)ⁿ / ((1+r)ⁿ − 1)
```
Where P = principal (price - down - trade + tax), r = monthly rate (APR/12/100), n = months.

Reverse mode solves for P given M (closed-form derivation handles trade-in credit + tax cap correctly — see `reversePrice()` in `index.html`).

**Test invariants** before any change to math:
- Forward mode: `loanAmt = price - down - trade + tax`
- Reverse mode round-trip: `forward(reversePrice(maxLoan, ...)) === maxLoan`
- Total cost displayed = price + tax + total interest

---

## 8. Common pitfalls (already paid the cost; avoid repeating)

1. **Linking to non-existent state pages → 404 + SEO penalty.** Use `<span>` placeholders for unbuilt states; convert to `<a>` only when each page exists.
2. **`/california.html` vs `/california/` vs `/california`** — Cloudflare strips trailing slash. Canonical form is **no slash, no `.html`**.
3. **Form 1098-VLI / IRS Notice 2025-57** — these specific identifiers are unverified by AI. Owner must confirm with IRS official sources before publishing. AI suggestion: use vague form references if unsure ("an IRS-issued reporting form, final number TBD").
4. **VLF only changes with vehicle price**, not loan terms. If users complain it "doesn't change", the answer is correct behavior, not a bug.
5. **Trade-in tax credit ≠ universal**. CA / HI / MD / VA / DC / PR don't allow it. Calculator copy must reflect the active state's rule, not generic "trade-in lowers your tax".
6. **Don't pre-埋 AdSense placeholders** until the site has actual traffic and is ready for review submission. Adding `ADSENSE:HEAD` style anchors prematurely is over-engineering.
7. **CDTFA / DMV links return 403 from China** — owner cannot directly verify them. Test from US IP via Cloudflare WARP / Mullvad / ProtonVPN, or use https://check-host.net.
8. **Fake metrics ("X calculations today")** are dishonest, AdSense / Google detect them, and they hurt long-term trust. Don't propose this even if users seem to want social proof shortcuts.
9. **`<label>` without `for=` attribute** doesn't fire on click. Always pair labels with form controls explicitly.

---

## 9. What's been built (state of project as of 2026-04-29)

### Files
```
carpaymentcalculator/
├── index.html           Main calculator + 50-state footer (1 link to /california, 49 spans)
├── california/
│   └── index.html       First state landing page; full template reference
├── legal.html           About / Methodology / Privacy / Disclaimer / Contact (Trust page)
├── embed.html           Marketing landing page for the embeddable widget
├── widget.html          Lite embeddable iframe build (noindex, 3 query params for accent/APR/tax)
├── sitemap.xml          4 URLs: /, /california, /embed.html, /legal.html
├── robots.txt
├── favicon.svg
├── og-image.png         1200×630 OG image
├── README.md            Public docs with 30+ backlinks
└── CLAUDE.md            (this file)
```

### Strategy docs in repo (do NOT treat as source of truth)
- `seo-50-states-plan-v1.1.md` — historical aspiration; pacing was scaled back from "14 days × 50 states" to "10 weeks × 5/week"
- `adsense-strategy-v1.0.md` — historical plan; AdSense submission deferred until traffic exists. Don't act on the pre-埋 anchors yet.

### Done
- Tax engine refactor (per-state rules, boundary ZIPs, partial credit, IMF cap)
- Manual rate option-A inheritance from active ZIP
- Long-term loan warnings (72/84 months)
- Low down-payment hint (<10%)
- APR data timestamp + sourcing
- legal.html (5 sections: About / Methodology / Privacy / Disclaimer / Contact)
- Organization JSON-LD
- embed.html + widget.html with 3 customization params (accent / defaultApr / taxRate)
- Source on GitHub footer links
- california/index.html with 322 cities, OBBBA section, Top-5 vehicles
- All external links have `rel="nofollow noopener"`
- TDK character compliance
- 49 footer spans (no dead state links)
- URL canonical form: `/california` (no slash, no .html)

### Open / TODO
- Push current state to GitHub (most changes still local)
- Configure Cloudflare Email Routing for `hello@carpaymentcalculator.app`
- Submit sitemap to Google Search Console after push
- Request indexing for `/california` in GSC
- Verify CA DMV / CDTFA links work for US users (need VPN)
- Verify Form 1098-VLI / IRS Notice 2025-57 references against IRS official sources
- Outreach: Reddit wikis, calculator directories, Product Hunt, credit-union outreach with embed widget

### Pilot 5-state plan (decided 2026-04-29)

Build these 5 states first. After all 5 are live + 4-8 weeks of GSC data, decide whether to scale to 50 or pivot strategy.

| Order | State | Why this state |
|-------|-------|----------------|
| 1 | ✅ California | Largest market; no-trade-in-credit story; 322 cities differentiation |
| 2 | Texas | Second-largest market; HAS trade-in credit (deliberate contrast with CA) |
| 3 | Illinois | UNIQUE $10,000 partial trade-in credit cap (Chicago metro relevance) |
| 4 | South Carolina | UNIQUE $500 IMF flat cap — your tax engine's signature feature |
| 5 | New York | Biggest county-vs-state tax discrepancy (NYC 8.875% vs state 4%) |

**Rationale**: each state has a genuinely different story to tell, which prevents the doorway-page trap. If any of these 5 fail to gain GSC impressions in 8 weeks, the strategy needs rethinking before continuing to 50.

**Owner researches data for state N+1 while AI is building state N.** Owner pacing: roughly 1 state per day, 5 per week.

---

## 10. Working norms with the owner

- Owner pushes back hard on yes-man answers and explicitly asks for skepticism. Don't agree without analyzing.
- Owner's primary language is Chinese; respond in Chinese.
- Owner wants honest verification of AI's claims. When you can't verify (e.g., recent tax law specifics), say so explicitly. Don't fabricate citations.
- Owner has tendency to keep building features when the real bottleneck is outreach / traffic. Push back on this when it recurs.
- Before committing destructive operations (file deletion, force push, schema changes), confirm explicitly.
- Don't add documentation files (`*.md`) unless asked. Don't add comments to code unless they explain a non-obvious WHY.
- TDK changes are always low-stakes; URL changes are HIGH-stakes (must update everywhere consistently).
- The owner's phrasing "你看下" / "你觉得呢" = "give me your honest opinion before I commit", not "do it".

---

## 11. Quick decision shortcuts

| User asks | Default answer |
|-----------|----------------|
| "Add another feature?" | Push back if outreach hasn't started; ask if there's a measurable goal |
| "Build all 50 states fast?" | Recommend 5/week pacing; warn against doorway penalty |
| "Add fake counter / fake testimonials?" | Refuse; explain Google detection + integrity cost |
| "Use external API for live data?" | Default no — privacy + cost + outage risk; offline data is the brand |
| "Add AdSense now?" | Defer until 3-6 months of traffic + GSC data |
| "Move URL structure?" | High-risk — must touch canonical/og/JSON-LD/sitemap/all internal links/all references in same commit |
| "Verify a specific external link?" | Try WebFetch; if blocked, advise owner to use VPN or third-party checker (`check-host.net`) |
