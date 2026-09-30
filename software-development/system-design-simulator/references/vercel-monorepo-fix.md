# Vercel Monorepo Workspace Dependency Fix

## Problem
Vercel build fails with "Cannot find module 'html-to-image' or its corresponding type declarations" even though the package is in root `package.json` dependencies.

## Root Cause
Yarn workspaces install dependencies per-workspace. Vercel's build environment runs `yarn install` in the workspace directory (`apps/web`), not the monorepo root. Dependencies in root `package.json` are not automatically hoisted/linked to workspaces during Vercel's isolated workspace install.

## Solution
Add required dependencies to **each workspace's** `package.json` that actually uses them.

### Files to Update
- `apps/web/package.json` — add `html-to-image`, `jspdf`, `@types/node` to `dependencies`

### Example
```json
{
  "dependencies": {
    "@types/node": "^22.10.0",
    "@xyflow/react": "^12.4.0",
    "html-to-image": "^1.11.13",
    "jspdf": "^2.5.2",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "zustand": "^5.0.2",
    "recharts": "^2.15.0",
    "lucide-react": "^0.468.0"
  }
}
```

## Additional Vercel Config
`apps/web/vercel.json`:
```json
{
  "buildCommand": "node scripts/build-icon-manifest.cjs && tsc --noEmit && vite build",
  "outputDirectory": "dist",
  "framework": "vite",
  "installCommand": "yarn install --frozen-lockfile"
}
```

## Why Not `devDependencies`?
- `html-to-image` and `jspdf` are imported in production code (`Toolbar.tsx`, `CostPanel.tsx`)
- Must be in `dependencies`, not `devDependencies`
- `@types/node` needed for `tsc --noEmit` (Vercel runs type check)

## Checklist for New Dependencies
- [ ] Add to root `package.json` (for local dev via yarn workspaces)
- [ ] Add to `apps/web/package.json` `dependencies` (for Vercel build)
- [ ] Run `yarn install` locally to verify
- [ ] Run `npm run build` to verify type check passes
- [ ] Commit and push — Vercel will pick up workspace deps

## Files in This Project
- `apps/web/package.json` — Web workspace dependencies
- `package.json` — Root dependencies (for local dev)
- `apps/web/vercel.json` — Vercel build config