# Advanced State Management: useReducer

> Section: building the **React Quiz** app, starting with a first look at the **`useReducer`** hook

## A First Look at useReducer

Project setup: `npx create-react-app@5 react-quiz`, then the usual cleanup. Warm-up: replace the `useState` hooks of the **`DateCounter`** component (from an earlier challenge) with `useReducer`.

- 👉 `useReducer` is a **more advanced and more complex way of managing state** than `useState`
- It works with a **reducer function**: a **pure function** that takes the **current state** and an **action** and **returns the next state**

### Step 1: Wire up the hook

```jsx
const [count, dispatch] = useReducer(reducer, 0);
//      ↑ state  ↑ updater   ↑ reducer fn, ↑ initial state
```

- `useReducer(reducer, initialState)` takes **two arguments**: the reducer function and the initial state
- It returns the **current state** and a **`dispatch` function** (instead of a state setter)
- Calling `dispatch(x)` runs the reducer with `(currentState, x)`: here `x` is the **action**

```jsx
function reducer(state, action) {
  console.log(state, action); // 0 1
  return state + action;      // whatever is returned becomes the NEXT state
}
```

- 👉 Calling `dispatch` is called **dispatching an action**
- At this point it's more work than `useState` for the same result; it pays off as complexity grows

### Step 2: Name the actions

Passing a bare value breaks `setCount`: typing `0` into the input would **add** `0` to the current state instead of replacing it. The reducer needs to know **what kind of update** is happening, so the action becomes an **object**:

```jsx
dispatch({ type: "dec", payload: 1 });
dispatch({ type: "inc", payload: 1 });
dispatch({ type: "setCount", payload: Number(e.target.value) });

function reducer(state, action) {
  if (action.type === "inc") return state + action.payload;
  if (action.type === "dec") return state - action.payload;
  if (action.type === "setCount") return action.payload;
}
```

- 👉 Convention: actions are objects with a **`type`** and an optional **`payload`** (the same convention is used later in **Redux**)
- In theory the action could have any shape; in practice always use `type` + `payload`
- The reducer **decides what to do based on `action.type`**, using the `payload` as the value passed in

### Step 3: Move logic into the reducer

- ☝️ The reducer should **know how to perform its own actions**: `inc`/`dec` just add/subtract 1, so **drop the payload** and dispatch `{ type: "inc" }` / `{ type: "dec" }`
- **`payload` is optional**: keep it only when the reducer needs a value it can't know itself (e.g. `setCount`)

### Recap

1. `useReducer(reducer, initialState)` → `[state, dispatch]`
2. Components **dispatch actions** (`type` + optional `payload`)
3. The **reducer** gets the **current state + action**, decides based on `action.type`, and **returns the next state**
4. That return value becomes the new state → component **re-renders** → UI updates
