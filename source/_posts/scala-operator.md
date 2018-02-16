title: Scala Generalized Type constraints
date: 2014-09-30 20:23:16
tags: Scala
---

Generalized Type Constraints, also known as `<:<`, `<%<`(deprecated though) and `=:=`, also known as type relation operator, or call [whatever](http://stackoverflow.com/questions/2603003/operator-in-scala/) you want, are not operators but identifiers. It's quite confusing for new comers to distinguish them from operators, well..., identifiers which are not that esoteric.

This is just plain Scala feature that non-alphanum symbols can act as legal identifiers, just like `+` method.
More specifically, they are type-constructors. But before we inspect their implementations, let's first consider their usage.

Usage
======

You want to implement a generic container for every type, however, you also want to add a special method that only applies to `Special` type. (notice: this is different from the annotation `@specialized` which deals with JVM's primitive type. Here `Special` is just a plain old scala type)

```scala
class Container[A](value: A) {
  def diff[A <: Int](b: Int) = value - b
}

// BOOM
// error: value - is not a member of type parameter A
//    def diff[A <: Int](b: A) = value - b
```

Why? The type bound `A <: Int` does not work. `A` has been defined at the class declaration, in the class body Scala compiler requries every type bound is consistent with A's definition. Here, `A` has no bound so it is bounded by `Any`, not `Int`.

Instead of setting type bound, methods may ask for some kinds of specific *ad-hoc* "evidence" for a type.

```scala
scala> class Container[A](value: A) {
  // other generic methods for A
  /* blah blah */

  // specialized method for Int
  def addIt(implicit evidence: A =:= Int) = 123 + value
}
defined class Container

scala> (new Container(123)).addIt
res11: Int = 246

scala> (new Container("123")).addIt
<console>:10: error: could not find implicit value for parameter evidence: =:=[java.lang.String,Int]
```

Cool, `evidence` is an implicit provided by scala predef. And `A =:= Int` is just a type like `Map[Int, String]`, but is infixed due to scala's syntactic sugar.

Scala does not impose type constraints until the specific method is called, so `addIt` does not violate `A`'s definition. Still, given the implicit `evidence`, compiler can still infer that value in `addIt` is an sub-instance of `Int`.

As stated before, type constraints are *ad-hoc*. So it can achieve type inference more specific than type bound. (Fairly, this is the power of implicit).

```
def foo[A, B <: A](a: A, b: B) = (a,b)

scala> foo(1, List(1,2,3))
res1: (Any, List[Int]) = (1,List(1, 2, 3))
```
1 is clearly `Int` but why does compiler infer it as `Any`? The `B <: A` bound requires the first argument type is a super type of the second. `A` is inferred as the most general type between `Int` and `List[Int]`, `Any`.

`<:<` comes to help.

```
def bar[A,B](a: A, b: B)(implicit ev: B <:< A) = (a,b)

scala> bar(1,List(1,2,3))
<console>:9: error: Cannot prove that List[Int] <:< Int.
```

Because generalized type constraints does not interfere with inference, `A` is `Int` here. Only then does the compiler find evidence for `<:<[Int, List[Int]]` and then fails.
(Actually, implicit can feedback type information back to inference, see [typelevel programming's](http://apocalisp.wordpress.com/2010/07/17/type-level-programming-in-scala-part-6d-hlist%C2%A0zipunzip/) `HList` and scala collection library's `CanBuildFrom`)

Also implicit conversion does not impact `<:<`

```
scala> def foo[B, A<:B] (a:A,b:B) = print("OK")

scala> class A; class B;

scala> implicit def a2b(a:A) = new B

scala> foo(new A, new B)  // implicit conversion!
OK

scala> def bar[A,B](a:A,b:B)(implicit ev: A<:<B) = print("OK")

scala> bar(new A, new B)  // does not work
<console>:17: error: Cannot prove that A <:< B.
```


Implementation
=====

Actually `=:=` is just a type constructor in scala.
It is somewhat like `Map[A, B]`, that is,
`=:=` is defined like

```
  class =:=[A, B]
```

so in the implictly's bracket, `Int =:= Int` is just a type
`A =:= B` is the infix form of type parameterization for
non-alphanumeric identifier. It is equivalent to `=:=[A, B]`

so one can define implicts for `=:=`, so that compiler can find

```
implicit def EqualTypeEvidence[A]: =:=[A, A] = new =:=[A, A]
```

So, when `implictly[A =:= B]` is compiled,
compiler tries to find the correct implicit evidence.

If and only If A and B are the same, say Int, the compiler can find
`=:=[Int, Int]`, by the result of `implicit function EqualTypeEvidence[Int]`

More compelling is <:<, the conformance evidence,
it leverages variance annotation in scala

```
class <:<[-A, +B]
implict def Conformance[A]: <:<[A, A] = new <:<[A, A]
```

Consider, when `String <:< java.io.Serializable` is needed,
compiler tries to find an instance of `<:<[String, j.i.Serializable]`
It can only find instance of the type `<:<[String, String]`
(or another alternative `<:<[Serializable, Serializable]`)
But given the variance annotation of <:<,
since String is the very type String
and String is a subtype of Serializable and B is in a covariant position
, or, in another direction
snice Serializable is a supertype of String and A is in a contravariant position
and Serializable is the very type Serializable


`<:<[String, String]` is a subtype of `<:<[String, Serializable]`
So compiler finds the correct implicit instance as the evidence that
String is a subtype of Serializable. By the principle of subtype subsititution.
(Liskov)

Similarly we can define

```
Conversion evidence

class <%<[A <% B, B]
implicit def Conversion[A, B] = new <%<[A, B]

Contra-conformance
class >:>[+A, -B]
implicit def Contra[A] = new >:>[A, A]
```

Magic, Right?
The actual implementations uses singleton pattern so it is more efficient. For this illustration post, sloppy implementation is just fine :).

Reference:
http://hongjiang.info/scala-type-contraints-and-specialized-methods/
http://apocalisp.wordpress.com/2010/07/17/type-level-programming-in-scala-part-6d-hlist%C2%A0zipunzip/
