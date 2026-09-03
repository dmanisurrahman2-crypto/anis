---
name: sls-zonwering-page-publish
description: Generate and publish a SEO-friendly, AEO+GEO optimized, mobile-responsive landing page on slszonwering.nl following the SLS Zonwering design template. Use when the user says "create page", "publish page", "maak een pagina", "nieuwe pagina", "add page to slszonwering", "publish to slszonwering", or any request to build a new page on the SLS Zonwering WordPress site. Handles full pipeline: keyword → copy → HTML → WordPress MCP publish.
---

# SLS Zonwering Page Publisher

This skill generates and publishes a complete SEO/AEO/GEO-optimized landing page on **slszonwering.nl** following a design template modeled on the QuasarAISEO page-publish pipeline. One request → one published page.

> **Unverified template — read before using.** This SKILL.md was drafted without being able to load `https://www.slszonwering.nl/` or connect the `sls-zonwering` MCP server: this environment's network egress proxy blocks the `www.slszonwering.nl` host entirely (confirmed via both a direct fetch attempt and the MCP connection, which failed with `AUTH_HEADER_REJECTED` / "no rule or allowlist entry allows host www.slszonwering.nl"). That means:
> - The **design tokens** below (colors, fonts, section layout) are placeholders inspired by the QuasarAISEO template, not the real SLS Zonwering brand — nobody has inspected the live site's actual look.
> - The **publish tool name** (`create_content`, `get_content`) is inferred only from the fact that the MCP endpoint path (`/wp-json/custom-web-render/v1/mcp`) matches the same "Custom Web Render" WordPress plugin route used on quasaraiseo.com — plausible, but unconfirmed.
> - Business specifics (product names, tone, CTA URL) are generic assumptions for a Dutch *zonwering* (sun-shading: screens, rolluiken, markiezen, terrasoverkappingen) company.
>
> Before relying on this skill for a real publish: fix the network/allowlist issue so the `sls-zonwering` MCP server can connect (or open the site from a machine that isn't blocked), pull the real brand colors/fonts/logo and an actual example page, and update this file's §Design system and §Publish parameters accordingly.

## When to invoke

Trigger whenever the user asks for a new page on slszonwering.nl, e.g.:
- "create a page about zonnescherm X"
- "publish a landing page for rolluiken"
- "maak een pagina over markiezen"
- "add a new service page for terrasoverkapping"
- "publish to slszonwering"

Do NOT invoke for: blog posts, editing existing pages (use `update_content` directly once the real tool name is confirmed), or non-SLS-Zonwering sites.

## Pipeline (always run in this order)

1. **Clarify inputs** — if user did not specify, ask in ONE question:
   - Topic / product-service name (e.g. "zonnescherm", "rolluiken", "markiezen", "terrasoverkapping")
   - Primary keyword (focus_keyword) — default to topic slug
   - Language: `nl-NL` (default; this is a Dutch business)
   - CTA target URL — default to the site's contact/offerte page, e.g. `https://www.slszonwering.nl/offerte-aanvragen/` (confirm the real path before first use)
2. **Generate the page** — build full HTML using the Template below (single `render_html` field, CSS inline, scripts at end).
3. **Publish via WordPress MCP** — call the `sls-zonwering` MCP server's content-creation tool (name unconfirmed — see warning above; try `create_content` first, and if it doesn't exist, run tool discovery against the connected server before publishing).
4. **Verify** — read the created page back (e.g. `get_content`) with the returned ID; confirm `render_html` saved and `status=publish`. Report the live URL to the user.

## Template — required structure

Every page follows this section order. Use a consistent 2-3 letter class prefix — default to `sz-` for SLS Zonwering pages.

### Section list (in order)

