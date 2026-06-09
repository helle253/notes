# Dendron → Foam Migration

**Date:** 2026-06-09
**Status:** Approved design

## Goal

Convert this repository from a Dendron workspace into a Foam workspace. Editor
and file-layout only. Publishing is **explicitly out of scope** — the live site
at `notes.hellbhoy.net` will go stale (not down) until a publishing setup is
wired up in a separate, future pass.

## Context

- Single self-contained Dendron vault rooted at `notes/`, 38 markdown notes.
- Notes use Dendron's flat dot-hierarchy filenames (e.g.
  `projects.software.packmule.md`), max depth 3 dots.
- **Zero wikilinks** across all notes (`[[...]]`, `![[...]]`, and `.md`
  cross-links all absent) — verified by grep. Nothing breaks when files move.
- Dendron frontmatter on every note: `id`, `title`, `desc`, `created`,
  `updated` (epoch ms).
- A Dendron-based publishing pipeline exists (`.github/workflows/publish.yml`,
  `.next/`, `@dendronhq/dendron-cli`) — to be removed with the rest of Dendron.

## Non-goals

- No replacement publishing system (Quartz/Jekyll/etc.). Deferred to a later pass.
- No introduction of wikilinks. Foam will resolve them by filename if/when the
  user adds them later; depth won't matter.
- No content rewriting beyond frontmatter cleanup.

## Work

### 1. Convert notes to folders

Mechanical rename, one `git mv` per file (preserves history). Rule: strip
`.md`, replace every `.` with `/`, re-append `.md`. A parent note that also has
children lands as a file beside its folder.

Examples / edge cases:

| from | to |
|---|---|
| `notes/projects.software.packmule.md` | `notes/projects/software/packmule.md` |
| `notes/projects.software.md` (parent) | `notes/projects/software.md` |
| `notes/projects.md` (parent, no dots) | `notes/projects.md` (unchanged) |
| `notes/books.md` (parent) | `notes/books.md` (unchanged) |
| `notes/tabletop.black-island-college.session-1.md` | `notes/tabletop/black-island-college/session-1.md` |
| `notes/_why.md`, `home.md`, `resume.md`, `root.md` | unchanged (no dots) |

The seven parent notes that sit beside a new folder: `books`, `misc`,
`projects`, `projects/software`, `templates`, `vedanta-reading-group`,
`writing`. No filesystem collisions (a file `foo.md` can coexist with a `foo/`
dir).

### 2. Strip Dendron frontmatter

One scripted pass over all notes: drop `id`, `desc`, `created`, `updated`;
**keep `title`**. Foam falls back to the first `# heading` for a title, so even
notes that lose their frontmatter title still display sensibly.

### 3. Foam editor config

- `.vscode/extensions.json`: recommend `foam.foam-vscode`,
  `yzhang.markdown-all-in-one`, `redhat.vscode-yaml`. Remove all `dendron.*`
  recommendations and the Dendron `unwantedRecommendations`.
- `.vscode/settings.json`: remove Dendron keys
  (`dendron.enableSelfContainedVaultWorkspace`), keep generic markdown/editor
  settings, add Foam-friendly defaults.
- Foam needs no mandatory config file; it scans the workspace for `.md`.
- **Optional, default skip:** port `templates.daily.md` to a Foam template at
  `.foam/templates/daily.md` for Foam's daily-note command. Not implementing
  unless requested.

### 4. Rip out Dendron

Delete:

- `dendron.yml`
- `.dendron.ws`, `.dendron.port`, `.dendron.port.cli`, `notes/.dendron.cache.json`
- `notes/root.schema.yml`, `notes/dendron.daily.schema.yml`
- `dendron.code-workspace`
- `.next/` (Dendron's Next.js publish build)
- `.github/workflows/publish.yml`
- `package.json`, `yarn.lock`, `node_modules/` (existed only for
  `@dendronhq/dendron-cli` and the antd theme generator)

Update `.gitignore` to drop now-irrelevant entries (e.g. `.next`,
`node_modules`).

## Verification

- `git status` / `git mv` history intact for every renamed note.
- No `dendron` references remain in tracked files: `grep -ri dendron .`
  returns only the intentionally-kept note `notes/projects/software/dendron.md`
  (a note *about* Dendron, content unrelated to config).
- Opening the repo in VS Code surfaces Foam, not Dendron, as the recommended
  extension; Foam's graph/backlinks panel loads against `notes/`.
- All 38 notes still present, now in the folder tree.

## Notes

- `notes/root.md` is Dendron's vault-root note. Left as-is (harmless); a
  separate `home.md` already exists.
- When the user later adds `[[wikilinks]]` and revisits publishing, a
  wikilink-aware static-site generator (Quartz was the leading candidate) will
  be needed — recorded here only as future context, not part of this work.
