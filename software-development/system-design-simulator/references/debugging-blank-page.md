# Debugging Blank/Black Page in Vite + React + ReactFlow Projects

## Symptoms
- App shows completely black screen or blank white page
- No visible React components in DOM (`document.getElementById('root').innerHTML === ''`)
- No console errors in some cases

## Diagnostic Checklist

### 1. Check for Silent React Render Crashes

**Most common cause**: Missing `import React from 'react'` when using `React.StrictMode`.

```tsx
// WRONG — crashes silently in Vite with new JSX transform
import ReactDOM from 'react-dom/client'
ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>  // ReferenceError: React is not defined
    <App />
  </React.StrictMode>
)

// CORRECT
import React from 'react'
import ReactDOM from 'react-dom/client'
// ...
```

**Test in browser console**:
```javascript
document.getElementById('root').innerHTML
// If empty but page title is set, React crashed during render
```

### 2. Check for Infinite Render Loops

**Symptom**: Browser tab freezes, console shows "Maximum update depth exceeded"

**Common cause with Zustand + ReactFlow**:
```tsx
// DANGEROUS — can cause infinite loop
useEffect(() => {
  setNodes(/* sync from store */)
}, [store.nodes])  // store.nodes changes → setNodes → triggers onNodesChange → store.setNodes → store.nodes changes → ...
```

**Fix**: Use `setInterval` polling instead, with guard flags:
```tsx
const syncingFromStore = useRef(false)
const syncingFromRf = useRef(false)

useEffect(() => {
  const interval = setInterval(() => {
    if (syncingFromRf.current) return
    setNodes(currentNodes => {
      if (changed) {
        syncingFromStore.current = true
        setTimeout(() => { syncingFromStore.current = false }, 0)
        return updated
      }
      return currentNodes
    })
  }, 200)
  return () => clearInterval(interval)
}, [store.nodes, setNodes])
```

### 3. Check Overlay Components Covers App

**Check for full-screen overlays**:
```javascript
Array.from(document.querySelectorAll('div'))
  .filter(d => {
    const s = getComputedStyle(d)
    return s.position === 'absolute' && parseInt(s.zIndex) > 10 && d.offsetWidth > 100
  })
  .map(d => ({ classes: d.className, zIndex: getComputedStyle(d).zIndex }))
```

### 4. Check Tailwind CSS Generation

```javascript
const el = document.createElement('div')
el.className = 'flex-1'
document.body.appendChild(el)
getComputedStyle(el).flexGrowth  // "1" = working, "0" = broken
document.body.removeChild(el)
```

### 5. Check ReactFlow Node Visibility

```javascript
const nodes = document.querySelectorAll('.react-flow__node')
for (const n of nodes) {
  console.log({ id: n.getAttribute('data-id'), visibility: n.style.visibility })
}
```

### 6. Headless Browser Viewport = 0px

Headless browser tools have 0px viewport. `h-screen` (100vh) = 0px.
App may render correctly in real browser but appear blank in headless tool.

### 7. Module Import Failures

```javascript
import('/src/stores/diagramStore.ts').then(m => 'ok').catch(e => 'FAIL: ' + e)
```

## Quick Fix Priority Order

1. Add `import React from 'react'` if using `React.StrictMode`
2. Remove `fitView` prop from `<ReactFlow>`
3. Add `visibility: visible !important` CSS for `.react-flow__node`
4. Check for infinite loop in Zustand<->ReactFlow sync
5. Check for full-screen overlay with high z-index
6. Verify Tailwind CSS is generating utilities
