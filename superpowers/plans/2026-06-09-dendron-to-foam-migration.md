# Dendron → Foam Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert this repo from a Dendron workspace into a Foam workspace — folder-based notes, Foam editor config, all Dendron tooling removed. Publishing is out of scope.

**Architecture:** Mechanical migration in four ordered phases: (1) rename flat dot-hierarchy notes into a folder tree with `git mv`, (2) strip Dendron-only frontmatter, (3) write Foam editor config, (4) delete all Dendron files and the dead publish pipeline. Each phase ends with verification commands rather than unit tests — there is no runtime code, so the "tests" are grep/count assertions over the working tree.

**Tech Stack:** git, bash, python3 (preinstalled on macOS), VS Code + Foam extension.

**Commits:** This plan does NOT run `git commit` — the user commits at the checkpoints. (See memory `no-auto-commit`.) Each task ends with a checkpoint for the user to review the diff and commit if they wish.

**Spec:** `docs/superpowers/specs/2026-06-09-dendron-to-foam-migration-design.md`

---

## File Structure

- `notes/**` — note content; flat dot-names become a folder tree, Dendron frontmatter stripped.
- `.vscode/extensions.json` — recommend Foam, drop Dendron.
- `.vscode/settings.json` — drop Dendron keys, add Foam-friendly markdown settings, keep existing generic + gitdoc settings.
- `.gitignore` — drop publish-artifact entries.
- **Deleted:** `dendron.yml`, `dendron.code-workspace`, `notes/*.schema.yml`, `notes/.dendron.cache.json`, `.dendron.ws`, `.dendron.port`, `.dendron.port.cli`, `.github/workflows/publish.yml`, `.next/`, `package.json`, `yarn.lock`, `node_modules/`.

---

## Task 1: Convert notes to a folder tree

**Files:**
- Modify (rename): all of `notes/*.md` whose name contains a dot before `.md`.

- [ ] **Step 1: Snapshot the current state for later verification**

```bash
ls notes/*.md | wc -l        # expect: 38
ls notes/*.md > /tmp/foam-before.txt
```

- [ ] **Step 2: Run the rename script**

The rule: strip `.md`, replace every `.` with `/`, re-append `.md`. Notes with no dot in the stem (e.g. `home.md`, `_why.md`, `root.md`, `books.md`) are left in place. `.yml`/`.json` files are not matched.

```bash
cd notes
for f in *.md; do
  [ -e "$f" ] || continue
  stem="${f%.md}"
  case "$stem" in
    *.*)
      newpath="${stem//.//}.md"
      mkdir -p "$(dirname "$newpath")"
      git mv "$f" "$newpath"
      ;;
  esac
done
cd ..
```

- [ ] **Step 3: Verify the rename — counts and no leftover dot-names**

```bash
find notes -name '*.md' | wc -l                          # expect: 38
ls notes/*.*.md 2>/dev/null && echo "LEFTOVER DOT-NAMES" || echo "OK: no flat dot-names"
```
Expected: `38`, then `OK: no flat dot-names`.

- [ ] **Step 4: Verify history was preserved (renames, not delete+add)**

```bash
git status --short | grep -c '^R'        # expect: a number > 20 (renamed entries)
git status --short | grep '^A' && echo "UNEXPECTED ADDS" || echo "OK: no fresh adds"
```
Expected: a count of renames, and `OK: no fresh adds`.

- [ ] **Step 5: Spot-check the tree shape**

```bash
ls notes/projects notes/projects/software notes/tabletop/black-island-college
test -f notes/projects/software.md && echo "OK: parent file beside folder"
test -f notes/projects.md && echo "OK: dotless parent unchanged"
```
Expected: directory listings present (`software.md`, `astro-cards.md`; `agorascope.md`, `packmule.md`, etc.; `session-1.md`..`session-4.md`), then both `OK:` lines.

- [ ] **Step 6: CHECKPOINT — user reviews `git status` / diff and commits if desired.**

---

## Task 2: Strip Dendron frontmatter

**Files:**
- Modify: every `notes/**/*.md` that has a leading `---` frontmatter block.
- Create (temporary): `/tmp/strip_fm.py` (delete after).

Drops `id`, `desc`, `created`, `updated` from the leading frontmatter block only; keeps `title` and any other keys. Notes without frontmatter are skipped untouched.

- [ ] **Step 1: Write the strip script**

```bash
cat > /tmp/strip_fm.py <<'PY'
import pathlib

DROP = ('id:', 'desc:', 'created:', 'updated:')
changed = 0
for p in sorted(pathlib.Path('notes').rglob('*.md')):
    lines = p.read_text().splitlines(keepends=True)
    if not lines or lines[0].strip() != '---':
        continue
    end = next((i for i in range(1, len(lines)) if lines[i].strip() == '---'), None)
    if end is None:
        continue
    body_fm = [l for l in lines[1:end] if not l.lstrip().startswith(DROP)]
    new = [lines[0]] + body_fm + lines[end:]
    if new != lines:
        p.write_text(''.join(new))
        changed += 1
print(f"rewrote {changed} files")
PY
```

- [ ] **Step 2: Run it**

```bash
python3 /tmp/strip_fm.py
```
Expected: `rewrote 38 files` (or close — any note lacking frontmatter is skipped).

- [ ] **Step 3: Verify the Dendron keys are gone but titles remain**

```bash
grep -rn '^id:' notes/ && echo "STILL HAS id" || echo "OK: no id keys"
grep -rn '^desc:' notes/ && echo "STILL HAS desc" || echo "OK: no desc keys"
grep -rln '^created:\|^updated:' notes/ && echo "STILL HAS dates" || echo "OK: no date keys"
grep -rln '^title:' notes/ | wc -l       # expect: 38
```
Expected: three `OK:` lines, then `38`.

