---
name: sls-zonwering-page-publish
description: Generate and publish a SEO-friendly, AEO+GEO optimized, mobile-responsive local landing page on slszonwering.nl for a specific city/plaats ("zonwering in {plaats}") following the SLS Zonwering design template. Use when the user says "create page", "publish page", "make a page about", "new landing page", "add page to slszonwering", "maak een pagina", "publish to slszonwering", "zonwering pagina voor {stad}", or any request to build a new location or product page on the SLS Zonwering WordPress site. Handles full pipeline: plaats/keyword → copy → HTML → WordPress MCP publish.
---

# SLS Zonwering Page Publisher

This skill generates and publishes a complete SEO/AEO/GEO-optimized local landing page on **slszonwering.nl** following the established SLS Zonwering design template. One request → one published page.

> **Note on this skill file**: this skill was drafted from what is publicly visible about slszonwering.nl (site content, product range, existing local-landing-page URLs found via search) — automated fetching of the live page source was blocked by the site's robots.txt, so the exact CSS tokens (hex colors, fonts, spacing) below are a **best-estimate design system**, not confirmed from the actual stylesheet. Before first use, open one real published page (e.g. `/zonwering-in-zwijndrecht/`) in a browser, inspect the CSS, and update the §Design system section with the real values. Everything else (pipeline, section structure, SEO rules) is a reusable template regardless of the exact colors.

## When to invoke

Trigger whenever the user asks for a new page on slszonwering.nl, e.g.:
- "create a page about zonwering in {stad}"
- "publish a landing page for {product}"
- "maak een pagina over zonwering in Dordrecht"
- "add a new city/service page for {plaats}"
- "publish to slszonwering"

Do NOT invoke for: blog posts (use a separate blog-write skill if one exists), editing existing pages (use update_content directly), or non-SLS-Zonwering sites.

## Pipeline (always run in this order)

1. **Clarify inputs** — if user did not specify, ask in ONE `ask_user_question` call:
   - Page type: **city/local page** ("zonwering in {plaats}") or **product page** (e.g. knikarmscherm, screens, markiezen, rolluiken, hybride rolluik, jaloezieën, verticale lamellen, plissegordijnen, terrasoverkapping)
   - Plaats name (if local page) or product name (if product page)
   - Primary keyword (focus_keyword) — default to `zonwering in {plaats}` or `{product} zonwering`
   - Language: `nl-NL` (default; SLS Zonwering serves Dutch-speaking customers)
   - CTA target: default to the contact/offerte form, fallback `mailto:info@slszonwering.nl`
2. **Generate the page** — build full HTML using the Template below (single `render_html` field, CSS inline, scripts at end).
3. **Publish via WordPress MCP** — call the site's WordPress MCP server → `create_content` with the parameters listed in §Publish Parameters.
4. **Verify** — call `get_content` with the returned ID; confirm `render_html` saved and `status=publish`. Report the live URL to the user.

I don't have confirmed access to the specific WordPress MCP server name/connector for this site — verify which MCP connector is available before step 3, and confirm the exact `create_content` field names against that connector's schema rather than assuming they match the example below.

## Template — required structure

Every page MUST follow this section order. Use the **class prefix** `sls-` (or a topic-specific prefix like `slz-` for a specific product) — pick one and stay consistent within the page.

### Section list (in order)

| # | Section | Required | Notes |
|---|---|---|---|
| 1 | `<head>` with SEO meta + JSON-LD | yes | See §Head block |
| 2 | Hero | yes | 2-col grid: copy + image, badge, H1 with plaats/product name, lead, CTA button, trust stats (e.g. jaren ervaring, aantal projecten) |
| 3 | Productenoverzicht (product range) | yes | Grid of the product line: knikarmschermen, screens, markiezen, rolluiken, jaloezieën, verticale lamellen, plissegordijnen, terrasoverkappingen — each a card linking to its product page |
| 4 | Werkwijze (how it works) | yes | 3-4 steps: adviesgesprek/opmeten → offerte → montage → nazorg |
| 5 | Waarom SLS Zonwering (why us) | yes | 4-6 cards: maatwerk, jarenlange ervaring, eigen montage, garantie |
| 6 | Lokale relevantie (local page only) | conditional | 1-2 paragraphs naming the plaats, nearby wijken/streets if known, and why local service matters (measure-on-site, local montage team) |
| 7 | FAQ | yes | 4-6 accordion items, click-to-toggle, chevron rotates — reuse the FAQ content style already used on the live "Hybride Rolluik" page (prijs, maatwerk, elektrische bediening, offerte aanvragen) |
| 8 | Scripts | yes | FAQ accordion toggle + header nav init (see §Scripts) |

