---
date: '2025-12-21T20:45:17Z'
draft: false
title: 'Multithreaded Js'
tags: ["algorithm", "babylonjs", "3D"]
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
    image: "https://raw.githubusercontent.com/Corbe30/3D-shape-editor/main/images/3dShapeEditor.png" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---



# Multithreaded JavaScript: Breaking the Single-Threaded Myth

JavaScript is famously known as a single-threaded language, but that doesn't mean we are stuck in a world of sequential execution. In a recent **Brown Bag Session**, Shashank Agarwal explored the depths of multithreading, the nuances of concurrency, and how modern web environments allow us to push the limits of browser compute.

---

## Understanding the Fundamentals: Concurrency vs. Parallelism

Before diving into workers, it is essential to distinguish between how tasks are managed:

*  **Concurrency**: Executing multiple tasks over overlapping time, though not necessarily at the exact same moment. In a single thread, this looks like tasks taking turns.


*  **Parallelism**: This is concurrency in a multi-threaded environment where tasks run simultaneously.


* **The Synchronous Model**: Each task must wait for the previous one to complete before starting.


*  **The Asynchronous Model**: Tasks can start without waiting for others to finish. However, it is a common misconception that async code makes a single thread "faster"—it simply prevents blocking.

## Is JavaScript Really Single-Threaded?

Technically, yes. JavaScript itself lacks built-in functionality to create threads or even make system calls for networking and filesystems. Even `setTimeout()` is not a native JavaScript feature. Instead, these capabilities are provided by the **Runtime Environment** (like Node.js or Browsers) via specific APIs.

---

## The Power of Web Workers

Web Workers allow you to spawn new environments to execute JavaScript in, effectively enabling multithreading. Communication between the main thread and workers is strictly **event-driven**.

### 1. Dedicated Workers

These are used to parallelize computationally intensive tasks to keep the UI responsive.

* **Note**: They do not have access to the `document` (DOM) but can use `localStorage` and `indexedDB`.


* **Data Transfer**: They use a **Structured Clone Algorithm**, which acts like a smarter version of JSON parsing to handle cyclical data and complex types.

### 2. Shared Workers

Unlike dedicated workers, a Shared Worker can be accessed by multiple tabs from the same origin.

* **Stay Alive**: They persist until the very last tab is closed.

* **Use Case**: Managing a single Server-Sent Events (SSE) or WebSocket connection to save network traffic across multiple open tabs.

### 3. Service Workers

Service workers act as a proxy between your app and the network.

* **Persistence**: They can exist even after all tabs are closed.

* **Use Case**: Enabling offline functionality, caching static assets, and handling push notifications.

---

## CPU vs. GPU: Choosing Your Tool

When pushing the limits of compute, you have two primary paths for parallelism:

| Feature       | Web Workers                                   | GPU.js / GPGPU                                               |
|---------------|-----------------------------------------------|--------------------------------------------------------------|
| Resource used | CPU                                           | GPU                                                          |
| When          | Good for parallelism                          | Excellent for massive parallelism                            |
| Best for      | Long, sequential tasks                        | Identical, independent operations on large datasets          |
| Goal          | Maintain UI responsiveness by offloading work | Achieve maximum calculation throughput for suitable problems |

---

## Conclusion and Next Steps

While combining Workers with **Promises** doesn't technically increase speed—Promises act as "syntactic sugar" for easier coding—it significantly improves code maintainability.

If you are ready to dive deeper:

* **For CPU Mastery**: Look into WebAssembly, Atomic Methods, and Thread Pools.

* **For GPU Mastery**: Explore Three.js or interactive 3D graphics courses.