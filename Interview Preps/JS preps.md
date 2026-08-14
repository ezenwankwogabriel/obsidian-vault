# JavaScript Interview Prep — Variables & Event Loop

## 1. `var` vs `let` vs `const`

|                             | `var`       | `let` | `const` |
| --------------------------- | ----------- | ----- | ------- |
| Scope                       | Function    | Block | Block   |
| Reassign                    | ✅           | ✅     | ❌       |
| Redeclare                   | ✅           | ❌     | ❌       |
| Hoisted                     | ✅           | ✅     | ✅       |
| Initialized during hoisting | `undefined` | ❌     | ❌       |
| TDZ                         | ❌           | ✅     | ✅       |

### `var`

* Function-scoped, not block-scoped.
* Hoisted and initialized to `undefined`.
* Generally avoid in modern JavaScript.

```js
console.log(x); // undefined
var x = 10;
```

### `let`

* Block-scoped.
* Can be reassigned.
* Cannot be redeclared in the same scope.
* Exists in the **Temporal Dead Zone (TDZ)** until initialization.

```js
console.log(x); // ReferenceError
let x = 10;
```

### `const`

* Block-scoped.
* Cannot be reassigned.
* Must be initialized when declared.
* Objects/arrays can still be mutated.

```js
const user = { name: "A" };
user.name = "B"; // ✅

user = {}; // ❌
```

**Key interview point:**
`let` and `const` **are hoisted**, but unlike `var`, they aren't initialized during hoisting, so accessing them before declaration causes a `ReferenceError` due to the TDZ.

### Loop gotcha

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 3 3 3
```

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 0 1 2
```

`var` shares one binding; `let` creates a per-iteration binding.

---

# 2. JavaScript Event Loop

JavaScript executes **synchronous code first**.

After the current task finishes:

> **Drain all microtasks before processing the next task/macrotask.**

Mental model:

```text
Synchronous code
      ↓
Drain ALL microtasks
      ↓
Next task/macrotask
      ↓
Drain ALL microtasks
      ↓
Next task...
```

---

## Microtasks

Common examples:

```js
Promise.then()
Promise.catch()
Promise.finally()
queueMicrotask()
await continuation
```

## Macrotasks / Tasks

Common examples:

```js
setTimeout()
setInterval()
DOM events
```

---

## Classic interview question

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```

Output:

```text
A
D
C
B
```

Why?

1. Synchronous → `A`, `D`
2. Microtask → `C`
3. Next task → `B`

---

## Microtasks can schedule microtasks

```js
Promise.resolve().then(() => {
  console.log("A");

  Promise.resolve().then(() => {
    console.log("B");
  });
});

setTimeout(() => console.log("C"), 0);
```

Output:

```text
A
B
C
```

The event loop **drains the microtask queue completely** before moving to the next task.

This can theoretically cause **microtask starvation**.

---

# 3. `async/await`

`await` pauses the async function and its continuation runs asynchronously through the promise/microtask mechanism.

```js
async function test() {
  console.log("A");
  await Promise.resolve();
  console.log("B");
}

console.log("C");
test();
console.log("D");
```

Output:

```text
C
A
D
B
```

Think:

```text
C → A → D → microtask → B
```

---

# Must-remember interview answers

**What is the difference between `let` and `var`?**
`let` is block-scoped and has TDZ behavior; `var` is function-scoped and is initialized to `undefined` when hoisted.

**Is `let` hoisted?**
Yes. Its binding is hoisted, but it remains uninitialized in the TDZ until execution reaches its declaration.

**Is `const` immutable?**
No. The binding cannot be reassigned, but referenced objects/arrays can be mutated.

**What runs first: Promise callback or `setTimeout(..., 0)`?**
The Promise callback, because Promise callbacks are microtasks and timers are tasks/macrotasks.

**What happens after synchronous JavaScript finishes?**
The event loop drains **all microtasks** before moving to the next task.

**Does `setTimeout(fn, 0)` execute immediately?**
No. It schedules `fn` as a future task; it must wait for the current work and relevant microtasks to finish.
