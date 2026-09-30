# Editable Nodes: Labels, Tags, Colors, and Icons

## Pattern: Inline Label Editing on Canvas

Allow users to double-click a node's label to edit it inline:

```tsx
const [editing, setEditing] = useState(false)
const [editValue, setEditValue] = useState(data.label)

const handleDoubleClick = (e: React.MouseEvent) => {
  e.stopPropagation()
  setEditing(true)
  setEditValue(data.label)
}

const handleBlur = () => {
  setEditing(false)
  if (editValue.trim() && editValue !== data.label) {
    store.updateNodeLabel(id, editValue.trim())
  }
}
```

Use `e.stopPropagation()` to prevent ReactFlow selection. Use `onClick={(e) => e.stopPropagation()}` on the input to prevent drag. Sync `editValue` with `data.label` via `useEffect` for external changes.

## Pattern: Node Tag

Tags are short labels displayed as colored badges (`data.tag`). In PropertiesPanel, provide inline editing with "+ Add tag" placeholder. Badge color matches node's custom color: `style={{ backgroundColor: nodeColor + '20', color: nodeColor }}`.

## Pattern: Custom Node Color

Per-node color for the left border: `const nodeColor = data.color || '#6366f1'`. Use color picker with 12 preset colors in PropertiesPanel. Store via `store.updateNodeColor(nodeId, color)`.

## Pattern: Custom Node Icon (Emoji + Local SVG)

**Store the icon value** in `SimNode.data.icon`. The value can be either:
- An **emoji string** (e.g. `"☁️"`, `"🗄️"`, `"⚡"`) — used for built-in component icons
- A **URL path** (e.g. `"/icons/aws/Compute/ec2.svg"`) — used for custom SVG icons from the icon picker

### Rendering Logic (CRITICAL)

```tsx
// Distinguish emoji from URL: URLs start with '/'
const iconUrl = data.icon && data.icon.startsWith('/') ? data.icon : null
const emojiIcon = data.icon && !data.icon.startsWith('/') ? data.icon : null

return (
  <div>
    {iconUrl ? (
      <img src={iconUrl} alt="" className="w-4 h-4 shrink-0" />
    ) : emojiIcon ? (
      <span className="text-sm shrink-0">{emojiIcon}</span>
    ) : (
      <span className="text-sm shrink-0">⚙️</span>
    )}
    ...
  </div>
)
```

**⚠️ PITFALL:** If you use `const iconUrl = data.icon || null` (treating all icons as URLs), emoji icons will break — they'll try to load as image paths and show broken images. Always check `startsWith('/')` to distinguish URL from emoji.

### Why NOT `import.meta.glob` or CDN

- **Devicon CDN** requires internet, has CORS issues, and depends on external availability
- **`import.meta.glob('/public/icons/**/*.svg')` fails** with special characters in filenames (`#`, `()`, spaces) and absolute `/public/` path resolution issues in Vite
- **`import.meta.glob` with relative paths** from `src/data/` to `public/` is fragile across different file locations

### Build-Time Manifest Generation (Recommended)

Use a Node.js `.cjs` script (`.cjs` extension required when `package.json` has `"type": "module"`):

1. **Script**: `scripts/build-icon-manifest.cjs` scans `public/icons/` recursively
2. **Output**: Generates `src/data/icon-manifest.ts` with all icon metadata
3. **Hook into package.json**:
```json
{
  "build:icons": "node scripts/build-icon-manifest.cjs",
  "dev": "node scripts/build-icon-manifest.cjs && vite",
  "build": "node scripts/build-icon-manifest.cjs && tsc -b && vite build"
}
```

### Manifest Format

```typescript
// src/data/icon-manifest.ts (auto-generated)
import type { IconDef } from './icons'

export const ICON_CATEGORIES = [
  { key: 'all', label: 'all' },
  { key: 'aws', label: 'aws' },
  // auto-detected from folder names
] as const

export const ICON_MANIFEST: IconDef[] = [
  { id: "aws-Compute-ec2", name: "EC2", category: "aws", url: "/icons/aws/Compute/ec2.svg" },
  // ...
]
```

### Icon Data Module

