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
