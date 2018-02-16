title: Deep dive into Vue2.5 Typing -- A tour of advanced typing feature
date: 2017-10-12
tags: TypeScript

---

Vue 2.5 improves TypeScript definition! Before that, TS users will have to use class component API to get proper typing, but now canonical API is both precise and concise with few compromises!

For ordinary users, Vue's [official blog](https://medium.com/the-vue-point/upcoming-typescript-changes-in-vue-2-5-e9bd7e2ecf08) and [updated documentation](https://vuejs.org/v2/guide/typescript.html) will guide you to upgrade or create projects.
But curious audience might wonder how the improvement is done and why TS support isn't integrated in Vue2.0 at first place.

This blog post will deep dive into the technical details of Vue2.5 typing, which seems [daunting](https://github.com/vuejs/vue/blob/dev/types/options.d.ts#L54) at first glance. Don't worry! We will show how TypeScript's [advanced types](https://www.typescriptlang.org/docs/handbook/advanced-types.html) can be used in a popular framework.

> Note: Reader's familiarity with Vue and TypeScript is assumed in this post. If you are new to these two, checkout their [official](https://vuejs.org/) [website](https://www.typescriptlang.org/docs/home.html)!

TL;DR;
----

Vue2.5 exploits [ThisType](https://github.com/Microsoft/TypeScript/pull/14141), [mapped type](https://blog.mariusschulz.com/2017/01/20/typescript-2-1-mapped-types), [generic defaults](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-3.html#generic-parameter-defaults) and a clever trick to cover most APIs.

We will also list some limitations in current typing schema.


`this` is Vue
--------

Let's examine a basic Vue usage. We pass an object literal as component option to Vue constructor.

```js
new Vue({
  methods: {
    greet() {
      this.$el // `this` is Vue
      console.log('Hello World!')
    }
  }
})
```

`this` keyword is bound to Vue instance in component option. Prior to Vue2.5, we declare `this` as a plain Vue type. Here is a simplified `ComponentOption` type.

```ts
interface ComponentOption {
  methods: { [key: string]: (this: Vue) => any }
  // other fields ...
}
```

However, we cannot access our custom methods/data via the declaration above since `this` is nothing but Vue. The typing doesn't capture the fact that the VM injected into methods is instantiated with our custom methods/data/props.

A new type parameter `V` can allow users to specify their custom properties. So a better solution will be:

```ts
interface ComponentOption<V extends Vue> {
  methods: { [key: string]: (this: V) => void }
}
```

And users can use it like this.

```ts
declare function newVue<V extends Vue>(option: ComponentOption<V>): V

interface MyComponent extends Vue {
  greet(str: string): void
  hello(): void
}
newVue<MyComponent>({
  methods: {
    greet(str) {
      console.log(str)
    },
    hello() {
      this.greet('hello world')
    }
  }
})
```

It works, but also requires one interface declaration and one explicit type annotation.
Can compiler be smarter and infer this for us?

`ThisType<Vue>`
-----------

We can strongly type `this` by a special marker interface `ThisType`. It is introduced in TypeScript 2.3, which is the very reason why we didn't have strong type until Vue2.5.

The original [pull request](https://github.com/Microsoft/TypeScript/pull/14141) has a detailed introduction and example for `ThisType`.

The most important rule is quoted here.


> (If) the containing object literal has a contextual type that includes a `ThisType<T>`, `this` has type T

What does this mean? Let's break this rule down to several pieces.

`object literal` means the component option in Vue's case; `contextual type` means the component option is passed to a function as argument and the component option is typed via function declaration, without caller's annotation; and finally `ThisType<T>` needs to be used in the function parameter declaration. The type parameter `T` refers to the type of `this` in the component option. In simple terms, this rule says we can change `this` keyword's type according to the component option passed to `new Vue` or so.

Combining these together, we can write a simple declaration that understands our Vue component option.


> Note, you will need `noImplicitThis` compiler flag to enable this new type checking.


```ts
interface ComponentOption<Method> {
  methods: Method
}

declare function newVue<Method>(
  option: ComponentOption<Method> & ThisType<Method & Vue>
): Vue&Method

// Method is inferred as
// { greet(str): void, hello(): void }
newVue({
  methods: {
    greet(str) {
      console.log(str)
    },
    hello() {
      // this is typed as Method & Vue
      this.greet('hello world')   // custom methods works!
      this.$el // vue property also works!
    }
  }
})
```

This code needs some explanation. First we define an `ComponentOption` and it takes a type parameter `Method`, which acts as a "stub" for compiler to infer custom properties on `this`.
Then in the function we declare a type parameter `Method` again and pass it to `ComponentOption` and `ThisType`.
Finally, `ThisType<Method & Vue>` means the type of `this` inside option will be an intersection of `Vue` and `Method`.

When we call `newVue`, compiler will first infer `Method` from `ComponentOption` object we pass to the function. Then the `Method` will flow into `this` keyword, resulting a type that has both Vue property and our own methods.




Mapping Computed
----------

Typing `methods` alone is so far so good. However fields like `computed` have a different story.
The object in `methods` field has the same shape as part of `this` type. Say, `methods` has a `hello` function property and `this` also has a function property with the same name (in algebraic terms, [endomorphism](https://en.wikipedia.org/wiki/Homomorphism#Endomorphism)). But a property in `computed` is a function that returns a value and `this` has a namesake property with the same value type. For example.

```ts
newVue({
  computed: {
    myname: () => 'world' // a function returns string
  },
  methods: {
    greet() {
      console.log(this.myname)
      // myname is a string, not a function returning string
    }
  }
})
```

How can we get a new type from `computed` definition object?

Here comes the mapped type, a new kind of object type that maps a type representing property names over a property declaration template. In other words, we can create computed type in Vue instance based on that in component option. (algebraically, [homomorphism](https://en.wikipedia.org/wiki/Homomorphism#Definition))

> In a mapped type, the new type transforms each property in the old type in the same way.

The [official documentation](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types) is crystal clear. Let's see how we integrate this awesomeness into Vue.

```ts
// we map a plain type to a type of which the property is a function
// e.g. { myname: string } will be mapped to { myname: () => string }
// note this process can also be reversed during type inference
type Accessors<T> = { [K in keyof T]: () => T[K] }

interface ComponentOption<Method, Computed> {
  methods: Method
  computed: Accessors<Computed>
}

type ThisTypedOption<Method, Computed> =
  ComponentOption<Method, Computed> & ThisType<Method & Computed & Vue>

declare function newVue<Method, Computed>(
  option: ThisTypedOption<Method, Computed>
): Method & Computed & Vue
```

`Accessors<T>` will map the type `T` to a new type with same property names. But property value type is a function returning the type in the original `T`. This process is reversed during type inference.  When we pass `computed` field as `{myname: () => string}` to `newVue` function, compiler will try to map the type to `Accessors<T>`, which results in `Computed` being `{myname: string}`.

And `Computed` is mixed into `this`, so we can access `myname` as `string` from `this`.


We skipped here [computed setter style](https://vuejs.org/v2/guide/computed.html#Computed-Setter) declaration for a more lucid demonstration. Supporting setter in `computed` is similar.

Prop Types Trick
----
`props` has a subtle difference from `computed`: we define a `prop` by giving a constructor of that value type.

```ts
type PropDef<T> = { new(...args: any[]): T }
type Props<T> = { [K in keyof T]: PropDef<T[K]> }
interface ComponentOption<Prop> {
  props: Props<Prop>
}
type ThisTypedOption<Prop> =
  ComponentOption<Prop> & ThisType<Prop & Vue>

declare function newVue<Prop>(option: ThisTypedOption<Prop>): Prop & Vue

class User {}
newVue({
  props: {
    user: User,
    name: String
  }
})
```

One would naturally expect `newVue` will infer `Prop` as `{ user: User, name: string }`. Sadly, it is not.

The problem lies in `PropDef`, which uses constructor type `new(): T`. Custom constructor is fine. For example `User`'s  constructor returns `User`. But primitive value's constructor doesn't work because `String` has the signature `new(): String`.

Alas! The return value is `String`, rather than `string`. Their difference is listed in the [first rule](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html) of _TypeScript Do's and Don'ts_. A `string` type is what we use and `String` refers to non-primitive boxed objects that are almost never used.


We can use another signature to type primitive constructor and [union](https://www.typescriptlang.org/docs/handbook/advanced-types.html#union-types) custom constructor together. Note every primitive constructor has a call signature, that is, `String(value)` will return a primitive `string` value rather than a wrapper object.

```ts
type PropDef<T> = { (): T } | { new(...args: any[]): T }
```

It should work, shouldn't it? Sadly again, NO.

```ts
declare function propTest<T>(t: PropDef<T>): T
propTest(String) // return String, not string!
```

Because `String` satisfy both call and constructor signature in `PropDef`, compiler will prefer returning `String`.

How can we nudge compiler to prefer primitive type? [Here](https://github.com/HerringtonDarkholme/av-ts/issues/62) is an undocumented trick.
The main idea is to exploit [type inference priority](https://github.com/Microsoft/TypeScript/blob/8e47c18636da814117071a2640ccf87c5f16fcfd/src/compiler/types.ts#L3563-L3583). If a type parameter is single naked, that is, not in [intersection type](https://github.com/Microsoft/TypeScript/pull/3622#issuecomment-116888221) nor in [union type](https://github.com/Microsoft/TypeScript/pull/1035), compiler will prefer to infer from that single naked position over intersection/union position. So we can add an intersection to constructor signature and then compiler will first infer call signature. Exactly what we want! To make the signature more self explanatory, we can use the [`object` type](https://github.com/Microsoft/TypeScript/pull/12501) to flag constructor type should not return primitive type.

```ts
type PropDef<T> = { (): T } | { new(...args: any[]): T & object }
declare function propTest<T>(t: PropDef<T>): T
propTest(String) // return string, yay!
```

Now we can happily infer `props` without manual annotation!

Compatibility
----

For better inference, our new type has many more type parameters than original `ComponentOption<V>` which only has one parameter. Nevertheless, it will be a catastrophic breaking change if we ship the new type without proper fallback. [Generic defaults](https://blog.mariusschulz.com/2017/06/02/typescript-2-3-generic-parameter-defaults) introduced in TS2.3 gives us a chance to bring about a more smooth upgrade.

```ts
interface ComponentOption<V extends Vue, Method=any, Data=any, Prop=any, Computed=any> {
  // ....
}

interface MyVue extends Vue {
 // ...
}

// users can use ComponentOption without changing their code
// parameter with default can be skipped
var option: ComponentOption<MyVue> = {
  // ...
}
```

Happy ending!

Limitation
-------
The "No silver bullet" rule also applies to typing. **The more advanced types we use, the more complex error messages will be generated.** Hope this blog post will help you to understand the new typing better and help you to debug your own application.

There are also some type system limitations in Vue typing. Let's see some examples.

* functions in `computed` need return type annotation

Return type annotation is required if `computed` method uses `this`. It turns out that using mapped type and ThisType at the same time without explicit annotation will cause cyclic inference error in current compiler.

```ts
new Vue({
  computed: {
    foo() {
      return 123
    },
    bar(): number { // required
      return this.foo
    }
  }
})
```

TypeScript has already opened an [issue](https://github.com/Microsoft/TypeScript/issues/18805) tracking this.

* Prop types' union declaration requires manual type cast

Vue accepts an array of constructors in prop's definition as union type. However, `PropDef` cannot unify primitive constructors and custom constructors which have two heterogeneous signatures.

```ts
Vue.component('union-prop', {
  props: {
    primitive: [String, Number], // both primitive, ok
    custom: [Cat, User],         // both custom, ok
    mixed: [User, Number] as {new(): User | Number}[] // requires annotation
  }
})
```

In general, you should avoid mixing primitive type and object type.

Final words
---

TypeScript has been constantly evolving since its birth. And finally its expressiveness enable us to type Vue's cannonical API!

Thank you, TypeScript team, for bring us these awesome features!
Thank you, Vue team, for embracing new advance in type system!
