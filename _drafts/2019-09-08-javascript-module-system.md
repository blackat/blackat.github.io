---
layout: single
title: "JavaScript Module System"
date: '2019-09-08'
last_modified_at: '2019-09-08'
comments: true
categories:
  - javascript, module, es6
tags:
  - [javascript, es6]
toc: true
toc_label: "On this page"
toc_icon: "list"
toc_sticky: true
---

Modular programming is a cross-language design technique to build large softwares. It allows the reuse of functionalities via interfaces.

JavaScript is affected by the _global scope syndrome_ so different patterns have been developed during the years to partition this scope. Modules evolved from the first rudimentary solutions to module loaders both server side and frontend side, till the ESM (ECMAScript Module) that standardize how modules are loaded by a _modern browser_.

## Lingo

- _Loosely coupled:_ the use of interfaces hides the implementation details, so two units are not tighly linked by the specific implementation.
- _Expose/export:_ the way to make interface elements visible to the outside of the world.
- _Use/import:_ the way the outside world can see and use the exported interface elements of a module.
- _Module:_ independent and autonomous unit of work.
- _Unit of work:_ limited set of functionalities related to a specific domain and/or field.
- _Interface:_ the _"handle"_ to use a module, a set of elements (functions, objects, arrays and values) exposed to the external world, _simply the API_. It could be also seen as the _documentation_ of the module.
- _Function scope:_ private space of each function, everything defined within it is invisible outside.

## Why modules

__Modular programming__ is a software design technique that facilitates the construction of large software programs by _decoupling_ them into small _loosely coupled_ modules.

A __module__ is an _independent and autonomous_ unit that provides a _functionality_. In order to use this functionality the _module exposes an interface_ to document the usage and hide the implementation details.

The key concepts of module are:

