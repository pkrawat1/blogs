---
layout: post
title: "TodoApp in Angular2 with TypeScript"
date: 2015-08-25 15:35:36 +0530
comments: true
categories: ['Angular2', 'TypeScript']
---

In this post I'll be sharing my experience during the application setup.
I'll be creating an application from scratch, right from creating application folder.

#Let's get started, Hurrrrayyy!!!

###First thing first, create a the app folder
```bash
  $ mkdir todoapp
  $ cd todoapp
```

Now we need to download typescript definition manager(tsd), just like npm(node package manager).
You can install tsd with this simple command.

```bash
  $ npm install -g tsd  #use sudo if required
  $ tsd --version # Validate installation by typing tsd in the console.
```
You can checkout some commands and how to use them in this [npm repo](https://www.npmjs.com/package/tsd).

###Initializing the application
```bash
  $ tsd init
```
This generated the tsd.json file. Just like in case of npm package.json is created.
Anyone who has the access to this repository can do a $ tsd reinstall , like npm install we all are so used to.

###Let's install typescript libraries
```bash
  $ tsd query angular2
    - angular2 / angular2
```
Let's fetch angular2 from the definitely typed repo.
```bash
  $ tsd query angular2 --action install --save
```
Doing this will create a directory called angular2 in the typings directory with a file called 
angular2.d.ts
This will also update the tsd.json with current angular2 head sha.

```javascript tsd.json
  {
    "version": "v4",
    "repo": "borisyankov/DefinitelyTyped",
    "ref": "master",
    "path": "typings",
    "bundle": "typings/tsd.d.ts",
    "installed": {
      "angular2/angular2.d.ts": {
        "commit": "38fb591c6eba840e0b53d1110302b8e4fb04651c"
      },
    ...
  }
```
>**NOTE**: Typing for angular2 are not complete at the moment they plan to complete it before the beta lauch but we should be good to start playing around with the current basic type definitions.

##Let's install TypeScript compiler

For the Typescript to compile we need a tsconfig.json file in the root of the project.

There are two basic components of a tsconfig.json file.

1. compilerOptions
2. files

Sample tsconfig.json from [TypeScript Wiki](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json)

```javascript tsconfig.json
  {
    "compilerOptions": {
      "module": "commonjs",
        "noImplicitAny": true,
        ...
    },
      "files": [
        "core.ts",
      "sys.ts",
      ...
        ]
  }
```

A lot of options can be passed to [compilerOptions](https://github.com/Microsoft/TypeScript/wiki/Compiler-Options). You can find that out from the wiki. It also uses default compiler options if none is given and also all the files will be included by default.

So let’s go ahead and add our own tsconfig.json file.

```javascript tsconfig.json
  {
    "version": "1.5.0",
    "compilerOptions": {
        "target": "es5",
        "module": "commonjs",
        "declaration": false,
        "noImplicitAny": false,
        "removeComments": true,
        "noLib": false,
        "emitDecoratorMetadata": true
    },
    "filesGlob": [
        "./**/*.ts",
        "!./node_modules/**/*.ts"
    ],
    "files": [
        "./app.ts",
        "./typings/angular2/angular2.d.ts",
        "./typings/es6-promise/es6-promise.d.ts",
        "./typings/rx/rx-lite.d.ts",
        "./typings/rx/rx.d.ts",
        "./typings/tsd.d.ts"
    ]
  } 
```

Now install the typescript that will be compiling or converting typescript code to 
Javascript code to make the browsers understand the code. As of now browsers don't
have native support for typescript nor ES6 completely.

Once installed we can use tsc for transpiling ts to js.

```bash
  $ npm install -g typescript
  $ tsc -w #realtime transpile ts to js.
```

###Application
First we will be creating a file called app.js, that was included in tsconfig.json.

```javascript app.ts
/// <reference path="typings/angular2/angular2.d.ts" />

import {Component, View, bootstrap, NgFor, NgIf} from "angular2/angular2";

@Component({
  selector: 'todo-app'
})

@View({
  template: `
    <h2>
      <ul>
        <li *ng-for="#todo of todos">
          {{ todo }}
        </li>
      </ul>

      <input #todotext (keyup)="doneTyping($event)">
      <button (click)="addTodo(todotext.value, $event)">Add Todo</button>
    </h2>
  `,
  directives: [NgFor, NgIf]
})

class TodoApp {
  todos: Array<string>;
  
  constructor() {
    this.todos = ["Eat Breakfast", "Walk Dog", "Breathe"];
  }

  addTodo(todo: string) {
    this.todos.push(todo);
  }

  doneTyping($event) {
    console.log($event.which);
    
    if($event.which === 13) {
      this.addTodo($event.target.value);
      $event.target.value = null;
    }
  }
}

bootstrap(TodoApp);
```
> For details on the code please visit these [step by step guides](https://angular.io/docs/js/latest/guide/).

Also install angular2, traceur, systemjs by adding it inside package.json.
Or by `npm install angular2 traceur systemjs`, whichever suits you.

1. Angular2 so that our transpiled Js can be used on the frontend
2. Traceur is es6 to es5 transpiler browsers don’t support es6 as of yet but not for long.
3. SystemJs is univerval dynamic module loader.

```javascript package.json
{
  "name": "todoapp",
  "version": "1.0.0",
  "description": "Todo App in Angularjs 2",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/pkrawat1/todoapp.git"
  },
  "author": "Pankaj Kumar Rawat",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/pkrawat1/todoapp/issues"
  },
  "homepage": "https://github.com/pkrawat1/todoapp#readme",
  "keywords": [
    "angular2",
    "typescript",
    "tds"
  ],
  "dependencies": {
    "angular2": "latest",
    "systemjs": "latest",
    "traceur": "latest"
  }
}

```

In order to use these components we need to create an index.html file.
The code is self explanatory here if not head over to [angular.io](https://angular.io/docs/js/latest/guide/) they have very good explanations up on their websites.

```html
<!doctype html>
<html>
  <head>
    <title>Angular 2 Todo App</title>
    <!--
    <script src="https://github.jspm.io/jmcriffey/bower-traceur-runtime@0.0.87/traceur-runtime.js"></script>
    <script src="https://jspm.io/system@0.16.js"></script>
    <script src="https://code.angularjs.org/2.0.0-alpha.28/angular2.dev.js"></script>
    -->
    <script src="node_modules/traceur/bin/traceur-runtime.js"></script>
    <script src="node_modules/systemjs/dist/system.js"></script>
    <script src="node_modules/angular2/bundles/angular2.dev.js"></script>
  </head>
  <body>
    <script>
      System.import('app');
    </script>
    <todo-app></todo-app>
  </body>
</html>

```

>For starting a local server, just install `npm install http-server`

If everything goes right you should see the list of tasks with an input box in the browser when you go to [http://localhost:8080](http://localhost:8080).

The repository of this whole code is hosted on [github](https://github.com/pkrawat1/todoapp.git) in v2 branch.

Reach out to me on my email, twitter or comment below if you are stuck anywhere and I’d be more than happy to help you.
