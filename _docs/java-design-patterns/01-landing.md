---
title: "Java Design Patterns"
permalink: /docs/java-design-patterns/landing/
excerpt: "Java design patterns explained."
last_modified_at: 2017-11-15T09:49:52-05:00
toc: true
---
>Each pattern describes a problem which occurs over and over again in our environment, and then describes the core of the solution to that problem, in such a way that you can use this solution a million times over, without ever doing it the same way twice.

<cite>Christopher Alexander</cite>
{: .small}


##Design Patterns Categories
Design patterns can be divided in some categories according to what they achieve.

### Creational Patterns
They abstract the instantiation process, helping to make the system independent with respect to how the objects are created. 
	
The instantiation is _delegated_ to another object. The system will depends on _object composition_ rather than _class inheritance_. Two themes recur in the creational patterns [Gamma et all.]:

* they encapsulate knowledge about which concrete class the system uses;
* they hide how instances of these classes are created and put together.

What a system knows about an object is only is _interface_ so the _programming at the interfaces_ using creational patterns gives a lot of flexibility about _what_ is created, _who_ creates the object, _how_ and _when_ the instance is created.

The configuration of _what_ is created can be decided _dynamically_, at _run-time_ or _statically_, at _compile time_.

- [Factory Pattern]({{"/docs/java-design-patterns/factory-pattern/" | absolute_url}})
- [Singleton]({{"/docs/java-design-patterns/" | absolute_url}})
- Prototype
- Builder

###Structural Patterns
They are related to how classes and objects are composed to from large structures, using inheritance it is possible to compose interfaces or implementations.