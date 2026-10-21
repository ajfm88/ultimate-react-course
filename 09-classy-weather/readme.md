# React Before Hooks: Class-Based React

> Section: **[Optional] React Before Hooks: Class-Based React** (~1 hr 22 min) — building the **Classy Weather** app with class components

## Class Components vs. Function Components

| | Function components | Class components |
|---|---|---|
| **Introduced in** | **v16.8** (2019, with hooks) | **v0.13** (2015) |
| **How to create** | JavaScript **function** (any type: declaration or arrow) | **ES6 class** extending `React.Component` |
| **Reading props** | Destructuring or `props.X` | `this.props.X` |
| **Local state** | `useState` hook | `this.setState()` |
| **Side effects / lifecycle** | `useEffect` hook | **Lifecycle methods** |
| **Event handlers** | **Functions** defined in the component body | **Class methods** (one per handler) |
| **Returning JSX** | Return JSX **from the function** | Return JSX from the **`render` method** |

- 👉 Function components have **existed since the beginning**, but **without hooks** (before 16.8) they were very limited: no state of their own
- Class components rely on **OOP principles**: the **`this` keyword** is needed to read props and define local state, which gets annoying over time

> 🔑 **Hooks are THE big difference.** They introduced a completely new way of thinking about and writing React apps.

### Side Effects and Lifecycle

- **Class components:** special **lifecycle methods** defined by React run code at different points of the lifecycle (e.g. `componentDidMount`, `componentWillUnmount`)
- **Function components:** focus on **synchronization** rather than lifecycle, via `useEffect` (which still maps onto the lifecycle)

### Advantages

**Function components (with hooks)** 👍

- **Easier to build**: much less React-specific **boilerplate**
- **Cleaner code**: `useEffect` **combines all lifecycle-related code in a single place**, while class components **spread it across multiple lifecycle methods** (confusing in large components)
- **Easier to share stateful logic** (custom hooks)
- **No `this` keyword** anymore (annoying, error-prone, hard for beginners)

**Class components** 👍

- Lifecycle **might be easier to understand for beginners**, thanks to explicitly named methods like `componentDidMount` and `componentWillUnmount`
