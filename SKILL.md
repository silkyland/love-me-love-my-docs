---
name: love-me-love-my-docs
description: >-
  Auto-generates a user manual with REAL evidence captured by executable
  scripts: censuses the app's user flows from routes, screens, or the
  command surface, picks the capture mechanism the platform actually has
  (browser automation for web, OS-level capture for desktop, Maestro or
  similar for mobile, real command runs for CLI), seeds safe demo data so
  no real user data appears, writes the manual chapter-per-flow with
  numbered steps and expected results, and renders it beautifully (MkDocs
  Material site, Markdown, or PDF). The capture scripts are committed, so
  the manual regenerates when the app changes instead of rotting; if the
  app cannot run in this environment, the skill reports exactly which
  parts stay unverified instead of faking captures. Use when the user
  asks to generate a user manual, user guide, documentation with
  screenshots or terminal output, onboarding docs, or mentions
  love-me-love-my-docs or /love-me-love-my-docs.
license: MIT
argument-hint: "[project-root] [platform: web/desktop/mobile/cli] [output: mkdocs/markdown/pdf] [language]"
---

# Love Me, Love My Docs

Manuals rot for one reason: screenshots are pasted, not generated. The day
the UI changes, every image lies. This skill makes captures a build
artifact — a committed capture script walks the real app and records every
step (screenshots on GUI platforms, real command output for CLI) — so
regenerating the whole manual is one command, forever.

## The Prime Directive (family rule)

> **Every image and every terminal transcript is reproducible, and every
> documented step was actually performed.** Captures come from the
> committed harness in `docs/capture/`, never from hand-cropped one-offs,
> generated mockups, or output imagined from memory. Steps come from flows
> the harness successfully ran — a step that couldn't be executed is a
> finding about the app, not a thing to paper over with prose.

**Scope boundary:** this skill documents the app as it is. Installing,
building, seeding, and starting it are preparation you perform — but if
the app is broken, that is a reported finding, never a task. Do not fix
the app.

## Progress checklist

Copy this into your response and check items off:

```
Docs Progress:
- [ ] Step 1: Frame — audience, language(s), platform, output format
- [ ] Step 2: Flow census — user journeys mined from routes/screens/commands; Chapter Plan Gate passed
- [ ] Step 3: Demo data — target passed the production gate; safe seeded account; zero real user data
- [ ] Step 4: Smoke capture — ONE real artifact end-to-end (boot → session → seed visible → shot/transcript → rendered page)
- [ ] Step 5: Capture harness — the mechanism the platform actually has (web/desktop/mobile/CLI)
- [ ] Step 6: Capture run — captures generated, consistent and complete
- [ ] Step 7: Manual written — chapter per flow, step per capture, expected results, rendered beautifully
- [ ] Step 8: Verify + report — no broken refs, rot pre-mortem passed, regeneration command documented, unverified parts listed
```

## Step 1 — Frame

- **Audience:** end users / admins / both — separate manuals if both
  (mixed audiences make unusable docs).
- **Language(s):** write in the product's language first; if multiple,
  captures may need per-locale runs (the harness parameterizes locale).
- **Platform:** identify what the program IS from evidence — web app,
  desktop app, mobile app, or CLI — and confirm the pick in the frame
  questions; never assume web. Each platform has its own capture mechanism
  (Step 5): browser automation for web, OS-level capture for desktop,
  Maestro-style device automation for mobile, real command runs for CLI.
  A program with no user-facing surface (pure library, API-only) has
  nothing honest to capture — report that and propose the closest
  verifiable artifact instead of forcing a manual shape.
- **Output:** MkDocs Material site (recommended — beautiful by default,
  searchable), plain Markdown in `docs/manual/`, or PDF. One choice.
  CLI-heavy manuals lean Markdown: readers copy-paste commands.

## Step 2 — Flow census

Mine the user journeys from evidence, not memory — the surface differs
per platform: routes and navigation menus for web, windows/screens and
menus for desktop and mobile, the command tree (`--help`, subcommands,
parser definitions) for CLI (`file:line` each). A flow that exists in code
but can't be executed is itself a finding. Classify:

- **Core flows** — what 80% of users do (login, create X, publish, search)
  → each becomes a chapter.
- **Secondary flows** — settings, profile, exports → grouped chapters.
- **Admin flows** — separate manual or section, per Step 1.
- **Skip** — internal/debug routes, with a note.

**Chapter Plan Gate** — before writing a single capture script, present a
compact brief in chat: one numbered line per chapter (flow, route/nav
or command evidence `file:line`, screens/commands touched, capture count
estimate) plus the
skip list. 10–20 lines total; ask for confirmation **once**. Changing the
chapter list here costs one message; changing it after the harness exists
costs a rewrite. No harness code may exist before this gate passes. If the
user cannot respond (headless/CI run), proceed and mark the chapter plan
`UNCONFIRMED` in the final report.

## Step 3 — Demo data hygiene

Screenshots and transcripts outlive databases. Before any capture — and
only when the app actually has state (a stateless CLI needs none; say so
rather than inventing seed data):

