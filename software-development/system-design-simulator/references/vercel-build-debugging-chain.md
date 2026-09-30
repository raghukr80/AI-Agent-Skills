# Vercel Build Debugging: Complete Error Chain & Fixes

## Overview
Vercel builds for monorepo projects can fail in multiple ways. This documents the exact chain of 9 errors encountered and their fixes, in order.

## Error Chain

### 1. `Cannot find module 'path'` in vite.config.ts
**Fix**: Replace `import { resolve } from 'path'` + `resolve(__dirname, './src')` with:
```ts
import { fileURLToPath, URL } from 'url'
// ...
'@': fileURLToPath(new URL('./src', import.meta.url)),
```

### 2. `Cannot find name '__dirname'` in vite.config.ts
**Fix**: Same as above — ESM doesn't have `__dirname`.

### 3. `Cannot find type definition file for 'node'`
**Fix**: Add `@types/node` to **dependencies** (not devDependencies — Vercel skips devDeps in production builds).

### 4. `Referenced project 'tsconfig.node.json' may not disable emit'`
**Fix**: Change `noEmit: false` to `noEmit: true` in `tsconfig.node.json`.

### 5. `Option 'allowImportingTsExtensions' can only be used when either 'noEmit' or 'emitDeclarationOnly' is set`
**Fix**: Same as above — `noEmit: true` in tsconfig.node.json.

### 6. `Could not resolve "./sim_engine.js"` from simulationController.ts
**Fix**: WASM files were not committed to git. Fix `apps/web/src/wasm/.gitignore` to whitelist specific files:
```
*
!.gitignore
!simulationController.ts
!simulationController.d.ts
!sim_engine.js
!sim_engine_bg.wasm
!sim_engine_bg.wasm.d.ts
```

### 7. `Could not resolve "./sim_engine.js"` at build time (Rollup)
**Fix**: WASM module must use **dynamic import**, not static:
```ts
// simulationController.ts
const { default: init, SimController } = await import('./sim_engine.js')
```

### 8. `sh: line 1: cd: apps/web: No such file or directory`
**Fix**: Don't use `cd` in vercel.json buildCommand. Vercel handles the directory automatically.

### 9. `Could not find output directory`
**Fix**: Put `vercel.json` in `apps/web/` (NOT repo root) with:
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "installCommand": "npm install",
  "framework": "vite"
}
```

## Key Principles
- Vercel runs `npm install` from repo root for workspaces
- Vercel skips devDependencies in production builds → `@types/node` must be in dependencies
- `tsc -b` (build mode) has stricter rules than `tsc --noEmit` → use `--noEmit` in build script
- WASM `.js` glue code cannot be resolved by Rollup at build time → dynamic import
- `vercel.json` location determines build context → put it in the app directory
- `outputDirectory` is relative to the vercel.json location → use `"dist"` not `"apps/web/dist"`
