# Vercel / Production Build Configuration

## Problem: `path` and `__dirname` not available in ESM

Vercel's build environment uses ESM (`"type": "module"` in package.json). The Node.js `path` module and `__dirname` global are not available in ESM modules.

```
vite.config.ts(3,25): error TS2307: Cannot find module 'path' or its corresponding type declarations.
vite.config.ts(9,20): error TS2304: Cannot find name '__dirname'.
```

## Solution

Replace CommonJS-style `path` + `__dirname` with ESM-compatible `url` module:

```ts
// ❌ BREAKS on Vercel (CommonJS-style)
import { resolve } from 'path'
'@': resolve(__dirname, './src'),

// ✅ WORKS (ESM-compatible)
import { fileURLToPath, URL } from 'url'
'@': fileURLToPath(new URL('./src', import.meta.url)),
```

Apply to BOTH `vite.config.ts` AND `vite.config.js` (if both exist).

## Problem: `tsconfig.node.json` emit conflict

```
tsconfig.json(24,18): error TS6310: Referenced project may not disable emit.
```

## Solution

Remove the `references` from `tsconfig.json` entirely and include `vite.config.ts` directly. Exclude `src/wasm` from tsc. Change build script from `tsc -b` to `tsc --noEmit`.

## Problem: `@types/node` not installed during Vercel build

Vercel skips `devDependencies` in production. Move `@types/node` to `dependencies`.

## Problem: WASM module static import fails at build time

```
error during build:
Could not resolve "./sim_engine.js" from "src/wasm/simulationController.ts"
```

## Solution

Use **dynamic import** inside the `init()` method:

```ts
async init(diagram: DiagramData) {
  const { default: init, SimController } = await import('./sim_engine.js')
  await init()
  this.controller = new SimController()
}
```

Add `// @ts-nocheck` at the top of WASM module files.

## Problem: WASM files not in git

The `src/wasm/.gitignore` had `*` ignoring all files. Fix with whitelist pattern and commit `sim_engine.js` and `sim_engine_bg.wasm`.

## Vercel Configuration File

Place `vercel.json` in `apps/web/` (NOT root):

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "installCommand": "npm install",
  "framework": "vite"
}
```

Vercel auto-detects the workspace root. The `outputDirectory` is relative to `apps/web/`.

## Checklist for Vercel Deploys

- [ ] `vite.config.ts` uses `fileURLToPath(new URL(...))` not `resolve(__dirname, ...)`
- [ ] `tsconfig.json` excludes `src/wasm`, no project references
- [ ] Build script uses `tsc --noEmit` not `tsc -b`
- [ ] `@types/node` in `dependencies`
- [ ] WASM module uses dynamic `await import()`
- [ ] WASM files have `// @ts-nocheck`
- [ ] WASM `.gitignore` whitelists all necessary files
- [ ] `sim_engine.js` and `sim_engine_bg.wasm` committed to git
- [ ] `vercel.json` present in `apps/web/` with npm build commands