| # | Section | Required | Notes |
|---|---|---|---|
| 1 | `<head>` with SEO meta + JSON-LD | yes | See §Head block |
| 2 | Hero | yes | 2-col grid: copy + product image, badge, H1 with gradient span, lead, CTA button, 4 stats |
| 3 | How it works | yes | 4 steps in a 4-col grid (e.g. adviesgesprek → opmeten → maatwerk productie → montage) |
| 4 | Process timeline (optional) | no | Use only for topics with 5+ sequential phases |
| 5 | What you get | yes | 6 cards in 2-col grid, each with icon + H3 + p (materials, warranty, options, motorisatie, onderhoud, service) |
| 6 | FAQ | yes | 5 accordion items, click-to-toggle, chevron rotates |
| 7 | Scripts | yes | FAQ accordion toggle + header nav init (see §Scripts) |

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
<script type="application/ld+json">{"@context":"https://schema.org","@type":"Service","mainEntityOfPage":{"@type":"WebPage","@id":"https://www.slszonwering.nl/{slug}/"},"headline":"{title}","description":"{meta description}","url":"https://www.slszonwering.nl/{slug}/","datePublished":"{ISO 8601 now}","dateModified":"{ISO 8601 now}","provider":{"@type":"LocalBusiness","name":"SLS Zonwering","url":"https://www.slszonwering.nl/"}}</script>
<style>
html, body { height: auto; }
body { overflow-x: hidden; }
.sz-header-container { position: sticky; top: 0; z-index: 99999; }
.sz-header-nav { z-index: 999999; }
.sz-how, .sz-deliver, .sz-faq { scroll-margin-top: 100px; }
</style>
</head>
<body>
```

### Design system (PLACEHOLDER — confirm against the real site before publishing)

- **Font**: `Inter` (400, 500, 600, 700, 800) via Google Fonts `@import` — unverified, check the live site's actual font
- **Icons**: FontAwesome 6.5.1 via `@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css')`
- **Primary gradient (placeholder — sun/shade theme)**: `linear-gradient(90deg, #F59E0B, #1E3A8A)` (amber → deep blue) — used for H1 gradient span, primary buttons, icon backgrounds
- **Secondary gradient (icons)**: `linear-gradient(180deg, #FB923C 0%, #1E40AF 100%)`
- **Text colors**: H1/H2 `#111827`, body `#4B5563`, muted `#6B7280`, dark `#374151`
- **Backgrounds**: hero `#ffffff`, how `#ffffff`, deliver `#f8fafc`, FAQ `#ffffff`
- **Borders**: card `#E5E7EB`, hover `#1E40AF`
- **Border radius**: cards `16px`, buttons `10px`, pill `9999px`, icons `999px`
- **Section padding**: hero `72px 24px`, others `86px 24px`
- **H1**: `56px / weight 800 / line-height 1.06 / letter-spacing -0.03em`
- **H2**: `42px / weight 800 / line-height 1.2 / letter-spacing -0.025em`
- **Animation**: fade-up keyframe (`opacity 0→1, translateY 18px→0`, `.6s ease-out forwards`) with staggered `animation-delay` per child

**Before first real publish**: replace this section with the actual brand colors, font, logo, and hero image URL pulled from the live site or supplied by the site owner.

### Hero section skeleton

