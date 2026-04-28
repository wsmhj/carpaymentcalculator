# Car Payment Calculator

Free online car payment calculator with real-time results, credit score-based rates, amortization schedule, and term comparison.

**Live Site:** [carpaymentcalculator.app](https://carpaymentcalculator.app/)

## Table of Contents

- [Features](#features)
- [Calculator Modes](#calculator-modes)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [SEO](#seo)
- [Local Development](#local-development)
- [License](#license)

## Features

- **Real-time calculation** — results update instantly as you type, no submit button needed
- **Credit score tiers** — toggle between Excellent / Good / Fair / Poor to auto-fill APR for [new and used cars](#calculator-modes)
- **ZIP code tax lookup** — enter a U.S. ZIP code to auto-fill state sales tax rate
- **Term comparison table** — side-by-side view of 24 / 36 / 48 / 60 / 72 / 84 month options
- **Amortization schedule** — stacked area chart + annual/monthly breakdown table
- **Sticky summary bar** — monthly payment stays visible as you scroll
- **Fully responsive** — works on mobile, tablet, and desktop
- **Accessible** — ARIA labels, keyboard navigation, screen reader announcements

## Calculator Modes

| Mode | Description |
|------|-------------|
| **I know the price** (Forward) | Enter vehicle price → get monthly payment |
| **I know my budget** (Reverse) | Enter monthly budget → get max affordable vehicle price |

## Tech Stack

- **Pure HTML / CSS / JavaScript** — single-file architecture, no frameworks or build tools
- **Fonts** — [Fraunces](https://fonts.google.com/specimen/Fraunces) (headings) + [Outfit](https://fonts.google.com/specimen/Outfit) (body) via Google Fonts
- **SVG** — hand-drawn car favicon and decorative background pattern, all inline

## Project Structure

```
carpaymentcalculator/
├── index.html        # Full application (HTML + CSS + JS)
├── favicon.svg       # SVG favicon (car icon)
├── og-image.png      # Open Graph share image (1200×630)
├── robots.txt        # Crawler rules
├── sitemap.xml       # Sitemap for search engines
└── README.md
```

## SEO

The page includes comprehensive SEO setup:

- [Canonical URL](https://carpaymentcalculator.app/) and hreflang tags
- [Open Graph](https://ogp.me/) and Twitter Card meta tags
- JSON-LD structured data — [WebSite](https://schema.org/WebSite), [WebApplication](https://schema.org/WebApplication), [FAQPage](https://schema.org/FAQPage)
- Semantic HTML with keyword-rich content sections
- [Sitemap](https://carpaymentcalculator.app/sitemap.xml) and [robots.txt](https://carpaymentcalculator.app/robots.txt)

## Local Development

No build step required. Open `index.html` in a browser:

```bash
# Option 1: direct open
open index.html

# Option 2: local server (for proper MIME types)
npx serve .
```

ZIP code tax lookup runs fully offline — all state-level vehicle tax rules are bundled inside `index.html`. No network requests are made.

## License

All rights reserved.
