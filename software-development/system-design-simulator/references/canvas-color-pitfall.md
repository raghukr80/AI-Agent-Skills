# Canvas CSS Variable Pitfall

## Problem

Canvas 2D API (`CanvasGradient.addColorStop()`, `ctx.fillStyle`, `ctx.strokeStyle`) **cannot parse CSS custom properties** like `var(--color-success)`.

```tsx
// ❌ BREAKS — produces "var(--color-success)50" which is not a valid color
const grad = ctx.createRadialGradient(x, y, 0, x, y, r)
grad.addColorStop(0, p.color + '50')  // p.color = 'var(--color-success)'
// → Error: "The value provided ('var(--color-success)50') could not be parsed as a color"
```

```tsx
// ❌ BREAKS — CSS var not resolved in canvas context
ctx.fillStyle = 'var(--color-accent)'
```

## Solution

Use **hex color values** for all canvas operations. If you need alpha, use 8-digit hex:

```tsx
// ✅ CORRECT — hex color with alpha suffix
const COLORS = { running: '#22c55e', degraded: '#eab308', failed: '#ef4444', idle: '#6366f1' }
grad.addColorStop(0, p.color + '4D')  // 4D = ~30% alpha in hex
grad.addColorStop(1, p.color + '00')  // 00 = fully transparent
```

For trail strokes with alpha:

```tsx
// ✅ Use globalAlpha for stroke transparency instead of color alpha
ctx.globalAlpha = p.alpha * 0.25
ctx.strokeStyle = p.color  // hex color, alpha handled by globalAlpha
ctx.stroke()
```

## Key Rule

- **Tailwind/CSS**: Use `var(--color-success)` freely (DOM elements, Tailwind classes)
- **Canvas 2D API**: Always use hex (`#22c55e`) or rgba(`rgba(34, 197, 94, 0.3)`)
- **Never mix**: Don't pass CSS variable strings to any Canvas API method
