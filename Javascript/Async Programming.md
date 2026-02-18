# 8. Async Programming

JavaScript is single-threaded but can do concurrency via the event loop. The core async abstractions are the same in JS and TS:

* callbacks
* Promises
* async/await

TypeScript adds typed Promises and typed async function return values.

## The mental model: single thread, async I/O

JavaScript typically runs your code on a single thread, but it can overlap work by:

* delegating I/O (network, disk, timers) to the host environment (browser/Node)
* resuming your code later via the event loop

This is different from CPU parallelism (multiple cores). If you block the event loop with heavy computation, everything “async” becomes slow.

## Promises\`\`\`js

function fetchUser() { return Promise.resolve({ id: 1, name: "Alice" }); }

fetchUser().then((u) => console.log(u.name)); `</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>`ts type User = { id: number; name: string };

function fetchUser(): Promise { return Promise.resolve({ id: 1, name: "Alice" }); }

fetchUser().then((u) => console.log(u.name)); `</div></div>## async/await<div data-gb-custom-block data-tag="tabs"><div data-gb-custom-block data-tag="tab" data-title='JavaScript'>`js async function main() { const user = await Promise.resolve({ id: 1, name: "Alice" }); return user.name; }

main().then(console.log); `</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>`ts async function main(): Promise { const user = await Promise.resolve({ id: 1, name: "Alice" }); return user.name; } \`\`\`### Common pitfall: forgetting to `await`

If you forget to `await`, errors may escape your `try/catch` because the function returns before the Promise settles.

{% tabs %}
{% tab title="JavaScript" %}
\`\`\`js async function run() { try { Promise.reject(new Error("boom")); return "ok"; } catch { return "caught"; } }

````

This does **not** catch the rejection.</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>```ts
async function run(): Promise<string> {
  try {
    await Promise.reject(new Error("boom"));
    return "ok";
  } catch {
    return "caught";
  }
}
````

TypeScript won’t automatically detect “missing await” in all cases, so this is primarily a discipline + linting concern.
{% endtab %}
{% endtabs %}

\## Error handling in async code

* In Promises: `.catch(...)`
* In async/await: `try/catch`

```js
async function load() {
  try {
    const data = await Promise.reject(new Error("boom"));
    return data;
  } catch (err) {
    return null;
  }
}
```

## Parallel vs sequential awaits

<details>

<summary>Show sequential vs parallel awaits</summary>

\`\`\`js const a = await fetchA(); const b = await fetchB(); \`\`\`\`\`\`js const \[a, b] = await Promise.all(\[fetchA(), fetchB()]); \`\`\`

</details>

## Promise.allSettled and failure handling

```js
const results = await Promise.allSettled([fetchA(), fetchB()]);
for (const r of results) {
  if (r.status === "fulfilled") {
    console.log(r.value);
  } else {
    console.error(r.reason);
  }
}
```

## Timeouts and cancellation

Promises don’t have built-in cancellation. In practice, you usually implement:

* **timeouts** (reject after a delay)
* **cancellation** via `AbortController` (supported by `fetch` and many APIs)

<details>

<summary>Show timeout wrapper</summary>

\`\`\`js function withTimeout(promise, ms) { return Promise.race(\[ promise, new Promise((\_, reject) => setTimeout(() => reject(new Error("timeout")), ms) ), ]); } \`\`\`\`\`\`ts function withTimeout(promise: Promise, ms: number): Promise { return Promise.race(\[ promise, new Promise((\_, reject) => setTimeout(() => reject(new Error("timeout")), ms) ), ]); } \`\`\`

</details>

<details>

<summary>Show AbortController with fetch</summary>

```js
const controller = new AbortController();
const id = setTimeout(() => controller.abort(), 500);

try {
  const res = await fetch("https://example.com", { signal: controller.signal });
  const data = await res.json();
  console.log(data);
} catch (err) {
  console.error(err);
} finally {
  clearTimeout(id);
}
```

</details>

## Typed async results (TS)

TypeScript can model fulfilled vs rejected results when you write helpers.

<details>

<summary>Show a typed Result helper</summary>

```ts
type Ok<T> = { ok: true; value: T };
type Err = { ok: false; error: unknown };
type Result<T> = Ok<T> | Err;

async function safe<T>(p: Promise<T>): Promise<Result<T>> {
  try {
    return { ok: true, value: await p };
  } catch (error) {
    return { ok: false, error };
  }
}
```

</details>

## Event loop (conceptual)

* JS runs your synchronous code first.
* Promise callbacks run as microtasks.
* Timers run as macrotasks.

This matters for ordering bugs.

<details>

<summary>Show microtask vs macrotask ordering</summary>

```js
console.log("sync start");

setTimeout(() => console.log("timeout"), 0);

Promise.resolve().then(() => console.log("microtask"));

console.log("sync end");

// Typical output:
// sync start
// sync end
// microtask
// timeout
```

</details>

## Concurrency limiting (practical)

When you have lots of async tasks (e.g., HTTP calls), you often want to limit concurrency.

<details>

<summary>Show a simple concurrency limiter</summary>

\`\`\`js async function mapLimit(items, limit, fn) { const results = \[]; const queue = \[...items];async function worker() { while (queue.length) { const item = queue.shift(); results.push(await fn(item)); } }const workers = Array.from({ length: limit }, () => worker()); await Promise.all(workers); return results; } \</div>\<div data-gb-custom-block data-tag="tab" data-title='TypeScript'>ts async function mapLimit\<T, R>(items: T\[], limit: number, fn: (item: T) => Promise): Promise\<R\[]> { const results: R\[] = \[]; const queue = \[...items];async function worker(): Promise { while (queue.length) { const item = queue.shift() as T; results.push(await fn(item)); } }const workers = Array.from({ length: limit }, () => worker()); await Promise.all(workers); return results; }## Async iterationIn modern JS, you can iterate over async streams with \`for await...of\`.\<details>\<summary>Show async generator + for await...of\</summary>\<div data-gb-custom-block data-tag="tabs">\<div data-gb-custom-block data-tag="tab" data-title='JavaScript'>\`\`\`jsasync function\* numbers() {  yield 1;  await new Promise((r) => setTimeout(r, 50));  yield 2;}for await (const n of numbers()) {  console.log(n);}\`\`\`\</div>\<div data-gb-custom-block data-tag="tab" data-title='TypeScript'>\`\`\`tsasync function\* numbers(): AsyncGenerator\<number> {  yield 1;  await new Promise((r) => setTimeout(r, 50));  yield 2;}\`\`\`\</div>\</div>\</details>## Summary- JS and TS share the same async runtime model.- TS adds safer APIs around Promise result shapes.- Prefer \`async/await\` for clarity, but understand \`Promise.all\` for performance.## Important Keywords### \*\*Event loop\*\*Mechanism that schedules and runs callbacks/tasks after the current synchronous code finishes.### \*\*Promise\*\*An object representing a value that may be available now, later, or never.### \*\*Microtask\*\*High-priority queued work (Promise \`.then\` callbacks) that runs before timers.### \*\*Macrotask\*\*Lower-priority queued work (e.g., timers, I/O callbacks).### \*\*Concurrency vs parallelism\*\*Concurrency overlaps waiting; parallelism runs simultaneously on multiple cores.### \*\*Cancellation\*\*Stopping ongoing async work. Often implemented via \`AbortController\` or cooperative flags.

</details>
