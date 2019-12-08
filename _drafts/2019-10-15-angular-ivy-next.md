---
layout: single
title: "Angular Ivy Next"
date: '2019-11-10'
last_modified_at: '2019-11-10'
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

Angular Ivy is the new *rendering architecture* that comes, by default, with version Angular 9. The Angular rendering architecture is not new to a completely revamp, Angular 2.0, Angular 4.0 and now Angular 9.0 have introduced new *compilers* and *runtime engines*.

Currently Angular stable version is 8.3.9 and Angular 9 is in RC4.

## Lingo

- rendering architecture
- runtime rendering function set/instruction set
- runtime/compiler
- virtual DOM
- incremental DOM
- View Engine
- `angular.json` is the workspace or build configuration file.
- `tsconfig.app.json` is the project configuration file.
- `.ngfactory.js` suffix for decorator factory files, class decorators like `@Component` are translated by the compiler into external files.
- Locality: compiler should use only information from the component decorator and its class.
- Global compilation: compilation process requires global static analysis to emit the application code.

## Rendering Architecture

What is a rendering architecture? It is the pair *compiler:runtime*. Angular framework is composed by two main parts:

- *compiler* to transform templates written in Angular declarative syntax into JavaScript instructions enriched with change detection;
- *runtime* to execute the application code produced by the compiler.

Currently, Angular 8 uses as rendering architecture called *View Engine*:

- **View Engine** has been introduced with Angular version 4 and still used in version 8, but some limitations has been identified
  - *no tree-shakable:* both the `Hello World` application and a very complex one are executed by the same and full runtime. If the internationalization module is not used, for instance, it is part of the runtime anyhow, basically the runtime it cannot be tree-shakable;
  - *no incremental compilation:* *Angular compilation is global*{: .italic-red-text} and it involves not only the application but also the libraries.
- **Ivy** will the new default rendering engine starting from version 9 and should solve the View Engine current issues:
  - *simplify* how Angular works internally;
  - *three-shakable*  the `Hello World` application does not required the full Angular runtime and wil be bundled in only 4.7 KB;
  - *incremental compilation* is not possible so the compilation is faster than ever and `--aot` can be now used even during development mode (advice from Angular team).

> "Ivy is an enabler."

<cite>Igor Minar</cite> - Angular team
{: .small}

The incremental DOM is the [foundation of the new rendering engine][nrwl-ivy]

## Incremental DOM vs. Virtual DOM

> Every component gets compiled into a series of instructions. These instructions create DOM trees and update them in-place when the data changes.

<cite>Viktor Savkin</cite> - [Nrwl][nrwl-ivy]
{: .small}

Each compiled component has *two main set of instructions*:

- *view creation* instructions executed when the component is rendered for the first time;
- *change detection* instructions to update the DOM when the component changes.

Change detection is basically a set of instructions added at compile time. The developer life is made easier since he is aware only of the variable *binding* in the Angular template declarative.

Incremental DOM enables *better bundle size and memory footprint*{: .italic-red-text} so that applications can perform really well on mobile devices.

### Virtual DOM

React and Vue are based on the concept of *Virtual DOM* to create a component and re-render it when change detection happens.

Render the DOM is a very expensive operation, when a component is added to the DOM or changes happen, the repaint operation has to take place. Virtual DOM strategy aims to reduce the amount of work on the real DOM and so the number of time the user interface needs to be repainted.

DOM manipulations happens every time a new component is going to be added, removed or changed from the DOM, so instead of operating directly on the DOM it operates on a JSON object called Virtual DOM. When a new component is added or an existing one removed, a new Virtual DOM is created, the node added or removed and the difference between Virtual DOMs is computed. A sequence of transformations is applied to the real DOM.

![image-center](/assets/images/posts/angular-ivy-intro/virtual-dom-transparent.png ){: .align-center}

React advices to use JSX, a *syntax extension* to JavaScript, to define *React elements*. JSX is not a template language. A template is an enriched JavaScript expression that is interpreted at runtime. Plain JavaScript can also be used instead of JSX.

