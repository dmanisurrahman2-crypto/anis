# Zonneschermen op maat — draft (NOT published)

Status: **draft only**. This page has not been published to slszonwering.nl.

## Why it's not published yet

The `sls-zonwering` MCP server (custom-web-render at
`https://www.slszonwering.nl/wp-json/custom-web-render/v1/mcp`) cannot connect
in this session: the environment's network egress proxy blocks the
`www.slszonwering.nl` host entirely. This has been true since the MCP server
was first configured and is unchanged as of this draft. Because of that,
neither the WordPress media library nor the `create_content` publish call is
reachable from here.

## What's in this folder

- `render.html` — the full self-contained page (head/meta/JSON-LD, hero,
  product overview, werkwijze, "waarom SLS Zonwering", FAQ with `FAQPage`
  schema, scripts). Written natively in Dutch per the brief in
  `slsnewpageseobriefs.docx`, page 1: "Zonneschermen op maat" (`/zonneschermen/`).
- `publish-params.json` — the `create_content` call payload, with the full
  `render_html` embedded, `use_global_header`/`use_global_footer` set to
  `true` as requested, `render_enabled: true`.

## Before this can actually be published

1. **Fix the network/allowlist block** so the `sls-zonwering` MCP server can
   connect (environment/network settings, outside what this session can
   change on its own).
2. **Replace the two image placeholders** in `render.html` (and the
   `social_image` placeholder in `publish-params.json`) with real URLs picked
   from the SLS Zonwering WordPress media library:
   - Hero image: a photo of an installed knikarmscherm or uitvalscherm on a
     terrace/gevel.
   - Product-overview image (near the "Drie soorten zonneschermen op maat"
     heading): a close-up of the doek/frame or a markies.
   Both already have descriptive Dutch `alt` text written in — only the `src`
   needs swapping in.
3. **Confirm the CTA target.** No real offerte/contact page URL was
   confirmed for this site, so the CTA currently points to
   `mailto:info@slszonwering.nl` (the one confirmed contact channel from the
   skill file). Swap in the real offerte-form URL if one exists.
4. **Verify the `create_content` field names** against whatever WordPress
   MCP tool is actually exposed once the server connects — the field names
   in `publish-params.json` are inferred from the quasaraiseo Custom Web
   Render example, not confirmed for this site (see the warning in
   `.claude/skills/sls-zonwering-page-publish/SKILL.md`).

No prices, review counts, or years-in-business figures were invented — the
brief and skill file both flag that these weren't available, so the FAQ price
answer points people to requesting a quote instead of stating a number.
