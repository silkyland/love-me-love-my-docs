# CLI Capture Harness — real command runs, real output

For a command-line program there is no screenshot. The capture IS the
command: a committed script runs each documented command for real and
saves stdout, stderr, and the exit code as a transcript. The manual pastes
that output — never output typed from memory, never "cleaned up" by hand.
Sibling paths: [capture-web.md](capture-web.md),
[capture-desktop.md](capture-desktop.md), [capture-mobile.md](capture-mobile.md).

> Verify tool flags against the installed versions (`--help`, man pages) —
> CLI tooling drifts.

## Why transcripts rot differently

A GUI screenshot lies when the UI changes. A transcript lies when the
*output* changes — a flag renamed, a column added, a version string
bumped. The defense is the same: the transcript is a build artifact of a
committed script, and regeneration diffs it.

## Harness skeleton

```bash
# docs/capture/run_all.sh — one entry per documented step
#!/usr/bin/env bash
set -euo pipefail
OUT=docs/manual/transcripts

run() {  # run <chapter> <NN-slug> <command...>
  local chapter="$1" slug="$2" rc=0; shift 2
  mkdir -p "$OUT/$chapter"
  {
    echo "\$ $*"
    "$@" 2>&1 || rc=$?   # capture the failure, don't abort on it
    echo "# exit: $rc"
  } > "$OUT/$chapter/$slug.txt"
  # a non-zero rc is evidence for triage (SKILL.md Step 6), not a harness error
}

run getting-started 01-version        mycli --version
run getting-started 02-help           mycli --help
run publish-article  01-init          mycli init --demo
run publish-article  02-create        mycli articles create --title "How to Grow Orchids"
```

Naming rule (same as GUI platforms): `<chapter>/<NN>-<slug>.txt` — the
filename IS the step order.

## Pinned environment — the CLI's viewport

Output changes with the environment, so pin it once, in one place:

- **Terminal width** — wrapping changes everything. `COLUMNS=100` (or
  `stty cols 100`) for the whole run.
- **`TERM`** — `xterm-256color` for rich output, `dumb`/`TERM=dumb` or
  `NO_COLOR=1` when you want plain text; pick one and never mix.
- **Locale** — `LC_ALL=C.UTF-8` (or the manual's locale) so messages and
  sorting are stable.
- **Clean env** — run under `env -i` plus only the variables the app
  needs. Your shell's tokens and config must not leak into output.
- **Fixed cwd and seeded `HOME`** — apps that read `~/.config` get a
  seeded demo HOME so state is deterministic and contains no real data.
- **No TTY surprises** — many CLIs change behavior when stdout isn't a
  terminal (no colors, no progress bars, pagination kicks in). Capture
  through a pty (`script -q`, Python's `pty`, or `expect`) for faithful
  interactive output, or pass `--no-pager`/`PAGER=cat` and document that
  readers see the piped form. Pick one mode per manual and say which.

## Capturing faithfully

- **stdout vs stderr:** the skeleton merges them with `2>&1` for
  readability — acceptable, but say so. If you keep them separate,
  label which stream produced what. Error paths matter to readers
  either way.
- **Exit codes are part of the output.** Record `$?` per step; a
  documented command that exits non-zero unexpectedly is an app finding,
  not a docs problem. Expected failures (usage errors) are documented
  *with* their non-zero exit — that is real behavior users must know.
- **Interactive prompts:** drive them with piped stdin or `expect`; the
  transcript must show the prompt, the input, and the result — the whole
  exchange, not just the aftermath.
- **Volatile bits:** timestamps, durations, random IDs, temp paths,
  versions. Freeze what you can (`faketime`, seeded data, pinned
  versions); normalize the rest through documented `sed` rules in the
  harness — and list every normalization in the final report.

## Output hygiene — the CLI's "zero real user data" rule

Command output leaks what screenshots never could: tokens in env vars,
absolute home paths (`/Users/you`), hostnames, emails, internal URLs.

- Run against localhost/seeded backends; the production gate from
  SKILL.md Step 3 applies to any command that writes state.
- Normalize machine-specific noise (`$HOME` → `~`, host → `<host>`,
  ephemeral ports → `<port>`) via the harness's documented rules —
  never by editing the manual's text by hand.
- An elision marker (`…`) is allowed only for middle-truncating long
  output, and must be visibly marked. Never elide an error.
- A secret that can't be scrubbed without changing what the output
  means is an app finding — report it.

## Manual format

Short outputs go inline in the chapter as fenced blocks — they ARE the
expected result:

````markdown
2. Create a draft article:

   ```console
   $ mycli articles create --title "How to Grow Orchids"
   Created article 42: How to Grow Orchids
   # exit: 0
   ```

   *You should see the new ID and title echoed back.*
````

Long outputs (> ~40 lines) live as transcript files under
`docs/manual/transcripts/<chapter>/NN-slug.txt`; the chapter shows the
head, an elision, and a link. The harness writes the transcript; a small
splice step (markers like `<!-- transcript:publish-article/02-create -->`
in the chapter) re-embeds refreshed output on regeneration — the manual
never hand-edits captured text.

## Findings the harness surfaces

- `--help` text that contradicts behavior, undocumented flags, output
  that leaks paths or secrets — all app findings, reported with the
  command that produced them.
- A flow that exists in `--help` but fails when run — the CLI's version
  of "a step the harness couldn't execute."

## Honesty mode — the CLI can't run here

Binary missing, wrong platform, needs hardware/network you don't have:

1. Still commit the command list and harness skeleton — they encode the
   steps.
2. Emit a capture checklist: exact commands, pinned env, where each
   transcript lands.
3. Mark every transcript slot in the manual with a visible TODO — and
   never write "expected output" from memory. That is fabrication; the
   whole point of this skill is that the output was real.
4. The report lists exactly which commands/flows remain unverified.
