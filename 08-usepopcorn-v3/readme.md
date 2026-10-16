# Custom Hooks, Refs and more State

> Section: **Custom Hooks, Refs and more State** (~2 hr) — finalizing the **usePopcorn** project

## What Are React Hooks?

- Hooks are **special built-in functions** that let us **"hook" into React internals**, e.g.:
  - Creating and accessing **state** from the Fiber tree (`useState`)
  - Registering **side effects** in the Fiber tree (`useEffect`)
  - Manual **DOM selections** (`useRef`)
  - Many more…
- 👉 All hooks **start with `use`** (`useState`, `useEffect`, etc.) — this lets both us and React tell hooks apart from regular functions
- 👉 **Custom hooks** (also named `use…`) let us compose multiple hooks into our own reusable **non-visual logic**
- 👉 Give **function components** the ability to **own state** and **run side effects** at lifecycle points. Before **React 16.8**, this was only possible in **class components**

> 🔑 Hooks were a huge step forward for React — they're a big part of why it became even more popular.

### Overview of Built-in Hooks (React 18)

| Category | Hook | Status in this course |
|---|---|---|
| **Most used** | `useState` | ✅ Learned |
| **Most used** | `useEffect` | ✅ Learned |
| **Most used** | `useReducer` | 👉 Will learn |
| **Most used** | `useContext` | 👉 Will learn |
| **Less used** | `useRef` | 👉 Will learn |
| **Less used** | `useCallback` | 👉 Will learn |
| **Less used** | `useMemo` | 👉 Will learn |
| **Less used** | `useTransition` | 👉 Will learn |
| **Less used** | `useDeferredValue` | 👉 Will learn |
| **Less used** | `useLayoutEffect` | ❌ Will not learn |
| **Less used** | `useDebugValue` | ❌ Will not learn |
| **Less used** | `useImperativeHandle` | ❌ Will not learn |
| **Less used** | `useId` | ❌ Will not learn |

- Roughly **20 built-in hooks** in total. The ❌ ones are obscure or intended only for **library authors**

## The Rules of Hooks

**1. Only call hooks at the top level**

- ❌ Don't call hooks inside **conditionals**, **loops**, **nested functions**, or **after an early return**
- Why: hooks must be called in the **same order on every render** — React relies on this

**2. Only call hooks from React functions**

- ✅ Only inside a **function component** or a **custom hook**
- ❌ Not from regular functions or class components

> 👉 These rules are **automatically enforced by React's ESLint rules**, so the linter will flag violations.

## Why Hooks Rely on Call Order

**How the Fiber tree stores hooks**

1. **Initial render:** React builds the **Fiber tree** from the **React element tree** (the virtual DOM)
2. Each **fiber** holds props, a list of work, and a **linked list of the hooks** used by that component instance
3. The list is built in the **call order** of the hooks — and that **order number uniquely identifies each hook**

**Correct code** (every hook called unconditionally, in the same order):

```jsx
const [A, setA] = useState(23);   // 1
const [B, setB] = useState('');   // 2
useEffect(fnZ, []);               // 3
```

→ Linked list: `State A → State B → Effect Z`. On **re-render**, the list has the **same order** → React matches each hook to its stored value correctly.

**Why breaking the rule breaks everything** (conditional hook):

```jsx
const [A, setA] = useState(23);
if (A === 23) {
  const [B, setB] = useState('');  // ❌ conditionally called
}
useEffect(fnZ, []);
```

1. Initially `A === 23` → `State B` is created and linked into the list
2. A re-render sets `A` to `7` → the condition is false → `useState` for `B` is **not called**
3. But fibers (and their hook lists) are **not recreated** on every render — the first hook still points to the old `State B` link, which no longer corresponds to a call. `Effect Z` is now orphaned and nothing points to it → **the linked list is broken**
4. React is confused and can't correctly track which hook is which

> 🔑 **Conclusion:** hooks must be called in the same order on every render, which is only guaranteed by calling them **at the top level** — exactly what Rule #1 says.

**Why use a linked list at all?**

- It's the simplest way to **associate each hook with its value** based on call order
- Developers **don't have to manually name** each hook — the order does it for us

## Initializing State With a Callback (Lazy Initial State)

Persisting the watched list in `localStorage`: **write** on every change, **read** once on mount.

```jsx
// Read: initialize state from localStorage on the initial render only
const [watched, setWatched] = useState(function () {
  const storedValue = localStorage.getItem("watched");
  return JSON.parse(storedValue);
});

// Write: keep localStorage in sync whenever watched changes
useEffect(
  function () {
    localStorage.setItem("watched", JSON.stringify(watched));
  },
  [watched]
);
```

- 👉 `useState` also accepts a **callback function**. React calls it **once, on the initial render**, and uses its return value as the initial state. It's **ignored on re-renders**
- The callback must be **pure** and take **no arguments**
- ☝️ Whenever the initial state depends on some **computation**, pass a function in. Don't call one:
  - ✅ `useState(fn)`: React calls it once
  - ❌ `useState(fn())`: the function runs on **every render**, even though React ignores the result after the first
