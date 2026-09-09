# How React Works Behind the Scenes

> Section: **How React Works Behind the Scenes**

## Section Overview

- 👉 How things work **inside** React
- 👉 You'll become a **better** and more **confident** React developer
- 👉 Will be a bit **intense**... 😬
- 👉 Watch **at least the final lecture!**

### The Render and Commit Cycle (High-Level Overview)

#### The 4 Phases

```
[1] TRIGGER  →  [2] RENDER PHASE  →  [3] COMMIT PHASE  →  [4] BROWSER PAINT
```

**[1] Trigger** 💥

- 👉 Happens **only** on **initial render** and **state updates**
- An update starts with **updated React elements** (new JSX to render)

**[2] Render phase**

- Takes the **updated React elements** + the **current Fiber tree**, builds a **new Virtual DOM**, and runs **Reconciliation + Diffing** → produces an **updated Fiber tree** and a **list of DOM updates**
- 👉 Does **not** produce any visual output — nothing is painted to the screen yet
- 👉 Rendering a component **also renders all of its child components**
- 👉 **Asynchronous**: work can be **split, prioritized, paused, resumed**

**[3] Commit phase**

- Takes the **list of DOM updates** and writes them to the **actual DOM** → **Updated DOM**
- 👉 **Synchronous**: DOM updates are written **in one go**, to keep the UI consistent

**[4] Browser paint**

- The browser repaints the screen → **Updated UI on screen**

#### Reconciliation + Diffing

```
CURRENT FIBER TREE          UPDATED FIBER TREE (workInProgress)
      App                          App
   ┌───┼────┐                   ┌───┼────┐
 Video Modal Btn      →       Video Modal Btn
        │                            │
     Overlay                     Overlay
     ┌──┴──┐                     ┌──┴──┐
    h3   button                 h3   button
```

- Example: toggling `showModal` from `true` to `false` produces a **new Virtual DOM**
- React compares the **current Fiber tree** against the **new Virtual DOM** using **Reconciliation + Diffing**
- 🔑 Elements are compared based on their **position in the tree**
- The result is an **updated Fiber tree**, marking exactly what needs to change in the DOM: text updates, deletions, etc.
- Only the **DOM work** that's actually needed gets flagged — here, `Modal`, `Overlay`, `h3`, and `button` are marked for **deletion**, and `Btn`'s text is marked for an **update**

> 🔑 **Takeaway:** React doesn't throw away and rebuild the whole DOM on every update. It builds a new Virtual DOM, diffs it against the previous Fiber tree by tree position, and only touches the DOM nodes that actually changed.

## Project Setup and Walkthrough

> 🎒 This section uses a small pre-built demo project (this `06-how-react-works` folder) to explore Fiber, rendering, state, and closures in later lectures.

### Setup — different from previous sections

- 👉 This project was **not** created with `npx create-react-app`
- Instead, the **starter files were copied in directly** (source folder, public folder, config files already provided)
- Only `npm install` was needed to pull in `node_modules` (React, ReactDOM, etc., as listed in `package.json`), then `npm start`

### Why start from someone else's code?

- 👉 On a real team, you constantly need to **read and understand code you didn't write**
- 👉 This project simulates exactly that: pretend another developer on the team already wrote it
- 👉 **Practice reading strategy:** start at the entry component (`App`), follow props down, use VS Code's **double-click → highlight all usages** to trace where a variable/prop is read and set, and check the **React DevTools component tree** to see the actual structure

### Component Tree

```
App
 └── Tabbed
      ├── Tab (num=0) ├── Tab (num=1) ├── Tab (num=2) ├── Tab (num=3)
      └── TabContent  (or DifferentContent, when num === 3)
```

### `App`

- Defines a `content` array of objects (`{ summary, details }`) — hardcoded **data**
- Renders `<Tabbed content={content} />`, passing the array down as a **prop**

### `Tabbed`

- Owns the **`activeTab` state** (a number, starting at `0`) — this is the piece of state that decides which tab is showing
- Renders 4 `<Tab>` components, each passed:
  - `num` — that tab's own index (`0`–`3`)
  - `activeTab` — the current state, so each `Tab` knows if **it** is the active one
  - `onClick` — the **`setActiveTab` state setter itself**, passed straight down as a prop (no wrapper function needed)
- Conditionally renders the content area:
  ```jsx
  {activeTab <= 2 ? (
    <TabContent item={content.at(activeTab)} />
  ) : (
    <DifferentContent />
  )}
  ```
  - `content.at(activeTab)` — modern JS array access (equivalent to `content[activeTab]`), pulling out the object at the current tab's position
  - Tabs `0`–`2` render `TabContent` with the matching data object; tab `3` renders a **completely different component**, `DifferentContent`

### `Tab`

- Just a `<button>` — receives `num`, `activeTab`, `onClick` as props
- `className` toggles `"tab active"` vs `"tab"` by comparing `activeTab === num`
- `onClick={() => onClick(num)}` — clicking calls the passed-down `setActiveTab(num)`, which updates state in the **parent** (`Tabbed`) → **child-to-parent communication**

