---
layout: single
title: "Node.js Intro"
date: '2018-11-01'
last_modified_at: '2019-08-18'
comments: true
categories:
  - node.js
tags:
  - [node.js]
toc: true
toc_label: "On this page"
toc_icon: "list"
toc_sticky: true
---

## Event-driven Programming

It is a programming paradigm in which the flow of the program depends on events. GUIs are predomintaly based on this paradigm, actions happen as responses to user inputs.

An event-drive application is generally characterized by a main loop that listen to events and triggers callback function as response to events that happened.

Writing an event-driven application means implementing the event handlers, the main loop is generally provided by the framework or the environment such as Node.js.

## Event emitter

The `EventEmitter` [class](https://nodejs.org/api/events.html#events_class_eventemitter) is exposed by the `events` module

```javascript
const EventEmitter = require('events');
```

// put a simple example where an event is defined and the callback called when happens

## Event loop

The [Node.js Event Loop](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/) allows to perform _non-blocking I/O operations_ by offloading operations to system kernel whenever possible.

Kernels are multi-thread since can execute multiple operations in background. Once an operation completes, the kernel tells (find the right name from the book) Nodes.js that the callback can be added to the _poll_ queue.

When Node.js starts it initializes the event loop.

### JavaScript

The even loop allows JavaScript to use callbacks and promises. JavaScript uses a _single call stack_ to keep track of what _function_ is currently executing and which function will be executed afterwards. A call stack is a _storage location_ array shaped managed by FIFO (First In First Out) policy.

When a function is invoked is put on the stack, if the function calls another function, the latter is put on top of the stack and so on. Getting an error causes a long message that shows up the execution path.

Jargon: stack frames, activation record, post-mortem debugging, nested function called up to the point where the stack trace has been generated, where the exception was thrown. On the top of the stacktrace you have the old function put on the stack, going down you have, step by step, the most recent one. Stacktrace is a reverse order of the stack.

Every time an _async operation_ is invoked, it is added on top of the Event Table. Event Table know that a certain function should be triggered when a certain event occurs. Event Table just keep tracks of events and send them to the Event Queue which store the order the function must be executed.

The Event Loop is a constantly running process 
