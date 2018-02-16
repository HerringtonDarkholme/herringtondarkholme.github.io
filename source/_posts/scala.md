title: Scala Symbol Soup Salvage
date: 2014-4-15
tags: Scala
---

{%img /images/yandere.jpg%}
source: [yande.re](https://yande.re/post/show/284731/hatsune_miku-headphones-pantsu-shimapan-suemizu_yu)

By no several pages could one explain or exhausted the absurd bizarre creepy daunting esoteric flabbergasting type system in Scala.
Here are some aspects that Learn Scala by example does not cover.

## Digest:
http://twitter.github.io/scala_school/advanced-types.html

##  Context Bound and Implicitly:
`def sort[A : Ordered] => def sort[A](implicit x: Ordred[A])`
`def implicitly[T](implicit e: T) = e`
http://stackoverflow.com/questions/3855595/what-is-the-scala-identifier-implicitly

## Type Bound:
`=:= <:< <%<`
http://stackoverflow.com/questions/3427345/what-do-and-mean-in-scala-2-8-and-where-are-they-documented

## Existential Type:
`def foo(x: Array[_]) => def foo(x: Array[T] forSome {type T})`
http://www.drmaciver.com/2008/03/existential-types-in-scala/

## Self Type
`class BarUsingFooable {self: Fooable => ....} // yet-to-be Fooable, make this binded to self`
https://coderwall.com/p/t_rapw

## Structural Type
`def Quack(duck: {def quack: Unit})// inline anonymous class like ducktyping`
http://java.dzone.com/articles/duck-typing-scala-structural

## Abstract Type member
`trait Job {type A; def get: A}`
http://docs.scala-lang.org/tutorials/tour/abstract-types.html

## Type Level Programming:
http://apocalisp.wordpress.com/2010/06/08/type-level-programming-in-scala/


## Miscellaneous:
http://stackoverflow.com/questions/1025181/hidden-features-of-scala

## Last but not least:
[Site significantly serving Scala symbol soup salvage saves spiritually severed souls](http://symbolhound.com/)
