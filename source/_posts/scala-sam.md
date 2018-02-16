title: Explaining Scala SAM type
date: 2015-01-24 16:21:47
tags: Scala
---

{%img /images/42665600.jpg%}
[source](http://www.pixiv.net/member_illust.php?mode=medium&illust_id=42665600)

For better syntax and user friendliness (and a lot more), Java 8 introduces [SAM](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-state-3.html) type, Single Abstract Method type, starting to embrace the functional programming world.

Previous to 1.8, Java already has somewhat a (bulky and leaky) type of closure: annonyous inner classes. For example, to start a working thread in Java usually requires following trivial but bloated statements, without a lexical `this`:

```java
// from android developer guide
public void onClick(View v) {
  new Thread(new Runnable() {
    public void run() {
      Bitmap b = loadImageFromNetwork("http://www.example.org/image.gif");
      mImageView.setImage(b);
    }
  }).start()
}
```

Two classes, one method. The introduction of SAM type greatly reduces the syntactical overhead of Java.

```java

public void onClick(View v) {
  new Thread(#{ ->
    mImage.setImage(loadImageFromNetwork("/image.gif"));
  }).start();
}
```

I'm not explaining Java8's new features here, as a Scala user has already relished the conciseness and expressiveness of functional programming.

So, why would scala user cares about SAM type in Java? I would say that interoperation with Java and performance are the main concerns here.

Java8 introduces `Function` type, which is widely used in library like `stream`. Sadly, `Function` is not that of scala. Scala compiler just frowns at you using scala native function with Stream.

```scala
import java.util.Arrays
Arrays.asList(1,2,3).stream.map((i: Int) => i * 2)

// <console>: error: type mismatch;
//  found   : Int => Int
//  required: java.util.function.Function[_ >: Int, _]
```

Side notes: because function parameter is in a contravariant position,  `Function[_ >: Int, _]` has a lower bound Int rather than a upper bound.
That is, the function passed as argument must accept types that are super types of `Int`.

One can manually provide implicit conversion here to transform scala function types to java function types. However, [implementing](http://www.tikalk.com/incubator/simulating-sam-closures-scala/) such implicit conversion is of no fun. The implementation is either not generic enough, or requires mechanical code duplication(another alternative is advanced macro generation). Compiler support is more ideal, not only because it generates more efficient byte code, but also because this precludes incompatibility across different implementations.


**SAM type is enabled by `-Xexperimental` flag in scala 2.11.x flags. Specifically, in scala 2.11.5, SAM type is better [supported](https://github.com/scala/scala/pull/4101).**
SAM gets eta-expansion(one can use a method from another class as SAM), overloading(overloaded function/method can also accept functions as SAM) and existential type support in scala 2.11.5.

Basic usage os SAM is quite simple, if a `trait/abstract class` with *exactly one* `abstract method`, then a `Function` of the *same parameter and return type* of the abstract method can be converted into the `trait/abstract class`.

```scala
trait Flyable {
  // exactly one abstracg method
  def fly(miles: Int): Unit
  // optional concrete object
  val name = "Unidentified Flyable Object"
}

// to reference SAM type itself
// create a named self-referencing lambda expression
val ufo: Flyable = (m: Int) => println(s"${ufo.name} flies $m miles!")
ufo.fly(123)
// Unidentified Flyable Object flies 123 miles!
```
Easy Peasy. So for the `stream` example, if compiler has the `-Xexperimental` flag, scala will automatically change the function to java's function, which grant scala user a seamless experience with the library.

Usually, you don't need SAM in scala, as scala already has first class generic function type, eta-expansion and a lot more. SAM reduces the readability as implicit conversion does. One can always use type alias to give function a more understandable name, instead of using SAM. SAM type cannot be pattern matched, [at least for now](https://issues.scala-lang.org/browse/SI-8429).

However, interoperating with Java requires SAM. And self-referencing SAM gives you additional flexibity in designing API. SAM also generates more efficient byte code since SAM has a native byte code counterpart. Using annonymous class for event handler or callback can be more pleasant in Scala just as in Java.

Anyway, adding a feature is [easy](https://github.com/scala/scala/pull/4101), but adding a feature that couples with edge case is hard. Scala already has bunches of features(variance, higher kinded, type level, continuation), whether SAM will gain its popularity is still an open question.
