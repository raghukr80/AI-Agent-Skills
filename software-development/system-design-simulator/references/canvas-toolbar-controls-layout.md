# Canvas Toolbar + Controls Layout

The persistent challenge: placing a custom toolbar (Suggestions, Notes, Trace buttons) alongside ReactFlow's Controls (Zoom +/-, Fit View) at the bottom-right without overlap.

## The Problem

ReactFlow's `<Controls>` component uses internal `position: absolute` styling that ignores flexbox layout. Any attempt to place it inside a flex container with other elements causes overlap.

## Failed Approaches

1. **Nesting Controls inside a Panel with toolbar buttons** — Controls' absolute positioning ignores the flex layout, causing it to render behind/over the toolbar
2. **Two separate Panels at `position="bottom-right"`** — Both render at the same corner; `!mb-14` margin doesn't create space because both are absolutely positioned
3. **Single Panel with both, Controls inside flex-col** — Controls' internal absolute positioning breaks out of the flex flow

## Working Solution

**Do NOT use ReactFlow's `<Controls>` component.** Replace it with a custom `<ZoomButtons>` component that uses `useReactFlow()` hook. Both the toolbar and ZoomButtons live inside a single Panel, stacked vertically:

```tsx
<Panel position="bottom-right">
  <div className="flex flex-col gap-1">
    {/* Toolbar buttons */}
    <div className="flex flex-col gap-1 bg-surface border border-border rounded-lg shadow-lg p-1">
      <CanvasToolbarButtons ... />
    </div>
    {/* Zoom buttons */}
    <div className="bg-surface border border-border rounded-lg shadow-lg flex flex-col items-center justify-center gap-0.5 p-1">
      <ZoomButtons />
    </div>
  </div>
</Panel>
```

## Custom ZoomButtons Component

```tsx
function ZoomButtons() {
  const { zoomIn, zoomOut, fitView } = useReactFlow()
  return (
    <div className="flex flex-col gap-0.5">
      <button onClick={() => zoomIn()} className="p-1 rounded text-text-dim hover:text-text hover:bg-surface-hover" title="Zoom In">
        <svg className="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
          <circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35M11 8v6M8 11h6"/>
        </svg>
      </button>
      <button onClick={() => zoomOut()} className="p-1 rounded text-text-dim hover:text-text hover:bg-surface-hover" title="Zoom Out">
        <svg className="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
          <circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35M8 11h6"/>
        </svg>
      </button>
      <button onClick={() => fitView()} className="p-1 rounded text-text-dim hover:text-text hover:bg-surface-hover" title="Fit View">
        <svg className="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
          <path d="M15 3h6v6M9 21H3v-6M21 3l-7 7M3 21l7-7"/>
        </svg>
      </button>
    </div>
  )
}
```

## Key Takeaways

- ReactFlow `<Controls>` uses `position: absolute` internally — it cannot be placed in a flex container
- `useReactFlow()` hook provides `zoomIn()`, `zoomOut()`, `fitView()` methods for custom controls
- `Panel` from ReactFlow renders with `position: relative` — children with `position: absolute` break out
- Stack elements vertically with `flex flex-col gap-1` inside a single Panel
- Both toolbar and zoom buttons should be in bordered `bg-surface` containers for visual consistency
