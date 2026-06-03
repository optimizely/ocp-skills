---
name: ocp-node22-runtime-migration
description: Use when migrating an OCP app from node18 to node22 runtime, upgrading SDK from 2.x to 3.x, replacing node-fetch with native fetch, migrating ESLint to v9, or migrating from Jest to Vitest in an OCP app.
allowed-tools: Bash, Read, Edit, Write, Glob, Grep
---

# OCP Node22 Runtime Migration

Step-by-step guide for migrating OCP apps to node22 runtime and modernizing dependencies.

## When to Use

- Migrating an OCP app from `node18` to `node22` runtime
- Upgrading `@zaiusinc/app-sdk` from `1.x` to `2.x` or from `2.x` to `3.x`
- Replacing `node-fetch` with native `fetch`
- Migrating ESLint to v9 flat config
- Migrating from Jest to Vitest

## Migration Paths

There are two sequential migration paths:

1. **node18 → node22** (SDK 1.x → 2.x) — Required first
2. **SDK 2.x → 3.x** (modernization) — Optional, requires path 1 complete

## Pre-Flight Checks

Before starting, verify:
- Current runtime in `app.yml` (`node18` or `node22`)
- Current SDK version in `package.json`
- You are running the correct Node.js version (`node --version` → `v22.x.x`)

## Path 1: node18 → node22 (SDK 2.x)

### Step 1: Update runtime

In `app.yml`:
```yaml
runtime: node22
```

### Step 2: Upgrade required dependencies

```bash
yarn add --ignore-engines "@zaiusinc/app-sdk@2.0.0" "@zaiusinc/node-sdk@2.0.0"
yarn add --dev --ignore-engines "@types/node@^22.15.17"
```

### Step 3: Upgrade ESLint to v9

Remove old ESLint dependencies (ignore errors for missing packages):
```bash
yarn remove --ignore-engines @eslint/compat @stylistic/eslint-plugin @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint eslint-plugin-import eslint-plugin-jsdoc eslint-plugin-prefer-arrow eslint-plugin-react tslint @typescript-eslint/eslint-plugin-tslint @zaiusinc/tslint-presets
```

Install new ESLint:
```bash
yarn add --ignore-engines --dev "@zaiusinc/eslint-config-presets@^3.0.0" "eslint-plugin-jest@^28.11.0"
```

Create `eslint.config.mjs` (replaces `.eslintrc.js`):
```javascript
import jest from 'eslint-plugin-jest';
import node from '@zaiusinc/eslint-config-presets/node.mjs';

export default [
  ...node,
  {
    files: ['**/*.test.ts'],

    plugins: {
      jest,
    },

    rules: {
      '@typescript-eslint/unbound-method': 'off',
      'jest/unbound-method': 'error',
    },
  }];
```

