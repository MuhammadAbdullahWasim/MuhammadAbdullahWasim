# Profile README — Build Log

How the GitHub profile (`README.md` on `main`) was designed and built, the techniques used, the decisions made, and how to maintain it.

> This file lives on the `build-log` branch only (not on `main`), so it doesn't appear on the rendered profile. Note: a public repo has no truly private files/branches — for real privacy this belongs in a separate **private** repo.

---

## Final structure (top → bottom)

1. **Hero banner** — self-hosted SVG (`assets/banner.svg`)
2. **Contact row** — LinkedIn · email (self-hosted badges) · profile-views (live)
3. **About** — baked icon heading + intro paragraph + a `→` focus list
4. **Tech Stack** — baked icon headings + uniform self-hosted brand icons (6 categories)
5. **Featured Work** — four projects (NEURA, Operational Analytics Platform, Radiology Analytics App, ChatLens), each with a hook line, a bolded description, and self-hosted metric + tech badges
6. **Footer CTA** — one line + a self-hosted LinkedIn "Message me" (DM-compose) button

---

## Components — what they are and why

### Hero banner — `assets/banner.svg`
Hand-authored static SVG: diagonal teal gradient, two soft radial "glow" orbs, dotted accent grids, and the name + tagline as SVG `<text>`.
**Why self-hosted:** banner generators (capsule-render) returned broken SVGs — empty wave paths (`d=""`) and styling locked in a `<style>` block that GitHub's sanitizer strips — so the name vanished. A committed static SVG renders reliably and is fully under our control.

### Contact row
**Self-hosted flat-square** SVG badges (`assets/badges/b-*.svg`):
- **LinkedIn** — logo + word, links to the profile.
- **Email** — shows the address (lowercase), `mailto:` link.
- **Profile Views** — komarev counter, teal. This is the **only** image still fetched at runtime (live data, can't be self-hosted).

**Why self-hosted:** originally `shields.io` badges; later baked into committed SVGs so a shields.io outage can't break the header. **Why flat-square:** the bold `for-the-badge` style force-uppercases text, which made the email render `MAW180604@GMAIL.COM`. flat-square preserves case and keeps the badges consistent.

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
Four full-width project cards (NEURA, Operational Analytics Platform, Radiology Analytics App, ChatLens). Each = a baked heading icon + an italic **hook** line + a description with key metrics in **bold** + a row of **self-hosted metric badges** + a compact row of **self-hosted tech badges** + a one-line note (🔒 academic / 💼 professional).
All badges are committed SVGs under `assets/badges/`: two-segment **metric** badges (gray label + teal value) and single-segment **tech** badges (brand color + embedded white/black logo). Logos are embedded from simple-icons; every text run uses SVG `textLength` + `lengthAdjust="spacingAndGlyphs"` so it can't clip regardless of the viewer's font.

### Footer CTA
One centered line + a **self-hosted** `for-the-badge`-style LinkedIn button (`assets/badges/b-cta.svg`; 28px tall, uppercase, tracked) linking to the DM-compose URL
(`https://www.linkedin.com/messaging/compose/?recipient=muhammadabdullahwasim`).

---

## Key technical lessons

- **Prefer self-hosted SVGs over third-party generators.** Observed during the build: capsule-render returned broken SVGs; the Heroku streak widget and github-profile-trophy were down/limited (trophy returned HTTP 402); the main github-readme-stats card was rate-limited ("Something went wrong"). Reliable building blocks: **shields.io**, **self-hosted SVG**, and (while they were used) github-readme-activity-graph and readme-typing-svg.
- **GitHub sanitizes README HTML:** strips `<style>`/CSS, ignores `align` for vertical centering, and borders tables. This is what drove the baked-image heading approach and the self-hosted icons.
- **Removed earlier experiments:** typing animation and the contribution-activity graph (duplicated GitHub's native graph), the streak widget and trophies (unreliable).
- **Contribution-graph hygiene:** iteration history was squashed; design options were explored on non-default branches (commits there don't count toward the contribution graph).
- **Self-host badges too, not just icons.** Every `shields.io` badge (project metrics, tech, header pills, CTA) is a committed SVG under `assets/badges/`, so a shields.io outage can't break rendering. Only the live komarev view counter stays external.
- **Font-independent badge text.** Self-hosted badges pin each text run with `textLength` + `lengthAdjust="spacingAndGlyphs"`, because the viewer's fallback font (Arial/DejaVu, wider than Verdana) was overflowing/clipping the CTA.
- **Contribution-graph phantoms need a repo re-create, not just a squash.** Rewriting `main` left ~20 orphaned June-18 commits in the object store that GitHub kept counting; deleting and re-creating the repo (pushing only the clean history) gave a fresh object store and dropped that day to its true count. The rendered graph is CDN-cached and can lag the corrected value (verify the truth via the GraphQL `contributionsCollection` API, not the cached page).

---

## Asset inventory

| Path | What |
|------|------|
| `assets/banner.svg` | Hero banner (name + tagline) |
| `assets/icons/h-*.svg` + `h-*-d.svg` | 13 baked section/project headings × 2 (light/dark) |
| `assets/icons/<tool>.svg` | 40 normalized brand icons + 3 hand-drawn glyphs |
| `assets/badges/m-*.svg` | 12 self-hosted metric badges (two-segment) |
| `assets/badges/t-*.svg` | 14 self-hosted tech badges (brand color + embedded logo) |
| `assets/badges/b-*.svg` | 3 self-hosted header/CTA badges (LinkedIn, email, DM button) |

---

## Maintaining it

- **Add a tool:** drop a normalized 56×56 SVG into `assets/icons/` and add an `<img src="assets/icons/<tool>.svg" width="46" height="46" title="<Name>" alt="<Name>"/>` to the right category cell.
- **Change a heading (text/icon/color):** regenerate that heading's two SVGs (light + dark) — they bake the icon + text together; the README references them via `<picture>`.
- **Brand icons** were fetched from simple-icons (jsdelivr) / devicon and normalized to the 56×56 canvas by a Python script; the heading icons were generated by a separate Python script (icon paths + text baked in, light/dark variants).
- **Keep colors mid-tone** so they stay readable in both light and dark mode.
- **Add/change a badge:** badges are committed SVGs in `assets/badges/`, generated by a Python script (Verdana text metrics via Pillow; simple-icons logos embedded and tinted; text pinned with `textLength`). Regenerate the affected `m-*`/`t-*`/`b-*` SVG rather than hand-editing.

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
9. Recreated the repo (delete + re-push clean history) to purge ~20 orphaned June-18 commits from the contribution graph; earned the **Quickdraw** and **YOLO** achievements.
10. Expanded **Featured Work** to four projects (added Radiology Analytics App and ChatLens with new baked heading icons); rewrote copy with a hook line + bolded metrics; unified the professional-setting note.
11. **Self-hosted every badge** (project metrics, tech logos, header pills, CTA) under `assets/badges/`, embedding simple-icons logos and hardening text with `textLength`; removed em dashes.
12. Squash-merged the Featured Work work into `main` as a single commit.
