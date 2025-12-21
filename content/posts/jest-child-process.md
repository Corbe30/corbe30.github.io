---
date: '2025-06-03T21:34:10+05:30'
title: 'Jest Worker and his 4 Horrifying Children'
# weight: 1
# aliases: ["/first"]
tags: ["jest", "react"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
---

When writing unit tests using Jest and React Testing Library, you might encounter this error:

```
Jest Worker Encountered 4 Child Process Exceptions, exceeding retry limit
```

This usually signals that Jest retried your test multiple times across its worker threads but failed each time, eventually giving up. Here's how I traced the root cause of this issue and resolved it.

## The Culprit: fireEvent and Asynchronous Side Effects
At first glance, my test code looked fine. I was using `fireEvent.click` to simulate user interaction. But here's the catch — while `fireEvent` is a synchronous function, the side effects it triggers (like state updates or calls inside `useEffect`) are often asynchronous.

This led to race conditions in my tests. One `fireEvent` would trigger some async behavior, and before that completed, the next `fireEvent` would run. This overlap created unpredictable behavior, and Jest interpreted it as a failing test, retrying it across different workers until the retry limit was exceeded.

References:
* [Async function without await in JavaScript](https://stackoverflow.com/questions/45594596/async-function-without-await-in-javascript)
* [Dev.to article on Jest Worker Exceptions](https://dev.to/ninthsun91/jest-worker-encountered-4-child-process-exceptions-exceeding-retry-limit-82o)

## The Fix: Wrap fireEvent in waitFor
Since fireEvent isn't async, it won’t wait for the event loop to flush out async side effects. That's where `waitFor` comes in.

```javascript
await waitFor(() => fireEvent.click(button));
```
`waitFor` ensures that your test pauses until any resulting async behavior completes. Keep in mind that `fireEvent` is already wrapped in `act()` internally by Jest, so you don’t need to do that manually. But wrapping it in `waitFor()` is essential if you're seeing race conditions.

For deeper understanding of `act()` vs `waitFor()`, check out:
* [React Testing Library: act and waitFor](https://medium.com/@AbbasPlusPlus/act-and-waitfor-react-testing-library-dba78bb57e30)

## Event Queue vs Worker Threads
It's important to understand that async tasks (e.g., a `setTimeout`, fetch, or `useEffect`) don’t create new threads. They’re queued in the event loop. Jest spawns new worker threads only for new unit tests, not for async operations inside a test.

So, if `fireEvent.click()` triggers an async `useEffect`, it’s still the same thread — it's just relying on the JavaScript event loop to handle timing.

## A Better Way: Use Default Props for Inputs

Using `waitFor()` adds delays. If you have 5+ `fireEvent.click()` calls wrapped in `waitFor()`, your unit test will become noticeably slower.

Rather than simulating all UI input changes via `fireEvent`, you can pass default values as props:

```javascript
const MyPreferences = (
    defaultSize,
    defaultColor
) => {
  // component logic
};
```

This makes the component easier to test and avoids unnecessary `fireEvent` calls.

While it's ideal to keep test logic and component logic separate, sometimes you need to tweak components slightly to improve testability — and that’s okay. Design decisions should balance testability and maintainability.

## Bonus Tip: Speed Up Jest with --maxWorkers=50%
Surprisingly, running Jest with fewer workers can speed up your tests in some cases:
```
jest --maxWorkers=50%
```
Spawning and managing too many worker threads has its own cost. By reducing the number of workers, you may see faster execution due to reduced overhead. Reference: [Why does Jest run faster with maxWorkers=50%?](https://stackoverflow.com/questions/71287710/why-does-jest-run-faster-with-maxworkers-50)