```html
<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');
  .sz-hero-container{font-family:'Inter',sans-serif;background:#ffffff;color:#111827;padding:72px 24px;overflow:hidden;position:relative}
  .sz-hero-wrapper{max-width:1200px;margin:0 auto;display:grid;grid-template-columns:1.05fr .95fr;align-items:center;gap:52px;position:relative;z-index:2}
  .sz-badge{display:inline-block;padding:7px 12px;background:#fff;color:#1E40AF;font-size:14px;font-weight:700;border-radius:9999px;margin-bottom:18px;border:1px solid rgba(30,64,175,.18)}
  .sz-h1{font-size:56px;font-weight:800;line-height:1.06;margin:0 0 18px 0;color:#111827;letter-spacing:-0.03em}
  .sz-gradient{background:linear-gradient(90deg,#F59E0B,#1E3A8A);-webkit-background-clip:text;-webkit-text-fill-color:transparent}
  .sz-lead{font-size:18px;line-height:1.7;color:#4B5563;margin:0 0 26px 0;max-width:560px}
  .sz-btns{display:flex;flex-wrap:wrap;gap:14px;margin:0 0 44px 0}
  .sz-btn{display:inline-flex;align-items:center;justify-content:center;padding:14px 26px;font-size:16px;font-weight:700;border-radius:10px;text-decoration:none;border:2px solid transparent;transition:all .28s ease;cursor:pointer}
  .sz-primary{background:linear-gradient(90deg,#F59E0B,#1E3A8A);color:#fff}
  .sz-primary:hover{opacity:.92;box-shadow:0 12px 22px -6px rgba(30,64,175,.28);transform:translateY(-2px)}
  .sz-stats{display:grid;grid-template-columns:1fr 1fr;gap:28px}
  .sz-stat-num{font-size:34px;font-weight:800;color:#111827;letter-spacing:-0.02em}
  .sz-stat-label{font-size:12px;color:#6B7280;text-transform:uppercase;letter-spacing:.08em}
  .sz-imgwrap{position:relative;display:flex;justify-content:center;align-items:center}
  .sz-imgwrap img{width:100%;max-width:600px;border-radius:18px;box-shadow:0 26px 52px -16px rgba(0,0,0,.25)}
  @media(max-width:768px){.sz-hero-wrapper{grid-template-columns:1fr;text-align:center}.sz-h1{font-size:40px}.sz-lead{margin-left:auto;margin-right:auto}.sz-btns{justify-content:center}.sz-imgwrap{order:1}}
  @media(max-width:480px){.sz-h1{font-size:28px}.sz-hero-container{padding:52px 16px}.sz-stats{grid-template-columns:1fr;gap:18px}}
  @media(max-width:360px){.sz-h1{font-size:24px}.sz-hero-container{padding:44px 14px}}
</style>

<div class="sz-hero-container">
  <div class="sz-hero-wrapper">
    <div class="sz-hero-content">
      <span class="sz-badge">☀️ {badge text}</span>
      <h1 class="sz-h1">{H1 line 1}<br><span class="sz-gradient">{H1 gradient line}</span></h1>
      <p class="sz-lead">{lead paragraph with <strong>bold</strong> keywords}</p>
      <div class="sz-btns">
        <a class="sz-btn sz-primary" href="{CTA url}">{CTA label} →</a>
      </div>
      <div class="sz-stats">
        <div><div class="sz-stat-num">{stat1}</div><span class="sz-stat-label">{label1}</span></div>
        <div><div class="sz-stat-num">{stat2}</div><span class="sz-stat-label">{label2}</span></div>
        <div><div class="sz-stat-num">{stat3}</div><span class="sz-stat-label">{label3}</span></div>
        <div><div class="sz-stat-num">{stat4}</div><span class="sz-stat-label">{label4}</span></div>
      </div>
    </div>
    <div class="sz-imgwrap">
      <img src="{real product image URL — placeholder, confirm on the live site}" alt="{descriptive alt}" onerror="this.src='https://placehold.co/600x400/1E3A8A/FFF?text=Afbeelding+Fout'; this.onerror=null;" />
    </div>
  </div>
</div>
```

### How it works section skeleton (4 steps)

