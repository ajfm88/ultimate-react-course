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

> ⚠️ **Terminology trap:** React's "render phase" ≠ the everyday meaning of "render" (displaying stuff on screen). The everyday meaning = **render phase + commit phase** combined. Only the **commit phase** actually touches the DOM.

### How Renders Are Triggered

- Only **2** things trigger a render: **initial render** (app first runs) and a **state update** (re-render)
- 👉 A triggered render applies to the **entire application**, not just the component that changed — React re-runs *all* component functions from the root down
  - In practice it *looks* like only the updated component re-renders (what we assumed earlier in the course) — that's the practical effect, not what happens internally
- 👉 Renders aren't triggered instantly — they're **scheduled** for when the JS engine is free (usually imperceptible, a few ms). Multiple `setState` calls in the same event handler get **batched** into one render

### Correcting the Earlier "State → View" Mental Model

Two simplifications from the earlier `STATE → RENDER → UPDATED VIEW` diagram turn out to be **not literally true**:

- ❌ "Rendering updates the screen/DOM" — rendering is just calling component functions, no DOM touched
- ❌ "React discards the old view and replaces it entirely on re-render" — the DOM is **not** thrown away wholesale for a re-rendered instance

What actually happens instead is covered next (render phase internals).

### The Virtual DOM (React Element Tree)

- **Virtual DOM** = the tree of **all React elements** created from every instance in the component tree — just a plain JS object, cheap/fast to (re)create
- 👉 The React team downplays the term now (not in official docs anymore) — it's just a React element tree, nothing magical. Also **unrelated** to the browser's "Shadow DOM"
- 🚨 **Key rule:** rendering a component **re-renders all of its child components too** — regardless of whether their props changed
  - Why: React doesn't know in advance whether a parent update will affect its children, so it plays it safe by default
  - So updating state high in the tree (e.g. the root) re-renders the *entire app* — but again, this only recreates the **virtual DOM**, not the real DOM, so it's cheap for small/medium apps
- 👉 The new virtual DOM then gets **reconciled** against the **current Fiber tree** (the tree from before the update) — this is done by React's reconciler, which is literally named **Fiber** (hence "Fiber tree"). Output = an **updated Fiber tree**

### Why Reconciliation Exists

- Why not just rewrite the whole DOM on every state change? Because that'd be wasteful:
  1. Writing to the DOM is (relatively) **slow**
  2. Usually only a **small part** of the DOM actually needs to change (e.g. `showModal = true` only needs the modal's markup inserted — the rest of the page stays put)
- **Reconciliation** = deciding exactly which DOM elements need to be **inserted, deleted, or updated** to match the latest state → produces a list of DOM ops
- The **reconciler** (Fiber) is the "engine"/heart of React — it's what lets us just describe *what* the UI should look like (via state) instead of manually touching the DOM ourselves

### The Fiber Tree

- On initial render, Fiber builds a **Fiber tree** from the virtual DOM — one **Fiber** per component instance **and** per DOM element (both trees cover the full DOM structure, not just React components)
- 🔑 Unlike React elements, **Fibers are never recreated** — the Fiber tree persists and is **mutated in place** on every reconciliation. That's why it's the natural home for a component instance's **current state, props, side effects, hooks list, and queue of pending work**
- A Fiber = a **"unit of work"**
- Structurally, Fibers form a **linked list**, not a plain parent/child tree: first child links to parent, other children link to their previous sibling — this makes the work easier for React to process incrementally
- Because work is broken into Fibers, rendering can happen **asynchronously**: split into chunks, prioritized, paused/resumed, or discarded — invisible to us, but it's what powers concurrent features (**Suspense**, transitions, React 18+) and keeps long renders from blocking the JS engine. Only possible because the render phase produces no visible DOM output yet

### Reconciliation Worked Example (`showModal: true → false`)

```
CURRENT FIBER TREE          UPDATED FIBER TREE (workInProgress)
      App                          App
   ┌───┼────┐                   ┌───┼────┐
 Video Modal Btn      →       Video Modal Btn   ← text updated
        │                       (unchanged)  ✗ deleted (with children)
     Overlay
     ┌──┴──┐
    h3   button
```

- New virtual DOM is diffed against the current Fiber tree → produces the **"work in progress" tree** (React's internal name for the updated Fiber tree)
- **Diffing** = comparing elements **by their position in the tree**, current vs. updated
- Per-Fiber outcomes seen in this example:
  - `Btn` text changed → marked **DOM update**
  - `Modal`/`Overlay`/`h3`/`button` no longer in the new tree → marked **DOM deletion**
  - `Video` re-rendered (child of `App`) but **unchanged** → no DOM mutation at all, even though its component function ran again
- All the flagged mutations get collected into a **list of effects**, which the **commit phase** then applies to the real DOM
- 👉 Jonas notes even this is still a simplified version of what Fiber actually does

### Commit Phase, Precisely

- Commit walks the **list of effects** and applies each DOM insert/delete/update — "flushing" updates to the DOM
- **Synchronous, uninterruptible** — unlike the render phase, it can't pause. Necessary so the DOM never shows a half-updated (inconsistent) UI
- After commit, the `workInProgress` Fiber tree **becomes the `current` tree** for the next cycle (reused, never rebuilt — same tree, just mutated again next time)
- 🔑 **Library split:** the **render phase** is done by **React**; the **commit phase** (actually writing the DOM) is done by a separate library, **ReactDOM**; the final repaint is the **browser's** job, unrelated to React

### Why React and ReactDOM Are Separate: "Hosts" and "Renderers"

- React itself **never touches the DOM** and doesn't even know where its render output will end up — it's platform-agnostic by design
- The DOM is just **one possible "host"**. Others: **React Native** (iOS/Android), **Remotion** (video), Word/PDF/Figma via other renderer packages
- Each host has its own **renderer** package (ReactDOM, React Native, etc.) that takes the render phase's output and **commits** it to that host
- 👉 "Renderer" is a misleading name — renderers don't render, they **commit** (name predates React splitting render/commit into two phases)
- This is also *why* `index.js` imports both **React** (render phase) and **ReactDOM** (commit phase) separately

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
