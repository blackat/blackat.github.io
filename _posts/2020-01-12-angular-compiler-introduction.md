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
> Alex Rickabaugh - Angular Connect 2019

Basically what the compiler actually doing? The goal is to make an application smaller and faster.

> Main job of the compiler is to turn the template you write into the code that runs at runtime.

In Angular the developer write templates *declaratively*, what to render, the binding, etc. *but not how it happens at runtime*, there is not description about how the change detection mechanism works. The browser does not udnerstand the *declarative template* so it has to be translated into something a browser can understand.

The compiler takes the *declarative Angular syntax* and turn it into *imperative code*.

Why, again, a compiler is required? not possible to do by hand?

- lot of boilerplate code, better to focus on the business logic than how things work under the hood;
- Angular team can optimize and improve by release w.r.t. browser evolution the iperative code.

## Lingo

- *Angular Compiler:*{: .italic-violet-text} part of the Angular rendering architecture that transforms templates and decorators into something that the Angular runtime can understand.
- *Angular Runtime:*{: .italic-violet-text} part of the Angular rendering architecture that runs an Angular application.
- *Angular template declarative syntax:*{: .italic-violet-text} describes *what* the view has to look like, *what* the view has to display data, but not *how*.
- *Angular application imperative code:*{: .italic-violet-text} describes *how* the view has to be rendered via a sequence of JavaScript instructions/commands.

## Why Angular has a compiler

The quick answer is to transform templates and decorators into something the *runtime* can understand.

The *Angular rendering architecture*{: .italic-red-text } is composed by the *Angular compiler*{: .italic-red-text } and the *Angular runtime*{: .italic-red-text }: the former transforms the application source code into something the latter can run.

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

The previous code is made of *decorators*{: .italic-red-text } like `@Component`, `@Input` and *Angular template declarative syntax*{: .italic-red-text } that provides developers with an easy grammar to both write control flow statements, like the `for loop` and `if` conditional statement, and make data binding between the variables declared in the controller and used in the templates.

**Tip**
For most of the developers the binding is just the use of the same variable name between the template and the controller without thinking about the mechanism that manages the *change detetction* at runtime. This is a great thing, the Angular compiler adds this mechanism. Less boilerplate code, templates are mode readable and less error prone.
{: .notice--info}

The Angular runtime is *a collection of JavaScript instructions/functions* able to render a component template into the DOM and to answer to change detection when something in the model has changed (MVC). So everything must be in JavaScript, ready to be run, so the Angular declarative syntax must be then translated into this instructions.

To make it more clear try to develop a standard Web Component: when the developer has to deal with the template he must deal with the DOM API to create an element and attach it to the DOM, write some code to detect changes in the model and update the view. In the Angular code there is not trace of these operations since a template is translated into *JavaScript imperative code*{: .italic-red-text }, a series of JavaScript instructions, part of the Angular runtime, that, when invoked, create the component in the browser.

Most of the tedious and repetitive job is done by the Angular compiler in conjunction with the Angular runtime.

### Compiler enables decoupling

The developer just writes the Angular modules and components, the *what*, via the Angular declarative syntax but it does not know/care about the *how* things are run at runtime: *compiler enables to decouple the what from the how*{: .italic-red-text }.

This approach has multiple advantages:
- [minimize or eliminate side effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) making the developer life simpler;
- if the Angular rendering architecture evolve small or null changes are required to use the latest version;
- browsers change more and more often as the EcmaScript hence the *how* the templates are rendered can change accordingly optimizing web performance for instance;
- templates and decorators can be compiled differently w.r.t. the platform where the code is run such ES5 or ES6 with module support.

### JiT vs. AoT

Compiler can work in *JiT (Just in Time)* mode, it is delivered along with the application and compiles at runtime. *AoT (Ahead of time) compilation* instead compiles everything at build time making the application faster and it does not required to ship the compiler with the application.

## Compilation steps

![image-center](/assets/images/posts/angular-compiler-introduction/compiler-steps.png){: .align-center}

The Angular compilation process is built on top the TypeScript one which is made of three steps.

**Tip**
The TypeScript compiler cannot compile the Angular templates and decorators, so the Angular compiler kicks in doing the job the previous one cannot do. Once finished the TypeScript compiler can go on, using now the outcome of the `ngc`. In other words the Angular compiler allows the code written in Angular declarative syntax to participate to the TypeScript compilation process.
{: .notice--info}

#### Program creation

Starting from the `tsconfig.json` file, the TypeScript process discover the application source files via the `import` statements.

#### Analysis

The Angular compiler takes all the `.ts` files collected and, class by class, looks for Angular declarative syntax code, basically Angular things. The compiler gathers isolation information about components for instance, but not about modules.

#### Resolve

The Angular compiler looks again at the whole application but his this in a larger picture including modules as well. All the code now is understandable and parsable by the next step of the TypeScript compiler. Optimizations will take place at this step.

#### Type checking

TypeScript checks error in the application, templates included, that now are a series of imperative instructions.

#### Emit

The most expensive operation in the compilation process, the TypeScript code is transformed into JavaScript ready to be run by the browser. Angular component classes have now only imperative code to describe what a template looks like.

## TypeScript compilation model

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

## Angular Ivy compilation model

An Angular component has some useful information declared in the decorator like the `selector`, these information are really important to use the component elsewhere and static type check the code.

The Angular Ivy compiler has been improved to not waste this information and enrich the definition file public interface. For instance, the following Angular component

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

The definition file became the *component public API*{: .italic-red-text }. The compiler can use it in order to type check the code that will use it:

```javascript
<awesome-comp [value]="a value">
    just a component
</awesome-comp>
```

## Compiler features

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

## Export compilation scope

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

## Partial evaluation

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

## Template type checking

A template and expressions

```html
<account-view
    [account]="getAccount(user.id, 'primary') | async">
</account-view
```

they are compiled into TypeScript code becoming *type check blocks*{: .italic-red-text }. So blocks are sent to TypeScript compiler and possible errors are then return *in the context of the template*.

```javascript
fucntion typeCheckBlock(ctx: AppComponent) {
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

## References

- (The theory of Angular Ivy | Alex Richabaugh)[https://www.youtube.com/watch?v=isb5Ef6yI48]
- (Angular Ivy by example | Jason Aden)[Angular Ivy by example | Jason Aden]
- (Deep dive into the Angular Compiler | Alex Richabaugh)[https://www.youtube.com/watch?v=anphffaCZrQ]
- (deep-dive-compiler)[https://blog.angularindepth.com/a-deep-deep-deep-deep-deep-dive-into-the-angular-compiler-5379171ffb7a]
- (deep-dive-compiler video)[https://www.youtube.com/watch?v=QQ2plVD0gDI&feature=youtu.be&t=27m]

// to be removed

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