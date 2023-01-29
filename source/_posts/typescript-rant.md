title: Rant on TypeScript type guard
date: 2015-1-17
tags: JavaScript
---
{%img /images/hisuitei.jpg%}
source: [Ad: hisuitei](http://hisuitei.com)

Recently TypeScript team has released TypeScript 1.4, adding a new feature called `Union Type` which is intended for better incorporation into native JavaScript.
Type guard, as a natural dyad of union type, also comes into TypeScript world. But sadly, Microsoft choose a bizzare way to introduce type guard as they said in the [manual](https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript%3F#typescript-14)

> TypeScript now understands these conditions and will change type inference accordingly when used in an if block.

It *accepts and only accepts* conditional statement like `if (typeof x === 'string')` as type guard.
TypeScript now creates new type like `number | string` meaning a type of either number or string.
Users can further refine the type by comparing the value `typeof` gives, like the example below:

```typescript
function createCustomer(name: { firstName: string; lastName: string } | string) {
    if (typeof name === "string") {
        // Because of the typeof check in the if, we know name has type string
        return { fullName: name };
    }
    else {
        // Since it's not a string, we know name has
        // type { firstName: string; lastName: string }
        return { fullName: name.firstName + " " + name.lastName };
    }
}

// Both customers have type { fullName: string }
var customer = createCustomer("John Smith");
var customer2 = createCustomer({ firstName: "Samuel", lastName: "Jones" });

```

I would rather say this is a bad idea because:

1. it intermixes type level constucts with value level constructs.
2. for complex and flexible language like javascript, microsoft's approach is unable to handle various expressions regarding types.

Value level contructs are expression or statements dealing with values, e.g. assignment, comparison. `typeof` and `instanceof` in JavaScript are value level constructs because they generate boolean value, and their values can be passed to other variable or compared with other variable. Value type constucts do imply types, say, creating a new object of specific type, but they do not explicitly manipulate the types of expressions. There is no type casting JavaScript can do. On the other hand, type level constructs deals with types, for example, type annotation and generics.

Doubling `typeof` as type guard blurs the demarcation between type level and value level, and naturally reduces program's readability(somewhat subjective claim though). A variable can, without distinct syntax, change its type in a conditional block. `if` branching is ubiquitous typescript programs, from hello-world toys to cathedral-like projects. It's quite hard to find the "type switch" for union type among other irrelevant `if`s. Also, one has to pay attention to call correct method of a same variable in different branches. So TypeScript's type guard introduces a new `type scope` different from both lexical scope and function scope. It also cripples the compiler because now compiler has to check whether the condition in a `if` parentheses is type guard.

What's worse? Type guard is a value level constructs so it can interact with all other language constucts. But microsoft does not intend to support that. None of the following code compiles in TypeScript 1.4.1, but they ought to run correctly in plain javascript, if they can be compiled.

```typescript
function testNot(x: string|number|Function) {
  var isNonNum = typeof x !== 'number'
  if (isNonNum) return x.length
}

function testReturn(x: string|number) {
  if (typeof x === 'number') return;
  return x.length
}

function testReturn(x: string|number) {
  if (typeof x === 'number') throw new Error('error type')
  return x.length
}

function testFor(xs: (string|number)[]) {
 for(var i = 0, x = xs[i]; typeof x === 'string'; i++) {
   console.log(x.length)
 }
}

function testWhile(xs: [](string|number)) {
  var i = 0;
  while (typeof xs[i] === 'string') {
    console.log(xs[i].length)
    i++
  }
}

function testFilter(xs: (string|number)[]) {
  xs.filter((x) => typeof x === 'string').map((x) => x.length)
}
```

Indeed, TypeScript is not the first to mix type level and value level. Language constructs like `Pattern Match` also do that(and usually introduce bugs related to type inference, see scala bug track). But at least Pattern Match is a specialized syntax that does not interact much with other syntax. But Type guard is, well, too ubiquitous to be good.
