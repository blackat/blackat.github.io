---
layout: single
title: "Angular 8 Bootstrap in deep"
date: '2019-12-22'
last_modified_at: '2019-12-22'
comments: true
categories:
  - angular
tags:
  - [angular, view-engine]
toc: true
toc_label: "On this page"
toc_icon: "list"
toc_sticky: true
---
The Angular bootstrap sequence starts after the load of the `index.html` and the JavaScript bundles produced by Webpack. *Angular Runtime* creates the *platform* where the *application* can be started and finally the root component is rendered.

What follows is related to Angular View Engine and experimented on *Angular 8.2.9*{: .italic-red-text }.

View Engine is going to be replaced by Angular Ivy in version 9, detail introduction can be found in this [blog post](../angular-ivy-introduction/).

__Disclaimer__
The post contains the thoughts of a preliminary investigation on how Angular works reading the some parts of the source code, debugging a simple application and reading how the compiler works. Some terms or definition could be wrong.
{: .notice--danger}

## Slides

This post comes along with a [presentation](https://blackat.github.io/presentations/docs/angular-bootstrap-deep/index.html) written in markdown, rendered via [`reveal.js`](https://github.com/hakimel/reveal.js/) and available on GitHub.

## Lingo

- *Angular View Engine:*{: .italic-violet-text} Angular rendering architecture (compiler and runtime) introduced in version 4 and substituted with Ivy in Angular version 9.
- *Angular compiler:*{: .italic-violet-text} compiles templates and decorators into a code the runtime can execute.
- *Angular runtime:*{: .italic-violet-text} execute the JavaScript code produced by the compiler to run the application.
- *Object Model (OM):*{: .italic-violet-text} a way to model via object-oriented techniques (objects, classes, interfaces, properties, inheritance, encapsulation, etc) a system for development purpose. For instance Apache POI implements a OM of Microsoft Excel that manipulates via a Java API.
- *Data Model (DM):*{: .italic-violet-text} it represents entities at database level, it deals with table schema, relationships between tables (FKs, PKs) but not advanced object-oriented concepts like inheritance or polymorphism. DM represents how OM classes are stored in a database.
- *DOM:*{: .italic-violet-text} an object oriented representation of a HTML document in a tree fashion that can be manipulated via the DOM API, for instance `HTMLButtonElement` is one of the DOM interfaces.
- *Shadow DOM:*{: .italic-violet-text} it allows to separate DOM into smaller and encapsulated object-oriented representations of a HTML element.
- *Tree and nodes:*{: .italic-violet-text} the DOM is organized in a logical tree where its nodes are the components or HTML elements.
- *Rendering/painting:*{: .italic-violet-text} the browser process that transform the DOM into the UI.
- *Virtual DOM:*{: .italic-violet-text} the virtual representation of the real DOM.
- *Diffing:*{: .italic-violet-text} operation that compare two Virtual DOM.
- *Incremental DOM:*{: .italic-violet-text} a technique to render and update an Angular component when change detection is fired.

## The plan

Hello Reader, this is a long post so feel free to skip certain sections I have used to introduce and give a more complete context to the Angular bootstrap sequence that is the goal :bowtie:

The post starts with an introduction about the DOM and two *rendering strategies*{: .italic-red-text} used to speed up the page repainting. The *incremental DOM* strategy is the base of the Angular rendering architecture.

The `Welcome to Angular` simple application will help to introduce and talk about the *Angular compiler*, why and how the *Angular declarative syntax* is transformed into JavaScript code that can be executed by the *Angular runtime* in the browser. A deep look into the generated code and the Angular source code will show how the framework creates the DOM element and answer to change detection.

Some of the content and mechanisms has been changed with the introduction of the new rendering architecture called Angular Ivy.

## Browser DOM

> The Document Object Model (DOM) connects web pages to scripts or programming languages by representing the structure of a document such as the HTML representing a web page in memory.

<cite>MDN</cite> - Mozilla Developer Network
{: .small}

**Tip**
The HTML document is represented in object-oriented fashion, as objects in a logical tree, by the DOM that also provides the [API](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) to manipulate those objects.
{: .notice--info}

The DOM rendered gives the HTML page visible to the end user.

### DOM rendering is slow

Being the DOM represented as a tree, it makes easier to change and update it. What the user sees is the result of the DOM rendering operation that is the *slow part*{: .italic-red-text }. More a page or a component is complex more could take time to render it.

A page is usually made of many components, complex and non-complex. Every time one of them changes the all page (or big part of it) needs to be re-rendered, a really expensive operation.

**Tip**
Frequent DOM manipulations make the user interface slow since the re-painting of the user interface is the most expensive part. In general, it is something that is not considered when the page is going to be implemented. For instance changing visibility of an element forces the browser to verify/check visibility of all other dom nodes.
{: .notice--info}

Actions like changing visibility or the background of an element trigger a repaint. A simple click of the user could correspond to many different actions behind the scene and so many repainting actions slowing down the web page.

Two different techniques have been developed to overcome the rendering issue for complex web applications: *Virtual DOM* and *Incremental DOM*.

## Virtual DOM

The key idea is to *render the DOM the least possible*{: .italic-red-text }. When a change detection occurs, instead of updating the real DOM, frameworks like React updates a *Virtual DOM*.

The Virtual DOM is a _tree_ as well, made of _nodes_ that are the page elements. When a new element is added/removed a new Virtual DOM is created, the _difference_ between the two trees is calculated.

A transformations series is calculated to update the browser DOM so that it *matches*{: .italic-red-text } the latest new Virtual DOM. These transformations are both the minimal operations to be applied to the real DOM and the ones that reduce the performance cost of the DOM update.

__Internals__
The rendering process happens only on the *difference*. The *bulk changes* to be applied are optimized to improve the performance cost.
{: .notice--internals}

![image-center](/assets/images/posts/angular-bootstrap-in-deep/virtual-dom-transparent.png){: .align-center}

### What Virtual DOM looks like

The Virtual DOM is something *not official*, no specification is provided *differently* from DOM and shadow DOM.

It is a *copy*{: .italic-red-text } of the original DOM as a *plain JavaScript object (JSON)* so that it can be modified how many times we want without affecting the real DOM. The Virtual DOM can be divided in chunks so that it is easier to *diffing*{: .italic-red-text } the changes.

#### Example

When a new item is added to an unordered list of elements a copy of the Virtual DOM containing the new element is created.

The *diffing*{: .italic-red-text } process collects the differences between the two Virtual DOM objects so changes can be transformed in a bulk update against the real DOM.

**Tip**
No distinction about _reflow_ (element layout that is position recalculation and geometry) and *repaint* (element visibility) has been done so far since most of the considered actions involve the repaint operation.
{: .notice--info}

### How React uses the Virtual DOM

In React a user interface is made of a set of components, each component has a *state*{: .italic-red-text }. For instance the state of a drop-down menu is the array of the available elements and the current selected one.

Via the [observer pattern](https://www.baeldung.com/java-observer-pattern) React listens to *state change*{: .italic-red-text } to update the Virtual DOM. The *diffing*{: .italic-red-text } process makes React aware of which Virtual DOM objects have changed, *only*{: .italic-red-text } those objects will be updated in the real DOM.

**Tip**
As developer you don't need to be aware about how DOM manipulation happens at each state change. React does the job optimizing the performance cost behind the scenes.
{: .notice--info}

React reduces the re-painting cost applying updates in bulk *not*{: .italic-red-text } at every single state change.

The *great advantage*{: .italic-red-text } of using the Virtual DOM is that we don't need any compiler. JSX for instance, is really close to JavaScript, the key point is *the render function*{: .italic-red-text } that can be implemented using any programming language.

### Virtual DOM drawbacks

- The Virtual DOM required an *interpreter* to interpret the component. At compile time no way exists to know which parts of the interpreter will be required at runtime, so the *whole stuff*{: .italic-red-text } has to be loaded by the browser.
- Every time there is a change, a new Virtual DOM has to be created, may be a chunk and not the whole tree, but *the memory footprint is high*{: .italic-red-text }.

## Incremental DOM

The key idea of the incremental DOM is:

> Every component gets compiled into a series of instructions. These instructions create DOM trees and update them in-place when the data changes.

<cite>Victor Savkin</cite> - [Understanding incremental DOM](https://blog.nrwl.io/understanding-angular-ivy-incremental-dom-and-virtual-dom-243be844bf36)
{: .small}

Each component is then compiled into *two main instruction sequences*:

- *view creation:*{: .italic-blue-text } invoked the first time the page is rendered, add the component to the DOM;
- *change detection:*{: .italic-blue-text } invoked at every state change to update the component into the DOM.

The advantages of the Incremental DOM are a low memory footprint and a skinny interpreter/runtime tailored on the compiled application.

**Angular Ivy**
The Incremental DOM strategy is already present in the Angular View Engine. As it will be shown, each component is compiled into a creation function and an update function. Angular Ivy goes further, it allows the *tree-shaking* of the Angular runtime that is not possible with the current rendering architecture.
{: .notice--special}

## Angular compiler

An Angular application is mainly made by *Angular components* organized in a tree fashion. Each component is implemented to accomplish a certain mission, for instance the navigation bar, the drop-down menu, etc.

### Angular component

An Angular component is characterized by a class, *TypeScript code*{: .italic-red-text } that expresses the *logic*, and a decorator that allow to define some *metadata*{: .italic-red-text } such as the `selector`, the `template`, etc. The *HTML template*{: .italic-red-text } represents the *presentation layer* of the component and it is implemented using a specific *Angular declarative syntax*.

**Tip**
When the developer write a component uses TypeScript and the Angular declarative syntax for the template to *bind*{: .italic-blue-text} a variable from the logic to the presentation layer and vice versa. Pay attention that *no change detection*{: .italic-blue-text} is required to be added. Change detection works at runtime thanks to the compiler that adds it during the compilation phase.
{: .notice--info}

#### Example

Consider a very simple component, template can be inline or separated:

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

A *template* is a bunch of HTML code with binding variables to *present*{: .italic-red-text }, with a certain look and feel, some content.

```html
<div style="text-align:center">
  <h1>
    {% raw %}Welcome to {{ title }}!{% endraw %}
  </h1>
</div>
```

### Browser cannot render an Angular component

The browser is the *execution environment*, it loads the application and executes it. Unfortunately it cannot execute an Angular component *as it is*{: .italic-red-text }.

**Tip** A browser can interpret JavaScript and render HTML but not if written using the *Angular declarative syntax*.
Angular provides a compiler that, together with the TypeScript one, transforms *"everything in something else"* that a browser can understand.
{: .notice--info}

During the build of an Angular project, two compilers come into play with _different purposes_:

- `tsc` is the TypeScript compiler and generates the JavaScript w.r.t. the target specified in the `tsconfig.json`, for instance `target: es2015`.
- `ngc` is the Angular compiler that translates the templates and decorators into JavaScript. The Angular compiler can work in two different modes:
  - *Ahead-of-Time (AoT):*{: .italic-blue-text } work at build time so that the templates are bundled along with the application, suitable for production.
  - *Just-in-Time (JIT):*{: .italic-blue-text } templates are not pre-compiled, the compiler comes along with the application, it is loaded by the browser and does the work at runtime, suitable for development.

**Internals** During the development phase `ng serve` provides _live reload_ functionality.
The process goes through `@ngtools/webpack`, compiled code is *not saved to disk*{: .italic-violet-text }, everything is consumed in memory via streams and emitters.
{: .notice--internals}

## Angular vs. browser

What are then the *roles*{: .italic-red-text } of the browser and Angular?

Once the Angular application has been *fully transformed into JavaScript* (HTML templates included), WebPack bundles it along with library dependencies in order to improve performance and load times.

### Browser

The *browser role*{: .italic-red-text } is to load the `index.html` and to provide the execution environment, the render and the event loop.
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

    <!-- nomodule defer -->
    <script src="runtime-es5.js" nomodule defer></script>
    ...
  </body>
</html>
```

The scripts can be loaded both by modern browsers that supports ESM modules and by old ones that do not support modules via `nomodule defer` attributes.

### Angular

Consider an Angular application made of only the component previously introduced. The `main-es2015.js` contains the fully bundled application while `runtime-es2015.js` is the Angular runtime. Finally third party libraries and styles.

**Tip**
The transformed HTML template into JavaScript becomes a series of instructions that, once called, *render* the page building the components.
Skipping some details, roughly an element is a factory function that uses the *injected Angular renderer* to render the element w.r.t. the *browser platform*.
{: .notice--info}

The *Angular runtime* bootstraps the `AppModule` that, in turn, creates and renders the root element of the application `<app-root>`. The file `main-es2015.js` contains the *view definition factories*{: .italic-red-text } produced by the compiler and enriched by Webpack.

**Internals**
 If the browser platform is chosen, `@angular/platform-browser`, the element will be rendered creating the `HTML` code into the DOM via the `Document` interface: `document.createElement()`. When something changes, the element will update itself calling the update function.
{: .notice--internals}

**Angular Ivy**
The compilation process of View Engine produces `.metadata.json` and `.ngfactory.js` files. With Angular Ivy no more special files are produced, too complex to manage and to merge them. Ivy instructions are directly put into the component, a component knows how to create and update itself.
{: .notice--special}

## Analyze the compiled code

Let's see how to compile the application invoking *only*{: .italic-red-text } the `ngc` compiler and nothing else to inspect the compiled code easily and see where the generated JavaScript code invoke the DOM API to create the element.

**Tip**
The `HTML` template has been compiled into a sequence of JavaScript instructions that will be executed by the Angular runtime. The *goal*{: .italic-blue-text} of the coming sections is to find where the `document.createElement()` is invoked after the different Angular entities (platform, application and component) have been instantiated.
{: .notice--info}

### Setup the compilation task

Open the `package.json` file and add:

```javascript
"scripts": {
  ...
  "compile": "ngc"
},
```

then in the `tsconfig.json` enable the `d.ts` files generation to have the TypeScript definitions:

```javascript
"compilerOptions": {
  ...
  "declaration": true,
  ...
}
```

### One and simple component

Create a new `Welcome to Angular` application via the Angular CLI.

#### The module and the component

The *module* is as follows:

```javascript
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

then the *component* of the `Welcome to Angular` application:

```javascript
@Component({
  selector: 'app-root',
  template: `
    <div style="text-align:center">
      <h1>
        {% raw %}Welcome to {{ title }}!{% endraw %}
      </h1>
    </div>
  `,
  styleUrls: []
})
export class AppComponent {
  @Input() title = 'Angular';
}
```

#### Compile

Run the command `npm run compile` and look into the folder `dist/out-tsc/src/app` where everything has been transformed into JavaScript and *saved to disk*.

The Angular compiler has produced some files, skip the `.metadata` and `.d.ts` ones:

```bash
app.module.js               // module class
app.module.ngfactory.js     // module factory, transformed metadata decorator
app.component.js            // component class
app.component.ngfactory.js  // component factory, transformed metadata decorator
```

### Module factory function

The `app.module.ngfactory.js` contains the _factory function creator_:

```javascript
import * as i0 from "@angular/core";
import * as i1 from "./app.module";
import * as i2 from "./app.component";
import * as i3 from "./app.component.ngfactory";
...
var AppModuleNgFactory = i0.ɵcmf(i1.AppModule, [i2.AppComponent], function(_l) {...}
...
```

__Warning__
The functions produced by the Angular template compiler start with `ɵ` to clearly warn *to not use them*{: .italic-yellow-text} because for sure the code will change soon in the future.
{: .notice--warning}

The function `ɵcmf` stands for *create module factory*, the map between the name and the real function is defined in the following static map object [`Map<ExternalReference, any>`](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/platform-browser-dynamic/src/compiler_reflector.ts#L70):

```javascript
function createBuiltinExternalReferencesMap() {
  const map = new Map<ExternalReference, any>();
  ...
  map.set(Identifiers.createModuleFactory, ɵcmf);
  ...
  return map;
```

**Angular Ivy**
The aforementioned map object is one of the reason why the View Engine is not tree-shakable. Angular Ivy should get rid off or change how this static map is defined to allow the runtime to be tree-shaked by the any open source tool.
{: .notice--special}

### What is going to happen

The compiler has transformed the decorators, `@NgModule` and `@Component`, into JavaScript instructions. Now *"imagine"* that the TypeScript class has been transpiled into JavaScript and the `@Component` decorator that decorates the class became the factory that tell Angular runtime how to create the component into the DOM (*create view*) and how to update it (*change detection*). The `@NgModule` decorators will tell the Angular runtime how to instantiate the application module and get *service providers* injected.

The *module factory function* will create an *application object* that, in turn, will bootstrap the *application module* and finally the *root component*.

#### Module factory implementation

The module factory function `ɵcmf` creates the [module factory object](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/view/entrypoint.ts#L35) `AppModuleNgFactory` previously shown. here the implementation:

```javascript
export function createNgModuleFactory(
    ngModuleType: Type<any>, bootstrapComponents: Type<any>[],
    defFactory: NgModuleDefinitionFactory): NgModuleFactory<any> {
      return new NgModuleFactory_(ngModuleType, bootstrapComponents, defFactory);
    }
```

it implements the [following interface](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/linker/ng_module_factory.ts#L60):

```javascript
export abstract class NgModuleFactory<T> {
    abstract get moduleType(): Type<T>;
    abstract create(parentInjector: Injector|null): NgModuleRef<T>;
}
```

#### Module creation

The *module factory object* can create a *module* of type `AppModule` defined in the class `app.module.js`, that will bootstrap a component of type `AppComponent` defined in the file `app.component.js`.

The `defFactory` is a *module defintion function*, `ɵmod`, used by the `create` method to produce the real module object. It contains an array of `ɵmpd` *module provider definitions* that, for instance, tell which sanitizer or producer has to be created and injected:

```javascript
...
var AppModuleNgFactory = i0.ɵcmf(i1.AppModule, [i2.AppComponent], function(_l) {
  return i0.ɵmod([
    ...
    i0.ɵmpd(4608, i5.DomSanitizer, i5.ɵDomSanitizerImpl, [i4.DOCUMENT]),
    i0.ɵmpd(6144, i0.Sanitizer, null, [i5.DomSanitizer]),
    ...
    i0.ɵmpd(6144, i0.RendererFactory2, null, [i5.ɵDomRendererFactory2]),
    ...
  ]
}
```

### Component factory function

Open `app.component.ngfactory.js` and look at `ɵccf` or *create component factory* function:

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

it is defined as [follows](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/view/refs.ts#L33):

```javascript
export function createComponentFactory(
    selector: string, componentType: Type<any>, viewDefFactory: ViewDefinitionFactory,
    inputs: {[propName: string]: string} | null, outputs: {[propName: string]: string},
    ngContentSelectors: string[]): ComponentFactory<any> {
  
  return new ComponentFactory_(
      selector, componentType, viewDefFactory, inputs, outputs, ngContentSelectors
    );
}
```

The factory function is similar to the module one except for some more parameters. A component can have `@Input()` and `@Output` properties and hence the arrays `inputs` and `outputs`.

**Tip**
It starts to be more and more clear how the component declaration is transformed into a set of arguments used by a factory to *programmatically* create the component at runtime.
{: .notice--info}

#### Compiled template

What happened to template? This is why you have read so far... I hope :sweat_smile:

The component template has been transformed into a JavaScript object with the [following interface](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/view/types.ts#L53):

```javascript
export interface ViewDefinition extends Definition<ViewDefinitionFactory> {
  flags: ViewFlags;
  updateDirectives: ViewUpdateFn;
  updateRenderer: ViewUpdateFn;
  handleEvent: ViewHandleEventFn;
  nodes: NodeDef[];
  nodeFlags: NodeFlags;
  rootNodeFlags: NodeFlags;
  lastRenderRootNode: NodeDef|null;
  bindingCount: number;
  outputCount: number;
  nodeMatchedQueries: number;
}
```

The *view definition* `ɵvid` with the `app-root` *host selector*:

```javascript
export function View_AppComponent_Host_0(_l) {
  return i1.ɵvid(
    0,
    [
      (_l()(),
        i1.ɵeld(
          0,0,null,null,1,"app-root",[],null,null,null,
          View_AppComponent_0,RenderType_AppComponent
        )),
      i1.ɵdid(1, 49152, null, 0, i2.AppComponent, [], null, null)
    ],
    null,
    null
  );
}
```

*Host selector*{: .italic-red-text} since the component is attached/hosted by the selector, the Angular component is a directive, hence the view definition is characterized by (*links point to the Angular source code on GitHub*):

- *element definition*{: .italic-blue-text }, `ɵeld`, the `app-root`, the function produces an [`ElementDef`](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/view/types.ts#L245);
- *directive definition*{: .italic-blue-text }, `ɵdid`, the directive that represents the component, the function [`directiveDef`](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/view/provider.ts#L30) produces an object of type [`NodeDef`](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/view/types.ts#L110).

Both produced objects are of type [`NodeDef`](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/view/types.ts#L110).

The *element definition* `ɵeld` references then `View_AppComponent_0`, the other JavaScript code that represents the component template:

```javascript
export function View_AppComponent_0(_l) {
  return i1.ɵvid(0,
    [
      (_l()(),
      i1.ɵeld(0, 0, null, null, 1, "h1", [], null, null, null, null, null)),
      (_l()(), i1.ɵted(1, null, ["Welcome to ", "!"]))
    ],
    null,
    function(_ck, _v) {
      var _co = _v.component;
      var currVal_0 = _co.title;
      _ck(_v, 1, 0, currVal_0);
    }
  );
}
```

The `ɵvid`, [`viewDef`](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/view/view.ts#L23) function, takes two view update functions: `updateDirectives` and `updateRenderer` for the change detection along with the function to create the element the first time the application is loaded.

{% capture notice-text %}
In a view defintion function `ɵvid` there are two interesting things:

- `NodeDef[]` array of nodes that is responsible of the view creation;
- `updateDirectives` and `updateRenderer` functions responsible of the change detection update.
{% endcapture %}

<div class="notice--internals">
  <strong>Internals</strong>
  {{ notice-text | markdownify }}
</div>

**Angular Ivy**
In Angular Ivy there are no more `.ngfactory.js` files, all the required code for *view creation*{: .italic-lime-text} and *change detection*{: .italic-lime-text} is inside the component. Imho the incremental DOM if fully implemented in Ivy, what is missing in View Engine is the possibility to tree-shake the runtime to squeeze it as much as possible.
{: .notice--special}

## How Angular application bootstraps

Once the compiled code has been analyzed, it is interesting to see the call sequence to the Angular runtime to find which function renders the component. At the end of the sequence there must be the sought after `document.createElement()` function call to the DOM API.

Build the application and start a live server to debug it into the browser:

```bash
ng build --aot
npx http-server dist/test-ivy
```

> The Angular AOT compiler extracts metadata to interpret the parts of the application that Angular runtime is supposed to manage.

Basically the compiler manages metadata interpretation and template compilation that can be controlled specifying some template compiler options in the `tsconfig.json`.

**Angular Ivy**
Activate the Ahead-of-Time compilation to have everything in JavaScript and *saved to disk*{: .italic-lime-text} make it easier to inspect the generated code. With Angular Ivy `--aot` is not necessary anymore since it is activated by default. Ivy compilation is so fast that AoT compilation can be always used.
{: .notice--special}

### 0. IIEF

The application starts in the file `main-es2015.js`. The option `--aot` contributes to some optimizations, `bootstrapModule` is replaced by `bootstrapModuleFactory` as you can observe from the file [`main-aot.ts`](https://github.com/angular/angular.io/blob/281efb9ca0d278b36e2e7fa0850a807d7005e50b/public/docs/_examples/upgrade-phonecat-3-router/ts/app/main-aot.ts#L7):

```javascript
import { platformBrowser } from '@angular/platform-browser';

import { AppModuleNgFactory } from './app.module.ngfactory';

// *** Follow bootstrapModuleFactory() ***
platformBrowser().bootstrapModuleFactory(AppModuleNgFactory);
```

*Pay attention:*{: .italic-red-text} in each piece of code there is a comment that allows to follow the bootstrap call sequence `// *** Follow`.

**Tip**
When invoking the `ng build` and not simply the compiler as done before, Webpack bundles what has been produce by the compiler, so opening the files results in a slightly different code.
{: .notice--info}

![image-center](/assets/images/posts/angular-bootstrap-in-deep/bootstrap-sequence.png){: .align-center}

*Basically*{: .italic-blue-text } the IIEF function bootstraps the platform `PlatformRef`, that, in turn, instantiates the application `ApplicationRef` and then the module along with all the required injectable providers. Finally the component is created and rendered into the DOM.

**Internals**
The application code is composed by `app.module.ts` and `app.component.ts`. First Angular runtime has to be started, then it creates the *platform* linked to the page, starts the *application* that is the *module*. Once the module has been started the *component* can be instantiated and rendered.
{: .notice--internals}

### 1. Platform

The Angular platform [`PlatfromRef`](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/application_ref.ts#L235) is the *entry point for Angular on a web page*. Each page has *exactly one platform* and services bound to its scope. A *page's platform* is initialized implicitly when a platform is created via a platform factory (e.g. `platformBrowser`).

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
      return ngZone.run((
        const ngZoneInjector = Injector.create(
          {providers: providers, parent: this.injector, name: moduleFactory.moduleType.name});

        // from here the ApplicationRef is created and available to be injected
        const moduleRef = InternalNgModuleRef<M>moduleFactory.create(ngZoneInjector);
        ...
        // *** Follow _moduleDoBootstrap() ***
        // moduleType: *class AppModule*
        this._moduleDoBootstrap(moduleRef);
        return moduleRef;
        ...
      ));
    }
    ...
    /**
     * Bootstrap all the components of the module
     */
    _moduleDoBootstrap(moduleRef) {
      ...
      const appRef = moduleRef.injector.get(ApplicationRef) as ApplicationRef;
      ...
      // loop over the array defined in the @NgModule, bootstrap: [AppComponent]
      moduleRef._bootstrapComponents.forEach((
        // *** Follow bootstrap() ***
        // bootstrap the root component *AppComponent* with selector *app-root*
        f => appRef.bootstrap(f)));
      ));
    }
}
```

*Basically*{: .italic-blue-text } change detection is managed by [`Zone.js`](https://blog.angularindepth.com/i-reverse-engineered-zones-zone-js-and-here-is-what-ive-found-1f48dc87659b) that run the module bootstrap. `ApplicationRef` reference is created and then it bootstraps the `AppComponent` component.

### 2. Application

The [`ApplicationRef`](https://github.com/angular/angular/blob/d2222541e8acf0c01415069e7c6af92b2acbba4f/packages/core/src/application_ref.ts#L503) reference represents an Angular application *running on a page*.
  
  ```javascript
  class ApplicationRef {
      ...
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
        // *** Follow create() ***
        const compRef = componentFactory.create(Injector.NULL, [], selectorOrNode, ngModule);
        ...
      }
  }
  ```

### 3. Root component

Build the root component:
  
  ```javascript
  class ComponentFactory_ extends ComponentFactory {
    ...
    create(injector, projectableNodes, rootSelectorOrNode, ngModule) {
      const view = Services.createRootView(injector, projectableNodes || [], rootSelectorOrNode, viewDef, ngModule, EMPTY_CONTEXT);
    }
  }
  ```

*Basically*{: .italic-blue-text } the Angular [`component_factory.ts`](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/core/src/linker/component_factory.ts#L79) holds the base class method to create a component of a certain type:

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
    // *** Follow createRootView() ***
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

*Basically*{: .italic-blue-text } the implementation uses the function `resolveDefinition()` to load the view definition. This function will be used many times around the code. The `createRootView()` function creates a `ViewData` object that contains the information that will be used later on to render the node into the DOM.

### 4. Create nodes

The code is going to arrive to the point where the DOM API is called to create and attach the element to the DOM.

```javascript
function createRootView(root, def, context) {
  const view = createView(root, root.renderer, null, null, def);
  initView(view, context, context);
  // *** Follow createViewNodes() ***
  createViewNodes(view);
  return view;
}
```

the function [`function createViewNodes(view: ViewData){...}`](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/core/src/view/view.ts#L250) creates a DOM element w.r.t. its type:

```javascript
function createViewNodes(view) {
  const nodes = view.nodes;
  for (let i = 0; i < def.nodes.length; i++) {
    switch (nodeDef.flags & 201347067 /* Types */) {
      case 1 /* TypeElement */:
        // H1 DOM element of type any, the function calls the DOM renderer to render the element
        // *** Follow createElement() ***
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

### 5. The renderer

The [`createElement`](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/core/src/view/element.ts#L151) function will use the injected renderer to create the element.w.r.t. the platform where the application runs.

In case of `PlatformBrowser`, the [`DefaultDomRenderer2`](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/platform-browser/src/dom/dom_renderer.ts#L115) class invokes the `document` interface method to create the real DOM element. `DefaultDomRenderer2` extends and implements [`abstract class Renderer2`](https://github.com/angular/angular/blob/d192a7b47a265aee974fb29bde0a45ce1f1dc80c/packages/core/src/render/api.ts#L108).

```javascript
createElement(name: string, namespace?: string): any {
    if (namespace) {
      // In cases where Ivy (not ViewEngine) is giving us the actual namespace, the look up by key
      // will result in undefined, so we just return the namespace here.
      return document.createElementNS(NAMESPACE_URIS[namespace] || namespace, name);
    }

    // *** FOUND ***
    return document.createElement(name);
  }
```

**Tip** An HTML template is transformed in an *intermediate* format or Object Model by the Angular compiler.
Factory functions are automatically generated by the compiler and they are able to produce an object that can create a component or a node or a module. Then a renderer, specified by the chosen platform, will produce DOM elements in case of a DOM renderer.
{: .notice--info}

## Conclusions

It has been shown how the Angular compiler transforms the Angular declarative syntax and the decorators into something the Angular runtime can execute. The Angular compiler and the runtime constitute the rendering architecture.

A developer can use a simple syntax without worrying about change detection and performance optimization w.r.t. the DOM updates since the Angular framework, behind the scenes, does all the job. When new optimizations are available can be got transparently and effortlessly.

One of the big issue with the current rendering architecture, View Engine, is to not be tree-shakable and difficult to expand. Angular Ivy will solve all these issues being composed by an *instruction set* that can be easily expanded and tree-shaken to avoid the delivery of the full Angular runtime to the browser as today.


## References

#### DOM

- [Understanding the critical rendering path](https://bitsofco.de/understanding-the-critical-rendering-path/)
- [Document Object Model (DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
- [What exactly is the DOM](https://bitsofco.de/what-exactly-is-the-dom/)

#### Virtual DOM

- [React Virtual Dom](https://programmingwithmosh.com/react/react-virtual-dom-explained/)
- [Understanding the Virtual Dom](https://bitsofco.de/understanding-the-virtual-dom/)

#### Angular compiler

- [Deep dive into the Angular compiler](https://blog.angularindepth.com/a-deep-deep-deep-deep-deep-dive-into-the-angular-compiler-5379171ffb7a)
- [Deep dive into the Angular compiler](https://www.youtube.com/watch?v=QQ2plVD0gDI&feature=youtu.be&t=27m)
- [The Angular Compiler 4.0](https://www.youtube.com/watch?v=RXYjPYkFwy4)
- [Mad science with the Angular Compiler](https://www.youtube.com/watch?v=tBV4IQwPssU)

#### Incremental DOM and Ivy

- [Inside Ivy: Exploring the New Angular Compiler](https://blog.angularindepth.com/inside-ivy-exploring-the-new-angular-compiler-ebf85141cee1)
- [Understanding Angular Ivy: Incremental DOM and Virtual DOM](https://blog.nrwl.io/understanding-angular-ivy-incremental-dom-and-virtual-dom-243be844bf36)
- [Incremental DOM](http://google.github.io/incremental-dom/)
- [Why incremental DOM](http://google.github.io/incremental-dom/#why-incremental-dom)
- [Introducing Incremental DOM](https://medium.com/google-developers/introducing-incremental-dom-e98f79ce2c5f)

#### Zone

- [I reverse-engineered Zones (zone.js) and here is what I’ve found](https://blog.angularindepth.com/i-reverse-engineered-zones-zone-js-and-here-is-what-ive-found-1f48dc87659b)