# Effects and Data Fetching

> Section: **Effects and Data Fetching** (~3 hr) — continuing the **usePopcorn** project, now with real data

## Section Overview

- 👉 Roughly **95% of React apps** fetch data from some API → **data fetching is an essential skill**
- 👉 One way to fetch data in React is **inside an effect** — that's the focus of this section
- 👉 Loading external data makes the app feel **real-world and alive**

**What we'll cover**

- 👉 Data **fetching** is essential
- 👉 **Effects** with the **`useEffect`** hook (how and when effects execute)
- 👉 Effect **cleanup**
- 👉 A **real-world** application (usePopcorn with live movie search: results list + movie details, rating, "Add to list")

## The Component (Instance) Lifecycle

> 👉 Technically only a component **instance** goes through a lifecycle — "component" is used as shorthand (as everyone does). Relevant for the rest of the section because we can **hook into** lifecycle phases with `useEffect`.

```
🐣 MOUNT / INITIAL RENDER  →  🐔 RE-RENDER (optional)  →  💀 UNMOUNT
```

**1. Mount / initial render** 🐣 — the instance is **born**

- Rendered for the **first time**
- **Fresh state and props** are created

**2. Re-render** 🐔 *(optional — not every component re-renders; some are mounted and unmounted right away)*

- Can happen **any number of times**, when:
  - **State** changes
  - **Props** change
  - **Parent** re-renders
  - **Context** changes (more later)
- 👉 Extends the earlier "state update → whole app re-renders" idea to the level of a **specific component instance**

**3. Unmount** 💀 — the instance **dies**

- Instance is **destroyed and removed** from the screen, along with its **state and props**
- E.g. user navigates to another page/section, or closes the app
- A **new instance** of the same component can be mounted later — but *this* instance is gone

> 🔑 **Why it matters:** we can **define code to run at these specific points in time** (on mount, on re-render, on unmount) — done with the **`useEffect`** hook, the big topic of this section.

## How NOT to Fetch Data in React

We fetched movie data from the OMDb API directly in the component's top-level code, deliberately breaking the "no side effects in render logic" rule:

```jsx
export default function App() {
  const [movies, setMovies] = useState([]);
  const [watched, setWatched] = useState([]);

  fetch(`http://www.omdbapi.com/?apikey=${KEY}&s=interstellar`)
    .then((res) => res.json())
    .then((data) => setMovies(data.Search));
```

Just logging the data looks fine, but calling `setMovies` there creates an infinite loop:

1. The fetch runs during render and `setMovies` updates state
2. The state update re-renders the component, so the function body runs again
3. That runs the fetch again, which sets state again, and so on forever

The network tab shows endless requests to the API. Setting state at the top level with no fetch at all (e.g. `setWatched([])`) makes React throw a "too many re-renders" error.

The API key belongs in a variable declared outside the component so it isn't recreated on every render. The fix is the `useEffect` hook, covered next.

## A First Look at Effects

We just used `useEffect` for the first time to fetch movie data as the component mounts. What is an effect, and how does it differ from an event handler? (Details in the next slides.)

### Where to Create Side Effects

**Review: what is a side effect?**

- Any **interaction between a React component and the world outside it**, or "code that actually does something"
- Examples: **data fetching**, setting up **subscriptions**, setting up **timers**, **manually accessing the DOM**
- We need side effects **all the time** (they make apps *do something*), but **never in render logic**

**Two places to create them**

| | Event handlers | Effects (`useEffect`) |
|---|---|---|
| **Triggered by** | **Events**: `onClick`, `onSubmit`, etc. | **Rendering** |
| **Use when** | Reacting to a user event is enough | The code must run **automatically as the component renders**, not in response to an event |

- 👉 Reacting to events is **sometimes not enough** for what an app needs, which is where effects come in
- 👉 An effect lets us write code that runs at different moments of the component instance lifecycle: **mount, re-render, or unmount**

### Event Handlers vs. Effects

Fetching movie data is a side effect, and it can be done in two places. Both produce the **same result, but at different moments**:

```jsx
// Event handler: runs when the event happens
function handleClick() {
  fetch(`http://www.omdbapi.com/?s=inception`)
    .then((res) => res.json())
    .then((data) => setMovies(data.Search));
}

// Effect: runs after the component renders
useEffect(function () {
  fetch(`http://www.omdbapi.com/?s=inception`)
    .then((res) => res.json())
    .then((data) => setMovies(data.Search));

  return () => console.log('Cleanup');
}, []);
```

**The 3 parts of an effect**

1. **Effect code** (the function body)
2. **Cleanup function** (optional): returned from the effect, called **before the component re-renders or unmounts**
3. **Dependency array** (`[]`): controls **when** the effect runs

| | Event handlers | Effects (`useEffect`) |
|---|---|---|
| **Executed** | When the **corresponding event happens** | **After the component mounts** (initial render) and **after subsequent re-renders** (according to the dependency array) |
| **Used to** | **React** to an event | Keep a component **synchronized with an external system** (here: the movie data from the API) |

- 👉 **Think synchronization, not lifecycles.** Mount/re-render/unmount is a helpful mental model, but the real purpose of effects is to keep the component **in sync with the external world**
- ☝️ **Event handlers are the preferred way of creating side effects.** Don't overuse `useEffect`: anything that can be handled in an event handler should be

> We'll come back to all of this after using `useEffect` more in practice.
