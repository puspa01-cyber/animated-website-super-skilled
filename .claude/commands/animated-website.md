---
description: 9-step pipeline that builds a complete animated single-page website from a text brief using Brand, UI UX Pro Max, Design System, Design, UI Styling, Stitch, Magic, Firecrawl, and Nano Banana 2.
argument-hint: [optional brief]
---

# Animated Website Builder Pipeline

You are orchestrating a 9-step pipeline that turns a short brief into a complete animated single-page website.

Brief-resolution order (use the FIRST source that resolves):

1. **Arguments** — if `$ARGUMENTS` is non-empty, treat it as the brief and proceed.
2. **brief.txt** — if no arguments, check for `brief.txt` in the current working directory. If it exists, Read it and treat its contents as the brief. Before proceeding, print exactly:

   > Loaded brief from `brief.txt`. Starting pipeline.

   Then continue to Step 1 without re-prompting. If the user wants to override, they can re-invoke with `/animated-website <inline brief>`.

**HARD STOP — no brief, no run:** if neither `$ARGUMENTS` nor `brief.txt` resolves, your one and only response must be exactly:

> No brief found. Provide one inline (`/animated-website <brief>`) or create `brief.txt` in the project root, then re-run.

Do NOT ask clarifying questions, do NOT proceed to Step 1, do NOT invent a brief. Stop immediately.

After receiving the brief, execute the pipeline below, narrating each step briefly (one sentence per step) before running its tool calls. For every optional tool, if it is unavailable or fails, apply the fallback and keep moving — never halt the pipeline on an optional failure.

---

## Step 0 — MCP Pre-flight Check (REQUIRED before Step 1)

Before running Step 1, verify that the two MCP servers used later in the pipeline are available in this session:

- **`stitch`** (used in Step 5 — screen layout generation)
- **`firecrawl-mcp`** (used in Step 4 — reference URL scraping)

Check by looking for any tool whose name begins with `mcp__stitch__` and any tool whose name begins with `mcp__firecrawl-mcp__`. If EITHER is missing, print exactly this warning and then **stop and wait for user confirmation** before continuing:

> ⚠ **MCP servers missing.** This pipeline expects the following MCPs to be installed globally in Claude Code:
>
> - `stitch` — required by Step 5 (screen layouts)
> - `firecrawl-mcp` — required by Step 4 (reference scrape)
>
> Missing: `<list the missing ones>`
>
> Install them with `claude mcp add <name> ...` and restart Claude Code, or reply **"continue anyway"** to run the pipeline with those steps falling back to hand-authored alternatives.

If both MCPs are present, print one line (`MCP pre-flight OK — stitch + firecrawl available.`) and proceed to Step 1 without pausing.

If the user replies "continue anyway" (or equivalent), note which MCPs will fall back and proceed to Step 1 — do not re-prompt later.

---

## Step 1 — Intake & Plan

Extract from the brief: product/topic, audience, tone, primary CTA, any reference URLs, any brand colors/fonts the user named, and preferred style keywords. If critical info is missing, ask one consolidated question; otherwise proceed with reasonable defaults.

Produce a short plan (sections, palette direction, typography direction, motion language). Show the plan to the user and ask for confirmation before continuing. If the user says "go" or similar, continue without re-confirming later steps.

## Step 2 — Brand Voice (brand — REQUIRED)

Invoke the `brand` skill to lock in:
- voice and tone rules (e.g., playful vs. authoritative, formal vs. casual)
- messaging framework (tagline, value prop, 3–5 key phrases)
- word choices to use and to avoid

Record these as the single source of truth for ALL copy written in later steps. Headlines, subtitles, button labels, body text, meta description, and OG text must all conform to this voice.

## Step 3 — Design System (ui-ux-pro-max + design-system — REQUIRED)

First, invoke `ui-ux-pro-max` to select:
- a design style (from the 50+ styles)
- a color palette (from the 161 palettes)
- a font pairing (from the 57 pairings)

Then invoke `design-system` to formalize those choices into a three-layer token architecture:
- **primitive** tokens (raw hex, raw px)
- **semantic** tokens (color-surface, color-accent, space-section, font-heading)
- **component** tokens (button-bg, card-radius, hero-title-size)

Record hex values, font families, radii, spacing scale, and motion timings as CSS custom properties. These tokens are locked — do not invent new values in later steps.

## Step 4 — Reference Scrape (Firecrawl — OPTIONAL)

