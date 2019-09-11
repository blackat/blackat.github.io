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

## Why modules?

Modular programming is a software design technique to facilitate the construction of large software programs by _decoupling_ it into small _loose coupled_ modules.

A __module__ is an independent and autonomous unit that provides a _functionality_. In order to exploit the functionality the module exposes an interface to simplify the usage and hide the implementation details.

Key concepts of module are:

- separation of concerns: different from monolithic design, separation in units w.r.t. the functionality
- hidden implementation details: interact via the _elements exposed in the interface_, not tight to a specific implementation
- interaction through well-defined interfaces: easier to exploit a functionality, _abstract_ from implementation details
- reusability: being an autonomous and independent unit of work can be used elsewhere
- independent and interchangeable units: can be tested in isolation, easier to spot issues, can be substituted if required.

Concepts like `module`, `package`, `component` and `assembly` vary a lot among lanaguages.

/// todo: previous of ESM there were some module variations or object [literal pattern](http://rmurphey.com/blog/2009/10/15/using-objects-to-organize-your-code)


### Export and import

__Export__ and __import__ are really the most important operation when we talk about module:

`export`: a module implement a functionality and _export_ and interface to use it;
`import`: who wants to use the functionality has to _import_ the _elements_ defined in the interface.

## JavaScript <script>

In the past the only way to separate a program in small pieces was using `<script>` to load via HTML file a piece of code into the _global space_.

```html
<script src="module-one.js"></script>
<script src="module-two.js"></script>
<script src="app.js"></script>
```

A common JavaScript pattern, pre ES6 (not buit-in modules, no `let`, no `const`), to create a module is the use of __IIFE (Immediately Invoking Function Expression)__, a function tra wraps a _code fragment_ and it is invoked in the last line.

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

Basically an IIEF (Ben Alman) is the way of wrapping a code fragment, it is automatically invoked in the last line `();` and assigned to the _global variable_ `moduleTwo`.

IIEF is also a way of _creating a new scope (function-scoped)_ to decide what to hide and what to show but also to _reduce name clashing_ among functions. This approach has some __drawbacks__:

- import/export functionality relies on global variables and this increase the possibility of name clashes;
- a script does not explicitly load the dependencies it depends on, so the developer has to list in the proper order all the direct and indirect dependencies
- scripts are _synchronous_.

Scripts cannot import module declaratively, the programmatic module loader API must be use exploiting a new variant of `<script>`:

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

Due to the lacking of a built-in module system, two custom ones arised:

- CommonsJS to target the server side (node.js)
- AMD (Asynchronous Module Defintion) _format_ to target the client side.

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

### ECMASscript 6 modules

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