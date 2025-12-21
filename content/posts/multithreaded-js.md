---
date: '2025-12-21T20:45:17Z'
draft: false
title: 'Multithreaded Js'
tags: ["algorithm", "multithread", "javascript"]
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: true
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://raw.githubusercontent.com/Corbe30/corbe30.github.io/refs/heads/main/assets/images/MULTITHREADED%20JAVASCRIPT%20%5BBROWN%20BAG%20SESSION%5D_pages-to-jpg-0001.jpg" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---

## Understanding the Fundamentals: Concurrency vs. Parallelism

Before diving into workers, it is essential to distinguish between how tasks are managed:

<img src="https://raw.githubusercontent.com/Corbe30/corbe30.github.io/refs/heads/main/assets/images/MULTITHREADED%20JAVASCRIPT%20%5BBROWN%20BAG%20SESSION%5D_pages-to-jpg-0003.jpg" />

> ###### source: https://medium.com/swift-india/concurrency-parallelism-threads-processes-async-and-sync-related-39fd951bc61d

*  **Concurrency**: Executing multiple tasks over overlapping time, though not necessarily at the exact same moment. In a single thread, this looks like tasks taking turns.


*  **Parallelism**: This is concurrency in a multi-threaded environment where tasks run simultaneously.

<img src="https://raw.githubusercontent.com/Corbe30/corbe30.github.io/refs/heads/main/assets/images/MULTITHREADED%20JAVASCRIPT%20%5BBROWN%20BAG%20SESSION%5D_pages-to-jpg-0004.jpg" />

> ###### source: https://medium.com/swift-india/concurrency-parallelism-threads-processes-async-and-sync-related-39fd951bc61d

* **The Synchronous Model**: Each task must wait for the previous one to complete before starting.


*  **The Asynchronous Model**: Tasks can start without waiting for others to finish. However, it is a common misconception that async code makes a single thread "faster"—it simply prevents blocking.

## Is JavaScript Really Single-Threaded?

Technically, yes. JavaScript itself lacks built-in functionality to create threads or even make system calls for networking and filesystems. Even `setTimeout()` is not a native JavaScript feature. Instead, these capabilities are provided by the **Runtime Environment** (like Node.js or Browsers) via specific APIs.

---

## Web Workers

Web Workers allow you to spawn new environments to execute JavaScript in, effectively enabling multithreading. Communication between the main thread and workers is strictly **event-driven**.

### 1. Dedicated Workers

These are used to parallelize computationally intensive tasks to keep the UI responsive.

> Code example: https://github.com/Corbe30/multithreaded-js/tree/main/01-browsers/01-dedicated-worker

* **Note**: None of the workers have access to the `document` (DOM) but can use `localStorage` and `indexedDB`.

### 2. Shared Workers

Unlike dedicated workers, a Shared Worker can be accessed by multiple tabs from the same origin.

> Code example: https://github.com/Corbe30/multithreaded-js/tree/main/01-browsers/02-shared-worker

* **Stay Alive**: They persist until the very last tab is closed.

* **Use Case**: Managing a single Server-Sent Events (SSE) or WebSocket connection to save network traffic across multiple open tabs.


### 3. Service Workers

Service workers act as a proxy between your app and the network.

> Code example: https://github.com/Corbe30/multithreaded-js/tree/main/01-browsers/03-service-worker

* **Persistence**: They can exist even after all tabs are closed.

* **Use Case**: Enabling offline functionality, caching static assets, and handling push notifications.

---
## Promisifying Workers

When working with Web Workers, the native communication is strictly event-driven, using postMessage() to send data and onmessage to receive it. Promisifying these workers wraps this message-passing logic into a Promise structure, making the asynchronous code cleaner and easier to read.

Note that combining workers and promises does not provide any benefit in terms of execution speed. It's just a 'syntacting sugar'.

> Code example: https://github.com/Corbe30/multithreaded-js/tree/main/01-browsers/04-promises-worker/1p-4w

---

## Structured Clone Algorithm

It turns out that both Chrome and Safari defer running StructuredDeserialize() until you actually access the `.data` property on the `MessageEvent`.

You can think of StructuredSerialize() and StructuredDeserialize() as smarter versions of JSON.stringify() and JSON.parse(), respectively. They are smarter in the sense that they handle cyclical data structures, built-in data types like Map, Set, etc.

---
## CPU vs. GPU

| Feature       | Web Workers                                   | GPU.js / GPGPU                                               |
|---------------|-----------------------------------------------|--------------------------------------------------------------|
| Resource used | CPU                                           | GPU                                                          |
| When          | Good for parallelism                          | Excellent for massive parallelism                            |
| Best for      | Long, sequential tasks                        | Identical, independent operations on large datasets          |
| Goal          | Maintain UI responsiveness by offloading work | Achieve maximum calculation throughput for suitable problems |