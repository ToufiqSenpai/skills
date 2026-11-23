---
name: setup-typescript-project
description: Initialize new TypeScript projects. Use when creating a new TypeScript project, bootstrapping a repository, configuring oxlint or formatting, or migrating an existing JavaScript/TypeScript codebase to standardized linting and TypeScript settings.
disable-model-invocation: true
---

# Setup TypeScript Project

Bootstrap new TypeScript projects or retrofit existing repositories with a standardized `tsconfig.json` (ESNext), type-aware oxlint rules, runtime-aware formatting (bun fmt vs oxfmt), and unified `package.json` scripts.

## When to Use

- Initializing a brand-new TypeScript project from scratch
- Applying strict ESNext TypeScript settings and oxlint rules to an existing repository
- Configuring linting, formatting, and type-checking scripts in `package.json`

## Workflow

### 1. Detect Mode and Runtime

Inspect the target directory before making changes.

**Determine the scenario:**

- **New Project**: No `package.json` exists. Initialize the project first with `bun init -y` for Bun, or `npm init -y` / `pnpm init` for Node.
- **Existing Repository**: `package.json` already exists. Preserve existing dependencies, scripts, and framework-specific paths.

**Determine the runtime and package manager:**

- **Bun**: `bun.lock` or `bun.lockb` exists, or the user specifies Bun.
- **Node**: `pnpm-lock.yaml`, `package-lock.json`, or `yarn.lock` exists, or Bun is not used. Use the matching package manager (`pnpm`, `npm`, or `yarn`).

### 2. Install Dev Dependencies

Install `typescript`, `oxlint`, and `oxlint-tsgolint` (required because `.oxlintrc.json` enables `"typeAware": true`). For Node runtimes, also install `oxfmt`.

For Bun:

```bash
bun add -d typescript oxlint oxlint-tsgolint @types/bun
```

For Node (pnpm / npm):

```bash
pnpm add -D typescript oxlint oxlint-tsgolint oxfmt @types/node
# or: npm install -D typescript oxlint oxlint-tsgolint oxfmt @types/node
```

### 3. Configure tsconfig.json

Ensure `tsconfig.json` uses `ESNext` for both `target` and `module`, alongside strict type-checking flags so type-aware oxlint rules work accurately.

- **New Project**: Create `tsconfig.json` from the baseline below.
- **Existing Repository**: Merge `target: "ESNext"`, `module: "ESNext"`, `moduleResolution: "bundler"`, and `strict: true` into the existing `compilerOptions` without breaking framework-specific settings (such as `jsx`, `paths`, or `plugins`).

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ESNext", "DOM", "DOM.Iterable"],
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noEmit": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"]
}
```

### 4. Apply .oxlintrc.json from Assets

Copy `assets/.oxlintrc.json` into the project root as `.oxlintrc.json`.

**Existing Repository**: If `.oxlintrc.json` already exists, merge the `rules`, `plugins`, and `ignorePatterns` from `assets/.oxlintrc.json` rather than discarding project-specific ignores. If conflicting legacy ESLint or Prettier configs exist, note them to the user before removing anything.

### 5. Add Standard package.json Scripts

Merge the scripts below into `package.json` based on the detected runtime, keeping existing scripts intact.

For Bun:

```json
{
  "scripts": {
    "lint": "oxlint",
    "lint:fix": "oxlint --fix",
    "format": "bun fmt",
    "format:check": "bun fmt --check",
    "typecheck": "tsc --noEmit",
    "check": "bun run lint && bun run format:check && bun run typecheck"
  }
}
```

For Node:

```json
{
  "scripts": {
    "lint": "oxlint",
    "lint:fix": "oxlint --fix",
    "format": "oxfmt",
    "format:check": "oxfmt --check",
    "typecheck": "tsc --noEmit",
    "check": "pnpm run lint && pnpm run format:check && pnpm run typecheck"
  }
}
```

`pnpm` is the default for Node. Replace `pnpm run` with `npm run` only when the repository uses npm or yarn.

### 6. Verify Setup

Run the verification commands to confirm the configuration is valid and executable:

- **Bun**: `bun run lint && bun run typecheck`
- **Node**: `pnpm lint && pnpm typecheck` (use `npm run` only when the repository uses npm or yarn)

If an existing repository surfaces pre-existing lint or type errors from the newly enabled strict rules, report the summary clearly and offer to fix them rather than weakening the rules.

## Gotchas

- **Missing oxlint-tsgolint**: The template sets `"options": { "typeAware": true }`. Running oxlint without `oxlint-tsgolint` installed fails on type-aware rules such as `typescript/no-unnecessary-condition`.
- **Formatter mismatch**: Never configure oxfmt for a project that runs on Bun; use `bun fmt`. Only install and configure oxfmt for Node.
- **Overwriting existing project configs**: On existing repositories, never overwrite `package.json` or `tsconfig.json` wholesale. Always read and merge into the existing fields.
