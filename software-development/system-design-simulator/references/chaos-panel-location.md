# Chaos Panel Location

The chaos engineering UI is in `src/components/palette/ComponentPalette.tsx` as the `ChaosTab` function — NOT in `src/components/toolbar/ChaosPanel.tsx`.

## Key Facts
- `ChaosPanel.tsx` exists but is NOT imported anywhere — it's dead code
- The actual chaos UI is a tab in the left sidebar ComponentPalette, toggled via "Components" / "Chaos" tab buttons
- Chaos scenarios are defined inline in `CHAOS_SCENARIOS` array at the top of ComponentPalette.tsx
- The `CATEGORY_CONFIG` object (with icons, colors, descriptions) is defined after `CATEGORY_LABELS` and `CATEGORY_COLORS`
- Target selection modal is inline within the `ChaosTab` function

## When asked to modify chaos
Always edit `src/components/palette/ComponentPalette.tsx`, never `ChaosPanel.tsx`.
