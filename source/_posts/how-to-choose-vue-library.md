title: Compare TypeScript libraries for Vue -- a review
date: 2016-11-01
tags: TypeScript
---

Vue has a lot of TypeScript  binding libraries. A lot of.

They look similar, with subtle difference. How to choose one?

Here I compare three libraries.

* [vue-typescript-component](https://libraries.io/npm/vue-typescript-component)
* [vuets](https://github.com/vuets/vuets)
* [av-ts](https://github.com/HerringtonDarkholme/av-ts/)

They are representative, and the analysis probably also applies to the [official library](https://github.com/vuejs/vue-class-component).

> Disclaimer: I'm the author of av-ts. So this article is probably biased. But I tried my best to give an objective review.

### 0. Overview
Vue-typescript-component and av-ts are decorator based libraries. Vue-typescript-component is more like the popular typescript [library](https://github.com/itsFrank/vue-typescript), but supports Vue2.0. av-ts is my own library, crated after I read many alternatives.
vuets is quite different. It resembles the original object literal style API from vue.js.
Before I start the review, I want to state a rule of thumb before.

> The more flexiblity a library boasts, the harder for a type checker to verify it.

It is quite common in programming world. For example, JavaScript is very flexible and dynamic, but it is usually hard to determine whether a property access is legal. Java is a static language, `javac` can help you check many pernicious typos in code, but Java cannot easily do reflective programming.

Vue has a very flexible and elegant API, but it comes [at the cost of type safety](http://herringtondarkholme.github.io/2016/08/21/vue-ts/). That's why all these Vue TypeScript libraries exist. How to balance between expressiveness and type safety is the main concern of such a library.

### 1. Compatibility
All the three libraries, vue-typescript-component, vuets and av-ts, have good support for Vue2 and TS2. So there is not much to compare here. (vuets has vue1.0 support while the other two do not)

### 2. Extensibility
I think this is the best part of av-ts. av-ts provides a `Component.register` for custom decorator. For example, if you want to use vuex in your project but also want to preserve type safety, you can create your own decorator. This is exactly what I do in [kilimanjaro](https://github.com/HerringtonDarkholme/kilimanjaro), a typed vuex fork.
vue-ts-copmonent does not provide extensibility at all. vue-ts has the same extensibility as original vue because it uses object literal style (that is, the most extensible one among the three). However, vuets also support class component like syntax. Mixing two style is somewhat confusing.

### 3. Idiom
av-ts and vue-ts-component are both idiomatic TypeScript code. class/property/component just feel like plain-old-typescript. vue-ts chooses object literal style API. While it looks the most similar to original vue, it is not as idiomatic as the other two in TypeScript land. Also, object literal style requires many manual typing annotation, which feels foreign to both JavaScript and TypeScript.

### 4.Conciseness
vue-ts is the most verbose as mentioned above. Object literal style API is almost impossible to have good type inference for contemporary compilers. You can read more about it [here](http://herringtondarkholme.github.io/2016/08/21/vue-ts/). Basically I don't think vue-ts is an ideal approach.
av-ts and vue-ts-component are concise. Most methods/properties require only annotating once. av-ts requires more decorators for special methods like render and lifecycle, while it has a more concise and type safe property syntax. So I think this is a tie.

### 5. Type Safety
av-ts and vue-ts-component can inject Vue's native types by `extends Vue`. Neat. vuets users have to re-annotate all the method again, [alas](http://herringtondarkholme.github.io/2016/08/21/vue-ts/). av-ts has more type safety when defining special methods like `render`, `lifecycle` and `transition`. This is powerer by new decorators introduced. Again, these decorators are created by `Component.register` and the implementation is not hard-coded into one file.

### 6. Expressiveness
vuets is as expressive as original vue, at the cost of more annotation. vue-ts-component can express most basic API in vue, but there is something it cannot. av-ts can use [mixin](https://github.com/HerringtonDarkholme/av-ts#mixin) and can [use props in data method](https://github.com/HerringtonDarkholme/av-ts#data). I have thought over these scenarios before hand.

### Conclusion
av-ts may not be the best component library for vue, but I believe it strikes the sweet point between conciseness, type safety and expressiveness. I'm happy to see more TS+Vue users try it out, and give me feedback so it can become better.
