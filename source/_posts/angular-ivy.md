title: A quick intro to Angular Ivy
date: 2018-02-25
tags: Angular
---

Angular recently announced a new render engine called [Ivy](https://github.com/angular/angular/issues/21706).
Ivy is an amazing present from Angular team! It produces hello-world app in mere [3.2KB](https://twitter.com/IgorMinar/status/961484134425088001),on a par with minimal framework like preact. Yet little documentation, if any, exists to explain how Ivy works.

This blog post strives to fill this gap, by translating an amazing answer from [zhihu](https://www.zhihu.com/question/266923267/answer/316279829), a superior Q&A website to Quora. I hope this post can communicate Angular's innovation to broader front-end world by breaking language barrier.

>> TL;DR: Ivy is an extremely building-tool friendly, web-first render engine that paves Angular's way to web components.

-------------------

Orignal post:

How to comment on the fact that Angular's new Ivy render engine produces 3.2KB compiled JavaScript? _(traslator note: `how to comment` is a jargon on zhihu)_

Let's first answer several frequent questions:

1. 3.2KB is the result after Minify + Gzip(a convention to compare framework size), which is the exact payload size we send to browser. (if we use more advanced compression like brotli, the size will be smaller on modern browsers).

2. Ivy renderer won't be default in Angular v6. You need to manually enable it by switching on certain compiler option. It might be default in v7 if no further issues are found.

3. Code completion isn't proportional to feature completion. 50% code completed doesn't imply 50% feature completed. For now Ivy has seemingly decent code completion rate, but it still has a long way before ready for production. It can't even complete a simple app for [benchmark](https://github.com/krausest/js-framework-benchmark).

4. Compared with previous Angular's compiled code, Ivy's result is almost like hand-written. And actually you can write it by hand! (Though in practice you won't.)

5. `NgModule` is challenged, **again**.

6. Ivy won't promise you a plunge in code weight. Your bundle will still be bulky if a lot of CSS is inlined in your components.


------

Since the question is to _comment on_ 3.2KB compiled file, I will focus on payload size optimization, thus, answering the question _"What optimization has Ivy Renderer done?"._


It can never be overstated that the extreme size isn't achieved by the ADVANCED mode of Closure Compiler. **Rollup** can achieve the same level optimization (within less than 1KB difference). Therefore Ivy is arguably the true **building-tool friendly** renderer, not Closure-Compiler-only renderer.


Disclaimer: all the below is based on current(Feb, 2018) implementation, published roadmap. If following version changes leads to the following content outdated or incorrect, please pardon our no updating.


A. Removing dependence on `platform-browser`

As a platform independent framework, can we run application without depending on platform specific code? The answer is NO, of course. Ivy just **inlines DOM Rendere to its core.** If you only need run your application on browser (taking no account of WebWorker), you can run Angular without including platform specific code. For example, the [code for binding text](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/instructions.ts#L842-L845) is:

```ts
value !== NO_CHANGE &&
  ((renderer as ProceduralRenderer3).setValue ?
    (renderer as ProceduralRenderer3).setValue(existingNode.native, stringify(value)) :
    existingNode.native.textContent = stringify(value));
```

The fallback code is obvious: if no `renderer` exists(translator note: the condition is better explained as `renderer` isn't `ProceduralRenderer3`), Ivy will directly modify dom element's `textContent`.

Currently no Ivy code relies on Platform, thus common problems arise:

* No support for [Event Plugin](https://angular.io/api/platform-browser/EVENT_MANAGER_PLUGINS). e.g. syntactic sugar for keyboard event(like `keydown.enter`) and events from Hammer, which are all provided by platform-browser.

* No `Sanitizer`. Though Angular isn't string template by itself and is naturally immune to XSS, it still cannot survive abhorrent abuse of `innerHTML`. `Sanitizer`, again currently bestowed by platform-browser, comes to rescue by filtering user content. Without platform hardly can we achieve the same level security.

* No support for Component Style; No View Encapsulation. They are all implemented by different `Renderer`s in platform-browser package. Only inline style is available for component styling.(Another better way is independent CSS and dynamic class).

So the current Ivy renderer demoed in hello-world app isn't a full featured Angular for some users.

B. No NgFactory file anymore

Under the new Ivy mode, compiled template will be stored in the static fields in class, instead of generating new wrapper class(NgFactory). For example:


`Component` -> `ngComponentDef`

```ts
@Component({ /*...*/ })
class MyComponent { }

// ->

class MyComponent {
  static ngComponentDef = defineComponent({ /*...*/ })
}
```

`Directive` -> `ngDirectiveDef`

```ts
@Directive({ /*...*/ })
class MyDirective { }

// ->

class MyDirective {
  static ngDirectiveDef = defineDirective({ /*...*/ })
}
```

`NgModule` -> `ngInjectorDef`

```ts
@NgModule({ /*...*/ })
class MyModule { }

// ->

class MyModule {
  static ngInjectorDef = defineInjector({ /*...*/ })
}
```

~~WAT? How can Your Majesty NgModule compiled to one sole Injector?(~~

Due to the colocation of compiled template and class, the new compilation can fully enjoy building tool's normal optimization. (Tree-Shaking for the most part), waiving most special process in build-optimizer.


**In fact, this improvment is more significant for building than for size optimization.** The new style compiler enables single file compilation, one ts file to one js file. We no longer need to worry about mapping from source file to ngfactory.

Further more, this unifies library consumption in JIT and AOT. Once the compilation is stabilized, all libraries can compile code before publication, rather than at end uses' bundling phase. This can lead to faster building.


C. Greatly simplify bootstrap code

All Angular applications need to configure bootstrap component(actually it isn't mandatory, but alternatives are more complex), and initialize one `NgModule[Factory]`. Something like:

```ts
import { NgModule } from '@angular/core'
import { platformBrowser } from '@angular/platform-browser'
import { AppComponent } from './app.component'

@NgModule({
  bootstrap: [AppComponent]
})
class AppModule { }

platformBrowserDynamic().bootstrapModule(AppModule)
```

New bootstrapping code is based on component:

```ts
// not available yet
import { renderComponent } from '@angular/core'
import { AppComponent } from './app.component'

renderComponent(AppComponent)
```

~~Hey, why we still need AppModule?~~

Ivy is based on NgModule-less bootstrapping for now. Though not completed yet, compilers can make bootstrapping accept NgModule (if it still exists) to provide Injector. In other word, `renderComponent` can accept [an optional configuration argument](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/component.ts#L37).

By the way, this is also friendlier to bootstrapping multiple Angular instances in one page.


D. Redesigned DevMode configuration and checking

In the previous Angular, `DevMode` is disabled by function call dynamically.


```ts
import { enableProdMode } from '@angular/core'

enableProdMode()

platformFoo().bootstrapBar(baz)
```

In other word, whether DevMode is on is totally Angular's [internal state](https://github.com/angular/angular/blob/5.2.4/packages/core/src/application_ref.ts#L64-L67). Thus all debug related code is dynamically used and cannot be excluded in compile time. (Even closure compiler is ineffective).

But in Ivy DevMode is purely compile time configuration, all debugging code will be executed after checking the global variable `ngDevMode`.  [For example](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/instructions.ts#L231):

```ts
ngDevMode && assertEqual((state as LView).node, null, 'lView.node');
```

So building tool can replace the corresponding variable (e.g Webpack's DefinePlugin), and corresponding debug code can be detected as dead code, and then optimizer(e.g UglifyJS) can remove it.

E. Feature named as Feature

Ivy mode has added a new feature called Feature. Or, Ivy introduces a new concept called Feature. Simply put, Feature is a pre-processor for `DirectiveDef`, which can be thought as a "Decorator pattern" targeted for `DirectiveDef`. (Translator note: More simply, you can provide a custom function to Angular, and Angular will apply it against `ComponentDef` metadata. So you can extend `ComponentDef`'s `Feature` in your own function. [Source](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/definition.ts#L58-L59)).

Let's take an example in Ivy. `OnChanges` isn't implemented by `Renderer` but a [predefined `Feature`](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/definition.ts#L71-L122). The Feature will use `defineProperty` to listen on the property decorated by `@Input`, and automatically stores `previousValue` in the instance to generate `SimpleChanges` for change detection, and finally trigger `OnChanges` by intercepting `OnInit` and `DoCheck` without Angular core's help.

This means `OnChanges`' code is also tree-shakable! If no component ever declares the lifecycle `ngOnChanges`, no `DirectiveDef` will import `NgOnChangesFeature`. So optimizer can reasonably eliminate dead code.


~~WAT? How about the dirty check promise? From now on we have exception in Angular's change detection?!~~

So it is notable that `OnChanges` in Ivy is not a [real lifecycle](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/definition.ts#L50-L56) any more(Translator note: it is not listed in the `ComponentDef`).
Put differently, users can extend lifecycle themselves, do as they please.


But the direct aim of Feature concept is for the incoming [LifeCycle as Observables](https://github.com/angular/angular/issues/10185). Users don't need to declare methods(ngOnInt) in class, but rather use the lifecycle observables directly. Indeed. by intercepting `DirectiveDef`, users and third-party library authors can modify lifecycle hooks in runtime. We can even use decorators to declare lifecycles.


In summary, `Feature` can be thought as a variant of `Higher Order Component` (compared with class factory): it modifies component type itself based on runtime requirement, instead of changing component's internal state according to component logic.

F. new Injectable API
