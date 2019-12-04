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

Angular Ivy is the new *rendering architecture* that comes, by default, with version 9. The Angular rendering architecture is not new to a completely new revamp, Angular 2.0, Angular 4.0 and now Angular 9.0 have introduced new compilers and rendering engines.

Currently Angular stable version is 8.3.9 and Angular 9 is in RC4.

## Rendering Architecture

What is a rendering architecture? It is the pair compiler:runtime that allows a developer to write templates via Angular declarative syntax and run the optimized bundled application in a web browser.

- **View Engine** has been introduced with Angular version 4 and still used in version 8, but some limitations has been identified
  - *no tree-shakable:* both the `Hello World` application and a very complex one are supported by the full runtime. If the internationalization module is not used it is included in the runtime as well, currently it cannot be tree-shaked.
  - *no incremental compilation:* Angular compilation is global, it involves not only the application but the libraries as well.
- **Ivy** will be activated by default in version 9 and should solve the View Engine current issues and it enables new possibilities, "Ivy is an enables (Igor Minar)":
  - *simplify* how Angular works internally;
  - *three-shakable:* the `Hello World` application does not required the full Angular runtime and wil be bundled in only 4.7 KB;
  - *incremental compilation* is not possible so the compilation is faster than ever and `--aot` can be now used even during development mode (advice from Angular team).

## Incremental DOM vs. Virtual DOM

> Every component gets compiled into a series of instructions. These instructions create DOM trees and update them in-place when the data changes.

Each compiled component has two main sections: one for *view creation* and one for *change detection*. The first one will be called when the component will be rendered the first time, the second one when changes involving the component will happen. Change detection is basically a function that is added at compile time, it is something the developer is not aware of when implement the component, the Angular template declarative syntax makes use only of *binding*.

Incremental DOM enables better bundle size and memory footprint so that applications can really well perform on mobile devices.

### Virtual DOM

React as Vue make use the concept of Virtual DOM to create a component and re-render it when change detection happens.

Render the DOM is a very expensive operation, when a component is added to the DOM or changes, the repaint operation has to take place. Virtual DOM strategies aims to reduce the amount of work on the real DOM and so the number of time the user interface needs to be repainted.

DOM manipulations happens every time a new component is going to be added, removed or changed from the DOM, so instead of operating directly on the DOM it operates on a JSON object called Virtual DOM. When a new component is added or an existing one removed, a new Virtual DOM is added, the difference between virtual DOMs is computed and a serie of transformation is applied to the real DOM.

![image-center](/assets/images/posts/angular-ivy-intro/virtual-dom.png ){: .align-center}

React advices to use JSX, a *syntax extension* to JavaScript, to define *React elements*, it is not a template languages. A template is an enriched JavaScript expression that is interpreted at runtime. Plain JavaScript can also be used instead of JSX.

Virtual DOM technique has some disadvantages:

- create a whole tree every time a change happens (add or remove a node), so the memory footprint is quite important;
- an interpreter is required as long as the *diff* algorithm to compute the difference among the Virtual DOMs. At compile time it is not known which functionalities are required to render the application, so the whole thing has to be shipped to the browser.

### Incremental DOM

It is the foundation of the new rendering engine. Each *component template* gets compiled into creation and change detection instructions: one to add the component to the DOM and the other one to updated the DOM *in-place* when data changes.

**Internals**
Instructions are not interpreted by the Angular runtime, the rendering engine, but *the instructions are the rendering engine*.
{: .notice--internal}

Since the runtime does not interpret the template component instructions, but the *component references instructions* it is quite easy to remove those instructions that are not referenced. At compile time the unused instruction can be excluded from the bundle.

Moreover the amount of memory required to render the DOM is proportional to the size of the component.

**Tip**
The compiled template component references some instructions of the Angular runtime which holds their implementation.
{: .notice--info}

## Angular Ivy Compilation Example

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

it gets compiled into the following code

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

The Angular decorators are compiled into *static fields* in the decorated class. So `@Component` becomes `ngComponentDef` static field. Back to View Engine, the `ngc` compiler produces `.ngfactory` separated files for each component and modules. With Ivy the code produced by the compiler is move into *component class static fields*.

The `elementStart()`, `elementEnd()`, etc are the component referenced instructions, *every component is its own factory*, the framework does not interpret the component.

**Internals**
The View Engine runtime is a *single monolith interpreter* that is not tree-shakable and has to be entirely shipped to the browser. Unlike, Angular Ivy runtime is an *instruction set* that is a *set of rendering functions* like an assembly language for templates.
{: .notice--internal}

## What Angular Ivy enables

Angular Ivy is an enabler. Simplifying how Angular works internally and the compilation process enable really interesting features and makes Angular easily opened to future improvement.

### Tree-shaking

Tree-shaking is the operation or removing *dead code* from the bundle so if the application does not use all the functions from a library, they can be omit from the bundle making it smaller.

Dead code comes from libraries Angular included. Angular CLI is powered by Webpack uglify plugin as tree-shaker that, in turn, receives information from Angular Build Optimizer Plugin about which code is used and which not. This plugin, after compilation, can gather information about component referenced instructions hence can instruct Uglify about what to include in the bundle.

Angular compiler simply does not emit those instructions so they are not included into the bundle.

While the `@angular/core` framework is tree-shakable, the View Engine runtime is not, it cannot be broken into small pieces and this is mainly due to the static `Map<Component, ComponentFactory>`.

### Incremental compilation

The Angular compilation pipeline started by `ng build prod --aot` is composed by five phases where the `tsc` and the `ngc` generates the template factories. `ngc` compiles the libraries as well. Ivy enables *Incremental compilation* that is libraries can be compiled and deployed on npm.

### Locality

*Locality is a rule*. Ivy enable an Angular application to safely refer to components and directives *public API*, no more needed to know much about dependencies since *extra information* are added to `.d.ts` files.

#### Example

Consider the following component snippet:

```javascript
@Input('property') field: string;
```

where `property` is the name part of the public API while `field` is the private name, *an implementation detail*. Since it can change, the code must be compile every time, so for this reason current Angular version must rely on *global compilation*.

Angular Ivy instead relies on the *public API*, so library code can be compiled and safely shipped to npm.

### Flexibility

// todo

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

Imperative code is generated at rutime. TypeScript is transpiled (`tsc`) into JavaScript, decorators (`@Component`) invoke the compiler.

#### AoT

Imperative code is generated at build time. `ngc` compiles TypeScript code into JavaScript and pre-compile the decorators into imperative code to be rendered in the browser avoiding the cost of rutime compilation.

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

ac-2019-keynote:https://www.youtube.com/watch?v=6Zfk0OcFGn4&list=PLAw7NFdKKYpE-f-yMhP2WVmvTH2kBs00s
ac-2019-alex-rickabaugh:https://www.youtube.com/watch?v=anphffaCZrQ&list=PLAw7NFdKKYpE-f-yMhP2WVmvTH2kBs00s&index=2