```typescript
// src/data/icons.ts
export interface IconDef { id: string; name: string; category: string; url: string }
import { ICON_CATEGORIES as _CATS, ICON_MANIFEST } from './icon-manifest'
export const ICON_CATEGORIES = _CATS
export const ICONS: IconDef[] = [...ICON_MANIFEST]
export function searchIcons(query: string): IconDef[] { ... }
export function getIconsByCategory(category: string): IconDef[] { ... }
```

### Filename → Display Name

Build script converts filenames to display names: `ec2.svg` → "EC2", `api-gateway.svg` → "Api Gateway". Maintain an ACRONYMS map for proper casing. Sanitize IDs: remove `#`, `()`, `+`, `&`, `@`, spaces → `-`.

### Folder Structure

```
public/icons/
├── aws/           → subfolders: Compute/, Database/, Storage/, etc.
├── azure/         → subfolders: compute/, databases/, networking/, etc.
├── gcp/
├── tech/
├── database/
├── networking/
├── security/
├── devtools/
├── containers/
├── ai/
└── general/
```

## ⚠️ CRITICAL: Dual-Write Pattern for Instant UI Updates

**Problem:** Calling `store.updateNodeColor()` (or label/tag/icon) only updates the Zustand store. The store→ReactFlow sync only runs during simulation (`store.simState === 'running'`). So when the user is editing properties with simulation stopped, the canvas node never updates.

**Solution:** The PropertiesPanel must receive ReactFlow's `setNodes` function and update BOTH the store AND ReactFlow simultaneously:

```tsx
// In SimulatorCanvas — pass setNodes to PropertiesPanel
<PropertiesPanel setNodes={setNodes} />

// In PropertiesPanel — accept the prop
export function PropertiesPanel({ setNodes }: { setNodes: (fn: (nodes: Node[]) => Node[]) => void }) {
  const store = useDiagramStore()

  // Dual-write helper: updates store AND ReactFlow for instant feedback
  const updateNodeData = (patch: Partial<SimNode['data']>) => {
    // Update store (for simulation reads)
    if (patch.label !== undefined) store.updateNodeLabel(selectedNode.id, patch.label)
    if (patch.tag !== undefined) store.updateNodeTag(selectedNode.id, patch.tag)
    if (patch.color !== undefined) store.updateNodeColor(selectedNode.id, patch.color)
    if (patch.icon !== undefined) store.updateNodeIcon(selectedNode.id, patch.icon)
    // Update ReactFlow directly (for instant canvas feedback)
    setNodes(currentNodes =>
      currentNodes.map(n =>
        n.id === selectedNode.id
          ? { ...n, data: { ...n.data, ...patch } }
          : n
      )
    )
  }
}
```

**Why both?** The store is the simulation source of truth. ReactFlow is the rendering source of truth. They must be kept in sync for these user-driven property changes.

## Store Actions

```tsx
updateNodeLabel: (nodeId, label) => {
  set({ nodes: get().nodes.map(n => n.id === nodeId ? { ...n, data: { ...n.data, label } } : n) })
},
updateNodeTag: (nodeId, tag) => {
  set({ nodes: get().nodes.map(n => n.id === nodeId ? { ...n, data: { ...n.data, tag } } : n) })
},
updateNodeColor: (nodeId, color) => {
  set({ nodes: get().nodes.map(n => n.id === nodeId ? { ...n, data: { ...n.data, color } } : n) })
},
updateNodeIcon: (nodeId, icon) => {
  set({ nodes: get().nodes.map(n => n.id === nodeId ? { ...n, data: { ...n.data, icon } } : n) })
},
```

## Type Updates

Add to `SimNode.data`:
```typescript
tag?: string
color?: string
icon?: string   // public URL path to local SVG, e.g. "/icons/aws/Compute/ec2.svg"
```

## PropertiesPanel Identity Section

First section when selecting a node — Label, Tag, Color, Icon — before configuration sliders. Place at the very top of the panel. Each field has inline editing or a picker button.

## Color Picker UI

Show a grid of 12 preset color swatches. The selected one gets a ring highlight. Clicking a color calls `updateNodeData({ color })` and closes the picker.

## Icon Picker UI (Inline Dropdown — NOT Modal)

**⚠️ Do NOT use a modal window for icon selection.** Users rejected this approach. Use an inline dropdown within the PropertiesPanel.

### Implementation