Virtual DOM technique has some disadvantages:

- create a whole tree every time a change happens (add or remove a node), so the memory footprint is quite important;
- an interpreter is required as long as the *diff* algorithm to compute the difference among the Virtual DOMs. At compile time it is not known which functionalities are required to render the application, so *the whole thing has to be shipped to the browser*.

### Incremental DOM

It is the foundation of the new rendering engine. Each *component template* gets compiled into creation and change detection instructions: one to add the component to the DOM and the other one to update the DOM *in-place*{: .italic-red-text}.

> Instructions are *not interpreted* by the Angular runtime, the rendering engine, but *the instructions are the rendering engine*{: .italic-red-text}.

<cite>Viktor Savkin</cite> - [Nrwl][nrwl-ivy]
{: .small}

Since the runtime does not interpret the template component instructions, but the *component references instructions*{: .italic-red-text} it is quite easy to remove those instructions that are not referenced. At compile time the unused instruction can be excluded from the bundle.

The amount of memory required to render the DOM is *proportional* to the size of the component.

**Tip**
The compiled template component references some instructions of the Angular runtime which holds the implementation.
{: .notice--info}

## Enable Angular Ivy

Ivy can be enable in an existing project with latest Angular version but also directly scaffold a project with Ivy.

### Enable Ivy in an existing project

Having an existing [Angular (8.1.x)](https://angular.io/guide/ivy) project run:

```bash
$ ng update @angular/cli@next @angular/core@next
```

both the Angular core and the CLI will be updated at the latest release candidate. One interesting thing to notice is the `"aot": true` in the `angular.json` *workspace configuration file*:

> AOT compilation with Ivy is faster and should be used by default.

[Angular docs](https://angular.io/guide/ivy)
{: .small}

Then add the angular compiler options in the `tsconfig.app.json`:

```javascript
{
  "compilerOptions": { ... },
  "angularCompilerOptions": {
    "enableIvy": true
  }
}
```

### New project with Ivy

To start a new project with Ivy run:

```bash
$ new my-app --enable-ivy
```

### Disable Ivy

To disable Ivy:

- in `angular.json` set `"aot": false`;
- in `tsconfig.app.json` remove the `angularCompilerOptions` option or set `"enableIvy": false`.

## Angular Ivy Compilation

Consider the following *Hello World* Angular component:

```javascript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div style="text-align:center">
      <h1>
        Welcome to {{ title }}!
      </h1>
    </div>
  `,
  styleUrls: []
})
export class AppComponent {
  @Input() title = 'Angular!';
}
```

it gets compiled into the following code:

```javascript
class AppComponent {
  constructor() {
    this.title = 'Angular!';
  }
}
AppComponent.ngComponentDef = defineComponent({
        selectors: [['app-root']],
        factory: function() { return new AppComponent();}
    },
    template: function(flags, context) {
        if (flags & 1) {
            elementStart(0, "div", 0);
            elementStart(1, "h1");
            text(2);
            elementEnd();
            elementEnd();
        } if (flags & 2) {...}
    },
    directives: [...]
  });
