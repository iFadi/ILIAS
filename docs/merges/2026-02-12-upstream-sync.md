# Upstream Sync - February 12, 2026

## Summary

- **Sync Date:** 2026-02-12
- **Upstream Version:** v10.5-131-g26b4201ad6
- **Branch:** release_10
- **Total Files Changed:** 406 files
- **Lines Added:** 8,226
- **Lines Deleted:** 6,066
- **Net Change:** +2,160 lines
- **Conflicts:** 1 file

## File Operations

- **Modified:** 372 files
- **Added:** 18 files
- **Deleted:** 16 files

## Conflicts Resolved

### File: `components/ILIAS/UI/src/Implementation/Component/Input/ViewControl/Renderer.php`

**Lines:** 141-171  
**Method:** `renderSortation()`  
**Conflict Type:** Code architecture refactoring collision

#### Why It Happened

**Upstream Changes (ILIAS):**
- Applied fix for bug #46579 (commit 434c238564)
- Added inline jQuery event handler to properly handle array values in Sortation ViewControl
- Inline JavaScript code splits colon-separated values and handles arrays:
  ```javascript
  let val = Array.isArray(signal_data.options.value)
    ? signal_data.options.value
    : signal_data.options.value.split(':');
  ```

**Local Changes (Our Fork):**
- Refactored ViewControl components to use modern ES6 class architecture
- Extracted inline JavaScript to dedicated modular classes
- Created `components/ILIAS/UI/resources/js/Input/ViewControl/src/sortation.js`
- Uses cleaner initialization pattern: `il.UI.Input.Viewcontrols.Sortation.init()`

#### Resolution

**Decision:** Kept our modular architecture (local version)

#### Justification

1. **Our code already includes the upstream fix** - Our `sortation.js` file (lines 51-53) handles both array and colon-separated values:
   ```javascript
   const val = Array.isArray(signalData.options.value)
     ? signalData.options.value
     : signalData.options.value.split(':');
   ```

2. **Superior architecture** - Our refactored approach:
   - Uses modern ES6 classes with proper separation of concerns
   - Follows single responsibility principle
   - More maintainable and testable
   - Consistent with other ViewControl components (FieldSelection, Pagination, Mode)

3. **No functionality loss** - Both implementations solve the same bug, but ours is better structured

#### Code Comparison

**Upstream Version (Inline):**
```php
fn($id) => "$(document).on('{$internal_signal}', 
    function(event, signal_data) { 
        let container;
        if(signal_data.options.parent_container) {
            container = document.querySelector(
                '#' + signal_data.options.parent_container 
                + ' .il-viewcontrol-sortation'
            );
        } else {
            container = event.target.closest('.il-viewcontrol-sortation');
        }
        let inputs = container.querySelectorAll('.il-viewcontrol-value > input');
        let val = Array.isArray(signal_data.options.value)
          ? signal_data.options.value
          : signal_data.options.value.split(':');
        inputs[0].value = val[0];
        inputs[1].value = val[1];
        $(event.target).trigger('{$container_submit_signal}');
        return false;
    });"
```

**Our Version (Modular):**
```php
fn($id) => "il.UI.Input.Viewcontrols.Sortation.init(
    document.getElementById('{$id}'),
    '{$internal_signal}',
    '{$container_submit_signal}',
);"
```

With implementation in `sortation.js`:
```javascript
init(component, internalSignal, containerSubmitSignal) {
  this.#eventDispatcher.register(
    component.ownerDocument,
    internalSignal,
    (event, signalData) => {
      let container;
      if (signalData.options.parent_container) {
        container = component.ownerDocument.querySelector(
          `#${signalData.options.parent_container} .il-viewcontrol-sortation`,
        );
      } else {
        container = event.target.closest('.il-viewcontrol-sortation');
      }
      const inputs = container.querySelectorAll('.il-viewcontrol-value > input');
      const val = Array.isArray(signalData.options.value)
        ? signalData.options.value
        : signalData.options.value.split(':');
      [inputs[0].value, inputs[1].value] = val;
      this.#eventDispatcher.dispatch(event.target, containerSubmitSignal);
      return false;
    },
  );
}
```

#### Testing Required

- [ ] Test Sortation ViewControl with different data types (arrays and strings)
- [ ] Verify parent container functionality works correctly
- [ ] Test sortation in all UI contexts (tables, lists, etc.)
- [ ] Ensure event dispatching works as expected

#### Related Files

- `components/ILIAS/UI/resources/js/Input/ViewControl/src/sortation.js`
- `components/ILIAS/UI/resources/js/Input/ViewControl/src/index.js`
- `components/ILIAS/UI/resources/js/Input/ViewControl/dist/input.viewcontrols.min.js`

---

## Auto-Merged Files (No Conflicts)

All other **405 files** were automatically merged by Git without conflicts.

### Notable Changes from Upstream

#### Security Fixes
- SOAP permission checks for `getSCORMCompletionStatus`, `hasSCORMCertificate`
- Authorization check for `getLearningProgressChanges`
- Fix for unauthorized object moving in SOAP API

#### New Files (18)
- Object Properties classes for CategoryReference, CourseReference, GroupReference
- Container Setup classes (WebFeedCreationDeletedObjective)
- Exercise text submission purifier
- Wiki import resolver
- MediaPool permanent link manager
- Test archive template
- Setup objectives

#### Deleted Files (16)
- 2 Badge picture Flavour classes (refactored)
- 2 Chatroom images (unused)
- 12 Forum tree images (replaced with CSS)

#### Top Components Updated
1. COPage - 25 files (editor improvements)
2. Forum - 20 files (UI modernization)
3. DataCollection - 19 files
4. Exercise - 18 files (text submission features)
5. Badge - 14 files (table refactoring)

#### Other Updates
- 32 language files updated (all translations)
- Version bump in `ilias_version.php`
- Composer dependencies updated
- Style/CSS improvements
- GitHub workflow updates

---

## Notes

- This sync incorporates changes from ILIAS release_10 branch up to commit 26b4201ad6
- The merge was completed successfully with minimal conflicts
- Our fork maintains architectural improvements while incorporating upstream bug fixes
- No breaking changes detected

## Commit

```
commit 2cd1d91781...
Author: [Your Name]
Date: Thu Feb 12 06:49:43 2026 +0200

    merge: sync with upstream/release_10
    
    Merge upstream changes from ILIAS release_10 branch.
    
    Resolved conflict in ViewControl Renderer:
    - Kept local refactored Sortation.init() architecture
    - Local version already includes fix from upstream #46579
    - Maintains cleaner modular JS structure with dedicated classes
```