### Head block (always include)

```html
<!doctype html>
<html lang="nl-NL" prefix="og: https://ogp.me/ns#">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{Title tag ≤60 chars} | SLS Zonwering</title>
<meta name="description" content="{Meta description 150-160 chars, includes primary keyword}">
<link rel="canonical" href="https://www.slszonwering.nl/{slug}/">
<meta name="robots" content="index, follow, max-snippet:-1, max-image-preview:large, max-video-preview:-1">
<meta property="og:type" content="website">
<meta property="og:url" content="https://www.slszonwering.nl/{slug}/">
<meta property="og:title" content="{title}">
<meta name="twitter:title" content="{title}">
<meta property="og:description" content="{meta description}">
<meta name="twitter:description" content="{meta description}">
<meta name="twitter:card" content="summary">
<script type="application/ld+json">{"@context":"https://schema.org","@type":"LocalBusiness","name":"SLS Zonwering","description":"{meta description}","url":"https://www.slszonwering.nl/{slug}/","email":"info@slszonwering.nl","areaServed":"{plaats}"}</script>
<style>
html, body { height: auto; }
body { overflow-x: hidden; }
.sls-header-container { position: sticky; top: 0; z-index: 99999; }
.sls-header-nav { z-index: 999999; }
.sls-how, .sls-deliver, .sls-faq { scroll-margin-top: 100px; }
</style>
</head>
<body>
```

### Design system (best-estimate — verify against the live site before use)

- **Font**: system-safe sans-serif (e.g. `Inter` or similar) via Google Fonts `@import` inside the first section's `<style>` — confirm the actual font against the live page
- **Icons**: FontAwesome (or the icon set already used on the site) via CDN `@import`
- **Suggested palette** (unverified — sun/shade theme, common for zonwering companies): warm accent (amber/orange, e.g. `#E8871E`–`#F2A93C`) for CTAs and highlights, cool neutral (dark navy/slate, e.g. `#1F2937`) for headings, white/light-grey backgrounds for content sections
- **Text colors**: headings dark neutral, body mid-grey, muted light-grey — verify exact hex against live CSS
- **Border radius**: cards `12-16px`, buttons `8-10px`, pill `9999px`
- **Max widths**: hero wrapper `1200px`, content blocks `800-860px`
- **Section padding**: hero `72px 24px`, others `80-86px 24px`
- **H1**: roughly `48-56px / bold / tight line-height`
- **H2**: roughly `36-42px / bold`

Treat every value in this section as a placeholder until confirmed — do not present it to the user as the site's real brand system without checking.

### Hero section skeleton