```

The Angular decorators are compiled into *static fields*{: .italic-red-text} in the decorated class. So `@Component` becomes `ngComponentDef` static field. Back to View Engine, the `ngc` compiler produces `.ngfactory` separated files for each component and modules. With Ivy the code produced by the compiler is move into *component class static fields*.

The `elementStart()`, `elementEnd()`, etc are the *component referenced instructions*, *every component is its own factory*{: .italic-red-text}, the framework does not interpret the component.

All the *not referenced*{: .italic-red-text} instructions at compile time are removed from the final application bundle.

![image-center](/assets/images/posts/angular-ivy-intro/incremental-dom.png ){: .align-center}

**Tip**
The View Engine runtime is a *single monolith interpreter* that is not tree-shakable and has to be entirely shipped to the browser. Differently, *Angular Ivy runtime* is an *instruction set*{: .italic-red-text} that is a *set of rendering functions*{: .italic-red-text} like an assembly language for templates.
{: .notice--info}

## What Angular Ivy enables

*Angular Ivy is an enabler.*{: .italic-red-text} Simplifying how Angular works internally and the compilation process resolves current View Engine limitations and makes Angular easily extensible to new features.

The new Ivy engineering has been driven by three main goals: *three-shaking, locality and flexibility*.

### Tree-shaking

Tree-shaking is the operation of removing *dead code*{: .italic-red-text} from the bundle so if the application does not reference some of the runtime rendering function, they can be omit from the bundle making it smaller.

Dead code comes from libraries, Angular included. Angular CLI is powered by Webpack uglify plugin as tree-shaker that, in turn, receives information from *Angular Build Optimizer Plugin* about which code is used and which not. Angular compiler simply does not emit those instructions, the plugin can gather information about component referenced instructions so can instruct Uglify about what to include/exclude in/from the bundle.

While the `@angular/core` framework is tree-shakable, the View Engine runtime is not, it cannot be broken into small pieces and this is mainly due to the static `Map<Component, ComponentFactory>` variable.

### Incremental compilation

The Angular 8 compilation pipeline started by `ng build prod --aot` is composed by five phases where the `tsc` and the `ngc` generates the *template factories*. `ngc` compiles the libraries as well. Ivy enables *Incremental compilation* that is libraries can be compiled and deployed on npm.

### Locality

Currently, Angular relies on *global compilation*{: .italic-red-text}, the compilation process requires a global static analysis of the entire application to combine different compilation outputs (application, libraries from the monorepo and libraries from npm) before emitting the bundle. Moreover it is really complex to combine AOT libraries into a JIT application.

**Tip**
The compiler should use only information provided by component decorator and its class and nothing else. This simplifies the overall compilation process, no more `component.metadata.json` and `component.ngfactory.json` that requires complex management in the compilation pipeline.
{: .notice--info}

*Locality is a rule*. Ivy compilation introduces the concept of *component/directive public API*{: .italic-red-text}: an Angular application can *safely refer to components and directives public API*, no more needed to know much about dependencies since *extra information* are added to `.d.ts` component files.

#### Ivy library compilation example

Add a library to the monorepo where your application is running `ng generate library mylib`.

Compile the library with `ng build mylib`, the following files are produced:

```bash
├── bundles
├── ...
├── lib
│   ├── mylib.component.d.ts
│   ├── mylib.module.d.ts
│   └── mylib.service.d.ts
├── mylib.d.ts
├── package.json
└── public-api.d.ts
```

Notice as well that this new message is displayed in version 9 due to Ivy activation:

```bash
Building Angular Package
******************************************************************************
It is not recommended to publish Ivy libraries to NPM repositories.
Read more here: https://next.angular.io/guide/ivy#maintaining-library-compatibility
******************************************************************************
```

##### Generated component

This is the component generated by the Angular CLI:

```javascript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'lib-mylib',
  template: `
    <p>mylib works!</p>
  `,
  styles: []
})
export class MylibComponent implements OnInit {

  constructor() { }

  ngOnInit() { }
}
```

##### Compiled library code

The metadata file `mylib.metadata.json` is not generated anymore, *metadata* are now part of the *definition files*{: .italic-red-text}.

Definition file of the component:

```javascript
import { OnInit } from "@angular/core";
import * as i0 from "@angular/core";
export declare class MylibComponent implements OnInit {
  constructor();
  ngOnInit(): void;
  static ɵfac: i0.ɵɵFactoryDef<MylibComponent>;
  static ɵcmp: i0.ɵɵComponentDefWithMeta<MylibComponent,"lib-mylib",never,{},{},never>;
}
```

Definition file of the module:

```javascript
import * as i0 from "@angular/core";
import * as i1 from "./mylib.component";
export declare class MylibModule {
    static ɵmod: i0.ɵɵNgModuleDefWithMeta<MylibModule, [typeof i1.MylibComponent], never, [typeof i1.MylibComponent]>;
    static ɵinj: i0.ɵɵInjectorDef<MylibModule>;
}
```

and definition file of the service:

```javascript
import * as i0 from "@angular/core";
export declare class MylibService {
    constructor();
    static ɵfac: i0.ɵɵFactoryDef<MylibService>;
    static ɵprov: i0.ɵɵInjectableDef<MylibService>;
}
```

##### Add a property to the component

Add to the library component an input field:

```javascript
@Component({
  selector: 'lib-mylib',
  template: `
    <p>Please input your phone</p>
    <input #phone placeholder="phone number" />
  `,
  styles: []
})
export class MylibComponent implements OnInit {

