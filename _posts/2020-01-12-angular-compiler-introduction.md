---
layout: single
title: "Angular 9 compiler"
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

> Why does Angular have a compiler at all?
> Main job of the compiler is to turn the template you write into the code that runs the Angular runtime.
> 
> Alex Rickabaugh - Angular Connect 2019

In Angular, the developer write templates *declaratively* that is *what to render*{: .italic-red-text}, the binding, etc. *but not how it happens at runtime*{: .italic-red-text}, there is not description about how the change detection mechanism works. The Angular runtime does not understand the *declarative template syntax* so it has to be translated into something the runtime can run into a browser.

The compiler takes the *declarative Angular syntax*{: .italic-red-text } and turns it into *imperative code*{: .italic-red-text }.

Why, again, a compiler is required? not possible to do by hand?

- Lot of boilerplate code, better to focus on the business logic than how things work under the hood.
- Angular team can optimize and improve by release w.r.t. browser evolution the imperative code.

## Lingo

- *Angular Compiler:*{: .italic-violet-text} part of the Angular rendering architecture that transforms *templates* and *decorators* into something that the Angular runtime can understand.
- *Angular Runtime:*{: .italic-violet-text} part of the Angular rendering architecture that runs an Angular application.
- *Angular template declarative syntax:*{: .italic-violet-text} describes *what*{: .italic-red-text } the view has to look like, *what*{: .italic-red-text } the view has to display, but not *how*{: .italic-red-text }.
- *Angular application imperative code:*{: .italic-violet-text} describes *how*{: .italic-red-text } the view has to be rendered via a sequence of JavaScript instructions/commands.
- *Metadata:*{: .italic-violet-text} set of data enclosed into a decorator that describes the entity represented by the decorator itself. For instance a component has the selector, the template, etc. Metadata are then reused by the compiler and put into the definition files `.d.ts`, the component API. Basically the metadata help to preserve the information removed from `.js` files.

## Quick overview

