---
name: love-me-love-my-docs
description: >-
  Auto-generates a beautiful user manual with REAL screenshots captured by
  executable scripts: censuses the app's user flows from routes and
  navigation, builds a Playwright (Python) capture harness for web or a
  Maestro flow for mobile (iOS/Android), seeds safe demo data so no real
  user data appears in images, writes the manual chapter-per-flow with a
  numbered step per screenshot, and renders it beautifully (MkDocs Material
  site, Markdown, or PDF). The capture scripts are committed, so the manual
  regenerates when the UI changes instead of rotting. Use when the user
  asks to generate a user manual, user guide, documentation with
  screenshots, onboarding docs, or mentions love-me-love-my-docs or
  /love-me-love-my-docs.
license: MIT
argument-hint: "[project-root] [platform: web/mobile] [output: mkdocs/markdown/pdf] [language]"
---

# Love Me, Love My Docs

Manuals rot for one reason: screenshots are pasted, not generated. The day
the UI changes, every image lies. This skill makes screenshots a build
artifact — a committed capture script walks the real app and shoots every
step — so regenerating the whole manual is one command, forever.

## The Prime Directive (family rule)

> **Every screenshot is reproducible and every documented step was
> actually performed.** Images come from the capture script (committed to
> the repo), never from hand-cropped one-offs. Steps come from flows the
> script successfully walked — a step the harness couldn't execute is a
> finding about the app, not a thing to paper over with prose.

## Progress checklist

Copy this into your response and check items off:

```
Docs Progress:
- [ ] Step 1: Frame — audience, language(s), platform, output format
- [ ] Step 2: Flow census — user journeys mined from routes/screens, chapters chosen
- [ ] Step 3: Demo data — safe seeded account/content; zero real user data
- [ ] Step 4: Capture harness — Playwright (web) / Maestro (mobile) script per flow
- [ ] Step 5: Capture run — screenshots generated, consistent and complete
- [ ] Step 6: Manual written — chapter per flow, step per screenshot, rendered beautifully
- [ ] Step 7: Verify + report — no broken images, regeneration command documented
```

## Step 1 — Frame

- **Audience:** end users / admins / both — separate manuals if both
  (mixed audiences make unusable docs).
- **Language(s):** write in the product's language first; if multiple,
  captures may need per-locale runs (the harness parameterizes locale).
- **Platform:** web → Playwright; mobile → Maestro (see feasibility notes
  in [references/capture-mobile.md](references/capture-mobile.md));
  both → two harnesses, one manual.
- **Output:** MkDocs Material site (recommended — beautiful by default,
  searchable), plain Markdown in `docs/manual/`, or PDF. One choice.

## Step 2 — Flow census

Mine the user journeys from evidence, not memory: routes, navigation
menus, screen registries (`file:line` each). Classify:

- **Core flows** — what 80% of users do (login, create X, publish, search)
  → each becomes a chapter.
- **Secondary flows** — settings, profile, exports → grouped chapters.
- **Admin flows** — separate manual or section, per Step 1.
- **Skip** — internal/debug routes, with a note.

Present the chapter plan (one line per chapter: flow, screens touched,
screenshot count estimate) for a quick user OK before building the harness.

## Step 3 — Demo data hygiene

Screenshots outlive databases. Before any capture:

- A dedicated demo account (name like "Somchai Demo", not a real person)
  and seeded content that looks real but is fictional.
- **Zero real user data, emails, tokens, or keys in any frame** — treat
  every screenshot as public forever.
- Consistent state: the same seed produces the same screens, so re-runs
  diff cleanly. Seed script lives next to the capture script.

## Step 4 — Build the capture harness

- **Web:** Playwright for Python — patterns in
  [references/capture-web.md](references/capture-web.md): stored auth
  state, fixed viewport/theme, wait-for-stable strategies, element
  highlighting before the shot, per-locale parameterization.
- **Mobile:** Maestro YAML flows with `takeScreenshot` — patterns and
  alternatives (Fastlane snapshot/screengrab, raw simctl/adb) in
  [references/capture-mobile.md](references/capture-mobile.md). No
  simulator/emulator available = degraded mode: generate the flow files +
  a manual capture checklist, and say so honestly.
- One script/flow per chapter; screenshots named
  `<chapter>/<step-number>-<slug>.png` — the filename IS the step order.
- Commit the harness to `docs/capture/`. It is product code now.

## Step 5 — Capture run

Run the harness. Every failure is triaged, not skipped: a step that can't
be automated is either a missing test-id/accessibility-label in the app (a
finding — report it) or a flow that changed since the census (update the
census). Re-run until the set is complete and consistent (same viewport,
same theme, same locale per set).

## Step 6 — Write the manual

Structure and style per
[references/manual-structure.md](references/manual-structure.md): chapter
per flow — goal, prerequisites, numbered steps (one screenshot each, with
a caption saying what to notice), expected result, troubleshooting.
Written in user language ("click **Publish**"), never developer language
("trigger the POST endpoint"). Render the chosen format; MkDocs Material
config included when that's the choice.

## Step 7 — Verify and report

- Every image referenced exists; every capture script runs green
  end-to-end; the regeneration command is documented in the manual's own
  README (`python docs/capture/run_all.py` or `maestro test flows/`).
- Report: chapters written, screenshots generated, flows that failed
  automation (app findings), and the one-command regeneration story.
- Offer the standing suggestion: wire the capture run into CI so UI
  changes that break the manual fail loudly instead of rotting silently.