Only if the brief included a reference URL:
- Call `firecrawl_scrape` to pull the page's structure, copy tone, and hero layout.
- Summarize what to borrow (layout rhythm, headline voice) vs. what to avoid.

Fallback: skip silently if no URL was given or the MCP is unavailable.

## Step 5 — Screen Layouts (Stitch — OPTIONAL)

If the `stitch` MCP is green:
- Call `create_project` → `generate_screen_from_text` for each section (hero, features, social proof, pricing/CTA, footer) using the locked design tokens from Step 3.
- Call `fetch_screen_code` to retrieve the generated React/HTML per screen.

Fallback: if Stitch is unavailable, hand-author section layouts directly in the final HTML in Step 9 using the Step 3 tokens.

## Step 6 — Hero & Feature Imagery (OPTIONAL, 4-tier fallback chain)

Generate every image named in the brief's imagery section (hero, about, menu, play area, adoptable portraits, gallery, knight character cards). Walk the tiers in order — use the FIRST tier that succeeds for each image; do not downgrade successful generations.

### Tier 1 — `nano-banana-2` skill (preferred)

Invoke the `nano-banana-2` skill. For each image:
- Pass the full prompt from the brief, the target aspect ratio (16:9 / 4:3 / 3:4 / 1200×630 per the brief's imagery section), and `resolution: "4K"` for the hero.
- Run generations in parallel when independent.
- Save returned URLs/paths.

Skill shells out to `infsh` under the hood — requires `infsh login` once on the machine. If `infsh` is not installed or not authenticated, move to Tier 2.

### Tier 2 — `design` skill (SVG illustrations via Gemini 3.1 Pro)

Invoke the `design` skill to produce illustrated SVG for each remaining image. Uses its own Gemini path (not `infsh`) — different auth surface, may succeed when Tier 1 fails.

Instruct the skill to match Step 3's palette and the brief's art direction (Pixar 3D / painterly storybook), at the correct aspect ratios.

### Tier 3 — Hand-authored inline SVG (high-quality rules — MANDATORY if reached)

If Tiers 1 and 2 both fail, hand-author inline SVG. Do NOT produce flat geometric blobs. Every illustration must use at least THREE of the following techniques:

- **Organic texture:** `<feTurbulence>` + `<feDisplacementMap>` filters for paper/grain feel
- **Directional light:** 2–3 layered radial gradients (top-left warm highlight, bottom-right cool shadow) simulating a single light source
- **Painterly depth:** 3–5 overlapping `<path>` layers per subject, slightly offset, with `mix-blend-mode="multiply"` on overlays for richer color
- **Atmospheric perspective:** background layers get `filter="blur(0.3px)"` and reduced saturation; foreground stays crisp
- **Palette from Step 3 tokens only** — no ad-hoc colors. Use CSS custom properties via `fill="var(--color-primary)"` so the illustration re-themes automatically.
- **Friendly geometry:** match Step 3's radius scale (default `lg` 24px equivalent on SVG corners), rounded caps on all strokes, no sharp 90° angles on decorative elements.
- **Motion-ready:** wrap animatable sub-groups in `<g class="breathe">` or similar hooks so Step 9 can add subtle CSS keyframe animation (slow scale 1.00↔1.02, 6s ease-in-out infinite).

Match the brief's style keywords (warm, playful, cozy, Pixar 3D, storybook-illustrative) — rounded friendly characters, not minimalist corporate vector art.

### Tier 4 — Curated free-SVG library (absolute last resort)

If hand-authoring is blocked (e.g., time/complexity), pull from a royalty-free library and recolor to Step 3's palette:
- `undraw.co` — flat illustration, easy palette swap via single accent color
- `Storyset` — more character-rich, customizable via their generator
- `Blush` — modular character illustrations

Document the source and license in a comment at the top of each embedded SVG.

**Whichever tier supplies each image, write descriptive `alt` text derived from the brief.**

## Step 7 — Brand Assets (design — REQUIRED)

Invoke the `design` skill to produce three deliverables using Step 2's voice and Step 3's tokens:

1. **Logo** — generate a logo (55 styles available, Gemini AI). Save as SVG/PNG at the project root.
2. **Section icons** — generate an SVG icon set (15 styles available, Gemini 3.1 Pro) covering every section named in Step 1's plan. Keep stroke/fill consistent with Step 3's tokens.
3. **Social/OG image** — generate a share card (HTML→screenshot or direct render) featuring the logo, tagline, and hero visual for use in `<meta property="og:image">`.

Run these in parallel when possible.

## Step 8 — Polished Components (Magic + ui-styling — OPTIONAL/REQUIRED mix)

**Marketing blocks (Magic — OPTIONAL):** If the brief specifically asked for a 21st.dev component (navbar, pricing table, testimonial grid, bento, etc.) AND the `magic` MCP is green:
- Call `21st_magic_component_builder` with the request plus the Step 3 tokens.
- Splice the returned component into the corresponding section.

Fallback: hand-code the component using Tailwind + the Step 3 tokens.

**Interactive components (ui-styling — REQUIRED):** Invoke the `ui-styling` skill to produce shadcn/ui + Tailwind implementations of any interactive patterns the plan needs:
- FAQ accordion
- Contact form (with validation states)
- Testimonial carousel
- Menu/feature cards
- Dialog/modal for adoption details, event signup, etc.

All component styles must pull from Step 3's token layer — no ad-hoc colors or spacing.

## Step 9 — Assemble & Animate (REQUIRED)

Produce a single `index.html` at the project root that:
- Loads Tailwind (CDN is fine for a one-file deliverable) and the chosen Google Fonts.
- Loads Framer Motion via its UMD build (or uses CSS `@keyframes` + `IntersectionObserver` if Motion is not desired).
- Applies the Step 3 tokens as CSS custom properties on `:root`.
- Composes Step 5's sections (or hand-authored fallbacks) top-to-bottom.
- Embeds Step 6's images (or SVG fallbacks) and Step 7's logo + icons.
- Splices Step 8's marketing blocks and interactive components into their sections.
- Uses Step 2's voice for every piece of copy (headlines, subtitles, buttons, meta, OG).
- References Step 7's OG image in `<meta property="og:image">`.
- Adds motion: hero entrance (fade + rise), scroll-reveal on each section, subtle hover micro-interactions on CTAs, prefers-reduced-motion respected.
- Is responsive (mobile-first, breakpoints at 640 / 1024).
- Includes a real `<title>`, meta description, and OG tags derived from the brief and Step 2 voice.

### Build provenance (REQUIRED — for grading)

As you assemble, track for every optional tool whether it **fired** or **fell-back**, and for each Step 6 image record which tier won (1 nano-banana-2, 2 design-skill SVG, 3 hand-authored SVG, 4 free-library SVG). Emit this in TWO places:

**1. HTML comment at the very top of `index.html`** (immediately after `<!DOCTYPE html>`):

```html
<!-- build-provenance
  pipeline: /animated-website
  generated-at: <ISO-8601 UTC timestamp>
  brief-source: arguments | brief.txt
  mcp:
    stitch: fired | fell-back | skipped
    firecrawl: fired | fell-back | skipped
    magic: fired | fell-back | skipped
  skills:
    brand: fired
    ui-ux-pro-max: fired
    design-system: fired
    design: fired
    ui-styling: fired
    nano-banana-2: fired | fell-back | skipped
  images:
    hero: tier-1 | tier-2 | tier-3 | tier-4
    <each-named-image>: tier-N
-->
```

**2. Sidecar `build-report.json`** written next to `index.html` with the same fields as structured JSON:

```json
{
  "pipeline": "/animated-website",
  "generatedAt": "<ISO-8601 UTC>",
  "briefSource": "arguments|brief.txt",
  "mcp": { "stitch": "...", "firecrawl": "...", "magic": "..." },
  "skills": { "brand": "fired", "ui-ux-pro-max": "fired", "design-system": "fired", "design": "fired", "ui-styling": "fired", "nano-banana-2": "..." },
  "images": { "hero": "tier-1", "...": "tier-N" }
}
```

Values: `fired` = tool actually ran and its output is in the page; `fell-back` = tool was attempted and failed, fallback used; `skipped` = tool was never attempted (e.g., no reference URL for firecrawl, no 21st.dev block requested for magic). Do NOT mark anything `fired` unless it genuinely ran in this session — the instructor will grade against these values.

### Summary to user

After writing the file, print a short summary:
- what was built,
- which optional tools fired vs. fell back (must match the provenance block),
- how to preview (`python3 -m http.server 8000` then open `http://localhost:8000`).

Do NOT open a browser or start a server unless the user asks.

---

## Pipeline Rules

- Never stop the pipeline for an optional-tool failure — log it and fall back.
- Never invent design tokens outside Step 3's output.
- Never write copy that violates Step 2's voice rules.
- Keep tool calls parallel when steps are independent (e.g., Step 6 image generations, Step 7 logo/icons/social in parallel).
- The final deliverable is one `index.html` unless the user asks for a multi-file build.

$ARGUMENTS
