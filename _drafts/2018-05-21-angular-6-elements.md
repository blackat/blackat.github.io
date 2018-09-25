---
layout: single
title: "Angular 6 Elements"
date: '2018-05-20'
last_modified_at: 2018-05-2'
comments: true
categories:
  - Angular
tags:
  - [angular, angular elements, web components, encupasulation, custom elements]
toc: true
toc_label: "On this page"
toc_icon: "list"
toc_sticky: true
---

Angular Elements *library*{: .italic-red-text} provides a method to extend the HTML tag library writing components that can be used with any framework.
The new components can be used *everywhere*{: .italic-red-text} without any Angular knowledge and regardless the technology used to implement a web site:

- CMS
- HTML and VanillaJS
- React, Vue, etc
- legacy AngularJS application instead of the `ngUpdate` for the migration.

**Pro Tip.** Angular Elements are *Angular Components* packed as [*Custom Elements*][custom elements], part of the [*Web Components*][web components], a suite of Web platform APIs.
{: .notice--info}

## World of Custom Components

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Want to know the syntax difference between native Web Components, <a href="https://twitter.com/polymer?ref_src=twsrc%5Etfw">@polymer</a>, <a href="https://twitter.com/stenciljs?ref_src=twsrc%5Etfw">@stenciljs</a> and <a href="https://twitter.com/angular?ref_src=twsrc%5Etfw">@Angular</a> Elements? I created a repo that implements the same Todo list with those technologies<br>Repo: <a href="https://t.co/dWlM4Hsz4P">https://t.co/dWlM4Hsz4P</a> <br>Demo: <a href="https://t.co/5eo8NrxZQb">https://t.co/5eo8NrxZQb</a><a href="https://twitter.com/hashtag/TodoMVC?src=hash&amp;ref_src=twsrc%5Etfw">#TodoMVC</a> <a href="https://t.co/RMoNQOuBmV">pic.twitter.com/RMoNQOuBmV</a></p>&mdash; Julien Renaux 👨‍💻 (@julienrenaux) <a href="https://twitter.com/julienrenaux/status/957985488320843776?ref_src=twsrc%5Etfw">January 29, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

Each framework provides a specific tailored mechanism to smoothly extend the HTML tag library w.r.t. its own way to configure, interact and update the DOM element.

For instance in AngularJS, the developer can implement new HTML tags via the *directive*{: .italic-red-text} approach: a powerful technique to link together a controller and a template encapsulating everything in a new DOM element.

Framework specific components libraries cannot be used neither by another framework or in a CMS or in a website implemented in HTML and JavaScript with standard Web API.

**Nitty Gritty.** Web Components allow to extend the HTML developing new *cross browser DOM elements*. Encapsulation and private features enpower reusability avoiding collision with other elements ([Shadow DOM][shadow dom]).
{: .notice--info}

## Web Components

[Web components][what are web components] are a set of *Web Platform APIs*{: .italic-red-text} to create new custom, reusable, encapsulated HTML tags that can be used *everywhere*{: .italic-red-text}.

Developers can then extend HTML tag library writing new elements that encapsule style and behavior.

Web Components include four different technologies:

- [Custom Elements:][custom elements] JavaScript APIs to define new DOM elements.
- [Shodow DOM:][shadow dom] JavaScript APIs to render and attach and encapsulated Shadow DOM tree to an element whose features stay private and not collide with the rest of the document.
- [HTML templates:][html template] write HTML templates that can be reused by custom elements using `<template>` and `<slot>` tags.
- [HTML Imports:][html import] how to include and reuse HTML templates.

**Pro Tip.** Angular Elements are powered by *Custom Elements*.
{: .notice--info}

**Watch out.** Web Components specification has moved from V0 to V1.
{: .notice--primary}

## Extend the HTML with v1 Custom Elements

Let's introduce an example to show how to implement a new custom element.

<iframe src="https://stackblitz.com/edit/custom-element-head-first?embed=1&file=icon-text.js&hideExplorer=1&view=editor" frameborder="0" style="overflow: hidden; height: 500px;width: 100%;"></iframe>

<br/>

To define a new [v1 Custom Element][v0 to v1]:

