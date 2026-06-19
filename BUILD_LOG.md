# Profile README — Build Log

How the GitHub profile (`README.md` on `main`) was designed and built, the techniques used, the decisions made, and how to maintain it.

> This file lives on the `build-log` branch only (not on `main`), so it doesn't appear on the rendered profile. Note: a public repo has no truly private files/branches — for real privacy this belongs in a separate **private** repo.

---

## Final structure (top → bottom)

1. **Hero banner** — self-hosted SVG (`assets/banner.svg`)
2. **Contact row** — LinkedIn · email · profile-views badges
3. **About** — baked icon heading + intro paragraph + a `→` focus list
4. **Tech Stack** — baked icon headings + uniform self-hosted brand icons (6 categories)
5. **Featured Work** — NEURA and Operational Analytics Platform, each with metric badges + `<kbd>` tech keys
6. **Footer CTA** — one line + a LinkedIn "Message me" (DM-compose) button

---

## Components — what they are and why

### Hero banner — `assets/banner.svg`
Hand-authored static SVG: diagonal teal gradient, two soft radial "glow" orbs, dotted accent grids, and the name + tagline as SVG `<text>`.
**Why self-hosted:** banner generators (capsule-render) returned broken SVGs — empty wave paths (`d=""`) and styling locked in a `<style>` block that GitHub's sanitizer strips — so the name vanished. A committed static SVG renders reliably and is fully under our control.

### Contact row
`shields.io` **flat-square** badges:
- **LinkedIn** — logo + word, links to the profile.
- **Email** — shows the address (lowercase), `mailto:` link.
- **Profile Views** — komarev counter, teal.

**Why flat-square:** the bold `for-the-badge` style force-uppercases text, which made the email render `MAW180604@GMAIL.COM`. flat-square preserves case and keeps all three badges consistent.

### Section headings — baked icon + text SVGs
Files: `assets/icons/h-<name>.svg` (light) and `assets/icons/h-<name>-d.svg` (dark), referenced via `<picture>` with `prefers-color-scheme`.
Each heading is **one image that contains both the custom line-icon and the heading text**.
**Why baked into one image:** GitHub strips CSS from READMEs and ignores `align` for vertical centering, and it draws borders on tables — so a *big* icon next to markdown heading text can't be vertically centered (it sits on the text baseline and looks high) without shrinking it. Baking icon + text into a single SVG locks their alignment at any size.
- **Dual-theme:** two variants per heading (dark text `#1F2328` for light mode, light text `#E6EDF3` for dark mode).
- **Colors:** each icon has a meaning-fitting color; the **Tools** gear is theme-adaptive (`#4B5563` light / `#B6C0CE` dark) because a single gray reads as a dark "light-mode" icon on the dark background.
- **Centering:** icon centered on its box; text baseline at `H/2 + 0.35·fontSize`; the bar-chart icon is nudged up `1.5` because its bars hang from the baseline.

### Tech-stack icons — uniform self-hosted brand SVGs
Files: `assets/icons/<tool>.svg` (40 icons).
Real brand logos pulled from **open-source icon sets** — [simple-icons](https://simpleicons.org) (CC0) via jsdelivr, plus **Matplotlib** from [devicon](https://devicon.dev) (MIT) — then **committed into the repo** (self-hosted → can't rate-limit or 404).
- **Uniform sizing:** every logo is normalized into an invisible **56×56** square canvas (icon scaled so its longest side = 40px, centered → even transparent padding). So despite different native dimensions, all render at the same visual size with consistent gaps. No CSS needed.
- **Dual-theme color:** brand-colored, with navy/near-black/yellow logos shifted to tones readable on both light and dark.
- **Hover tooltips:** each `<img>` has a `title` attribute, so hovering shows the tool name (desktop).
- **Fallback glyphs (hand-drawn, no official logo exists):** SQL (database cylinder), nnU-Net v2 (neural net), ETL pipelines (pipeline flow). Codex → OpenAI logo; Claude Code → Claude logo; GitHub Copilot → its own logo.

### Featured Work
Full-width cards. Achievements shown as `shields.io` metric badges (e.g., `Whole-Tumor Dice 0.92`, `~16M rows`, `sub-500ms`), tech shown as `<kbd>` keys.

### Footer CTA
One centered line + a `for-the-badge` LinkedIn button linking to the DM-compose URL
(`https://www.linkedin.com/messaging/compose/?recipient=muhammadabdullahwasim`).

---

## Key technical lessons

- **Prefer self-hosted SVGs over third-party generators.** Observed during the build: capsule-render returned broken SVGs; the Heroku streak widget and github-profile-trophy were down/limited (trophy returned HTTP 402); the main github-readme-stats card was rate-limited ("Something went wrong"). Reliable building blocks: **shields.io**, **self-hosted SVG**, and (while they were used) github-readme-activity-graph and readme-typing-svg.
- **GitHub sanitizes README HTML:** strips `<style>`/CSS, ignores `align` for vertical centering, and borders tables. This is what drove the baked-image heading approach and the self-hosted icons.
- **Removed earlier experiments:** typing animation and the contribution-activity graph (duplicated GitHub's native graph), the streak widget and trophies (unreliable).
- **Contribution-graph hygiene:** iteration history was squashed; design options were explored on non-default branches (commits there don't count toward the contribution graph).

---

## Asset inventory

| Path | What |
|------|------|
| `assets/banner.svg` | Hero banner (name + tagline) |
| `assets/icons/h-*.svg` + `h-*-d.svg` | 11 baked section headings × 2 (light/dark) |
| `assets/icons/<tool>.svg` | 40 normalized brand icons + 3 hand-drawn glyphs |

---

## Maintaining it

- **Add a tool:** drop a normalized 56×56 SVG into `assets/icons/` and add an `<img src="assets/icons/<tool>.svg" width="46" height="46" title="<Name>" alt="<Name>"/>` to the right category cell.
- **Change a heading (text/icon/color):** regenerate that heading's two SVGs (light + dark) — they bake the icon + text together; the README references them via `<picture>`.
- **Brand icons** were fetched from simple-icons (jsdelivr) / devicon and normalized to the 56×56 canvas by a Python script; the heading icons were generated by a separate Python script (icon paths + text baked in, light/dark variants).
- **Keep colors mid-tone** so they stay readable in both light and dark mode.

---

## Timeline (high level)

1. Cleaned the original profile (fixed the dead Heroku streak domain → demolab; removed a fake `+1000` on the views counter; deleted stale comments).
2. Several overhaul/trim passes; diagnosed why image-banner generators broke on GitHub.
3. Explored 4 design directions on branches (terminal, minimal-editorial, hero-banner, split-panel dashboard); chose the **hero banner**.
4. Built the uniform **self-hosted icon system** (normalized 56×56) + hand-drawn fallback glyphs.
5. Enhanced **Skills** and **Featured Work** (metric badges); refined the **contact row**.
6. Removed the typing animation and activity section; rewrote **About**; added the **footer CTA** + LinkedIn DM button.
7. Replaced heading emojis with **custom icons**, then **baked icon + text** into single SVGs for perfect alignment; per-icon colors; theme-adaptive Tools gear.
8. Merged to `main`, deleted the working branches, tidied the footer, and added **hover tooltips** to skill icons.
