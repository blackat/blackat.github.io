---
layout: single
title: "DOM"
date: '2018-05-19'
last_modified_at: 2018-05-19'
comments: true
categories:
  - HTML, DOM
tags:
  - [DOM, HTML, JavaScript]
toc: true
toc_label: "on this page"
toc_icon: "list"
toc_sticky: true
---

> The Document Object Model (DOM) is a programming interface for HTML and XML documents. It represents the page so that programs can change the document structure, style, and content. The DOM represents the document as nodes and objects. That way, programming languages can connect to the page.

__The DOM is where the stuff happens.__ The HTML written by the developer is parsed by the browser and turned into the DOM. HTML tags become objects implementing the special DOM interface (`HTMLTableElement`, `HTMLInputElement`, etc).

Some facts:
- __Yep:__ A Web page is a document. This document can be either displayed in the browser window or as the HTML source: the document is the same.
- __Yep:__ The document is the same, it changes the _way it is represented_.
- __Nope:__ the HTML you write in the editor is not the DOM.
- __Nope:__ the HTML you see through the `View Source` functionality is not the DOM, _probably_ is the HTML you have written.
- __Yep, Kinda:__ the HTML you see in the browser debugger when you want to inspect an element is the _visual representation_ of the DOM. It is often like the HTML written by you but not always:
    - if you forget to write some closing tag the browser will add them automatically;
    - in the visual representation of the DOM you can find some styles applied by JavaScript API dynamically and not present in the HTML when it is written.
- __Yep:__ The DOM is an object-oriented representation of the web page, which can be modified with JavaScript.

In the DOM tree there are DOM nodes or _elements_, it is not just markup. Once in the DOM a HTML markup turns into an _element_ with some particular characteristics such as the _events_ it can emit.
```javascript
var el = document.getElementById("myInput");
el.addEventListener("mouseenter", function(){
    console.log("Mouse entered on myInput element.");
})
```
via a DOM property (`addEventListener`) we can attach a _listener_ on the element or DOM node which is the one that emits that event

#### DOM API: JavaScript functions to manipulate the DOM
Each HTML element has a DOM interface, for instance the [`<p>` element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/p).

```html
<div id="emptyDiv"></div>

<script>
var emptyDiv = document.getElementById("emptyDiv");
emptyDiv.innerHTML = "A bit of content to make the div no more empty.";
</script>
```

The result will be
```html
<div id="emptyDiv">A bit of content to make the div no more empty.</div>
```
so dynamically, via the API, the content of the `div` has been changed, so the HTML you see in the _DevTools_ is not more the same than the code you see in _View Source_.

Where `document` object represents the document itself

{% capture notice-text %}
* [What is the DOM](https://css-tricks.com/dom/)
* [What is the Document Object Model?](https://www.w3.org/TR/DOM-Level-2-Core/introduction.html)
* [A Guide to Web Components](https://css-tricks.com/modular-future-web-components/)
* [Introduction to the DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)
{% endcapture %}

<div class="notice--primary">
  <h4>References</h4>
  {{ notice-text | markdownify }}
</div>