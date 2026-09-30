# Draggable Window Pattern: Top Positioning with Delta Movement

## Problem
Making a fixed-position panel draggable seems simple but has a gotcha: if you use `bottom` CSS positioning, the drag direction is inverted vertically (mouse up → panel goes down).

## Root Cause
With `position: fixed; bottom: Ypx`, the `bottom` value represents distance from the viewport bottom. When you move the mouse up, you want the panel to move up, which means `bottom` must INCREASE. But naive implementations use `e.clientY - offset` which decreases, causing inversion.

## Solution: Use `top` Positioning + Delta Movement

```tsx
const [position, setPosition] = useState({ x: 16, y: 0 })
const [dragging, setDragging] = useState(false)
const dragStart = useRef({ mouseX: 0, mouseY: 0, posX: 0, posY: 0 })

// Initialize at bottom-left on mount
useEffect(() => {
  setPosition({ x: 16, y: window.innerHeight - 280 }) // 280 = max panel height
}, [])

const handleMouseDown = useCallback((e: React.MouseEvent) => {
  if ((e.target as HTMLElement).closest('button')) return // exclude buttons
  setDragging(true)
  dragStart.current = {
    mouseX: e.clientX,
    mouseY: e.clientY,
    posX: position.x,
    posY: position.y,
  }
}, [position])

useEffect(() => {
  if (!dragging) return
  const handleMouseMove = (e: MouseEvent) => {
    const dx = e.clientX - dragStart.current.mouseX
    const dy = e.clientY - dragStart.current.mouseY
    const newX = Math.max(0, Math.min(dragStart.current.posX + dx, window.innerWidth - panelWidth))
    const newY = Math.max(0, Math.min(dragStart.current.posY + dy, window.innerHeight - minVisibleHeight))
    setPosition({ x: newX, y: newY })
  }
  const handleMouseUp = () => setDragging(false)
  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('mouseup', handleMouseUp)
  return () => {
    window.removeEventListener('mousemove', handleMouseMove)
    window.removeEventListener('mouseup', handleMouseUp)
  }
}, [dragging])
```

### Rendering
```tsx
<div
  className="fixed z-20 w-[380px] max-h-[280px] bg-surface/95 border border-border rounded-lg shadow-2xl backdrop-blur-sm flex flex-col overflow-hidden"
  style={{ left: `${position.x}px`, top: `${position.y}px` }}
>
  {/* Header with grip icon for drag handle */}
</div>
```

## Key Rules
1. **Use `top` positioning** — NOT `bottom` (causes inversion) or `right` (same issue horizontally)
2. **Delta-based movement** — Store initial position + mouse position on mousedown, then `newPos = startPos + (currentMouse - startMouse)`
3. **Clamp to viewport** — `Math.max(0, Math.min(newX, window.innerWidth - panelWidth))` and same for Y with `window.innerHeight - minVisible` (use ~40px minVisible, NOT panel height, so user can drag panel to very bottom edge)
4. **Initialize at bottom** — `window.innerHeight - panelMaxHeight` on mount for bottom-left default position
5. **Global window listeners** — Use `window.addEventListener('mousemove')` not element events, so dragging continues even if mouse leaves the panel
6. **Filter button clicks** — Check `e.target.closest('button')` to prevent drag initiation when clicking buttons inside the header
7. **Grip cursor** — Add `cursor-grab` to header, `cursor-grabbing` while dragging, and a grip icon (⠿) for visual affordance

## Common Mistakes
- Using `bottom` positioning → inverted vertical drag
- Using absolute mouse position instead of delta → panel jumps to mouse position
- Clamping with `window.innerHeight - panelHeight` → can't drag to very bottom (leaves gap). Use `window.innerHeight - 40` instead so only 40px needs to be visible
- Not filtering button clicks → buttons don't work because drag starts first
