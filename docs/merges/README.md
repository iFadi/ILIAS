# Merge Resolution Documentation

This directory contains documentation for all merge operations when syncing our fork with upstream ILIAS.

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
