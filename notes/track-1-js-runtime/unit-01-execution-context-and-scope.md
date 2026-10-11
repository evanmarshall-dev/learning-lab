# Track 1, Unit 1: Execution context and scope

Written: 2026-10-09. Repo location: `notes/track-1-js-runtime/unit-01-execution-context-and-scope.md` [ASSUMPTION: folder name; rename freely].

Conventions: [ASSUMPTION] marks an inference. [TO FILL] marks something you complete.

---

## Part 1: Theory

### The problem scope solves

A real app has thousands of names: `config`, `total`, `user`, `init`. If every name lived in one shared space, any file could overwrite any other file's variables, and you could never reason about a function without reading the whole codebase. Scope is the rule set that decides **where a name is visible**. Good scoping lets you read one function and know exactly which variables it can touch. That is what makes large codebases (like your day job's 20+ services) workable.

### 1. Execution contexts and the call stack

When JavaScript runs code, it tracks each piece of running code in an **execution context** (also called a stack frame). Each one records the code being run, its variable bindings (`var`, `let`, `const`, `function`, `class`), and its `this` value. ([MDN: Execution model](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Execution_model))

Contexts live on the **call stack**, last in, first out:

- The script or module itself gets the first context.
- Every function call pushes a new context on top.
- When a function returns, its context is popped and control goes back to the one below.

Why you care: the **Call Stack panel** in Chrome DevTools, VS Code, and Rider is a live view of exactly this stack. A stack trace in an error message is a snapshot of it at the moment of the throw.

### 2. Lexical scope and the scope chain

Every function, every block `{ }`, and the script or module as a whole has its own **lexical environment**. It holds that region's variables, plus a reference to the environment it was _written inside_. ([javascript.info: Variable scope, closure](https://javascript.info/closure))

When code reads a name, the engine looks in the current environment, then the outer one, then the next one out, until it reaches the global level. If the name isn't found anywhere, you get `ReferenceError: x is not defined`. That outward walk is the **scope chain**.

**The rule that matters most:** the chain follows where the code is **written**, not who **called** it. "Lexical" means it's decided by position in the source code. ([MDN: Closures, Lexical scoping](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures#lexical_scoping))

```js
const greeting = "hi";
function outer() {
  const name = "Evan";
  inner(); // outer CALLS inner...
}
function inner() {
  console.log(greeting); // found: module level
  console.log(name); // ReferenceError: inner was not WRITTEN inside outer
}
```

So the call stack and the scope chain are **two different things**: the call stack says who called whom, and the scope chain says what is visible. The DevTools **Scope panel** shows the scope chain (Local, Block, Closure, Script or Module, Global), separately from the Call Stack panel.

### 3. `var` versus `let` and `const`

|                               | `var`                                         | `let` / `const`                              |
| ----------------------------- | --------------------------------------------- | -------------------------------------------- |
| Scope                         | Whole enclosing function (blocks ignored)     | Nearest block `{ }`                          |
| Before its declaration line   | Reads `undefined`                             | Throws `ReferenceError` (temporal dead zone) |
| Redeclare in same scope       | Allowed silently                              | `SyntaxError`                                |
| Top level of a classic script | Becomes a property of `globalThis` (`window`) | Does not                                     |

Sources: [MDN: let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let), [MDN: var](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var).

**Hoisting and the temporal dead zone (TDZ).**

- **Every declaration** is registered when its scope is entered, before any line in that scope runs.
- **`var`** starts out as `undefined`.
- **`let`, `const`, and `class`** stay uninitialized until execution reaches their line. Touching one before that throws. That window is the TDZ.
- **Function declarations** are fully hoisted: you can call them before their line.

"Temporal" is the key word. The TDZ depends on _when_ code runs, not where it's written. A function that reads a `const` is fine if it's called after the `const` line has executed, and throws if it's called before. ([MDN: let, TDZ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz))

### 4. Global scope versus module scope

