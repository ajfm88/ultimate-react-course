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
