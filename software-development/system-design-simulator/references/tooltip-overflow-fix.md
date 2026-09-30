# Tooltip Overflow Escape Pattern

## Problem
Tooltips/overlays using `absolute` positioning get clipped when any ancestor has `overflow-hidden` or `overflow-auto`.

## Root Cause
CSS `overflow` creates a new stacking context that clips absolutely positioned children.

## Solution Pattern

### Option 1: Remove overflow clipping on scroll containers
```tsx
// In ComponentPalette.tsx - ComponentsTab scroll container
<div className="flex-1 overflow-y-auto overflow-x-visible py-1">
  {/* items with absolute tooltips render here */}
</div>

// Also remove overflow-hidden from parent chain:
// ComponentPalette outer: remove overflow-hidden
// Tab content container: remove overflow-hidden
// ComponentsTab outer: remove overflow-hidden
```

### Option 2: Portal (for complex cases)
```tsx
import { createPortal } from 'react-dom'

function Tooltip({ children, open }) {
  if (!open) return null
  return createPortal(
    <div className="fixed z-50 ...">...</div>,
    document.body
  )
}
```

### Option 3: Relative parent + visible overflow
```tsx
// Parent needs: relative + overflow-x-visible
<div className="relative overflow-y-auto overflow-x-visible">
  <ItemWithTooltip />
</div>

// Tooltip: absolute bottom-full left-1/2 -translate-x-1/2
<div className="absolute bottom-full left-1/2 -translate-x-1/2 hidden group-hover:block z-50">
  <TooltipContent />
</div>
```

## Files Fixed in This Project
- `apps/web/src/components/palette/ComponentPalette.tsx`
  - Removed `overflow-hidden` from ComponentPalette root (line 117)
  - Removed `overflow-hidden` from tab content container (line 145)
  - Added `overflow-x-visible` to ComponentsTab scroll container (line 165)
  - Added `relative` to each draggable item (line 179)

## Checklist for Future Tooltips
- [ ] Check all ancestors from tooltip up to root for `overflow-hidden`/`overflow-auto`
- [ ] If scroll container needed, use `overflow-y-auto overflow-x-visible`
- [ ] Add `relative` to immediate parent of tooltip trigger
- [ ] Tooltip uses `z-50` to stay above other content
- [ ] Test hover at edges of container — tooltip should be fully visible