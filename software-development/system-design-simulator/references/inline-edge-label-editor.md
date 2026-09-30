# Inline Edge Label Editor

Double-click on any connector to add a custom label directly on the canvas. No browser popups.

## Implementation

### State
```tsx
const [editingEdgeId, setEditingEdgeId] = useState<string | null>(null)
const [editingLabel, setEditingLabel] = useState('')
const [editingPos, setEditingPos] = useState({ x: 0, y: 0 })
const editInputRef = useRef<HTMLInputElement>(null)
```

### Double-click handler
```tsx
const onEdgeDoubleClick = useCallback((_e: React.MouseEvent, edge: Edge) => {
  const rect = (_e.target as HTMLElement).getBoundingClientRect()
  setEditingEdgeId(edge.id)
  setEditingLabel((edge.data as Record<string, unknown>)?.label as string || '')
  setEditingPos({ x: rect.left + rect.width / 2, y: rect.top + rect.height / 2 })
}, [])
```

### Auto-focus
```tsx
useEffect(() => {
  if (editingEdgeId && editInputRef.current) {
    editInputRef.current.focus()
    editInputRef.current.select()
  }
}, [editingEdgeId])
```

### Commit/cancel
```tsx
const commitLabel = useCallback(() => {
  if (!editingEdgeId) return
  const newLabel = editingLabel.trim()
  setEdges(eds => eds.map(ed =>
    ed.id === editingEdgeId
      ? { ...ed, label: newLabel, data: { ...ed.data, label: newLabel } }
      : ed
  ))
  store.setEdges(store.edges.map((se: any) =>
    se.id === editingEdgeId ? { ...se, data: { ...se.data, label: newLabel } } : se
  ))
  setEditingEdgeId(null)
  setEditingLabel('')
}, [editingEdgeId, editingLabel, setEdges, store])
```

### Inline input overlay
```tsx
{editingEdgeId && (
  <div
    className="fixed z-[100] pointer-events-auto"
    style={{ left: editingPos.x, top: editingPos.y, transform: 'translate(-50%, -50%)' }}
  >
    <input
      ref={editInputRef}
      value={editingLabel}
      onChange={e => setEditingLabel(e.target.value)}
      onBlur={commitLabel}
      onKeyDown={e => {
        if (e.key === 'Enter') commitLabel()
        if (e.key === 'Escape') { setEditingEdgeId(null); setEditingLabel('') }
      }}
      placeholder="Label"
      className="px-2 py-1 text-[11px] text-text bg-surface border border-accent rounded shadow-lg outline-none w-32 text-center"
      maxLength={40}
    />
  </div>
)}
```

### Key points
- Use `onEdgeDoubleClick` ReactFlow handler
- Position input at the edge center using `getBoundingClientRect()`
- Use `fixed` positioning with `z-[100]` to overlay the canvas
- `pointer-events-auto` on the input so it receives clicks
- Save to both ReactFlow edges AND Zustand store
- No default labels — edges are clean by default
