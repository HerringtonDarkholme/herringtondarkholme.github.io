title: A mostly wrong web frameworks comparison by food analogy
date: 2024-01-27
tags: JavaScript

---

Welcome to the web framework restaurant, where we serve you a delicious dish of web frameworks with a side of humor and a pinch of salt.

I'm your waiter, Herrington, and I'll be serving you a mostly wrong comparisons of the web. Don't worry, I won't spit in your food, but I might twist some facts for your entertainment. 😈

I hope you enjoy your meal.

## The Analogy Ingredients

Before we dive into our main course, let's have a taste of our analogy ingredients.

Our first ingredient is HTML, the basic structure of a web page. It defines the elements and layout of a web page, such as headings, paragraphs, tables, and images. HTML is static, plain and bland, meaning it does not change or interact with the user. It's like bread, the staple food of many cuisines. You can eat it by itself, but it's not very exciting.

The second ingredient is JavaScript, which adds logic and functionality to a web page. It's like meat, the star of many dishes. JavaScript can manipulate HTML elements, handle user events, and communicate with servers. JavaScript is our Meat. It adds flavor and protein to your web page, making it more dynamic and interactive.

Together, they form the foundation of many web framework, just like bread and meat form the foundation of many dishes.

Cooking can have many variations, depending on the type, quality and quantity of bread and meat, as well as the addition of other ingredients, such as cheese, lettuce, tomato, sauce, etc.

Similarly, a web framework can have many variations, depending on the type, quality and quantity of HTML and JavaScript, as well as the addition of other features.

In this blog post, we will explore some of the most popular and influential web framework approaches, and compare them to some of the most popular foods.

Are you ready to sink your teeth into this tasty topic? Then let’s begin!

## Web Frameworks Review Menu

### HTML only, Dry and Crusty Bread 🍞

