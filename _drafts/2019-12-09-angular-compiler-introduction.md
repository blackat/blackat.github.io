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