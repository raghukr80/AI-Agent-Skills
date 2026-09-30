# Config Slider with Numeric Input + Stepper

The configuration slider in a system design simulator's PropertiesPanel must combine a numeric input with up/down stepper buttons AND a range slider, all bidirectionally synced.

## Layout

```
[▾] [  12,500  ] [▴] [ms] ━━━━━━━━━━━━━━━[●━━━━━━━━━━━━]
```

## Component API

```tsx
<ConfigSlider
  label="Max RPS"
  value={config.maxRps}
  min={100}
  max={100000}
  step={100}
  unit=""
  onChange={(v: number) => void}
  isPercent={false}
/>
```

## Implementation

```tsx
function ConfigSlider({ label, value, min, max, step, unit, onChange, isPercent }) {
  const [localValue, setLocalValue] = useState(String(value))

  useEffect(() => {
    setLocalValue(String(isPercent ? +(value * 100).toFixed(1) : value))
  }, [value, isPercent])

  const handleInputChange = (e) => {
    const raw = e.target.value
    setLocalValue(raw)
    const num = parseFloat(raw)
    if (!isNaN(num)) {
      const clamped = Math.min(max, Math.max(min, num))
      onChange(isPercent ? clamped / 100 : clamped)
    }
  }

  const handleInputBlur = () => {
    const num = parseFloat(localValue)
    if (isNaN(num)) {
      setLocalValue(String(isPercent ? +(value * 100).toFixed(1) : value))
    } else {
      const clamped = Math.min(max, Math.max(min, num))
      setLocalValue(String(isPercent ? +clamped.toFixed(1) : clamped))
      onChange(isPercent ? clamped / 100 : clamped)
    }
  }

  const handleStep = (direction) => {
    const num = parseFloat(localValue) || 0
    const newVal = Math.min(max, Math.max(min, num + direction * step))
    const formatted = isPercent ? +(newVal / 100).toFixed(1) : newVal
    setLocalValue(String(formatted))
    onChange(isPercent ? newVal / 100 : newVal)
  }

  return (
    <div>
      <div className="text-[10px] mb-1"><span className="text-text-dim">{label}</span></div>
      <div className="flex items-center gap-1.5">
        <div className="flex items-center border border-border rounded-md bg-bg overflow-hidden shrink-0">
          <button type="button" onClick={() => handleStep(-1)}
            className="px-1.5 py-1 text-text-dim hover:text-text hover:bg-surface-hover text-[10px] font-bold">▾</button>
          <input type="text" value={localValue}
            onChange={handleInputChange} onBlur={handleInputBlur}
            onKeyDown={(e) => { if (e.key === 'Enter') handleInputBlur() }}
            className="w-16 text-center text-[10px] text-text font-mono bg-transparent outline-none border-x border-border py-1" />
          <button type="button" onClick={() => handleStep(1)}
            className="px-1.5 py-1 text-text-dim hover:text-text hover:bg-surface-hover text-[10px] font-bold">▴</button>
          {unit && <span className="text-[9px] text-text-dim pr-1.5">{unit}</span>}
        </div>
        <input type="range"
          min={isPercent ? min * 100 : min} max={isPercent ? max * 100 : max}
          step={isPercent ? step * 100 : step} value={isPercent ? value * 100 : value}
          onChange={(e) => onChange(isPercent ? Number(e.target.value) / 100 : Number(e.target.value))}
          className="flex-1 h-1 accent-accent cursor-pointer min-w-0" />
      </div>
    </div>
  )
}
```

## Key Patterns

- **Local state for input**: Use `useState` for the input string to allow intermediate typing
- **Sync with useEffect**: When external `value` prop changes, update local string
- **Clamp on blur**: On blur, parse the value, clamp to min/max, and call onChange
- **Revert on invalid**: If blur value is NaN, revert to last known good value
- **Percent mode**: For values stored as 0-1 but displayed as 0-100, use `isPercent` flag
- **Stepper buttons**: Increment/decrement by the configured `step` value
- **updateConfig type**: Must accept `number | string | boolean` for all config field types

## ConfigSelect (Enum Dropdown)

```tsx
function ConfigSelect({ label, value, options, onChange }) {
  return (
    <div>
      <div className="text-[10px] mb-1"><span className="text-text-dim">{label}</span></div>
      <select value={value} onChange={(e) => onChange(e.target.value)}
        className="w-full bg-bg border border-border rounded text-[10px] text-text px-2 py-1.5 outline-none focus:border-accent cursor-pointer">
        {options.map(opt => <option key={opt.value} value={opt.value}>{opt.label}</option>)}
      </select>
    </div>
  )
}
```

## ConfigToggle (Boolean Switch)

```tsx
function ConfigToggle({ label, value, onChange }) {
  return (
    <div className="flex items-center justify-between">
      <span className="text-[10px] text-text-dim">{label}</span>
      <button type="button" onClick={() => onChange(!value)}
        className={`relative w-8 h-4 rounded-full transition-colors ${value ? 'bg-accent' : 'bg-border'}`}>
        <div className={`absolute top-0.5 w-3 h-3 rounded-full bg-white transition-transform ${value ? 'left-4.5' : 'left-0.5'}`} />
      </button>
    </div>
  )
}
```

## Conditional Rendering Pattern

Only render config fields when the key exists on the node's config:

```tsx
{config.cacheHitRatio !== undefined && (
  <>
    <ConfigSlider label="Cache Hit Ratio" value={config.cacheHitRatio} min={0} max={1} step={0.01} unit="%" onChange={v => updateConfig('cacheHitRatio', v)} isPercent />
    {config.cacheTtlSeconds !== undefined && <ConfigSlider label="Cache TTL" value={config.cacheTtlSeconds} min={10} max={3600} step={10} unit="s" onChange={v => updateConfig('cacheTtlSeconds', v)} />}
  </>
)}
{config.writeConsistency !== undefined && (
  <ConfigSelect label="Write Consistency" value={config.writeConsistency} options={[
    { value: 'strong', label: 'Strong' },
    { value: 'eventual', label: 'Eventual' },
    { value: 'session', label: 'Session' },
  ]} onChange={v => updateConfig('writeConsistency', v)} />
)}
{config.autoScale !== undefined && (
  <ConfigToggle label="Auto Scale" value={config.autoScale} onChange={v => updateConfig('autoScale', v)} />
)}
```
