---
date: '2025-02-26T21:40:25+05:30'
draft: false
title: 'A Bottom-up Approach to Async Modals in React'
tags: ["react"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
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
    image: "https://raw.githubusercontent.com/Corbe30/corbe30.github.io/refs/heads/main/assets/images/asyncModal_gif.gif" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---

In React, you can use combination of Context and custom hooks to implement async modals. Async modals let you capture the response from modal asynchronously, saving the hassle of managing another state and improves readibility.

Let's try to build it from the ground up:

## 1. Bare bones

<details open>
<summary>App.js</summary>

```jsx
function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);

  const onResponseClick = (response) => {
    setIsModalOpen(false);
    console.log(response);
  };

  return (
    <div>
      {isModalOpen && <Modal onResponseClick={onResponseClick} />}
      <button
        onClick={() => {
          setIsModalOpen(true);
        }}
      >
        Open Modal
      </button>
    </div>
  );
}
```

</details>

To get the response (yes/no) from the modal, App component is forced to manage `isModalOpen` state and include `<Modal />` component in its JSX. We know there is a better way to manage this by using Context.

## 2. `Context` for consolidation

Context can help us manage `isModalOpen` state and `Modal` component. But notice that the control for handling response moves to ModalContext.

<details open>
<summary>App.js</summary>

```jsx
function App() {
  const modalCtx = useContext(ModalContext);

  return (
    <div>
      <button onClick={() => {modalCtx.setIsModalOpen(true)}}>
        Open Modal
      </button>
    </div>
  );
}
```

</details>
<details open>
<summary>ModalContext.js</summary>

```jsx
export const ModalContext = createContext();
export const ModalProvider = ({ children }) => {
  const [isModalOpen, setIsModalOpen] = useState(false);

  const onResponseClick = (response) => {
    setIsModalOpen(false);
    console.log(response);
  };

  return (
    <ModalContext.Provider value={{ setIsModalOpen }}>
      {isModalOpen && <Modal onResponseClick={onResponseClick} />}
      {children}
    </ModalContext.Provider>
  );
};
```

</details>

To solve this, ModalContext can maintain another state `response` which will be observed by an `useEffect` in App component. Even so, the new `response` might be [same as the previous](https://stackoverflow.com/a/52625068), so we will need a `responseKey` which will be unique for every response.

This seems like a hassle:

1. we don't want App to deal with context
2. but want App to have control over response

A **custom hook** will help us do both.

## 3. Introducing `useAsyncModal`

<img src="https://raw.githubusercontent.com/Corbe30/corbe30.github.io/refs/heads/main/assets/images/asyncModal_flow.png" />

We create `useAsyncModal` hook, which will lie between ModalContext and App. It will manage context, and provide App component with a method to get response. `useAsyncModal` creates a promise, which is only resolved when the user clicks on either of the modal buttons.

<details open>
<summary>App.js</summary>

```jsx
function App() {
  const { confirm } = useModal();

  const onClickHandler = async () => {
    const response = await confirm();
    console.log(response);
  };

  return (
    <div>
        <button onClick={onClickHandler}>Open Modal</button>
    </div>
  );
}
```

</details>

<details open>
<summary>useAsyncModal.js</summary>

```jsx
const useAsyncModal = () => {
  const modalCtx = useContext(ModalContext);

  const confirm = async () => {
    modalCtx.setIsModalOpen(true);
    const response = await modalCtx.forUserToConfirm();
    return response;
  };
  return { confirm };
};
```

</details>

<details open>
<summary>ModalContext.js</summary>

```jsx
export const ModalContext = createContext();
export const ModalProvider = ({ children }) => {
  const [settlePromise, setSettlePromise] = useState({});
  const [isModalOpen, setIsModalOpen] = useState(false);

  const onResponseClick = (val) => {
    setIsModalOpen(false);
    settlePromise.resolve(val);
  };

  const forUserToConfirm = () => {
    const confirmPromise = new Promise((resolve, reject) => {
      setSettlePromise({ resolve, reject });
    });
    return confirmPromise;
  };

  return (
    <ModalContext.Provider value={{ setIsModalOpen, forUserToConfirm }}>
      {isModalOpen && <Modal onResponseClick={onResponseClick} />}
      {children}
    </ModalContext.Provider>
  );
};

```

</details>

This Modal can now be used by any component, all while improving the code readability! You can view the code at my [Github repo](https://github.com/Corbe30/async-modals). It contains 3 branches, one for each step of this article. 