```tsx
// In PropertiesPanel — inline icon search dropdown
const [iconSearch, setIconSearch] = useState('')
const [iconDropdownOpen, setIconDropdownOpen] = useState(false)
const iconSearchRef = useRef<HTMLDivElement>(null)

// Close dropdown when clicking outside
useEffect(() => {
  const handler = (e: MouseEvent) => {
    if (iconSearchRef.current && !iconSearchRef.current.contains(e.target as globalThis.Node)) {
      setIconDropdownOpen(false)
    }
  }
  document.addEventListener('mousedown', handler)
  return () => document.removeEventListener('mousedown', handler)
}, [])

const filteredIcons = useMemo(() => {
  if (!iconSearch.trim()) return ICONS.slice(0, 50) // Show first 50 when no search
  return searchIcons(iconSearch).slice(0, 100) // Limit to 100 results
}, [iconSearch])
```

### Dropdown Layout

```tsx
<div ref={iconSearchRef} className="relative">
  <button onClick={() => setIconDropdownOpen(!iconDropdownOpen)}>
    {iconUrl ? <img src={iconUrl} /> : <Image icon />}
    <span>{iconUrl ? 'Change icon' : 'Select icon'}</span>
    <ChevronDown />
  </button>

  {iconDropdownOpen && (
    <div className="absolute left-0 right-0 top-full mt-1 z-50 bg-surface border rounded-lg shadow-2xl" style={{ maxHeight: '280px' }}>
      {/* Search input */}
      <div className="p-2 border-b">
        <input value={iconSearch} onChange={(e) => setIconSearch(e.target.value)} placeholder="Search icons..." />
        {iconSearch && <span>{filteredIcons.length} found</span>}
      </div>
      {/* Icon grid */}
      <div className="overflow-y-auto p-1.5" style={{ maxHeight: '200px' }}>
        <div className="grid grid-cols-7 gap-1">
          {/* Clear icon option */}
          <button onClick={() => selectIcon(null)}><X /></button>
          {filteredIcons.map(icon => (
            <button key={icon.id} onClick={() => selectIcon(icon)}>
              <img src={icon.url} alt={icon.name} />
            </button>
          ))}
        </div>
      </div>
    </div>
  )}
</div>
```

### Key Details

- **Search**: Filters by name, category, ID, URL. Supports fuzzy multi-word matching. Sorts by relevance (starts-with → contains → other).
- **Result limit**: 50 when no search, 100 when searching (performance with 1600+ icons)
- **Clear option**: X button to remove current icon
- **Click outside to close**: Use `useEffect` with `mousedown` listener
- **Result count**: Show "X found" below search input when searching
- **No modal**: This is critical — users explicitly rejected the modal approach

### Search Function

```typescript
export function searchIcons(query: string): IconDef[] {
  const q = query.toLowerCase().trim()
  if (!q) return ICONS
  return ICONS.filter(icon => {
    const name = icon.name.toLowerCase()
    const cat = icon.category.toLowerCase()
    const id = icon.id.toLowerCase()
    if (name.includes(q) || id.includes(q) || cat.includes(q)) return true
    // Fuzzy: match individual words
    const words = q.split(/\s+/)
    return words.every(w => name.includes(w) || id.includes(w) || cat.includes(w))
  }).sort((a, b) => {
    // Sort: name starts with query first, then contains, then others
    const aName = a.name.toLowerCase()
    const bName = b.name.toLowerCase()
    const aStarts = aName.startsWith(q) ? 0 : aName.includes(q) ? 1 : 2
    const bStarts = bName.startsWith(q) ? 0 : bName.includes(q) ? 1 : 2
    if (aStarts !== bStarts) return aStarts - bStarts
    return aName.localeCompare(bName)
  })
}
```

## Pattern: Inline Label Editing on Canvas

Allow users to double-click a node's label to edit it inline:

```tsx
const [editing, setEditing] = useState(false)
const [editValue, setEditValue] = useState(data.label)

const handleDoubleClick = (e: React.MouseEvent) => {
  e.stopPropagation()
  setEditing(true)
  setEditValue(data.label)
}

const handleBlur = () => {
  setEditing(false)
  if (editValue.trim() && editValue !== data.label) {
    store.updateNodeLabel(id, editValue.trim())
  }
}
```

