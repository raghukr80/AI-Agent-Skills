# Controls Panel Overlap Fix

## Problem
ReactFlow's `<Controls>` component (Zoom +/-, Fit View) uses internal `position: absolute` styling that cannot be overridden. When placed inside a flex container alongside other elements (like toolbar buttons), it overlaps them regardless of margin/padding.

## Root Cause
The Controls component renders its own absolutely-positioned container. The `position` prop (`"bottom-right"`) only affects which corner it anchors to, but the component itself ignores parent flex layout.

## Solution
**Replace the Controls component with a custom ZoomButtons component** that uses `useReactFlow()` hook:

```tsx
function ZoomButtons() {
  const { zoomIn, zoomOut, fitView } = useReactFlow()
  return (
    <>
      <button onClick={() => zoomIn()} className="p-1.5 rounded text-text-dim hover:text-text hover:bg-surface-hover" title="Zoom In">
        <svg className="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
          <circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35M11 8v6M8 11h6"/>
        </svg>
      </button>
      <button onClick={() => zoomOut()} className="p-1.5 rounded text-text-dim hover:text-text hover:bg-surface-hover" title="Zoom Out">
        <svg className="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
          <circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35M8 11h6"/>
        </svg>
      </button>
      <button onClick={() => fitView()} className="p-1.5 rounded text-text-dim hover:text-text hover:bg-surface-hover" title="Fit View">
        <svg className="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
          <path d="M15 3h6v6M9 21H3v-6M21 3l-7 7M3 21l7-7"/>
        </svg>
      </button>
    </>
  )
}
```

## Working Layout Structure
```tsx
<Panel position="bottom-right">
  <div className="flex flex-col gap-1">
    {/* Toolbar buttons on top */}
    <div className="flex flex-col gap-1 bg-surface border border-border rounded-lg shadow-lg p-1">
      <CanvasToolbarButtons ... />
    </div>
    {/* Zoom buttons below — regular DOM, not Controls component */}
    <div className="bg-surface border border-border rounded-lg shadow-lg flex items-center justify-center gap-0.5 p-1">
      <ZoomButtons />
    </div>
  </div>
</Panel>
```

## Key Points
- Both toolbar and zoom buttons are in a single Panel with `flex flex-col gap-1`
- No absolute positioning conflicts because all elements are regular DOM
- `useReactFlow()` hook provides `zoomIn()`, `zoomOut()`, `fitView()` functions
- Import: `import { useReactFlow } from '@xyflow/react'`
- Remove the `Controls` import entirely when using custom ZoomButtons
