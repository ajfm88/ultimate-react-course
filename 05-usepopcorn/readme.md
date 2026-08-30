# Thinking in React: Components, Composition, and Reusability

## How to Split a UI Into Components

### Component Size Matters

- Example: an Airbnb listing card — could be built as **many small components** (image, superhost badge, title, amenities, rare-find tag, rating, price) or as **one huge component**
- Generally, we need to find the **right balance** between too specific and too broad

**Too small (many mini-components)**

- 👉 We end up with **100s of mini-components**
- 👉 **Confusing codebase**
- 👉 **Too abstracted** — creating something new just to hide the implementation details of that thing

**Too huge**

- 👉 Too many **responsibilities**
- 👉 Might need too many **props**
- 👉 Hard to **reuse**
- 👉 Complex code, hard to understand

> 🔑 There's no fixed rule for the "right" size — aim for the balance in the middle, where a component has one clear responsibility without being needlessly fragmented.

### The 4 Criteria for Splitting a UI Into Components

1. **Logical separation of content/layout**
2. **Reusability**
3. **Responsibilities / complexity**
4. **Personal coding style**

**Example — the Airbnb card, split correctly**

- ✅ **Logical separation** — image, details (title, specs, rare-find), and price/rating are each their own concern
- ✅ **Some are reusable** — e.g. a `Rating` or `Price` component could be reused elsewhere in the app
- ✅ **Low complexity** — each piece stays simple and focused

> Splitting **too little** (whole card as one block, 👎) or **too much** (every span wrapped in its own component, 👎) both fail these criteria — the good split groups elements by what they logically represent (image / info / price), not just by what's easy to wrap.

### Framework: When to Create a New Component?

> 💡 **Suggestion:** When in doubt, start with a relatively **big** component, then **split it into smaller components** as it becomes necessary.

Run the component against the 4 criteria:

1. **Logical separation of content/layout**
   - 👉 Does the component contain pieces of content or layout that **don't belong together**?
2. **Reusability**
   - 👉 Is it possible to **reuse** part of the component?
   - 👉 Do you **want** or **need** to reuse it?
3. **Responsibilities / complexity**
   - 👉 Is the component doing too **many different things**?
   - 👉 Does the component rely on too **many props**?
   - 👉 Does the component have too **many pieces of state** and/or effects?
   - 👉 Is the code, including JSX, too **complex/confusing**?
4. **Personal coding style**
   - 👉 Do you prefer **smaller** functions/components?

- If the answer to any of these is "yes" → **you might need a new component**
- ⚠️ Skip criterion 2 if you're sure you need to reuse. But otherwise, **you don't need to focus on reusability and complexity early on**

> 👋 These are all **guidelines**, not strict rules... it will become intuitive over time!

### Some More General Guidelines

- 💰 Creating a new component **creates a new abstraction**. Abstractions have a **cost**, because **more abstractions require more mental energy** to switch back and forth between components. So try **not to create new components too early**
- 🏷️ Name a component according to **what it does** or **what it displays**. Don't be afraid of using **long component names**
- 🪆 **Never declare a new component inside another component!**
- 🗄️ **Co-locate related components inside the same file.** Don't separate components into different files too early
- ↔️ It's completely normal that an app has components of **many different sizes**, including very small and huge ones

### Any App Has Components of Different Sizes and Reusability

```
SMALL             Components with all different sizes             HUGE
REUSABLE          (different degrees of size, reusability,     NON-REUSABLE
                    responsibility, and complexity)
```

- Example: the Airbnb card breaks down into several **small** components (superhost badge, image carousel, title, amenities, rare-find tag, rating, price) — while the whole **page** (search results + map) is one **huge** component

**Small end**

- 👉 Some **very small components are necessary**!
- 👉 **Highly reusable**
- 👉 **Very low complexity**

**Huge end**

- 👉 **Most apps will have a few huge components** (e.g. a page component)
- 👉 **Not meant to be reused** (**not a problem!**)

> 🔑 A healthy app has components spread across the whole spectrum — small, reusable, low-complexity pieces **and** a handful of huge, non-reusable ones (like page components). Neither extreme is wrong on its own.

## Component Categories

👉 Most of your components will **naturally** fall into **one of three categories**:

**1. Stateless / presentational components**

- 👉 **No state**
- 👉 Can receive **props** and simply **present** received data or other content
- 👉 Usually **small and reusable**
- Example: `usePopcorn` logo, `Found 3 results`, a single `Movie` item (`Inception 🗓 2010`)

**2. Stateful components**

- 👉 **Have state**
- 👉 Can still be **reusable**
- Example: the `Search` input (`Search movies...`), the movie list that toggles open/closed

**3. Structural components**

- 👉 **"Pages"**, **"layouts"**, or **"screens"** of the app
- 👉 Result of **composition**
- 👉 Can be **huge and non-reusable** (but don't have to)
- Example: the whole `usePopcorn` app layout (navbar + movie list + watched list)

## What Is Component Composition?

**"Using" a component (❌ not composition)**

```jsx
function Modal() {
  return (
    <div className="modal">
      <Success />
    </div>
  );
}

function Success() {
  return <p>Well done!</p>;
}
```

- 👉 `Success` is **hardcoded inside** `Modal`
- ❌ We can **NOT reuse** `Modal` with different content (e.g. an `Error` message)

**Component composition (✅)**

```jsx
function Modal({ children }) {
  return (
    <div className="modal">
      {children}
    </div>
  );
}

function Error() {
  return <p>This went wrong!</p>;
}
```

```jsx
<Modal>
  <Success />
</Modal>

<Modal>
  <Error />
</Modal>
```

- 👉 `Success` (or `Error`) is **passed into** `Modal` as `children`, instead of being hardcoded
- ✅ Now we **CAN reuse** `Modal` with whatever content we pass in

> 🔑 **Component composition** = combining different components using the `children` prop (or explicitly defined props). It lets us create **highly reusable and flexible components**, and is essential for building layouts (e.g. wrapping a page, sidebar, or modal around arbitrary content).

**With component composition, we can:**

1. **Create highly reusable and flexible components**
2. **Fix prop drilling** (great for layouts)

> 💡 This is possible because **components don't need to know their children in advance** — whatever JSX gets passed in as `children` (or another prop) is simply rendered where that placeholder sits.