Use `e.stopPropagation()` to prevent ReactFlow selection. Use `onClick={(e) => e.stopPropagation()}` on the input to prevent drag. Sync `editValue` with `data.label` via `useEffect` for external changes.

## Pattern: Node Tag

Tags are short labels displayed as colored badges (`data.tag`). In PropertiesPanel, provide inline editing with "+ Add tag" placeholder. Badge color matches node's custom color: `style={{ backgroundColor: nodeColor + '20', color: nodeColor }}`.

## Pattern: Custom Node Color

Per-node color for the left border: `const nodeColor = data.color || '#6366f1'`. Use color picker with 12 preset colors in PropertiesPanel. Store via `store.updateNodeColor(nodeId, color)`.

## ⚠️ CRITICAL: Dual-Write Pattern for Instant UI Updates

**Problem:** Calling `store.updateNodeColor()` (or label/tag) only updates the Zustand store. The store→ReactFlow sync only runs during simulation (`store.simState === 'running'`). So when the user is editing properties with simulation stopped, the canvas node never updates.

**Solution:** The PropertiesPanel must receive ReactFlow's `setNodes` function and update BOTH the store AND ReactFlow simultaneously:

```tsx
// In SimulatorCanvas — pass setNodes to PropertiesPanel
<PropertiesPanel setNodes={setNodes} />

// In PropertiesPanel — accept the prop
export function PropertiesPanel({ setNodes }: { setNodes: (fn: (nodes: Node[]) => Node[]) => void }) {
  const store = useDiagramStore()

  // Dual-write helper: updates store AND ReactFlow for instant feedback
  const updateNodeData = (patch: Partial<SimNode['data']>) => {
    // Update store (for simulation reads)
    if (patch.label !== undefined) store.updateNodeLabel(selectedNode.id, patch.label)
    if (patch.tag !== undefined) store.updateNodeTag(selectedNode.id, patch.tag)
    if (patch.color !== undefined) store.updateNodeColor(selectedNode.id, patch.color)
    // Update ReactFlow directly (for instant canvas feedback)
    setNodes(currentNodes =>
      currentNodes.map(n =>
        n.id === selectedNode.id
          ? { ...n, data: { ...n.data, ...patch } }
          : n
      )
    )
  }

  // Use it everywhere:
  const saveLabel = () => updateNodeData({ label: labelValue.trim() })
  const saveTag = () => updateNodeData({ tag: tagValue.trim() })
  // In color picker: onClick={() => updateNodeData({ color: c.value })}
}
```

**Why both?** The store is the simulation source of truth. ReactFlow is the rendering source of truth. They must be kept in sync for these user-driven property changes.

## Store Actions

```tsx
updateNodeLabel: (nodeId, label) => {
  set({ nodes: get().nodes.map(n => n.id === nodeId ? { ...n, data: { ...n.data, label } } : n) })
},
updateNodeTag: (nodeId, tag) => {
  set({ nodes: get().nodes.map(n => n.id === nodeId ? { ...n, data: { ...n.data, tag } } : n) })
},
updateNodeColor: (nodeId, color) => {
  set({ nodes: get().nodes.map(n => n.id === nodeId ? { ...n, data: { ...n.data, color } } : n) })
},
```

## Type Updates

Add `tag?: string` and `color?: string` to `SimNode.data`.

## PropertiesPanel Identity Section

First section when selecting a node — Label, Tag, Color — before configuration sliders. Place at the very top of the panel.

## Color Picker UI

Show a grid of 12 preset color swatches. The selected one gets a ring highlight. Clicking a color calls `updateNodeData({ color })` and closes the picker:

```tsx
const NODE_COLORS = [
  { name: 'Indigo', value: '#6366f1' },
  { name: 'Blue', value: '#3b82f6' },
  { name: 'Cyan', value: '#06b6d4' },
  { name: 'Teal', value: '#14b8a6' },
  { name: 'Green', value: '#22c55e' },
  { name: 'Yellow', value: '#eab308' },
  { name: 'Orange', value: '#f97316' },
  { name: 'Red', value: '#ef4444' },
  { name: 'Pink', value: '#ec4899' },
  { name: 'Purple', value: '#a855f7' },
  { name: 'Slate', value: '#64748b' },
  { name: 'Rose', value: '#f43f5e' },
]
```
