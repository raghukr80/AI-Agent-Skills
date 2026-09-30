# Color Picker Pattern

## Layout

```
[Color Swatch]  [#hexinput]  [X reset]
[Preset color swatches grid]
```

## Implementation

```tsx
{/* Color swatch — click opens native color picker */}
<div className="relative shrink-0">
  <input
    type="color"
    value={nodeColor}
    onChange={(e) => updateNodeData({ color: e.target.value })}
    className="absolute inset-0 w-full h-full opacity-0 cursor-pointer"
  />
  <div
    className="w-8 h-8 rounded-md border-2 border-border hover:border-accent/50 transition-colors cursor-pointer"
    style={{ backgroundColor: nodeColor }}
  />
</div>

{/* Hex input */}
<input
  type="text"
  value={nodeColor}
  onChange={(e) => {
    const val = e.target.value
    if (/^#[0-9a-fA-F]{0,6}$/.test(val)) {
      updateNodeData({ color: val })
    }
  }}
  onBlur={(e) => {
    if (!/^#[0-9a-fA-F]{6}$/.test(e.target.value)) {
      e.target.value = nodeColor
    }
  }}
  className="flex-1 text-[10px] text-text font-mono bg-bg border border-border rounded px-2 py-1.5 outline-none focus:border-accent"
  placeholder="#6366f1"
  maxLength={7}
/>

{/* Reset */}
<button onClick={() => updateNodeData({ color: undefined })}>
  <X className="w-3 h-3" />
</button>

{/* Preset swatches — always visible */}
<div className="grid grid-cols-6 gap-1.5">
  {NODE_COLORS.map(c => (
    <button
      key={c.value}
      onClick={() => updateNodeData({ color: c.value })}
      style={{ backgroundColor: c.value }}
    />
  ))}
</div>
```

## Key Points

- Native `<input type="color">` gives system color picker (hue slider, hex input, eye dropper)
- Position it absolutely with `opacity-0` over a visible swatch div
- Hex input validates with regex `/^#[0-9a-fA-F]{0,6}$/` on change
- On blur with invalid input, revert to last known good value
- Reset sets color to `undefined` → falls back to default `#6366f1`
- Preset swatches always visible (not behind a toggle)