The work done on the new Angular compiler can be divided in three categories as stated in the [implementation status](https://github.com/angular/angular/blob/master/packages/core/src/render3/STATUS.md):
- `@angular/compiler-cli`: TypeScript transformer pipeline which includes two CLI tools
  - `ngtsc`: the Angular TypeScript compiler which looks for the [Angular decorators](https://github.com/angular/angular/blob/master/packages/core/src/render3/STATUS.md#decorators) like `@Component` and substitute them with their specific Angular Runtime instructions/counterparts like `ɵɵdefineComponent`.
  - `ngcc`: the Angular Compatibility Compiler which converts pre-ivy modules into ivy-module, can be even run as part of a code loader like Webpack to have packages from `node_modules` transpiled on-the-fly.
- `@angular/compiler`: Ivy Compiler which converts decorators into Ivy.
- `@angular/core`: decorators which can be converted by the `@angular/compiler`.

## Angular Ivy compiler model

The [Ivy model](https://github.com/angular/angular/blob/master/packages/compiler/design/architecture.md#the-ivy-compilation-model) foresees to compile the Angular decorators like `@Injectable`, `@Component`, etc into *static properties*{: .italic-red-text }.

All the decorators do not need global knowledge of the application, except `@Component` which requires information coming from `@NgModule`. In the module other selectors used by the *component template*{: .italic-red-text } are declared and so [the transitive closure of the exports of that module imports](https://github.com/angular/angular/blob/master/packages/compiler/design/architecture.md).

Without those information the *component def*{: .italic-red-text } (`ɵcmp`) cannot be correctly generated.

Consider the `Welcome to Angular!` application:

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

The previous code is characterized by *decorators*{: .italic-red-text } like `@Component` and `@Input`, part of the *Angular template declarative syntax*{: .italic-red-text } that provides developers with an easy template grammar to both

- write *control flow statements*, like the `for loop` and `if` *conditional statement*,
- and make *data binding* between the variables declared in the controller and used in the templates.

**Tip**
For most of the developers the *binding* is just the use of the same variable name between the template and the controller, but there is a mechanism that manages the *change detection* at runtime. This is a great thing and the Angular compiler automatically adds this mechanism. Less boilerplate code, templates are mode readable and less error prone.
{: .notice--info}

The *Angular runtime*{: .italic-red-text } is *a collection of JavaScript instructions/functions* able to render a component template into the DOM and to answer to change detection when something in the model has changed (MVC). Everything must be in JavaScript, ready to be run, so the Angular declarative syntax is translated into these instructions.

To make it more clear try to develop a standard Web Component: when the developer has to deal with the template he must deal with the DOM API to create an element and attach it to the DOM, write some code to detect changes in the model and update the view. In the Angular code there is not trace of these operations since a template is translated into *JavaScript imperative code*{: .italic-red-text }, a series of JavaScript instructions, part of the Angular runtime, that, when invoked, creates the component in the browser.

Most of the tedious and repetitive job is done by the Angular compiler in conjunction with the Angular runtime.

### Compiler enables decoupling

The developer just writes the Angular modules and components, that is the *what*{: .italic-red-text }, via the Angular declarative syntax but it does not know/care about the *how*{: .italic-red-text }, that is how things are executed at runtime: *compiler enables to decouple the what from the how*{: .italic-red-text }.

This approach has multiple advantages:
- [minimize or eliminate side effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) making the developer life simpler;
- if the Angular rendering architecture evolves, small or null changes are required to use the latest version;
- browsers change more and more often as the EcmaScript hence the *how*{: .italic-red-text } the templates are rendered can change accordingly optimizing web performance for instance;
- templates and decorators can be compiled differently w.r.t. the platform where the code is run such ES5 or ES6 with module support.

## Simple project setup

Angular 9 has been recently released, generate a simple project with Angular 9 running:

```bash
npm install -g @angular/cli     // to install the Angular CLI
ng new angular-nine-ivy         // or the name you want
```

In case you already have a workspace with some changes you have to commit or stash them, otherwise add the flag `--allow-dirty`.

If you already have the old Angular CLI, it seems the better way is to uninstall the old one and the install the new globally.

### Let's compile

There are two compiler entry points in the `@angular/compiler-cli`: `ngtsc` and `ngcc`.

#### `ngtsc`

It is the [TypeScript to JavaScript transpiler that reifies the Angular decorators into static properties](https://github.com/angular/angular/blob/master/packages/compiler/design/architecture.md#high-level-proposal). `ngc` works like the `ngtsc` when Ivy is enabled.

#### `ngcc`

It is the Angular Compatibility Compiler that process the NPM library code from the `node_modules` folder producing equivalent library code Ivy compatible. `ngcc` can also be run by a code loader like Webpack to get packages transpiled on demand instead of written in the `node_module` package folder.

#### `ngc`

Open the `package.json` and add the `ngc` script:

```javascript
"scripts": {
    "ng": "ng",
    ...
    "ngc": "ngc"
}
```

In the `tsconfig.json` set `"declaration": true,` in order to have the `.d.ts` files as well, then run:

```bash
$ npm run ngc
```

The result of the component compilation is located at `dist\out-tsc\src`. The `Welcome to Angular!` component is translated into:

```javascript
import { Component, Input } from '@angular/core';
import * as i0 from "@angular/core";
export class AppComponent {
    constructor() {
        this.title = 'Angular';
    }
}
AppComponent.ɵfac = function AppComponent_Factory(t) { return new (t || AppComponent)(); };
AppComponent.ɵcmp = i0.ɵɵdefineComponent({ type: AppComponent, selectors: [["app-root"]], inputs: { title: "title" }, decls: 3, vars: 1, consts: [[2, "text-align", "center"]], template: function AppComponent_Template(rf, ctx) { if (rf & 1) {
        i0.ɵɵelementStart(0, "div", 0);
        i0.ɵɵelementStart(1, "h1");
        i0.ɵɵtext(2);
        i0.ɵɵelementEnd();
        i0.ɵɵelementEnd();
    } if (rf & 2) {
        i0.ɵɵadvance(2);
        i0.ɵɵtextInterpolate1(" Welcome to ", ctx.title, " ");
    } }, styles: [""] });
/*@__PURE__*/ (function () { i0.ɵsetClassMetadata(AppComponent, [{
        type: Component,
        args: [{
                selector: 'app-root',
                template: `
    <div style="text-align:center">
      <h1>
        Welcome to {{ title }}
      </h1>
    </div>
  `,
                styleUrls: ['./app.component.css']
            }]
    }], null, { title: [{
            type: Input
        }] }); })();
```

and then the `app.component.d.ts` definition file:

```javascript
import * as i0 from "@angular/core";
export declare class AppComponent {
    title: string;
    static ɵfac: i0.ɵɵFactoryDef<AppComponent>;
    static ɵcmp: i0.ɵɵComponentDefWithMeta<AppComponent, "app-root", never, { "title": "title"; }, {}, never>;
}
```

**Attention**
By default Ivy is enabled, if you disable it you will get a different compilation output.
{: .notice--warning}

### JIT vs. AoT

Compiler can work in *JiT (Just in Time)* mode, it is delivered along with the application and compiles at runtime. *AoT (Ahead of Time) compilation* instead compiles everything at build time making the application faster and it does not required to ship the compiler with the application.

With Angular 9, thanks to Ivy, the compilation is faster and by default `"aot": true`.

## Compilation models

Both TypeScript and Angular Compiler preserve important metadata that cannot be part of the emitted `.js` files. Angular enhances the `.d.ts` files with framework specific metadata to be reused for better component type checking.

### TypeScript compilation model

JavaScript code has no type information, it is just ready to be executed in the browser. The TypeScript compiler has transformed all the `.ts` source files but typing information are not totally lost, the definition files help to preserve the interface for future use.

For instance the file `library.ts` is compiled into `library.js` and `library.d.ts` is produced, it describes the *interface*{: .italic-red-text } or *public API*{: .italic-red-text } of the library.

The definition file brings *type information*{: .italic-red-text } to help TypeScript to static type check the usage of the library. Fo example, when an application declare a npm dependency library can then import functionality:

```javascript
import {AwesomeLib} from 'awesome-lib';
// use the lib
```

TypeScript will static type check the use of the library in the `app.js` exploiting the `library.d.ts` definition file:

```javascript
declare class AwesomeLib {
    awesomeMethod(): string;
}
```

### Angular Ivy compilation model

An Angular component has some useful information declared in the decorator like the `selector`. These information are really important to use the component elsewhere and static type check the code.

The Angular Ivy compiler has been improved to not waste this information and to enrich the definition file public interface. For instance, the following Angular component:

```javascript
@Component({
    selector: 'awesome-comp'
})
export class AwesomeComponent {
    @Input() value: string;
}
```

will be compiled into JavaScript and the metadata will enrich the definition file as following:

```javascript
export declare class AwesomeComponent() {
    value: string;
    static ngComponentDef: ng.ComponentDef<
        AwesomeComponent,
        `awesome-comp`,
        {value: 'value'}
    >;
}
```

The definition file became the *component public API*{: .italic-red-text }. The compiler can exploit it in order to type check the code that will use it:

```javascript
<awesome-comp [value]="a value">
    just a component
</awesome-comp>
```

## TypeScript transpiler architecture

The following diagram, [thanks to Angular team](https://github.com/angular/angular/blob/master/packages/compiler/design/architecture.md#typescript-architecture), shows the normal `tsc` flow and the steps to transpile a `.ts` file into the `.js` one.

```bash
                                                                |------------|
                           |----------------------------------> | TypeScript |
                           |                                    |   .d.ts    |
                           |                                    |------------|
                           |
|------------|          |-----|               |-----|           |------------|
| TypeScript | -parse-> | AST | ->transform-> | AST | ->print-> | JavaScript |
|   source   |    |     |-----|       |       |-----|           |   source   |
|------------|    |        |          |                         |------------|
                  |    type-check     |
                  |        |          |
                  |        v          |
                  |    |--------|     |
                  |--> | errors | <---|
                       |--------|
```

- **parse:** recursive descent parser that produces the abstract syntax tree (AST);
- **type check:** perform type analysis on every file, report found error. Not modified by the `ngtsc`;
- **AST to AST:** remove type declarations, convert class into ES5, `async` methods, etc.

### Extension points

TypeScript `tsc` provides some extension points to alter its output:

- `CompilerHost.getSourceFile` to modify the source;
- `CustomTransformers` to alter the list of transforms;
- `WriteFileCallback` to intercept the output before it is written.

## Compilation steps

The `ngtsc` is a wrapper around the `tsc`, the TypeScript compiler, that extends and modify the normal compilation flow.

![image-center](/assets/images/posts/angular-compiler-introduction/compiler-steps.png){: .align-center}

**Tip**
The TypeScript transpiler cannot compile the Angular templates and decorators, so the Angular compiler kicks in to *reify Angular decorators into static properties*. Once finished the TypeScript compiler can go on producing JavaScript code. In other words *the Angular compiler allows the code written in Angular declarative syntax to participate to the TypeScript compilation process*{: .italic-blue-text }.
{: .notice--info}

### 1. Program creation

Starting from the `tsconfig.json` file, the TypeScript process discover the application source files via the `import` statements.

### 2. Analysis

The Angular compiler takes all the `.ts` files collected and, class by class, looks for Angular declarative syntax code, basically Angular things. The compiler gathers isolation information about components for instance, but not about modules. *Remember*, for the `@Component` the compiler requires a global knowledge about the module due transitive closure resolution of the exports of the component module template imports used in the component template.

### 3. Resolve

The Angular compiler looks again at the whole application but this time in a larger picture including modules as well. All the code now is understandable and parsable by the next step of the TypeScript compiler. Optimizations will take place at this step.

### 4. Type checking

TypeScript checks error in the application, templates included, that now are a series of imperative instructions.

### 5. Emit

The most expensive operation in the compilation process, the TypeScript code is transformed into JavaScript ready to be run by the browser. Angular component classes have now only imperative code to describe what a template looks like.

## Compiler features

Angular compiler has many interesting features, some of them have been enhanced and improved with the new Angular Ivy architecture. Let's see some of them.

### Module compilation scope

The module scope allows the compiler to resolve *uniquely* the Angular components used by the application. Consider the following application module:

```javascript
@NgModule({
    declarations: [
        AppComponent,
        HelloComponent
    ]
})
export class AppModule {}
```

The module decorator property `declarations` is the *module compilation scope*{: .italic-red-text }, it holds an array of Angular components used in the application templates. The `HelloComponent` is a component coming from a library with its own definition file enriched with metadata as seen before.

The developer declares the willingness to use the component in the array, then can use the component adding the corresponding selector in one of the application templates.

The compiler can than uniquely match the component, verify the use of the selectors along with its attributes.

### Export compilation scope

A module can be used to export Angular components as a method to make visible the external world some implemented components.

**Tip**
The module pattern is quite frequently used in computer languages as a way to separate a complex application in small chunks that can be reused elsewhere. Just think about ESM or CommonJS etc. Angular provides a way to create a modular application via the module concept as well. A module is used also to hide implementation details and to select what should be *public*. The `exports` property is the way chosen to make Angular component public so re-usable by the rest of the world.
{: .notice--info}

In the following example the `HelloModule` both declares and exports the `HelloWorld` component. The module library just implements one component and exports it to all the applications that want to use it:

```javascript
@NgModule({
    declaration: [HelloComponent],
    exports: [HelloComponent]
})
export class HelloModule {}
```

When an application wants to use the `HelloWorld` component has just to declare it:

```javascript
@NgModule({
    declarations: [
        AppComponent,
        HelloComponent
    ]
})
export class AppModule {}
```

The compiler then can:

- uniquely find the component definition;
- figure out from the template which components are used;
- generate the code and reference it accordingly;
- help the tree-shaker to remove things that are not referenced.

### Partial evaluation

> Compiler actually attempt to almost run TypeScript code in decorators
>
> Alex Rickabaugh - Angular Connect 2019
{: .cite }

Angular compiler contains almost a complete *TypeScript interpreter*. In the following example:

```javascript
import {SOME_MODULES} from './some_module';

@NgModule({
    declarations: [HelloComponent, ByeComponent],
    exports: [HelloComponent, ByeComponent],
    imports: [...SOME_MODULES]
})
export class AnotherModuel {...}
```

the compiler can follow and evaluate the *import references*. Some evaluation are just partial since there are not enough information at compiler time.

For instance:

```javascript
import {SOME_MODULES} from './some_module';

@NgModule({
    imports: [SOME_MODULES.modules]
})

export const SOME_MODULES = {
    modules: [HelloModule, ByeModule],

    // not available at compile time
    viewportSize: {
        x: document.body.scrollWidth,
        y: document.body.scrollHeight
    }
}
```

only *at runtime* some values will available. This value are *dynamic expressions*. The compiler sees the previous object more or less like this:

```javascript
SOME_MODULES: {
    "modules": [
        Reference(HelloModule),
        Reference(ByeModule)
    ],
    "viewportSize": {
        x: DynamicValue(document.body.scrollWidth),
        y: DynamicValue(document.body.scrollHeight)
    }
}
```

where:
- `SOME_MODULES` is an object with two properties;
- `DynamicValue` is an *indicator* to say *"cannot get past"*.

The compilation does not stop, it goes on since a situation like this could be quite common.

```javascript
import {SOME_MODULES} from './some_module'
@Component({
    styles: [`
        :host {
            width: $SOME_MODULES.viewportSize.x
        }
    `]
})
```

While the compilation goes on for `modules` properties, it cannot for `viewportSize` since the value cannot be figured out. An explicative error message is produced about styles.

### Template type checking

A template and expressions

```html
<account-view
    [account]="getAccount(user.id, 'primary') | async">
</account-view
```

they are compiled into TypeScript code becoming *type check blocks*{: .italic-red-text }. So blocks are sent to TypeScript compiler and possible errors are then return *in the context of the template*.

```javascript
function typeCheckBlock(ctx: AppComponent) {
    let cmp!: AccountViewCmp;
    let pipe!: ng.AsyncPipe;

    cmp.account /*273,356*/ = (pipe.transform(
        ctx.getAccount((ctx.user /*311,315*/).id /*311,318*/,
        "primary" /*320,329*/)
        /*300,331*/)
    /*300,338*/) /*289,339*/;
}
```

How is it possible to return an error in the context of a template? Well, code is translated with additional `offset comments` that allows then the contextualization.

#### Example

```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div style="text-align:center">
      <h1>
        {% raw %}Welcome to {{ title }}!{% endraw %}

        <tr *ngFor="let hero of heroes">
            <td>{% raw %}{{hero.name}}{% endraw %}</td>
        </tr>
      </h1>
    </div>
  `,
  styleUrls: []
})
export class AppComponent {
  @Input() title = 'Angular';

  heroes = 'fake array';
}
```

Angular version 9 *with Ivy enabled* maintains the behavior of the `fullTemplateTypeCheck` flag and introduces a strict mode with the flag `strictTemplates` that goes beyond the Angular 8 type checker. Activate the strict mode in the `tsconfig.json`:

```javascript
...
"angularCompilerOptions": {
    "fullTemplateTypeCheck": true,
    "strictTemplates": true,
    ...
}
```

Re-run the `npm run ngc` and get a full better error reported by the compiler:

```bash
src/app/app.component.html:7:13 - error NG2339: Property 'name' does not exist on type 'string'.

7       <td>{% raw %}{{hero.name}}{% endraw %}</td>
              ~~~~~~~~~

  src/app/app.component.ts:5:16
    5   templateUrl: './app.component.html',
                     ~~~~~~~~~~~~~~~~~~~~~~
    Error occurs in the template of component AppComponent.

```

More details available on the [template type-check page](https://angular.io/guide/template-typecheck).

## Conclusions

The Angular 9 Ivy rendering architecture introduces a new compiler and a new rendering engine not only to exploit the incremental DOM technique but also a more powerful compiler.

Ivy is enabled by default, `--aot` is the default way of developing since the new compiler is faster than the previous one. Having AoT mode enable by default reduces also the risk of discrepancies between development and production code.

Ivy compiler goes even further, it has a better type-check making the reported error much more explicative and easier for the developer to identify the root cause of the issue.

We all are looking forward to see the new compiler applied to Angular Elements.

## References

- [Compiler architecture (Ivy)](https://github.com/angular/angular/blob/master/packages/compiler/design/architecture.md)
- [The theory of Angular Ivy, Alex Richabaugh](https://www.youtube.com/watch?v=isb5Ef6yI48)
- [Angular Ivy by example](https://www.youtube.com/watch?v=MMPl9wHzmS4)
- [Deep dive into the Angular Compiler, Alex Richabaugh](https://www.youtube.com/watch?v=anphffaCZrQ)
- [deep-dive-compiler](https://blog.angularindepth.com/a-deep-deep-deep-deep-deep-dive-into-the-angular-compiler-5379171ffb7a)
- [deep-dive-compiler video](https://www.youtube.com/watch?v=QQ2plVD0gDI&feature=youtu.be&t=27m)
- [Ivy engine in Angular](https://indepth.dev/ivy-engine-in-angular-first-in-depth-look-at-compilation-runtime-and-change-detection/)

### Angular Compiler Architecture

- [Angular Compiler status](https://github.com/angular/angular/blob/master/packages/core/src/render3/STATUS.md)
- [Angular Compiler Architecture](https://github.com/angular/angular/blob/master/packages/compiler/design/architecture.md)
- [Angular separate compilation](https://github.com/angular/angular/blob/master/packages/compiler/design/separate_compilation.md)
- [Angular compiler implementation status](https://github.com/angular/angular/blob/master/packages/core/src/render3/STATUS.md)

### Tools

- [Tsickle](https://github.com/angular/tsickle)
- [Terser](https://github.com/terser/terser)