# Yarn to npm Conversion

Steps to convert a Yarn workspaces project to npm workspaces.

## Steps

1. **Delete yarn.lock**
   ```bash
   rm yarn.lock
   ```

2. **Update root package.json**
   - Remove `"packageManager": "yarn@1.22.22"`
   - Change scripts from `yarn workspace web build` to `npm run build --workspace=web`
   - Keep `"workspaces"` array (npm supports it)

3. **Update web/package.json**
   - Remove workspace-specific dependencies like `"sim-engine": "*"` (npm workspaces handle this)
   - Scripts stay the same (`dev`, `build`, `preview`)

4. **Install dependencies**
   ```bash
   npm install
   ```
   This generates `package-lock.json`.

5. **Create vercel.json** (if deploying to Vercel)
   ```json
   {
     "buildCommand": "npm run build",
     "outputDirectory": "dist",
     "installCommand": "npm install",
     "framework": "vite"
   }
   ```
   Place in the web app directory (`apps/web/vercel.json`), NOT the root.

6. **Update README** to use `npm install`, `npm run dev`, `npm run build` instead of yarn commands.

## Key Differences

| Yarn | npm |
|------|-----|
| `yarn install` | `npm install` |
| `yarn dev` | `npm run dev` |
| `yarn workspace web build` | `npm run build --workspace=web` |
| `yarn workspace web preview` | `npm run preview` (run from web dir) |
| `yarn.lock` | `package-lock.json` |
| `packageManager` field | Not needed |

## Pitfalls

- npm workspaces require `"workspaces"` in root `package.json`
- Vercel auto-detects npm if `package-lock.json` exists
- If a workspace dependency is listed in both root and web `package.json`, npm may hoist it incorrectly — remove duplicates
