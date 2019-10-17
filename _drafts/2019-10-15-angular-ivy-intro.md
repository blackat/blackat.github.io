---
layout: single
title: "Angular Ivy Primer"
date: '2019-10-15'
last_modified_at: '2019-10-15'
comments: true
categories:
  - angular, ivy
tags:
  - [angular, ivy]
toc: true
toc_label: "On this page"
toc_icon: "list"
toc_sticky: true
---

Different rendering strategies are used by commercial frameworks to improve the rendering perfomance of a web application. 

Each interface component can dynamically change at runtime, for instance there is a new item in a list, the DOM has to be consequently updated, the view or user interface has to be re-painted.

The DOM update and view re-painting are costly operations. Angular, React and Vue try to improve the update/rendering cost via two different sofisticated strategies:

- __Virtual DOM:__ when a component changes a new VDOM is created, the two VDOM are compared via _diffing_ operation, optimized batch update is applied to the DOM.
- __Incremental DOM:__ each HTML component part is transformed in an optimal and efficient JavaScript piece of code that, when triggered because of a change, update it-self.

Angular Ivy is the new Angular _renderer_ that uses _incremental DOM_.

## Lingo

- _Object Model (OM):_ a way to model via object-oriented techniques (objects, classes, interfaces, properties, inheritance, encapsulation, etc) a system for development purpose: i.e. Apache POI implements a OM of Microsoft Excel that manipulates via a Java API.
- _Data Model (DM):_ it represents entities at database level, it deals with table schema, relationships between tables (FKs, PKs) but not advanced object-oriented concepts like inheritance or polymorphism. DM represents how OM classes are stored in a database.
- _DOM:_ an object oriented representation of a HTML document in a tree fashion that can be manipulated via the DOM API: i.e. `HTMLButtonElement` is one of the DOM interfaces.
- _Virtual DOM:_ the virtual representation of the real DOM.
- _Shadow DOM:_ it allows to separate DOM into smaller and encapsulated object-oriented representation of a HTML element.
- _Tree and nodes:_ the DOM is organized in a logical tree where its nodes are the components or HTML elements.
- _Rendering/painting:_ the browser process that transform the DOM into the UI.
- _Diffing:_ operation that compare two virtual DOM.
- _Incremental DOM:_

## Browser DOM

> The Document Object Model (DOM) connects web pages to scripts or programming languages by representing the structure of a document such as the HTML representing a web page in memory.

<cite>MDN</cite> - Mozilla Developer Network
{: .small}

__Pro Tip__
The HTML document is represented in object-oriented fashion, as objects in a logical tree, by the DOM that also provides the API to manipulate those objects.
{: .notice--info}

// todo: improve the description

The DOM rendered gives the HTML page visible to the end user.

https://bitsofco.de/understanding-the-critical-rendering-path/
https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model

### DOM rendering is slow

Being the DOM represented as a tree, it makes easier to change and update the tree. What the user see is the result of the DOM rendering that is the _slow part_. More a component is complex more take time to be rendered.

Moreover, a page is usually made of many components, complex and non-complex, every time one of them changes the all page needs to be re-rendered, a really expensive operation.

__Pro Tip__
Frequent DOM manipulations make the user interface slow since the re-painting of the user interface is the most expensive part. It is something that has not been expected when it has been designed. For instance changing visibility of an element forces the browser to verify/check visibility of all other dom nodes.
{: .notice--info}

Actions like changing visibility or the background of an element would trigger a repaint.

// todo: add also the diagram from Chrome devTools to show the rendering events in the UI.

https://bitsofco.de/what-exactly-is-the-dom/
https://programmingwithmosh.com/react/react-virtual-dom-explained/

## Virtual DOM

The key idea is to _render the DOM the least possible_. Instead of updating the real DOM, the frameworks like React updates a virtual DOM.