- A dedicated demo account (name like "Somchai Demo", not a real person)
  and seeded content that looks real but is fictional. If the repo
  already uses seed-ah or the user asks for it, that is the family's
  demo-data provider — its ledger keeps every seeded row wipeable.
- **Zero real user data, emails, tokens, or keys in any frame or
  transcript** — command output leaks secrets just as surely as a
  screenshot does. Treat every capture as public forever.
- Consistent state: the same seed produces the same screens and the same
  command output, so re-runs diff cleanly. Seed script lives next to the
  capture script in `docs/capture/` — it is agent scaffolding, never
  reader-facing manual content (Step 7).
- **Production gate (mechanical):** seeding writes rows and the harness
  clicks real buttons ("Publish", "Delete") — pointed at production, it
  mutates production. That is a ONE-WAY action. Before seeding one row or
  scripting one click, check the target base URL host: `localhost`,
  `127.0.0.1`, or a `.local`/`.test` domain passes automatically; **any
  other host requires the user to confirm it by name, once**. No
  confirmation available (headless run) = do not seed, do not capture —
  fall back to honesty mode in Step 5.

## Step 4 — Smoke capture (one real artifact end-to-end)

Before writing per-flow scripts, prove the thinnest slice works. This is
where every operational unknown lives — install/build, app boot, auth,
seed visibility, capture permissions — and finding them on flow 1 of 1 is
cheap; finding them on flow 12 of 12 is a rewrite:

1. Get the app running by its documented path (install → build → start;
   record the exact commands — they are preparation, kept out of
   reader-facing content per Step 7). If it won't run in this
   environment, stop here: that is a named finding, and the run continues
   in honesty mode (Step 5).
2. Establish the session or initial state the manual assumes (web: log
   the demo account in once, save storage state to `docs/capture/auth.json`;
   CLI: a clean pinned shell/env; desktop: launch to a known window;
   mobile: launch to a known screen with cleared state —
   `launchApp: clearState: true` — and the demo account logged in).
3. Verify one seeded entity or one real command works end-to-end.
4. Capture ONE artifact through the harness skeleton and render ONE
   chapter page that references it, in the chosen output format.

**Exit criterion (mechanical):** one real artifact exists on disk — an
image file or a command transcript — and one rendered chapter page
displays it. "Real" means produced by running the app just now; a mockup,
a remembered output, or a generated image does not count. Until then, no
second capture script may be written. Any numbered item that fails is a
named finding (app won't boot, auth broken, seed invisible, capture
permission denied) — report it; do not script around it.

## Step 5 — Build the capture harness

Pick the mechanism the platform actually has — never force one tool onto
every program:

- **Web:** Playwright for Python — patterns in
  [references/capture-web.md](references/capture-web.md): stored auth
  state, fixed viewport/theme, wait-for-stable strategies, element
  highlighting before the shot, per-locale parameterization.
- **Desktop:** OS-level capture — `screencapture` (macOS),
  `gnome-screenshot`/`import`/`maim` (Linux), .NET screen capture via
  PowerShell (Windows); GUI automation via AppleScript/`cliclick`,
  `xdotool`, or cross-platform `pyautogui`/accessibility APIs. Electron
  apps can often reuse the web harness through Playwright's Electron
  bindings — verify against the installed version. The window must be
  frontmost, fixed-size, and unobstructed: a screenshot of a buried
  window documents nothing. Patterns in
  [references/capture-desktop.md](references/capture-desktop.md).
- **Mobile:** Maestro YAML flows with `takeScreenshot` — patterns,
  alternatives (Fastlane snapshot/screengrab, raw simctl/adb), and the
  install-security rule (auditable channels only — **never
  `curl | bash`**) in
  [references/capture-mobile.md](references/capture-mobile.md).
- **CLI:** there is no screenshot — the capture IS the command. A
  committed script runs each documented command in a pinned environment
  (terminal width, `TERM`, locale, clean env) and saves stdout, stderr,
  and the exit code as a transcript. Patterns, interactive-prompt
  handling, and output-scrubbing rules in
  [references/capture-cli.md](references/capture-cli.md).
- **Locator policy where the platform has locators** (web, mobile,
  desktop): committed scripts use stable locators — `get_by_test_id` /
  `get_by_role` on web, accessibility labels/ids on mobile and desktop.
  Every bare text or CSS selector that survives is counted and listed in
  the final report as an app finding (missing `data-testid`s or
  accessibility labels). CLI has no selectors; its rot vector is output
  drift — covered in Step 8.
- One script/flow/command-set per chapter; artifacts named
  `<chapter>/<step-number>-<slug>.<ext>` — the filename IS the step
  order (`.png` for images, `.txt`/`.log` for transcripts).
- Commit the harness to `docs/capture/`. It is product code now.
- **Honesty mode — the app cannot run here.** If the app cannot be
  executed in this environment (no display, missing toolchain, app won't
  boot, platform tool not installable), never fake a capture: still
  commit the harness and seed scripts (they encode the steps), emit a
  capture checklist with the exact artifacts to produce, mark every
  unfilled slot in the manual with a visible TODO placeholder, and the
  Step 8 report lists exactly which flows remain unverified. The manual
  is a draft, not a finished manual — say so.

