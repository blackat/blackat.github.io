---
layout: single
title: "Polyfill vs. Shim"
date: '2016-10-25 18:47'
comments: true
categories:
  - JavaScript
tags:
  - polyfill
  - shim
toc: true
toc_label: "on this page"
toc_icon: "list"  # corresponding Font Awesome icon name (without fa prefix)
---

With the new ES6 specification, people talk more and more about polyfill and shim, sometimes as the term are interchangeable but, are they the same thing?

## Polyfills and shims
There are many definitions and interpretation about what is Polyfill and shim. Agree that both are libraries there are some differences between them.

{% capture notice-text %}
  - the API exists in major browser;
  - has its own API;
  - is more about fixing some functionality and aligning browser behavior with another browser;
  - is strictly tailored for an _old environment_;
  - is a _graceful degradation_.
{% endcapture %}
<div class="notice--info">
  <h4>Shim:</h4>
  {{ notice-text | markdownify }}
</div>

{: .notice--warning}
__Watch out!__ Saying _"has its own API"_ doesn't mean it exposes a broader API, more methods, but just that all the methods from the API are reimplemented. The shim intercepts all the API calls and provides its own implementation so that all the browsers have the same behavior.

{% capture notice-text %}
  - [is a shim for a browser API](http://www.2ality.com/2011/12/shim-vs-polyfill.html);
  - is related to browsers;
  - implements missing feature in an API;
  - is something that [you could drop in and it would silently work](https://remysharp.com/2010/10/08/what-is-a-polyfill);
  - brings future EcmaScript features back in time to old and latest browsers (_regressive enhancement_).
{% endcapture %}
<div class="notice--info">
  <h4>Polyfill:</h4>
  {{ notice-text | markdownify }}
</div>

## In a nutshell
[Respoke](http://blog.respoke.io/post/111278536998/javascript-shim-vs-polyfill) has created an interesting flow chart to decide if a certain library is a shim or a polyfill. The demarcation between them is not always really sharp. Try to answer the following question and make a decision:

1. Does your library fix some functionality and/or normalize a JavaScript API across the major browsers?
2. Does the JavaScript API exist in some major browsers?
3. Does your library implement the JavaScript API where it does not exist?

![image-center](/images/polyfill-vs-shim/flow-chart.png){: height="70%" .align-center}

## Polyfill, back to 2010
The term _polyfill_ comes from a [2010 blog post](https://remysharp.com/2010/10/08/what-is-a-polyfill) by [Remy Sharp](https://twitter.com/search?q=%40rem&src=typd) where Remy Sharp was looking for a new term or word:

> I wanted a word that meant "replicate an API using JavaScript (or Flash or whatever) if the browser doesn't have it natively.
...
> I wanted something you could drop in and it would silently work.

The term _shim_ doesn't work for him because:

> I knew what I was after wasn't progressive enhancement because the baseline that I was working to required JavaScript and the latest technology. So that existing term didn't work for me.

So Remy gives the following definitions:

> A __polyfill__, or polyfiller, is a piece of code (or plugin) that provides the technology that you, the developer, expect the browser to provide natively. Flattening the API landscape if you will.

> Poly meaning it could be solved using any number of techniques - it wasn't limited to just being done using JavaScript, and fill would fill the hole in the browser where the technology needed to be. It also didn't imply "old browser" (because we need to polyfill new browser too).

> __Shim__, to me, meant a piece of code that you could add that would fix some functionality, but it would most often have __its own API__.

## Other definitions

> I like classify __polyfilling__ as a form of Regressive Enhancement
>
> <cite>[Alex Sexton](https://twitter.com/SlexAxton/status/25600963629)</cite>

> A __shim__ that mimics a future API providing fallback functionality to older browsers.
>
> <cite>[Paul Irish](http://paulirish.com/)</cite>

> A __polyfill__ is code that detects if a certain "expected" API __is missing__ and manually implements it. E.g.   
> `if (!Function.prototype.bind) { Function.prototype.bind = ...; }`
>
> A __shim__ is code that intercepts existing API calls and implements different behavior.
>
> The idea here is to normalize certain APIs across different environments. So, if two browsers implement the same API differently, you could _intercept the API calls_ in one of those browsers and make its behavior align with the other browser. Or, if a browser has a bug in one of its APIs, you could again intercept calls to that API, and then circumvent the bug.
>
> <cite>[Sime Vidas](http://stackoverflow.com/questions/6599815/what-is-the-difference-between-a-shim-and-a-polyfill/17331540#17331540)</cite>

> A __shim__ is a library that brings a new API to an older environment, using only the means of that environment.
>
> A __polyfill__ is a shim for a browser API. It typically checks if a browser supports an API. If it doesn’t, the polyfill installs its own implementation. That allows you to use the API in either case.
>
> <cite>[Axel Rauschmayer](http://www.2ality.com/2011/12/shim-vs-polyfill.html)</cite>

## Polyfills and shims libraries
[Paul Irish](http://paulirish.com/) provides a [list of HTML5 cross browser polyfills and shims](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-browser-Polyfills).
