title: How to write copy-paste friendly code -- An introductory parody
date: 2016-11-05
tags: JavaScript
---

With the growing population of SOA ([StackOverflow](https://www.reddit.com/r/shittyprogramming/comments/4jstef/how_long_does_it_takes_to_master_stack_overflow/) [Oriented](https://twitter.com/divineomega/status/695744177557106688) [Architecture](https://github.com/drathier/stack-overflow-import)), keeping your code copy-paste friendly is becoming more and more important in demonstrating, developing and question answering.

You want your answer to be available instantly, free of fussy debugging. So copy-paste friendliness is a key property of your code robust enough to be ubiquitously runnable and, eventually, help you gain more reputation.

Adopting copy-paste style is amazing. Copy-paste boosts your productivity by freeing you from rumination on architecture complexity. Your boss will be happy with all the [SLOC](http://www.itestra.de/fileadmin/Redaktion/Documents/07_itestra_measuring_productivity_using_lines_of_code.pdf) you commit. Your colleagues will respect your absolute ownership of the magnificent, enigmatic, labyrinthine code artifact built by the blob gleaned from everywhere.

# 1. Always prefer "==" to strict equality.
Implicit conversion grants you additional robustness. Your code will never complain about ill input. Copy-paste without worry. Yeah

```javascript
var x = '10'
if (x == 10) x += 5
y = x / 5 // wow, it works!
```

And you can enjoy this artistic [equality table](https://dorey.github.io/JavaScript-Equality-Table/) when debugging. Cool.
{%img /images/comparison.png %}

## 2. Always prefer positive condition.
Never return early on invalid case. You can copy-paste it any where in your control flow. Yeah.

If `return` is added, you have to remove the unnecessary control flow abrupting keyword when you need to copy paste these code into a deeply nested code block, which is quite common in CPS(copy-paste-style) programming.

```javascript

function someFunction(someCondition) {
  if (someCondition) {
    // Do whatever you want
  }
}
```

## 3. Always prefer tautological check.
Repetition here is for genericity. Every conditional path can be copy-pasted free of reading. They are very safe compile time constructs that preclude the very possibility of runtime `undefined` panic.

`else` on individual line is a bonus. You can copy-and paste code without looking at `else` keyword! What a profit!

```javascript
if (a && a.b && a.b.c == 1) {
  // never worried about a.b.c is not undefined
}
else
if (a && a.b && a.b.c == 2) {
  // repition is one aesthetic rule
}
else
if (a && a.b && a.b.c == 3) {
  // very safe even programmer inadvertedly paste it to elsewhere

}
```


## 4. Always prefer fully qualified name
Never cache common subexpression. Declaring new variable name will force you read code and find declaration before copy-pasting.

```javascript
topVariable.someProperty.nestedProp.anotherProp = 123
console.log(topVariable.someProperty.nestedProp.anotherProp)
topVariable.someProperty.nestedProp.anotherProp += 1
```

## 5. Always prefer inlining statements in function
Never factor out functions. Helpers require nonlocal function name lookup when copy-pasting. Dev experience terminator.


## Conclusion
Oh, GG. I forgot GitHub Gists!
