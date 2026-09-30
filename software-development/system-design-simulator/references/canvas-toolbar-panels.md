# Canvas Toolbar Panels (Suggestions, Notes, Trace)

Floating toolbar on the right side of the canvas with three action buttons and associated panels.

## Layout

```
┌─────────────────────────────────────┐
│  [Suggestions] [Notes] [Trace] [↺]  │  ← Toolbar (bottom-right, above Controls)
└─────────────────────────────────────┘
         ↓ (click Suggestions)
┌──────────────────────┐
│ 💡 Suggestions       │  ← Panel opens to the left of toolbar
│ • Isolated Nodes     │
│ • High Traffic Nodes │
│ • Missing LB         │
│ • No Data Layer      │
└──────────────────────┘
```

## Toolbar Positioning

- Use a **separate `<Panel>`** positioned at `bottom-right` with `className="!mb-14"` (margin-bottom pushes it above the Controls panel)
- **CRITICAL: Render Controls FIRST, then toolbar Panel AFTER in the JSX** — both use `position="bottom-right"`, and the toolbar's `!mb-14` margin pushes it above
- Do NOT nest inside the `<Controls>` component — it renders children horizontally
- The toolbar container is a vertical flex column: `flex flex-col gap-1` inside a bordered/surface container

```tsx
{/* Controls renders first (at bottom-right) */}
<Controls position="bottom-right" showZoom={true} showFitView={true} ... />
{/* Toolbar renders second, positioned above via !mb-14 */}
<Panel position="bottom-right" className="!mb-14">
  <div className="flex flex-col gap-1 bg-surface border border-border rounded-lg shadow-lg p-1">
    <CanvasToolbarButtons ... />
  </div>
</Panel>
```

## Suggestions Panel

- Uses `fixed` positioning: `right-16 bottom-28` (appears to the left of the toolbar)
- `z-40` to sit above other elements
- Auto-generates recommendations based on architecture analysis:
  - Isolated nodes (no connections)
  - High traffic nodes (4+ connections)
  - Missing load balancer
  - No data layer
  - Possible circular dependencies
- Each suggestion has severity (info/warning/error) with corresponding border/bg colors

## Notes Panel

- Same positioning as Suggestions
- Textarea for persistent architecture notes
- Saves to `store.architectureNotes` via `store.setArchitectureNotes()`
- "Saved!" feedback on save

## Trace Request Flow

- Click nodes in sequence to build path
- Validates edge exists between consecutive nodes
- Highlights traced edges in green with numbered badges
- Breadcrumb at top-center shows the flow path
- Reset button clears trace

## State Management

```tsx
const [toolbarMode, setToolbarMode] = useState<string | null>(null)
```

- `null` — no panel open
- `'suggestions'` — suggestions panel visible
- `'notes'` — notes panel visible
- Trace is managed via the store (`traceMode`, `tracePath`)

## Key Pitfalls

1. **Do NOT use `!mt-14`** — this pushes the panel ABOVE the Controls but can cause overlap. Use `!mb-14` instead (margin-bottom pushes it up from the bottom)
2. **Panels must use `fixed` positioning** — not `absolute` — so they don't scroll with the canvas
3. **Toolbar buttons must be `flex-col`** — vertical layout matches the Controls panel aesthetic
4. **Do NOT put toolbar inside Controls component** — it renders children horizontally