The virtual DOM is a _tree_ as well made of _nodes_ that are the page elements. When a new element is added a new virtual DOM is created, the _difference_ between the two trees is calculated.

Atransformations serie is calculated to update the browser DOM so that it matches the new virtual DOM. This transformations is both the minimal operations to be applied to the real DOM and the ones that reduce the perfomance cost of the DOM update.

__Note__ The rendering process happens only on a _difference_, the _batch changes_ to be applied are optimized to improve the performance cost.
{: .notice--info}

### What virtual DOM looks like

The virtual DOM is something _not official_, no specification is provided _differently_ from DOM and shadow DOM.

It is a copy of the original DOM as a _plain JavaScript object_ so that it can be modified how many times we want without affecting the real DOM. The virtual DOM can be divided in chunks so that it is easier to _diffing_ the changes.

#### Example

When a new item is added to an unordered list of elements a copy of the virtual DOM containing the new element is created.

The _diffing_ process collects the differences between the two virtual DOM so changes can be transformed in a batch update against the real DOM.

__Pro Tip__
Not distinction about _reflow_ (element layout: position recalculation and geometry) and _repaint_ (element visibility) has been done so far since most of the considered actions involve the repaint operation.
{: .notice--info}

// todo: how the browser render process happens? at each single change or collect a bunch of changes?

https://bitsofco.de/understanding-the-virtual-dom/

## How React uses the Virtual DOM

In React a user interface is made of a set of components, each component has a _state_. For instance the state of a drop-down menu is the array of the available elements and the current selected one.

