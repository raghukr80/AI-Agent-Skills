# Layout Zones — Implementation Reference

## Concept
Layout zones are visual grouping containers rendered as absolutely-positioned divs alongside the ReactFlow canvas. They are NOT ReactFlow nodes. Components can be dropped on top of them and moved normally.

## Key Architecture Decision: pointerEvents
The **individual zone box** uses `pointerEvents: 'none'` so mouse events pass through to ReactFlow nodes underneath. Only specific sub-elements override this via inline `style={{ pointerEvents: 'auto' }}`:
- **Header bar**: `pointerEvents: 'auto'` — for dragging the zone
- **Resize handles**: `pointerEvents: 'auto'` — for resizing
- **Background/border/label**: No inline style needed — inherits `pointerEvents: 'none'` from parent

**CRITICAL**: The renderer wrapper div should NOT have `pointerEvents: 'none'` — each zone box manages its own pointer events. The renderer is just a container.

## Render Position
Render zones inside `reactFlowWrapper` div, AFTER the `<ReactFlow>` component and `<ParticleCanvas>`:
```tsx
<div ref={reactFlowWrapper} className="flex-1 relative">
  <ReactFlow ...>...</ReactFlow>
  <ParticleCanvas />
  <LayoutZonesRenderer ... />
</div>
```

**DEADLY**: Rendering zones BEFORE ReactFlow in the DOM — ReactFlow sits on top and hides zones from interaction. Always render zones AFTER ReactFlow.

## Drag Pattern
Each zone manages its own drag state. Header `onMouseDown` → `window.addEventListener('mousemove')` → `onUpdate({ x, y })`. On `window.mouseup`, clear drag state. Store start position in `useRef`.

## Resize Pattern
Same per-zone global mouse event pattern. Edge handles (n/s/e/w) are 2px thick spanning full edge minus 4px inset. Corner handles are 5×5px. All have `pointerEvents: 'auto'` and `hover:bg-accent/20` feedback. For `n` and `w` edges, adjust both size AND position.

## Common Mistakes
1. Putting `pointerEvents: 'none'` on the renderer wrapper — prevents ALL zone interaction
2. Using local `onMouseMove` instead of `window.addEventListener` — drag/resize stops when mouse leaves element
3. Registering zones as ReactFlow node types — they should be completely separate
4. Not using global mouseup listener — drag state never clears
5. Forgetting `e.preventDefault()` in handleHeaderMouseDown — can trigger unwanted text selection
6. Rendering zones BEFORE ReactFlow in the DOM — zones become invisible to interaction