There are two kinds of top-level code, and they behave differently.

**Classic scripts** (`<script src="a.js">`):

- All classic scripts on a page share **one global scope**.
- Top-level `var` and `function` declarations from every script land in it, and on `window`.
- Two scripts declaring the same name collide. The last one to run wins.

**ES modules** (`<script type="module">`, `import`/`export`, Vite, Node with ESM):

- Each file gets **its own module scope**. Top-level names stay private to the file unless exported.
- Modules run in **strict mode** automatically. In strict mode, assigning to an undeclared variable (`total = 0` with no declaration) throws a `ReferenceError` instead of quietly creating a global.

Sources: [MDN: JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules), [MDN: Strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode).

**Shadowing:** declaring an inner variable with the same name as an outer one. The inner one hides the outer one for the whole inner scope. It's legal, and it's a common source of bugs. ESLint has a rule for it, `no-shadow`. It's not in ESLint's recommended set, so a team has to opt in. ([ESLint: no-shadow](https://eslint.org/docs/latest/rules/no-shadow))

### The rules to carry forward

1. **Lookup is lexical.** A function sees what is around it in the source, never what is around its caller.
2. **Keep scope narrow.** Default to `const`, use `let` only when you reassign, and don't use `var` in new code. Declare variables in the smallest block that needs them. The fewer places a name is visible, the fewer places can break it. ([MDN code style guide: variable declarations](https://developer.mozilla.org/en-US/docs/MDN/Writing_guidelines/Code_style_guide/JavaScript#variable_declarations); Simpson, _You Don't Know JS Yet: Scope & Closures_, ch. 6 "Limiting Scope Exposure")

### Further reading (optional depth)

- _You Don't Know JS Yet: Scope & Closures_, 2nd ed., chapters 1 to 5. Free on GitHub: https://github.com/getify/you-dont-know-js/tree/2nd-ed/scope-closures
- javascript.info, "Variable scope, closure". Its lexical environment diagrams are the clearest visual model available: https://javascript.info/closure
- ECMAScript spec, sections 9.1 (Environment Records) and 9.4 (Execution Contexts). Reference only; you don't need to read it now: https://tc39.es/ecma262/#sec-execution-contexts
- "JavaScript: Understanding the Weird Parts": the execution context and scope chain sections. [ASSUMPTION: section names not checked against the current syllabus]

---

## Part 2: Exercises

**Rules:**

- No running code until you've written an answer for every question.
- Each answer has two parts: the prediction, plus **why**, using the terms above (execution context, call stack, scope chain, lexical, TDZ, module scope).
- Then verify each one in the browser, using the debugger (see "Verify" at the end).

### Exercise 1: The checkout crash

A teammate's cart code throws when a user clicks Checkout.

```js
// cart.js (ES module)
const TAX_RATE = 0.13;

export function renderCart(cart) {
  const currency = "CAD";
  const rows = cart.items.map((item) => formatLine(item));
  return rows.join("\n");
}

function formatLine(item) {
  const total = item.price * item.qty * (1 + TAX_RATE);
  return `${item.name}: ${total.toFixed(2)} ${currency}`;
}
```

```js
// main.js (ES module)
import { renderCart } from "./cart.js";

const cart = { items: [{ name: "Mug", price: 12, qty: 2 }] };

document.querySelector("#checkout").addEventListener("click", () => {
  console.log(renderCart(cart));
});
```

1. What error is thrown, and on which line?
2. At the moment it throws, write out the call stack from top to bottom. Include every frame, even ones that aren't your code.
3. List, in order, the scopes the engine searches for `currency` from inside `formatLine`. Explain why `renderCart`'s scope is not in that list, even though `renderCart` is on the call stack.
4. Describe two different fixes. Which would you approve in a code review, and why?

### Exercise 2: The staging request that hits production

```js
// api.js (ES module)
const apiBaseUrl = "https://api.example.com";

export async function getUsers(env) {
  if (env === "staging") {
    const apiBaseUrl = "https://staging-api.example.com";
    console.log("Using staging");
  }
  const res = await fetch(`${apiBaseUrl}/users`);
  return res.json();
}
```

1. `getUsers('staging')` logs "Using staging". Which URL is actually requested? Why?
2. Which field in the DevTools Network tab would prove your answer without opening the code?
3. Someone "fixes" it by changing the inner `const` to `var`. Now what URL does `getUsers('staging')` request? What URL does `getUsers('production')` request? Explain both, using hoisting and function scope.
4. Write a version of `getUsers` that works correctly for both environments and shadows nothing.

### Exercise 3: Two scripts on a legacy page

A client's CMS page loads a third-party analytics file and your app file as classic scripts:

```html
<script src="analytics.js"></script>
<script src="app.js"></script>
```

```js
// analytics.js
var config = { endpoint: "/collect" };
function init() {
  console.log("analytics ready", config.endpoint);
}
init();
document.addEventListener("click", () => {
  console.log("analytics ->", config.endpoint);
});
```

```js
// app.js
var config = { endpoint: "/api" };
function init() {
  console.log("app ready", config.endpoint);
}
init();
```

1. Predict the console output on page load. Then predict it when the user clicks anywhere.
2. Explain the click result, using what you know about classic-script global scope.
3. Both files are changed to use `const config` (still classic scripts). What happens now, and in which file?
4. Instead, both `<script>` tags get `type="module"`, with the files otherwise unchanged. Predict the load and click output, and explain what changed.

### Exercise 4: Broken for new visitors only

```js
// settings.js (ES module)
applyTheme(); // line A

document.querySelector("#theme-toggle").addEventListener("click", applyTheme);

const DEFAULT_THEME = "light";

function applyTheme() {
  document.documentElement.dataset.theme =
    localStorage.getItem("theme") ?? DEFAULT_THEME;
}
```

A first-time visitor reports two problems: the page has no theme applied, and the toggle button does nothing. You can't reproduce either one on your machine.

1. Why can `applyTheme` be called on line A at all, before its declaration?
2. Explain exactly why it breaks for a first-time visitor and works for you. (Hint: when does the right side of `??` get evaluated?)
3. Why is the toggle _also_ broken, even though the toggle only calls `applyTheme` later?
4. Using debugging-playbook.md step 1, how would you reproduce it on your machine?
5. Describe the fix, and name an ESLint rule that would have caught this before review.

### Verify (after answering)

1. **Serve the files over HTTP.** Module scripts fail to load from `file://` URLs in Chrome, so use VS Code's Live Server or run `npx serve` in the folder.
2. **Exercise 1:** set a breakpoint on the `return` line in `formatLine`, then click Checkout. Compare the **Call Stack** panel and the **Scope** panel against your answers to questions 2 and 3.
3. **Exercise 2:** check the request URL in the Network tab. The requests will fail against example.com; that's fine, the URL is what you're checking.
4. **Exercise 3:** reload the page and click, once with classic scripts and once with `type="module"`.
5. **Exercise 4:** follow your own reproduction steps from question 4.
6. **Log any wrong answer below.** A wrong prediction is the most useful data this unit produces.

---

## Part 3: My words [TO FILL after the exercises]

Write these without looking at Part 1.

- What an execution context is, in two sentences: [TO FILL]
- The difference between the call stack and the scope chain, with an example from the exercises: [TO FILL]
- Why `let`/`const` exist when `var` already did: [TO FILL]
- What the TDZ is, and why "temporal" is the right word: [TO FILL]
- Why ES modules fixed the classic-script collision problem: [TO FILL]

## Misconceptions found [TO FILL]

- [TO FILL] e.g. "I thought a called function could see its caller's variables."

## Exit criteria covered by this unit

From roadmap.md, Track 1: "Explain an execution context and scope chain." You've met it when you can do Part 3 out loud, without notes.