```html
<style>
  @keyframes szFadeUp{from{opacity:0;transform:translateY(18px)}to{opacity:1;transform:translateY(0)}}
  .sz-how{font-family:'Inter',sans-serif;background:#ffffff;color:#111827;padding:86px 24px;overflow:hidden}
  .sz-how .sz-how-head{max-width:820px;margin:0 auto 60px auto;text-align:center;opacity:0;animation:szFadeUp .6s ease-out forwards;animation-delay:.08s}
  .sz-how h2{font-size:42px;font-weight:800;line-height:1.2;letter-spacing:-0.025em;margin:0}
  .sz-how p{font-size:18px;line-height:1.6;color:#4B5563;margin:16px 0 0 0}
  .sz-steps{display:grid;grid-template-columns:repeat(4,1fr);gap:30px;max-width:1200px;margin:0 auto}
  .sz-step{text-align:center;opacity:0;animation:szFadeUp .6s ease-out forwards}
  .sz-step:nth-child(1){animation-delay:.28s}
  .sz-step:nth-child(2){animation-delay:.44s}
  .sz-step:nth-child(3){animation-delay:.60s}
  .sz-step:nth-child(4){animation-delay:.76s}
  .sz-step-ico{width:82px;height:82px;border-radius:999px;background:linear-gradient(180deg,#FB923C 0%,#1E40AF 100%);display:inline-flex;align-items:center;justify-content:center;margin-bottom:22px;box-shadow:0 10px 20px -6px rgba(30,64,175,.35)}
  .sz-step-ico i{font-size:36px;color:#fff}
  .sz-step-kicker{font-size:13px;font-weight:800;color:#1E40AF;margin-bottom:8px;text-transform:uppercase;letter-spacing:.08em}
  .sz-step h3{font-size:20px;font-weight:800;margin:0 0 10px 0}
  .sz-step-desc{font-size:16px;line-height:1.65;color:#4B5563;margin:0}
  @media(max-width:992px){.sz-steps{grid-template-columns:repeat(2,1fr);gap:44px}}
  @media(max-width:768px){.sz-steps{grid-template-columns:1fr}.sz-how h2{font-size:32px}}
  @media(max-width:480px){.sz-how h2{font-size:26px}.sz-how{padding:52px 16px}}
  @media(max-width:360px){.sz-how h2{font-size:22px}.sz-how{padding:44px 14px}}
</style>

<div class="sz-how" id="feature">
  <div class="sz-how-head">
    <h2>{how it works H2, e.g. "Zo werkt het"}</h2>
    <p>{how it works subtitle}</p>
  </div>
  <div class="sz-steps">
    <!-- repeat 4x, e.g. Adviesgesprek, Opmeten, Maatwerk productie, Montage -->
    <div class="sz-step">
      <div class="sz-step-ico"><i class="fa-solid fa-{icon}"></i></div>
      <div class="sz-step-kicker">{Stap 1}</div>
      <h3>{step title}</h3>
      <p class="sz-step-desc">{step description}</p>
    </div>
  </div>
</div>
```

### What you get section skeleton (6 cards)