- Writing in an **effect** rather than in `handleAddWatched` **synchronizes** `watched` with `localStorage`, so deleting a movie updates storage automatically. In the event handler, `watched` would also still be **stale**, so you'd have to build `[...watched, movie]` by hand
- `localStorage` only stores **strings**: `JSON.stringify` on write, `JSON.parse` on read

## Summary of Defining and Updating State

**1. Creating state**

| | Code | Notes |
|---|---|---|
| **Simple** | `const [count, setCount] = useState(23);` | The most common way |
| **Based on a function** (lazy evaluation) | `const [count, setCount] = useState(() => localStorage.getItem('count'));` | Use when the initial state depends on a **computation** |

- 👉 The function must be **pure** and accept **no arguments**. It's called **only on the initial render** and ignored on re-renders

**2. Updating state**

| | Code | Notes |
|---|---|---|
| **Simple** | `setCount(1000);` | Pass the next state directly |
| **Based on current state** | `setCount((c) => c + 1);` | Callback must be **pure** and **return the next state** |

- 👉 The callback form is the **preferred way** of updating state whenever it makes sense (e.g. incrementing a counter)
- 🚨 **Never mutate objects or arrays** in state. **Replace** them with a new object/array that includes the changes, and pass that to the setter

> 🔑 Both creating and updating state have a **simple form** and a **callback form**. Use the callback when the value depends on a computation (creating) or on the current state (updating).

## Introducing Another Hook: useRef

### What Are Refs?

```jsx
const myRef = useRef(23);
myRef.current = 1000; // write
myRef.current;        // read
```

- 👉 A **ref** ("reference") is a **"box" (object) with a mutable `.current` property** that is **persisted across renders** ("normal" variables are always reset)
- `useRef(initialValue)` sets the initial `.current`; we read and write it via `.current`
- 👉 `.current` is **mutable**, unlike everything else in React

**Two big use cases**

1. Creating a **variable that stays the same between renders** (e.g. **previous state**, **`setTimeout` id**)
2. **Selecting and storing DOM elements** (a DOM element is just data to preserve across renders)

- 👉 Refs are for **data that is NOT rendered**: they usually appear only in **event handlers or effects**, not in JSX. Data that participates in the visual output → use **state**
- ☝️ **Do NOT write or read `.current` in render logic** (like state): it would be a side effect. Mutate refs inside **`useEffect`** (or event handlers)

### State vs. Refs

Refs are like **state with less powers**.

| | Persists across renders | Updating causes re-render | Immutable | Asynchronous updates |
|---|:---:|:---:|:---:|:---:|
| **State** | ✅ | ✅ | ✅ | ✅ |
| **Refs** | ✅ | ❌ | ❌ | ❌ |

- Both are **remembered across renders**; the big difference is that **updating state re-renders the component, updating a ref does not**
- Refs are **not async**: the new `.current` can be read **immediately** after updating, like any regular JS object (state can't)

```
Need to store data
  → Will data change at some point?
      NO  → Regular const variable
      YES → Should it re-render the component?
              NO  → Ref (useRef)      e.g. preserving previous state, storing a setTimeout id
              YES → State (useState)
```

> 🔑 Use **state** for data that should **re-render** the component; use **refs** for data the component should only **remember over time**, never re-render for.

## Reusing Logic With Custom Hooks

### What Are Custom Hooks and When to Create One

- 👉 In React we can reuse two things: **UI** and **logic**

```
"I need to reuse:"
  UI    → Component
  LOGIC → Does the logic contain any hooks?
            NO  → Regular function (inside or outside the component)
            YES → Custom hook
```

- Custom hooks let us reuse **non-visual logic** in multiple components: stateful logic, and really **any logic that contains one or more React hooks**
- A logic with hooks **can't** be extracted into a regular function, so it needs a custom hook
- 👉 **One custom hook, one purpose**: it should do one specific, well-defined thing, to make it **reusable and portable** (even across projects). Don't just dump all of a component's hooks into one custom hook
- Many custom hook **libraries** are shared on **NPM**
- 🚨 **Rules of hooks apply to custom hooks too**, since they're built from regular hooks

### Anatomy of a Custom Hook

```jsx
function useFetch(url) {
  const [data, setData] = useState([]);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(function () {
    fetch(url)
      .then((res) => res.json())
      .then((res) => setData(res));
  }, []);

  return [data, isLoading];
}
```

- **Function name must start with `use`** (`useFetch`): this is **not optional**. Otherwise it's just a regular function in the eyes of React (and the linter)
- **Needs to use one or more hooks** (here: two `useState` + one `useEffect`). That is what separates it from a regular function
- **Unlike components**, which only receive props and must return JSX, a custom hook can **receive and return any relevant data** (usually `[]` or `{}`)

| | Component | Custom hook |
|---|---|---|
| **Is** | A JS function | A JS function |
| **Receives** | Props only | Any data |
| **Returns** | JSX | Any data (usually an array or object) |
| **Name** | Capitalized | Starts with `use` |