- create a ES6 class extending [`HTMLElement`][html element];
- register the new class with the browser via the [`CustomElementRegistry`][custom registry] Web API: use [`window.customElements`] property of the `Window` interface to get an instance of it;
- use *lifecycle hooks*{: .italic-red-text} or *reactions*{: .italic-red-text}
  - `constructor`: initialize the state, set up event listeners or create a [Shadow DOM][shadow dom]
  - `connectedCallback`: called when the element is attached to the DOM, useful for setting up, fetching some resources or rendering
  - `disconnectedCallback`: called when the element is removed from the DOM, useful for cleaning up
  - `attributeChangedCallback(attrName, oldVal, newVal)`: called when an attribute registered in the `observedAttributes` property has been added, removed, updated or replaced
  - `adoptedCallback()`: called when the element has been moved into a new `document` (`document.adoptNode(el)`)
  - [more examples][lifecycle hooks example]
- events can be associated, as any other DOM element, using  [`EventTarget`][event target] method `addEventListener`.

**Custom Element.** Web API that allow to extend the HTML tag system creating new custom DOM elements that encapsulate functinalities away from the rest of the code. It refers to [encapsulation principle][encapsulation] of the software engineering.
{: .notice--info}

## Shadow DOM Quick

Shadow DOM is more or less like a *scoped subtree inside the Custom Element*{: .italic-red-text} that helps to build a new DOM element.

```javascript
class RocketParagraph extends HTMLElement {
  constructor() {

    super()

    const shadowRoot = this.attachShadow({ mode: 'open' })
    shadowRoot.innerHTML = `<link rel="stylesheet" href="fontawesome.com/all.css">`

    const span = document.createElement('span')
    const icon = document.createElement('i')

    ...

    shadowRoot.appendChild(span)
    shadowRoot.appendChild(icon)
  }
}
window.customElements.define('rocket-paragraph', RocketParagraph)
```

> A [Shadow Root][shadow root] is the document fragment that gets attached to a "host" element. The act of attaching a shadow root is how the element gains its shadow DOM.

{% include figure image_path="/assets/images/posts/shadow-dom-debugger.png" alt="Chrome debugger for Shadow DOM" caption="Shadow Root in the Chrome debugger." %}

## Building Libraries

There are [many libraries][web components help libraries] that help to build custom Web Components providing features such as:

- boilerplate to define the project scaffolding
  - Polymer Boilerplate
  - ValillaJS Boilerplate
  - etc
- polyfills to have web components work in all major browsers
- automatic build systems
  - Yeoman generator to automate scaffolding and building web components.
  - Polymer CLI to automate scaffolding and build Polymer-based web components.
  
Once created the component it can be [published][publish web component] and the downloaded and installed via a package manager like Bower.

