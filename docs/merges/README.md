# Merge Resolution Documentation

This directory contains documentation for all merge operations when syncing our fork with upstream ILIAS.

## Fork Branch Model

This fork uses the **mirror + rebased patch** model. Two branches, each with one job:

```
release_10              Pristine mirror of upstream/release_10.
                        NEVER commit here. Advanced only by fast-forward.

custom/manual-scoring   The ConsecutiveScoring backport (11 -> 10), ~61 files.
                        Rebased onto release_10 at every sync.
                        <- work from and deploy from this branch.
```

Because `release_10` never diverges, the fork's entire delta is always one command:

```bash
git diff release_10...custom/manual-scoring
```

### Sync Routine

Run this whenever you want to pick up upstream changes:

```bash
# 1. Advance the mirror (fast-forward only)
git fetch upstream --filter=blob:none
git switch release_10
git merge --ff-only upstream/release_10
git push origin release_10

# 2. Replay our patch onto the new upstream
git switch custom/manual-scoring
git rebase release_10
# ...resolve conflicts (historically a small surface: ~9 files)
git push --force-with-lease origin custom/manual-scoring
```

**`--ff-only` is the guardrail.** If it ever refuses, someone committed directly to the mirror.
Do not merge to "fix" it — reset the mirror back to upstream:

```bash
git switch release_10 && git reset --hard upstream/release_10
```

**Always `--force-with-lease`, never bare `--force`.** The lease aborts the push if the remote
moved unexpectedly, which protects against clobbering work you have not seen.

### Remotes

```
origin     git@github.com:iFadi/ILIAS.git              (fetch + push, blobless)
upstream   https://github.com/ILIAS-eLearning/ILIAS.git (fetch only, blobless)
```

`upstream`'s push URL is deliberately set to `DISABLED_read_only` so an accidental
`git push upstream` fails loudly instead of reaching the ILIAS project.

This is a **blobless clone** (`--filter=blob:none`): full history, file contents fetched on
demand. Rebasing and checking out old commits needs network. Do not run full-history scanners
against it without unshallowing first.

### Anti-Pattern: The Fake Merge

Never commit an upstream sync as a single-parent commit. A commit whose message says `merge:`
but whose parent count is 1 severs ancestry with upstream and causes escalating conflicts on
every subsequent sync. Check before pushing a sync:

```bash
git cat-file -p HEAD | grep -c '^parent'   # a real merge prints 2
```

This exact mistake was made in the 2026-02 sync and repaired on 2026-09-01. See
`2026-09-01-fork-restructure.md`.

## Purpose

Track merge conflicts, resolutions, and rationale for future reference. This helps:
- Understand why certain decisions were made
- Maintain consistency in conflict resolution
- Document architectural divergences from upstream
- Provide context for future maintainers

## File Naming Convention

`YYYY-MM-DD-upstream-sync.md`

Example: `2026-02-12-upstream-sync.md`

## Document Template

Each merge document should include:

1. **Summary**
   - Sync date
   - Upstream version/commit
   - Statistics (files changed, lines added/deleted, conflicts)

2. **Conflicts Resolved** (if any)
   - File path
   - Line numbers
   - Conflict type
   - Why it happened (both sides' changes)
   - Resolution decision (kept ours/theirs/manual merge)
   - Justification
   - Testing requirements

3. **Auto-Merged Files**
   - Brief summary of major changes
   - Notable additions/deletions
   - Security fixes
   - Breaking changes (if any)

4. **Notes**
   - Any special considerations
   - Follow-up tasks
   - Related issues

## Best Practices

### When Creating a Merge Document

1. **Be Specific**: Include exact line numbers, commit hashes, and file paths
2. **Explain Why**: Don't just document what was done, explain the reasoning
3. **Show Code**: Include relevant code snippets for context
4. **List Testing**: Document what needs to be tested after resolution
5. **Link Issues**: Reference bug tracker issues when relevant

### Conflict Resolution Guidelines

When resolving conflicts, consider:
- **Functionality**: Does our version include upstream bug fixes?
- **Architecture**: Is our approach more maintainable?
- **Compatibility**: Will this cause issues with future upstream merges?
- **Testing**: Can we verify the resolution works correctly?

### Decision Priority

1. If upstream has a security fix → Use upstream version
2. If our architecture is superior AND includes upstream fixes → Use ours
3. If both versions are equivalent → Use upstream (easier future merges)
4. If conflict is complex → Manual merge with careful testing

## Viewing Merge History

To see all merges:
```bash
ls -lt docs/merges/*.md
```

To search for specific conflicts:
```bash
grep -r "File:" docs/merges/
```

## Related Documentation

- Main docs: `docs/README.md`
- Development guidelines: `docs/development/`
- Configuration: `docs/configuration/`
