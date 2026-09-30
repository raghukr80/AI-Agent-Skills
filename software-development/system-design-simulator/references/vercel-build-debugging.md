# Vercel Build Debugging

Transcript of Vercel build failures and fixes for syssim (React + Vite + Yarn workspaces monorepo).

## Error 1: `Cannot find module 'path'`
```
vite.config.ts(3,25): error TS2307: Cannot find module 'path' or its corresponding type declarations.
```
**Fix**: Replace `import { resolve } from 'path'` + `resolve(__dirname, './src')` with ESM-compatible:
```ts
import { fileURLToPath, URL } from 'url'
'@': fileURLToPath(new URL('./src', import.meta.url))
```

## Error 2: `tsconfig.node.json may not disable emit`
```
tsconfig.json(24,18): error TS6310: Referenced project may not disable emit.
```
**Fix**: Remove `references` from main tsconfig. Set `noEmit: true` in tsconfig.node.json. Use `tsc --noEmit` instead of `tsc -b` in build script.

## Error 3: `Cannot find type definition file for 'node'`
**Fix**: Move `@types/node` from `devDependencies` to `dependencies` — Vercel skips devDeps in production builds.

## Error 4: `Could not resolve "./sim_engine.js"`
```
Could not resolve "./sim_engine.js" from "src/wasm/simulationController.ts"
```
**Fix**: Use dynamic import for WASM modules:
```ts
const { default: init, SimController } = await import('./sim_engine.js')
```
Static imports fail because Rollup can't resolve WASM glue code at build time.

## Error 5: WASM files not in git
**Fix**: The `src/wasm/.gitignore` had `*` pattern ignoring all files. Fix with whitelist:
```
*
!.gitignore
!simulationController.ts
!sim_engine.js
!sim_engine_bg.wasm
```

## Error 6: `Cannot find module 'sim_engine.js'` (TypeScript)
**Fix**: Add `// @ts-nocheck` to top of `simulationController.ts` and `sim_engine.js`.

## Error 7: `vercel.json` location
**Fix**: Put `vercel.json` in `apps/web/` (not root) with:
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "installCommand": "npm install",
  "framework": "vite"
}
```

## Error 8: `cd apps/web` fails in Vercel
```
sh: line 1: cd: apps/web: No such file or directory
```
**Fix**: Don't use `cd` in build command. Vercel detects workspace root. Use `"buildCommand": "npm run build"` and `"outputDirectory": "apps/web/dist"` from root, OR put vercel.json in apps/web and use `"outputDirectory": "dist"`.

## Error 9: Yarn lockfile issues
**Fix**: Delete `yarn.lock`, use `npm install` to generate `package-lock.json`, update scripts to use `npm run --workspace=web`.
