# React Router: Building Single-Page Applications (SPA)

> Section: **React Router** — building the **WorldWise** app as a real single-page application

## What Is Routing? SPAs and React Router

### Routing

- 👉 With routing, we match **different URLs** to **different UI views** (React components): each match is a **route**
- Visiting a URL **renders the corresponding component**

| URL | Component / view |
|---|---|
| `example.com/` | Home (landing page) |
| `example.com/login` | Login |
| `example.com/app` | The app itself (after a successful login, redirect here) |

- Users **navigate between application screens** using links and the **browser URL**
- Keeps the **UI in sync with the current browser URL** (benefits covered later)
- Allows us to build **Single-Page Applications**
- This is **client-side routing**: it happens in the **user's browser**. Server-side routing also exists, but not in client-side React apps like ours

### React Router

- Most front-end frameworks have client-side routing **built in**; **React doesn't**: it relies on **third-party packages** for many things, routing included
- Routing in React is usually handled by **React Router**

> 🔑 React Router is probably the **most important and most used third-party React library**. To learn React development, you need to learn it.

### Single-Page Applications (SPA)

- Application **executed entirely on the client** (browser)
- **Routes**: different URLs correspond to different views (components)
- **JavaScript (React) updates the DOM**, not the browser loading a new page
- 👉 **The page is never reloaded**: the entire app lives in one page, with no hard reloads
- **Feels like a native app** (desktop/mobile) → great UX
- **Additional data might be loaded from a web API**: the SPA runs on the client but can still talk to a server

### The SPA cycle

1. **User clicks a router link**
2. **URL is changed** (in React, the **`react-router`** package does this job)
3. **DOM is updated**: the React component corresponding to the new URL is rendered (may **load data from a web API** along the way)
4. Repeat from step 1 as often as needed, **without reloading the page**

### Normal page vs. SPA

| | Traditional page | SPA |
|---|---|---|
| **Clicking a link** | Browser loads a **completely new page** | **JavaScript updates the DOM** |
| **Page reload** | Yes | **Never** |
| **Fetching data** | n/a | Allowed via **web API** |
| **Loading a whole new page** | n/a | ❌ Not allowed (it would no longer be an SPA) |

- ☝️ Technically **every React app so far was an SPA** (React updates the DOM, never reloads)
- But **professional, complex apps need URLs and routing** → only then do they become **real single-page applications**

## Styling Options in React

- React is **unopinionated**: it **doesn't care how you style** your app → lots of options, most from **third-party libraries**

| Styling option | Where? | How? | Scope | Based on |
|---|---|---|---|---|
| **Inline CSS** | JSX elements | `style` prop | **JSX element** (local) | CSS |
| **CSS / Sass file** | External file | `className` prop | **Entire app** ❌ (global, causes problems) | CSS |
| **CSS Modules** | **One external file per component** | `className` prop | **Component** ✅ | CSS |
| **CSS-in-JS** | External file or component file | **Creates a new component** | **Component** ✅ | JavaScript |
| **Utility-first CSS** (Tailwind) | JSX elements | `className` prop | **JSX element** | CSS |

> 👋 Alternative to writing CSS: **UI component libraries** like **MUI, Chakra UI, Mantine**

### Inline CSS

- `style` prop on JSX elements; **scoped to that exact element**
- More common in React than plain HTML, because of React's **separation of concerns** idea (a component owns its own look)

### Global CSS / Sass file

- Used so far: one external file, then `className` in JSX. Sass works the same way
- 🚨 **Global**: any element in the whole app can use any class
  - Unclear which components use which classes → editing a class has **repercussions elsewhere**
  - Adding a class with an **existing name** → **clashes**
- 👉 **Global CSS is a nightmare in large apps**; in professional projects CSS is almost never global. It should be **scoped to a component**

### CSS Modules ✅ (used in this project)

- Like regular CSS, but **one CSS file per component**
- Styles are **scoped to that component only**: no other component can use them
- Components become **more modular and reusable**, and it better reflects React's **separation of concerns**

### CSS-in-JS

- **Write CSS inside JavaScript**, in the same file as the component (e.g. **styled-components**)
- Creates **React components with the styles already applied**, used like any other component
- Fully embraces the idea that a **component contains all info about its appearance**, CSS included

### Utility-first CSS (Tailwind)

- **Predefined utility classes** for individual styles: flexbox, responsive layouts, hover effects, the whole UI, **without leaving the JSX markup**
- Getting more popular every day

### UI component libraries

- **Prebuilt, pre-styled components** common to most web apps (MUI, Chakra UI, Mantine): build a whole project **without writing CSS**
- ☝️ Not ideal for beginners; worth exploring later

## Storing State in the URL

- 👉 The **URL is an excellent place to store UI state**, and an **alternative to `useState`** in some situations
- **UI state** = state that affects **what the UI looks like**
  - Examples: open/closed panels, currently selected list item, list sorting order, applied list filters
- With React Router, that state is **managed by the URL**

### Why?

1. Easy way to store state in a **global place**, accessible to **all components** in the app
   - No more lifting state to a parent and passing it down with **props**: read it from the URL **anywhere in the tree**
2. Good way to **"pass" data from one page into the next**, without storing it somewhere temporary in the app
3. Makes it possible to **bookmark and share** the page with the **exact UI state** it had (e.g. an online shop filtered by color and price)

### Anatomy of a URL

```
www.example.com/app/cities/lisbon?lat=38.728&lng=-9.141
               └───path──┘ └params┘└────query string───┘
```

| Part | Example | Used for |
|---|---|---|
| **Path** | `/app/cities` | Which component is displayed. Technically state, but not useful for state management in this sense |
| **Params** | `lisbon` | **Passing data to the next page** |
| **Query string** | `lat=38.728&lng=-9.141` | **Global state** that should be accessible everywhere |

- 👉 **Params** and the **query string** are the **React Router tools** for storing state in the URL

### Example: params and query string in WorldWise

```
/app/cities/lisbon?lat=38.728&lng=-9.141
/app/cities/berlin?lat=52.536&lng=13.377
```

- Clicking a city in the list links to a URL with its **param** (`lisbon`) → the next page loads **that city's** details
- The **query string** (`lat`, `lng`) holds the **map position**, unique to each city
- 👉 **City name and GPS location are read from the URL** instead of from application state
