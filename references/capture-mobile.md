# Mobile Capture Harness — Maestro

Use this path when the program under documentation is a mobile app
(iOS/Android). Mobile has a Playwright-equivalent: **Maestro** — one YAML
flow runs on both iOS simulators and Android emulators, with a built-in
`takeScreenshot` command. Sibling paths: [capture-web.md](capture-web.md)
for web apps, [capture-desktop.md](capture-desktop.md) for desktop apps,
[capture-cli.md](capture-cli.md) for command-line programs.

> Verify current syntax against the installed version (`maestro --help`,
> maestro.dev docs) — commands drift.

## Setup

**Security rule: never pipe remote installer scripts into a shell**
(`curl … | bash` executes unreviewed remote code — the same standard this
skill family applies to screenshots applies to installers).

Install via an auditable channel instead:

```bash
brew install maestro          # macOS — verify the formula first: brew info maestro
```

or download a **pinned release** from the official GitHub repository
(github.com/mobile-dev-inc/maestro), inspect it, then install. Then:
iOS needs Xcode + a booted simulator; Android needs a running emulator or
attached device.

If Maestro cannot be installed through an auditable channel, use honesty
mode (below) rather than compromising on the install.

## Flow skeleton

```yaml
# docs/capture/flows/publish-article.yaml
appId: com.example.magazine
---
- launchApp:
    clearState: true          # deterministic start
- takeScreenshot: docs/manual/images/publish-article/01-home.png
- tapOn: "New Article"
- takeScreenshot: docs/manual/images/publish-article/02-empty-form.png
- inputText: "How to Grow Orchids"
- tapOn: "Publish"
- assertVisible: "Published"
- takeScreenshot: manual/publish-article/03-success
```

Run: `maestro test docs/capture/flows/` (runs every flow in the folder).

## Rules (mirror the web rules)

- Same demo account and seeded backend as the web harness — one data
  hygiene standard for both platforms.
- One device profile per manual set (e.g. iPhone 15 simulator, Pixel 8
  emulator) — mixed device frames look sloppy in a manual.
- `assertVisible` before every screenshot — Maestro waits, but assert the
  element the caption talks about.
- Elements need accessibility labels/ids to be tappable by name — missing
  labels are a finding (they hurt real accessibility too, not just docs).
- Per-locale runs: boot the simulator/emulator in the target locale, rerun
  the same flows.

## Alternatives & special cases

| Tool | When |
|------|------|
| **Fastlane snapshot (iOS) / screengrab (Android)** | You specifically need App Store / Play Store screenshot sets across many devices+locales — it's built for exactly that |
| **Appium** | Complex programmatic flows or device-cloud matrices; heavier setup — overkill for manual screenshots |
| **Flutter `integration_test`** | Flutter apps: capture inside widget/integration tests |
| **Raw `xcrun simctl io booted screenshot` / `adb exec-out screencap`** | Fallback: script the taps yourself or shoot manually with a checklist |

## Honesty mode — no simulator/emulator available

Be honest, don't fake it:

1. Still generate the Maestro flows (they encode the steps — that IS the
   manual's skeleton) and commit the seed scripts.
2. Emit a manual capture checklist: device profile, locale, the exact
   screens and `docs/manual/images/<chapter>/NN-slug.png` filenames to
   shoot.
3. Mark every unfilled image slot in the manual with a visible TODO
   placeholder, listed in the final report — never ship an image slot
   silently empty. The manual is a draft, not a finished manual.