```html
<style>
  .sz-deliver{font-family:'Inter',sans-serif;background:#f8fafc;color:#111827;padding:86px 24px;overflow:hidden}
  .sz-deliver .head{max-width:860px;margin:0 auto 46px auto;text-align:center;opacity:0;animation:szFadeUp .6s ease-out forwards;animation-delay:.08s}
  .sz-deliver .head h2{font-size:42px;font-weight:800;line-height:1.2;letter-spacing:-0.025em;margin:0}
  .sz-deliver .head p{font-size:18px;line-height:1.6;color:#4B5563;margin:16px auto 0 auto;max-width:700px}
  .sz-cards{display:grid;grid-template-columns:repeat(2,1fr);gap:22px;max-width:1020px;margin:0 auto}
  .sz-card{display:flex;align-items:flex-start;gap:16px;background:#fff;border:1px solid #E5E7EB;border-radius:16px;padding:22px;box-shadow:0 4px 12px -1px rgba(0,0,0,.03),0 2px 8px -1px rgba(0,0,0,.02);transition:all .28s ease;opacity:0;animation:szFadeUp .6s ease-out forwards}
  .sz-card:hover{transform:translateY(-4px);border-color:#1E40AF;box-shadow:0 12px 22px -10px rgba(0,0,0,.12)}
  .sz-card:nth-child(1){animation-delay:.26s}
  .sz-card:nth-child(2){animation-delay:.34s}
  .sz-card:nth-child(3){animation-delay:.42s}
  .sz-card:nth-child(4){animation-delay:.50s}
  .sz-card:nth-child(5){animation-delay:.58s}
  .sz-card:nth-child(6){animation-delay:.66s}
  .sz-icobox{flex-shrink:0;width:34px;height:34px;border-radius:999px;background:linear-gradient(180deg,#FB923C 0%,#1E40AF 100%);display:inline-flex;align-items:center;justify-content:center}
  .sz-icobox i{color:#fff;font-size:14px}
  .sz-card h3{font-size:18px;font-weight:800;margin:0 0 6px 0}
  .sz-card p{font-size:16px;line-height:1.65;color:#4B5563;margin:0}
  .sz-footer{max-width:900px;margin:44px auto 0 auto;text-align:center;opacity:0;animation:szFadeUp .6s ease-out forwards;animation-delay:.78s}
  .sz-footer p{font-size:18px;font-weight:600;color:#374151;line-height:1.65;margin:0 0 26px 0}
  .sz-pill{display:inline-flex;align-items:center;justify-content:center;text-decoration:none;font-size:16px;font-weight:800;color:#fff;background:linear-gradient(90deg,#FB923C 0%,#F59E0B 50%,#1E40AF 100%);background-size:200% auto;padding:14px 28px;border-radius:9999px;transition:all .35s ease;box-shadow:0 5px 18px rgba(30,64,175,.30)}
  .sz-pill:hover{background-position:right center;transform:translateY(-2px);box-shadow:0 8px 24px rgba(30,64,175,.35)}
  .sz-pill i{margin-left:10px;font-size:14px;transition:transform .2s ease}
  .sz-pill:hover i{transform:translateX(4px)}
  @media(max-width:768px){.sz-cards{grid-template-columns:1fr}.sz-deliver .head h2{font-size:32px}}
  @media(max-width:480px){.sz-deliver .head h2{font-size:26px}.sz-deliver{padding:52px 16px}}
  @media(max-width:360px){.sz-deliver .head h2{font-size:22px}.sz-deliver{padding:44px 14px}}
</style>

<div class="sz-deliver" id="review">
  <div class="head">
    <h2>{what you get H2, e.g. "Wat je krijgt"}</h2>
    <p>{what you get subtitle}</p>
  </div>
  <div class="sz-cards">
    <!-- repeat 6x, e.g. materiaalkeuze, motorisatie, garantie, onderhoudsservice, kleuren/stoffen, montage door eigen team -->
    <div class="sz-card"><div class="sz-icobox"><i class="fa-solid fa-check"></i></div><div><h3>{card title}</h3><p>{card description}</p></div></div>
  </div>
  <div class="sz-footer">
    <p>{closing line}</p>
    <a class="sz-pill" href="{CTA url}">{CTA label} <i class="fa-solid fa-arrow-right"></i></a>
  </div>
</div>
```

### FAQ section skeleton (5 accordion items)

