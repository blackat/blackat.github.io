---
layout: single
title: "Web Components"
date: '2018-05-20'
last_modified_at: 2018-05-20'
comments: true
categories:
  - Web Components, HTML
tags:
  - [web components, HTML, DOM]
toc: true
toc_label: "on this page"
toc_icon: "list"
toc_sticky: true
---
> Web Components is a suite of different technologies allowing you to create reusable custom elements — with their functionality encapsulated away from the rest of your code — and utilize them in your web apps.

<cite><a href="https://developer.mozilla.org/en-US/docs/Web/Web_Components">MDN web docs</a></cite>
{: .small}

__Code reuse__ It is a good idea reuse code as much as possible. Markup structure are really complex due to associated styles and scripts. Web components aim to reuse html markup easier. 
{: .notice--info}

Web components are made of _four main technologies_ that facilitate the creation of _custom elements_ that can be reused:
- __Custom elements:__ set of JavaScript APIs that allow the definition of new elements and their behaviours. A custom element can then be used in the user interface.
- __Shadow DOM:__ set of JavaScript APIs that allow to attach an _encapsulated shadow DOM tree_ to an element.
- __HTML Templates:__ resusable templates by custom elements, these markup is not displayed in the rendered page thanks to (`<template>`)[https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template] and `<slot>` elements.
- __HTML Imports:__ is mechanism to import a _custom component_ into pages.

## <template>

```html
<link rel="import" href="component.tpl.html">
```

## Refences
[1]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template "HTML Content Template"

[template example]: http://jsbin.com/qaxiw/7/edit?html,js,output
[template tag]: https://www.html5rocks.com/en/tutorials/webcomponents/template/
[html import]: https://developer.mozilla.org/en-US/docs/Web/Web_Components/HTML_Imports
