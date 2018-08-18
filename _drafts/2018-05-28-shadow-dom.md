---
layout: single
title: "Shadow DOM"
date: '2018-05-28'
last_modified_at: 2018-05-28'
comments: true
categories:
  - HTML, DOM
tags:
  - [html, DOM, shadow DOM]
toc: true
toc_label: "on this page"
toc_icon: "list"
toc_sticky: true
---
## Shadow DOM
_Shadow DOM_ is part of Web Components technology suite to create resusable custom elements along with the _encapsulated functionalities_.

__Whatch out!__ Web Components are not custom elements.
{: .notice--warning}

The Shadow DOM technique implements the encapsulation of the custom elements in order to hide implementation details, such as markup structure, style and bahavior. Hidden code is then kept separate from the code of the page that result nicer, cleaner and it does not clash with the custom element one.

The custom elements are then attached to a given one.

Before the only way to achieve encapsulation in a browser was through an iFrame.

__Encapsulation:__ the _internal representation_ of an object is hidden preventing unauthorized parties the direct access to them, tipically everything outside the object definition.
{: .notice--info}

__Shadow DOM__ is a powerful technique, already used by many browser, to hide implementation details such as markup structure, styles and behavior. The Shadow DOM API allows to attach a _hidden separate DOM_ to an element keeping the code simple and clean.
{: .notice--info}

## Is it Shadow DOM really new?
For instance, try to render in a browser
```html
<input id="myRange" type="range">
```
and then in the DevTools console type
```bash
> var slider = document.getElementById('myRange')
> slider.firstChild
> null
```
Why `null`? _The implementation has been hidden inside of a shadow DOM subtree._ Sometimes the implementation is too complex, such as the `<video>` tag, or too difficult to represent, but, most of the times, _"useless"_.

__Useless?__ Well, when you develop a web application do you really care of how the `<input>` or `<select>` fields are implemented by the browser?
{: .notice--warning}

__Nice and clean code?__ Do you really want to see the implementation of all the HTML elements you need in your web application in the markup?ot will it just pollute your code making it more difficult to debug and to read?
{: .notice--warning}

The _shadow DOM_ of the element is not inlcuded in the main `document` tree, but for instance, the _events_ fired in the shadow DOM can be listened in the document, for instance:
```html
<div onclick="alert('Event fired by: ' + event.target)">
    <audio controls src="foo.wav"></audio>
</div>
```
The _target_ will be the all `<audio>` element and not some button because the _implementation is hidden and separate_, we won't get any implementation detail.

__What's new?__ The Shadow DOM specification, so developers can better implements custom components.
{: .notice--info}

// how to display the implementation, is it possible?

### At Glance
__[```Element.attachShadow()```](https://developer.mozilla.org/en-US/docs/Web/API/Element/attachShadow) method:__ method to attach a shadow root to any element.

__[```<shadow>```](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/shadow) element:__ this element has been declared obsolete.

__Shadow root:__ it is the mount point of a shadow DOM.

__Shadow DOM:__ it is the internal DOM structure of a custom element.

## New Custom HTMLElement Element
We would like to create a custom element that takes an attribute named `text` and render the content with a rocket icon on the right.

<iframe width="100%" height="300" src="//jsfiddle.net/agz0p65r/5/embedded/js,html,css,result/dark/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### 1. Attach a shadow root to the current element
```javascript
// this is the reference to the current HTML element
var shadow = this.attachShadow({mode: 'open'});
```

### 2. Create the shadow dom structure
The structure that will be attached, it will be hidden or not w.r.t. the open or closed parameter.
```javascript
let paragraph = document.createElement('p');
```

### 3. Attach the shadow DOM to the shadow root
Attache the create element to the shadow root.
```javascript
shadowRoot.appendChild(paragraph);
```

### 4. Use the


### Browser Implementation
#### Firefox
Set the preferences dom.webcomponents.enabled and dom.webcomponents.shadowdom.enabled to true.

Firefox's implementation is planned to be enabled by default in version 60/61.

#### Edge
It is on going.

#### Chrome and Opera
Supported by default.

## References
- [Shadow DOM](https://developers.google.com/web/fundamentals/web-components/shadowdom)
- [Web Components Best Practices](https://www.webcomponents.org/community/articles/web-components-best-practices)
- [Custom Elements Best Practices](https://developers.google.com/web/fundamentals/web-components/best-practices#create-a-shadow-root-to-encapsulate-styles)
- [Web components using a shared style sheet](https://www.smashingmagazine.com/2016/12/styling-web-components-using-a-shared-style-sheet/)