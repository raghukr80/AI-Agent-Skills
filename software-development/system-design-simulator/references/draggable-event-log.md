# Draggable Panel Pattern: Delta-Based Movement with `top` Positioning

## Problem
Making a panel draggable seems simple but has multiple pitfalls:
- Using `bottom` positioning causes inverted vertical movement
- Absolute positioning from mouse offset causes jitter
- Clamping prevents placing panel at screen edges

## Solution: Delta-Based Movement + `top` Positioning

### Core Pattern
```tsx
const [position, setPosition] = useState({ x: 16, y: 0 })
const [dragging, setDragging] = useState(false)
const dragStart = useRef({ mouseX: 0, mouseY: 0, posX: 0, posY: 0 })

// Initialize at bottom-left
useEffect(() => {
  setPosition({ x: 16, y: window.innerHeight - 280 })
}, [])

// Drag start
const handleMouseDown = useCallback((e: React.MouseEvent) => {
  if ((e.target as HTMLElement).closest('button')) return
  setDragging(true)
  dragStart.current = {
    mouseX: e.clientX,
    mouseY: e.clientY,
    posX: position.x,
    posY: position.y,
  }
}, [position])

// Drag move
useEffect(() => {
  if (!dragging) return
  const handleMouseMove = (e: MouseEvent) => {
    const dx = e.clientX - dragStart.current.mouseX
    const dy = e.clientY - dragStart.current.mouseY
    const newX = Math.max(0, Math.min(dragStart.current.posX + dx, window.innerWidth - 380))
    const newY = Math.max(0, Math.min(dragStart.current.posY + dy, window.innerHeight - 280))
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
  {/* Header with drag handle */}
  <div
    className={`flex items-center justify-between px-3 py-2 border-b border-border shrink-0 select-none ${dragging ? 'cursor-grabbing' : 'cursor-grab'}`}
    onMouseDown={handleMouseDown}
  >
    <GripVertical className="w-3 h-3 text-text-dim" />
    {/* ... header content ... */}
  </div>
</div>
```

### Clamping Rules
- **Horizontal**: `Math.max(0, Math.min(posX + dx, window.innerWidth - 380))` — 380 = panel width
- **Vertical**: `Math.max(0, Math.min(posY + dy, window.innerHeight - 280))` — 280 = panel max-height
- **Initial position**: `window.innerHeight - 280` places panel at bottom

### Key Decisions
1. **Use `top` not `bottom`** — `bottom` requires inverting dy (`posY - dy`), which is confusing and error-prone
2. **Delta-based not absolute** — Store start position + mouse delta, not mouse-to-element offset. Smoother.
3. **Global window listeners** — Mouse may leave the panel during drag. Window-level listeners ensure smooth tracking.
4. **Clamp to keep at least 40px visible** — Don't allow the panel to go fully off-screen
5. **Grip cursor on header** — `cursor-grab` on hover, `cursor-grabbing` while dragging

### Common Mistakes
- Using `bottom: Ypx` with `posY + dy` → inverted movement. Either use `top` or invert: `posY - dy`
- Clamping with wrong window size — if panel is `max-h-[280px]` but actual content is shorter, the clamp still reserves 280px, preventing placement at very bottom
- Forgetting `select-none` on header — text selection may trigger during drag
- Not filtering button clicks — header buttons (close, clear) should not initiate drag
