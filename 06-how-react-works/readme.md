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

> ⚠️ **Correcting the earlier mental model:** the old `STATE → RENDER → UPDATED VIEW` diagram implied rendering touches the screen directly and that re-rendering wholesale replaces the old DOM. Neither is literally true — rendering just calls component functions (no DOM touched), and the DOM is never thrown away wholesale. What actually happens is covered next.

### The Virtual DOM (React Element Tree)

- **Virtual DOM** = the tree of **all React elements** created from every instance in the component tree — just a plain JS object, cheap/fast to (re)create
- 👉 The React team downplays the term now (not in official docs anymore) — it's just a React element tree, nothing magical. Also **unrelated** to the browser's "Shadow DOM"
- 🚨 **Key rule:** rendering a component **re-renders all of its child components too** — regardless of whether their props changed
  - Why: React doesn't know in advance whether a parent update will affect its children, so it plays it safe by default
  - So updating state high in the tree (e.g. the root) re-renders the *entire app* — but again, this only recreates the **virtual DOM**, not the real DOM, so it's cheap for small/medium apps
- 👉 The new virtual DOM then gets **reconciled** against the **current Fiber tree** (the tree from before the update) — this is done by React's reconciler, which is literally named **Fiber** (hence "Fiber tree"). Output = an **updated Fiber tree**

### Why Reconciliation Exists

- Rewriting the whole DOM on every state change would be wasteful: DOM writes are (relatively) **slow**, and usually only a **small part** needs to change (e.g. `showModal = true` only needs the modal inserted — the rest of the page stays put)
- **Reconciliation** = deciding exactly which DOM elements need to be **inserted, deleted, or updated**
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

