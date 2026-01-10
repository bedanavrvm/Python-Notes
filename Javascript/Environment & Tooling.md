# Environment & Tooling (JavaScript vs TypeScript)

This chapter focuses on how you *run* JavaScript and how TypeScript fits into that.

## Where JavaScript runs

- **Browser**: includes the DOM, Web APIs (fetch, timers), and security constraints.
- **Node.js**: includes Node APIs (fs, http), CommonJS/ESM module system choices.

<details>
<summary>Practical note: Node version matters</summary>

Modern JavaScript features (ES modules, `fetch`, top-level await, etc.) depend on your Node version and configuration.

In teams, pin a Node version to avoid “works on my machine” issues.

</details>

## Tooling goals

- Run code consistently
- Manage dependencies
- Provide formatting/linting
- (TS) compile and type-check

## Node version management

If you use Node.js, you usually want a reproducible Node version.

Common approaches:

- `.nvmrc` / `.node-version` files (for Node version managers)
- Docker for fully pinned environments

<details>
<summary>Show a minimal .nvmrc example</summary>

```text
20
```

</details>

## Minimal setup: running a file

{% tabs %}

{% tab title="JavaScript" %}

```bash
node index.js
```

```js
// index.js
console.log("Hello JS");
```

{% endtab %}

{% tab title="TypeScript" %}
TypeScript must be type-checked and then executed as JavaScript.

Typical flow:

```bash
# 1) type-check + emit JS
npx tsc

# 2) run output
node dist/index.js
```

```ts
// src/index.ts
console.log("Hello TS");
```

{% endtab %}

{% endtabs %}

## Package managers and lockfiles

JavaScript projects are built around `package.json` (dependencies + scripts) and a lockfile (exact versions).

{% tabs %}

{% tab title="npm" %}

- Default with Node
- Lockfile: `package-lock.json`
{% endtab %}

{% tab title="yarn" %}

- Popular in monorepos
- Lockfile: `yarn.lock`
{% endtab %}

{% tab title="pnpm" %}

- Disk-efficient store, strict dependency behavior
- Lockfile: `pnpm-lock.yaml`
{% endtab %}

{% endtabs %}

<details>
<summary>Why lockfiles matter</summary>

Without a lockfile, two installs can resolve different transitive dependency versions.
That can change runtime behavior even if your `package.json` did not change.

</details>

## Dependency ranges (semver)

Most tooling follows semantic versioning (semver): `MAJOR.MINOR.PATCH`.

Common specifiers:

- `^1.2.3`: allow minor/patch updates (`1.x.x`)
- `~1.2.3`: allow patch updates (`1.2.x`)
- `1.2.3`: pin exactly

This interacts with lockfiles: lockfiles usually pin exact versions.

## Project structure (recommended)

- `src/` for source
- `dist/` for compiled output (TS)
- `package.json` for dependencies and scripts

```text
my_project/
  package.json
  src/
    index.ts
  dist/
    index.js
```

<details>
<summary>Why separate src/ and dist/</summary>

- Keeps your runtime output clean and disposable.
- Avoids mixing generated files with source control.
- Makes it obvious what runs in production (the emitted JS).

</details>

## package.json scripts (JS vs TS)

{% tabs %}

{% tab title="JavaScript" %}

```json
{
  "name": "my-js-project",
  "type": "module",
  "scripts": {
    "start": "node src/index.js"
  }
}
```

{% endtab %}

{% tab title="TypeScript" %}

```json
{
  "name": "my-ts-project",
  "type": "module",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

{% endtab %}

{% endtabs %}

## Development workflow: running TS without a manual build

During development, people often want “run TS directly” (fast feedback) while still keeping `tsc` for type-checking.

Common approaches (conceptual):

- `tsc --noEmit` for type-check-only in CI
- a runtime loader for dev (examples: `tsx`, `ts-node`)
- a bundler/dev server for browser apps (examples: Vite)

<details>
<summary>Show a dev script pattern</summary>

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "dev": "tsx src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

</details>

## tsconfig.json (what it means)

A `tsconfig.json` controls TypeScript compiler behavior.

<details>
<summary>Show a practical tsconfig.json</summary>

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "moduleResolution": "Bundler",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noImplicitAny": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedIndexedAccess": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

Key ideas:

- `strict`: enables strong safety checks.
- `rootDir` / `outDir`: separates TS source from JS output.
- `target`: which JS features are allowed in output.

</details>

### Key tsconfig concepts (reference)

- `target`: what syntax TS emits (e.g. `ES2020`).
- `module`: the module format TS emits (`ESM` vs `CommonJS`).
- `moduleResolution`: how TS resolves imports.
- `lib`: what built-in environment types exist (DOM vs Node, etc.).
- `types`: which `@types/*` packages are included globally.

<details>
<summary>Show lib/types gotcha (DOM vs Node)</summary>

If you compile for Node but include DOM libs, you can accidentally think browser globals exist.
If you compile for the browser but omit DOM libs, `document` and `fetch` types may be missing.

</details>

### Strictness flags worth knowing

- `strict`: umbrella flag enabling many checks.
- `noImplicitAny`: avoid silent `any`.
- `exactOptionalPropertyTypes`: makes optional properties more precise.
- `noUncheckedIndexedAccess`: makes `obj[key]` return possibly-undefined.

These flags tend to push you toward safer boundary handling.

### `noEmit` vs `emitDeclarationOnly`

- `noEmit`: type-check only (common in CI)
- `emitDeclarationOnly`: emit `.d.ts` without JS (common for libraries)

## Linting and formatting

Even if you use TypeScript, you still need runtime discipline:

- Avoid implicit globals
- Prefer consistent formatting
- Catch foot-guns early (unused vars, shadowing)

<details>
<summary>Common JS/TS tooling stack</summary>

- Formatter: Prettier
- Linter: ESLint
- TS-aware linting: `@typescript-eslint/*`
- Unit tests: Vitest/Jest

</details>

### What linting catches that TS often doesn't

- Unhandled Promises / missing `await`
- Accidental shadowing
- Dead code / unused variables
- Complexity hotspots

TypeScript is about *types*, not code quality conventions.

## Testing

Tests are runtime safety, even in TypeScript.

<details>
<summary>Common testing layers</summary>

- Unit tests: small pure logic
- Integration tests: DB/network boundaries
- End-to-end tests: user flows (browser automation)

</details>

## Source maps

Both JS bundlers and the TS compiler can generate source maps so stack traces map back to your original source.

<details>
<summary>Common debugging workflow</summary>

- Enable source maps so stack traces point to `src/`.
- Prefer reproducing bugs in the emitted JS when diagnosing runtime behavior.
- Remember: if it fails at runtime, it’s a JS problem first (types may have missed a boundary).

</details>

## Summary

- JavaScript runs in multiple environments (browser/Node) with different APIs.
- TypeScript adds compile-time checking and usually a build step.
- Lockfiles make installs reproducible.
- `tsconfig.json` must align with your runtime/bundler.
- Linting and tests remain essential, even with TypeScript.

## Important Keywords

### **Runtime**

The environment that actually executes JavaScript (Node/browser).

### **Build step**

Tooling process that transforms/checks code before running or deploying.

### **Lockfile**

File that pins exact dependency versions for reproducible installs.

### **Semver**

Versioning convention (`MAJOR.MINOR.PATCH`) used for dependency ranges.

### **`tsconfig.json`**

TypeScript compiler configuration (emit format, strictness, resolution).

### **Source map**

Mapping from emitted JS back to original source for debugging.
