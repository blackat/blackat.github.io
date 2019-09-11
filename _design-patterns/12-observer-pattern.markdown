---
title: "Observer Pattern"
permalink: /design-patterns/observer-pattern/
last_modified_at: 2017-11-15T09:49:52-05:00
comments: false
categories: [design patterns, observer pattern, java]
toc: true
toc_sticky: true
---
>The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically.

<cite>Bates and Sierra</cite> --- [Head First Design Patterns](http://shop.oreilly.com/product/9780596007126.do)
{: .small}

## Real life example: magazine subscription

Subscribe to a magazine (*publisher*), for instance The New Yorker, via an electonic device:

- every time a new issue is released it is push to you, you don't have to poll or to ask for it;
- once you are not interested anymore just unsubscribe to not get anymore any new issue.

Publishers and subscribers make the *Observer pattern*.

## Set the jargon

- the publisher is the *subject*;
- the seubscribers are the *observers*;
- subjects and observers can interact with little knowledge of each other, they are *loosely coupled*.

## Pilot project

A pilot project is something we can use to explain how to arrive to the Observer Pattern.

Consider a Single Board Computer (Raspberry pi, LattePanda, etc) equipped with sensors to monitor some parameters of a room such as temperature, humidity, light and so on. Each room is monitored by one SBC.

Data can be retrieved by another SBC for instance, or by an application that provides display functionality.

### Apply designing principles

#### Encapsulate what varies

There are *two areas of change*:

- each SBC could have different sensors;
- the observers of sensor data could be different.

*Encapsulate what varies* so that objects can interact having little knowledge of each other. The *subject* as the *observers* need to know just the minimal amount of information to exchange data. 

#### Program to an interface, not to an implementation

*Interface* represents the minimal amount of knowledge an object needs to know about the other. Observers don't need to know a lot about the subject but only how to register and unregister. A subject just need to know how to send data to an
observer and not how the observer is done.

#### Strive for loosely coupled designs between objects that interact

Interfaces allow to minimize the interdependency between objects, objects can change without affecting the others. Such a OO system is flexible, can evolve and can live longer without massive redesign.

## Class diagram

<img src="/assets/images/design-patterns/java/observer_pattern.png">

Use `Subject` interface to register or remove an observer. A potential _observer_ must implements the `Observer` interface, its method will be called when the _subject's_ state changes.

The `ConcreteSubject` contains and controls the _state_, it is the sole _owner of the data_.
##Design Principle
[Loosely Coupled Design Principle.](/oo-design-principles/index.html#loosely_coupled_principle)

## Scenario
A news stand produces news during all the day. Each person interested in having news has to subscribe to the stand to get notifications. In this way the person doesn't have to check every time if a new article has been published.

## Key Points

* __Subject.__ It is the observable and updates the observers using a common interface.
* __Loosely coupled.__ Observable doesn't know anything about observers except that they implement an interface.
	* Allow flexible design minimizing the interdependency between objects.
* __Pull data.__ When data changes, observable _pull data_ to the observers.
* __Order.__ There is not any _notification order_.
* __Register.__ An  observer can register to an observable at any time.

## Observer Pattern in Action

```java
public interface Subject {
	void registerObserver(Observer observer);

	void removeObserver(Observer observer);

	void notifyObservers();
}
```

```java
import java.util.ArrayList;
import java.util.List;

public class NewsProvider implements Subject {

	private List<Observer> observerList;
	private News news;

	public NewsProvider() {
		observerList = new ArrayList<Observer>();
	}

	@Override
	public void registerObserver(Observer observer) {
		observerList.add(observer);
	}

	@Override
	public void removeObserver(Observer observer) {
		observerList.remove(observer);
	}

	@Override
	public void notifyObservers() {
		for (Observer o : observerList) {
			o.update(news);
		}
	}

	void setNews(News n) {
		news = n;
		notifyObservers();
	}
}
```

```java
public interface Observer {
    void update(News news);
}
```

```java
public class NewsObserver implements Observer {

	private Subject subject;

	public NewsObserver(Subject s) {
		subject = s;
		s.registerObserver(this);
    }

	@Override
	public void update(News news) {
		System.out.println("new news published: " + news.toString());
	}
}
```

```java
public class News {
	private String title;
	private String author;
	private String content;

	public News(String t, String a, String c) {
		title = t;
		author = a;
		content = c;
	}

	@Override
	public String toString() {
		return title + " " + author + ": " + content;
	}
}
```

```java
public class NewsStand {
	public static void main(String[] args) {
		NewsProvider newsProvider = new NewsProvider();

		Observer userOne = new NewsObserver(newsProvider);
		Observer userTwo = new NewsObserver(newsProvider);

		newsProvider.setNews(new News("New economy", "Barry Lindon", "Many new changes happened."));
	}
}
```