If the existing `.eslintrc.js` has customizations beyond the default, migrate them to the new flat config format. See [ESLint v9 migration guide](https://eslint.org/docs/latest/use/migrate-to-9.0.0).

Run linter and fix:
```bash
yarn lint --fix
```

Remove old config:
```bash
rm .eslintrc.js
```

### Step 4: Rebuild and validate

```bash
rm yarn.lock && rm -rf node_modules && yarn install
ocp app validate
```

Approve SDK version upgrades if prompted.

### Step 5 (Optional): Upgrade TypeScript

```bash
yarn add --dev "typescript@^5.8.3"
ocp app validate
```

### Step 6 (Optional): Replace @atao60/fse-cli with cpy-cli

```bash
yarn add --dev "cpy-cli@^6.0.0"
yarn remove --ignore-engines @atao60/fse-cli chalk
```

Update `build` script in `package.json`:
```json
"build": "yarn && npx rimraf remove dist && npx tsc && cpy app.yml dist && cpy --up 1 \"src/**/*.{yml,yaml}\" dist"
```

### Step 7 (Optional): Upgrade Jest

```bash
yarn add --dev "@types/jest@^29.5.14" "jest@^29.7.0"
ocp app validate
```

### Step 8 (Optional): Upgrade other dev-dependencies

```bash
yarn add --dev "dotenv@^16.5.0" "rimraf@^6.0.1" "@types/node-fetch@^2.6.12"
```

---

## Path 2: SDK 2.x → 3.x (Modernization)

**Prerequisite:** App already on `node22` runtime with SDK `2.x.x`.

### Step 1: Upgrade SDK to 3.x

```bash
yarn add --ignore-engines "@zaiusinc/app-sdk@3.3.2" "@zaiusinc/node-sdk@3.0.0" "@zaiusinc/eslint-config-presets@^3.0.0"
```

### Step 2: Remove node-fetch

```bash
yarn remove --ignore-engines node-fetch @types/node-fetch
```

Migrate all `node-fetch` imports to native `fetch`. Native `fetch` is a near drop-in replacement — remove import statements and adjust any type references.

### Step 3: Clean up ESLint dependencies

Remove old ESLint dependencies (same list as Path 1 Step 3 — ignore errors for missing packages):
```bash
yarn remove --ignore-engines @eslint/compat @stylistic/eslint-plugin @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint eslint-plugin-import eslint-plugin-jsdoc eslint-plugin-prefer-arrow eslint-plugin-react tslint @typescript-eslint/eslint-plugin-tslint
```

### Step 4: Rebuild and validate

```bash
rm yarn.lock && rm -rf node_modules && yarn install
ocp app validate
```

### Step 5 (Optional): Replace copyfiles with cpy-cli

```bash
yarn add --dev "cpy-cli@^6.0.0"
yarn remove --ignore-engines copyfiles
```

Update `build` script in `package.json`:
```json
"build": "yarn && npx rimraf remove dist && npx tsc && cpy app.yml dist && cpy --up 1 \"src/**/*.{yml,yaml}\" dist"
```

### Step 6 (Optional): Migrate from Jest to Vitest

Remove Jest dependencies:
```bash
yarn remove --ignore-engines @types/jest jest ts-jest eslint-plugin-jest
```

Add Vitest:
```bash
yarn add --dev "vitest"
```

Update `test` script in `package.json`:
```json
"test": "npx vitest run --passWithNoTests"
```

Create `vitest.config.ts`:
```typescript
import { defineConfig } from 'vitest/config';

process.env.ZAIUS_ENV = 'test';

import('dotenv').then(dotenv => {
  dotenv.config({ path: '.env.test' });
});

export default defineConfig({
  test: {
    environment: 'node',
    include: ['src/**/*.test.ts'],
    exclude: ['node_modules', 'dist'],
    coverage: {
      provider: 'v8',
      include: ['src/**/*.ts'],
      exclude: ['src/test/**/*', 'src/**/index.ts'],
    },
    globals: true,
  },
});
```

Update `eslint.config.mjs` — remove jest, add vitest:
```javascript
import node from '@zaiusinc/eslint-config-presets/node.mjs';
import vitest from '@zaiusinc/eslint-config-presets/vitest.mjs';

export default [
  ...node,
  ...vitest,
  {
    files: ['**/*.test.ts'],

    rules: {
      '@typescript-eslint/unbound-method': 'off'
    },
  }];
```

Migrate test files — replace jest references with vitest:
```bash
# Add vitest imports to all test files
find . -name "*.test.ts" -not -path "*/node_modules/*" -exec sed -i '' '1s/^/import { vi, beforeEach, afterEach, describe, it, expect } from "vitest";\n/' {} +

# Replace `as jest.Mock` with vitest equivalent
find . -type f \( -name "*.ts" -o -name "*.tsx" \) -not -path "*/node_modules/*" -exec sed -i '' 's/as jest\.Mock/as ReturnType<typeof vi.fn>;/g' {} +

# Replace `jest.` with `vi.`
find . -type f \( -name "*.ts" -o -name "*.tsx" \) -not -path "*/node_modules/*" -exec sed -i '' 's/jest\./vi./g' {} +
```

Fix up and validate:
```bash
yarn lint --fix
yarn build test
rm jest.config.js
ocp app validate
```

See [vitest migration guide](https://vitest.dev/guide/migration.html#jest) for edge cases.

---

## Final Cleanup (Both Paths)

```bash
rm yarn.lock && rm -rf node_modules && yarn install
ocp app validate
```

Deploy a `-dev` or `-beta` version and test thoroughly before production release.

## Typical Errors

| Error | Fix |
|-------|-----|
| Compilation errors from dependency upgrades | Fix individually |
| Third-party deps conflicting with SDK sub-dependencies | Upgrade the conflicting dependency |
| Code incompatible with `@types/node@22.x.x` | Update code for new type definitions |
| `yarn install` fails after lockfile removal | Upgrade incompatible deps with `yarn add --ignore-engines [--dev] "<dep>@<version>"` |

## Quick Reference: `--ignore-engines`

Always use `--ignore-engines` flag when adding/removing dependencies during migration to bypass engine checks temporarily. The final `yarn install` (after removing `yarn.lock`) validates true compatibility.