```html
<style>
  .sz-faq{font-family:'Inter',sans-serif;background:#ffffff;color:#111827;padding:86px 24px;overflow:hidden}
  .sz-faq .head{max-width:860px;margin:0 auto 46px auto;text-align:center;opacity:0;animation:szFadeUp .6s ease-out forwards;animation-delay:.08s}
  .sz-faq .head h2{font-size:42px;font-weight:800;line-height:1.2;letter-spacing:-0.025em;margin:0}
  .sz-faq .head p{font-size:18px;line-height:1.6;color:#4B5563;margin:16px 0 0 0}
  .sz-faq-list{max-width:860px;margin:0 auto;display:flex;flex-direction:column;gap:14px}
  .sz-faq-item{background:#fff;border:1px solid #E5E7EB;border-radius:16px;box-shadow:0 4px 12px -1px rgba(0,0,0,.03),0 2px 8px -1px rgba(0,0,0,.02);transition:all .28s ease;opacity:0;animation:szFadeUp .6s ease-out forwards}
  .sz-faq-item:nth-child(1){animation-delay:.26s}
  .sz-faq-item:nth-child(2){animation-delay:.34s}
  .sz-faq-item:nth-child(3){animation-delay:.42s}
  .sz-faq-item:nth-child(4){animation-delay:.50s}
  .sz-faq-item:nth-child(5){animation-delay:.58s}
  .sz-q{display:flex;justify-content:space-between;align-items:flex-start;width:100%;text-align:left;padding:22px 22px;font-size:18px;font-weight:750;color:#111827;background:transparent;border:none;cursor:pointer;border-radius:16px}
  .sz-q:hover{background:#f8fafc}
  .sz-q span{flex:1;margin-right:14px;line-height:1.4}
  .sz-q i{font-size:16px;color:#1E40AF;transition:transform .3s ease;flex-shrink:0;padding-top:4px}
  .sz-a{max-height:0;overflow:hidden;padding:0 22px;border-top:1px solid #E5E7EB;margin-top:-1px;transition:max-height .4s ease,padding .4s ease}
  .sz-a p{margin:0;font-size:16px;line-height:1.65;color:#4B5563}
  .sz-faq-item.active .sz-a{max-height:360px;padding:18px 22px 22px 22px}
  .sz-faq-item.active .sz-q{color:#1E40AF}
  .sz-faq-item.active .sz-q i{transform:rotate(180deg)}
  @media(max-width:768px){.sz-faq .head h2{font-size:32px}.sz-q{font-size:16px;padding:18px}.sz-a p{font-size:15px}.sz-q i{padding-top:2px}}
  @media(max-width:480px){.sz-faq .head h2{font-size:26px}.sz-faq{padding:52px 16px}}
  @media(max-width:360px){.sz-faq .head h2{font-size:22px}.sz-faq{padding:44px 14px}}
</style>

<div class="sz-faq" id="faq">
  <div class="head">
    <h2>{FAQ H2, e.g. "Veelgestelde vragen"}</h2>
    <p>{FAQ subtitle}</p>
  </div>
  <div class="sz-faq-list">
    <!-- repeat 5x -->
    <div class="sz-faq-item">
      <button class="sz-q"><span>{question}</span><i class="fa-solid fa-chevron-down"></i></button>
      <div class="sz-a"><p>{answer}</p></div>
    </div>
  </div>
</div>
```

### Scripts (always append at end of body)

