title: TypeSafe DI in TypeScript
date: 2016-02-01 19:26:14
tags: TypeScript
---

{%img /images/miku805.jpg%}

> Yet Another Type Level Arithmetics

DI is getting more popularity in JavaScript. Though, those solution is far way beyond JSR-330 compatible libraries in terms of type safety and performance.

This snippet strives to dig more type-safety from TS' type system. However, it can hardly achieve equivalent type safety like Java's counterpart, e.g., Dagger, Guice.


Solution
------
[This DI](https://gist.github.com/HerringtonDarkholme/f11152e6e29145040fa4) snippet can, ideally, ensure every binding is resolved in compile time, which is a hard task for other DI solution. The main idea is an `Injector` can statically know its current binding, and judge whether dependencies of a new added binding can be already resolved by itself. Since dependency graph is a DAG, there exists a topological sorting order that every binding's dependency can be resolved solely by those of preceding bindings . So once the injector is created and bindings are attached to it, we can assert that the dependencies can be resolved.

Say, there is a minimal example to illustrate this:

```TypeScript
@Inject
class Clerk {}
@Inject
class Shop {constructor(c: Clerk)}
var inj = Injector.create()
  .bind(Shop).toClass(Shop) // compile error here, injector cannot resolve Clerk

var inj = Injector.create()
  .bind(Clerk).toClass(Clerk)
  .bind(Shop).toClass(Shop) // compiles. Clerk resolved before Shop
```

To implement this, Injector has a shadow type `Base` that indicates resolved bindings. When new binding is added to injector, compiler will verify the new coming constructor/function will only depend on classes the injector has already resolved. Concretely, every argument in newly added constructor must be a subtype of `Base`.

```TypeScript
type Ctor = new (...args: Base[]): T

injector<Base>
  .bind(NewClass)
  .toClass(NewClass as Ctor)
/*
Make sure here `toClass` is defined like
type toClass = (Ctor) => Injector<Base | T>
the union type indicates resolved type, and `T extends Base|T` holds valid
*/
```

`Base` is a large union type storing all binding types, so every resolved type is a subtype of `Base`. And `bind` will return a `binder` that has `toClass / toFactory` method which further returns an injector whose resolved binding is a union of the previous binding type and the newly added binding type. Hence, after `bind ... toClass`, the injector has a new class appended to its resolved type list.

The implementation and test can be found at [Github Gist](https://gist.github.com/HerringtonDarkholme/f11152e6e29145040fa4).


Problem
---

But TS' type system does not allow a full-fledged DI in this way.

1. First, runtime types are erased. One must annotate dependency for function in `toFactory` method. `toClass` is better because TS supports `emitDecoratorMetadata`. (maybe resolved in TS2.0). TS' specific metadata implementation is also problematic. For cyclic dependent classes, at least one class' annotation is `undefined`(ES3/5), or the script is crashed before it can run (ES6). Because metadata is attached to class declaration, in cyclic case there must be one class is used before it's declared.

2. TypeScript has a double-edged sutructural type system. To fully exploit DI's type check, user has to add a `private` `brand` field to every injectable class. This is not a good UI, though.

3. But even metadata is not enough. Runtime type data is first-order (in type-system's view), that is, every type is represented by its constructor, no generic information is emitted. To work around this, token is introduced.

Token alleviates runtime type-system's weakness, and enables binding multiple implementations to one single type. It also introduces more problem this DI wants to resolve in the first place. To work around point 1, we attached runtime types to constructor. Binding token will make type system think a type has resolved, but a following binding may not resolve it in runtime because it depends on constructor to find resolution.

```TypeScript
injector
  .bind(clerkToken).toClass(Clerk)
  .bind(Shop).toClass(Shop) // compiles. but runtime error
// toClass will analyze Shop's signature and extract the Clerk constructor
// it can be found in type-level because Token<Clerk> enable injector to resolve Clerk, but at runtime injector can only resolve clerkToken, not Clerk
```

Also, tokens with same types cannot avoid this.

The workaround is, well, abusing string literal type. So every token is different at type-level. This requires users to type more types, and casts string literal from string type to string literal type. (TS's generic inference does not have something like `T extends StringLiteral` so that `T` is inferred as string literal type)

Also, the `toClass` and `toFactory` signature should differentiate what can be resolved by constructor and by token.  This is technically possible, just override these signature to support distinguishing between token and constructor. But the number of resulting overriding is exponential to the number of argument. 2 ^ n, where n is the number of arguments.

Conclusion
----

To fully support type-safe DI and higher performance, a compiler extension or a code generator is needed. Java's DI relies on annotations and code generation.

Maybe Babel can do this right now. But TypeScript still needs a long way to go for a customizable emitter.

Source
---

```TypeScript
import 'reflect-metadata'

type _ = {}

type ClassN<N, T> = { new (...a: N[]): T }
type FN8<A, B, C, D, E, F, G, H, R> = (a?: A, b?: B, c?: C, d?: D, e?: E, f?: F, g?: G, h?: H) => R
type CLS8<A, B, C, D, E, F, G, H, R> = { new (a?: A, b?: B, c?: C, d?: D, e?: E, f?: F, g?: G, h?: H): R}

class Token<T> {
  private _tokenBrand
  constructor(public name: string) {}
}
type Bindable<T> = Token<T> | ClassN<_, T>

const PARAMTYPE = 'design:paramtypes'

function getSignature<T>(cls: ClassN<T, _>): Array<Bindable<T>> { return Reflect.getOwnMetadata(PARAMTYPE, cls).slice() || [] }

const enum BindType { CLASS, VALUE, FACTORY }
class Binder<T, Base> {
  dependencies: Bindable<Base>[] = []
  cacheable: boolean = false
  private _type: BindType
  private _cls: ClassN<any, T> = null
  private _val: T
  private _fn: Function

  constructor(private _injector: Injector<Base>) {}

  private _releaseInjector() {
    let injector = this._injector
    this._injector = null
    return injector
  }

  toClass<A extends Base, B extends Base, C extends Base, D extends Base, E extends Base, F extends Base, G extends Base, H extends Base>(fn: CLS8<A, B, C,  D, E, F, G, H, T>, a?: Bindable<A>, b?: Bindable<B>, c?: Bindable<C>, d?: Bindable<D>, e?: Bindable<E>, f?: Bindable<F>, g?: Bindable<G>, h?: Bindable<H>): Injector<Base | T>
  toClass(cls: ClassN<Base, T>, ...deps: Bindable<Base>[]): Injector<Base | T> {
    this._type = BindType.CLASS
    this._cls = cls
    this.dependencies = getSignature(cls)
    deps.forEach((d, i) => {
      if (d) this.dependencies[i] = d
    })
    return this._releaseInjector()
  }

  toValue(val: T): Injector<Base | T> {
    this._type = BindType.VALUE
    this._val = val
    this.cacheable = true
    return this._releaseInjector()
  }

  toFactory<A extends Base, B extends Base, C extends Base, D extends Base, E extends Base, F extends Base, G extends Base, H extends Base>(fn: FN8<A, B, C,  D, E, F, G, H, T>, a?: Bindable<A>, b?: Bindable<B>, c?: Bindable<C>, d?: Bindable<D>, e?: Bindable<E>, f?: Bindable<F>, g?: Bindable<G>, h?: Bindable<H>): Injector<Base | T>
  toFactory(fn: (...a: Base[]) => T, ...deps: Bindable<Base>[]): Injector<Base | T> {
    this._type = BindType.FACTORY
    this._fn = fn
    this.dependencies = deps
    return this._releaseInjector()
  }

  resolve(deps: any[]): Promise<T> {
    switch (this._type) {
      case BindType.CLASS:
        let cls = this._cls
        return Promise.resolve(new cls(...deps))
      case BindType.VALUE:
        return Promise.resolve(this._val)
      case BindType.FACTORY:
        return Promise.resolve(this._fn(...deps))
    }
  }
}

export class Injector<Base> {
  private _resolving = new Set<Bindable<_>>()
  private _typeBinderMap = new Map<Bindable<_>, Binder<_, Base>>()
  private _cache = new Map<Bindable<_>, Promise<any>>()

  get<T extends Base>(cls: Bindable<T>): Promise<T> {
    let binder = this._typeBinderMap.get(cls)
    if (!binder) throw new NoBinding(cls['name'])

    let cache = binder.cacheable ? this._cache.get(cls) : null
    if (cache) {
      return cache
    }

    if (this._resolving.has(cls)) throw new CyclicDependency(cls['name'])

    this._resolving.add(cls)
    let depPromise = binder.dependencies.map(dep => this.get(dep))
    this._resolving.delete(cls)

    let ret = Promise.all(depPromise).then(args => binder.resolve(args))
    if (binder.cacheable) this._cache.set(cls, ret)
    return ret
  }

  bind<T>(cls: Bindable<T>): Binder<T, Base> {
    let binder = new Binder<T, Base>(this)
    this._typeBinderMap.set(cls, binder)
    return binder
  }

  static create(): Injector<Injector<_>> {
    let inj = new Injector<Injector<_>>()
    inj.bind(Injector).toValue(inj)
    return inj
  }
}

```
