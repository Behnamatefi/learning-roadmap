# CLAUDE.md — Learning Roadmap Trackers

Two self-contained, bilingual (Persian ⇄ English) HTML learning trackers plus a router page. No build system, no dependencies, no framework — each `.html` file contains all CSS, data, and JS. Open directly in a browser.

## Files

- `index.html` — router/landing page. Links to both trackers from a static `ROUTES` array. ~20KB.
- `ai-pm-roadmap-tracker-v1.html` — Project management from zero → managing AI projects. 4 sections: Roadmap (8 levels, 54 items), Case Studies (10 real AI cases), Library (9 books with summaries), Toolbox (14 tools). ~147KB.
- `design-roadmap-tracker-v10.html` — Product design from zero → pro. 3 sections: Roadmap (10 levels), UI Rebuild Lab (33 base64-embedded JPEGs with download/copy buttons), Toolbox (17 tools). ~1.9MB (images).
- `README.md` — repo readme. GitHub repo: `Behnamatefi/learning-roadmap`.

## Shared architecture (all three files)

Single `<script>` at end of `<body>`, ordered: helpers (`toFa`, `esc`, storage shim) → i18n dicts → data arrays → engine (render functions + init). All state in a flat `done={}` object keyed by item id.

**i18n pattern (the most important thing to understand):**
- Persian is the source of truth, stored in data arrays: `LEVELS` (items with `id`, `kind`, `min`, `t`, `desc`, optional `prompt`, `resources`), plus per-file: `MOCK_STEPS` (design), `CASE_GROUPS`/`BOOK_GROUPS`/`TOOLS` (PM).
- English lives in parallel dicts keyed by id or Persian string: `STR` (UI chrome, per-lang, includes template functions like `itemsOf(a,b)`), `EN_LEVELS`, `EN_ITEMS`, `RES_EN` (resource label → EN), `TOOL_EN`, `CAT_EN`, and (PM file) `CASE_EN`, `BOOK_EN`, `CASE_GROUP_EN`, `BOOK_GROUP_EN`; (design file) `MOCK_EN`, `SKILL_EN`, `TAG_EN`.
- `lang` global (`"fa"`/`"en"`); `applyLang()` sets `dir`/`lang` attrs, rewrites static DOM text by element id, then re-renders everything. `locNum()` converts digits to Persian only in fa mode. Accessors: `S(key)`, `trLevel/trItem/trPrompt/...`.
- **Rule: any content edit must touch BOTH the Persian array and the matching EN dict entry.** A missing EN entry silently falls back to Persian.

**Storage:** `window.storage` async API (Claude artifact environment) with a localStorage shim so it also persists on GitHub Pages / file://. All three files carry the shim. `theme`/`lang` use **shared** keys (`roadmap-theme`, `roadmap-lang`) so the choice carries across navigation between the router and either tracker; progress stays per-tracker (`pm-roadmap-progress`, `design-roadmap-progress`).

**Videos/links:** `VIDEOS` (by item id, `{url, fa, en}` label pair) and `TOOL_VIDEOS` (by tool name) are merged into resources at render time — never edit resource arrays to add videos; add to these maps.

## Hard rules

1. **Never invent URLs.** Every YouTube link was verified via `https://www.youtube.com/oembed?url=<URL>&format=json`; articles were fetched or taken verbatim from search results. Verify the same way before adding any link.
2. **design-roadmap-tracker-v10.html contains ~33 lines of 26KB–146KB base64 image data** (inside `MOCK_STEPS`, roughly lines 900–990 — locate the current range with `awk 'length($0)>5000 {print NR}' FILE`). Never read/edit those lines wholesale; image translations are keyed by design id, so you never need to touch them.
3. Versioning: git branches carry major changes now that the repo has history — edit in place rather than copying to `-v11`/`-v2`. (The `-v1`/`-v10` suffixes in the current filenames predate the repo and are kept only so existing links don't break.)
4. Keep everything in one file per page — no external CSS/JS, no CDN beyond Google Fonts.
5. Persian text uses kasra-style diacritics (e.g. «مدیرِ پروژه») — match the existing register when writing fa content.

## Design system ("reading instrument")

Art-directed by `ART-DIRECTION.md`. The token block is **identical in all three files** — change one, port to the other two.

- Near-black canvas, one signal color `--sig` (`#FF4514` dark / `#F03E10` light). Orange is reserved for live/active/progress: focus rings, hero percentage, progress fills, level ring, active-tab marker, eyebrow dot, hovered chevron. **Never** on body text, category chips, or as a default link color; never two orange moments in one viewport.
- The old hue tokens `--violet/--coral/--green/--amber` were kept **by name** but remapped to a neutral tonal ramp, so the ~30 existing call sites still work. `--coral` is the one orange-family category tint.
- `--shadow` is still defined but inert (`0 0 0 0 transparent`) — hairline borders carry elevation.
- `--muted-2` is ~3.6:1 on the dark canvas: **label-only**, never running text.
- **Persian typography:** `--mono` is `'JetBrains Mono','Vazirmatn',…` — JetBrains Mono has no Arabic coverage, so Persian falls through to Vazirmatn by design. All `letter-spacing` and `text-transform:uppercase` live in `html[lang="en"] …` rules; base selectors must stay untracked, because tracking breaks the joined Persian script.
- `html[lang="en"]` also swaps `--display`/`--body` to Space Grotesk, so one selector re-faces the whole page per language.
- The ASCII cursor-trail (`#cursorFx` canvas + IIFE) reads `--sig`/`--sig-ink` via `getComputedStyle`, refreshed from `applyTheme()`. It bails out entirely under `prefers-reduced-motion` or a coarse pointer.

## Verify after any change

```bash
# 1. JS syntax (extract last <script> block):
python3 -c "
import re; src=open('FILE.html',encoding='utf-8').read()
open('/tmp/x.js','w').write(re.findall(r'<script>(.*?)</script>',src,re.S)[-1])"
node --check /tmp/x.js

# 2. Referenced ids exist:
python3 -c "
import re; src=open('FILE.html',encoding='utf-8').read()
js=re.findall(r'<script>(.*?)</script>',src,re.S)[-1]
used=set(re.findall(r'getElementById\(\"([^\"]+)\"\)',js))
defined=set(re.findall(r'id=\"([^\"]+)\"',src))
print('missing:',sorted(used-defined))"

# 3. Every consumed CSS var is declared (a var(--x) with no declaration
#    renders as nothing, silently — this has bitten this repo before):
python3 -c "
import re; src=open('FILE.html',encoding='utf-8').read()
css=re.search(r'<style>(.*?)</style>',src,re.S).group(1)
used={m for m in re.findall(r'var\((--[\w-]+)',css)}
declared={m for m in re.findall(r'(--[\w-]+)\s*:',css)}
print('undeclared:',sorted(used-declared))"

# 4. Full smoke test (npm i jsdom): load with runScripts:'dangerously',
#    click #langBtn twice, click a [data-toggle], assert no PAGE ERROR
#    and counts (PM file: 8 levels / 54 items / 10 cases / 9 books / 14 tools).
```

## Git

Repo is `Behnamatefi/learning-roadmap`, default branch `main`. GitHub Pages
(Settings → Pages → main) serves `index.html` at the repo root; the localStorage
shim keeps progress working there.