  @Input('phone-number') private phone: string;

  constructor() { }

  ngOnInit() {
  }
}
```

The alias `phone-number` will be added to the *input property*{: .italic-red-text} providing additional metadata for the public API. The compiler generates the following definition file:

```javascript
import { OnInit } from '@angular/core';
import * as i0 from "@angular/core";
export declare class MylibComponent implements OnInit {
    private phone;
    constructor();
    ngOnInit(): void;
    static ɵfac: i0.ɵɵFactoryDef<MylibComponent>;
    static ɵcmp: i0.ɵɵComponentDefWithMeta<MylibComponent, "lib-mylib", never, { 'phone': "phone-number" }, {}, never>;
}
```

> Decorator that marks a class field as an input property and supplies configuration metadata. The input property is bound to a DOM property in the template. During change detection, Angular automatically updates the data property with the DOM property's value.

<cite>Input decorator</cite>[Angular docs](https://angular.io/api/core/Input)
{: .small}

The property `phone-number` is the name part of the public API while `phone` is the private name, *an implementation detail*{: .italic-red-text}. Since it can change, the code must be compile every time to emit, in case, an error if there is a property name mismatch. For this reason current Angular version must rely on *global compilation*.

Angular Ivy instead relies on the *public API*, so library code can be compiled and safely shipped to npm.

##### Browser property

> The input property is bound to a DOM property in the template.

Basically

> When the browser loads the page, it “reads” (another word: “parses”) the HTML and generates DOM objects from it. For element nodes, most standard HTML attributes automatically become properties of DOM objects.

<cite>Attributes and properties</cite>[javascript.info](https://javascript.info/dom-attributes-and-properties)

Angular compiler transforms the decorators and the templates into JavaScript instructions not only to create element into the DOM but also *extra content properties and attributes* used by runtime to *"keep alive"* the application. 

![image-center](/assets/images/posts/angular-ivy-intro/input-property.png ){: .align-center}

### Flexibility

// todo

tsickle@0.37.1 requires a peer of typescript@~3.6.4 but none is installed. You must install peer dependencies yourself.

## Angular Ivy Build Pipeline

The compilation of anf Angular application is just half of the process since the libraries the application depends on have to be make compatible with the *new runtime*.

`ngcc` (Angular compatibility compiler) is a new compiler that convert and compile the libraries. Libraries compatible with `ViewEngine`, the previuos *rendering engine* of Angular, are converted into Ivy instructions so that the *"library can partecipate in the Ivy runtime"* and be fully compatible.

The new compiler has been implemented to make libraries compatible with the new format without oblige mantainers to rewrite important parts of them and especially becase not all the applications need to be compatible with Ivy.

In Angular version 9 Ivy is enable for application only and `ngcc` is used to convert existing libraries making them Ivy compatible. Over time application will start to become more and more Ivy compatible and so the libraries, then `ngcc` will not be anymore necessary. Libraries can be coverted on the fly into Ivy compatible libraries during the *build or installation process*.

Incremental transition from version 9 to version 11 where `ngcc` will be only for some few cases. (add img)

- Angular 9: app on Ivy (opt-out) and libraries VE compatible
- Angular 10: stabilise Ivy instruction set and libraries ship Ivy code
- Angular 11: `ngcc` a backup solution for obsolete libraries or not been updated yet

`ngcc-validation` [project](https://github.com/angular/ngcc-validation) is the way Angular team test the libraries compatibility.

Ivy is not just about improving the built time and the size but also a predectible merging of styles on the component. Angualr since version 2 has a compenent lazy loading feature but just at *router level*. To do at *component level* requires a lot of boilerplate code and some patches to make it work.

A simple example is just click on an image, lazy load the bundle and add the component to the view improving then the speed of an application. Fro the time being still need to pass via the `ComponentFactoryResolver` to load the component, ideally it will be:

```javascript
@Component(...)
export class AppComponent{
  constructor(
      private viewContainer: ViewContainer,
      private cfr: ComponentFactoryResolver) {

    // lazy click handler
    async lazyload() {
      // use the dynamic import
      const {LazyComponent} = await import('./lazy/lazy.component');
      this.viewContainer.createComponent(LazyComponent);
    }
  }
}
```

So basically avoid to resolve the lazy component into a factory as needed today:

```javascript
this.viewContainer.createComponent(this.cfr.resolveComponentFactory(LazyComponent));
```

## Try Ivy

Enable Ivy on your project running `ng update @angular/cli@next @angular/core@next`, if something goes wrong open an issue in the bug report :-)
Being a library author compile with View Engine and use the `ng-packagr`, compatibility will be ensured.

## Deep dive into the Angular compiler

> Why does Angular have a compiler at all?
> Alex Rickabaugh - Angular Connect 2019

Basically what the compiler actually doing? The goal is to make an application smaller and faster.

> Main job of the compiler is to turn the template you write into the code that runs at runtime.

In Angular the developer write templates *declaratively*, what to render, the binding, etc. *but not how it happens at runtime*, there is not description about how the change detection mechanism works. The browser does not udnerstand the *declarative template* so it has to be translated into something a browser can understand.

The compiler takes the *declarative Angular syntax* and turn it into *imperative code*.

Why, again, a compiler is required? not possible to do by hand?

- lot of boilerplate code, better to focus on the business logic than how things work under the hood;
- Angular team can optimize and improve by release w.r.t. browser evolution the iperative code.

### Compiler can transform in two ways: JIT and AOT

#### JIT (development mode)

Imperative code is generated at runtime. TypeScript is transpiled (`tsc`) into JavaScript, decorators (`@Component`) invoke the compiler.

#### AoT

Imperative code is generated at build time. `ngc` compiles TypeScript code into JavaScript and pre-compile the decorators into imperative code to be rendered in the browser avoiding the cost of runtime compilation.

### Angular architecture

To understand how the Angular compiler works and the all process, it is important to know how TypeScript works since the *Angular compiler is based on TypeScript*.

The TypeScript compiler has *three phases*: Program Creation, Type Checking and Emit. The Angular compiler in built on top of the TypeScript one.

Angular compiler uses the TypeScript ones plus additional phases. Let's see all the *compilation phases*.

#### 1. Program creation (TypeScript)

TypeScript process to discover all the application source files, it starts with the `tsconfig.json`. TypeScript gets initial files of the application, then via the `import` statements discover other files in the application itself or in the libraries. Once all the files have been collected, TypeScript can understand correctly check the types of the application.

In the Angular compiler some more is added to this process, some extra steps that allows, for instance, to import `ngfactory` files to do some adavnced things in Angular.

#### 2. Analysis (Angular)

In this phase the Angular compiler takes all the TS files, class by class, and looks, one by one, for the classes decorated with Angular decorators, basically look for classes with *"Angular things"*. For instance it understands if something is a component or part of a template. The process is gathering information *in isolation*, for instance it gather information about a *component* but it does not know the *module* yet.

#### 3. Resolve (Angular)

Once all the information of classes has been collected, it looks again at the whole application. The compiler now looks at things in a larger picture.

## References

[ac-2019-keynote]: https://www.youtube.com/watch?v=6Zfk0OcFGn4&list=PLAw7NFdKKYpE-f-yMhP2WVmvTH2kBs00s
[ac-2019-alex-rickabaugh]: https://www.youtube.com/watch?v=anphffaCZrQ&list=PLAw7NFdKKYpE-f-yMhP2WVmvTH2kBs00s&index=2
[nrwl-ivy]: https://blog.nrwl.io/understanding-angular-ivy-incremental-dom-and-virtual-dom-243be844bf36
