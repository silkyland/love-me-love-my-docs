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
├── images/<chapter>/NN-slug.png      # GUI captures (web/desktop/mobile)
└── transcripts/<chapter>/NN-slug.txt # CLI captures — real command output
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
   **You should see:** <the expected result, in one line>

2. …

## Result

<What success looks like — matches the final capture.>

## If something goes wrong

- **<Symptom>** — <cause in plain words> → <what to do>
```

Every step carries an expected result — "**You should see:**" for GUI
steps, the exit code plus key output lines for CLI steps. A step without
an observable result is a step the user can't verify; cut it or capture
it properly.

## CLI chapter variant

For command-line programs the capture is the command itself — paste the
real transcript, never typed-from-memory output:

````markdown
## Steps

1. Install the demo profile:

   ```console
   $ mycli init --demo
   Created demo workspace at ~/.mycli/demo
   # exit: 0
   ```

   *You should see the workspace path echoed back.*

2. Create a draft article:

   ```console
   $ mycli articles create --title "How to Grow Orchids"
   Created article 42: How to Grow Orchids
   # exit: 0
   ```

   *Note the new ID — you'll need it to publish.*
````

Rules for transcript blocks: show the `$ command` exactly as the reader
types it, paste the captured output verbatim (elisions marked `…`, never
hiding an error), and keep `# exit: N` when the exit code matters —
expected failures are documented *with* their non-zero exit.

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
- CLI manuals: name commands exactly as the binary exposes them
  (`mycli articles create`, not "create an article"), show real flags,
  and keep output verbatim — if the output is ugly, that's what users
  see; fix it with flags/env in the harness, never by editing the text.

## Reader-facing content vs agent scaffolding

The manual documents the **product**; `docs/capture/` + the regeneration
README document the **photoshoot**. Keep them separate:

| Belongs in the manual | Belongs in `docs/capture/` / regeneration README |
|-----------------------|--------------------------------------------------|
| How to install and first-run the product — *when the product itself requires it* | Building from source, dev-server boot, ports |
| Concepts and task flows with expected results | Seed scripts, demo accounts, `auth.json` re-mint |
| Real-user troubleshooting | The production gate, capture environment pins |

A reader must never be told to run `docs/capture/seed.py`, re-mint
`auth.json`, or start a dev server they don't own. If the product is
self-hosted software, install instructions ARE manual content — write
them as a user would follow them, not as the agent executed them.

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

The manual's own README states the full regeneration chain for its
platform: install/build the app, the seed command, the session re-mint
command where the platform has one (`auth.json` and friends expire), the
capture command, and "if you change the UI or the CLI's output, re-run
capture before merging." Recommend a CI job running the harness — a red
build is a manual that would have lied.