```html
<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');
  .sls-hero-container{font-family:'Inter',sans-serif;background:#ffffff;color:#1F2937;padding:72px 24px;overflow:hidden;position:relative}
  .sls-hero-wrapper{max-width:1200px;margin:0 auto;display:grid;grid-template-columns:1.05fr .95fr;align-items:center;gap:52px;position:relative;z-index:2}
  .sls-badge{display:inline-block;padding:7px 12px;background:#fff;color:#E8871E;font-size:14px;font-weight:700;border-radius:9999px;margin-bottom:18px;border:1px solid rgba(232,135,30,.25)}
  .sls-h1{font-size:52px;font-weight:800;line-height:1.1;margin:0 0 18px 0;color:#1F2937;letter-spacing:-0.02em}
  .sls-accent{color:#E8871E}
  .sls-lead{font-size:18px;line-height:1.7;color:#4B5563;margin:0 0 26px 0;max-width:560px}
  .sls-btns{display:flex;flex-wrap:wrap;gap:14px;margin:0 0 44px 0}
  .sls-btn{display:inline-flex;align-items:center;justify-content:center;padding:14px 26px;font-size:16px;font-weight:700;border-radius:10px;text-decoration:none;border:2px solid transparent;transition:all .28s ease;cursor:pointer}
  .sls-primary{background:#E8871E;color:#fff}
  .sls-primary:hover{opacity:.92;box-shadow:0 12px 22px -6px rgba(232,135,30,.28);transform:translateY(-2px)}
  .sls-stats{display:grid;grid-template-columns:1fr 1fr;gap:28px}
  .sls-stat-num{font-size:32px;font-weight:800;color:#1F2937}
  .sls-stat-label{font-size:12px;color:#6B7280;text-transform:uppercase;letter-spacing:.08em}
  .sls-imgwrap{position:relative;display:flex;justify-content:center;align-items:center}
  .sls-imgwrap img{width:100%;max-width:600px;border-radius:16px;box-shadow:0 26px 52px -16px rgba(0,0,0,.2)}
  @media(max-width:768px){.sls-hero-wrapper{grid-template-columns:1fr;text-align:center}.sls-h1{font-size:38px}.sls-lead{margin-left:auto;margin-right:auto}.sls-btns{justify-content:center}.sls-imgwrap{order:1}}
  @media(max-width:480px){.sls-h1{font-size:28px}.sls-hero-container{padding:52px 16px}.sls-stats{grid-template-columns:1fr;gap:18px}}
  @media(max-width:360px){.sls-h1{font-size:24px}.sls-hero-container{padding:44px 14px}}
</style>

<div class="sls-hero-container">
  <div class="sls-hero-wrapper">
    <div class="sls-hero-content">
      <span class="sls-badge">☀️ {badge text, e.g. "Zonwering specialist in {plaats}"}</span>
      <h1 class="sls-h1">Zonwering in <span class="sls-accent">{plaats}</span></h1>
      <p class="sls-lead">{lead paragraph naming the plaats and the product range, with <strong>zonwering {plaats}</strong> as the focus keyword}</p>
      <div class="sls-btns">
        <a class="sls-btn sls-primary" href="{CTA url}">Vraag een gratis offerte aan →</a>
      </div>
      <div class="sls-stats">
        <div><div class="sls-stat-num">{stat1}</div><span class="sls-stat-label">{label1}</span></div>
        <div><div class="sls-stat-num">{stat2}</div><span class="sls-stat-label">{label2}</span></div>
      </div>
    </div>
    <div class="sls-imgwrap">
      <img src="{hero image url}" alt="Zonwering in {plaats} - SLS Zonwering" onerror="this.style.display='none'">
    </div>
  </div>
</div>
```

I don't have real stat numbers (years in business, number of projects, number of reviews) for SLS Zonwering — ask the user for these or omit the stats block rather than inventing figures.

### Scripts (always include, end of body)

```html
<script>
(function(){
  const root=document.querySelector('.sls-faq');
  if(!root) return;
  const items=root.querySelectorAll('.sls-faq-item');
  items.forEach(it=>{
    const btn=it.querySelector('.sls-q');
    if(!btn) return;
    btn.addEventListener('click',()=>{it.classList.toggle('active');});
  });
})();
</script>
<script>
(function(){
  function initHeaderNav(){
    var toggle = document.querySelector('.sls-mobile-toggle');
    var nav = document.querySelector('.sls-header-nav');
    if(!toggle || !nav) return;
    if(toggle.dataset.bound === '1') return;
    toggle.dataset.bound = '1';
    toggle.addEventListener('click', function(e){e.preventDefault(); nav.classList.toggle('active');});
    nav.querySelectorAll('a').forEach(function(a){a.addEventListener('click', function(){nav.classList.remove('active');});});
  }
  if(document.readyState === 'loading'){document.addEventListener('DOMContentLoaded', initHeaderNav);} else {initHeaderNav();}
})();
</script>
</body>
</html>
```

## Mobile responsiveness — 4 breakpoints (MANDATORY)

| Breakpoint | Size | Key adjustments |
|---|---|---|
| Tablet | `≤992px` | Product/step grids → 2 columns |
| Mobile | `≤768px` | All grids → 1 column, H1 ~38px, H2 ~28px, hero text centered, image `order:1` |
| Small Mobile | `≤480px` | H1 ~28px, H2 ~24px, stats → 1 column, tighter padding (52px 16px) |
| Extra Small | `≤360px` | H1 ~24px, H2 ~22px, minimal section padding (44px 14px) |

## SEO / AEO / GEO content rules