- New virtual DOM is diffed against the current Fiber tree → produces the **"work in progress" tree** (React's internal name for the updated Fiber tree; how diffing itself decides these outcomes is detailed below)
- Per-Fiber outcomes seen in this example:
  - `Btn` text changed → marked **DOM update**
  - `Modal`/`Overlay`/`h3`/`button` no longer in the new tree → marked **DOM deletion**
  - `Video` re-rendered (child of `App`) but **unchanged** → no DOM mutation at all, even though its component function ran again
- All the flagged mutations get collected into a **list of effects**, which the **commit phase** then applies to the real DOM
- 👉 Jonas notes even this is still a simplified version of what Fiber actually does

### How Diffing Works

> 👉 Left out of the render-phase lecture, but essential: **diffing** is the specific algorithm reconciliation uses to compare renders.

- **2 fundamental assumptions** diffing relies on:
  1. **Two elements of different types will produce different trees**
  2. **Elements with a stable `key`** (consistent across renders) **stay the same** across renders
- 🔑 These assumptions look obvious, but they're what let diffing be **fast**: without them, comparing trees would cost ~**O(n³)** (~1 billion ops for 1000 elements); with them, it's **O(n)** (~1000 ops for 1000 elements)
- Diffing compares elements **by position in the tree**, current vs. updated render. Only **2 situations** matter:
  1. **Different element at the same position**
  2. **Same element at the same position** (next lecture)

#### Situation 1 — Different Element, Same Position

```
<div>                      <header>
  <SearchBar />      →       <SearchBar />
</div>                     </header>
```

- "Different" = the **type** changed (`div` → `header`, or one component → a different component, e.g. `SearchBar` → `ProfileMenu`) — applies the same way to DOM elements and React elements (component instances)
- 🚨 React assumes the element **and its entire sub-tree** are no longer valid:
  - The old element and all its children are **destroyed and removed from the DOM**
  - This **includes their state** — even if a child element looks unchanged, if its parent's type changed, the whole branch is torn down and rebuilt from scratch as **brand-new instances**
- 👉 **State is not preserved** across a type change at the same tree position — this is what "resets state" in practice, with real implications for how apps behave (examples next lecture)

#### Situation 2 — Same Element, Same Position

```
<div className="hidden">                <div className="active">
  <SearchBar wait={1} />        →         <SearchBar wait={5} />
</div>                                   </div>
```

- More straightforward: if the element at a position is the **same type** as before, React just **keeps it in the DOM** — including all child elements and, crucially, **component state**
- Works identically for **DOM elements** and **React elements** (components)
- If something about it *did* change, it's not the type — just an **attribute** (e.g. `className`) or a **prop** (e.g. `wait`). React handles this efficiently:
  - DOM element → **mutates** the changed attribute(s) in place
  - React element (component) → just **passes in the new props**
- 👉 Nothing is torn down or recreated — the underlying DOM node and the component's state **persist** across the render
- 🔑 Sometimes this default (state persisting) is **not** what we want — that's what the **`key` prop** is for: forcing React to treat an element as a brand-new instance (destroy + recreate) even when type and position stay the same (next lecture)

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

## The `key` Prop

- 👉 A **special prop** used to tell the diffing algorithm that an element is **unique** — works for both **DOM elements** and **React elements** (components)
- In practice, a `key` gives each **component instance** a unique identity, letting React **distinguish between multiple instances** of the same component type
- Ties directly back to diffing's **2nd assumption**: elements with a **stable `key`** (same across renders) are treated as the **same** element

### Two behaviors, two use cases

1. **Key stays the same across renders** → the element is **kept in the DOM**, even if its **position in the tree changes**
   - 🔑 **Use case 1: keys in lists** — this is *why* we've been adding `key` to list items all course
2. **Key changes between renders** → the element is **destroyed and a new one created** in its place, even if its **position in the tree stays exactly the same**
   - 🔑 **Use case 2: using keys to reset state**

> 👉 So the `key` prop lets us **override** diffing's default position-based behavior in both directions: force React to **preserve** an instance across a position change (lists), or force it to **tear down and recreate** an instance even though nothing about its position or type changed (state reset).

### Use Case 1 — Keys in Lists, Worked Example

**Without keys:**

```jsx
<ul>
  <Question question={q[1]} />
  <Question question={q[2]} />
</ul>
```

- Adding a new item to the **top** of the list:

```jsx
<ul>
  <Question question={q[0]} />   {/* new */}
  <Question question={q[1]} />   {/* was 1st child, now 2nd */}
  <Question question={q[2]} />   {/* was 2nd child, now 3rd */}
</ul>
```

- The `q[1]` and `q[2]` elements are clearly still **the same questions as before**, but they now sit at **different positions** in the tree (2nd/3rd instead of 1st/2nd)
- Diffing only compares **by position**, so it has no way to know these are "the same" element that just moved — per the diffing rules, it **removes and recreates** both DOM elements at their new positions
- 🚨 This is **wasted work**: destroying and rebuilding an unchanged DOM element hurts performance, but React has **no way of knowing** it's unnecessary — developers can *see* it intuitively, React can't

**With keys:**

```jsx
<ul>
  <Question key="q1" question={q[1]} />
  <Question key="q2" question={q[2]} />
</ul>
```

- Adding the new item to the top now looks like this:

```jsx
<ul>
  <Question key="q0" question={q[0]} />   {/* new */}
  <Question key="q1" question={q[1]} />   {/* different position, same key */}
  <Question key="q2" question={q[2]} />   {/* different position, same key */}
</ul>
```

- `q1` and `q2` are still at **different positions** in the tree, but their **`key` stays stable** across renders
- 👉 Per the diffing rules, elements with a stable key are **kept in the DOM** rather than destroyed/recreated, even though their position changed
- Result: a **more performant UI** — the unaffected elements are left alone, and only the truly new element is created
- 👉 The difference is invisible on tiny lists, but becomes **significant on large lists** (thousands of elements), which does happen in real apps

> 🔑 **Rule of thumb:** always use the `key` prop on multiple **child elements of the same type** (e.g. items rendered via `.map()`) — not just to silence React's warning, but because it's the only way to give React the information it needs to skip unnecessary DOM work.

### Use Case 2 — Key Prop to Reset State, Worked Example

> 👉 No big code example needed here — this gets built for real in the **next lecture**. This is just the concept.

```jsx
<QuestionBox>
  <Question
    question={{ title: "React vs JS", body: "Why should we use React?" }}
    key="q23"
  />
</QuestionBox>
```

- `Question` owns its own **`answer` state** — say the user has typed *"React allows us to build apps faster"*
- Now the **question prop changes** to a new question, but the element stays at the **same position** in the tree:

```jsx
<QuestionBox>
  <Question
    question={{ title: "Best course ever :D", body: "This is THE React course!" }}
    key="q23"   /* same key */
  />
</QuestionBox>
```

- 🚨 With the **same key** (or no key at all), this is "same element, same position" → per the diffing rules, the DOM element **and its state are kept** — the `answer` state (and whatever the user typed) **stays around**
- 👉 But that old answer is now **irrelevant** to the new question — keeping it doesn't make logical sense for the app
- **Fix:** give the new question a **different key**:

```jsx
<QuestionBox>
  <Question
    question={{ title: "Best course ever :D", body: "This is THE React course!" }}
    key="q89"   /* different key */
  />
</QuestionBox>
```

- 👉 A changed key tells React this is a **different component instance**, even though type and position are identical — React **destroys** the old instance and **creates a brand-new one**
- Result: the `answer` state is **reset** (back to empty) — exactly the behavior needed so the leftover answer doesn't linger on a new, unrelated question

> 🔑 **Takeaway:** whenever you need to **reset state** tied to a component instance, give that element a `key` that **changes** across renders. Doesn't come up constantly, but when it does, this is *the* solution — worth recognizing on sight.

## Rules for Render Logic

> To make the rendering process work the way described above, render logic must follow a few simple rules.

### The Two Types of Logic in React Components

**1. Render logic**

- 👉 Code that lives at the **top level** of the component function
- 👉 **Participates in describing** how the component view looks like (including helper functions called from JSX, like a `createList()` invoked inside the `return`)
- 👉 **Executed every time** the component renders — i.e. every time the function is called

**2. Event handler functions**

- 👉 **Executed as a consequence of the event** the handler is listening for (e.g. a `change` event)
- 👉 Code that actually **does things**: update state, perform an HTTP request, read an input field, navigate to another page, etc.

```jsx
function Question({ question }) {
  const [newAnswer, setNewAnswer] = useState('');     // render logic
  const numAnswers = question.answers.length ?? 0;    // render logic

  const handleNewAnswer = function (e) {               // event handler
    if (question.closed) return;
    setNewAnswer(e.target.value);
  };

  const createList = function () {                     // render logic (called from JSX)
    return (
      <ul>
        {question.answers.map((q) => (
          <li>{q}</li>
        ))}
      </ul>
    );
  };

  return (
    <div>
      <h3>{question.title}</h3>
      <p>{question.body}</p>
      {question.hasAnswer ? (
        createList()
      ) : (
        <input value={newAnswer} onChange={handleNewAnswer} />
      )}
    </div>
  );
}
```

> 🔑 **Why the distinction matters:** render logic **describes** the view (must stay pure — see rules next lecture), while event handlers **make things happen** in the app. Mixing the two — e.g. mutating state directly inside render logic instead of inside a handler — breaks React's assumptions about components being pure functions of state/props.

### Refresher: Functional Programming Principles

- **Side effect:** a function **depends on**, or **modifies**, data **outside its own scope** — i.e. the function's **"interaction with the outside world"**
  - Examples: mutating an external variable/object, HTTP requests, writing to the DOM, setting timers

```js
// ✅ Pure function
function circleArea(r) {
  return 3.14 * r * r;
}

// ✋ Impure — side effect: mutates an outside variable
const areas = {};
function circleArea(r) {
  areas.circle = 3.14 * r * r;
}

// ✋ Impure — unpredictable output: `date` changes every call
function circleArea(r) {
  const date = Date.now();
  const area = 3.14 * r * r;
  return `${date}: ${area}`;
}
```

- **Pure function:** a function with **no side effects**
  - Does **not** change any variables outside its own scope
  - Given the **same input**, it **always returns the same output** → predictable
- An **impure** function is the opposite: output can differ for the same input, and/or it mutates something outside itself

> 👋 **Side effects are not bad!** A program is only useful if it interacts with the outside world at some point (an app that never touches data or the DOM does nothing). The goal isn't "no side effects ever" — it's keeping side effects **out of render logic** (next lecture covers exactly where they *do* belong: event handlers, and later, the `useEffect` hook).

### Rules for Render Logic

> ☝️ **The one big rule: components must be pure functions when it comes to render logic.**

- Given the **same props** (input), a component instance should always return the **same JSX** (output)
- In practice: **render logic must produce no side effects** — no interaction with the "outside world" is allowed at the top level of a component function. So, in render logic:
  - 👉 Do **NOT** perform **network requests** (API calls)
  - 👉 Do **NOT** start **timers**
  - 👉 Do **NOT** directly use the **DOM API** (e.g. `addEventListener`)
  - 👉 Do **NOT** mutate objects or variables **outside the function's scope** — 🔑 **this is exactly why we can't mutate props**: doing so would be a side effect
  - 👉 Do **NOT** update **state or refs** — updating state in render logic would create an **infinite loop** (state updates aren't technically side effects, but they're forbidden here for this separate reason)
- Some side effects are technically "not allowed" by this rule but are harmless and used constantly anyway: `console.log`, generating random numbers — safe to keep doing

### Where Side Effects *Do* Belong

- 👋 **Event handler functions** are not render logic → side effects are **allowed and encouraged** there
- For a side effect that needs to run **as soon as the component first renders**, register it with the special **`useEffect`** hook (covered next section)

> 🔑 **Takeaway:** "no side effects" only applies **inside render logic**. Anything reactive to user interaction goes in an event handler; anything that needs to run on render/mount goes in `useEffect` — never directly in the component's top-level code.
