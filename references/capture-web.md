# Web Capture Harness — Playwright (Python)

Patterns for manual-grade screenshots: consistent, annotated, re-runnable.

## Setup

```bash
pip install playwright
playwright install chromium
```

## Harness skeleton

```python
# docs/capture/run_all.py
from playwright.sync_api import sync_playwright

BASE = "http://localhost:8000"   # seeded demo instance — production gate:
                                 # host must be localhost/127.0.0.1/.local/.test,
                                 # anything else needs explicit user confirmation
VIEWPORT = {"width": 1280, "height": 800}  # ONE viewport for the whole manual

def shot(page, chapter, n, slug):
    page.wait_for_load_state("networkidle")
    page.screenshot(path=f"docs/manual/images/{chapter}/{n:02d}-{slug}.png")

with sync_playwright() as p:
    browser = p.chromium.launch()
    ctx = browser.new_context(
        viewport=VIEWPORT,
        locale="th-TH",                      # parameterize per manual language
        storage_state="docs/capture/auth.json",  # pre-saved demo login
        color_scheme="light",                # pin the theme
    )
    page = ctx.new_page()

    # Chapter: publish-article
    page.goto(f"{BASE}/articles/new")
    shot(page, "publish-article", 1, "empty-form")
    page.get_by_test_id("article-title").fill("How to Grow Orchids")
    shot(page, "publish-article", 2, "filled-form")
    page.get_by_role("button", name="Publish").click()
    shot(page, "publish-article", 3, "success")
```

## Rules

- **Auth once:** log the demo account in manually one time, save
  `context.storage_state(path="auth.json")`, reuse it — never automate
  real credentials into the script. Stored auth state **expires** — the
  most common way a capture harness dies at month three. Document the
  re-mint procedure (the login-and-save command) in the manual's README
  next to the regeneration command.
- **Selector policy:** committed scripts use `get_by_test_id` and
  `get_by_role` (with the accessible name). A bare text or CSS selector
  (`button:has-text('Publish')`, `#title`) is a rot vector — the first
  copy change or refactor silently breaks capture. Every one that
  survives into a committed script is counted and reported as an app
  finding: the page lacks `data-testid`s or accessible roles.
- **Stability:** `wait_for_load_state("networkidle")` plus explicit
  `expect(locator).to_be_visible()` on the element the step talks about —
  a screenshot of a spinner documents nothing.
- **Determinism killers to neutralize:** dates ("2 minutes ago"),
  animated banners, random avatars. Freeze via seeded data, `page.clock`
  (Playwright's clock API), or a small CSS injection disabling animations:
  `page.add_style_tag(content="*{animation:none!important;transition:none!important}")`.
- **Highlight the target** before the shot so the reader's eye lands where
  the caption points:

```python
def highlight(page, selector):
    page.eval_on_selector(selector, """el => {
        el.style.outline = '3px solid #FF5A5F';
        el.style.outlineOffset = '2px';
    }""")
```

  (Shoot, then remove the outline or reload — never leave it for the next
  step's shot.)
- **Element vs full page:** default full viewport for orientation shots;
  `locator.screenshot()` for dense forms where the reader needs zoom.
- **Locales:** wrap the chapter walk in a loop over locales; output to
  `images/<locale>/<chapter>/...` — one harness, N language manuals.
- **Failures are findings:** a step that can't be reached by selector
  usually means the app lacks stable selectors (`data-testid`) — report
  it; adding test-ids helps QA, not just docs.

## Regeneration contract

`python docs/capture/run_all.py` from a clean seeded state regenerates
every image. The manual README documents the full chain — boot the app,
run the seed script, re-mint `auth.json` if expired, then run capture —
so regeneration is copy-paste, not archaeology. Recommend a CI job so a
UI change that breaks capture fails the build instead of silently
rotting the manual.
