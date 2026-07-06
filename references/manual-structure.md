# Manual Structure & Rendering

The anatomy of a manual people actually read, and how to make it
beautiful with near-zero design effort.

## Manual anatomy

```
docs/manual/
├── index.md                  # What this product does, who this manual is for,
│                             # "start here" pointing to the 3 most common tasks
├── getting-started.md        # Account, login, the one first success moment
├── <flow-chapter>.md         # One per core flow from the census
├── faq-troubleshooting.md    # Collected from chapters' troubleshooting notes
└── images/<chapter>/NN-slug.png
```

## Chapter template

```markdown
# <Doing the thing — verb phrase users say, e.g. "Publishing an article">

**Who:** <role>   **You'll need:** <prerequisites>   **Takes:** ~N minutes

<One sentence: what you'll have when done.>

## Steps

1. <Action in user words — "Click **New Article** in the top bar">
   ![<caption>](images/publish-article/01-home.png)
   *<What to notice in this screen — one line.>*

2. …

## Result

<What success looks like — matches the final screenshot.>

## If something goes wrong

- **<Symptom>** — <cause in plain words> → <what to do>
```

## Writing rules

- **User language, never developer language:** "click **Publish**", not
  "submit the form / trigger the endpoint". Name UI elements exactly as
  the UI labels them (bold), in the manual's language.
- One action per step. A step with "and" in it is usually two steps.
- Captions say what to *notice*, not what the image is ("Your draft is
  saved automatically — see the badge top-right", not "Screenshot of the
  editor").
- Present tense, second person, no passive voice, no apologies.
- Troubleshooting entries come from real failure modes found during
  capture runs and the flow census — not invented hypotheticals.

## Rendering

**MkDocs Material (recommended default)** — beautiful, searchable,
dark-mode, zero design work:

```bash
pip install mkdocs-material
mkdocs serve   # preview
mkdocs build   # static site
```

```yaml
# mkdocs.yml
site_name: <Product> User Manual
docs_dir: docs/manual
theme:
  name: material
  palette:
    primary: custom        # map to the product's brand color
  features: [navigation.sections, navigation.footer, search.suggest, content.action.edit]
```

Pull the brand color from the product's actual CSS (cite the source) so
the manual wears the product's identity.

**Plain Markdown** — when docs live in the repo/wiki: the same files
without mkdocs.yml; keep relative image paths.

**PDF** — when someone insists on a file: render via
`mkdocs-with-pdf` plugin or Pandoc/WeasyPrint from the same Markdown.
One source, three formats — never fork the content per format.

## Freshness contract

The manual's own README states: the capture command, the seed command,
and "if you change the UI, re-run capture before merging." Recommend a CI
job running the harness — a red build is a manual that would have lied.
