# Trace Request Flow

Click nodes in sequence to trace the request flow. Edges highlight in green with numbered badges.

## Store additions
```tsx
traceMode: boolean
tracePath: string[]
traceEdgeNumbers: Record<string, number>  // edgeId -> sequence number
setTraceMode: (mode: boolean) => void
addTraceNode: (nodeId: string) => void
clearTrace: () => void
```

## addTraceNode logic
```tsx
addTraceNode: (nodeId) => {
  const path = [...get().tracePath]
  if (path.length === 0) {
    path.push(nodeId)
  } else {
    const lastNode = path[path.length - 1]
    if (nodeId === lastNode) return
    const edge = get().edges.find((e: any) =>
      (e.source === lastNode && e.target === nodeId) ||
      (e.source === nodeId && e.target === lastNode)
    )
    if (!edge) return  // no edge between nodes
    path.push(nodeId)
    const numbers = { ...get().traceEdgeNumbers }
    numbers[edge.id] = path.length - 1
    set({ tracePath: path, traceEdgeNumbers: numbers })
    return
  }
  set({ tracePath: path })
}
```

## Edge rendering (in animated state useEffect)
```tsx
const traceNum = traceEdgeNumbers[e.id]
const isTraced = traceNum !== undefined
return {
  ...e,
  animated: store.simState === 'running' || isTraced,
  style: {
    ...e.style,
    stroke: isTraced ? 'var(--color-success)' : ...,
    strokeWidth: isTraced ? 3 : ...,
  },
  markerEnd: showArrow ? {
    ...,
    color: isTraced ? 'var(--color-success)' : ...,
    width: isTraced ? 18 : 15,
    height: isTraced ? 18 : 15,
  } : undefined,
  label: isTraced ? ` ${traceNum} ` : (e.data?.label || ''),
  data: { ...e.data, label: isTraced ? ` ${traceNum} ` : (e.data?.label || '') },
}
```

## Node click handler
```tsx
const onNodeClickHandler = useCallback((_e, node) => {
  if (isTraceMode) addTraceNode(node.id)
}, [isTraceMode, addTraceNode])
```

## Breadcrumb UI
```tsx
{tracePath.map((nodeId, i) => {
  const node = store.nodes.find(n => n.id === nodeId)
  const label = node?.data.label || nodeId
  return (
    <div key={i} className="flex items-center gap-1 shrink-0">
      <span className="flex items-center justify-center w-5 h-5 rounded-full bg-accent text-white text-[10px] font-bold shrink-0">
        {i + 1}
      </span>
      <span className={`text-[10px] font-medium ${i === 0 ? 'text-accent' : i === tracePath.length - 1 ? 'text-success' : 'text-text'}`}>
        {label}
      </span>
      {i < tracePath.length - 1 && <ChevronRight className="w-3 h-3 text-text-dim" />}
    </div>
  )
})}
```

## Key points
- Validate edge exists between consecutive nodes
- Assign sequential numbers (1, 2, 3...) to edges in the path
- Green color for traced edges, larger stroke and arrow
- Breadcrumb at top with node names and numbered circles
- Reset button clears tracePath and traceEdgeNumbers

## Session 2026-08-18 Fix: Button Toggle & Sequential Numbering

### Problem
The "Trace Request Flow" button in the CanvasToolbar was non-functional:
1. **No click handler** — button was static markup with no onClick
2. **Zero-based numbering** — breadcrumb showed `{i}` instead of `{i + 1}`
3. **Conditional numbering** — only showed numbers for `i > 0`, skipping the first step
4. **No state toggle** — button didn't switch between trace mode and clear trace

### Fix Applied in `CanvasToolbar.tsx`

#### 1. Button Toggle Logic (Lines 52-68)
```tsx
<button
  onClick={() => {
    if (!isTraceActive) {
      store.setTraceMode(true)
    } else {
      store.clearTrace()
    }
  }}
  className={`p-1.5 rounded transition-all ${
    isTraceActive
      ? 'bg-accent text-white'
      : 'text-text-dim hover:text-text hover:bg-surface-hover'
  }`}
  title={isTraceActive ? 'Clear Trace' : 'Trace Request Flow'}
>
  <Route className="w-3.5 h-3.5" />
</button>
```

#### 2. Sequential Numbering (Lines 88-90, 130-132)
```tsx
// BEFORE (broken):
{i > 0 && (
  <span className="w-4 h-4 text-[8px]">{i}</span>
)}

// AFTER (fixed):
<span className="w-5 h-5 text-[10px] font-bold">{i + 1}</span>
```

#### 3. Updated Both Components
- `CanvasToolbarButtons` — inline breadcrumb at top (Lines 80-101)
- `TraceBreadcrumb` — exported standalone component (Lines 124-143)

### Verification
```bash
yarn workspace web build   # ✅ PASSED
npx tsc --noEmit          # ✅ PASSED
```