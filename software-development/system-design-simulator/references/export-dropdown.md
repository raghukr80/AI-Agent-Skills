# Export Dropdown Pattern: Click Outside Handler Fix

## Problem
Export dropdown with JSON/PNG/PDF options doesn't work — clicking any option has no effect. The dropdown closes immediately before the export handler can execute.

## Root Cause
The outside-click handler uses `mousedown` event, which fires BEFORE the button's `click` handler. When the user clicks an export option:
1. `mousedown` fires on the button
2. The document `mousedown` handler closes the dropdown
3. The button's `click` handler never executes because the button is removed from the DOM

## Solution
1. **Use `click` event instead of `mousedown`** for outside-click detection
2. **Use a ref** to check if the click was inside the dropdown container

```tsx
const exportMenuRef = useRef<HTMLDivElement>(null)

useEffect(() => {
  if (!showExportMenu) return
  const handler = (e: MouseEvent) => {
    if (exportMenuRef.current && !exportMenuRef.current.contains(e.target as Node)) {
      setShowExportMenu(false)
    }
  }
  document.addEventListener('click', handler)  // NOT mousedown
  return () => document.removeEventListener('click', handler)
}, [showExportMenu])
```

## PNG Export
```tsx
import { toPng } from 'html-to-image'

const handleExportPNG = async () => {
  const el = document.querySelector('.react-flow') as HTMLElement | null
  if (!el) return
  const dataUrl = await toPng(el, {
    backgroundColor: 'var(--color-bg)',
    quality: 1,
    pixelRatio: 2,
  })
  const a = document.createElement('a')
  a.href = dataUrl
  a.download = 'syssim-diagram.png'
  a.click()
}
```

## PDF Export
```tsx
import { jsPDF } from 'jspdf'

const handleExportPDF = async () => {
  const el = document.querySelector('.react-flow') as HTMLElement | null
  if (!el) return
  const dataUrl = await toPng(el, { backgroundColor: 'var(--color-bg)', quality: 1, pixelRatio: 2 })
  const img = new Image()
  img.src = dataUrl
  await new Promise(resolve => { img.onload = resolve })
  const pdf = new jsPDF({
    orientation: img.width > img.height ? 'landscape' : 'portrait',
    unit: 'px',
    format: [img.width, img.height],
  })
  pdf.addImage(dataUrl, 'PNG', 0, 0, img.width, img.height)
  pdf.save('syssim-diagram.pdf')
}
```

## Key Takeaway
When implementing dropdown menus that close on outside-click:
- **NEVER use `mousedown`** for the outside-click handler if the dropdown contains clickable elements
- **Always use `click`** event — it fires after the button's click handler, allowing the action to execute before the dropdown closes
- **Use a ref** to check if the click target is inside the dropdown, rather than blanket closing on any outside mousedown
