---
name: bump-version
description: >
  Bump the zigporter version and update CHANGELOG.md. Analyses unreleased commits
  to determine the correct semver bump, moves [Unreleased] entries to the new version
  section, fixes comparison links, and commits. Does NOT tag or push — follow with
  the tagging steps in guides/publishing.md.
---

## Steps

1. **Find the last released version and tag**

```bash
git tag --sort=-version:refname | head -1
grep '^version' pyproject.toml
```

2. **List commits since the last tag**

```bash
git log <last-tag>..HEAD --oneline
```

3. **Determine the semver bump** using these rules:
   - `feat:` commits → **minor** bump (0.x.0)
   - `fix:` / `docs:` / `chore:` only → **patch** bump (0.0.x)
   - Any breaking change indicator (`!` after type, or `BREAKING CHANGE` in body) → **major** bump
   - If `[Unreleased]` in CHANGELOG.md is empty, ask the user what version to target before proceeding.

4. **Update `pyproject.toml`** — change `version = "X.Y.Z"` to the new version.

5. **Update `CHANGELOG.md`**:
   - Replace `## [Unreleased]` section header with both a new empty unreleased section and the new version entry:
     ```
     ## [Unreleased]

     ## [X.Y.Z] - YYYY-MM-DD
     ```
   - Today's date is available from the `currentDate` context.
   - Preserve all existing bullet points under the new version heading.
   - Update the comparison URL links at the bottom of the file:
     - `[Unreleased]` link: `compare/vX.Y.Z...HEAD`
     - Add new `[X.Y.Z]` link: `compare/v<prev>...vX.Y.Z`

6. **Show a diff summary** of both files and confirm with the user before committing.

7. **Commit** (only after user confirms):
```bash
git add pyproject.toml CHANGELOG.md
git commit -m "chore: bump version to X.Y.Z"
```

8. **Remind the user** of the next steps from `guides/publishing.md`:
```
Next: git tag vX.Y.Z && git push origin main && git push origin vX.Y.Z
```