- __separation of concerns:__ separation in units based on the functionality, better organized than monolithic design;
- __interaction through well-defined interfaces:__ interface _abstracts_ from implementation details that are _hidden_;
- __reusability:__ being an autonomous and independent unit of work can be used elsewhere, a [simple API encourages reusability](https://ponyfoo.com/books/practical-modern-javascript#toc);
- __independent and interchangeable units:__ can be tested in isolation, easier to spot issues and can be substituted if required.

__Warning:__ concepts like `module`, `package`, `component` and `assembly` vary a lot among lanaguages.
{: .notice--warning}

### Export and import

_Export_ and _import_ are really the most important operations when we talk about modules:

- __export__ means that a module implements a functionality and _export_ the interface to use it, the API;
- __import__ means that anybody wants to use the functionality has to _import_ the _elements_ defined in the interface.

### JavaScript module implementation options

There are several options to modularize an application:

- Global modules
- Object literal notation
- The Module pattern
- AMD modules
- CommonJS modules
- ECMAScript Modules (ESM)

## JavaScript and global scope syndrome

JavaScript is _"affected by the global scope syndrome:"_ everything belongs to the `window` or `global` object that means global scope. To organize the software program then we need to:

- partition this space/object;
- keep some stuff secret;
- expose only a minimal list of elements;
- avoid names clashing.

### First attempt: the script tag

Before ES6 (not buit-in modules, no `let`, no `const`) the only way to separate a program in small pieces was using the `<script>` tag to load via a HTML files pieces of code into the _global space_. Each file was a module.

 A common JavaScript pattern to create a module was to make use of [IIEF](https://benalman.com/news/2010/11/immediately-invoked-function-expression) (pronounced _iffy_), _Immediately Invoking Function Expression_.

>In JavaScript, every function, when invoked, creates a new execution context. Because variables and functions defined within a function may only be accessed inside, but not outside, that context, invoking a function provides a very easy way to create privacy.

<cite>Ben Alman</cite> - Immediately-Invoked Function Expression (IIFE), 2010
{: .small}

 An IIEF is then a function that wraps a _code fragment_ and it is invoked in the last line.

#### Example

The application `app.js` depends on `module-two.js` then, in turn, depends on `module-one.js`:

```html
<html>
  <head>
    <script src="module-one.js"></script>
    <script src="module-two.js"></script>
    <script src="app.js"></script>
  </head>
  <body>...</body>
</html>
```

```javascript
var moduleOne = (function(x, y) {
    // Body of the module.
    function sum(x, y) {
      return x + y;
    }
  
    function diff(x, y) {
      return x - y;
    }
  
    // Export assigning function to the global variable moduleOne.
    return {
      sum: sum,
      diff: diff
    };
  })();
```

An IIFE is a _function expression_ that immediately gets invoked in the last line `();` and creates a _new scope (function-scoped)_ along with a _private space_. The interface is assigned to the _global variable_ `moduleOne`, the _namespace_, usable by all. The namespace allows then to invoke the public API and to hide the logic of the module.

__Pro Tip.__ Think about the first paranthesis of the IIFE `(` as a shield to protect the logic, and the _anonymous object literal_ `return` as the interface _export_.
{: .notice--info}

The second module defined as IIEF get `moduleOne` as parameters since it is global and re-export some functions.

```javascript
var moduleTwo = (function(x, y) {
  // Body of the module.
  function prod(x, y) {
    return x * y;
  }

  function quot(x, y) {
    return x / y;
  }

  // Export and re-export assigning function to the global variable moduleTwo.
  return {
    sum: moduleOne.sum,
    diff: moduleOne.diff,
    prod: prod,
    quot: quot
  };
})(moduleOne);
```

__Pro Tip.__ The `var moduleTwo` implicitly _imports_ the visibile elements from `moduleOne` and make them available in the global scope. The module argument mitigate the indirect dependency import issue.
{: .notice--info}

Finally the application use the functionalities from the module.

```javascript
console.log("Loading application...");

// Import all the arithmetic operations, exported and re-exported.
var ops = moduleTwo;
console.log("Sum of 1 and 2 is: ", ops.sum(1,2));
console.log("Difference between 6 and 2 is: ", ops.diff(6,2));
console.log("Multiplication of 4 by 2 is: ", ops.prod(4,2));
console.log("Division of 9 by 3 is: ", ops.quot(9,3));
```

#### Drawbacks

This approach has some drawbacks:

- _import/export functionality_ relies on _global variables_ and this increase the possibility of name clashes;
- a script does not _explicitly load_ the dependencies it depends on, so the developer has to list in the _proper order_ all the direct and indirect dependencies;
- scripts are _synchronous_.

### Second attempt: the Object Literal pattern

The object literal notation is a list of _name/value pairs_ enclosed by curly brackets. It is way to create an object and assign it some properties like: functions, objects, array etc. The property names are the _interface elements_ exposed by the module. The object is the module and viceversa.

Rebecca Murphey has written an in depth [post](http://rmurphey.com/blog/2009/10/15/using-objects-to-organize-your-code)

#### Example: object literal notation

The `moduleOne` is just an object that exposes some functionality hiding the implementation details.

```javascript
var moduleOne = {
  // Body of the module.
  sum: function sum(x, y) {
    return x + y;
  },

  diff: function diff(x, y) {
    return x - y;
  }
};

moduleOne.sum(1, 2);
moduleOne.diff(4, 3);
```

https://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript

### Revealing Module Pattern (RMP)

The [Revealing Module Pattern][todd-motto] is so called since it _"reveals the public pointers to methods inside the Module's scope"_.

This pattern has been designed to emulate the concept of a class as put in evidence by [Addy Osmani](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript)

http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html


## TypeScript namespace

TypeScript defines the concept of [`namespace`](https://www.typescriptlang.org/docs/handbook/namespaces.html) to make the definition of modules simpler. For instance

```javascript
namespace module {
    var cat = {
        name: "crazy cat",
        meow: "meoooooooow"
    };

    function _private() {
        // do some stuff privately
    }

    export function name() {
        return cat.name;
    }

    export function meow() {
        return cat.meow;
    }
}
```

that is traslated in ES5 code level as:

```javascript
"use strict";
var module;
(function (module) {
    var cat = {
        name: "crazy cat",
        meow: "meoooooooow"
    };
    function _private() {
        // do some stuff privately
    }
    function name() {
        return cat.name;
    }
    module.name = name;
    function meow() {
        return cat.meow;
    }
    module.meow = meow;
})(module || (module = {}));
```

Open a browser debugger and copy/paste the code in the console, invoke the public function:

```javascript
> module.meow()
> "meoooooooow"
```

### More complex namespace

We can easily define more complex namespaces such as:

```javascript
namespace module.cats {
    var cat = {
        name: "crazy cat",
        meow: "meoooooooow"
    };

    function _private() {
        // do some stuff privately
    }

    export function name() {
        return cat.name;
    }

    export function meow() {
        return cat.meow;
    }
}

namespace module.dogs {
    var dog = {
        name: "crazy dog",
        bau: "bauuuuuuuuu"
    };

    function _private() {
        // do some stuff privately
    }

    export function name() {
        return dog.name;
    }

    export function bau() {
        return dog.bau;
    }
}
```

and it is translated in ES5 code level into:

```javascript
"use strict";
var module;
(function (module) {
    var cats;
    (function (cats) {
        var cat = {
            name: "crazy cat",
            meow: "meoooooooow"
        };
        function _private() {
            // do some stuff privately
        }
        function name() {
            return cat.name;
        }
        cats.name = name;
        function meow() {
            return cat.meow;
        }
        cats.meow = meow;
    })(cats = module.cats || (module.cats = {}));
})(module || (module = {}));
(function (module) {
    var dogs;
    (function (dogs) {
        var dog = {
            name: "crazy dog",
            bau: "bauuuuuuuuu"
        };
        function _private() {
            // do some stuff privately
        }
        function name() {
            return dog.name;
        }
        dogs.name = name;
        function bau() {
            return dog.bau;
        }
        dogs.bau = bau;
    })(dogs = module.dogs || (module.dogs = {}));
})(module || (module = {}));
```

As before, in the browser debugger console copy/paste the ES5 code and invoke the public functions as:

```javascript
> module.cats.meow()
> "meoooooooow"
> module.dogs.bau()
> "bauuuuuuuuu"
```

### EcmaScript 6

In ES6 scripts cannot import module declaratively, the _programmatic module loader API_ must be use exploiting a new variant of `<script>`:

```javascript
<script type="module">
  // Implicitly in strict mode.
  import * from 'lib/moduleOne';

  // Some variable local to the module scope,
  // not a property of the global object such as window in browsers.
  var greeting = "hello";
  ...
</script>
```

This new flavour of `<script>` brings module support to old browsers via a polyfill.

## Custom module systems (client and server side)

Due to the lacking of a built-in module system in ES6 previous versions, two custom ones arised:

- CommonsJS to target the _server side_ (node.js)
- AMD (Asynchronous Module Defintion) _format_ to target the _client side_.

### CommonsJS modules

Let's rewrite the previous example using CommonJS:

```javascript
// Import elements.
var importedFunctionOne = require('./module-one.js').importedFunctionOne;
var importedFunctionTwo = require('./module-two.js').importedFunctionTwo;

// Body.
function internalFunction() {
  // do something
}

// Re-export and expose an internal function.
function interfaceFunction() {
  importedFunctionOne();
  internalFunction();
}

// Export the interface elements.
module.exports = {
  interfaceFunction: interfaceFunction
};
```

Pretty much the same except that:

- it does not rely on any global variable
- a module system will load and execute _synchronously_ the code wrapped by the modules
- each module imports its own dependencies that are loaded in a chain, no more explicit loading.

In CommonsJS every file is a module, each module has an _implicit_ local scope. CommonsJS modules import dependencies dynamically via _asynchronous_ function calls.

The `module.export` allows to expose everything as an API: basic types, objects, functions and arrays.

__Pro Tip.__ Explore the content of the global variable `module` to see how the module system manages the modules using `console.log(module)` from the main file that imports the module.
{: .notice--info}

### AMD module format

RequireJS is the most known implementation of the AMD format, created to explicitly target the code executed in browsers.

We rewrite the previous example as an AMD module:

```javascript
define(['./module-one.js', './module-two.js'], function(moduleOne, moduleTwo){
  var importedFunctionOne = moduleOne.importedFunctionOne;
  var importedFunctionTwo = moduleOne.importedFunctionTwo;

  // Body.
function internalFunction() {
  // do something
}

// Re-export and expose an internal function.
function interfaceFunction() {
  importedFunctionOne();
  internalFunction();
}

return {
  interfaceFunction: interfaceFunction
}
});
```

AMD modules are loaded _asynchronously_ since the browser cannot stop the execution to wait the downloading of a module and, moreover, multiple module can be loaded in parallel.

### Yet on modules

All of the implementations are quite similar:

- a module wraps some code and it is contained in a single file
- a module defines a local scope, an import and export sections:
  - local scope: everything defined into a module is _not global_
  - import: module elements are imported
  - export: explicitly exports what a module _exposes_
- modules are _singletons_, it does not matter how many time a module is imported, only one instance exists
- a module system does not relies anymore on _global variables_.

## Native ECMASscript 6 Modules (ESM)

ES6 module or ESM take the best from the two worlds, CommonJs and AMD, and they are characterized by:

- cyclic import in a transparent way;
- _static_ module structure so that static analysis and code optimisation can be performed before the body is executed;
- module must be at _the top level only_, cannot be conditionally loaded as with CommonJS;
- both _synchronous_ (server) and _asynchronous_ (browser) loading.

We finally rewrite the example using ES6 modules:

```javascript
import {importedFunctionOne} from './moduleOne.js';
import {importedFunctionTwo} from './moduleTwo.js';

function internalFunction() {
  // do something
}

export function interfaceFunction() {
  importedFunctionOne();
  internalFunction();
}
```

So the syntax is more readable and compact: what to `import` and what to `export` is cristal clear. The loader API is still in porgress.

In ES6 `strict` mode is turned on [automatically](nicolas) so silent errors become thrown exceptions and bad parts of the language are not allowed.

In ES6 modules are files that expose and API via the `export` keyword. Variable declarations are local to a module as in CommonJS, they cannot be access from outside unless explicitly exported.

## Resources

browser support
https://caniuse.com/#feat=es6-module

https://github.com/whatwg/loader/
https://whatwg.github.io/loader/
https://github.com/ModuleLoader/es-module-loader/blob/master/docs/system-register.md
https://github.com/ModuleLoader/es-module-loader
https://github.com/google/systemjs

http://calculist.org/blog/2012/06/29/static-module-resolution/
https://v8.dev/features/modules

old
https://exploringjs.com/es6/ch_modules.html
https://www.sitepoint.com/using-es-modules/
https://github.com/tc39/

---
[todd-motto]: https://ultimatecourses.com/blog/mastering-the-module-pattern
[nicolas]:https://ponyfoo.com/articles/es6-modules-in-depth
[binding]:https://2ality.com/2015/07/es6-module-exports.html
[patterns-book]:https://addyosmani.com/resources/essentialjsdesignpatterns/book/