[Look for a component](https://www.webcomponents.org/), for instance [`emoji-rain`](https://www.webcomponents.org/element/notwaldorf/emoji-rain), install and use it in your web site.

More examples are available on the [Web Components Github MDN repo][web components mdn github examples].

## Angular Elements

> Angular Component on the inside, standards on the outside.
<cite>Rob Wormald</cite>
{: .small}

**Pro Tip.** Angular Elements are self-bootstrapping, they host an _Angular Component_ inside a _Custom Element_ and their usage is not anymore tied to a specific framework.
{: .notice--info}

**Angular Components** [act as a hosts][telerik angular elements] for **Custom Elements** mapping properties from the Angular world to the Web Platform API standard:

- `@Input` mapped to properties
- `@Output` mapped to events
- `@HostBinding` mapped to observed attributes
- lifecycle hooks mapped to Custom Elements lifecycle hooks

Whereas *Inputs*{: .italic-red-text} rely on the Custom Elements lifecycle hook, *Outputs*{: .italic-red-text} rely on the Custom Event API (spec). Both APIs are entirely separate from each other.

**[Dependency Injection.][telerik angular elements]** Angular Elements also let us take advantage of Angular's dependency injection. When we create a custom element using Angular Elements, we pass in a reference to the current module's injector. This injector lets us share context across multiple elements or use shared services.
{: .notice--info}

**[Content Projection.][telerik angular elements]** We also get content projection (transclusion) with Angular Elements, with a couple of caveats. Content project works correctly when the page first renders, but not with dynamic content projection yet. As of now, we also don't yet have support for `ContentChild` or `ContentChildren` queries. Content projection should get more robust over time, though, and we'll also have the ability to use slots and the shadow DOM as browser support increases.
{: .notice--info}

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">PR for basic Shadow DOM v1 support in Angular - should make Elements more flexible, as you can now use &lt;slot&gt; elements to do basic content projection. <a href="https://t.co/SBwNBQmwwe">https://t.co/SBwNBQmwwe</a></p>&mdash; Rob Wormald (@robwormald) <a href="https://twitter.com/robwormald/status/1013121891941269504?ref_src=twsrc%5Etfw">June 30, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Angular Elements: where to use

Angular Elements project has been revealed at the AngularConnect 2017 by Rob Worlmald

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Here&#39;s the slides from my Angular Elements talk @ <a href="https://twitter.com/hashtag/AngularConnect?src=hash&amp;ref_src=twsrc%5Etfw">#AngularConnect</a> - <a href="https://t.co/4kuySZ9zMs">https://t.co/4kuySZ9zMs</a></p>&mdash; Rob Wormald (@robwormald) <a href="https://twitter.com/robwormald/status/928250099054120960?ref_src=twsrc%5Etfw">November 8, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Rob Wormald has recently defined three main areas of use for the Angular Elements during the [ng-conf 2018][where use angular elements]:

### Elements in Apps

- CMS embeds
- Dynamic components
- Server-side/hybrid rendering

### Element Containers

- Mini-apps
- Micro-frontends
- ngUpgrade
- SharePoint

### Resusable Widgets

- Cross-framework compatibility
- Material/CDK components in any environment
- Design systems - build once, use anywhere

**Note.** Some work has still to be done for reusable widgets.
{: .notice--primary}

Angular Elements is one of the Angular Labs projects where new experimental ideas are developed. The first release became part of Angular 6 and a  more refined version should arrive along with Angular 7. For the time being Angular Elements is not ready for production yet:

- **element bundle size:** even if a small widget is delivered a great amount of Angular code comes along with. [Angular Yvi][telerik angular ivy], next Angular compiler generation, should solve this problem removing from the bundle unused Angular code;
- **laborius setup:** the set up required to use Angular Elements is a bit cumbersome, better way should come;
- **browser support:** native browser support for Custom Elements has been not fully implemented everywhere as can be seen from [this table][custom elements browser]. For the time being:
  - [Chrome 54][chrome support] has Custome Elements v1;
  - [Safari 10.1][safari support] has Custom Elements v1;
  - [Edge][edge support] has begun prototyping;
  - [Mozilla][mozilla support] has an open bug to implement;
  - use [`@webcomponents/webcomponentsjs`][polyfill] loader to asynchronously load only the necessary polyfills required by the browser.

## Create New Angular Elements Component

### Setup the Project

```bash
npm install -g @angular/cli@latest
ng new angular-geometry-elements
cd angular-geometry-elements
ng add @angular/elements --project=angular-geometry-elements
```

The latest command add both Angular Elements and the polyfill called [`document-register-element`][polyfill] to the configuration file `angular.json`.

### Create the Component

/// put the code here

- `ViewEncapulation`: change the encapsulation strategy w.r.t. the styles are applied to the component:
  - `Native`: use the real Shadow DOM if natively available, otherwise the `web-components` polyfills are required to shim the behavior.
  - `Emulated`: scoped styling emulating the Shadow DOM.
- `@Input` for a property with a default value.
- Some other properties.

rename `bootstrap`to `entryComponents` in the `app.module.ts`to avoid the component bootstrap along with the module.

Now create the custom element

```javascript
constructor(private injector: Injector) {
  const el = createCustomElement(AppComponent, {injector});
  customElements.define('geometry-elements', el);
}
```

this function returns a special class that can be used with the Custom Element Web API to define a new custom element as the pair selector and ES6 class.

**Pro Tip.** The Angular Component now act as a host for the Custom Element.
{: .notice--info}

## Build and Pack the Angular Element Component

So  fare there is not a straight forward way to pack an Angular Element component, here some approaches proposed by [Sam Julien][telerik angular elements]:

- Use a bash script, like in this article by Tomek Sułkowski.
- Use Gulp, like in this made-with-love component by Nitay Neeman.
- Use Manfred Steyer's CLI tool ngx-build-plus.
- Use Node with a script like in this article by Sebastian Eschweiler.

Following the Sam Julien's advice the Node approach is choosen: 

```bash
npm install --save-dev concat fs-extra
```

Create a file called `elements-build.js` at the root of the project with the recomended script. The script will combine what the Angular CLI has generated in a single file.

In the `package.json` add

```javascript
"build:elements": "ng build --prod --output-hashing none && node elements-build.js"
```

Then create the file `index.html` in the previously created folder `elements` to have a page where to test the new created component.

Fireup the build

```bash
npm run build:elements
```

The output of the build is put in the `elements` folder 

```bash
.
├── angular-geometry-elements.js
├── index.html
└── styles.css
```

**Note.** Style file is empty because of the `Native` strategy choice.
{: .notice-info}

### Test The Angular Elements Component

Install the `static-server` to serve the `elements/index.html`

```bash
npm install -g static-server
```

From the afrementioned folder type

```bash
static-server
```

and connect to `localhost:9080`.

### Check The Registered Element

Following [this advice][is custom element registered], in the browser console type:

```javascript
String.prototype.wasRegistered = function() { 
  switch(document.createElement(this).constructor) {
    case HTMLElement: return false; 
    case HTMLUnknownElement: return undefined; 
  }
  return true;
}
'element-cake'.wasRegistered()
//⇒ true
'element-my-cake'.wasRegistered()
//⇒ false
'xx'.wasRegistered()
//⇒ undefined
```


## Angular Components

Components are the fundamental building block in a component-driven application architecture. They are ES6 classes having the `@Component` decorator on top.
 
> __Custom Elements__ are one of the key features of the Web Components standard to encapsulate functionalities on a HTML page. JavaScript APIs allow you to define custom elements and their behaviour to be then used in a user interface.

> __Shadow DOM__ is part of Web Components technology suite to create resusable custom elements along with the _encapsulated functionalities_.

## Angular Elements in 10 Steps

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">RT <a href="https://twitter.com/sulco?ref_src=twsrc%5Etfw">@sulco</a>: A lot of chatter about <a href="https://twitter.com/hashtag/Angular?src=hash&amp;ref_src=twsrc%5Etfw">#Angular</a> Elements lately! <a href="https://twitter.com/robwormald?ref_src=twsrc%5Etfw">@robwormald</a>’s and <a href="https://twitter.com/PascalPrecht?ref_src=twsrc%5Etfw">@PascalPrecht</a>’s talks got me energized, so I’ve done some tinkering &amp; created this 10 step guide to building your own: from `npm init -y` to the production build. Hope you find it useful! 🔨https://… <a href="https://t.co/YKRken80vr">pic.twitter.com/YKRken80vr</a></p>&mdash; Learn Vue.js (@LearnVuejs2) <a href="https://twitter.com/LearnVuejs2/status/1007957528326606849?ref_src=twsrc%5Etfw">June 16, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

https://medium.com/@tomsu/building-web-components-with-angular-elements-746cd2a38d5b



## Browser Support

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Great to hear that! Firefox 63 are getting Shadow DOM and Custom Elements.<a href="https://t.co/Hgp7HXx8r5">https://t.co/Hgp7HXx8r5</a></p>&mdash; Hayato (Shadow DOM) (@shadow_hayato) <a href="https://twitter.com/shadow_hayato/status/1029185633678315520?ref_src=twsrc%5Etfw">August 14, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">We are planning on deprecate and remove Shadow DOM v0 and Custom Elements v0 from Blink. Shadow DOM v0 and Custom Element v0 are NOT *Web*. They are only available on Google Chrome.<br>If you are still using v0 APIs, please consider to migrate to v1 seriously.</p>&mdash; Hayato (Shadow DOM) (@shadow_hayato) <a href="https://twitter.com/shadow_hayato/status/1016911248863080448?ref_src=twsrc%5Etfw">July 11, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Little hack for enabling shadow DOM V1 and slots in Angular Elements,  till we fix <a href="https://t.co/Q3F1LoUcGP">https://t.co/Q3F1LoUcGP</a> - <a href="https://t.co/vh4TRGbQTP">https://t.co/vh4TRGbQTP</a></p>&mdash; Rob Wormald (@robwormald) <a href="https://twitter.com/robwormald/status/1005613409486848000?ref_src=twsrc%5Etfw">June 10, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Here&#39;s the slides from my Angular Elements talk @ <a href="https://twitter.com/hashtag/AngularConnect?src=hash&amp;ref_src=twsrc%5Etfw">#AngularConnect</a> - <a href="https://t.co/4kuySZ9zMs">https://t.co/4kuySZ9zMs</a></p>&mdash; Rob Wormald (@robwormald) <a href="https://twitter.com/robwormald/status/928250099054120960?ref_src=twsrc%5Etfw">November 8, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<!-- References -->
<!-- Web Components -->
[web components]: (https://developer.mozilla.org/en-US/docs/Web/Web_Components)
[what are web components]: (https://www.webcomponents.org/introduction)
[web components mdn github examples]: https://github.com/mdn/web-components-examples
[web components spec]: http://www.w3.org/TR/components-intro
[v0 to v1]: https://hayato.io/2016/shadowdomv1/
[web components help libraries]: https://www.webcomponents.org/resources
[publish web component]: https://www.webcomponents.org/publish
[best practices]: https://www.webcomponents.org/community/articles/web-components-best-practices

<!-- Custom Elements -->
[example of registering a cutom element]: (https://stackblitz.com/edit/angular-f3nrpv?file=app%2Fapp.module.ts)
[custom elements]: (https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements)
[custom registry]: https://developer.mozilla.org/en-US/docs/Web/API/CustomElementRegistry
[window custom elements]: https://developer.mozilla.org/en-US/docs/Web/API/Window/customElements
[custom elements google example]: https://developers.google.com/web/fundamentals/web-components/customelements
[custom element specification]: https://html.spec.whatwg.org/multipage/custom-elements.html#custom-element
[chrome support]: https://www.chromestatus.com/features/4696261944934400
[safari support]: https://webkit.org/status/#feature-custom-elements
[edge support]: https://twitter.com/AaronGustafson/status/717028669948977153
[mozilla support]: https://bugzilla.mozilla.org/show_bug.cgi?id=889230
[polyfill]: https://developers.google.com/web/fundamentals/web-components/customelements#polyfill
[is custom element registered]: https://stackoverflow.com/questions/27334365/how-to-get-list-of-registered-custom-elements
[polymer unregister custom element]: https://gist.github.com/ebidel/cea24a0c4fdcda8f8af2
[lifecycle hooks example]: https://github.com/mdn/web-components-examples/blob/master/life-cycle-callbacks/main.js

<!-- Shadow DOM -->
[shadow dom]: https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM
[shadow root]: https://www.webcomponents.org/introduction#creating-and-using-a-shadow-root

<!-- HTML Template -->
[html template]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template

<!-- HTML Imports -->
[html import]: https://developer.mozilla.org/en-US/docs/Web/Web_Components/HTML_Imports

<!-- Angular Elements -->
[more about Angular Elements]: (https://angular.io/guide/elements)
[Angular Elements a quick start video]: (https://www.youtube.com/watch?v=4u9_kdkvTsc)
[telerik angular elements]: https://www.telerik.com/blogs/getting-started-with-angular-elements
[where use angular elements]: https://www.youtube.com/watch?v=Z1gLFPLVJjY

<!-- Polyfill -->
[polyfill]: https://github.com/WebReflection/document-register-element

<!-- Angular Ivy -->
[telerik angular ivy]: https://www.telerik.com/blogs/first-look-angular-ivy

<!-- Demo -->
[web components differences]: (https://github.com/shprink/web-components-todo)

<!-- Browser Support -->
[custom elements browser]: https://caniuse.com/#feat=custom-elementsv1
[ie status]: https://developer.microsoft.com/en-us/microsoft-edge/platform/status/

<!-- Others -->
[encapsulation]: https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)

<!-- Web API -->
[event target]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget
[html element]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement

<!-- Examples -->
[dynamic dashboard]: https://t.co/DLlrTKpnfY

<!-- Open Question -->
<!--
- are custom elements registered per tab or browser wise?
- how unregister a custom element?
-->

// web components

// custom elements

// browser compatibility

#### Jargon
_web components_, _custom elements_, _code reuse_, _encapsulation_