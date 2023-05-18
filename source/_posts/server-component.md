title: Grok React Server Component by Quizzes
date: 2023-05-07 22:26:14
tags: React
---

React Server Component is a new React architecture that the React team introduced at the end of 2020, which enables developers to render components on the server side, thereby boosting performance and streamlining code.

However, this innovation, though already more than two years old, still poses some novel challenges and issues, and many React experts are baffled or perplexed by it.
I believe it's not only me how are confused by the heated discussion on Twitter and hours long video explaining the new paradigm.
[Dan Abramov](https://twitter.com/dan_abramov) presented [three quizzes](https://twitter.com/dan_abramov/status/1648923232937058304) about React Server Component, arguably the hottest topic in the React community, to help the audience to understand the new technology better.

This article will analyze the three quizzes that half of the React experts on Twitter didn't get it right. I hope this can help you understand the RSC better.

> TLDR: React will render Client Component as a reference to the script on the server side, and Server Component will be streamed and rendered as a JSON-like UI. The references and JSON will be passed to the browser for coordination and view updates.

# The Three RSC Quizzes

## First Quiz

```typescript
function Note({ note }) {
  return (
    <Toggle>
      <Details note={note} />
    </Toggle>
}
```

the *only* Client Component out of these is Toggle. It has state (`isOn`, initially false). It returns `<>{isOn ? children : null}</>`.

what happens when you `setIsOn(true)`?

* Details gets fetched
* Details appears instantly

## Second Quiz

Now say `isOn` is true. You've edited the note and told the router to “refresh” the route. This refetches the RSC tree for this route, and your `Note` server component receives a note prop with latest DB content.

(1) does `Toggle` state get reset?
(2) does `Details` show fresh content?


* (1) yes and (2) yes
* (1) yes and (2) no
* (1) no and (2) yes
* (1) no and (2) no

## Third Quiz

Here's a little twist.

```typescript
<Layout
  left={<Sidebar />}
  right={<Content />}
/>
```

All are Server components. But now your want to add a bit of state to `Layout`, like column width, that changes on mouse drag.

Can you make `Layout` a Client component? If yes, what happens on drag?

------


I believe some readers who don't know RSC may be completely confused after reading these three questions and don't understand what they are asking. So at first, we will briefly introduce what is RSC for those who are new here. If you already know the purpose of RSC, you can skip the section safely.

# What is React Server Component?

React Server Component is a special React component that does not run on the browser side, but instead on the server side. So that it can directly access the server's data and resources, without obtain them through indirection APIs like REST or GraphQL, etc.

React Server Component is a pattern that can help us reduce the number of network requests and the size of data, thereby improving the page loading speed and user experience. React Server Component can also serve dynamic content to users according to different requests and parameters, without having to recompile or deploy.

The purpose of React Server Component is to let developers build applications that span the server and client, combining the rich interactivity of client-side applications and the optimized performance of traditional server rendering.
React Server Component can solve some problems that existing technologies cannot solve or solve well, such as:

* Zero package size: React Server Component's code only runs on the server side and will never be downloaded to the client side, so it does not affect the client’s package size and startup time. The client only receives the rendered results of RSC.

* Full access to backend: React Server Component can directly access backend data sources, such as databases, file systems or microservices without additional API endpoints.

* Automatic code splitting: React Server Component can dynamically choose which client components to render, so that the client only downloads the necessary code.

* No client-server waterfall: React Server Component can load data on the server and pass it as props to client components, thus avoiding the client-server waterfall problem.

* Avoid abstraction tax: React Server Component can use native JavaScript syntax and features, such as `async` and `await`, without having to use specific libraries or frameworks to implement data fetching or rendering logic.


# Server Component and Client Component

Before understanding how RSC works, we must first understand two big concepts in RSC, server-side components (Server Component) and client-side components (Client Component).

## Server Component

As the name suggests, server components run only once per request on the server, so they have no state and cannot use features that only exist on the client. Specifically:

* ❌ You cannot use state and side effects, because they (conceptually) run only once per request on the server. So `useState`(), `useReducer`(), `useEffect`() and `useLayoutEffect`() are not supported. You also cannot use custom hooks that depend on state or side effects.
* ❌ You cannot use browser-specific APIs, such as DOM (unless you polyfill them on the server).
* ✅ You can use `async`/`await` to access server data sources, such as databases, internal (micro) services, file systems, etc.
* ✅ You can render other server components, native elements (`div`, `span`, etc.) or client components.

Developers can also create some custom hooks or libraries designed for the server. All rules for server components apply. For example, a use case for a server hook is to provide some helper functions for accessing server data sources.

## Client Component

Client Component is a standard React component. It obeys all the rules we learnt about React before. The new rules to consider are mainly what they can't import server components.

* ❌ Cannot not import server components or call server hooks/libraries, because they only work on the server. However, server components can pass another server component as children to a client component.
* ❌ Cannot not use server-only data sources.
* ✅ You can use state and side effects, as well as custom React hooks.
* ✅ You can use browser APIs.

Here we need to emphasize the nesting of server components and client components. Although client components cannot directly import server components, they can use server components as children. For example, you can write code like `<ClientTabBar><ServerTabContent/></ClientTabBar>`. From the perspective of the client component, its child component will be a rendered tree, such as the output of `ServerTabContent`. This means that server and client components can be nested and interleaved at any level. We will explain this design in later quizzes.

# How RSC works?

After understanding server components and client components, we can now start to learn how RSC works. RSC rendering is divided into two major phases: initial loading and view updating. There are also two environments for RSC: server and browser. Note that although server components only _run_ on the server, the browser also needs to be aware of them for actual view creation or updating. Client components are similar.


## Initial loading

### Server

* [Framework] The framework’s routing matches the requested URL with a server component, passing route parameters as props to the component. Then it calls React to render the component and its props.
* [React] React renders the root server component, and recursively renders any child components that are also server components.
* [React] Rendering stops at native components (div, span, etc.) and client components. Native components are streamed in a JSON description of the UI, and client components are streamed in a serialized props plus a reference to the component code.
* [Framework] The framework is responsible for streaming the rendered output to the client as React renders each UI unit.

By default React returns a description of the rendered UI, which is a JSON-like data structure, rather than HTML. Using JSON data will allow new data to be more easily reconciled with existing client components. Of course, frameworks can choose to combine server components with “server-side rendering” (SSR) so that the initial render is also streamed as HTML, which will speed up the initial non-interactive display of the page.

On the server, if any server component suspends, React will pause rendering that subtree and send client a placeholder value. When the component is able to continue (un`suspend`), React will re-render the component and stream the actual result of the component to the client. You can think of the data being streamed to the browser as JSON, but with slots for suspended components, where the values for those slots are provided as additional items in the response stream later.

Let’s dive into the RSC protocol a little bit by looking at an example of rendering a UI description to help us understand this paragraph. Note this paragraph might not be precise and RSC implementation might change in the future.

Suppose we want to render a div.

```html
<div>
  Hello World
  <ClientComponent/>
</div>
```

After calling `React.createElement`, a data structure similar to the following is generated.

```typescript
{
  $$typeof: Symbol(react.element),
  // element tag
  type: "div",
  props: {
    children: [
      // static text
      "Hello World",
      // client compoennt
      {
        $$typeof: Symbol(react.module.reference),
        type: {
          name: "ClientComponent",
          filename: "./src/ClientComponent.js"
        },
      },
    ]
  },
}
```

This data structure will be streamed to browser, in format like this:

```typescript
// Sever Component Ouptut
["$","div",null,{"children":["Hello World", ["$","$L1",null,{}]]}]
// Client Component Reference
1:I{"id":"./src/ClientComponent.js","chunks":["client1"],"name":"ClientComponent","async":false}
```

The first array represents the output of a Server Component, and you can see that its output is similar to React's data structure, except that the structure is not an object but an array.
The array element `$` represents `createElement`, and the following element `{children: xxx}` represents props. In children, the first child is directly transmitted with a string. `L1` is a placeholder, and the `1` in `1:I` below corresponds to it, and `1:I`’s data will be filled into `L1`'s position.

You might be curious about what does the `I` mean. In [react server component's protocol](https://github.com/facebook/react/blob/main/packages/react-server-dom-relay/src/ReactFlightDOMRelayProtocol.js#L21-L36), `I` represents `ClientReferenceMetadata`, a data structure helping browser to find the correct script entry to the client component.
`1:I`’s output is a reference to a client component, which contains the script name, chunk name and export name, for the browser runtime (such as webpack) to dynamically import client component code. This structure is the streaming structure mentioned above.

In summary, a Server Component will be rendered into a JSON-like data that represents UI, while client components will be converted into a JSON data that expresses script references.


## Browser

* [Framework] On the client side, the framework receives the streaming React response and uses React to render it on the page
* [React] React deserializes the response and renders native elements and client components.
* [React] Once all client components and all server component outputs have been loaded, the final UI state will be displayed to the user. By then all Suspense boundaries have been revealed.

Note that browser rendering is gradual. React does not need to wait for the entire stream to complete before displaying some content. Suspense allows developers to display meaningful loading states while loading client component code and server components are fetching remaining data.

## View Updating

Server components also support reloading to see the latest data. Note that developers do not fetch server components individually: one component by one request.
The idea is that given some starting server components and props, the entire subtree will be refetched at once. As with initial loading, this typically involves integration with routing and script bundling:

### On Browser
* [App] When the application changes state or changes routes, it requests the server to refetch the new UI for the changed Server Component.
* [Framework] The framework coordinates sending the new route and props to the appropriate API endpoint, requesting the rendering result.

### On Server
* [Framework] The interface receives the request and matches it with the requested server component. And it calls React to render the component and props, and handles the streaming of the rendering result.
* [React] React renders the component to the destination, with different rendering strategies for components and initial loading.
* [Framework] The framework is responsible for gradually returning the streaming response data to the client.

### On Browser
* [Framework] The framework receives the streaming response and triggers a rerender of the route with the new rendering output.
* [React] React reconciles the new rendering output with the existing components on the screen. Because the description of UI is data, not HTML, React can merge new props into existing components, preserving important UI state such as focus or input input, or triggering CSS transitions on top of existing content. This is a key reason why server components return UI output as data (“virtual DOM”) rather than HTML.


## Summary So Far...
This section is very long, but we can summarize the working principle of RSC in one sentence.

Client Component will be rendered into a script reference, Server Component will be streamed into a JSON-like UI, Server Component with `async`/`await` will be replaced by a placeholder first, and then streamed to the browser after resolving.

The table below has more details and principles analysis.

|Phase|Platform|ServerComponent|ClientComponent|
|---|---|---|---|
|Initial Load|Server|Run, transformed into JSON UI|Do not run, passed as script reference|
|Initial Load|Browser|Do not run, mutating dom by JSON UI|Run, resolving script reference and mutating dom|
|View Update|Browser|Do not run, requesting server for new JSON UI|Run, updating client state|
|View Update|Server|Run, transformed into new JSON accroding to props and routing|Do not run|
|View Update|Browser|Do not run, updating dom by new JSON UI|Run, reconciling client state and RSC to dom|


# Three Quizzes, Three Features

Now let's see how the above rendering process is applied in the RSC quizzes?

This article will combine these three questions to explain the three major features of RSC: rendering completeness, state consistency, and commutative client/server component.


## Rendering Completeness

The first quiz:

```typescript
function Note({ note }) {
  return (
    <Toggle>
      <Details note={note} />
    </Toggle>
}
```

Let's write some more components to provide context for this question. Our `Toggle` component and `Details` component looks like this:

```typescript
"use client";

import { useState } from "react"

export function Toggle(props) {
  const [isOn, setIsOn] = useState(false)
  return (
    <div>
      <button onClick={() => setIsOn(on => !on)}>Toggle</button>
      <div>{isOn ? "on" : "off"}</div>
      <div>{isOn ? props.children : <p>not showing children</p>}</div>
    </div>
  )
}
```

```typescript
export async function Details(props) {
  const details = await getDetails(props.note);
  return <div>{details}</div>;
}

async function getDetails(note: string) {
  await new Promise((resolve) => setTimeout(resolve, 2000));
  return `Details for ${note}`;
}
```

In this example, `Note` and `Details` are server-side components. `Toggle` is a client-side component, but its children `Details` appears directly under the server-side component `Note. So when rendering Note, it will roughly be rendered into

```typescript
{
  $$typeof: Symbol(react.element),
  type: {
    $$typeof: Symbol(react.module.reference),
    name: "default",
    filename: "./Toggle.js"
  },
  props: { children: [
   // children, note the
   {
      $$typeof: Symbol(react.element),
      type: Details,  // Details is rendered!
      props: { note: note },
   }
  ] },
}
```

Notice that `Details` is always rendered on the server side and delivered to the client.

When `Toggle` is rendered on the client side, `Details` is not used, but its rendering result is still sent to the client side.

Even though `Details` is an asynchronous server component that uses `async`/`await`, it can still be sent to the front-end after it finishes asynchronously due to the streaming process of React Server Component.

And when the user changes state, the client can directly use the server's pre-rendered results for dom operations because `Details` props are the same as those rendered by the server. Therefore, the answer to this question is that Details will appear immediately.

**This question reveals the "completeness" of React Server Component: as long as the component appears under the render function of the server-side component, it will be rendered regardless of its usage in client side.**

## State Consistency

> Now assume that isOn is `true`. You edit the note and tell the router to "refresh" the route. This will re-fetch the RSC tree for this route, and your `Note` server component will receive a `note` attribute with the latest database content.

The second question reveals the consistency of the RSC. When the `Toggle` component changes props on the client side, this change is synchronized between both the server-side component and the client-side component and remains consistent on both ends.
`<Details note={note} />` When a `note` changes, React detects the change in the note and sends a request to the server for the new `Details` rendering data.

Also, the state of the client component `Toggle` itself is not reset or lost in the browser.

**Thus, the design of RSC ensures that the state of the application is consistent across both server and browser.**

## 3. Commutative Client/Server Component

For the third question, let's expand the question code for context as well.

```typescript
function App() {
  return (
    <Layout
      left={<Sidebar />}
      right={<Content />}
    />
  )
}
```

`Layout` component is a server component.

```typescript
// Server Component
export function Layout(props: {
  left: React.ReactNode
  right: React.ReactNode
}) {
  return (
    <div>
      <div>
        <div style={{ width: `${width}px` }}>{props.left}</div>
        <div style={{ width: `${500 - width}px` }}>{props.right}</div>
      </div>
    </div>
  )
}
```

Let's rewrite it to a client component that `useState`. In this example, the width is changed on the client side by changing the input slider. (The implementation detail is inconsequential here).

```typescript
"use client"

import { useState } from "react"

export function Layout(props: {
  left: React.ReactNode
  right: React.ReactNode
}) {
  const [width, setWidth] = useState(200)
  return (
    <div>
      <input
        type="range"
        step={1}
        value={width}
        onChange={(e) => setWidth(Number(e.target.value))}
      />
      <div>
        <div style={{ width: `${width}px` }}>{props.left}</div>
        <div style={{ width: `${500 - width}px` }}>{props.right}</div>
      </div>
    </div>
  )
}
```

Note, in this case we can change the `Layout` from server component to client component without the `App` component.
The only thing changed is how the `App` is rendered on server side.

```diff
 {
   $$typeof: Symbol(react.element),
-  type: Layout,
+  type: {
+    $$typeof: Symbol(react.module.reference),
+    name: "default",
+    filename: "./Layout.js"
+  },
   props: {
    left: {
       $$typeof: Symbol(react.element),
       type: Sidebar,
    },
    right: {
       $$typeof: Symbol(react.element),
       type: Content,
    }
   },
 }
```

As you can see, during serialization, the Layout is transformed from a server-side component to a client-side component. The `type` field is change from a direct import of Layout component to a `module.reference`. Meanwhile its child components remain unchanged.

Before we change `Layout` to client component, the process of rendering `Layout` happens completely on the server side, and its children are also rendered on the server side. The rendered results are sent to the browser to be transformed into DOM.

After we change `Layout` to a client component, the process of rendering `Layout` happens in the browser, but the child components are still rendered on the server side. When the browser renders the server's JSON UI output, `Layout` inserts the results of the server-side child components into the browser DOM.

Since the props of the child components are not changed when user changes the layout width on client-side (because they have no props), so the rendering result on the server side does not need to be recaptured.

Therefore, the answer to this question is "it can be converted to a client-side component and the child components will not be recaptured".

**We can rewrite server-side components as client-side components in RSC projects without rewriting component composition at use site. We can call this interchangeability as "commutative" server/client components.**


# Conclusion

The documentation and RFC for React Server Component is relatively obscure and does not give practical examples, leading many people to wonder what it really is.

In this article, I tried to explain the design ideas and principles of React Server Component by explaining it with Dan's quizzes.
I hope it can help you understand this new feature and become one of the few materials that can let you learn RSC without watching Youtube or following Twitter threads. I hope this will let you understand more about the principle of RSC and the three performance characteristics! Complete Rendering, Consistent State, and Commutative Server/Client Components.

It is not easy to create, if you think this article is helpful to you, please follow me on [Medium](https://medium.com/@hchan_nvim) or [treat me a cup of coffee](https://github.com/sponsors/HerringtonDarkholme/).


# Reference
1. React 18: React Server Components | Next.js. https://nextjs.org/docs/advanced-features/react-18/server-components.
2. What you need to know about React Server Components. https://blog.logrocket.com/what-you-need-to-know-about-react-server-components/.
3. React Server Components. - It’s not server-side rendering. | by Nathan .... https://blog.bitsrc.io/react-server-components-1ca621ac2519.
4. What are React Server Components? - FreeCodecamp. https://www.freecodecamp.org/news/what-are-react-server-components/.
5. How React Server Compoents Wors. https://www.plasmic.app/blog/how-react-server-components-work#the-high-level-picture
6. 45% failed Dan's Server Component Quiz https://www.youtube.com/watch?v=AGAax7WzStc