### `TabContent`

- Receives the current `item` (`{ summary, details }`) as a prop, and displays `item.summary` / `item.details`
- Owns **two of its own state variables**, local to this component:
  - `showDetails` (boolean, default `true`) — conditionally renders the `<p>{item.details}</p>` paragraph (`showDetails && <p>...</p>`)
  - `likes` (number, default `0`) — incremented by `handleInc`
- The **"Hide details" button** toggles `showDetails` using the **updater function** form: `setShowDetails((h) => !h)`
- The **`+` heart button** calls `handleInc`, which does `setLikes(likes + 1)` — note this does **not** use the updater-function form (based on the *current* `likes` closure value) — more on why that matters in later lectures on **stale closures**
- The **`+++` button** and the two **Undo buttons** have **no `onClick` handler attached** — clicking them does nothing (left as-is for now, revisited later in the section)

### `DifferentContent`

- A minimal component — just an `<h4>` message
- Rendered instead of `TabContent` whenever tab `3` is active
- 🔑 Because it's a genuinely **different component type** in that position of the tree (not just different props), React **unmounts** `TabContent` and its state (`showDetails`, `likes` reset) when switching to it, and **mounts** a fresh instance if switching back — this is the setup for a later lecture on how state resets when the element type at a tree position changes

> 🔑 **Takeaway:** before diving into how React works internally, make sure you can trace this demo end-to-end yourself — follow every prop from where it's defined to where it's used, and know exactly which state lives in which component.

## Component vs. Instance vs. Element

> 🎒 A common interview question — understanding the difference clarifies what actually happens as your components get used.

### Component

```jsx
function Tab({ item }) {
  return (
    <div className="tab-content">
      <h4>All contacts</h4>
      <p>Your post will be visible</p>
    </div>
  );
}
```

- 👉 A **description** of a piece of UI
- A component is just a regular **JavaScript function** that **returns React elements** (an element tree), usually written with **JSX**
- 👉 A **generic** description of the UI — think of it as a **"blueprint"** or **"template"**
- React creates **one or multiple component instances** out of that one blueprint/template

### Component Instance

```jsx
function App() {
  return (
    <div className="tabs">
      <Tab item={content[0]} />
      <Tab item={content[1]} />
      <Tab item={content[2]} />
    </div>
  );
}
```

```
             App
        ┌─────┼─────┐
       Tab   Tab   Tab   ← instances of Tab
```

- 👉 Instances are created each time we **"use"** a component — here `Tab` is used **three times**, so React places **three instances** of `Tab` in the component tree
- Behind the scenes, React **internally calls `Tab()`** once for each instance
- An instance is the actual **"physical" manifestation** of a component living in the component tree, while the **component** itself is just the function we wrote, before being called
- 👉 Each instance **holds its own state and props**, and has its own **lifecycle** — it can **"be born," "live"** for a while, and eventually **"die"** (a bit like a living organism)
- In practice, **"component"** and **"component instance"** are often used **interchangeably** (e.g. "component lifecycle" instead of "component instance lifecycle," "a UI is made of components" instead of instances) — technically less accurate, but common in docs/conversation
- As React executes the code in each instance, that instance **returns one or more React elements**

### React Element

```jsx
function Tab({ item }) {
  return (
    <div className="tab-content">
      <h4>All contacts</h4>
      <p>Your post will be visible</p>
    </div>
  );
}
```

↓ JSX compiles to ↓

```js
React.createElement(
  'div',
  { className: 'tab-content' },
  React.createElement('h4', null, 'All contacts'),
  React.createElement('p', null, 'Your post will be visible')
);
```

↓ produces ↓

```js
{
  $$typeof: Symbol(react.element),
  key: null,
  props: {
    children: [ /* h4 element, p element */ ],
    className: 'tab-content',
  },
  ref: null,
  type: 'div',
  _owner: null,
  _store: { validated: false },
}
```

- 👉 As we learned about JSX behind the scenes, **JSX is converted to `React.createElement()` function calls**
- A **React element** is the **result** of calling these `createElement` functions — i.e. the result of **using a component** in our code
- It's a big, **immutable JavaScript object** that React keeps in memory
- 👉 It contains all the **information necessary to eventually create DOM elements** for the current component instance

### DOM Element

```
Component → <Tab /> → Component Instance → RETURNS → React Element → INSERTED TO DOM → DOM Element (HTML)
```

- 👉 DOM elements are the **actual, final, visual representation** of the component instance in the browser
- 👉 It is **not** React elements that get rendered to the DOM — React elements just live **inside the React app** and have **nothing to do with the DOM**
- They are simply **converted to DOM elements** when they are painted onto the screen, as this final step

> 🔑 **Takeaway — the full journey:** write a **component** (blueprint) → **use** it, which creates one or more **component instances** in the tree, each with its own state/props/lifecycle → each instance **returns** a **React element** (an immutable JS object, the result of `React.createElement()`, produced by compiled JSX) → that React element is eventually **inserted into the DOM** as a real **DOM element (HTML)**, which is what actually gets painted on screen.
