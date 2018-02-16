title: Vue2 + TypeScript2 -- an introductory guide
date: 2016-10-03
tags: TypeScript
---

Safety check and error detection are important to development experience. Vue has gone to [great lengths](https://github.com/vuejs/vue/issues/3831#issuecomment-250996462) to provide [component information](https://github.com/vuejs/vue/pull/3656) when typos or mistakes happen. But we can do better, error reporting can happen at compile time. This is why we want to use TypeScript with Vue.

![final result](https://github.com/HerringtonDarkholme/vue-ts-example/raw/master/screen.png)
> Our goal: writing Vue in TypeScript

Goals
-----
In this guide we will finally achieve these goals:

* writing type safe vue instance declaration
* writing TypeScript in Vue's [single file component](https://vuejs.org/guide/single-file-components.html)
* semantic completion in pure TypeScript file
* type checking all TypeScript, even TypeScript in `*.vue`'s script tag.

> All code example can be found [here](https://github.com/HerringtonDarkholme/vue-ts-example), with separte commits standing for sections in this article.


Prerequisite
----

You have to know about TypeScript2.0 and Vue2.0. And optionally webpack's knowledge is very useful. And of course if you're reading this article you must have known `npm`.


Basic setup:
-------

First make a new directory for your project. And then init your new project and install dependencies.

```bash
mkdir vue-ts-test
cd vue-ts-test
npm init
# pressing `enter`s and done
npm install vue --save
npm install typescript --save
```

Note both vue and TypeScript need to be version 2. They have both definition files baked in so it's much easier to start up with them than before. (I still respect definitely typed!)

make html template, say, `index.html`:

```html
<div id="app"></div>
<script src="node_modules/vue/dist/vue.js"></script>
<script src="app.js"></script>
```

Then in your `app.ts`:

```typescript
declare var Vue: any
new Vue({
  el: '#app',
  render(h) {
    return h('h1', 'hello world')
  }
})
```

And compile your code:
```bash
./node_modules/.bin/tsc app.ts
```

Then open `index.html` in your browswer! It should give you a hello world!


Exploring Vue with TypeScript
-----

But there are a lot of feature missing in the above example. For example, you want a build step to combine all your scripts. You want component style co-locates with template and script, yet still sytnax highlighting  every thing.

Let's use Vue's genius single file component. It requires [webpack](https://webpack.github.io/) and [vue-loader](http://vue-loader.vuejs.org/en/index.html)


```bash
npm install webpack --save-dev
npm install vue-loader --save-dev
npm install css-loader --save-dev
```
Or you can use [vue-cli](https://github.com/vuejs/vue-cli)

```bash
vue init webpack-simple vue-ts-test
```

You need to make vue-loader to transpile your TypeScript code. [vue-ts-loader](https://github.com/HerringtonDarkholme/vue-ts-loader) can work with vue-loader.

```bash
npm install vue-ts-loader --save-dev
```

And in `webpack.config.js`

```javascript
module.exports = {
    entry: { app: './app.ts', },
    output: { filename: 'app.js' },

    // resolve TypeScript and Vue file
    resolve: {
        extensions: ['', '.ts', '.vue', '.js']
    },

    module: {
        loaders: [
            { test: /\.vue$/, loader: 'vue' },
            { test: /\.ts$/, loader: 'vue-ts' }
        ],
    },
    vue: {
      // instruct vue-loader to load TypeScript
      loaders: { js: 'vue-ts-loader', },
      // make TS' generated code cooperate with vue-loader
      esModule: true
    },
}
```

Now you can add write Vue code in single file component!

Suppose in `app.vue`

```html
<template>
<h1 @click="hello">hello world</h1>
</template>

<script>
export default {
  methods: {
    // type annotation!
    hello(): void {
      alert('hello world')
    }
  }
}
</script>
```

And now you can change your `app.ts` into an entry file.

```typescript
declare var require: any

import Vue = require('vue')
var App = require('./app.vue').default

new Vue({
  el: '#app',
  components: { App },
  render: h => h('app')
})
```

Now `webpack` your in your application directory. You can still see the `hello world`? Click it should give you an alert.

Cool? And here is the [source](https://github.com/HerringtonDarkholme/vue-ts-example/tree/f3d0d409d9b5de8b6f20b5a06b156b7313703475).

Toward a more type safe API
---

Vue has a [typechecker hostile API](http://herringtondarkholme.github.io/2016/08/21/vue-ts/).
We can use helper library to get a better type checking. [av-ts](https://github.com/HerringtonDarkholme/av-ts), Awesome Vue for TypeScript, is a good start point to use.

Install it by npm, again.
```bash
npm install --save av-ts
```

Then you can change your `app.vue` to this.

```html
<template>
<h1 @click="hello">hello {{name}}</h1>
</template>

<script>
// import dependency
import {Vue, Component} from 'av-ts'

// decorat vue class
@Component
export default class App extends Vue {
  // undecorated property will be packed in `data` option
  name = 'world'
  // undecorated method is just method
  hello() {
    alert('hello ' + this.name)
  }
}
</script>
```

`webpack` will give you an error. Because decorator is still considered experimental. You need to switch it on explicitly.

Add a `tsconfig.json` in your directory.

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "target": "es5",
        "experimentalDecorators": true
    }
}
```

Now it compiles! Check it out in browser!


av-ts has several features that isn't present in other libraries.
First, `prop` is defined in class, so you can access these as properties in other methods without declaring in class again. Second, `data`, `render` and lifecycle hooks are declared by decorator, which is a mark to tell you these methods are different. Plus, their names and signatures can be checked by decorators. Third, extension is easy in av-ts. You can register new decorator, and still keep type safety.

For source as a whole, you can take a look [here](https://github.com/HerringtonDarkholme/vue-ts-example/tree/2609c7c754379c86788bb7bf515eb001989c0b6a).


Even more typesafety
----

[Vuex](http://vuex.vuejs.org/en/index.html) is another popular state management library for vue.

[Kilimanjaro](https://github.com/HerringtonDarkholme/kilimanjaro) is a type safe fork for Vuex.

To create a store is familiar with its original API.

```typescript
import { create } from 'kilimanjaro'

var store =
  create({count: 0}) // create a state with initial state
  .getter('count', s => s.count) // a getter
  .mutation('increment', s => () => s.count++) // define a mutation
  .mutation('decrement', s => () => s.count--) // payload is in the second parameter parenthesis
  .done()

store.commit('increment') // make a state change
// store.commit('ncrement') // typo won't compile! magic!
```

By heavy use of string literal type and overloading, kilimanjaro provides a similiar type safe API to vuex.
If you mistakenly commit a wrong mutation name, or the mutation payload does not have correct type, kilimanjaro will make TypeScript compiler report that for you.

There is also component helper in kilimanjaro. Check the example below

```typescript
import {Vuex, getHelper} from 'kilimanjaro'

const { getters, commit } = getHelper(store)

@Componet
class App extends Vue {
  @Vuex count: number = getters('count')
  @Vuex add: Function = commit('increment')
  // @Vuex typo: Function = commit('ncrement') // won't compile
  // @Vuex wrong: string = getters('count') // type incompatible
}
```

You can get component helpers by calling `getHelper`. Helpers are function that take a getter/mutation/action name and return corresponding result. Given a mutation name, `commit` helper will return a function that, once called, changes store state. The similiar for `dispatch` helper.

`getters` is more magical. Given a getter name, it will return a *symbol* with correct result type. But at runtime, it will be changed to a `computed` accessor in the VM. The magic is done by the `Vuex` decorator, which is an extension of `av-ts`. Like mentioned above, av-ts is a very extensible library.


> For a working example, you can take a look [here](https://github.com/HerringtonDarkholme/vue-ts-example). And [Live demo](https://herringtondarkholme.github.io/vue-ts-example/) is here.

Vue-router is type safe enough for pragmatic usage. It is really a well written and designed library. Designing a library that provides 100% type safety is just an overkill bringing marginal benefits at huge cost of debugging compiling error.
If vue-router cannot cater for a type paranoid, this [sketch](https://github.com/HerringtonDarkholme/paranoid-router) will be interesting.

Limitation
---

You might want even more features from TypeScript, say

* semantic completion in single file component
* type checking template code

![go home, you're drunk](https://s-media-cache-ak0.pinimg.com/originals/86/72/e6/8672e6fb04620cbcf72ab3b1222ae5b5.jpg)


They are way way hard to implement. Semantic completion in `vue` file requires editor support and [compiler support](https://github.com/Microsoft/TypeScript/pull/10160). None of them is easy to implement.

You can write your vm code in a separate file and import into your vue template. But that is opposite to the design philosophy of single file component. In reality, keeping `vue` small and clean is better choice. With small number of symbols in source file, generic completion should also work well. (at least as well as plain JavaScript)

Template code is an even more heculean task. Vue compiles its template into `with(this) { ... }` format ([doc](http://vuejs.org/guide/render-function.html)). `with` statements are not checked in TypeScript, nor flow-type. Plus, vue compiler does analyze no expression, making it impossible to do checking without reimplementing a compiler. However, again, it is no worse than plain JavaScript.


Conclusion
---

Vue is an elegant and lightweight library. With the integration of TypeScript, Vue can be more safe and productive!

Compared to other frameworks, you can instantly get precompiled template code, small code footprint, flat learning curve.
But when it comes to scale bigger, you might want additional help.

There are overhead to integrate TypeScript with Vue, indeed. But the error checking and refactoring help still worth your try.

Tools we used to achieve type-safety
* TypeScript2 and Vue2, for their definition files
* [vue-ts-loader](https://github.com/HerringtonDarkholme/vue-ts-loader): load TypeScript in vue's single file component
* [av-ts](https://github.com/HerringtonDarkholme/av-ts): wrap vue's API in type safe way

Optionally
* [kilimanjaro](https://github.com/HerringtonDarkholme/kilimanjaro): type safe vuex, requires deeper understanding of TypeScript's advanced features.
