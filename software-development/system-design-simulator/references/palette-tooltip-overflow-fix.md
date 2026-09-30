# Palette Tooltip Overflow Fix

## Problem
The component palette tooltip (showing cloud equivalents on hover) was not visible because:
1. The palette container had `overflow-hidden` clipping the tooltip
2. The tooltip was positioned with `absolute bottom-full` (above) but palette is on left edge → goes off-screen
3. Missing `relative` positioning on the item container

## Solution
1. **Remove `overflow-hidden` from parent containers** (ComponentPalette root, tab content, ComponentsTab wrapper)
2. **Add `overflow-x-visible` to the scroll container** to allow horizontal overflow for tooltips
3. **Add `group relative` to the draggable item** so absolute tooltip positions relative to it
4. **Position tooltip to the right**: `absolute left-full top-0 ml-2` instead of `bottom-full left-1/2`

## Files Modified
- `src/components/palette/ComponentPalette.tsx`

## Key CSS Classes
```tsx
// ComponentPalette root - removed overflow-hidden
<div className="w-60 bg-surface border-r border-border flex flex-col shrink-0">

// Tab content - removed overflow-hidden
<div className="flex-1 flex flex-col">

// ComponentsTab wrapper - overflow-hidden kept for vertical, added overflow-x-visible
<div className="flex-1 flex flex-col overflow-hidden">
  <div className="flex-1 overflow-y-auto overflow-x-visible py-1">

// Draggable item - added group relative
<div className="... group relative">

// Tooltip - right side positioning
<div className="absolute left-full top-0 ml-2 hidden group-hover:block z-50">
```

## Lessons
- `overflow-hidden` on ANY ancestor clips absolute-positioned tooltips
- For left-sidebar palettes, tooltips must go RIGHT (left-full), not UP (bottom-full)
- `group-hover` only works if the hovered element has `group` class AND tooltip is a descendant