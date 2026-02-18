# 6. Objects & Prototypes

JavaScript is prototype-based. TypeScript can _type_ both class-based and prototype-based patterns.

## Objects and property access\`\`\`js

const user = { name: "Alice", age: 25 };

console.log(user.name); console.log(user\["age"]); `</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>`ts type User = { name: string; age: number };

const user: User = { name: "Alice", age: 25 };

console.log(user.name);

````</div></div>###

- Dot access (`obj.key`) is for known, valid identifier names.
- Bracket access (`obj[key]`) is for dynamic keys.

<details>
<summary>Show dynamic keys and why they matter</summary><div data-gb-custom-block data-tag="tabs"><div data-gb-custom-block data-tag="tab" data-title='JavaScript'>```js
const key = "age";
const user = { name: "Alice", age: 25 };
console.log(user[key]);
```</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>```ts
type User = { name: string; age: number };
const key: keyof User = "age";

const user: User = { name: "Alice", age: 25 };
console.log(user[key]);
```</div></div></details>

## Destructuring<div data-gb-custom-block data-tag="tabs"><div data-gb-custom-block data-tag="tab" data-title='JavaScript'>```js
const user = { name: "Alice", age: 25 };
const { name, age } = user;
```</div><div data-gb-custom-block data-tag="tab" data-title='TypeScript'>```ts
type User = { name: string; age: number };

const user: User = { name: "Alice", age: 25 };
const { name, age } = user;
```</div></div>## Prototypes (what actually happens)

Every object has an internal prototype chain used for property lookup.

### Property lookup rules (high level)

When you read `obj.prop`, JavaScript does:

1. Check if `obj` itself has `prop`
2. If not, check `Object.getPrototypeOf(obj)` and repeat
3. Stop at `null` (end of chain)

This is why inheriting “works” without copying methods.

### Own properties vs inherited properties

`for...in` iterates enumerable properties, including inherited ones.
For “just my keys”, prefer `Object.keys()` (own enumerable only).

<details>
<summary>Show hasOwn / hasOwnProperty</summary>

```js
const obj = Object.create({ inherited: 1 });
obj.own = 2;

Object.keys(obj); // ["own"]
"inherited" in obj; // true

Object.hasOwn(obj, "own"); // true
Object.hasOwn(obj, "inherited"); // false
````

<details>

<summary>Show Object.create(null) dictionary gotcha</summary>

```js
const dict = Object.create(null);
dict["a"] = 1;

// dict.hasOwnProperty("a"); // TypeError: hasOwnProperty is not a function
Object.hasOwn(dict, "a"); // true
```

</details>

<details>

<summary>Show prototype inheritance in JS (and how TS types it)</summary>

\`\`\`js const animal = { speak() { return "..."; }, };const dog = Object.create(animal); dog.speak = function () { return "woof"; };console.log(dog.speak()); console.log(Object.getPrototypeOf(dog) === animal); \</div>\<div data-gb-custom-block data-tag="tab" data-title='TypeScript'>ts type Animal = { speak: () => string };const animal: Animal = { speak() { return "..."; }, };const dog = Object.create(animal) as Animal; dog.speak = () => "woof";console.log(dog.speak());## Classes (syntax sugar over prototypes)Classes are mostly syntax over prototypes.\<div data-gb-custom-block data-tag="tabs">\<div data-gb-custom-block data-tag="tab" data-title='JavaScript'>\`\`\`jsclass Person {  constructor(name) {    this.name = name;  }  greet() {    return \`Hi ${this.name}\`;  \}}const p = new Person("Alice");console.log(p.greet());\`\`\`\</div>\<div data-gb-custom-block data-tag="tab" data-title='TypeScript'>\`\`\`tsclass Person {  name: string;  constructor(name: string) {    this.name = name;  }  greet(): string {    return \`Hi ${this.name}\`;  \}}\`\`\`\</div>\</div>### Inheritance (extends) vs composition\`class\` supports \`extends\`, but in many codebases composition is preferred:- Inheritance couples subclasses to base class details.- Composition makes dependencies explicit.\<details>\<summary>Show inheritance vs composition (conceptual)\</summary>\<div data-gb-custom-block data-tag="tabs">\<div data-gb-custom-block data-tag="tab" data-title='Inheritance'>\`\`\`jsclass BaseLogger {  log(msg) {    console.log(msg);  \}}class AppLogger extends BaseLogger {  error(msg) {    this.log("ERROR: " + msg);  \}}\`\`\`\</div>\<div data-gb-custom-block data-tag="tab" data-title='Composition'>\`\`\`jsfunction createLogger(output) {  return {    log(msg) {      output(msg);    },    error(msg) {      output("ERROR: " + msg);    },  };}\`\`\`\</div>\</div>\</details>### \`this\` in methods (common pitfall)In JavaScript, \`this\` depends on how a function is called.When you pass a method as a callback, you can lose the binding.\<details>\<summary>Show method extraction pitfall\</summary>\`\`\`jsconst user = {  name: "Alice",  greet() {    return "hi " + this.name;  },};const greet = user.greet;// greet(); // often fails because this is undefined in strict mode

</details>

## Public/private fields

JS now has true private fields with `#`. TS also has `private` which is usually enforced at compile time.

<details>

<summary>Show JS #private vs TS private</summary>

\`\`\`js class Counter { #value = 0;inc() { this.#value += 1; return this.#value; } }const c = new Counter(); c.inc(); // c.#value; // SyntaxError \</div>\<div data-gb-custom-block data-tag="tab" data-title='TypeScript'>ts class Counter { private value = 0;inc(): number { this.value += 1; return this.value; } }const c = new Counter(); c.inc(); // c.value; // compile-time error\</details>## Interfaces and structural typingTypeScript interfaces describe shapes.\`\`\`tsinterface Logger {  log(message: string): void;}function run(logger: Logger) {  logger.log("running");}run({ log: (m) => console.log(m) });Property descriptors, getters, and settersProperties are not just “fields”. They can be computed via getters/setters.Index signatures and recordsWhen your object is a dictionary/map:const counts: Record\<string, number> = {};counts\["a"] = 1;Mutability and immutability patternsJavaScript objects are mutable by default.Common patterns:Treat objects as immutable in application code (create copies)Use Object.freeze for shallow freezingSummaryJS objects inherit via prototypes.Classes are common, but they still use prototype lookup.TS lets you model object shapes and catch missing/incorrect properties before runtime.Important KeywordsPrototypeInternal link used for property lookup (Object.getPrototypeOf(obj)).Prototype chainSequence of prototypes searched during property access.Own propertyProperty stored directly on the object (not inherited).Structural typingType compatibility based on shape (TypeScript).DescriptorMetadata controlling property behavior (writable/enumerable/configurable, getters/setters).CompositionBuilding behavior by combining objects/functions rather than inheriting from base classes.

</details>
