---
title: "Composite Pattern"
permalink: /design-patterns/composite-pattern/
last_modified_at: 2017-11-15T09:49:52-05:00
comments: false
categories: [design patterns, composite pattern, java]
toc: true
toc_sticky: true
---
>The Composite Pattern allows you to compose objects into tree structures to represent part-whole hierarchies. Composite lets client treat individual objects and composition of objects uniformly.

<cite>Bates and Sierra</cite> --- [Head First Design Patterns](http://shop.oreilly.com/product/9780596007126.do)
{: .small}

## Class Diagram

/images/posts/design-patterns/composite.jpg

`Client` uses the `Component` to manipulate objects.

`Component` defines an interface both for the composite and for the leaf which are element of the collection. It might implement a default behavior for methods.

`Leaf` has no children and inherits methods and override what make sense for the class itself.

`Composite` has children, the `Components`, which can be `Component` or `Leaf` type and inherits methods and override what make sense for the class itself.

## Key Points

* __New Approach.__ Not necessarily related to iterators.
* __Trees.__ Build object structures in the form of _trees_ containing both nodes and leaves.
  * Same operations are applied over both composite and leaves.
  * Ignore the differences between nodes and leaves.

* __Part-whole Hierarchy.__ Node with children and leaves are in the same structure.
  * Tree of objects made of parts, nodes and leaves, but threaten as a whole.
