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

## Why modules

__Modular programming__ is a software design technique to facilitate the construction of large software programs by _decoupling_ it into small _loose coupled_ modules.

A __module__ is an independent and autonomous unit that provides a _functionality_. In order to exploit the functionality the module exposes an interface to simplify the usage and hide the implementation details.

Key concepts of module are:

- __separation of concerns:__ different from monolithic design, separation in units w.r.t. the functionality;
- __hide implementation details:__ interact via the _elements exposed in the interface_, not tight to a specific implementation;
- __interaction through well-defined interfaces:__ easier to exploit a functionality, _abstract_ from implementation details;
- __reusability:__ being an autonomous and independent unit of work can be used elsewhere, a [simple API encourages reusability](https://ponyfoo.com/books/practical-modern-javascript#toc);
- __independent and interchangeable units:__ can be tested in isolation, easier to spot issues, can be substituted if required.

Concepts like `module`, `package`, `component` and `assembly` vary a lot among lanaguages.

### Export and import

__Export__ and __import__ are really the most important operations when we talk about module:

- `export`: a module implement a functionality and _export_ and interface to use it;
- `import`: who wants to use the functionality has to _import_ the _elements_ defined in the interface.

### JavaScript and global scope

JavaScript is _"affected by the global scope syndrome"_, everything belongs to the `global` scope. Arise then the need to:

- partition this space;
- keep some stuff secret;
- to expose only a minimal list of elements;
- to avoid names clashing.

/// todo: previous of ESM there were some module variations or object [literal pattern](http://rmurphey.com/blog/2009/10/15/using-objects-to-organize-your-code)

## JavaScript <script>

In the past the only way to separate a program in small pieces was using `<script>` to load via a HTML file a piece of code into the _global space_. For instance we load the application `app.js` thta depends on `module-two.js` tha, in turn, depends on `module-one.js`:

```html
<script src="module-one.js"></script>
<script src="module-two.js"></script>
<script src="app.js"></script>
```

A common JavaScript pattern, pre ES6 (not buit-in modules, no `let`, no `const`), to create a module is the use of __[IIEF](https://benalman.com/news/2010/11/immediately-invoked-function-expression) (Immediately Invoking Function Expression - Ben Alman)__, a function tath wraps a _code fragment_ and it is invoked in the last line.

```javascript
var moduleTwo = (function () {

  // Import interface elements using global variable.
  var importedFunctionOne = moduleOne.importedFunctionOne;

  // Body of the module.
  function internalFunction() {
    // do something
  }

  // Re-export a function from another module and expose a local function.
  function interfaceFunction() {
    importedFunctionOne();
    internalFunction();
  }

  // Export assigning function to the global variable moduleTwo.
  return {
    interfaceFunction: interfaceFunction
  };

})();
```

An IIFE is a _function definition_ that immediately calls itself in the last line `();` and creates a _new scope (function-scoped)_ along with a _private space_. The interface is assigned to the _global variable_ `moduleTwo`, the _namespace_, usable by all. The namespace to invoke the public API and the private space to hide the logic of the module.

__Pro Tip.__ Think about the first paranthesis of the IIFE `(` as a shield to protect the logic, and the _anonymous object literal_ `return` as the interface _export_. The `var moduleTwo` implicitly _imports_ the visibile elements make them available in the global scope.
{: .notice--info}

### Drawbacks

This approach has some drawbacks:

- import/export functionality relies on _global variables_ and this increase the possibility of name clashes;
- a script does not _explicitly load_ the dependencies it depends on, so the developer has to list in the proper order all the direct and indirect dependencies;
- scripts are _synchronous_.

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

### Revealing Module Pattern (RMP)

This pattern just explained is called [Revealing Module Pattern][todd-motto] since it _"reveals the public pointers to methods inside the Module's scope"_.

## TypeSscript namespace

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

__Pro Tip.__ Explore the contante of the global variable `module` to see how the module system manages the modules using `console.log(module)` from the main file that imports the module.
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
  - export: explicitly exports what a module should expose
- modules are _singletons_, it does not matter how many time a module is imported, only one instance exists
- a module system does not relies anymore on _global variables_.

### Native ECMASscript 6 Modules (ESM)

ES6 module or ESM take the best from the two worlds, CommonJs and AMD, and they are characterized by:

- cyclic import in a transparent way
- _static_ module structure so that static checking and code optimisation can be performed before the body is executed
  - module must be at the top level, cannot be conditionally loaded as with CommonJS
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