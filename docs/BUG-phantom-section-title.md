# BUG: Phantom "毫秒级反馈：静态分析与代码规范" text on every PDF page

## Problem

When generating PDFs with `npx honkit pdf`, the text "毫秒级反馈：静态分析与代码规范" (the title of `zh/chapters/05c1-linters.md`) appears as grey text at the top of **every single page** in the Chinese PDF — including the cover, table of contents, and all chapter pages. It appears above the actual content, in a lighter/grey color.

This is the title of one specific sub-chapter, but it renders on ALL pages, not just that chapter's page.

## Environment

- HonKit 5.1.5 (`npx honkit pdf`)
- Calibre ebook-convert from `/Applications/calibre.app/Contents/MacOS/ebook-convert`
- macOS Darwin 25.2.0
- Multilingual book (zh/en/ja) configured via `LANGS.md`

## How to reproduce

```bash
npm install
export PATH="/Applications/calibre.app/Contents/MacOS:$PATH"
npx honkit pdf ./ ./output.pdf
open output_zh.pdf   # observe grey text on every page
```

## Root cause investigation

The text has two potential sources. Neither fix attempt eliminated it.

### Source A: Calibre `_SECTION_` magic variable in header template

HonKit's `getPDFTemplate.js` sets `context.page.title = "_SECTION_"`. The default `pdf_header.html` template renders `{{ page.title }}` which outputs the literal string `_SECTION_`. Calibre replaces `_SECTION_` with the current chapter's title as a running header.

**File:** `node_modules/honkit/lib/output/ebook/getPDFTemplate.js` line 26

### Source B: `<h1 class="book-chapter">` in HTML body

HonKit's ebook page template (`node_modules/@honkit/honkit-plugin-theme-default/_layouts/ebook/page.html` line 30) inserts:

```html
<h1 class="book-chapter book-chapter-{{ page.depth }}">{{ page.title }}</h1>
```

This is meant to be hidden (`display:none` in `pdf.css`), used only as a marker for Calibre's chapter detection XPath:

```js
// getConvertOptions.js line 36
"--chapter": "descendant-or-self::*[contains(concat(' ', normalize-space(@class), ' '), ' book-chapter ')]"
```

Calibre's `DetectStructure.detect_chapters` finds `.book-chapter` elements and injects `display:block !important` to override the `display:none`, making the hidden `<h1>` visible.

**Files:**
- Template: `node_modules/@honkit/honkit-plugin-theme-default/_layouts/ebook/page.html:30`
- CSS hide: `node_modules/@honkit/honkit-plugin-theme-default/_assets/ebook/pdf.css` → `.page .book-chapter{display:none}`
- XPath: `node_modules/honkit/lib/output/ebook/getConvertOptions.js:36`
- Calibre override: injected `display:block !important; page-break-before: always !important`

## What was tried (all failed)

### Attempt 1: Override header/footer templates

Created `_layouts/ebook/pdf_header.html` with empty content and `_layouts/ebook/pdf_footer.html` with only page number.

**Result:** No effect. The text is not coming from the header template (or the template override isn't being picked up for this specific text).

### Attempt 2: Patch `_SECTION_` in getPDFTemplate.js

Changed `context.page.title` from `"_SECTION_"` to `"Agents特区 · agentszone.ai"` in `node_modules/honkit/lib/output/ebook/getPDFTemplate.js`.

**Result:** No effect. The phantom text remained unchanged, suggesting it's not from Source A.

### Attempt 3: Override ebook page template to remove `<h1 class="book-chapter">`

Created `_layouts/ebook/page.html` — a complete standalone HTML template (no `{% extends %}`) that omits the `<h1 class="book-chapter">` element entirely.

**Result:** No effect. The phantom text still appears. This suggests either:
1. The custom `_layouts/ebook/page.html` is NOT being picked up by HonKit's ebook output generator (it may only check the theme plugin's `_layouts/`, not the book root's `_layouts/`)
2. There is a third source of this text we haven't identified
3. The generated HTML is cached somewhere and not regenerating

### Attempt 4: Set `headerTemplate`/`footerTemplate` in book.json

Added `"pdf": { "headerTemplate": " ", "footerTemplate": " " }` to `book.json`.

**Result:** No effect. These are not valid/documented config options in HonKit's config schema.

## Likely next steps

1. **Verify template override is actually loading**: Run `npx honkit build` and inspect the generated HTML in `_book/zh/` to see if `<h1 class="book-chapter">` is still present. If yes, the `_layouts/ebook/page.html` override is not being picked up.

2. **Try patching the theme's actual template directly**: Instead of using `_layouts/` override, edit `node_modules/@honkit/honkit-plugin-theme-default/_layouts/ebook/page.html` directly to remove line 30.

3. **Try setting `--chapter-mark` to `"none"`**: In `node_modules/honkit/lib/output/ebook/getConvertOptions.js`, change `"--chapter-mark": String(pdfOptions.chapterMark)` to `"--chapter-mark": "none"`. This would prevent Calibre from injecting `display:block !important`.

4. **Inspect the actual intermediate HTML**: Find the temp directory HonKit creates (logged as something like `/tmp/honkit-*`) and look at the HTML being fed to `ebook-convert` to trace the exact origin of the text.

5. **Bypass HonKit entirely**: Use `npx honkit build`, then convert the HTML to PDF using a different tool (e.g., `wkhtmltopdf`, `weasyprint`, or Pandoc) that doesn't have Calibre's `display:block !important` behavior.

## Current state of files

- `_layouts/ebook/page.html` — custom page template (may not be loading)
- `_layouts/ebook/pdf_header.html` — custom header with "Agents特区 · agentszone.ai"
- `_layouts/ebook/pdf_footer.html` — custom footer with "AgentsZone Community · agentszone.ai" + page number
- `node_modules/honkit/lib/output/ebook/getPDFTemplate.js` — reverted to original `_SECTION_`
- `book.json` — clean, no PDF config hacks
