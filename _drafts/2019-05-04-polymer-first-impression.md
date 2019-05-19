---
layout: single
title: "Polymer: first impression"
date: '2019-05-04'
last_modified_at: '2019-05-04'
comments: true
categories:
  - polymer, javascript
tags:
  - [polymer, javascript]
toc: true
toc_label: Contents
toc_icon: "align-right"
toc_sticky: true
---

## Getting Started

The Polymer library provides a set of features for creating custom elements. These features are designed to make it easier and faster to make custom elements that work like standard DOM elements.

- components library
- expressive templating system
- two way data binding
- library for factoring web components and component-based apps

### Polymer porject vs. Polymer library

The project is about the plaform at large, the library is about helping you to build components.

### Setup

```bash
npm install -g polymer-cli
```

CLI projects types:

- [Elements project](https://polymer-library.polymer-project.org/3.0/docs/tools/create-element-polymer-cli): expose a single element or group of related elements.
- [Application project](https://polymer-library.polymer-project.org/3.0/docs/tools/create-app-polymer-cli): build an application, composed of Polymer elements.
  
Let's create an element project.

```bash
mkdir polymer-row-element && cd polymer-row-element
polymer init
```

then select `polymer-3-element` as starter templarte and then add name with an hyphen and a description.

The initialization creates the following files and folders:

```bash
├── README.md
├── demo
├── index.html //Automatically-generated API reference.
├── node_modules
├── package-lock.json
├── package.json
├── polymer.json //Configuration file for Polymer CLI.
├── row-element.js //Element source code.
└── test //Unit tests for the element.
```

```bash
polymer build // production ready build
polymer serve build/default //serve a build locally
```

## Links

- https://github.com/Polymer/polymer

## Reference

- Introduction: https://dev.to/bennypowers/lets-build-web-components-part-4-polymer-library-4dk2
- Feature overview: https://polymer-library.polymer-project.org/3.0/docs/devguide/feature-overview
- Polymer CLI: https://polymer-library.polymer-project.org/3.0/docs/tools/polymer-cli
- Custom element convention: https://www.w3.org/TR/2016/WD-custom-elements-20160226/#concepts
- Build for production: https://polymer-library.polymer-project.org/3.0/docs/apps/build-for-production#transforms