- [ ] **Step 4: Spot-check one rewritten note's frontmatter**

```bash
sed -n '1,6p' notes/projects/software/packmule.md
```
Expected: a `---` / `title: Packmule` / `---` block with no `id`/`desc`/`created`/`updated`.

- [ ] **Step 5: Clean up the temp script**

```bash
rm /tmp/strip_fm.py
```

- [ ] **Step 6: CHECKPOINT — user reviews diff and commits if desired.**

---

## Task 3: Write Foam editor config

**Files:**
- Modify: `.vscode/extensions.json`
- Modify: `.vscode/settings.json`

- [ ] **Step 1: Replace `.vscode/extensions.json`**

```json
{
  "recommendations": [
    "foam.foam-vscode",
    "yzhang.markdown-all-in-one",
    "redhat.vscode-yaml"
  ],
  "unwantedRecommendations": []
}
```

- [ ] **Step 2: Replace `.vscode/settings.json`**

Drops `dendron.enableSelfContainedVaultWorkspace`; keeps the existing generic editor + `gitdoc` settings (gitdoc is the user's own auto-commit workflow, unrelated to Dendron — leave it); adds Foam-friendly markdown completion.

```json
{
  "gitdoc.enabled": true,
  "files.autoSave": "onFocusChange",
  "editor.snippetSuggestions": "inline",
  "editor.suggest.snippetsPreventQuickSuggestions": false,
  "editor.suggest.showSnippets": true,
  "editor.tabCompletion": "on",
  "[markdown]": {
    "editor.quickSuggestions": {
      "other": true,
      "comments": false,
      "strings": false
    }
  },
  "foam.edit.linkReferenceDefinitions": "off"
}
```

- [ ] **Step 3: Verify no Dendron keys remain in `.vscode/`**

```bash
grep -rn 'dendron' .vscode/ && echo "STILL HAS dendron" || echo "OK: vscode clean"
grep -rn 'foam.foam-vscode' .vscode/extensions.json && echo "OK: foam recommended"
```
Expected: `OK: vscode clean`, then a match line + `OK: foam recommended`.

- [ ] **Step 4: CHECKPOINT — user reviews diff and commits if desired.**

---

## Task 4: Rip out Dendron and the dead publish pipeline

**Files:**
- Delete (tracked — use `git rm`): `dendron.yml`, `dendron.code-workspace`, `notes/root.schema.yml`, `notes/dendron.daily.schema.yml`, `.github/workflows/publish.yml`, `package.json`, `yarn.lock`.
- Delete (gitignored/untracked — use plain `rm`): `.dendron.ws`, `.dendron.port`, `.dendron.port.cli`, `notes/.dendron.cache.json`, `.next/`, `node_modules/`, `custom.css` (if present).
- Modify: `.gitignore`

- [ ] **Step 1: Remove tracked Dendron files**

```bash
git rm dendron.yml dendron.code-workspace \
  notes/root.schema.yml notes/dendron.daily.schema.yml \
  .github/workflows/publish.yml package.json yarn.lock
```
Note: if `.github/workflows/` is now empty, that's fine — leave the empty dir or remove it; git tracks files, not dirs.

- [ ] **Step 2: Remove untracked/gitignored Dendron + publish artifacts**

```bash
rm -f .dendron.ws .dendron.port .dendron.port.cli notes/.dendron.cache.json custom.css
rm -rf .next node_modules
```

- [ ] **Step 3: Rewrite `.gitignore` (drop publish-artifact entries)**

```
.DS_Store
```

- [ ] **Step 4: Verify Dendron is gone**

The only surviving `dendron` reference should be the note *about* Dendron and this spec/plan pair.

```bash
grep -rIl 'dendron' . --exclude-dir=.git \
  | grep -v 'notes/projects/software/dendron.md' \
  | grep -v 'docs/superpowers/' \
  && echo "UNEXPECTED dendron refs above" || echo "OK: dendron removed"
```
Expected: `OK: dendron removed` (no stray paths listed before it).

- [ ] **Step 5: Verify the config files are gone**

```bash
for f in dendron.yml dendron.code-workspace .dendron.ws .next node_modules package.json yarn.lock; do
  test -e "$f" && echo "STILL EXISTS: $f" || echo "gone: $f"
done
```
Expected: seven `gone:` lines.

- [ ] **Step 6: CHECKPOINT — user reviews `git status` / diff and commits if desired.**

---

## Final Verification

- [ ] **All notes present in folder form**

```bash
find notes -name '*.md' | wc -l         # expect: 38
```

- [ ] **No Dendron tooling or frontmatter remains**

```bash
grep -rn '^id:' notes/ || echo "OK: frontmatter clean"
grep -rIl 'dendron' . --exclude-dir=.git | grep -v 'notes/projects/software/dendron.md' | grep -v 'docs/superpowers/' || echo "OK: no dendron config"
```
Expected: both `OK:` lines.

- [ ] **Foam loads (manual):** open the repo in VS Code → accept the Foam recommendation → confirm the Foam panel/graph populates from `notes/`. (No links yet, so the graph is a set of disconnected nodes — expected.)

---

## Self-Review Notes

- Spec coverage: Task 1 ↔ spec §1 (folders), Task 2 ↔ §2 (frontmatter), Task 3 ↔ §3 (Foam config), Task 4 ↔ §4 (rip-out). All four spec work-items covered.
- The optional `.foam/templates/daily.md` (spec §3) is intentionally omitted — spec marked it "default skip."
- `notes/root.md` is intentionally retained (spec Notes).
- No `git commit` steps, per the `no-auto-commit` preference — replaced with user checkpoints.
