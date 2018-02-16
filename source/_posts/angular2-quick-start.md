title: The Real Angular2 quick start
date: 2015-10-25
tags: TypeScript
---

Angular2 [official page](https://angular.io/docs/ts/latest/quickstart.html) wants you to make some dirty hack to get the fastest  `hellow world` in Angular2. But it immediately requires to correct your first sin in the same 5-min quickstart page. Maybe it is possible for a newcomert to set up Angular2 properly in 5 minutes, but reverting previous dirty hack and then setting the correct thing up are annoying. So here is the REAL Angular2 quickstart that does not piss you off.

**DISCLAIMER**: Knowledge about [npm](https://www.npmjs.com/), [TypeScript](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json) and [SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/config-api.md) is recommended.
This quickstart deliberately skips explanation on the config and shell code for real real speed.


Step 1: Create a new folder for our application project.
====

```bash
mkdir angular2-quickstart
cd angular2-quickstart
mkdir -p src/app
```

Step 2: Install npm packages
===
First step towards front end project as usual...
```bash
npm init -y
npm i angular2@2.0.0-alpha.44 systemjs@0.19.2 --save --save-exact
npm i typescript live-server --save-dev
```
Version number sucks, but it will be removed after Angular2's public stable release.
We need to install angular2 and TypeScript as dependency, of course. SystemJS is used to load our app(alternatively one can use webpack or browserify).
Live-server gives us live-reloading in developement.

Step 4: Set up npm script tasks
===
Let's define some useful command.
**Find and replace** the `script` section in `package.json`
```json
...
  "scripts": {
    "tsc": "tsc -p src -w",
    "start": "live-server --open=src"
  }
...
```
`npm run tsc` compiles and watches file changes. `npm start` runs the live-reload server.


Step 5: Create our first Angular2 component
====

**Add a new file** called `app.ts` in `src/app`
```TypeScript
import {Component, bootstrap} from 'angular2/angular2';
@Component({
    selector: 'my-app',
    template: '<h1>My First Angular 2 App</h1>'
})
class AppComponent { }
bootstrap(AppComponent);
```

Step 6: Create our entrance html
===
**Create** an index.html in `src` foler.

```html
<html>
  <head>
    <title>Angular 2 QuickStart</title>
    <script src="../node_modules/systemjs/dist/system.src.js"></script>
    <script src="../node_modules/angular2/bundles/angular2.dev.js"></script>
    <script>
      System.config({
        packages: {
          'app': {defaultExtension: 'js'} // defaultExtension makes require `app/app` works without `.js`
          // more packages config goes here
          // ...
        }
      });
      System.import('app/app');
    </script>
  </head>
  <body>
    <my-app>loading...</my-app>
  </body>
</html>
```

Step 7: Compile TypeScript
===

**Create tsconfig.json** in the `src` folder.

```json
{
  "compilerOptions": {
    "target": "ES5",
    "module": "commonjs",
    "sourceMap": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "removeComments": false,
    "noImplicitAny": false
  }
}
```
And then, in a **new terminal window**.
```bash
npm run tsc
```

Step 8: Run the app!
===
And in **yet another new terminal window**.
```bash
npm start
```

You should see `My First Angular2  App` in your browser.


Hooray!
===
Now you can make some changes to your source file. `tsc` compiles file on the fly and `live-server` automatically refreshes the browser!

Hope you are not pissed off by this quickstart.



_P.S. Final structure_

```
angular2-quickstart
├── node_modules
├── src
│    ├── app
|    │    └── app.ts
│    ├── index.html
│    └── tsconfig.json
└── package.json
```
