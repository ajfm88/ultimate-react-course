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

## Managing Complex State With Reducers

### Why useReducer?

👉 State management with `useState` **is not enough in certain situations**:

1. Components with **a lot of state variables and state updates**, spread across many event handlers **all over the component** (or even multiple components)
2. **Multiple state updates need to happen at the same time**, as a reaction to the same event (e.g. "starting a game": score → 0, `isPlaying` → true, start a timer)
3. Updating one piece of state **depends on one or multiple other pieces of state**

> 👉 In all these situations, **`useReducer` can be of great help**.

### Managing State With useReducer

```jsx
const [state, dispatch] = useReducer(reducer, initialState);

function reducer(state, action) {
  switch (action.type) {
    case "dec":
      return state - 1;
    case "inc":
      return state + 1;
    case "setCount":
      return action.payload;
    default:
      throw new Error("Unknown");
  }
}
```

- 👉 An **alternative way of setting state**, ideal for **complex state** and **related pieces of state**
- **`state`**: related pieces of state are usually stored together in a **state object** (can be a primitive, but usually an object)
- **`reducer`**: function containing **all the logic to update state**. **Decouples state logic from the component** → cleaner, more readable components. Like **`setState()` with superpowers**
- **`reducer` is a pure function** (🚨 **no side effects!**) that takes the **current `state` and `action`** and **returns the next state**. State is immutable, so it must **return a new state**, never mutate
- **`action`**: an **object that describes how to update state**. Usually has a **`type`** and a **`payload`** (input data); the reducer uses them to determine the next state
- **`dispatch`**: function to **trigger state updates** by **"sending" actions from event handlers to the reducer**. Used **instead of `setState()`**

### How Reducers Update State

```
event handler → dispatch(action) → reducer(currentState, action) → RETURNS next state → RE-RENDER
```

- Example action: `{ type: "updateDay", payload: 23 }`: probably means "set `day` to 23"
- The action doesn't need this exact shape, but `type` + `payload` is the **standard** adopted by most developers
- `dispatch` has access to the reducer (passed to `useReducer`) and gives it the **current state**
- Updating state triggers a **re-render** of the component instance, as always
- 👉 **Why "reducer"?** Same idea as `array.reduce()`: it **accumulates all actions into one single state over time**

| | `useState` | `useReducer` |
|---|---|---|
| **Update with** | `setState(updatedState)` | `dispatch(action)` |
| **Flow** | setter → next (updated) state → re-render | dispatch → **reducer** (current state + action) → next state → re-render |
| **Setup** | Simpler, more straightforward | More complicated, but solves the problems above |

### A Mental Model: The Bank

👉 **Task:** withdraw $5,000 from your bank account. You don't walk into the vault and grab the cash yourself: you tell the person at the desk what you want, and **they** do it on your behalf.

| Bank | useReducer | Role |
|---|---|---|
| **Customer** | **Dispatcher** (`dispatch`) | **Requests** the update |
| **Bank employee** | **Reducer** | **Makes** the update |
| **Request message** | **Action** | **How** to make the update |
| **Vault** | **State** | **What** needs to be updated |

```js
{ type: "withdraw", payload: { amount: 5000, account: 923577 } }
```

- The employee knows how to deposit, withdraw, open/close accounts, request loans… **without you worrying about the details**
- Just like a reducer **decouples and abstracts all state updating logic** away from components
