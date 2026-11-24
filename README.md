# Capacitor Electron in an Nx workspace

Reproduction for [`@capawesome/capacitor-electron`](https://github.com/capawesome-team/capacitor-electron) in an [Nx](https://nx.dev) monorepo.

The workspace was created with `npx create-nx-workspace@latest --preset=apps` (Nx 24). As usual for Nx, there is one `package.json` and one `node_modules` at the workspace root, and the Capacitor app lives in a subfolder:

```
package.json              <- all dependencies (Capacitor, @capawesome/capacitor-electron)
node_modules/
nx.json
apps/my-app/
  capacitor.config.ts     <- Capacitor root
  www/index.html
```

## Steps

```bash
npm install
cd apps/my-app
npx cap add @capawesome/capacitor-electron    # works
npx cap sync @capawesome/capacitor-electron   # fails on `main`, works on `fix`
```

## Result on `main` (`@capawesome/capacitor-electron` 0.2.2)

```
[capacitor-electron] ERROR: ENOENT: no such file or directory, open '<repo>/apps/my-app/package.json'
[fatal] Error: capacitor:update hook on @capawesome/capacitor-electron failed with error code: 1 while running command: node ./dist/cli/index.js update
```

The plugin manifest step reads `package.json` and `node_modules` only from the Capacitor root (`apps/my-app`), not from the workspace root.

## Result on `fix` (patched build)

This branch installs `@capawesome/capacitor-electron` from `vendor/capawesome-capacitor-electron-0.2.2-monorepo-fix.tgz`, built (`npm run build && npm pack`) from the fix in [CrackerakiUA/capacitor-electron@f85c487](https://github.com/CrackerakiUA/capacitor-electron/commit/f85c487). The fork can't be installed from git directly, because `dist/` isn't committed and `prepare` doesn't build it.

`@capawesome/capacitor-live-update` is added as an example plugin with an Electron implementation, installed at the workspace root:

```
[capacitor-electron] Copied web assets to <repo>/apps/my-app/electron/app.
[capacitor-electron] Found 1 plugin(s) with an electron implementation.
```

## Note: `capacitor:*` scripts in the root `package.json`

In an Nx workspace the Capacitor CLI reads the platform hook scripts (`capacitor:add`, `capacitor:copy`, …) from the **workspace root** `package.json` instead of from the platform package, so without them `npx cap add/sync @capawesome/capacitor-electron` silently does nothing. That is a separate Capacitor CLI issue. The root `package.json` therefore repeats the platform's scripts; Capacitor runs them with the platform package as working directory, so the relative paths are the same.
