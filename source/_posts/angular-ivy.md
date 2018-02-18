title: A quick intro to Angular Ivy
date: 2018-02-19
tags: Angular
---

Angular recently announced a new render engine called [Ivy](https://github.com/angular/angular/issues/21706).
Ivy is an amazing present from Angular team! It produces hello-world app in mere [3.2KB](https://twitter.com/IgorMinar/status/961484134425088001), on a par with minimal framework like preact. Unfortunately little documentation, if any, exists to explain how Ivy works.

This blog post strives to fill this gap, by translating an amazing answer from [zhihu](https://www.zhihu.com/question/266923267/answer/316279829), a superior Q&A website to Quora. I hope this post can communicate Angular's innovation to broader front-end world by breaking language barrier.

> TL;DR: Ivy is an extremely building-tool friendly, web-first render engine that paves Angular's way to web components.

Opinion is not mine. Words in parentheses are from the original author unless explicitly annotated as from translator. Translator notes are added for audience with no Angular background.

Orignal post:
---

How to comment on the fact that Angular's new Ivy render engine produces 3.2KB compiled JavaScript? _(traslator note: `how to comment` is a jargon on zhihu, meaning `what's your opinion`)_

Let's first answer several frequent questions:

1. 3.2KB is the result after minification + gzip (a convention to compare framework size), which is the exact payload size we send to browser. (if we use more advanced compression like brotli, the size will be smaller on modern browsers).

2. Ivy renderer won't be default in Angular v6. You need to manually enable it by switching on certain compiler option. It might be default in v7 if no further issues are found.

3. Code completion isn't proportional to feature completion. 50% code completed doesn't imply 50% feature completed. For now Ivy has seemingly decent code completion rate, but it still has a long way before ready for production. It can't even complete a simple app for [benchmark](https://github.com/krausest/js-framework-benchmark).

4. Compared with previous Angular's compiled code, Ivy's result is almost like hand-written. And actually you can write it by hand! (Though in practice you won't.)

5. `NgModule` is challenged, **again**.

6. Ivy won't promise you a plunge in code weight. Your bundle will still be bulky if a lot of CSS is inlined in your components.


------

Since the question is to _comment on_ 3.2KB compiled file, I will focus on payload size optimization, thus, answering the question _"What optimization has Ivy Renderer done?"._


It can never be overstated that the extreme size isn't achieved by the ADVANCED mode of Closure Compiler. **Rollup** can achieve the same level optimization (within less than 1KB difference). Therefore Ivy is arguably the true **building-tool friendly** renderer, not Closure-Compiler-only renderer.


Disclaimer: all the below is based on current (Feb, 2018) implementation, published roadmap. If following version changes leads to the following content outdated or incorrect, please pardon our no updating.


A. Removing dependence on `platform-browser`
---

As a platform independent framework, can we run application without platform specific code? The answer is NO, of course. Ivy just **inlines DOM Rendere to its core.** If you only need run your application on browser (taking no account of WebWorker), you can run Angular without including platform specific code. For example, the [code for binding text](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/instructions.ts#L842-L845) is:

```ts
value !== NO_CHANGE &&
  ((renderer as ProceduralRenderer3).setValue ?
    (renderer as ProceduralRenderer3).setValue(existingNode.native, stringify(value)) :
    existingNode.native.textContent = stringify(value));
```

Fallback code is obvious: if no `renderer` exists _(translator note: the condition is better explained as `renderer` isn't a `ProceduralRenderer3`)_, Ivy will directly modify dom element's `textContent`.

Currently no Ivy code relies on `platform`, thus common problems arise:

* No support for [Event Plugin](https://angular.io/api/platform-browser/EVENT_MANAGER_PLUGINS). e.g. syntactic sugar for keyboard event (like `keydown.enter`) and events from Hammer, which are all provided by platform-browser.

* No `Sanitizer`. Though Angular isn't string template by itself and is naturally immune to XSS, it still cannot survive abhorrent abuse of `innerHTML`. `Sanitizer`, again currently bestowed by platform-browser, comes to rescue by filtering user content. Without platform hardly can we achieve the same level security.

* No support for Component Style; No View Encapsulation. They are all implemented by different `Renderer`s in platform-browser package. Only inline style is available for component styling (Another better way is independent CSS and dynamic class).

So the current Ivy renderer demoed in hello-world app isn't a full featured Angular for some users.

B. No NgFactory file anymore
---

Under the new Ivy mode, compiled template will be stored in the static fields in class, instead of generating new wrapper class (NgFactory). For example:


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

~~WAT? How can Your Majesty NgModule compiled to one single Injector?(~~

Due to the colocation of compiled template and class, the new compilation can fully enjoy building tool's normal optimization. (Tree-Shaking for the most part), waiving most special process in build-optimizer.


**In fact, this improvment is more significant for building than for size optimization.** The new style compiler enables single file compilation, one ts file to one js file. We no longer need to worry about mapping from source file to ngfactory.

Further more, this unifies library consumption in JIT and AOT. Once the compilation is stabilized, all libraries can compile code before publication, rather than at end uses' bundling phase. This can lead to faster building.


C. Greatly simplify bootstrap code
---

All Angular applications need to configure bootstrap component (actually it isn't mandatory, but alternatives are more complex), and initialize one `NgModule[Factory]`. Something like:

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
---

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

So building tool can replace the corresponding variable (e.g Webpack's DefinePlugin), and corresponding debug code can be detected as dead code, and then optimizer (e.g UglifyJS) can remove it.

E. Feature named as Feature
---

Ivy mode has added a new feature called Feature. Or, Ivy introduces a new concept called Feature. Simply put, Feature is a pre-processor for `DirectiveDef`, which can be thought as a "Decorator pattern" tailored for `DirectiveDef`. _(Translator note: More simply, you can provide a custom function to Angular, and Angular will apply it against `ComponentDef` metadata. So you can extend `ComponentDef`'s `Feature` in your own function. [Source](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/definition.ts#L58-L59))_.

Let's take an example in Ivy. `OnChanges` isn't implemented by `Renderer` but a [predefined `Feature`](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/definition.ts#L71-L122). The Feature uses `defineProperty` to listen on the property decorated by `@Input`, and automatically stores `previousValue` in the instance to generate `SimpleChanges` for change detection, and finally triggers `OnChanges` by intercepting `OnInit` and `DoCheck` without Angular core's help.

This means `OnChanges`' code is also tree-shakable! If no component ever declares the lifecycle `ngOnChanges`, no `DirectiveDef` will import `NgOnChangesFeature`. So optimizer can reasonably eliminate dead code.


~~WAT? How about the dirty check promise? From now on we have exception in Angular's change detection?!~~

So it is notable that `OnChanges` in Ivy is not a [real lifecycle](https://github.com/angular/angular/blob/92d7060cb086632d2188c8da37ff72838d1abeef/packages/core/src/render3/definition.ts#L50-L56) any more _(Translator note: it is not listed in the `ComponentDef`)_.
Put differently, users can extend lifecycle themselves, do as they please.


But the direct aim of Feature concept is for the incoming [LifeCycle as Observables](https://github.com/angular/angular/issues/10185). Users don't need to declare methods (ngOnInt) in class, but rather use the lifecycle observables directly. Indeed. by intercepting `DirectiveDef`, users and third-party library authors can modify lifecycle hooks in runtime. We can even use decorators to declare lifecycles.


In summary, `Feature` can be thought as a variant of `Higher Order Component` (compared with class factory): it modifies component type itself based on runtime requirement, instead of changing component's internal state according to component logic.

F. New Injectable API
---

Actually this isn't related to the hello-world demo, nor related to even Ivy (since it can be used in non Ivy mode). However, the new Injectable API has a huge impact on Angular's size.

Besides the `XXXDef` properties listed above, we have another one called `ngInjectableDef`:

```ts
@Injectable()
class MyService { }

// ->

class MyService {
  static ngInjectableDef = defineInjectable({ /*...*/ })
}
```

In classical sense (if anyone ever cared), we can easily find **dependency injection and dead code elimination are contradictory in priciple**.

* DI's nature is side effect: `Provider` _changes_ execution context via configuration, and `Consumer` retrieves content _from context_.

* DCE's assumption is side-effect-free: `Consumer` should import `Provider` directly, and if `Consumer` doesn't exist in application, `Provider` can be removed at all.


Apparently, all DI based code cannot be effectively DCE optimized.

In current Angular pattern, we will configure `Provider` in `NgModule`:

```ts
// lib.service.ts
@Injectable()
class LibService { }

// lib.module.ts
import { LibService } from './lib.service'

@NgModule({
  providers: [LibService],
})
class LibModule {}

// app.component.ts
import { LibService } from './lib.service'

@Component({ /*...*/ })
class AppComponent {
  constructor(libService: LibService) {}
}

// app.module.ts
import { LibModule } from './lib.module'
import { AppComponent } from './app.component'

@NgModule({
  declarations: [AppComponent],
  imports: [LibModule],
})
class AppModule {}
```

The dependence flow is: _(translator note: in the below items parentheses after is added by translator for Angular newcomer)_

* Library Module imports Library Service _(for exposing interface and providing implementation)_
* Application Component imports Library Service _(for consuming interface)_
* Application Module imports  Application Component
* Application Module imports Library Module _(for configuring which implementation to inject)_

Even if service isn't used in application component, service cannot be removed. This is because we introduce additional dependence during DI configuration. _(translator note: consider app component doesn't import libService. It is desirable libService is excluded from final build. But we cannot eliminate it because libModule imports it, and libModule is further imported by application module)_


To solve this problem, Angular allows to invert the dependence of configuration. The new API is like:

```ts
// lib.module.ts
@NgModule({ /*...*/ })
class LibModule {}

// lib.service.ts
import { LibModule } from './lib.module'

@Injectable({ scope: LibModule })
class LibService { }

// app.component.ts
import { LibService } from './lib.service'

@Component({ /*...*/ })
class AppComponent {
  constructor(libService: LibService) {}
}

// app.module.ts
import { LibModule } from './lib.module'
import { AppComponent } from './app.component'

@NgModule({
  declarations: [AppComponent],
  import: [LibModule],
})
class AppModule {}
```

The new dependence flow is:

* Library Service imports Library Module
* Application Component imports Library Service
* Application Module imports  Application Component
* Application Module imports Library Module

Thus, as long as application component is removed, library service is not depended at all and can be safely removed together.

~~WAT? So why is libModule there? As a cosmetic?~~

We can optionally choose providers like `useClass`, `useFactory` to set implementation to other values instead of current class.
_(translator note: they are Angular's alternative DI functions and are DCE ready)_

```ts
@Injectable({
  scope: SomeModule,
  useValue: { valueOfLife: 42 },
})
abstract class MyService {
  abstract valueOfLife: number
}
```

Of course, the new Injectable API only handles one ideal situation where **dependency declared** is **dependency used**. Nevertheless this is the most common situation. If intermediate Injector nodes overrides `Provider`, side effect is still ineluctable and thus adds code size. (But the injected dependency is probably used when one does override)


G. New template compilation
---

Template compilation, at macro level, **has only two types:**

* (structural) data
* (operational) instructions


Take a simple example. If we compile template (or equivalent) to some `render` method and if:
* calling `render` **doesn't make view changed**, but its return value is **what to be rendered**. This is the former type.
* calling `render` **does update view**, and thus **no return value is needed**. This is the latter type.

The most early Angular, v2 version, compiles template to instructions. One can refer to [this zhihu answer](https://www.zhihu.com/question/53434390/answer/134956516) for compilation behavior. This style is very similar to [Svelte](https://github.com/sveltejs/svelte): compile as much detail as possible, in order to reduce common runtime dependency (shared code).

But the problem of this approach is obvious. With more and more template authored, compiled code size will easily exceed shared code (Off-topic: the 0kb-boasting framework Svelte also provides a `shared` compile option to use library).


Later Angular v4 compiles template to data and introduces _View Engine_ as common dependency. Its compilation is explained [here](https://zhuanlan.zhihu.com/p/32137489). This style is very close to virtual dom, except that dom-like data is stored per type, not per instance.


Ivy renderer in v6 re-chooses compiling to instruction approach. Different from v2, v6 employs a strategy to minimize compiled code size.

```ts
export class AppComponent  {
  static ngComponentDef = defineComponent({
    type: AppComponent,
    tag: 'my-app',
    template: function AppComponent_Template(comp: AppComponent, cm: boolean) {
      if (cm) {
        E(0, 'div', ['class', 'container'])
          E(1, 'div', ['class', 'jumbotron'])
            E(2, 'div', ['class', 'row'])
              E(3, 'div', ['class', 'col-md-6'])
                E(4, 'h1')
                  T(5, 'Angular v6.x.x (Ivy Renderer)')
                e()
              e()
              E(6, 'div', ['class', 'col-md-6'])
                E(7, 'div', ['class', 'col-sm-6 smallpad'])
                  E(8, 'button', ['type', 'button', 'id', 'run', 'class', 'btn btn-primary btn-block'])
                    L('click', () => {
                      comp.run()
                      detectChanges(comp)
                    })
                    T(9, 'Create 1,000 rows')
                  e()
                  //...
                e()
              e()
            e()
          e()
          E(20, 'table', ['class', 'table table-hover table-striped test-data'])
            E(21, 'tbody')
              C(22, [NgForOf], trTemplate)
            e()
          e()
          E(24, 'span', ['aria-hidden', 'true', 'class', 'preloadicon glyphicon glyphicon-remove'])
          e()
        e()
      }
      p(22, 'ngForOf', b(comp.data))
      p(22, 'ngForTrackBy', b(comp.itemById))
      cR(22)
        r(23, 0)
      cr()

      function trTemplate(row: NgForOfContext<Data>, cm: boolean) {
        if (cm) {
          E(0, 'tr')
            E(1, 'td', ['class', 'col-md-1'])
              T(2)
            e()
            E(3, 'td', ['class', 'col-md-4'])
              E(4, 'a', ['href', '#'])
                L('click', (e: MouseEvent) => {
                  comp.select(row.$implicit, e)
                  detectChanges(comp)
                })
                T(5)
              e()
            e()
            //...
          e()
        }
        p(0, 'class.danger', b(row.$implicit.id === comp.selected))
        t(2, b(row.$implicit.id))
        t(5, b(row.$implicit.label))
      }
    },
    factory: () => new AppComponent()
  })
}
```

_(The density of information is almost as high as HTML.)_


Compiling to instructions has another advantage over compiling to data: common dependency is still DCE friendly.
Used instructions will be directly depended, and non-used instructions will be removed by optimizer.
On the other hand compiling to data requires all operations used by template processor, and thus cannot be optimized statically.

Therefore, Ivy's new compilation strategy should be the one with minimal code size (not regarding of the cost to implement compiler).


Memory and Speed
---

* New view layer employs compact binary frame design (and more bitwise operations).
* New view layer uses as many sequences (arrays) as possible rather than key-value pairs (objects) to store data.
* New view layer prefers adding expando properties on exposing types rather than new wrapping type.
* DI uses [bloom filter](https://en.wikipedia.org/wiki/Bloom_filter) to speed up `Directive` searching

(View layer is less friendly to pull request)

Summary
---

The new Ivy mode push "building-friendliness" to extreme, but you can also say it is not "non-building-friendly". It won't be useful for projects built with bare `<script>` tags.

Ivy's most crucial target is cooperation with [Angular Element](https://zhuanlan.zhihu.com/p/30717321). By encapsulating Angular components to custom elements (web components), we can achieve standalone publication, independent importing and independent usage of Angular as widgets. (To some degree this strangulates Svelte?)

The most important part of this strategy is code size. Even wihout common runtime library, Angular should work with minial size.
Components used as Angular Elements will not depended on external packages like forms and router, hence creating great value for size reduction.

Whether should NgModule still exist? It certainly has value on organizing code. That said, Dart version didn't ever has NgModule. It is sure that application structure can be well formed without NgModule (at least for Googler).
With decreasing usage of NgModule in real world API, it might be optional in future. But I'm sure v6 won't change NgModule. (I personally favor NgModule, but oppose enforcing it in app).

For applications plumbing many third party libraries, the main size problem might not come from Angular but rather from, for example, incorrect use of RxJS, importing `moment.js` inadvertently, or writing all styles in "comonent styles". For specific size problem one needs to analyze it specifically. Don't pray to Angular's advanced optimizer.

The worst part of Ivy is `OnChanges`. It is now implemented by `Object.defineProperty`! It is no longer right to say Angular is based **solely** on dirty check. All articles about change detection need updating! And every section needs separate explaination!


Lifecycles might have big change in near future. But it will await Ivy being the default, no earlier than v7. In fact, Angular's overall extensibility and runtime preprocessing has been greatly improved.