## Step 6 — Capture run

Run the harness. Every failure is triaged, not skipped: a step that can't
be automated is either a missing stable locator in the app (a finding —
report it) or a flow/command that changed since the census (update the
census). On CLI, a documented command that exits non-zero or can't run is
the same kind of finding. Re-run until the set is complete and consistent
— same pinned environment per set: viewport/theme/locale on web, device
profile/locale on mobile, window size/theme on desktop, terminal
width/`TERM`/locale on CLI.

## Step 7 — Write the manual

Structure and style per
[references/manual-structure.md](references/manual-structure.md): chapter
per flow — goal, prerequisites, numbered steps (one capture each, with a
caption saying what to notice and what the user should see), expected
result, troubleshooting. Written in user language ("click **Publish**",
`git commit -m "…"`), never developer language ("trigger the POST
endpoint"). Render the chosen format; MkDocs Material config included
when that's the choice.

**Reader-facing content vs agent scaffolding.** The manual documents the
product for its users: how to install and first-run it *when the product
itself requires that*, concepts, task flows, expected results,
troubleshooting. Everything the agent did to make captures possible —
building from source, seeding demo data, minting auth state, dev-server
ports, the production gate — is scaffolding: it lives in `docs/capture/`
and the regeneration README, never as reader-facing steps. A reader must
never be told to run `docs/capture/seed.py` or re-mint `auth.json`.

## Step 8 — Verify and report

- Every image referenced exists; every transcript block matches a
  captured file; every capture script runs green end-to-end; the
  regeneration command is documented in the manual's own README
  (`python docs/capture/run_all.py`, `maestro test flows/`, or the CLI
  equivalent).
- **Rot pre-mortem** — assume the manual rotted three months from now;
  the known causes are checked mechanically, not pondered:
  - **Locators:** grep committed scripts for bare text selectors
    (`:has-text`, `text=`) and CSS selectors (`#`, `.`, `[`); count must
    be zero or each one listed as an app finding with `file:line`.
    (Web/mobile/desktop only — CLI's rot vector is the next bullet.)
  - **Command drift (CLI):** flags, subcommands, and output formats
    change — the harness re-runs real commands so drift fails loudly;
    embedded expected-output blocks must be regenerated from fresh
    transcripts, never hand-edited.
  - **Auth/session:** the `auth.json` (or platform equivalent) re-mint
    procedure is documented in the manual's README — stored session
    state expires.
  - **Seed drift:** the seed script is committed next to the harness and
    `run_all` invokes it (or its README documents the seed command as
    step one).
  - **Environment:** base URL, viewport, theme, locale, window size,
    terminal width/`TERM` — whichever the platform pins — are constants
    in one place, never repeated per script.
- Report: chapters written, captures generated, flows that failed
  automation (app findings), the one-command regeneration story — and,
  in honesty mode, exactly which flows/screens/commands remain
  unverified and why. Never report the manual "complete" while any
  image slot or transcript block is a TODO.
- Offer the standing suggestion: wire the capture run into CI so changes
  that break the manual fail loudly instead of rotting silently.

## When things go wrong

| Situation | Response |
|-----------|----------|
| App won't boot, won't install, or can't run in this environment | Named finding in Step 4 — report it; this skill documents the app, it does not fix it. Honesty mode (Step 5): commit harness + seed skeletons, emit a capture checklist with exact artifacts to produce, mark unfilled slots TODO, list unverified parts in the report. Never claim the manual complete. |
| Capture tool not installable or no runnable target available | Same honesty mode — per-platform (Playwright, Maestro, OS capture tools, or the CLI runner itself). |
| App is broken — a bug surfaces during census or capture | Report it as an app finding with repro evidence; do not fix — out of scope. The affected flow's manual content is marked unverified. |
| Capture fails on a specific flow mid-run | Triage (Step 6): missing stable locator in the app (finding — report it) or flow/command changed since census (update census). CLI: a documented command that exits non-zero is the same triage. Never skip; re-run until complete or drop to honesty mode. |
| Stored session state (`auth.json` or platform equivalent) expires during regeneration | Re-mint procedure documented in manual's README (Step 8 rot pre-mortem). Log the demo account in manually once, save the state, document the exact command. |
| Captures inconsistent (viewport/theme/locale/device/window/terminal) | Environment constants not pinned (Step 8 rot pre-mortem). Pin the platform's constants in one place; never repeat per script. |
| CLI output contains secrets, home paths, timestamps, or random IDs | Scrub through documented harness normalization rules ([references/capture-cli.md](references/capture-cli.md)); every normalization is listed in the report. A secret that can't be scrubbed without changing the output's meaning is an app finding. |
| Manual regeneration command undocumented or fails | Step 8 exit criterion failed. Document the full chain in the manual's README: install/build, seed, session re-mint if the platform has one, run capture. Test end-to-end before reporting complete. |
