# Fork Restructure - September 1, 2026

## Summary

- **Date:** 2026-09-01
- **Type:** Structural repair + upstream sync
- **Upstream tip synced to:** `db0e5de43e` ([Survey] fix: Add url permalink (#12005), 2026-09-01)
- **Previous fork tip:** `c0b8766eaa` (preserved as tag `backup/pre-cleanup-2026-09-01`)
- **Result:** `release_10` is now a pristine upstream mirror; fork customization moved to
  `custom/manual-scoring`

## Root Cause

The fork was reporting **922 commits behind** `upstream/release_10` while the true gap was
**618 commits**. The discrepancy came from commit `2cd1d91781`:

```
2cd1d91781  "merge: sync with upstream/release_10"   parents = 1
```

Despite the `merge:` prefix, this commit has a **single parent**. It is not a merge. The
February 2026 sync was performed by copying upstream's file contents into the working tree and
committing them as ordinary fork work. Git therefore had no ancestry link to upstream:

| Effect | Detail |
|---|---|
| merge-base frozen | stuck at `5cb7b725eb` (Release 10.4, 2025-12-16) |
| Inflated "behind" count | 922 reported vs 618 actual |
| Guaranteed future conflicts | git would replay 922 commits onto a tree that already held most of them |
| Wrong attribution | ~900 commits of upstream code credited to the fork |

## Why This Fork Exists

The fork carries **one long-lived customization**: nhaagen's `ConsecutiveScoring` (manual
scoring) feature, backported from `release_11` to `release_10` via PR #1.

Verified at restructure time:

- Present in `upstream/release_11` ✅
- Present in `upstream/trunk` ✅
- Present in `upstream/release_10` ❌ — **and it will not be added**

The patch must therefore be maintained until an ILIAS 11 upgrade, at which point it becomes
redundant and this fork can be retired.

## What Was Done

1. **Safety net.** Tagged the pre-restructure tip as `backup/pre-cleanup-2026-09-01` and pushed
   it. All five original commits remain reachable.
2. **Mirror rebuilt.** `release_10` reset to `upstream/release_10` and force-pushed
   (`--force-with-lease`). It is now byte-identical to upstream.
3. **Patch branch created.** `custom/manual-scoring` branched from the mirror, then:
   - `git cherry-pick 5132b2bb40` — the backport, **authorship preserved (Nils Haagen)**
   - `git cherry-pick c0b8766eaa` — the docs commit
4. **Discarded** as redundant: `2cd1d91781` (fake merge), `18d79ba104` and `6eae8a475d`
   (merge wrappers). Their content is either upstream's already or preserved in the two
   cherry-picks.

## Conflicts Resolved

Only **2 conflicts** arose across 618 commits of upstream drift.

### 1. `components/ILIAS/UI/src/Implementation/Component/Input/ViewControl/Renderer.php`

**Lines:** 143-170 · **Method:** `renderSortation()`

- **Upstream side:** inline jQuery handler carrying the fix for bug #46579 (array-vs-colon
  value handling).
- **Backport side:** delegates to `il.UI.Input.Viewcontrols.Sortation.init(...)`.

**Resolution:** kept the backport's modular call.

**Justification — verified, not assumed.** The #46579 fix is present in the modular
implementation at `components/ILIAS/UI/resources/js/Input/ViewControl/src/sortation.js:51-53`:

```javascript
const val = Array.isArray(signalData.options.value)
  ? signalData.options.value
  : signalData.options.value.split(':');
```

The resolved file was diffed against `5132b2bb40` and matches the backport exactly — no
hand-introduced drift.

### 2. `templates/default/delos.css`

**Blocks:** 2, at lines ~2866 and ~4112.

**Resolution:** took the incoming additions.

**Justification:** both conflict blocks had an **empty** `HEAD` side (0 bytes) — they are pure
additions of Sequence component styles that git flagged only because surrounding context had
drifted. This was asserted programmatically before resolving, not eyeballed. `delos.css` is a
generated artifact; its SCSS source
(`templates/default/070-components/UI-framework/Navigation/_ui-component_sequence.scss`) and the
`@use` entry in `_index.scss` both came in with the backport, so source and output stay
consistent.

## Verification

| Gate | Result |
|---|---|
| `git merge-base custom/manual-scoring upstream/release_10` | `db0e5de43e` = current upstream tip ✅ |
| `git diff release_10 upstream/release_10` | empty ✅ |
| Fork-added files present (40) | 0 missing ✅ |
| Delta file count vs pre-restructure reference | 61 = 61, identical set ✅ |
| JS syntax (`node --check`, 7 modules) | 0 errors ✅ |
| Conflict markers remaining | none ✅ |
| Backup tag reachable | `backup/pre-cleanup-2026-09-01` ✅ |

**Not verified here:** PHP lint and PHPUnit — no PHP binary in the restructure environment.
Deferred to CI (`.github/workflows/checks.yml`) and to manual testing of the Test component's
manual-scoring screens on a running ILIAS instance.

## Follow-Up

- [ ] Run CI on `custom/manual-scoring`; confirm PHP lint and the UI test suite pass.
- [ ] Manually verify consecutive scoring works end-to-end on a running instance.
- [ ] Re-evaluate this fork at ILIAS 11 upgrade — the patch becomes redundant then.
