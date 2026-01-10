# Modules & Packaging (JavaScript vs TypeScript)

Modules solve code organization and reuse.

The key idea (similar to Python's `import` system) is:

- A module is a file that exports values.
- Importing a module creates a dependency edge in your program.
- Your runtime (browser/Node) must be able to resolve module specifiers to actual files.

JS has two dominant module systems:

- ESM (ECMAScript Modules): `import` / `export`
- CommonJS (Node legacy): `require` / `module.exports`

TypeScript supports both, but you must align TS config with your runtime/bundler.

## The import path problem (resolution)

In JavaScript, an `import` statement is only half the story. The runtime must answer:

- Where does `"./math.js"` live?
- What does `"react"` mean?
- What file should load for a package?

This is handled differently in:

- Browsers (URL-based, often bundler-assisted)
- Node.js (module resolution rules)
- Bundlers (Vite/Webpack/Rollup) which can rewrite/resolve imports

## ESM basics

{% tabs %}

{% tab title="JavaScript" %}

```js
// math.js
export function add(a, b) {
  return a + b;
}

// main.js
import { add } from "./math.js";
console.log(add(2, 3));
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

// main.ts
import { add } from "./math.js";
console.log(add(2, 3));
```

Note: TS often wants the runtime extension (`.js`) in imports when targeting ESM.
{% endtab %}

{% endtabs %}

### Why the `.js` extension appears in TypeScript imports

When you target ESM, Node's runtime import resolution expects file extensions.
TypeScript often recommends writing runtime-correct imports in TS:

- Source file: `math.ts`
- Runtime output file: `math.js`
- Import in source: `import { add } from "./math.js";`

This avoids “works in TS, breaks in Node” mismatches.

<details>
<summary>Show common ESM resolution mistakes</summary>

```js
// Often fails in Node ESM without extra tooling:
import { add } from "./math";

// Works:
import { add } from "./math.js";
```

</details>

## Default exports vs named exports

- Named exports are easier to refactor
- Default exports are convenient but can cause inconsistent naming

{% tabs %}

{% tab title="JavaScript" %}

```js
// named
export const VERSION = "1.0";
export function parse() {}

// default
export default function main() {}
```

{% endtab %}

{% tab title="TypeScript" %}
Same concepts, plus you can export types.

```ts
export type User = { id: string };
export function parseUser(u: User) {}
```

{% endtab %}

{% endtabs %}

### Practical guidance

- Prefer **named exports** for libraries and larger codebases.
- Prefer **default exports** only when a file clearly has one primary thing.
- In TypeScript, exporting types (`export type ...`) helps keep boundaries explicit.

## CommonJS (Node legacy)

<details>
<summary>Show CommonJS vs ESM</summary>

{% tabs %}

{% tab title="CommonJS" %}

```js
// math.cjs
function add(a, b) {
  return a + b;
}

module.exports = { add };

// main.cjs
const { add } = require("./math.cjs");
console.log(add(2, 3));
```

{% endtab %}

{% tab title="ESM" %}

```js
// math.js
export function add(a, b) {
  return a + b;
}

// main.js
import { add } from "./math.js";
console.log(add(2, 3));
```

{% endtab %}

{% endtabs %}

</details>

### Interop pitfalls (ESM ↔ CommonJS)

Interop can be confusing because ESM and CommonJS have different semantics.

Common issues:

- Default import from CJS may behave differently (`import x from "cjs"`)
- Named imports from CJS are not real named exports
- Some libraries ship both ESM and CJS builds with different entry points

<details>
<summary>Show CJS importing pitfalls (conceptual)</summary>

```js
// If "some-cjs" is CommonJS, this may or may not work depending on Node/tooling:
import thing from "some-cjs";

// Sometimes you need:
import * as thing from "some-cjs";
```

</details>

## package.json essentials

- `name`, `version`
- `type`: affects module interpretation in Node (`module` -> ESM)
- `main` / `exports`: entry points

<details>
<summary>Show a minimal package.json for ESM</summary>

```json
{
  "name": "demo",
  "version": "1.0.0",
  "type": "module",
  "exports": {
    ".": "./dist/index.js"
  }
}
```

</details>

### `exports` and why it matters

`exports` lets packages restrict what import paths are valid.
It also enables different entry points for:

- ESM vs CJS
- Node vs browser
- types vs runtime JS

<details>
<summary>Show an exports map example (conceptual)</summary>

```json
{
  "name": "my-lib",
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "default": "./dist/index.js"
    }
  }
}
```

</details>

### `type`: "module" vs "commonjs"

In Node:

- `"type": "module"` means `.js` files are treated as ESM
- Without it (or `"type": "commonjs"`), `.js` files default to CommonJS

This setting has a huge impact on how imports work.

## TypeScript + declarations (.d.ts)

If you publish a TS library, consumers benefit from `.d.ts` type declarations.

- TS can emit them with `declaration: true`
- `types` field can point to the declaration entry

<details>
<summary>Show emitting declarations</summary>

```json
{
  "compilerOptions": {
    "declaration": true,
    "outDir": "dist",
    "rootDir": "src"
  }
}
```

</details>

### How TS compilation fits into packaging

At publish time you often want:

- `.js` output for runtime
- `.d.ts` output for types

The consumer uses JS at runtime, but their TS tooling reads the declarations.

## TypeScript compiler settings that affect modules

Two settings often determine whether your compiled output works as expected:

- `compilerOptions.module`
- `compilerOptions.moduleResolution`

<details>
<summary>Show common TS module settings (conceptual)</summary>

```json
{
  "compilerOptions": {
    "module": "ES2020",
    "moduleResolution": "Bundler",
    "target": "ES2020"
  }
}
```

</details>

## Path aliases (TS) vs runtime resolution

TypeScript path aliases are compile-time conveniences; runtime still needs help (bundler or Node loader) unless you emit relative paths.

<details>
<summary>Show why path aliases can break at runtime</summary>

```ts
// tsconfig paths might allow this in TS:
import { parseUser } from "@/users/parse";

// But Node runtime doesn't understand "@/" without a bundler/loader.
```

</details>

## Circular imports

Circular imports happen when A imports B and B imports A.
They are legal, but can cause surprising `undefined` values because modules execute in dependency order.

Practical guidance:

- Keep modules small and layered (avoid cycles)
- Move shared types/constants into a third module
- Avoid side effects at import time (especially in CommonJS)

## Summary

- JS modules are about runtime loading and resolution.
- Node and browsers resolve imports differently; bundlers often bridge the gap.
- TS adds a compile step and declaration output; your emitted JS must match your runtime.
- Prefer ESM for modern projects unless you have a reason to use CommonJS.

## Important Keywords

### **ESM (ECMAScript Modules)**

Modern standardized module system using `import`/`export`.

### **CommonJS**

Legacy Node module system using `require` and `module.exports`.

### **Module resolution**

How a runtime/bundler maps an import specifier to an actual file.

### **`package.json` `type`**

Node setting that controls whether `.js` is treated as ESM or CommonJS.

### **`exports` map**

`package.json` field that controls which entry points consumers are allowed to import.

### **Declarations (`.d.ts`)**

TypeScript-generated type definitions consumed by TS tooling.

### **Circular import**

Two modules depending on each other, which can cause partially-initialized exports.
