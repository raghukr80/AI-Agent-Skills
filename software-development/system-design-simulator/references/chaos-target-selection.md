# Chaos Target Selection Modal

## Pattern: Selective Chaos Injection

When injecting chaos, if multiple compatible nodes exist, show a target selection modal instead of auto-applying to all.

### Implementation

```tsx
// In ChaosPanel.tsx

const handleInjectClick = (scenario: ChaosDefinition) => {
  const compatible = getCompatibleNodes(scenario)
  if (compatible.length === 0) return
  if (compatible.length === 1) {
    // Single target — inject directly
    store.injectChaos({ ...scenario } as ChaosScenario, compatible[0].id)
  } else {
    // Multiple targets — show selector modal
    compatible.forEach((n: any) => { n._selected = true }) // Pre-select all
    setTargetSelector({ scenario, compatible: [...compatible] })
  }
}
```

### Target Selector Modal

- Renders at `z-[60]` (higher than main chaos modal's `z-50`)
- Main chaos modal stays open when selector is shown
- Backdrop click closes only the selector (not the main modal)
- Node list with checkboxes showing: icon, label, component type
- Select All / Deselect All toggle
- Inject button disabled when nothing selected
- Cancel button to go back

### Key Details

- Initialize `_selected = true` on all compatible nodes when opening selector
- Use `(n as any)._selected` to track selection state on node objects
- Force re-render by creating new array: `setTargetSelector({ ...targetSelector, compatible: [...targetSelector.compatible] })`
- Filter selected nodes at inject time: `targetSelector.compatible.filter(n => n._selected).map(n => n.id)`

### Common Pitfalls

- **Main modal closing when selector opens**: Don't condition main modal on `!targetSelector`. Instead, keep main modal always open when `open` is true, and guard the backdrop click: `onClick={() => { if (!targetSelector) setOpen(false) }}`
- **Z-index stacking**: Target selector must be `z-[60]`, main modal `z-50`. Same z-index causes flickering.
- **Selection state not persisting**: Must mutate `node._selected` directly and force re-render via new array reference
