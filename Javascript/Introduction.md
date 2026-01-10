# Introduction (JavaScript vs TypeScript)

JavaScript (JS) is the language that runs in browsers and Node.js.
TypeScript (TS) is a superset of JavaScript that adds **types** and compiles back to JavaScript.

The key mindset:

- JavaScript answers: **What happens at runtime?**
- TypeScript answers: **Can we catch mistakes before runtime?**

## What stays the same

TypeScript is still JavaScript:

- The runtime is still JS (V8, SpiderMonkey, JavaScriptCore)
- The standard library and platform APIs are still JS (DOM, Web APIs, Node APIs)
- All JS syntax is valid TS (in normal configurations)

### What TypeScript does *not* change

- **The runtime is still JavaScript**. Your code runs in the same JS engine.
- **Performance characteristics** are the same unless you change the emitted JS.
- **You can still throw runtime errors** (TypeScript reduces them, but cannot eliminate them).

## What TypeScript adds

- **Static types** (compile-time only)
- Better editor tooling (autocomplete, refactors)
- Safer large-scale code changes
- Advanced type-system features (unions, generics, type narrowing)

### What TypeScript costs

- A build step (type-check + emit JS)
- Type design work (modeling domains and boundaries)
- Occasional friction when integrating untyped/loosely-typed libraries

In practice, most teams accept this cost because it pays back during refactoring.

## Where JavaScript runs (environments)

JavaScript is a language, but the **platform** determines what APIs you can call.

{% tabs %}

{% tab title="Browser" %}

- DOM APIs (`document`, `window`)
- Web APIs (`fetch`, `localStorage`, `WebSocket`)
- Security model (CORS, sandboxing)
{% endtab %}

{% tab title="Node.js" %}

- Node APIs (`fs`, `path`, `http`)
- Module resolution rules (ESM/CommonJS)
- Different globals (`process`, `Buffer`)
{% endtab %}

{% endtabs %}

TypeScript can target both environments, but type definitions differ (DOM vs Node types).

## How TypeScript runs: the compilation pipeline

TypeScript does not run directly in most environments.

Typical pipeline:

1. Write TS (`.ts` / `.tsx`)
2. Type-check (compile-time)
3. Emit JS (`.js`) + optionally `.d.ts`
4. Run emitted JS in the runtime (browser/Node)

<details>
<summary>Show what “type-check” vs “emit” means</summary>

- **Type-check**: TS validates your types. This can fail the build.
- **Emit**: TS outputs JavaScript. You can configure whether emit happens on errors.

</details>

## When to choose JavaScript

- Small scripts or quick prototypes
- Learning the runtime and the language fundamentals
- Environments where you cannot (or don’t want to) add a build step

### Typical JS-only scenarios

- Small automation scripts
- Quick experiments
- Embedded scripts (some tooling ecosystems)

## When to choose TypeScript

- Medium/large applications
- Teams, long-lived projects, refactoring-heavy codebases
- Libraries/SDKs that benefit from strong public API contracts

### Typical TS scenarios

- Codebases where “safe refactoring” is important
- Projects with many contributors
- Projects with complex data shapes (API-heavy frontends/backends)

## JavaScript vs TypeScript: a first comparison

{% tabs %}

{% tab title="JavaScript" %}

```js
function greet(user) {
  // No compile-time guarantees about user
  return "Hello " + user.name.toUpperCase();
}

console.log(greet({ name: "Alice" }));
console.log(greet({})); // runtime error
```

{% endtab %}

{% tab title="TypeScript" %}

```ts
type User = { name: string };

function greet(user: User): string {
  return "Hello " + user.name.toUpperCase();
}

console.log(greet({ name: "Alice" }));
// console.log(greet({})); // compile-time error
```

{% endtab %}

{% endtabs %}

## Mental model

- JavaScript is **dynamically typed**: any variable can hold any value.
- TypeScript is **statically typed (at compile time)**: you describe what values are allowed.

### How to think about “types” in daily work

- Use types to express *constraints* and *invariants*.
- Treat “external inputs” as `unknown` (JSON, env vars, user input).
- Convert external inputs to validated internal types early.

<details>
<summary>Show what “TypeScript types are erased” means</summary>

TypeScript does not change runtime behavior unless you also change the emitted JavaScript.

```ts
type Id = string;
const id: Id = "abc";
```

The emitted JS contains no `type Id` and no `: Id`:

```js
const id = "abc";
```

</details>

## Common terminology

- **Runtime**: the JS engine executing code.
- **Compile-time**: TypeScript checking types and producing JS.
- **Transpilation**: converting one language to another (TS -> JS).
- **Type narrowing**: TS refining a type based on checks (`typeof`, `in`, discriminated unions).

## Common pitfalls (JS and TS)

### 1. TypeScript cannot prevent all runtime errors

TypeScript checks the code you wrote, not the shape of data you receive.

<details>
<summary>Show the JSON boundary pitfall</summary>

```ts
type User = { id: string };

const raw: unknown = JSON.parse('{"id": 123}');
const user = raw as User;

// Compiles, but can fail later because id is not a string.
```

</details>

### 2. `any` spreads silently

If you use `any`, you lose type safety downstream. Prefer `unknown` + narrowing.

### 3. Types can lie

If you “force” types with assertions (`as Something`), you can create a false sense of safety.

## Strictness levels (TypeScript)

Most long-lived TS codebases enable `strict` mode.

Core idea:

- **Strict TS** pushes you to model nullability/optionality.
- This prevents common production bugs (undefined access).

## Summary

- JavaScript defines runtime behavior; TypeScript adds compile-time constraints.
- TypeScript does not change the runtime: types are erased.
- JS runs in multiple environments (browser/Node) with different APIs.
- TypeScript improves maintainability in larger or longer-lived codebases.

## Important Keywords

### **Runtime**

The JavaScript engine executing code (browser/Node).

### **Compile-time**

When TypeScript checks types and emits JavaScript.

### **Transpilation**

Converting TypeScript source into JavaScript output.

### **Type erasure**

TypeScript types do not exist at runtime.

### **Strict mode (TypeScript)**

Collection of compiler checks that strengthen type safety.