Via the [observer pattern](https://www.baeldung.com/java-observer-pattern) React listens to _state change_ to update the virual DOM. The _diffing_ process makes React aware of which virtual DOM objects have changed, _only_ those objects will be updated in the real DOM.

__Pro Tip__
As developer you don't need to be aware about how DOM manipulation happens at each state change. React does the job optimizing the performance cost behind the scenes.
{: .notice--info}

Another way used by React to reduce the re-painting cost is the _batch update:_ updates are sent in batch and _not_ at every single state change.

The _great advantage_ of using the virtual DOM is that we don't need any compiler. JSX for instance, is really close to JavaScript, the key point is instead the render function that can be implemented using any programming language.

### Virtual DOM drawbacks

- The virtual DOM required an _interpreter_ to interpret the component and, at compile time we don't know what we need, so the whole stuff has to be loaded by the browser.
- e

The incremental DOM, as we will see, is based on instructions referenced by the component (factories). If, for some reason, an instruction is not referenced it won't be bundled with the application.

// todo: improve the description, make it more clear.

## Incremental DOM

The key idea of the incremental DOM is:

>Every component gets compiled into a series of instructions. These instructions create DOM trees and update them in-place when the data changes.

Let's see what the Angular compiler produces and how Angular incremental DOM technique differs from virtual DOM.

## Angular template compiler

An Angular application is mainly made of by components organized in a tree fashion. Each component is implemented to accomplish a certain mission, for instance the navigation bar, the drop-down menu, etc.

### Simple component defintion

A component is mainly HTML, the presentation, and some code, the logic, the hearth. Code is written in TypeScript and template in HTML. Consider a simple component, it is made of _"code and template"_.

#### Code

TypeScript code defines the component, both the logic and the structure or organization such as HTML, CSS, which selector, etc.

```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'a simple component';
}
```

#### HTML template

Bunch of HTML code with binding variable to present, with a certain look and feel, some content.

```html
<div style="text-align:center">
  <h1>
    Welcome to {{ title }}!
  </h1>
</div>
```

In this simple example there is just a big title in the center of a blank page: _Welcome to a simple component_.

### Browser cannot use it

The browser is the _execution enviroment_ for an Angular application, so it has to load the application and execute it, but unfortuntely it cannot load and execute an Angular component _as it is_.

__ProTip.__ A browser can interpret JavaScript and render HTML but not in the Angular way.
Angular provides a compiler that, along with the TypeScript one, transforms _"everything in something else"_ that a browser can understand.
{: .notice--info}

During the build of an Angular project, two compilers come into play with _different purposes_:

- `tsc` is the TypeScript compiler and generates the JavaScript w.r.t. the target specifief in the `tsconfig.json`, for instance `"target": "es2015",`.
- `ngc` is the Angular template compiler that translate the component templates into _inline_ JavaScript. The Angular compiler can works in two different modes:
  - Ahead-of-Time (AoT): work at build time so that the templates are bundled along with the application at build time. Suitable for production.
  - Just-in-Time (JiT): templates are not pre-compiled, the compiler comes along with the application, is loaded by the browser and does the work at runtime. Suitable for development.

__ProTip.__ During the development phase `ng serve` provides _live reload_ functionality, change your code and get the page automatically reloaded.
The process goes through `@ngtools/webpack`, compiled code is not saved to disk, nothing it is, everything is consumed in memory via streams and emitters.
{: .notice--info}

### Who does what

What are then the roles of the browser and Angular?

Once the Angular application has been fully tranformed into JavaScript, HTML templates included, WebPack bundles it in order to boost even more the loading perfomance.
The browser loads the `index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    ...
  </head>
  <body>
    <app-root></app-root>

    <script src="runtime-es2015.js" type="module"></script>
    <script src="polyfills-es2015.js" type="module"></script>
    <script src="styles-es2015.js" type="module"></script>
    <script src="vendor-es2015.js" type="module"></script>
    <script src="main-es2015.js" type="module"></script>

    <script src="runtime-es5.js" nomodule defer></script>
    <script src="polyfills-es5.js" nomodule defer></script>
    <script src="styles-es5.js" nomodule defer></script>
    <script src="vendor-es5.js" nomodule defer></script>
    <script src="main-es5.js" nomodule defer></script>
  </body>
</html>
```

Firs thing the scripts can be loaded both by modern browsers that supports ESM modules and by old ones that does not support modules via `nomodule defer`.

The `main-es2015.js` contains the full bundled applications while `vendor-es2015.js` is mainly Angular framework. The _browser role_ is to load and execute the JavaScript code. The starting point is Angular module transformed that try to render the `app-root` element executing the factories inside the file `main-es2015.js`.

__ProTip.__ The transformed HTML template into JavaScript becomes a kind of series of instructions for Angular to know how to rendere the page, how to build components. An element in the `js` file uses Angular function to create itself into the DOM and to update itself when something changes. Highly performant functions that, when called, update the DOM. This happens thanks to _bridges_ between the element, or node, and the real HTML element in the DOM.
{: .notice--info}
  
>The Angular AOT compiler extracts metadata to interpret the parts of the application that Angular is supposed to manage.

Basically it manages metadata interpretation and template compilation that can be controlled specifying some template compiler options in the `tsconfig.json`.

## Compile with ngc

We want to simply invoke the compiler to inspect what it produces without any Webpack interference, so open the `package.json` file and add the following option:

```javascript
"scripts": {
  ...
  "compile": "ngc"
},
```

then in the `tsconfig.json` enable the `d.ts` files generation, important to have the TypeScript definitions of the factories:

```javascript
"compilerOptions": {
  ...
  "declaration": true,
  ...
}
```

Run `npm run compile` and look into the folder `dist/out-tsc/src/app` where everything has been transformed into JavaScript and saved to disk.

### Minimal component

Focus on the template, remember the performance optimization is related to the DOM update and user interface re-painting.

```html
<h1>Welcome to {{ title }}!</h1>
```

Consider the following files:

```bash
app.module.js               // class where metadata (decorators) have been interpreted
app.module.ngfactory.js     // 
app.component.js
app.component.ngfactory.js
```

In the `app.module.ngfactory.js` the important thing is the boostrap of the application specifying the _root component:_

```javascript
import * as i0 from "@angular/core";
import * as i1 from "./app.module";
import * as i2 from "./app.component";
import * as i3 from "./app.component.ngfactory";
...
var AppModuleNgFactory = i0.ɵcmf(i1.AppModule, [i2.AppComponent], function(_l) {...}
...
```

Factories are produced by the Angular compiler and basically are functions that build a module, a component, etc.

```javascript
export declare function ɵcmf(ngModuleType: Type<any>, bootstrapComponents: Type<any>[], defFactory: NgModuleDefinitionFactory): NgModuleFactory<any>;
```

Open `app.component.ngfactory.js`, start from the component factory function:

```javascript
import * as i1 from "@angular/core";
import * as i2 from "./app.component";
...
var AppComponentNgFactory = i1.ɵccf(
  "app-root",
  i2.AppComponent, /* class or type */
  View_AppComponent_Host_0, /* factory that produces the app-root, component host, the node defintion */
  {},
  {},
  []
);
```

In a very simple way, the function takes type or class of the component and the factory function that produces the view definition related to the `app-root` selector.
In turn the host node invoke the component factory to create the component defintion, `<h1>`, to be attached to the host.

To better clarify some functions, open their definitions availbale inside the Angular `core.d.ts` definition file:

```javascript
export declare function ɵccf(
  selector: string,
  componentType: Type<any>,
  viewDefFactory: ViewDefinitionFactory,
  inputs: {
    [propName: string]: string;
  } | null,
  outputs: {
    [propName: string]: string;
  },
  ngContentSelectors: string[]
): ComponentFactory<any>;

export declare function ɵeld(
  checkIndex: number,
  flags: ɵNodeFlags,
  matchedQueriesDsl: null | [string | number, ɵQueryValueType][],
  ngContentIndex: null | number,
  childCount: number,
  namespaceAndName: string | null,
  fixedAttrs?: null | [string, string][],
  bindings?: null | [ɵBindingFlags, string, string | SecurityContext | null][],
  outputs?: null | ([string, string])[],
  handleEvent?: null | ElementHandleEventFn,
  componentView?: null | ViewDefinitionFactory,
  componentRendererType?: RendererType2 | null
): NodeDef;

/**
 * A node definition in the view.
 */
declare interface NodeDef {
  flags: ɵNodeFlags;
  nodeIndex: number;
  checkIndex: number;
  parent: NodeDef | null;
  renderParent: NodeDef | null;
  /** this is checked against NgContentDef.index to find matched nodes */
  ngContentIndex: number | null;
  /** number of transitive children */
  childCount: number;
  /** aggregated NodeFlags for all transitive children (does not include self) **/
  childFlags: ɵNodeFlags;
  /** aggregated NodeFlags for all direct children (does not include self) **/
  directChildFlags: ɵNodeFlags;
  bindingIndex: number;
  bindings: BindingDef[];
  bindingFlags: ɵBindingFlags;
  outputIndex: number;
  ...
}
```

### Build and debug

Now build the application and start a live server to debug it into the browser:

```bash
ng build --aot
npx http-server dist/test-ivy
```

__ProTip.__ Activate the Ahead-of-Time compilation to have everything in JavaScript and saved to disk, we would like to inspect the generated code.
{: .notice--info}

#### Bootstrap

The bootstrap phase starts in the file `vendor-es2015.js`:

```javascript
class PlatformRef {
    ...    
    /**
     * Creates an instance of an `\@NgModule` for the given platform
     * for offline compilation.
     */
    bootstrapModuleFactory(moduleFactory, options) {
        // Note: We need to create the NgZone _before_ we instantiate the module,
        ...
        return ngZone.run((..));
    }
```

going up in the callstack:

```javascript
/**
 * A reference to an Angular application running on a page.
 */
class ApplicationRef {
  /**
   * Bootstrap a new component at the root level of the application.
   */
  bootstrap(componentOrFactory, rootSelectorOrNode) {
    ...
    // create the app-root node
    const compRef = componentFactory.create(Injector.NULL, [], selectorOrNode, ngModule);
    ...
  }
}
```

In the Angular `component_factory.ts` [source file](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/core/src/linker/component_factory.ts#L79) we have the base class method to create a component of a certain type:

```javascript
/**
 * Base class for a factory that can create a component dynamically.
 * Instantiate a factory for a given type of component with `resolveComponentFactory()`.
 * Use the resulting `ComponentFactory.create()` method to create a component of that type.
 *
 * @see [Dynamic Components](guide/dynamic-component-loader)
 */
export abstract class ComponentFactory<C> {
  ...
  /**
   * Creates a new component.
   */
  abstract create(
      injector: Injector, projectableNodes?: any[][], rootSelectorOrNode?: string|any,
      ngModule?: NgModuleRef<any>): ComponentRef<C>;
}
```

and its implementation:

```javascript
class ComponentFactory_ extends ComponentFactory<any> {
  
  viewDefFactory: ViewDefinitionFactory;

  /**
   * Creates a new component.
   */
  create(
      injector: Injector, projectableNodes?: any[][], rootSelectorOrNode?: string|any,
      ngModule?: NgModuleRef<any>): ComponentRef<any> {
    if (!ngModule) {
      throw new Error('ngModule should be provided');
    }
    const viewDef = resolveDefinition(this.viewDefFactory);
    const componentNodeIndex = viewDef.nodes[0].element !.componentProvider !.nodeIndex;
    const view = Services.createRootView(
        injector, projectableNodes || [], rootSelectorOrNode, viewDef, ngModule, EMPTY_CONTEXT);
    const component = asProviderData(view, componentNodeIndex).instance;
    if (rootSelectorOrNode) {
      view.renderer.setAttribute(asElementData(view, 0).renderElement, 'ng-version', VERSION.full);
    }

    return new ComponentRef_(view, new ViewRef_(view), component);
  }
}
```

// screenshot

```javascript
export interface Services {
  ...
  createRootView(
      injector: Injector, projectableNodes: any[][], rootSelectorOrNode: string|any,
      def: ViewDefinition, ngModule: NgModuleRef<any>, context?: any): ViewData;
```

there is an object literal that link the following function:

```javascript
function createProdRootView(
    elInjector: Injector, projectableNodes: any[][], rootSelectorOrNode: string | any,
    def: ViewDefinition, ngModule: NgModuleRef<any>, context?: any): ViewData {
  const rendererFactory: RendererFactory2 = ngModule.injector.get(RendererFactory2);
  return createRootView(
      createRootData(elInjector, ngModule, rendererFactory, projectableNodes, rootSelectorOrNode),
      def, context);
}
```

then jumping to the [file](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/core/src/view/view.ts#L250) `core/src/view/view.ts` the function `function createViewNodes(view: ViewData){...}` will creates the nodes of a certain view.

This function uses some methods from `element.ts` such as `function createElement(view: ViewData, renderHost: any, def: NodeDef): ElementData` in the [file](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/core/src/view/element.ts#L151) `core/src/view/element.ts` that will use the injected rendered to definitely create the element.

#### The renderer

What then about the renderer? What is that? A class that extends and implements `abstract class Renderer2` and [render a template into DOM](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/core/src/render/api.ts#L108). A custom renderer can be implemented to render a template, for instance, into XML or another format.

__ProTip.__ An HTML template is transformed in an _intermediate_ format or Object Model, precisily in JavaScript for the Angular case, by the template compiler.
Factory functions are automatically generated by the compiler and they are able to produce an object that can create a component or a node or a module. Then a renderer, specified by the chosen platform will produce DOM elements (in case of a DOM renderer) from the JavaScript objects.
{: .notice--info}

Condider the `DefaultDomRenderer2` [class](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/platform-browser/src/dom/dom_renderer.ts#L115) it is the one that invokes `document` interface methods to create the real DOM element or access the DOM tree.

```javascript
createElement(name: string, namespace?: string): any {
    if (namespace) {
      // In cases where Ivy (not ViewEngine) is giving us the actual namespace, the look up by key
      // will result in undefined, so we just return the namespace here.
      return document.createElementNS(NAMESPACE_URIS[namespace] || namespace, name);
    }

    return document.createElement(name);
  }
```

#### Types

What are now the produced types?



- ViewData

```javascript
export interface ViewData {
  def: ViewDefinition;
  root: RootData;
  renderer: Renderer2;
  // index of component provider / anchor.
  parentNodeDef: NodeDef|null;
  parent: ViewData|null;
  viewContainerParent: ViewData|null;
  component: any;
  context: any;
  // Attention: Never loop over this, as this will
  // create a polymorphic usage site.
  // Instead: Always loop over ViewDefinition.nodes,
  // and call the right accessor (e.g. `elementData`) based on
  // the NodeType.
  nodes: {[key: number]: NodeData};
  state: ViewState;
  oldValues: any[];
  disposables: DisposableFn[]|null;
  initIndex: number;
}
```

- ElementData
- NodeDef
- Services
- ComponentFactory_
- ComponentFactory
- Renderer2

### The essential sequence

```javascript
/**
 * The Angular platform is the entry point for Angular on a web page.
 */
PlatformRef {
  /**
   * Creates an instance of an `\@NgModule` for the given platform
   * for offline compilation.
   */
  bootstrapModuleFactory(moduleFactory, options) {
    return ngZone.run((
      ...
      this._moduleDoBootstrap(moduleRef); // moduleType: class AppModule
      return moduleRef;
      ...
  }

  /**
   * Bootstrap all the components of the module
   */
  _moduleDoBootstrap(moduleRef) {
    ...
    moduleRef._bootstrapComponents.forEach((
      f => appRef.bootstrap(f))); // bootstrap the root component AppComponent with selector app-root
    ))
  }
}
```

```javascript
class ApplicationRef {
  /**
   * Bootstrap a new component at the root level of the application.
   * When bootstrapping a new root component into an application, Angular mounts the
   * specified application component onto DOM elements identified by the componentType's
   * selector and kicks off automatic change detection to finish initializing the component.
   */
  bootstrap(componentOrFactory, rootSelectorOrNode) {
    ...
    /**
     * Use the componentFactory to create the root element app-root having this information:
     * componentType: class AppComponent
     * viewDefFactory: View_AppComponent_Host_0()
     * selector: app-root
     */
    const compRef = componentFactory.create(Injector.NULL, [], selectorOrNode, ngModule);
    ...
  }
}
```

```javascript
class ComponentFactory_ extends ComponentFactory {
  ...
  create(injector, projectableNodes, rootSelectorOrNode, ngModule) {
    const view = Services.createRootView(injector, projectableNodes || [], rootSelectorOrNode, viewDef, ngModule, EMPTY_CONTEXT);
  }
}
```

```javascript
function createRootView(root, def, context) {
  const view = createView(root, root.renderer, null, null, def);
  initView(view, context, context);
  createViewNodes(view);
  return view;
}
```

```javascript
function createViewNodes(view) {
  const nodes = view.nodes;
  for (let i = 0; i < def.nodes.length; i++) {
    switch (nodeDef.flags & 201347067 /* Types */) {
      case 1 /* TypeElement */:
        // H1 DOM element of type any, the function call the DOM renderer to 
        const el = (createElement(view, renderHost, nodeDef)));
        ...
        // View_AppComponent_0()
        const compViewDef = resolveDefinition(((nodeDef.element)).componentView)));
        ...
        break;
      case 2 /* TypeText */:
        ...
        break;
      ...
    }
  }
}
```

Finally resolve the token `title`.

### Angular component

```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'test-ivy';
}
```

```html
<div style="text-align:center">
  <h1>
    Welcome to {{ title }}!
  </h1>
</div>
<h2>Here there were some links but I have removed them :-)</h2>
```

It does nothing special except binding a variable and put some text between tags.

### Compile

Open the `package.json` file and add the following script:

```javascript
"scripts": {
  ...
  "compile": "ngc"
},
```

now run `npm run compile` and look into the folder `dist/src/app`.

The following component related files are produced:

```bash
app.component.css.shim.ngstyle.js
app.component.css.shim.ngstyle.js.map
app.component.js
app.component.js.map
app.component.metadata.json
app.component.ngfactory.js
app.component.ngfactory.js.map
app.component.ngsummary.json
app.component.spec.js
app.component.spec.js.map
app.component.spec.ngsummary.json
```

skip for the time being the styles and the test files `*.spec.*`.

File are not anymore in TypeScript but in JavaScript. Files are compiled using `tsickle`, type information are emitted and saved by the `compiler-cli` as `.metadata.json` and `ngsummary.json`.

If in the `tsconfig.json` we acitvate

```javascript
"compilerOptions": {
  ...
  "declaration": true,
  ...
}
```

we have the factory `d.ts` files as well.

All the commands that you run in the command line can be found under `node_modules/.bin/ngc` for instance.
Pay attention that when the process goes through `@ngtools/webpack` nothing is saved to disk, everything is consumed in memory via streams and emitters.

Wehn building an application (a library is different) there are two steps:

1. compile with `ngc`
2. bundle with Webpack

the compiler generate extra code such as ngfactory, ngsummary, ngstyle, etc. and then output the JavaScript code. When  the bundle process starts, Webpack will be configured to use the `ngfactory` file etc. but all the files are already in JavaScript.

__Pro Tip.__ Angular does not output anymore `.ts` file since version 5.
{: .notice-info}


Into `core.d.ts` you can find definition of some functions used in the template JavaScript trasformation, all of them are prefixed by the symbol `ɵ` like `ɵvid`. It is a way to say "ehi don't use it, for sure it will change in the future and you will be definitely screwed", it is a symbol to scare you, like the vivid colors of some venomous creatures "ehi pay attention, I am venomous!".

Now let's have a look at some files

```javascript
export declare function ɵvid(flags: ɵViewFlags, nodes: NodeDef[], updateDirectives?: null | ViewUpdateFn, updateRenderer?: null | ViewUpdateFn): ɵViewDefinition;
```

The `input` elements is translated in JavaScript like this:

```javascript
 i1.ɵeld(
        5,
        0,
        null,
        null,
        5,
        "input",
        [],
        [
          [2, "ng-untouched", null],
          [2, "ng-touched", null],
          [2, "ng-pristine", null],
          [2, "ng-dirty", null],
          [2, "ng-valid", null],
          [2, "ng-invalid", null],
          [2, "ng-pending", null]
        ],
        [
          [null, "ngModelChange"],
          [null, "input"],
          [null, "blur"],
          [null, "compositionstart"],
          [null, "compositionend"]
        ],
        function(_v, en, $event) {
          console.log(en);
          var ad = true;
          var _co = _v.component;
          if ("input" === en) {
            var pd_0 =
              i1.ɵnov(_v, 6)._handleInput($event.target.value) !== false;
            ad = pd_0 && ad;
          }
          if ("blur" === en) {
            var pd_1 = i1.ɵnov(_v, 6).onTouched() !== false;
            ad = pd_1 && ad;
          }
          if ("compositionstart" === en) {
            var pd_2 = i1.ɵnov(_v, 6)._compositionStart() !== false;
            ad = pd_2 && ad;
          }
          if ("compositionend" === en) {
            var pd_3 =
              i1.ɵnov(_v, 6)._compositionEnd($event.target.value) !== false;
            ad = pd_3 && ad;
          }
          if ("ngModelChange" === en) {
            var pd_4 = (_co.title = $event) !== false;
            ad = pd_4 && ad;
          }
          return ad;
        },
        null,
        null
      )),
```

The function instead is 

```javascript
export declare function ɵeld(checkIndex: number, flags: ɵNodeFlags, matchedQueriesDsl: null | [string | number, ɵQueryValueType][], ngContentIndex: null | number, childCount: number, namespaceAndName: string | null, fixedAttrs?: null | [string, string][], bindings?: null | [ɵBindingFlags, string, string | SecurityContext | null][], outputs?: null | ([string, string])[], handleEvent?: null | ElementHandleEventFn, componentView?: null | ViewDefinitionFactory, componentRendererType?: RendererType2 | null): NodeDef;
```

The function is then `handleEvent?: null | ElementHandleEventFn`, it has three parameters and the `if` block manages different kind of possible events. If we type something in the input field the `ngModelChange === en` will be true.

Then the two `ViewUpdateFn` are defined in this way:

```javascript
function(_ck, _v) {
      var _co = _v.component;
      var currVal_8 = _co.title;
      _ck(_v, 8, 0, currVal_8);
    },
function(_ck, _v) {
      var _co = _v.component;
      var currVal_0 = _co.title;
      _ck(_v, 2, 0, currVal_0);
      var currVal_1 = i1.ɵnov(_v, 10).ngClassUntouched;
      var currVal_2 = i1.ɵnov(_v, 10).ngClassTouched;
      var currVal_3 = i1.ɵnov(_v, 10).ngClassPristine;
      var currVal_4 = i1.ɵnov(_v, 10).ngClassDirty;
      var currVal_5 = i1.ɵnov(_v, 10).ngClassValid;
      var currVal_6 = i1.ɵnov(_v, 10).ngClassInvalid;
      var currVal_7 = i1.ɵnov(_v, 10).ngClassPending;
      _ck(
        _v,
        5,
        0,
        currVal_1,
        currVal_2,
        currVal_3,
        currVal_4,
        currVal_5,
        currVal_6,
        currVal_7
      );
    }
```

Then analyzing the other nodes from the array

```javascript
i1.ɵdid(
        6,
        16384,
        null,
        0,
        i2.DefaultValueAccessor,
        [i1.Renderer2, i1.ElementRef, [2, i2.COMPOSITION_BUFFER_MODE]],
        null,
        null
      ),
      i1.ɵprd(
        1024,
        null,
        i2.NG_VALUE_ACCESSOR,
        function(p0_0) {
          return [p0_0];
        },
        [i2.DefaultValueAccessor]
      ),
      i1.ɵdid(
        8,
        671744,
        null,
        0,
        i2.NgModel,
        [[8, null], [8, null], [8, null], [6, i2.NG_VALUE_ACCESSOR]],
        { model: [0, "model"] },
        { update: "ngModelChange" }
      ),
      i1.ɵprd(2048, null, i2.NgControl, null, [i2.NgModel]),
      i1.ɵdid(
        10,
        16384,
        null,
        0,
        i2.NgControlStatus,
        [[4, i2.NgControl]],
        null,
        null
      )
```

```javascript
/**
The default `ControlValueAccessor` for writing a value and listening to changes on input
 * elements. The accessor is used by the `FormControlDirective`, `FormControlName`, and
 * `NgModel` directives.
*/
```

#### Zone.js - how to handle change detection

The Application subscribes to zone’s onTurnDone event, which, if we do some more digging, calls tick, which actually calls all of the change detectors.
So, without zones, we don’t get any change detection, so we don’t get any of the nice UI updates that we’d expect!

Angular picks up changes and then calls tick so that any listeners for those changes are actually fired.

https://www.npmjs.com/package/zone.js?activeTab=readme
https://blog.angularindepth.com/i-reverse-engineered-zones-zone-js-and-here-is-what-ive-found-1f48dc87659b