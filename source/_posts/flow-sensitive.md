title: Grok control flow based analysis in TypeScript.
date: 2017-02-04
tags: TypeScript
---

TL;DR:
-----

1. Compiler does not understand control flow in closure / callback function.
2. In flow based type analysis, copmiler will be either optimistic or pessimistic. TypeScript is optimistic.
3. You can usually workaround (3) by using `const` or `readonly`

This is a long due introduction for TypeScript's flow sensitive typing (also known as control flow based type analysis) since its [2.0 release](https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript#control-flow-based-type-analysis). It is so unfortunate that no official documentation is in TypeScript's website for it (while both [flow](https://flowtype.org/docs/dynamic-type-tests.html#) and [kotlin](https://kotlinlang.org/docs/reference/typecasts.html#smart-casts) have!). But if you dig the issue list earnestly enough, you will always find some [hidden gems]( https://github.com/Microsoft/TypeScript/issues/9998) there!


To put it short, a variable's type in a [flow sensitive type system](https://www.wikiwand.com/en/Flow-sensitive_typing) can change according to the control flow like `if` or `while`. For example, you can dynamically check the truthiness of  a nullable variable and if it isn't null, compiler will automatically cast the variable type to non-null. Sweet?

What can be bitter here? TypeScript is an imperative language like JavaScript. The side-effectful nature prevents compiler from inferring control flow when function call kicks in. Let's see an example.

```ts

let a: number | null = 42

makeSideEffect()

a // is a still a number?

function makeSideEffect() {
  // omitted...
}
```

Without knowing what `makeSideEffect` is, we cannot guarantee variable `a` is still `number`. Side effect can be as innocuous and innocent as `console.log('the number of life', 42)`, or as evil as a billion dollar mistake like `a = null`, or even a control-flow entangler: `throw new Error('code unreachale')`.

One might ask compiler to infer what `makeSideEffect` does since we can provide the source of the function.
However this is not practically feasible because of [ambient function](https://basarat.gitbooks.io/typescript/content/docs/types/ambient/intro.html) and (possibly polymorphic) recursion. Compiler will be trapped in infinite loops if we instruct it to infer arbitrary deep functions, as [halting problem](https://www.wikiwand.com/en/Halting_problem) per se.


So a realistic compiler must guess what a function does by a consistent strategy. Naturally we have two alternatives:

1. Assume every function does **not** have relevant side effect: e.g. assignment like `a = null`. We call this **optimistic**.
2. Assume every function **does** have side effect. We call this strategy **pessimistic**.

Spoiler: TypeScript uses optimistic strategy.

We will walk through these two strategies and see how they work in practice.
But before that let's see some common gotchas in flow sensitive typing.

Closure / Callback
--------

Flow sensitive typing does not play well with callback functions or closures. This is explicitly mentioned in [Kotlin's document](https://kotlinlang.org/docs/reference/typecasts.html#smart-casts).

> var local variables - if the variable is not modified between the check and the usage and is not captured in a lambda that modifies it;

Consider the following example.

```TypeScript
var a: string | number = 42 // smart cast to number
setTimeout(() => {
  console.log(typeof a) // what should be print?
}, 100)
a = 'string'
```

As a developer, you can easily figure out that `string` will be output to console because `setTimeout` will call its function argument asynchronously, after assigning `string` to `a`. Unfortunately, this knowledge is not accessible to compiler. No keyword will tell compiler whether callback function will be called _immediately_, nor static analysis will tell the behavior of a function: `setTimeout` and `forEach` is the same in the view of compiler.

So the following example will not compile.

```ts
var a: string | number = 42 // smart cast to number
someArray.forEach(() => {
  a.toFixed() // error, string | number does not have method `toFixed`
})
```

Note: compiler will still inline control flow analysis for [IIFE(Immediately Invoked Function Expression)](https://developer.mozilla.org/en-US/docs/Glossary/IIFE).

```ts
let x: string | number = "OK";
(() => {
  x = 10;
})();
if (x === 10) { // OK, assignment in IIFE
}
```

In the future, we might have a keyword like [`immediate`](https://github.com/Microsoft/TypeScript/issues/11498) to help compiler reasoning more about control flow. But that's a different story.

Now let's review the strategies for function call.


Optimistic Flow Sensitive Typing
------

Optimistic flow typing assume a function without side-effect that changes a variable's type. TypeScript chooses this strategy in its implementation.

```ts
var a: number | null
a = 42 // assign, now a is narrowed to type `number`
sideEffect() // assume nothing happens here
a.toFixed() // a still has `number` type
```

This assumption usually works well if code observes immutable rule. On the other hand, a stateful program will be tolled with the tax of explicit casting. One typical example is scanner in a compiler. (Both Angular template compiler and TypeScript itself are victims).

```ts
// suppose we are tokenizing an HTML like language
enum Token { LeftBracket, WhiteSpace, Letter ... }
let token = Token.WhiteSpace;
function nextToken() {
    token = readInput(); // return a Token
}

function scan() {
  // token here is WhiteSpace
  while (token === Token.WhiteSpace) {
    // skip white space, a common scenario
    nextToken()
  }
  if (token === Token.LeftBracket) { // error here
    // compiler thinks token is still WhiteSpace, optimistically but wrongly
  }
}
```

Such bad behavior also occurs on fields.

```ts
// A function takes a string and try to parse
// if success, modify the result parameter to pass result to caller
declare function tryParse(x: string, result: { success: boolean; value: number; }): void;

function myFunc(x: string) {
    let result = { success: false, value: 0 };

    trySomething(x, result);
    if (result.success === true) { // error!
        return result.value;
    }
    return -1;
}
```

An alternative here is returning a new result object so we need no inline mutation. But in some performance sensitive code path might we want parse a string without new object allocation, which reduces garbage collection pressure. After all, mutation is legal in JavaScript code, but TypeScript fails to capture it.


Optimistic flow analysis sometimes is also unsound: a compiler verified program will cause runtime error. We can easily construct a function which reassigns a variable to an object of different type and uses it as of the original type, and thus a runtime error!

```ts
class A { a: string}
class B { b: string}
let ab: A | B = new A

doEvil()
ab.b.toString() // booooooom

function doEvil() {
  ab = new B
}
```

The above examples might leave to you a impression that compiler does much bad when doing optimistic control flow inference. In practice, however, a well architected program with disciplined control of side effect will not suffer much from compiler's naive optimistism. [Presumption of immutability innocence](https://www.wikiwand.com/en/Presumption_of_innocence) will save you a lot type casting or variable rebinding found in pessimistic flow sensitive typing.


Pessimistic Flow Sensitive Typing
-----

A pessimistic flow analysis places [burden of typing proof](https://www.wikiwand.com/en/Burden_of_proof) on programmers.
Every function call will invalidate previous control flow based narrowing. (*Pessimistic* possibly has a negative connotation, *conservative* may be a better word here). Thus programmers have to re-prove variable types is matching with previous control flow analysis.

Examples in this section are crafted to be runnable under both TS and flow-type checker.
Note, only flow-type checker will produce error because flow is more pessimistic/strict than TypeScript.

```ts
declare function log(obj: any): void

let a: number | string = 42

log(a) // invalidation!

a.toFixed() // error, a's type is reverted to `number | string`

// To work around it, you have to recheck the type of `a`
typeof a === 'number' && a.toFixed() // works
```

Alas, pessimistism also breaks fields. Example taken from [stackoverflow](http://stackoverflow.com/questions/38531182/weird-method-cannot-be-called-on-possibly-null-undefined-value).

```ts
declare function assert(obj: any): void

class TreeNode<V, E> {
    value: V
    children: Map<E, TreeNode<V,E>> | null

    constructor(value: V) {
        this.value = value
        this.children = null
    }
}

function accessChildren(tree: TreeNode<number, string>): void {
    if (tree.children != null) {
        assert(true) // negate type narrowing
        tree.children.forEach((v,k) => {}) // error!
    }

}
```

These false alarms root in the same problem as in optimistic strategy: compiler/checker has no knowledge about a function's side effect. To work with a pessimistic compiler, one has to assert/check repeatedly so to guarantee no runtime error will occur. Indeed, this is a trade-off between runtime safety and code bloat.


Workaround
-----

Sadly, no known panacea for flow sensitive typing. We can mitigate the problem by introducing more immutability.

### using `const`

Because a `const` identifier will never change its type.

```ts
const a: string | number = someAPICall() // smart cast to number
if (typeof a === 'string') {
  setTimeout(() => {
    a.substr(0) // success, `const` identifier will not lose its narrrowed type
  }, 100)
}
```

And using `const` will provide you runtime safety or bypass pessimistic checker.

```ts
function fn(x: string | null) {
  const y = x
  function assert() {
    // ... whatever
  }

  if (y !== null) {
    console.log(y.substr(0)); // no error, no crash
  }
}
```

The same should apply to `readonly`, but current TypeScript does not seem to support it.

Conclusion
-----

Flow sensitive typing is an advanced type system feature. Working with control flow analysis smoothly requires programmers to control mutation effectively.

Keeping mutation control in your mind. Flow sensitive typing will not in your way but make a pathway to a safer code base!
