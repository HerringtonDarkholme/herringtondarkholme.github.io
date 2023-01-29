title: Paamayim Nekudota Operator in ES7
date: 2015-06-09
tags: JavaScript
---

{%img /images/50799930.jpg%}
[Source: ななろば華 - 雨上がりタペストリー](http://www.pixiv.net/member_illust.php?mode=medium&illust_id=50799930)

Recently [Babel](http://babeljs.io/blog/2015/05/14/function-bind/) supports [Function Bind Syntax](https://github.com/zenparsing/es-function-bind) in ES7. A.K.A [Paamayim Nekudotayim](http://en.wikipedia.org/wiki/Scope_resolution_operator) Operator.(it should be Paamayim Nekudatayim, though)

It inherits the spirit of Golang's method, Switf's `extension`, Scala's `implicit class`, Ruby's `instance_exec`, Haskell's typeclass. While ES6 normalizes, and thus constrains, inheritance in JavaScript, `::` brings about ad-hoc virtual method to extends class's behavior.

Thanks to JavaScript's prototype based and dynamic typing nature, Paamayim Nekudata introduces least semantic complexity and grants best expressiveness. It is much more concise than `method.call` in previous JS or `instance_exec` in Ruby. Using double colon `::`, other than dot `.` provides visual cue to the source and definition of virtual method, which is more clear and less confusing than Swift's `extension` or Scala's `implicit class`. Extension to native object is trivial if one uses this new syntax. We can easily write a underscore like itertool library and apply its API directly on native array. This cannot be done without hacking (or screwing up) Array.prototype in ES6-. Both proposal and implementation on Function Bind Syntax are easy and straightforward, again, thanks to JS' nature.

However, Function Bind Syntax does not work well with type checking, for now. Current candidate proposals on ES type system does not cover `this` keyword in function body. TypeScript simply waives type checking or forbids referencing `this` in function body. Flow is the only type checker that open method which is aware of `this` context. However, the type checking on open method is implicit to code authors. One cannot explicitly annotate function's this type and type checking on open method is a sole compiler's matter.

Nontheless, it is great feature! I'm lovin it! Try it out!

Source: http://babeljs.io/blog/2015/05/14/function-bind/
