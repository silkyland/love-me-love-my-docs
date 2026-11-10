# Desktop Capture Harness — OS-level screenshots + GUI automation

Use this path when the program under documentation is a desktop app —
native (Qt, .NET, AppKit/SwiftUI) or cross-platform (Electron, Tauri).
Sibling paths: [capture-web.md](capture-web.md),
[capture-mobile.md](capture-mobile.md), [capture-cli.md](capture-cli.md).

> Verify tool flags and APIs against the installed versions and OS —
> desktop tooling drifts and differs per platform.

## First decision: can you drive it like a web page?

- **Electron apps** often can: Playwright ships Electron bindings
  (`_electron.launch`) that expose the renderer as a page — the
  [capture-web.md](capture-web.md) patterns (stable locators,
  `wait_for_load_state`, `page.screenshot`) mostly apply. Verify against
  the installed Playwright version; window chrome still needs OS-level
  handling if the manual shows it.
- **Tauri / native / other** → OS-level capture below.

## Screenshot mechanics per OS

| OS | Capture | GUI automation |
|----|---------|----------------|
| macOS | `screencapture -x` (silent), `-l <windowid>` for one window, `-R x,y,w,h` for a region | `osascript` (System Events), `cliclick` |
| Linux | `gnome-screenshot -w` (active window), `import -window` (ImageMagick), `maim` | `xdotool` (X11), `ydotool`/portal (Wayland — restricted), `pyautogui` |
| Windows | PowerShell + `System.Drawing`/`CopyFromScreen`, or `nircmd`/`flameshot` | `pyautogui`, UI Automation via PowerShell/Python (`pywinauto`, `uiautomation`) |

`pyautogui` is the cross-platform fallback for input; screenshots still
come from the OS tools above.

## Window discipline — where desktop captures go wrong

A desktop screenshot documents nothing if the window isn't exactly
right. Before EVERY shot:

1. **Frontmost and unobstructed** — activate the app, verify it owns
   focus; a buried or half-covered window is a wasted frame. Use a clean
   desktop/Space and silence notifications for the run.
2. **Fixed geometry** — pin window size and position once; record them
   as constants like the web viewport.
3. **Wait for stable content** — poll for the element/state the step
   describes (window title, accessibility tree, a pixel-diff quiet
   period) instead of fixed sleeps where possible.
4. **Theme and scaling pinned** — light/dark mode, OS zoom/HiDPI scale:
   one setting per manual set, never mixed.
5. **Animations killed where possible** — spinners and transitions make
   frames nondeterministic; prefer settled states.

## Locators and accessibility

Prefer driving the app through its **accessibility tree** (macOS
`AXUIElement`, Linux AT-SPI, Windows UI Automation, or `pyautogui` +
image locators as last resort). Missing accessibility labels are an app
finding — they hurt real assistive-tech users, not just docs. Count and
report bare image-recognition locators the way the web policy counts CSS
selectors.

## Permissions and environment traps

- **macOS:** the terminal/automation host needs Screen Recording
  permission — a capture that returns a blank/desktop-only image usually
  means permission was denied, not that the app is broken. Check before
  reporting a finding.
- **Linux:** X11 vs Wayland changes everything — Wayland restricts both
  screen capture and synthetic input. Detect the session type first;
  document which one the manual's captures came from.
- **Headless/SSH:** no display = no desktop capture. Honesty mode below.

## Determinism

Same rules as web: seeded demo data, one account, pinned locale. Desktop
adds: reset window state between runs (position, size, open panels),
clear recent-files menus that leak real paths, and remember the OS may
restore last-session state — launch with a clean profile dir where the
app supports it.

## Honesty mode — no GUI session available

Headless server, no display, permission denied you can't fix:

1. Still commit the capture and seed scripts — they encode the steps and
   the data.
2. Emit a manual capture checklist: OS, window geometry, theme, locale,
   exact screens and `docs/manual/images/<chapter>/NN-slug.png` filenames.
3. Mark every unfilled image slot with a visible TODO; list them in the
   report. Never substitute a mockup or a web screenshot "close enough."
   The manual is a draft, not a finished manual.
