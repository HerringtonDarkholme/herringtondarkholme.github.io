title: typelevel-html
date: 2014-11-01 18:26:14
tags: Scala
---

Type level programming is a technique that exploits type system to represents information and logic, to the extent of language's limit.

Since values are encoded by variable's type, type level drives compiler to validate logic or even determine program's output.
All validation and computation are conducted statically at compiling phase, so the greatest benefit of type-level programming is its safety and reliability.

As a rule of thumb, the more dynamic code is, the more flexible it can computes. Type level programming require all logic encoded into the source code. It is to hard to cram all logic into type system, as handling whimsical input from external source is either impossible or reduce source code to unwieldy state machine. So the niche of type level programming is usually encoding, pickling or parsing.

But there is field where code is statically written: GUI. HTML template is hard coded in source. Type level can be used as linting and validation in html edition, especially in authoring web components. A piece of HTML fragment can be encoded in ordinary object, with type denoting its structure. Once the structure of HTMl is fixed, the output of JavaScript and css can be determined as well.
This is more helpful when one wants to make component. A tab-container must have tab-pane as child, and a tab-pane must live within a tab-container. Current approach of constraining HTML structure is encoding requirements in JavaScript and checking it in runtime. For example, angular uses `require: '^parentDirective'` to express the constraints and enable directive communication. If the component is programmatically constructed, using type annotation is a natural way to express the constraints. (As in Angular 2.0, `query<ChildDirective>`). We can go further in a language with full-bloomed type system.

```
trait Tag
class Concat[A <: Tag, B <: Tag](a: A, b: B) extends Tag
trait NestTag[A <: Tag] extends Tag {
  type Child = A
}
trait Inline extends Tag
trait Block extends Tag
case class div[T <: Tag](t :T = null) extends NestTag[T] with Block
case class p[T <: Tag](t :T = null) extends NestTag[T] with Block
case class a[T <: Inline](t :T = null) extends NestTag[T] with Inline

implicit class PlusTag[A <: Tag](a: A) {
  def +[B <: Tag](b: B) = new Concat(a, b)
}

class Contains[A <: Tag, C[_ <: Tag] <: NestTag[_], T[_ <: Tag] <: NestTag[_]]

case class jQ[A <: Tag, C[_ <: Tag] <: NestTag[_]](c: C[A]) {
  def has[T[_ <: Tag] <: NestTag[_]](implicit ev: Contains[A, C, T]) = true
}

implicit def htmlEq[A <: Tag, C[_ <: Tag] <: NestTag[_], T[_ <: Tag] <: NestTag[_]](implicit ev: C[A] =:= T[A]) =
  new Contains[A, C, T]
implicit def recurEq[A <: Tag, B[_ <: Tag] <: NestTag[_], C[_ <: Tag] <: NestTag[_], T[_ <: Tag] <: NestTag[_]]
  (implicit ev: Contains[A, B, T]) = new Contains[B[A], C, T]

val ele = div(
  p(
    a()
  )
)
val r = jQ(ele).has[p]
println(r)
```

The code above is just a demo. All html elements has type that denotes its structure. And one can tell whether a tag is in a html elemnt by calling `jQ(ele).has[Tag]`. (Note: ele is value level variable and Tag is a type level constructor.) And inline element cannot contains block element, because inline element's child must be a subtype of `inline`.

Programmatical markup has several benefits:
  1. no switching between script and template
  2. static type checking
  3. component dependency requirements
  4. component communication
  5. subtyping, inheritance... Classical OOP features
  6. relatively clean layout (though not as concise as Jade/Slim)

The biggest problem is, well, type level templates is strongly constrained by host language. Dynamic languages simply cannot have it. Users of classical static typed language without type inference cannot afford the verbosity of deeply nested type. And languages capable of type level have different approaches and implementation towards Type Level Programming.

After all, type level is too crazy..., at least for daily business logic.
