# Changes Summary - nautobot-app-floor-plan Plugin

## Date
2026-10-22

## Issues Addressed

### 1. PDF Import Modal - Buttons Disappear After Import
**Problem**: When re-importing a PDF, the import button became disabled and pages were not properly cleared.

**Root Cause**: 
- The modal retained state from previous imports (currentPage, rotation, etc.)
- No proper cleanup when opening modal second time
- DOM elements weren't reset between PDF loads

**Fixes Applied to `floorplan-import.js`**:

1. **Added `resetModal()` function** (lines 111-119):
   ```javascript
   function resetModal() {
     els.file.value = "";
     els.pages.innerHTML = "";
     els.uploadBtn.disabled = false;
     setStatus("");
     currentPage = null;
     rotation = 0;
     els.stage.style.transform = "rotate(0deg)";
   }
   ```

2. **Called `resetModal()` in both `openModal()` and `closeModal()`**:
   - Ensures clean state when reopening modal
   - Prevents stale state accumulation

3. **Added `forceReflow()` function** (lines 138-142):
   - Forces browser to recalculate styles on new page loads
   - Fixes rendering issues with rotated images

4. **Improved `renderGrid()` function**:
   - Clears previous state before opening new page
   - Calls `forceReflow()` for proper styling updates

5. **Enhanced `openCrop()` function**:
   - Detects if switching to a different page
   - Clears grid and forces reflow for proper state transition

### 2. Rotate Image - Buttons Hidden Behind Transformed Content
**Problem**: When rotating an image during calibration/edit mode, toolbar buttons become inaccessible.

**Root Cause**:
- SVG transform creates a stacked context that can cover sibling elements
- Insufficient z-index stacking for toolbar over transformed canvas

**Fix Applied to `floorplan-tools.css`**:

Added GPU acceleration hint and backface visibility for toolbar:
```css
.floor-plan-toolbar {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 5;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 5px 6px;
  background: var(--bs-body-bg, #fff);
  border: 1px solid var(--bs-border-color, #cfd6dd);
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.14);
  /* Ensure toolbar buttons stay clickable even when blueprint is rotated */
  transform: translateZ(0);
  backface-visibility: visible;
}
```

This ensures the toolbar maintains its stacking order and remains clickable despite transforms on child elements.

## Files Modified

1. **nautobot_floor_plan/static/nautobot_floor_plan/js/floorplan-import.js**
   - Added resetModal() function
   - Modified openModal() to call resetModal()
   - Modified closeModal() to call resetModal()
   - Added forceReflow() helper
   - Enhanced renderGrid() with state clearing
   - Enhanced openCrop() with state detection

2. **nautobot_floor_plan/static/nautobot_floor_plan/css/floorplan-tools.css**
   - Added GPU hint for toolbar z-index preservation

## Testing Recommendations

1. **PDF Import Flow**:
   - Upload PDF → select page → crop → apply → success
   - Repeat upload without page refresh → should show fresh modal with enabled button
   - Switch between pages after first selection → should clear previous state

2. **Rotation Flow**:
   - Enter Calibrate mode → rotate blueprint → verify toolbar buttons remain accessible
   - Try clicking Edit button while blueprint is at 90° rotation

## Backwards Compatibility
All changes are backward compatible:
- No API changes
- No database migrations required
- Pure UI/UX improvements
- Existing functionality preserved

## Future Considerations
- Consider adding animation/transitions for cleaner modal resets
- May want to add keyboard shortcuts for common PDF operations
- Could enhance crop tool with aspect ratio presets
