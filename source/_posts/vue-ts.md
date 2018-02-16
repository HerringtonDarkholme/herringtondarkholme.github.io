title: Type Safety in Vue.js
date: 2016-08-21
tags: TypeScript
---

Static typing has become a hot word in frontend land: hundreds of tweets and blogs appears on social network and _XXXX weekly_, rivaling type checkers compete their features with each other.

Correspondingly new frameworks have a consciousness of type safety in their API design.
[Angular2](https://angular.io/) has partial type safety in ViewModel code notwithstanding template code. (There has been [some efforts](https://github.com/Microsoft/TypeScript/issues/6508) to pursue more type safety, though)

[React](https://facebook.github.io/react/index.html) has full, and strict, when checked with [flow](https://flowtype.org/docs/react.html), type safety by embedding templates in JavaScript(X).

But stakeholders of [Vue.js](https://rc.vuejs.org/), ~~the thirdwhee..~~ another popular MVVM framework , might be disappointed by Vue's type-checker hostility...

Type Checker Hostile API
----

Vue provides a set of simple and elegant [API](https://rc.vuejs.org/api/) via heavy use of reflection that extinguishes compiler's type inference.

It's not Vue's fault. Up to now static type checkers in JavaScript land have several limitations:
1. They cannot understand modification to objects' type or perform key-wise type inference. (more elaboration later)
2. Some cannot annotate function's `this` type. (not in [flowtype 0.30](https://github.com/facebook/flow/issues/452), supported in TypeScript)
3. Some cannot annotate [Type Property Type](https://github.com/Microsoft/TypeScript/issues/1295). (not in TypeScript until [#10425](https://github.com/Microsoft/TypeScript/pull/10425) is merged, [flowtype](https://github.com/facebook/flow/blob/master/src/typing/type_annotation.ml) has [undocumented $Magic](http://sitr.us/2015/05/31/advanced-features-in-flow.html) type like [$Keys](https://github.com/facebook/flow/blob/master/lib/react.js#L49), $Record)

For point1, an example will elucidate itself.

Suppose we are going to provide a type definition file for Vue's [config option](https://rc.vuejs.org/api/#Options-Data).

```typescript
interface VueConfig<D, P, PD, C, M, W> {
  data?: D
  props?: P
  propData?: PD
  computed?: C & {[k: string]: (this:D & P & C & M ) => {}}
  methods?: M & {[k: string]: (this:D & P & C & M) => {}}
  watch?: W & {[k: string]: (this:D & P & C & M) => {}}
}
```
This is not very precise but does highlight some basic idea of Vue's API. `computed` is a field of which the value is a function with `this` pointed to the object that has mixed in `data`, `props` and `method`. `this` in Vue's option is an object made out of reflection.

Then we will write some function like this.
```typescript
function getVue<D, P, PD, C, M, W>(opt: VueConfig<D, P, PD, C, M, W>): D & P & C & M
  {
  return null // placeholder
}

let a = getVueConfig({
  data: {
    a: 123,
  },
  watch: {
    a: function() {
      console.log(this.a) // oops
    }
  }
})
```

Compiler will complain about `this.a` in the watch function. Why? `this` cannot be inferred. To infer `this`, compiler will have to first infer `D, P, C, M` respectively. To infer `D & P & C & M`, compiler will have to first infer the whole expression for resolving all the type arguments. But to infer the whole expression we need first infer `watch`, where we need to infer `this`. So comes a recursion. Compiler cannot be too eager to infer type otherwise it will jump into a recursion trap. Sloth is a virtue here, even [Betelgeuse](http://rezero.wikia.com/wiki/Petelgeuse_Romanee-Conti) cannot blame.


Alternative API
----

Vue's original API is doomed to be hard to infer. However, we can build a thin layer of wrapper to leverage type checkers.

I have two alternatives to present here. One is chaining DSL, a novel approach to induct type checking and inference into Vue. The other alternative is more established and angular like: class decorator.


Chaining DSL
----

We can work around the recursion problem by nudging compiler to do more diligent work. Because every method/function call return a new type symbol, we can use it to escape from recursion trap:

Definition:
```typescript
declare class VueTyped<T> {
  data<D>(d: D): VueTyped<T & D>
  method<M extends {[k:string]: (this: T) => any}>(m: M): VueTyped<T & M>
  get(): T
  static new(): VueTyped<{}>
}
```

Usage:

```typescript
VueTyped.new()
  .data({msg: 'hehehe'})
   // return a new type symbol with field `msg`
  .method({method() {return 'hello: ' + this.msg}})
  // create a new type symbol with `msg` and `method`
  .get() // type as {msg: string, method(): string}
 ```

 Quick explanation. `data` has a signature like `data<D>(d: D): VueTyped<D & T>`. The intersection type in return position mocks `mixin` behavior. `method<M extends {[k:string]: (this: T) => any}>(m: M): VueTyped<T & M>` is more complicated. Parameter `M` is required for compiler to garner properties of option passed to `method` call. `M` is bounded by a constraint that every function in `option` must have `this` typed as the object we defined previously and that only defined property can be accessed via `this`. The final returning intersection type acts the same as `data`.

> Note: `method` does not work for current TypeScript. Probably it is a [bug](https://github.com/Microsoft/TypeScript/issues/10461)

 But step-wise inference still cannot resolve `watch` and `computed` property. `$Keys` magic type or `keysof` type does not exist in TS yet! Meanwhile, flowtype does not support `this`-typed function.

`computed` option is even harder to handle. There is no way to define `this` type in a getter/setter method. If we do not pass a getter/setter but a plain function as value, we cannot merge the computed properties into the resulting object.

A verbose workaround is [forward reference](http://blog.thoughtram.io/angular/2015/09/03/forward-references-in-angular-2.html) in type annotation:

```typescript
var a = VueTyped.new()
  .data({ msg: 'hehe' })
  .computed({
    get computed(this: typeof a) { // forward reference
      return this.msg + ' from WET computed!'
    }
  })
  .get()
```

It's not *DRY*.

Furthermore, this approach does not support language service feature like "looking for definition" or "finding all usage" because intersection type needs casting in implementation to work.

> **Irreparable! Irredeemable! Irremediable!**

However, this approach has some benefits. First, it is easier to extend its functionality. If one would like to add `vuex` field in option, it just requires defining a new method. It also prevents cyclic dependency because you cannot use fields before declaring. The API itself is akin to its original version, and thusly the implementation is very thin.


Class Decorator
----
This approach is much more conventional, and is discussed broadly in Vue's [issue](https://github.com/vuejs/vue/issues/478).

The basic idea is to define as many methods as possible in a class and to decorate fields to add Vue specific logic.

One exceptional API is provided by [itsFrank's vue-typescript](https://github.com/vuejs/vue/issues/478)

```typescript
import * as Vue from 'vue'
import { VueComponent, Prop } from 'vue-typescript'

@VueComponent
class MyComponent {
    @Prop someProp:string;

    @Prop({
        type: String
    })
    someDefaultProp:string = 'some default value';

    @Prop someObjProp:{some_default:string} = {some_default: 'value'}; //vue-typescript makes sure to deep clone default values for array and object types

    @Prop someFuncProp(){ //defined functions decorated with prop are treated as the default value
        console.log('logged from default function!');
    }

    someVar:string = 'Hello!';

    doStuff() {
        console.log('I did stuff');
    }
}
```

This TSish approach enables more capability of compiler tooling such as usage finding and definition lookup. Class decorator also guarantees every method's `this` correctly points to class instance, which cannot be achieved in **Chaining DSL** approach.

With higher abstraction comes more confusion. Indeed, class decorator smooths out the discrepancy between Vue and type checker. But syntactically its API is much further from Vue's original one. Adding new API is also harder because every decorator is [hard coded](https://github.com/itsFrank/vue-typescript/blob/master/src/vuecomponent.ts#L31) in `VueComponent` decorator's code. For example, adding `@vuex` is almost impossible without rolling out a new `VuexComponent`. It also cannot transform all Vue's API, such as `watch` and [`computed: { cache: false }`](https://vuejs.org/guide/reactivity.html#Inside-Computed-Properties), into idiomatic TypeScript, leaving some orifices in type safety.

~~I have an alternative API bike shedding but not ready to present. Maybe I will try it later.~~

Conclusion
------

This article presents type-safety problem in Vue and two ways to mitigate the problem. Rewritten in ES015 and type-checked by one of the most advanced type checkers, Vue is designed in ES5 era and, satirically, is still [designed for](https://github.com/vuejs/vue/issues/478#issuecomment-129498731) ES5 code.

Vue doesn't come with type safety in mind. But this is might be a mirroring of some part in the community where some developers have almost kind of _Stockholm Syndrome_: they encounter so many type unsafe ordeals that they are very happy and proud with their lavish use of reflection which backfires to themselves.


Yet one should always keep a leery eye a `Static Typist`'s maniacal malarkey. Static typing system works the same way as **BDSM**: the more constraints, the more pleasure. Once having tasted the relish of bondage, a bottom will avariciously demand more complex tricks and more powerful constraints from typing system. That urge is so strong that the bottom loses incentives to lumber out of _the fifty shades of types_.