In the old days, we only had [HTML4.0](https://www.w3.org/TR/html401/) compatible webpages. You can see layout using `<table>`s, styling using `<font>`s and grabbing attention using [`maraquee`s](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/marquee). Every webpage back then was just static HTML. It was like dry and crusty bread. Nothing fancy, but it worked to present [CERN like documents](https://www.home.cern/science/computing/birth-web) on web. **How boring, right?**

### jQuery, Slapping a Slice of Ham on Bread 🍖

Then, JavaScript library like [jQuery](https://jquery.com/) came along. They were the spiced ham of web development, adding flavor and spice to virtually every page. JS libraries simplify the manipulation of HTML elements, the handling of user events, and the communication with servers. The `$` dollar from jQuery king really shined (probably is still shining). It is like slapping a slice of ham on bread. Suddenly, your web pages became more tasty and satisfying, with a touch of meaty goodness. But usually you are still using HTML as the main structure and data-source of your web page. You can add a lot of JavaScript to your web page and make it as dynamic as you want, but the boundary between HTML and JavaScript is still clear. Slapping a ton of hams on your bread still makes it a bread.

### Component Frameworks, Barbecue Time 🔥

Manipulating DOM alone is not enough for modern web app. We need a way to systematically manage the state of our web app, and we need to do it in a more structured way.

Then, frameworks based on components came along, e.g. Angular, React, Vue and etc. They are all different types of meat. [Angular](https://angular.io/) is like beef, it’s rich and heavy, but it requires a lot of preparation and seasoning. [React](https://react.dev/) is like chicken, it’s light and versatile, but it needs some extra ingredients to make it tasty. [Vue](https://vuejs.org/) is like pork, it’s sweet and easy, but it may not be everyone’s cup of tea. They are all client-side only frameworks. They run on the browser, and they are all JavaScript frameworks.

Running these JavaScript to create DOM in browser is like cooking barbecue at your backyard home.

You can add as much meat and seasoning as you want, but your cooking time may be infinitely extended.
Similarly, adding a lot of interactive components or dynamic contents may hurt your page loading performance.

### SSR, Still Barbecue, but with Bread Stick 🥖

Hosting a barbecue party at your home (browser) is fun! But waiting for a slow grill, say a five-year-old mobile phone's CPU, is frustrating.
Hangry guests are waiting for their webpage food to be ready, but they only see smoke and fire (loading screen).

That's where SSR, or Server Side Rendering, comes in handy. SSR is not a framework, but a technique that lets you cook some of your food in advance at a professional kitchen, the server. SSR runs your component code on the server and generates HTML pages that are ready to consume.
The HTML pages are then sent to your users as static HTML bread stick appetizer. Your guests can munch on them while your grill is still working on the rest of the food.

But there's more than breadstick! SSR also sends you the component code, so you can run it on your local grill and make your food more juicy and meaty. This process is called hydration, which allows you to add interactivity and functionality to your static HTML, such as handling user events and communicating with servers.
This is serving JavaScript meat juice to make dry HTML breadstick moist (dynamic).

More often than not will you see hydration mismatch. Alas, that means the DOM generated on client side is not the same as HTML generated by server, usually caused by different rendering content or [violation of HTML rules](https://nextjs.org/docs/messages/react-hydration-error). Serving SSR needs your meticulous cooking and plating to perfectly plate your patty on your bread.

SSR is like serving pre-made breadsticks to your guests while you are still making barbecue. It improves the performance and user experience of your web page, by reducing the loading time and increasing the perceived speed. Yet, it is not the silver bullet.


### Island Architecture: The Burger 🍔

Let's face it: running the same JavaScript on both server and client is like cooking the same meat twice. It's inefficient, wasteful, and potentially harmful. We don't want to overcook our meat.

But not all components need to be cooked. Some of them are fine as they are since they are bread. Can we just keep them as pure HTML, and only cook the ones that need to be juicy and tender, like the meat?

That's exactly the idea behind the ["Islands" architecture](https://www.patterns.dev/vanilla/islands-architecture/), which is mostly promoted by [Astro](https://docs.astro.build/en/concepts/islands/). These frameworks allow you to create web pages that are a mix of static and dynamic content, like a burger.

The "Islands" architecture works like this:
1. It renders HTML pages on the server like pre-baking the bun. (no [pun](https://bun.sh/) intended)
2. It injects placeholders or slots around highly dynamic regions, like cutting holes in the bread for the meat and other toppings.
3. It sends static HTML pages to the client, like delivering the bread to your home.
4. It hydrates the placeholders or slots on the client into small self-contained widgets, like adding the meat and other toppings to the bread.

The result is a web page that is a burger: a dynamic JavaScript patty sandwiched between surrounding static HTML breads. It's fast, tasty, and satisfying.

Don't believe me? Just look at Astro's official site. They have a burger in their chart. It's exactly the same!


### Qwik: The New Pizza 🍕

Besides partial hydration techniques like islands-architecture, you might also think about the possibility of incremental hydration.
You may want to try [Qwik](https://qwik.builder.io/), a new framework that promises instant-on interactivity, regardless of size or complexity.

Qwik is different from other frameworks that use the Islands architecture, which selectively chooses components to hydrate. Qwik takes a more radical approach: it chops your app into tiny pieces and hydrates them only when they need to be interactive. Qwik calls this resumability, which means your app can resume from any state without reloading or re-rendering.

Qwik works like this:
- It renders static HTML pages on the server, like making pizza dough.
- It leaves a special html attribute to mark the dynamic components, like sprinkling cheese on the dough.
- It sends static HTML pages to the client, like delivering the pizza to your home.
- It hydrates the dynamic components on the client according to the user's interaction, like adding toppings to the pizza.

Qwik is like pizza: you can eat it as soon as it arrives, or you can customize it to your liking. You can add pepperoni, bacon, sausage, or whatever you want, slice by slice, until you get the full interactive pizza. It's delicious, satisfying, and quick. (pun intended)


![Qwik pizza dough makes your page load faster](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yknze8w385l13qsi6r8n.png)


### HTMX: The Renaissance of Ham Sandwich 🥪

Do you remember the good old days, when web pages were simple and fast, and JavaScript was just a garnish? Well, [HTMX](https://htmx.org/) wants to bring back those days, with a modern twist.

HTMX is a framework that lets you create web pages that are mostly HTML, with a little bit of JavaScript.

* HTMX creates web pages by rendering HTML pages on the server, like slicing bread, and sending them to the client, like putting bread on a plate.
* Then, it adds interactivity by running minimal JavaScript on the client when user interaction happens, like adding ham to the bread.
* Finally, it updates the web page by fetching new HTML pages from the server and appending or swapping them to the DOM, like making a sandwich.

HTMX is like a ham sandwich: it's simple, tasty, and satisfying, with generous sprinkles of [memes](https://htmx.org/essays/). It's also fast, because it doesn't load unnecessary JavaScript or re-render the whole page. HTMX is the ham sandwich of the web, and it will make you nostalgic.

HTMX is not the only framework that does this. [Turbo](https://github.com/hotwired/turbo) or [Turbolinks](https://github.com/turbolinks/turbolinks) are also frameworks that use the same technique. They are like different kinds of ham, or different kinds of bread.

### React Server Components: The Ultimate JSX Lasagna 🍝

You might have heard about [React](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md) [Server](https://react.dev/blog/2020/12/21/data-fetching-with-react-server-components) [Components](https://www.mayank.co/blog/react-server-components/) a lot. RSC is a relatively new feature that lets you render components on the server and stream them to the client in the form of JSX in JSON. Though the name contains React™, server component is not limited to certain frameworks.
Actually, Phoenix LiveView is a framework that has used the same model before RSC ever existed. [Streamlit](https://streamlit.io/) is also a nice example.

React Server Components are not a single type of components but a programming model. RSC has two kinds of components: server components and client components.

- Server components are rendered on the server into static JSX, in a JSON-like structure, and are streamed to the client, like cooking lasagna noodles on the stove and sending them to the oven.
- Client components are sent to the client as JavaScript files, and they generate JSX on the client, like adding raw meat sauce onto lasagna and cooking them in the oven.
- Server and Client components can be interleaved with each other, like layering noodles, sauce, cheese, and meat in the lasagna.

React Server Components are unique because they can mix static JSX and dynamic JSX all together, layer by layer. Unlike Astro or Qwik, which have strict rules about how to combine static and dynamic content, React Server Components are more flexible and versatile. You can have pepperoni over pizza, but you can't have pizza over pepperoni. That is, a dynamic component in Qwik can't have another pure static component inside it. Similarly, you can't have a burger bun inside the beef patty in Astro burger.

But you can have anything you want in React Server Components. You can have meat, bread, meat, bread, meat, bread, all the way down, freely mixing static and dynamic content. React Server Component model is like JSX lasagna, a delicious dish that can have any ingredients you like, as long as they fit in the layers. But the power does not come for free. RSC has a very steep learning curve and many features that cannot be covered in one blog post. You can read [tutorials](https://www.joshwcomeau.com/react/server-components/), [introductions](https://nextjs.org/docs/app/building-your-application/rendering/server-components) or even [quizzes](https://betterprogramming.pub/grok-react-server-component-by-quizzes-df4417905bc4). Oh, don't daunt people too much by going down the rabbit hole too deep. This article is just a parody for fun. Not for educational purpose.

Indeed, lasagna is a delicious and sophisticated dish, and, accordingly, has a complicated recipe that includes more steps and instructions.

(BTW there is [no Lasagna emoji](https://github.com/Crissov/unicode-proposals/issues/39) in unicode, [poor Garfield](https://twitter.com/Garfield/status/1524427614693244931))

## Take away

That's it for our web development history restaurant. I hope you enjoyed your meal and learned something new. Before you leave, here are some takeaways and leftovers for you to digest.

### Where do your Data or State Get Computed?

Our analogy is about the boundary between static and dynamic content. This is essentially asking where your data/state is cooked. If the data is cooked on the server and transformed into a static view, it will not change on the client side. It's like buying an off-shelf bread from a grocery store and eating it at home. You can't change anything in the ingredient list. You can only enjoy it as it is.

On the other hand, if the data is cooked on the client side, it can respond to user interaction and change the view, so it is dynamic content. It's like cooking meat at home and eating it fresh. Every interaction can be responded by JavaScript DOM nearly instantly without waiting. But instantiating JavaScript costs time, like grilling meat at home.

### What is the boundary between static and dynamic content?

We don't have a unanimous approach towards the boundary between static and dynamic content. So we have so many different web frameworks, each with their own flavor and texture.

* It can be totally separated. Static HTML or CSR falls into this category. It's like having bread and meat separately. You can eat them by themselves.

* SSR can mix static and dynamic content by first running JS on the server and rehydrating it on the client side.
* And on top of SSR, we can selectively and incrementally hydrate the dynamic content.
  * We can have the "Islands" architecture, which is SSR with selective hydration. It's like making html buns on the server, but leaving some spaces for the client to add their JavaScript beef patty.
  * Or we can have incremental hydration, which is Qwik's resumability. It's like making a pizza crust on the server, but letting the client add their own toppings, slice by slice.

* Finally, we have RSC, which is mixing static and dynamic content all together, layer by layer. We cook pasta on the server, but let the client add their meat sauce, layer by layer. The single mental model of React can rule all the server/client code. But you have to be a master chef to consciously cook the correct ingredients on the correct machines, either server or client.

I hope you enjoyed this article! Bon Appétit! 😋
