# Reproduction: `vite-plugin-css-injected-by-js@5.0.0` + pnpm monorepo

Minimal reproduction for [marco-prontera/vite-plugin-css-injected-by-js#164](https://github.com/marco-prontera/vite-plugin-css-injected-by-js/pull/164).

## Bug

`vite-plugin-css-injected-by-js@5.0.0` imports `picocolors` in `dist/utils.log.js` without declaring it as a dependency. In a **pnpm monorepo with `hoist=false`**, pnpm enforces strict package isolation and the package cannot resolve `picocolors` — causing a build failure.

> **Note**: this does **not** reproduce in a single-package pnpm project, only in a monorepo with `hoist=false`.

## Environment

- pnpm 10.x with `hoist=false` (`.npmrc`)
- `vite@8.0.8`
- `vite-plugin-css-injected-by-js@5.0.0`
- Node.js ≥ 20

## Steps to reproduce

```sh
git clone https://github.com/smouillour/picocolors-issue.git
cd picocolors-issue
pnpm install
pnpm --filter my-lib build
```

## Expected

Build succeeds.

## Actual

```
error during build:
Error [ERR_MODULE_NOT_FOUND]: Cannot find package 'picocolors' imported from
.../node_modules/.pnpm/vite-plugin-css-injected-by-js@5.0.0_vite@8.0.8/node_modules/vite-plugin-css-injected-by-js/dist/utils.log.js
    at Object.getPackageJSONURL (node:internal/modules/package_json_reader:301:9)
    ...
```

## Root cause

`src/utils.log.ts` imports `picocolors` assuming it is a transitive dependency of Vite. However, **Vite 8 does not include `picocolors`** as a dependency (it was removed in the migration to `rolldown`). With pnpm's strict isolation (`hoist=false`), the plugin cannot resolve undeclared transitive dependencies.

## Workaround

Add a `packageExtensions` entry in the root `package.json`:

```json
"pnpm": {
  "packageExtensions": {
    "vite-plugin-css-injected-by-js": {
      "dependencies": {
        "picocolors": "*"
      }
    }
  }
}
```

## Fix

Add `picocolors` as an explicit `dependency` in `package.json` of the plugin. See [PR #164](https://github.com/marco-prontera/vite-plugin-css-injected-by-js/pull/164).
