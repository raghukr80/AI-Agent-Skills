# Dark/Light Mode Implementation

## CSS Variable Strategy

Define all colors as CSS custom properties on `:root`, then override in `:root.light`:

```css
/* index.css */
:root {
  --color-bg: #0a0a0f;
  --color-surface: #141420;
  --color-surface-hover: #1a1a2e;
  --color-border: #2a2a3e;
  --color-text: #e4e4ef;
  --color-text-dim: #8888a0;
  --color-accent: #6366f1;
  --color-accent-glow: #818cf8;
  --color-success: #22c55e;
  --color-warning: #eab308;
  --color-error: #ef4444;
}

:root.light {
  --color-bg: #f8f9fc;
  --color-surface: #ffffff;
  --color-surface-hover: #f0f1f5;
  --color-border: #e2e4ea;
  --color-text: #1a1a2e;
  --color-text-dim: #6b7280;
  --color-accent: #6366f1;
  --color-accent-glow: #818cf8;
  --color-success: #16a34a;
  --color-warning: #ca8a04;
  --color-error: #dc2626;
}
```

## Store Integration

```typescript
// diagramStore.ts
interface DiagramStore {
  theme: 'dark' | 'light'
  toggleTheme: () => void
}

// Initial state
theme: 'dark' as const,

// Action
toggleTheme: () => {
  const newTheme = get().theme === 'dark' ? 'light' : 'dark'
  set({ theme: newTheme })
  if (newTheme === 'light') {
    document.documentElement.classList.add('light')
  } else {
    document.documentElement.classList.remove('light')
  }
  localStorage.setItem('syssim-theme', newTheme)
},
```

## App Initialization

```typescript
// main.tsx — before React renders
const savedTheme = localStorage.getItem('syssim-theme')
if (savedTheme === 'light') {
  document.documentElement.classList.add('light')
}
```

## Toolbar Toggle Button

```tsx
// Toolbar.tsx — between CostPanel and Metrics button
<button
  onClick={() => store.toggleTheme()}
  className="p-1.5 rounded hover:bg-surface-hover text-text-dim hover:text-text transition-colors"
  title={store.theme === 'dark' ? 'Switch to light mode' : 'Switch to dark mode'}
>
  {store.theme === 'dark' ? <Sun className="w-4 h-4" /> : <Moon className="w-4 h-4" />}
</button>
```

## Critical: No Hardcoded Colors in Components

Every color reference in JSX must use `var(--color-*)`. Find violations with:

```bash
grep -rn "#[0-9a-fA-F]\{3,6\}" src/components/ --include="*.tsx"
```

Bulk replace with sed:

```bash
sed -i 's/#6366f1/var(--color-accent)/g; s/#2a2a3e/var(--color-border)/g; s/#8888a0/var(--color-text-dim)/g; s/#141420/var(--color-surface)/g; s/#22c55e/var(--color-success)/g; s/#eab308/var(--color-warning)/g; s/#ef4444/var(--color-error)/g' src/components/**/*.tsx
```

## Dot Grid Background

Override the React Flow dot grid for light mode:

```css
.dot-grid {
  background-image: radial-gradient(circle, var(--color-border) 1px, transparent 1px);
  background-size: 20px 20px;
}

:root.light .dot-grid {
  background-image: radial-gradient(circle, #d1d5db 1px, transparent 1px);
}
```

## Recharts Tooltip Colors

```tsx
<Tooltip contentStyle={{
  background: 'var(--color-surface)',
  border: '1px solid var(--color-border)',
  borderRadius: 8,
  fontSize: 12
}} />
```
