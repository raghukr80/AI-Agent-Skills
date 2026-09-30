# Edge Arrows and Markers

## Pattern

All edges in the system design simulator should display an arrowhead at the target end. Users can toggle arrows per-edge via the Edge Properties Panel.

## Implementation

### Edge Creation (onConnect)

```tsx
const edge: Edge = {
  id: `edge_${params.source}_${params.target}_Date.now()}`,
  source: params.source!,
  target: params.target!,
  sourceHandle: params.sourceHandle,
  targetHandle: params.targetHandle,
  type: 'smoothstep',
  animated: store.simState === 'running',
  style: { stroke: 'var(--color-accent)', strokeWidth: 2 },
  markerEnd: {
    type: 'arrowclosed' as const,
    color: 'var(--color-accent)',
    width: 15,
    height: 15,
  },
  data: {
    label: '',
    protocol: 'HTTP',
    bandwidthMbps: 1000,
    latencyMs: 0,
    encrypted: false,
    showArrow: true,
  },
}
```

### Animated State (useEffect)

```tsx
setEdges(currentEdges =>
  currentEdges.map(e => {
    const showArrow = (e.data as Record<string, unknown>)?.showArrow !== false
    return {
      ...e,
      animated: store.simState === 'running',
      style: {
        ...e.style,
        stroke: store.simState === 'running' ? 'var(--color-accent)' : 'var(--color-border)',
        strokeWidth: store.simState === 'running' ? 2 : 1.5,
      },
      markerEnd: showArrow ? {
        type: 'arrowclosed' as const,
        color: store.simState === 'running' ? 'var(--color-accent)' : 'var(--color-border)',
        width: 15,
        height: 15,
      } : undefined,
    }
  })
)
```

### Initial Sync from Store

```tsx
setEdges(store.edges.map(e => ({
  id: e.id,
  source: e.source,
  target: e.target,
  type: 'smoothstep' as const,
  animated: false,
  style: { stroke: 'var(--color-accent)', strokeWidth: 2 },
  markerEnd: (e.data as Record<string, unknown>)?.showArrow !== false ? {
    type: 'arrowclosed' as const,
    color: 'var(--color-accent)',
    width: 15,
    height: 15,
  } : undefined,
  label: e.data?.label,
})))
```

### Edge Properties Panel Toggle

```tsx
const [showArrow, setShowArrow] = useState(Boolean(edgeData.showArrow !== false))

// In save handler:
const updatedData = { ..., showArrow }

// Toggle UI:
<div className="flex items-center justify-between">
  <span className="text-[10px] text-text-dim flex items-center gap-1">
    <ArrowRight className="w-3 h-3" /> Show Arrow
  </span>
  <button
    type="button"
    onClick={() => { setShowArrow(!showArrow); handleSave() }}
    className={`relative w-8 h-4 rounded-full transition-colors ${showArrow ? 'bg-accent' : 'bg-border'}`}
  >
    <div className={`absolute top-0.5 w-3 h-3 rounded-full bg-white transition-transform ${showArrow ? 'left-4.5' : 'left-0.5'}`} />
  </button>
</div>
```

## Key Points

- `markerEnd` goes at the edge level, NOT inside `style`
- Use `type: 'arrowclosed' as const` for TypeScript literal type
- Default to `showArrow: true` for new edges
- Check `(e.data as Record<string, unknown>)?.showArrow !== false` to handle edges created before the field existed
- Arrow color should match the current edge stroke color