```html
<script>
(function(){
  const root=document.querySelector('.sz-faq');
  if(!root) return;
  const items=root.querySelectorAll('.sz-faq-item');
  items.forEach(it=>{
    const btn=it.querySelector('.sz-q');
    if(!btn) return;
    btn.addEventListener('click',()=>{it.classList.toggle('active');});
  });
})();
</script>

<script>
(function(){
  function initHeaderNav(){
    var toggle = document.querySelector('.sz-mobile-toggle');
    var nav = document.querySelector('.sz-header-nav');
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

Every page MUST include all 4 breakpoints (already baked into the skeletons above):

| Breakpoint | Size | Key adjustments |
|---|---|---|
| Tablet | `≤992px` | Steps grid → 2 columns, gap 44px |
| Mobile | `≤768px` | All grids → 1 column, H1 40px, H2 32px, hero text centered, image `order:1`, FAQ font 16px |
| Small Mobile | `≤480px` | H1 28px, H2 24-26px, stats → 1 column, tighter padding (52px 16px) |
| Extra Small | `≤360px` | H1 24px, H2 22px, minimal section padding (44px 14px) |

## SEO / AEO / GEO content rules

1. **Answer-first**: hero lead and FAQ answers must open with the direct answer in the first sentence, then context.
2. **Primary keyword** appears in: title tag, H1, meta description, first 100 words, at least one H3, one FAQ answer, and the URL slug.
3. **Citeerbare passages**: include 1-2 short, fact-dense sentences (materiaal, garantietermijn, levertijd, etc.) that AI-zoekmachines letterlijk kunnen overnemen.
4. **Entities**: name concrete product types (screens, uitvalschermen, rolluiken, markiezen, terrasoverkappingen, jaloezieën) rather than only the generic term "zonwering".
5. **Schema**: JSON-LD `Service`/`LocalBusiness` type in head (default). Add `FAQPage` schema additionally if the page is FAQ-heavy.
6. **Language**: Dutch (`nl-NL`) with `lang="nl-NL"` on `<html>`.
7. **Internal links**: link to the site's offerte/contact page as primary CTA (confirm the real path — placeholder used above). Add 1-2 contextual internal links to related product pages where natural.
8. **No header/footer**: pages use `use_global_header: false` / `use_global_footer: false` (assumed to match the quasaraiseo Custom Web Render convention — confirm once the MCP tool is reachable) — the HTML is fully self-contained.
9. **Title tag**: ≤60 chars, format `{Page topic} | SLS Zonwering`.
10. **Meta description**: 150-160 chars, includes primary keyword + a verb + value proposition.

## Publish parameters — call the `sls-zonwering` MCP server's content tool

**Tool name unconfirmed** (see warning at top). Structure below mirrors the quasaraiseo `create_content` call for the same "Custom Web Render" plugin route; verify field names once the server actually connects.

```json
{
  "post_type": "page",
  "title": "{title tag without | SLS Zonwering suffix}",
  "slug": "{kebab-case-slug}",
  "status": "publish",
  "content": "{short plain-text excerpt of the lead paragraph, ~200 chars}",
  "excerpt": "{1-sentence summary}",
  "meta_title": "{full title tag}",
  "meta_description": "{meta description}",
  "focus_keyword": "{primary keyword}",
  "canonical_url": "https://www.slszonwering.nl/{slug}/",
  "robots_noindex": false,
  "robots_nofollow": false,
  "schema_type": "Service",
  "social_title": "{full title tag}",
  "social_description": "{meta description}",
  "social_image": "{real social share image URL — confirm on the live site}",
  "render_enabled": true,
  "render_html": "{FULL HTML DOCUMENT from <!doctype html> through </html>}",
  "render_css": "",
  "render_javascript": "",
  "use_global_header": false,
  "use_global_footer": false
}
```

**Important**: All CSS and JS must be inlined inside `render_html`. Leave `render_css` and `render_javascript` empty strings.

## Verification step (always run after publish)

1. Read the created page back by ID (e.g. `get_content`).
2. Confirm: `status` is `publish`, `render_html` is non-empty, `slug` matches, `canonical_url` matches.
3. Report to user:
   - Live URL: `https://www.slszonwering.nl/{slug}/`
   - Page ID
   - Title tag
   - Meta description
   - Focus keyword

## Reference files

None yet — unlike the QuasarAISEO skill, there are no confirmed example HTML files for this site's actual template in this repo. Once a real page is published (or an existing page's HTML is pulled from the site), save it here as a reference file and link it from this section.

## Class prefix convention

Default to `sz-` for SLS Zonwering pages, kept consistent for every class on a given page to avoid CSS collisions between pages.

## Common pitfalls to avoid

- Do NOT include `<!doctype html>`/`<html>`/`<head>`/`<body>` tags twice — `render_html` expects one complete document.
- Do NOT use external CSS files — all styles must be in `<style>` tags inside `render_html`.
- Do NOT enable global header/footer until confirmed this site's Custom Web Render setup wants them off (assumed off here, matching quasaraiseo).
- Do NOT forget the `onerror` fallback on the hero image.
- Do NOT skip the FAQ accordion script.
- Do NOT publish with the placeholder design tokens/colors still in place for a real client-facing page — confirm brand colors first.
