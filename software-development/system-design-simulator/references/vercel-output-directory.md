# Vercel Output Directory & Build Context

## Problem
Vercel build fails with "Could not find output directory" when deploying a Yarn/npm workspace monorepo.

## Root Cause
Vercel runs builds from a different working directory than expected. The `vercel.json` must be in the app directory (`apps/web/`), not the repo root. Vercel's workspace detection may pick the wrong directory.

## Solution
Place `vercel.json` in `apps/web/`:

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "installCommand": "npm install",
  "framework": "vite"
}
```

**Key**: Use `"outputDirectory": "dist"` (relative to `apps/web/`), NOT `"apps/web/dist"` (absolute from repo root).

## Build Command
Use `"buildCommand": "npm run build"` — this runs the `build` script from `apps/web/package.json` which is:
```
node scripts/build-icon-manifest.cjs && tsc --noEmit && vite build
```

Vite outputs to `dist/` by default relative to the `apps/web/` directory.

## Common Errors
- `sh: line 1: cd: apps/web: No such file or directory` — Vercel is not in repo root, don't use `cd` in buildCommand
- `Could not find output directory` — vercel.json is in wrong location or outputDirectory path is wrong
- `Cannot find module '../../wasm/simulationController'` — WASM files not committed to git (fix .gitignore)
- `Could not resolve "./sim_engine.js"` — WASM module uses static import, must use dynamic import
