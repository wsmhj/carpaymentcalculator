# Car Payment Calculator

A free, privacy-respecting, browser-based [car payment calculator](https://carpaymentcalculator.app/) for U.S. auto loans. Real-time monthly payment estimates with credit-tier APR defaults, accurate state-level vehicle tax handling (including trade-in credit rules and the South Carolina IMF cap), full amortization schedules, and a side-by-side term comparison from 24 to 84 months.

**Live site:** [carpaymentcalculator.app](https://carpaymentcalculator.app/)
**Embed widget:** [carpaymentcalculator.app/embed.html](https://carpaymentcalculator.app/embed.html)
**Trust &amp; transparency:** [carpaymentcalculator.app/legal.html](https://carpaymentcalculator.app/legal.html)

This project is also commonly searched as a [car loan calculator](https://carpaymentcalculator.app/) or [auto loan calculator](https://carpaymentcalculator.app/) — the underlying math is identical; the names are interchangeable.

---

## Table of Contents

- [Why this calculator](#why-this-calculator)
- [Features](#features)
- [Calculator modes](#calculator-modes)
- [State tax coverage](#state-tax-coverage)
- [Embed widget](#embed-widget)
- [Architecture](#architecture)
- [Tech stack](#tech-stack)
- [Project structure](#project-structure)
- [Local development](#local-development)
- [Calculation methodology](#calculation-methodology)
- [Privacy](#privacy)
- [SEO setup](#seo-setup)
- [Browser support](#browser-support)
- [Accessibility](#accessibility)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Changelog highlights](#changelog-highlights)
- [Acknowledgments](#acknowledgments)
- [License](#license)
- [Contact](#contact)

---

## Why this calculator

Most online auto loan tools are built for ad revenue first and accuracy second. They typically:

- Hard-code a single "national average" APR regardless of credit profile
- Apply state sales tax to the full purchase price even in states that allow trade-in credit
- Miss state-specific rules entirely (Illinois's $10,000 trade-in cap, South Carolina's $500 Infrastructure Maintenance Fee, the no-credit handling in California / Hawaii / Maryland / Virginia / DC / Puerto Rico)
- Ship with cookies, third-party trackers, and aggressive lead-capture forms

The [Car Payment Calculator](https://carpaymentcalculator.app/) takes the opposite stance: zero tracking, zero ads, zero signup, fully client-side, and a tax engine that actually models how each state computes vehicle sales tax. See the [methodology section](https://carpaymentcalculator.app/legal.html#methodology) for full data sources and known limitations.

---

## Features

- **Real-time calculation** — every input update instantly recomputes the monthly payment, no submit button. Implemented via a single reactive `update()` function that owns all DOM writes.
- **Credit-tier APR defaults** — toggle between Excellent / Good / Fair / Poor to auto-fill APR; values differ for new vs. used vehicles per [national auto-loan averages](https://carpaymentcalculator.app/legal.html#methodology).
- **ZIP-based state tax routing** — enter a U.S. ZIP and the calculator looks up the corresponding state's vehicle sales tax rule. 3-digit ZIP overrides handle five boundary states (Rhode Island, Delaware, Mississippi, Hawaii, Alaska) that were previously misrouted by simpler 2-digit lookups.
- **Trade-in credit handling per state** — the calculator correctly applies trade-in credit in 45 states; six states (CA / HI / MD / VA / DC / PR) tax on the full purchase price; Illinois caps the credited trade-in at $10,000; South Carolina charges a flat IMF capped at $500 instead.
- **Reverse mode** — switch to "I know my budget" and the [auto loan calculator](https://carpaymentcalculator.app/) solves the amortization equation backward to return the maximum vehicle price you can afford.
- **Term comparison table** — 24 / 36 / 48 / 60 / 72 / 84 month options computed side by side, click any row to switch the active term.
- **Amortization schedule** — stacked-area SVG chart plus annual/monthly tabular breakdown of principal, interest, and remaining balance.
- **Sticky payment summary** — monthly payment stays visible in the top bar as you scroll the page.
- **Long-term loan warnings** — the calculator surfaces a soft warning at 72 months and a stronger one at 84 months ("often leaves you owing more than the car is worth"), nudging users toward responsible terms.
- **Low-down-payment hint** — when the entered down payment is under 10% of the vehicle price, a non-blocking hint suggests the lender-recommended 10–20% range.
- **APR data freshness** — APR defaults are stamped with their last-reviewed month so users can see how recent the data is.
- **Free embeddable widget** — see [Embed widget](#embed-widget) below.
- **Fully responsive** — desktop, tablet, and mobile layouts down to 280px wide.
- **Accessible** — ARIA labels, keyboard navigation across all controls, screen-reader live regions for result updates.

---

## Calculator modes

| Mode | What you enter | What you get |
|------|---------------|--------------|
| **I know the price** (Forward mode, default) | Vehicle price, down payment, trade-in, credit tier or APR, loan term, ZIP / tax rate | Monthly payment, total interest, full amortization |
| **I know my budget** (Reverse mode) | Monthly budget, down payment, trade-in, credit tier or APR, loan term, ZIP / tax rate | Maximum affordable vehicle price |

Reverse mode is uncommon among free [car loan calculators](https://carpaymentcalculator.app/) — most tools only run forward. The math uses a closed-form solution to the standard amortization equation; see [methodology](https://carpaymentcalculator.app/legal.html#methodology).

---

## State tax coverage

The ZIP-to-state tax router is a two-tier lookup defined in `index.html` as `STATE_RULES`, `ZIP2_TO_STATE`, and `ZIP3_OVERRIDES`. Coverage:

- All 50 U.S. states
- District of Columbia
- Puerto Rico
- 5 boundary-ZIP overrides for Rhode Island, Delaware, Mississippi, Hawaii, and Alaska (which would otherwise be misrouted by 2-digit prefix lookup)

Each state rule encodes:

- Base tax rate (state-level only, not county / city)
- Trade-in credit policy: full / none / partial-with-cap
- Optional flat tax cap (only South Carolina's $500 IMF currently)

For users in major metros where county and city taxes add 0.5–4% on top of the state rate, the calculator displays a static hint pointing them to override the rate manually with their actual local figure. Examples surfaced in the UI: NYC ≈ +4.9%, Chicago ≈ +4%, Los Angeles ≈ +2.5%, Seattle ≈ +3.6%.

Full coverage and known limitations are documented on the [methodology page](https://carpaymentcalculator.app/legal.html#methodology).

---

## Embed widget

A standalone, minimal embeddable build of the calculator lives at [carpaymentcalculator.app/widget.html](https://carpaymentcalculator.app/widget.html). It's designed for credit unions, schools, financial-aid offices, personal-finance bloggers, and auto dealers who want to add a [free car payment calculator](https://carpaymentcalculator.app/embed.html) to their own site.

### Customization parameters

The widget accepts three optional URL query parameters:

| Parameter | Example | Effect |
|-----------|---------|--------|
| `accent` | `?accent=%23004488` | Replace the default coral accent with any hex color (URL-encoded `#`); strict regex validation prevents CSS injection |
| `defaultApr` | `?defaultApr=4.99` | Pre-fill the APR field with your lender's actual rate |
| `taxRate` | `?taxRate=8.875` | Pre-fill the sales tax rate (e.g., for a region-specific embed) |

### Quick embed

```html
<iframe src="https://carpaymentcalculator.app/widget.html?accent=%230066CC&defaultApr=5.99"
  width="100%" height="620"
  style="border:none; max-width:480px; border-radius:16px;"
  title="Car Payment Calculator"
  loading="lazy"></iframe>
<p style="font-size:12px; color:#777; text-align:center; margin-top:6px;">
  Calculator by <a href="https://carpaymentcalculator.app/" target="_blank" rel="noopener">carpaymentcalculator.app</a>
</p>
```

The interactive [embed builder](https://carpaymentcalculator.app/embed.html) provides a live preview, color picker, and copy-paste snippet generator.

### Backlink mechanics

The attribution `<a>` tag is intentionally placed **outside** the iframe in the snippet because Google does not consistently count in-iframe links as backlinks to the iframe source. Embedders can keep both the inside-widget badge and the outside-iframe attribution, or strip just the inside one — the outside link is what passes link equity to the main site.

`widget.html` itself is served with `<meta name="robots" content="noindex, follow">` plus a canonical link back to the main domain, so the widget never competes with the main page for search rankings.

---

## Architecture

The entire user-facing application is a small constellation of static HTML files with inline CSS and JavaScript. There is no build step, no bundler, no JavaScript framework, and no server. The trade-off:

| Pros | Cons |
|------|------|
| Zero deploy complexity (drop files on any static host) | Some CSS is duplicated across pages |
| Immediate page paint, no JS framework cost | No automatic code splitting |
| Easy to fork and host elsewhere | Larger single files for the main app |
| First-paint perceived latency under 200ms on broadband | Reading the source requires scrolling |

For a tool whose entire value lives in fast first interaction, this trade-off is correct. See the [tech stack](#tech-stack) section for specifics.

---

## Tech stack

- **HTML / CSS / vanilla JavaScript** — no frameworks (no React, Vue, Svelte, etc.)
- **No build pipeline** — no Webpack, no Vite, no PostCSS, no TypeScript transpilation
- **Inline assets** — all CSS in `<style>`, all JS in `<script>`, all SVG inline
- **Fonts** — [Fraunces](https://fonts.google.com/specimen/Fraunces) (display / numerals) and [Outfit](https://fonts.google.com/specimen/Outfit) (body) loaded from Google Fonts CDN with `display=swap`
- **No tracking, no analytics, no cookies** — see the [privacy section](https://carpaymentcalculator.app/legal.html#privacy)
- **No third-party JS** — only Google Fonts CSS is fetched cross-origin

The single external dependency is Google Fonts. Self-hosting the WOFF2 files is on the roadmap but not yet implemented.

---

## Project structure

```
carpaymentcalculator/
├── index.html              # Main calculator (HTML + CSS + JS, all inline)
├── widget.html             # Minimal embeddable build (lite UI, 3 query params)
├── embed.html              # Marketing landing page for the widget
├── legal.html              # About + Methodology + Privacy + Disclaimer + Contact
├── favicon.svg             # Hand-drawn SVG favicon (car + dollar circle)
├── og-image.png            # Open Graph share image (1200 × 630)
├── robots.txt              # Crawler rules
├── sitemap.xml             # Lists /, /embed.html, /legal.html (widget.html intentionally excluded)
└── README.md
```

---

## Local development

No build step required. Either:

```bash
# Option 1: open the file directly in a browser
open index.html      # macOS
start index.html     # Windows
xdg-open index.html  # Linux

# Option 2: serve over a local HTTP server (recommended — proper MIME types, no file:// CORS quirks)
npx serve .
# or
python3 -m http.server 8000
```

The [ZIP code tax lookup](#state-tax-coverage) runs entirely offline; all state-level tax rules are bundled inside `index.html` as JavaScript constants. No network requests are made when the calculator is in use beyond the initial page load and the Google Fonts CSS request.

---

## Calculation methodology

Loan payments use the standard amortization formula:

```
M = P × r(1+r)ⁿ / ((1+r)ⁿ − 1)
```

where **M** is the monthly payment, **P** is the financed principal (purchase price minus down payment minus trade-in plus applicable tax), **r** is the monthly interest rate (APR / 12 / 100), and **n** is the number of monthly payments.

Reverse mode solves the same equation for **P** given a target **M**, with a closed-form derivation that accounts for trade-in credit rules and any applicable tax cap. The full derivation, data sources, and known limitations are documented on the [methodology page](https://carpaymentcalculator.app/legal.html#methodology).

APR defaults by credit tier are sourced from [Bankrate's auto loan rate index](https://www.bankrate.com/loans/auto-loans/current-auto-loan-interest-rates/) and [Experian's *State of the Automotive Finance Market*](https://www.experian.com/automotive/auto-finance-trends) report, reviewed quarterly.

---

## Privacy

This [free car payment calculator](https://carpaymentcalculator.app/) is fully client-side. Inputs never leave the browser; there is no backend server.

- **No cookies** are set
- **No `localStorage` or `sessionStorage`** is used
- **No analytics SDK** is loaded
- **No personal data** is collected: no email, no name, no IP logging, no profiling
- **One third-party request**: Google Fonts CSS for typography (per [Google's privacy policy](https://policies.google.com/privacy))

Full privacy stance is published at [carpaymentcalculator.app/legal.html#privacy](https://carpaymentcalculator.app/legal.html#privacy).

---

## SEO setup

The main page and the [embed marketing page](https://carpaymentcalculator.app/embed.html) ship with comprehensive on-page SEO:

- [Canonical URL](https://carpaymentcalculator.app/) and `hreflang` tags
- [Open Graph](https://ogp.me/) + Twitter Card meta tags
- Four [JSON-LD](https://json-ld.org/) structured data blocks: [WebSite](https://schema.org/WebSite), [WebApplication](https://schema.org/WebApplication), [Organization](https://schema.org/Organization) (with `alternateName` for "Car Loan Calculator" and "Auto Loan Calculator"), and [FAQPage](https://schema.org/FAQPage)
- Semantic HTML with keyword-rich content sections
- [XML sitemap](https://carpaymentcalculator.app/sitemap.xml) and [robots.txt](https://carpaymentcalculator.app/robots.txt)
- `noindex, follow` on `widget.html` to prevent the embeddable build from competing with the main page in search results

---

## Browser support

Tested against current evergreen versions of Chrome, Firefox, Safari, and Edge. The calculator uses standard browser APIs only:

- ES2015+ JavaScript (no transpilation needed for any browser from 2017 onward)
- CSS Custom Properties (variables)
- `IntersectionObserver` for the sticky-bar transition and table-of-contents scroll-spy
- `URLSearchParams` for query parameter parsing in the widget
- `navigator.clipboard` for the copy-to-clipboard button (with fallback to `Range` selection)

Internet Explorer is not supported.

---

## Accessibility

- Every interactive control has an `aria-label` or wrapping `<label for="...">`
- Button groups (credit tier, loan term, vehicle condition) have `role="group"` and `aria-pressed` state
- Result updates announce to screen readers via a debounced `aria-live="polite"` region
- Keyboard navigation works across all inputs, sliders, and button groups (arrow keys on sliders adjust by $250)
- Tap targets meet the 44×44 px minimum recommended by WCAG 2.5.5
- Color contrast checked against WCAG AA for text and interactive elements

---

## Roadmap

Tracked informally; pull requests welcome. In rough priority order:

- [ ] Self-host Fraunces and Outfit WOFF2 to remove the only third-party dependency
- [ ] PWA support: `manifest.json` + minimal service worker for offline use
- [ ] Spanish (`es`) language toggle for accessibility to a larger U.S. demographic
- [ ] Per-state landing pages targeting long-tail queries like "California car loan calculator" — gated on validating SEO viability with the first 5 states
- [ ] Refinance calculator (separate page sharing the same engine)
- [ ] Lease vs. buy calculator
- [ ] PDF export of the amortization schedule

---

## Contributing

Bug reports and tax-rate corrections are very welcome. To contribute:

1. **Math errors**: open an issue with the inputs you used and the expected vs. actual output. Math fixes are prioritized over feature requests.
2. **Tax rate corrections**: please cite the state DMV or department of revenue page that documents the corrected rate or rule.
3. **Feature suggestions**: open an issue first to discuss scope before submitting a PR.
4. **Code style**: match the existing single-file style. No build pipeline, no framework imports, no TypeScript.

Sensitive matters (security disclosures, partnership inquiries) — please email rather than file a public issue.

---

## Changelog highlights

- **April 2026** — Tax engine rewrite: introduced per-state `STATE_RULES`, trade-in credit per state (CA/HI/MD/VA/DC/PR no-credit; IL partial cap; SC IMF), boundary-ZIP overrides for RI/DE/MS/HI/AK. Added the [embeddable widget](https://carpaymentcalculator.app/embed.html) and the [Trust &amp; Transparency page](https://carpaymentcalculator.app/legal.html). Long-term loan warnings (72/84 mo) and low-down-payment hints. Manual tax rate now inherits state-level cap and credit rules from the active ZIP. Email moved to a domain address.
- **Earlier 2026** — Initial release: forward + reverse modes, credit-tier APR defaults, term comparison table, amortization schedule, ZIP-based state tax routing.

For a complete history, see the git log.

---

## Acknowledgments

- [Bankrate Auto Loan Average Rate](https://www.bankrate.com/loans/auto-loans/current-auto-loan-interest-rates/) and [Experian *State of the Automotive Finance Market*](https://www.experian.com/automotive/auto-finance-trends) — APR data
- [Tax Foundation](https://taxfoundation.org/) — sales tax cross-references
- Each state's department of revenue / DMV — primary tax rate source
- [Fraunces](https://fonts.google.com/specimen/Fraunces) and [Outfit](https://fonts.google.com/specimen/Outfit) on Google Fonts — typography

---

## License

All rights reserved. The source is published openly for transparency and learning, but the [Car Payment Calculator](https://carpaymentcalculator.app/) name, branding, and hosted instance at [carpaymentcalculator.app](https://carpaymentcalculator.app/) are not licensed for republication or rehosting under the same identity.

If you want to embed the calculator on your own site, the [free embeddable widget](https://carpaymentcalculator.app/embed.html) is the supported and recommended path; it auto-updates with bug fixes and tax-table revisions. Please keep the small attribution link visible in the embed snippet.

For commercial licensing, white-label, or self-hosted enterprise builds, see [Contact](#contact).

---

## Contact

- **General**: [hello@carpaymentcalculator.app](mailto:hello@carpaymentcalculator.app)
- **Bug reports / tax corrections**: same address; aim to respond within 5 business days, math errors typically within 48 hours
- **Embed-widget partnership** (credit unions, schools, financial-aid offices, nonprofits): same address — see the [Trust &amp; Transparency page](https://carpaymentcalculator.app/legal.html#contact) for what we welcome and what we cannot help with
- **Live tool**: [carpaymentcalculator.app](https://carpaymentcalculator.app/)

---

*Built and maintained by an independent developer. No corporate affiliation, no lender partnerships, no sponsored content as of April 2026. If that ever changes, the [About section](https://carpaymentcalculator.app/legal.html#about) will be updated first.*
