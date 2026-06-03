---
name: ocp-app-sdk-v3-migration
description: Use when migrating an OCP app from @zaiusinc/app-sdk 2.x.x to 3.x.x on the node22 runtime, modernizing dependencies, replacing node-fetch with native fetch, or migrating from Jest to vitest
---

# OCP App SDK v3 Migration

Migrate an OCP app from `@zaiusinc/app-sdk` 2.x.x to 3.x.x on the `node22` runtime. This upgrades core SDKs, removes `node-fetch` in favor of native `fetch`, modernizes ESLint config, and optionally migrates from Jest to vitest.

## Prerequisites

Before starting, confirm:

1. `app.yml` specifies `runtime: node22`
2. `package.json` has `@zaiusinc/app-sdk` at `2.x.x`

## Core Migration (Required)

### 1. Upgrade SDKs

```bash
yarn add --ignore-engines "@zaiusinc/app-sdk@3.3.2" "@zaiusinc/node-sdk@3.0.0" "@zaiusinc/eslint-config-presets@^3.0.0"
```

### 2. Remove node-fetch

```bash
yarn remove --ignore-engines node-fetch
yarn remove --ignore-engines @types/node-fetch
```

Replace all `node-fetch` imports with native `fetch` in application code. Native `fetch` is a near drop-in replacement requiring minimal changes.

### 3. Remove old ESLint dependencies

These are now bundled in `@zaiusinc/eslint-config-presets`. Ignore errors about modules not in `package.json`:

```bash
yarn remove --ignore-engines @eslint/compat
yarn remove --ignore-engines @stylistic/eslint-plugin
yarn remove --ignore-engines @typescript-eslint/eslint-plugin
yarn remove --ignore-engines @typescript-eslint/parser
yarn remove --ignore-engines eslint
yarn remove --ignore-engines eslint-plugin-import
yarn remove --ignore-engines eslint-plugin-jsdoc
yarn remove --ignore-engines eslint-plugin-prefer-arrow
yarn remove --ignore-engines eslint-plugin-react
yarn remove --ignore-engines tslint
yarn remove --ignore-engines @typescript-eslint/eslint-plugin-tslint
```

### 4. Create new ESLint config

Replace existing ESLint config files (`.eslintrc.*`, `tslint.json`) with `eslint.config.mjs`:

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

If not migrating to vitest, omit the `vitest` import and spread.

### 5. Check dependency compatibility

```bash
rm yarn.lock
rm -rf node_modules
yarn install
```

If `yarn install` fails, upgrade or replace incompatible third-party dependencies using `yarn add --ignore-engines [--dev] "<dependency>@<compatible_version>"` until it succeeds.

### 6. Validate

```bash
ocp app validate --no-prompt
```

Approve if prompted to upgrade `@zaiusinc/app-sdk` and `@zaiusinc/node-sdk` to latest versions.

## Optional: copyfiles to cpy-cli

```bash
yarn add --dev "cpy-cli@^6.0.0"
yarn remove --ignore-engines copyfiles
```

Update the `build` script in `package.json`:

```json
"build": "yarn && npx rimraf remove dist && npx tsc && cpy app.yml dist && cpy --up 1 \"src/**/*.{yml,yaml}\" dist"
```

## Optional: Jest to vitest

### Remove Jest

```bash
yarn remove --ignore-engines @types/jest
yarn remove --ignore-engines jest
yarn remove --ignore-engines ts-jest
yarn remove --ignore-engines eslint-plugin-jest
```

### Add vitest

```bash
yarn add --dev "vitest"
```

### Update test script

In `package.json`:

```json
"test": "npx vitest run --passWithNoTests"
```

### Create vitest.config.ts

```typescript
import { defineConfig } from 'vitest/config';

process.env.ZAIUS_ENV = 'test';

// Load test environment variables
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

### Automated test file migration

Add vitest imports to every test file:

```bash
find . -name "*.test.ts" -not -path "*/node_modules/*" -exec sed -i '' '1s/^/import { vi, beforeEach, afterEach, describe, it, expect } from "vitest";\n/' {} +
```

Replace `as jest.Mock` with `as ReturnType<typeof vi.fn>`:

```bash
find . -type f \( -name "*.ts" -o -name "*.tsx" \) -not -path "*/node_modules/*" -exec sed -i '' 's/as jest\.Mock/as ReturnType<typeof vi.fn>;/g' {} +
```

Replace `jest.` with `vi.`:

```bash
find . -type f \( -name "*.ts" -o -name "*.tsx" \) -not -path "*/node_modules/*" -exec sed -i '' 's/jest\./vi./g' {} +
```

### Update ESLint config for vitest

In `eslint.config.mjs`:

1. Remove `import jest from 'eslint-plugin-jest';` and `jest` from plugins
2. Remove all rules containing `jest`
3. Add `import vitest from '@zaiusinc/eslint-config-presets/vitest.mjs';` and spread `...vitest`

### Fix and validate

```bash
yarn lint --fix
yarn build test
rm jest.config.js
ocp app validate --no-prompt
```

## Common Errors

- **Compilation errors** from third-party dependency upgrades: fix individually
- **Dependency conflicts** with SDK sub-dependencies: upgrade the conflicting third-party dependency to a compatible version

## Final Cleanup

```bash
rm yarn.lock
rm -rf node_modules
yarn install
ocp app validate --no-prompt
```

Deploy a `-dev` or `-beta` version and test thoroughly before releasing.
