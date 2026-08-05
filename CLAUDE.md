# CLAUDE.md — Learning Roadmap Trackers

Two self-contained, bilingual (Persian ⇄ English) HTML learning trackers. No build system, no dependencies, no framework — each `.html` file contains all CSS, data, and JS. Open directly in a browser.

## Files

- `ai-pm-roadmap-tracker-v1.html` — Project management from zero → managing AI projects. 4 sections: Roadmap (8 levels, 54 items), Case Studies (10 real AI cases), Library (9 books with summaries), Toolbox (14 tools). ~147KB.
- `design-roadmap-tracker-v10.html` — Product design from zero → pro. 3 sections: Roadmap (10 levels), UI Rebuild Lab (33 base64-embedded JPEGs with download/copy buttons), Toolbox (17 tools). ~1.9MB (images).
- `README.md` — repo readme. Intended GitHub repo name: `ai-pm-roadmap`.

## Shared architecture (both files)

Single `<script>` at end of `<body>`, ordered: helpers (`toFa`, `esc`, storage shim) → i18n dicts → data arrays → engine (render functions + init). All state in a flat `done={}` object keyed by item id.

**i18n pattern (the most important thing to understand):**
- Persian is the source of truth, stored in data arrays: `LEVELS` (items with `id`, `kind`, `min`, `t`, `desc`, optional `prompt`, `resources`), plus per-file: `MOCK_STEPS` (design), `CASE_GROUPS`/`BOOK_GROUPS`/`TOOLS` (PM).
- English lives in parallel dicts keyed by id or Persian string: `STR` (UI chrome, per-lang, includes template functions like `itemsOf(a,b)`), `EN_LEVELS`, `EN_ITEMS`, `RES_EN` (resource label → EN), `TOOL_EN`, `CAT_EN`, and (PM file) `CASE_EN`, `BOOK_EN`, `CASE_GROUP_EN`, `BOOK_GROUP_EN`; (design file) `MOCK_EN`, `SKILL_EN`, `TAG_EN`.
- `lang` global (`"fa"`/`"en"`); `applyLang()` sets `dir`/`lang` attrs, rewrites static DOM text by element id, then re-renders everything. `locNum()` converts digits to Persian only in fa mode. Accessors: `S(key)`, `trLevel/trItem/trPrompt/...`.
- **Rule: any content edit must touch BOTH the Persian array and the matching EN dict entry.** A missing EN entry silently falls back to Persian.

**Storage:** `window.storage` async API (Claude artifact environment) with a localStorage shim so it also persists on GitHub Pages / file://. Keys: `pm-roadmap-{progress,theme,lang}` and `design-roadmap-{progress,theme,lang}`. Note: the shim currently exists **only in the PM file**; the design file falls back to session-only outside the artifact environment (add the same shim if that matters).

**Videos/links:** `VIDEOS` (by item id, `{url, fa, en}` label pair) and `TOOL_VIDEOS` (by tool name) are merged into resources at render time — never edit resource arrays to add videos; add to these maps.

## Hard rules

1. **Never invent URLs.** Every YouTube link was verified via `https://www.youtube.com/oembed?url=<URL>&format=json`; articles were fetched or taken verbatim from search results. Verify the same way before adding any link.
2. **design-roadmap-tracker-v10.html contains ~45 lines of 26KB–146KB base64 image data** (inside `MOCK_STEPS`, roughly lines 560–610). Never read/edit those lines wholesale; image translations are keyed by design id, so you never need to touch them.
3. Versioning: copy to a new filename on major changes (`-v11`, `-v2`) rather than overwriting.
4. Keep everything in one file per tracker — no external CSS/JS, no CDN beyond Google Fonts.
5. Persian text uses kasra-style diacritics (e.g. «مدیرِ پروژه») — match the existing register when writing fa content.

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

# 3. Full smoke test (npm i jsdom): load with runScripts:'dangerously',
#    click #langBtn twice, click a [data-toggle], assert no PAGE ERROR
#    and counts (PM file: 8 levels / 54 items / 10 cases / 9 books / 14 tools).
```

## Git

```bash
git init -b main && git add . && git commit -m "..."
gh repo create ai-pm-roadmap --public --source=. --remote=origin --push
```

GitHub Pages (Settings → Pages → main) serves the trackers directly; the localStorage shim keeps progress working there.
