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
toc_label: "on this page"
toc_icon: "list"
---
## In a Nutshell
__Angular Elements__ projects was revealed at the AngularConnect 2017 by Rob Worlmald

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Here&#39;s the slides from my Angular Elements talk @ <a href="https://twitter.com/hashtag/AngularConnect?src=hash&amp;ref_src=twsrc%5Etfw">#AngularConnect</a> - <a href="https://t.co/4kuySZ9zMs">https://t.co/4kuySZ9zMs</a></p>&mdash; Rob Wormald (@robwormald) <a href="https://twitter.com/robwormald/status/928250099054120960?ref_src=twsrc%5Etfw">November 8, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Angular Elements are Angular Components which are packed as Custom Elements:

> Angular Component on the inside, standards on the outside.
<cite>Rob Wormald</cite>
{: .small}

Following this approach we can develop _re-susable components_ that can be used in every kind of website regardless the technology or the framework used to build it: JavaScript, React, Vue, etc.

### Features
Angular Elements:
- are self-bootstrapping;
- host an _Angular Component_ inside a _Custom Element_;
- are a bridge between the _DOM APIs_ and _Angular Components_;
- can be used by different implementing technology without any knowledge of Angular.

## Some Jargon
> __Web Component__ _is a suite_ of different technologies (_Custom Elements_, _Shodow DOM_, _HTML templates_ and _HTML Imports_) allowing you to create reusable _custom elements_ — with their functionality encapsulated away from the rest of your code — and utilize them in your web apps.

<cite><a href="https://developer.mozilla.org/en-US/docs/Web/Web_Components">MDN web docs</a></cite>
{: .small}

> __Custom Elements__ are one of the key features of the Web Components standard to encapsulate functionalities on a HTML page. JavaScript APIs allow you to define custom elements and their behaviour to be then used in a user interface.

> __Shadow DOM__ is part of Web Components technology suite to create resusable custom elements along with the _encapsulated functionalities_.

> __Angular Component__

## First Example


## References
- [example of registering a cutom element](https://stackblitz.com/edit/angular-f3nrpv?file=app%2Fapp.module.ts)
- [more about Angular Elements](https://angular.io/guide/elements)
- [Angular Elements a quick start video](https://www.youtube.com/watch?v=4u9_kdkvTsc)
- [web components](https://developer.mozilla.org/en-US/docs/Web/Web_Components)
- [custom elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements)

// web components

// custom elements

// browser compatibility

#### Jargon
_web components_, _custom elements_, _code reuse_, _encapsulation_