1. **Answer-first**: hero lead and FAQ answers open with the direct answer in the first sentence, then context.
2. **Primary keyword** (`zonwering in {plaats}` or `{product} zonwering`) appears in: title tag, H1, meta description, first 100 words, at least one H3, one FAQ answer, and the URL slug — matching the existing pattern seen on live pages such as `/zonwering-in-zwijndrecht/` and `/zonwering-in-gorinchem/`.
3. **Citeerable passages**: 1-2 short, fact-dense sentences AI engines can lift verbatim — only include numbers (prices, years of experience, project counts) that the user has actually confirmed; do not invent them.
4. **Local entity signals**: name the plaats, and — where the user confirms them — nearby service areas, so the page reads as genuinely local rather than templated.
5. **Product entities**: name the real SLS Zonwering product range explicitly where relevant: knikarmschermen, screens, markiezen, rolluiken (incl. hybride rolluik), jaloezieën, verticale lamellen, plissegordijnen, terrasoverkappingen.
6. **Schema**: `LocalBusiness` (or `Service` for product pages) JSON-LD in head. Add `FAQPage` schema as a second JSON-LD block if the page is FAQ-heavy, matching the visible FAQ.
7. **Language**: Dutch (`nl-NL`) — SLS Zonwering's existing pages are entirely in Dutch.
8. **Internal links**: link to the contact/offerte page as primary CTA; add 1-2 contextual internal links to related product or nearby-plaats pages.
9. **Title tag**: ≤60 chars, format `{Page topic} | SLS Zonwering`.
10. **Meta description**: 150-160 chars, includes the primary keyword + a verb + value proposition (e.g. "gratis offerte", "op maat gemaakt").
11. **Contact info**: use `info@slszonwering.nl` as the confirmed contact email; do not invent a phone number or physical address unless the user supplies one.

## Publish parameters — call the site's WordPress MCP `create_content`

```json
{
  "post_type": "page",
  "title": "{title tag without | SLS Zonwering suffix}",
  "slug": "zonwering-in-{plaats-kebab-case}",
  "status": "publish",
  "content": "{short plain-text excerpt of the lead paragraph, ~200 chars}",
  "excerpt": "{1-sentence summary}",
  "meta_title": "{full title tag}",
  "meta_description": "{meta description}",
  "focus_keyword": "{primary keyword}",
  "canonical_url": "https://www.slszonwering.nl/{slug}/",
  "robots_noindex": false,
  "robots_nofollow": false,
  "schema_type": "LocalBusiness",
  "social_title": "{full title tag}",
  "social_description": "{meta description}",
  "render_enabled": true,
  "render_html": "{FULL HTML DOCUMENT from <!doctype html> through </html>}",
  "render_css": "",
  "render_javascript": "",
  "use_global_header": true,
  "use_global_footer": true
}
```

I'm not certain whether SLS Zonwering's existing pages use a global header/footer (`use_global_header`/`use_global_footer` set to `true`) or fully self-contained pages like the QuasarAISEO example (`false`) — the live pages appear to share a common site header/nav across URLs, so `true` is the more likely default, but confirm this against how an existing page (e.g. `/zonwering-in-zwijndrecht/`) is actually built before publishing. Also verify the exact field names against whichever WordPress MCP connector is actually configured — the schema above is inferred from the QuasarAISEO example, not confirmed for this site.

## Verification step (always run after publish)

1. Call `get_content` with the returned `id`.
2. Confirm: `status` is `publish`, `render_html` is non-empty, `slug` matches, `canonical_url` matches.
3. Report to user:
   - Live URL: `https://www.slszonwering.nl/{slug}/`
   - Page ID
   - Title tag
   - Meta description
   - Focus keyword

## Class prefix convention

Use `sls-` for general/local pages, or a short product-specific prefix (e.g. `hr-` for hybride rolluik, `km-` for knikarmscherm) if building a dedicated product landing page, to avoid CSS collisions between pages on the same site.

## Common pitfalls to avoid

- Do NOT include `<!doctype html>`/`<html>`/`<head>`/`<body>` tags twice.
- Do NOT use external CSS files — all styles must be in `<style>` tags inside `render_html`.
- Do NOT invent trust statistics, review counts, prices, phone numbers, or addresses — ask the user or omit.
- Do NOT forget the `onerror` fallback on hero/product images.
- Do NOT skip the FAQ accordion script — without it the FAQ items won't toggle.
- Do NOT assume the design tokens in this file are final — verify colors/fonts against a live page before the first real publish, and update this file once confirmed.
