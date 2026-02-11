# React.js – End‑to‑End Interview Notes (Highly Descriptive)

## Table of Contents

1. [Prerequisites & Mental Model](#1-prerequisites--mental-model)  
2. [React Overview & Design Philosophy](#2-react-overview--design-philosophy)  
3. [JSX, Elements, and Rendering](#3-jsx-elements-and-rendering)  
4. [Components, Props, and Composition](#4-components-props-and-composition)  
5. [State Basics with `useState`](#5-state-basics-with-usestate)  
6. [Side Effects with `useEffect`](#6-side-effects-with-useeffect)  
7. [Lifecycle Mapping: Classes vs Hooks](#7-lifecycle-mapping-classes-vs-hooks)  
8. [Lists, Keys, and Conditional Rendering](#8-lists-keys-and-conditional-rendering)  
9. [Forms, Controlled Components, and Validation](#9-forms-controlled-components-and-validation)  
10. [Context API & Avoiding Prop Drilling](#10-context-api--avoiding-prop-drilling)  
11. [Advanced Hooks: `useReducer`, `useMemo`, `useCallback`, `useRef`](#11-advanced-hooks-usereducer-usememo-usecallback-useref)  
12. [Custom Hooks & Reusable Business Logic](#12-custom-hooks--reusable-business-logic)  
13. [Routing and SPA Architecture (React Router)](#13-routing-and-spa-architecture-react-router)  
14. [State Management Strategies (Local, Context, Redux, Server State)](#14-state-management-strategies-local-context-redux-server-state)  
15. [Rendering Behavior, Reconciliation & Performance Tuning](#15-rendering-behavior-reconciliation--performance-tuning)  
16. [Error Handling & Error Boundaries](#16-error-handling--error-boundaries)  
17. [React 18+ Concurrent Features & Suspense](#17-react-18-concurrent-features--suspense)  
18. [React Server Components – Conceptual Overview](#18-react-server-components--conceptual-overview)  
19. [Testing React Components](#19-testing-react-components)  
20. [Project Structure, Tooling & Environment](#20-project-structure-tooling--environment)  
21. [Common Interview Questions & How to Answer](#21-common-interview-questions--how-to-answer)  
22. [End‑to‑End Example: Todo App with Filters and Persistence](#22-end-to-end-example-todo-app-with-filters-and-persistence)  

***

## 1. Prerequisites & Mental Model

Before React, ensure comfort with:

- **Modern JavaScript**: ES6+ (arrow functions, destructuring, modules, spread/rest, classes, promises, async/await).
- **DOM & Browser**: basic understanding of how DOM updates work, event propagation, etc.
- **Functional programming basics**: pure functions, immutability, higher‑order functions (`map`, `filter`, `reduce`).

**React mental model:**

- Think in terms of **state → UI**:
  - UI = function of state (`UI = f(state)`).
- Components are **pure** with respect to their props and state:
  - Same props + same state → same output (ignoring effects).

***

## 2. React Overview & Design Philosophy

**Definition**:  
React is a **JavaScript library for building user interfaces** for web and native platforms. [react](https://react.dev)

**Core design principles:** [react](https://react.dev/learn)

1. **Declarative**  
   - Instead of manually manipulating the DOM, you declare *what* the UI should look like.
   - React handles minimal DOM updates via its reconciliation algorithm.

2. **Component‑based**  
   - UI is composed from small, reusable components (buttons, cards, forms, pages).
   - Each component manages its own state (if needed) and receives data via props. [dev](https://dev.to/brilworks/react-components-explained-a-2025-guide-for-developers-4dhe)

3. **Learn once, write anywhere**  
   - Same component model powers web (`react-dom`), native (`react-native`), desktop, etc.

4. **Composition over inheritance**  
   - React encourages composing behavior by nesting components instead of using class inheritance. [dev](https://dev.to/brilworks/react-components-explained-a-2025-guide-for-developers-4dhe)

**Why companies use React:**

- Huge ecosystem and community.
- Strong backwards compatibility; apps written years ago still run with few changes.
- Clear data‑flow model (one‑way data flow) simplifies reasoning about UI bugs.

***

## 3. JSX, Elements, and Rendering

### 3.1 JSX basics

JSX = JavaScript XML. It lets you write HTML‑like syntax that compiles to `React.createElement` calls. [react](https://react.dev/learn)

```jsx
const element = <h1 className="title">Hello, React!</h1>;
```

Compiles roughly to:

```js
const element = React.createElement("h1", { className: "title" }, "Hello, React!");
```

**Key JSX rules:**

- Must have **one root element** per component:

  ```jsx
  // ❌ Invalid
  return <h1 /> <h2 />;

  // ✅ Valid
  return (
    <>
      <h1 />
      <h2 />
    </>
  );
  ```

- Use `className`, `htmlFor`, camelCase event handlers (`onClick`, `onChange`).
- JavaScript expressions go inside `{}`:
  ```jsx
  const user = { name: "Rahul" };
  <p>Hello, {user.name.toUpperCase()}</p>;
  ```

### 3.2 Rendering into the DOM (React 18)

Entry point (e.g., `main.jsx`):

```jsx
import React from "react";
import { createRoot } from "react-dom/client";
import App from "./App";

const rootElement = document.getElementById("root");
const root = createRoot(rootElement);

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

- `createRoot` enables concurrent features in React 18+. [buttercms](https://buttercms.com/blog/react-concurrent-mode-the-future-of-web-apps/)
- `StrictMode` helps catch potential issues (double‑invokes some lifecycle logic in dev).

**Interview tip:** Be able to explain **what JSX is** and how it compiles, and that React does not “use HTML templates” but JS functions returning elements.

***

## 4. Components, Props, and Composition

### 4.1 Function components (modern standard)

```jsx
function Greeting({ name }) {
  return <h2>Hello, {name}!</h2>;
}

export default Greeting;
```

Usage:

```jsx
<Greeting name="Sagar" />
```

Characteristics:

- Input: **props** object.
- Output: JSX describing UI.
- Must be **pure**: no side effects during render (effects go in `useEffect`).

### 4.2 Props

Props are **read‑only configuration** supplied by parent:

```jsx
function Button({ label, onClick, type = "button" }) {
  return (
    <button type={type} onClick={onClick}>
      {label}
    </button>
  );
}
```

- Props follow **one‑way data flow**: parent → child.
- Child must not mutate props; if data must change, parent holds state and passes down new props.

### 4.3 Component composition patterns

**Simple composition:**

```jsx
function Avatar({ user }) {
  return <img src={user.avatarUrl} alt={user.name} />;
}

function UserCard({ user }) {
  return (
    <div className="card">
      <Avatar user={user} />
      <h3>{user.name}</h3>
    </div>
  );
}
```

**Using `children` for flexible layouts:**

```jsx
function Card({ title, children }) {
  return (
    <section className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </section>
  );
}

// usage
<Card title="Profile">
  <UserCard user={user} />
</Card>
```

**Interview angle:**  
Explain why React prefers composition over inheritance and give examples (cards with customizable headers/footers, layout components, etc.). [legacy.reactjs](https://legacy.reactjs.org/docs/design-principles.html)

***

## 5. State Basics with `useState`

### 5.1 `useState` fundamentals

`useState` adds **local, reactive state** to a function component. [daily](https://daily.dev/blog/react-hooks-explained-simply)

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0); // state variable + setter

  function increment() {
    setCount(prev => prev + 1);
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

Key points:

- Initial value (`0`) used only on the **first render**.
- Calling `setCount` schedules a re‑render; React passes new `count` to the next render.
- Updates may be **batched** and are **asynchronous**.

### 5.2 Updating state correctly

State should be treated as **immutable**:

```jsx
// ❌ Wrong: mutating state
user.name = "New Name";
setUser(user);

// ✅ Correct: create new object
setUser(prev => ({ ...prev, name: "New Name" }));
```

When new state depends on old state, always use the **functional form**:

```jsx
setCount(prev => prev + 1);
```

### 5.3 Multiple pieces of state

You can have multiple `useState` calls:

```jsx
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");
const [age, setAge] = useState(0);
```

or group them into one object if they belong together (but then update carefully):

```jsx
const [form, setForm] = useState({ firstName: "", lastName: "", age: 0 });

setForm(prev => ({ ...prev, firstName: "Rahul" }));
```

***

## 6. Side Effects with `useEffect`

### 6.1 What is a “side effect”?

Side effects are operations that affect something **outside** the function:

- Data fetching (`fetch`, `axios`)
- Subscribing to WebSocket or event listeners
- Timers (`setTimeout`, `setInterval`)
- Manually modifying DOM (`document.title`, refs)

`useEffect` tells React to run some code **after it renders the component**. [developer.mozilla](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Frameworks_libraries/React_getting_started)

### 6.2 Basic `useEffect` usage

```jsx
import { useEffect, useState } from "react";

function OnlineUsers() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    let cancelled = false;

    async function load() {
      const res = await fetch("/api/online-users");
      const data = await res.json();
      if (!cancelled) {
        setUsers(data);
      }
    }

    load();

    // cleanup if component unmounts before fetch completes
    return () => {
      cancelled = true;
    };
  }, []); // run once after mount

  // render logic...
}
```

### 6.3 Dependency array

`useEffect(effect, dependencies?)`:

- `[]`: run once after initial render (like `componentDidMount`).
- `[userId]`: run after mount and whenever `userId` changes.
- no second argument: run after **every** render (rare; often a smell).

Example: update document title when count changes:

```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

**Common mistakes:**

- Missing dependencies: leads to stale closures. React DevTools / ESLint plugin helps detect this.
- Doing heavy work synchronously in render instead of in an effect.

***

## 7. Lifecycle Mapping: Classes vs Hooks

Even though new code uses Hooks, interviews may ask about lifecycle equivalents.

| Class lifecycle           | Hooks equivalent / mental model                    |
|---------------------------|----------------------------------------------------|
| `constructor`            | Initialize state with `useState` defaults          |
| `componentDidMount`      | `useEffect(..., [])`                               |
| `componentDidUpdate`     | `useEffect(..., [deps])`                           |
| `componentWillUnmount`   | Cleanup function returned from `useEffect`         |
| `shouldComponentUpdate`  | `React.memo` + custom comparison, or `useMemo`     |

Example mapping:

```jsx
useEffect(() => {
  console.log("Mount or userId changed");

  return () => {
    console.log("Cleanup before next effect or unmount");
  };
}, [userId]);
```

This behaves like `componentDidMount` + `componentDidUpdate` for `userId` + `componentWillUnmount`.

***

## 8. Lists, Keys, and Conditional Rendering

### 8.1 Rendering lists

```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

**Keys must be:**

- Stable
- Unique among siblings
- Predictable (use DB id, not array index if list is dynamic)

Why keys matter:

- React uses keys to match elements between renders during reconciliation.
- Wrong keys can cause items to “jump” or lose state.

### 8.2 Conditional rendering patterns

Ternary operator:

```jsx
{isLoggedIn ? <Dashboard /> : <LoginForm />}
```

Logical AND:

```jsx
{error && <p className="error">{error}</p>}
```

Early return:

```jsx
if (loading) return <Spinner />;
if (error) return <ErrorView message={error} />;
return <DataView data={data} />;
```

***

## 9. Forms, Controlled Components, and Validation

### 9.1 Controlled components

A controlled input’s value is driven by React state. [react](https://react.dev/learn)

```jsx
function LoginForm() {
  const [form, setForm] = useState({ email: "", password: "" });

  function handleChange(e) {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
  }

  function handleSubmit(e) {
    e.preventDefault();
    // send form to API
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        value={form.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        name="password"
        type="password"
        value={form.password}
        onChange={handleChange}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

Advantages:

- Single source of truth (React).
- Easy validation and conditional UI (disable button, show messages).

### 9.2 Basic validation pattern

```jsx
const [errors, setErrors] = useState({});

function validate(form) {
  const newErrors = {};
  if (!form.email.includes("@")) newErrors.email = "Invalid email";
  if (form.password.length < 6) newErrors.password = "Min 6 chars";
  return newErrors;
}

function handleSubmit(e) {
  e.preventDefault();
  const newErrors = validate(form);
  if (Object.keys(newErrors).length > 0) {
    setErrors(newErrors);
    return;
  }
  // submit
}
```

***

## 10. Context API & Avoiding Prop Drilling

### 10.1 When to use context

Context is for sharing data that is needed by many components at different nesting levels, e.g.:

- Theme (light/dark)
- Authenticated user and permissions
- Localization, configuration

Avoid using context as a global dumping ground; it can cause unnecessary re‑renders.

### 10.2 Creating and using a context

```jsx
import { createContext, useContext, useState } from "react";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const value = {
    user,
    login: (userData) => setUser(userData),
    logout: () => setUser(null)
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth must be used within AuthProvider");
  return ctx;
}
```

Usage:

```jsx
function ProfileMenu() {
  const { user, logout } = useAuth();

  if (!user) return <a href="/login">Login</a>;

  return (
    <div>
      <span>Hi, {user.name}</span>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

**Performance tip:**  
For large apps, split contexts (AuthContext, ThemeContext, etc.) and avoid putting highly volatile values (like search input) in context.

***

## 11. Advanced Hooks: `useReducer`, `useMemo`, `useCallback`, `useRef`

### 11.1 `useReducer` – complex or branching state updates

Use when:

- State has multiple related fields and complex transitions.
- You want Redux‑style reducer logic locally.

```jsx
import { useReducer } from "react";

const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + state.step };
    case "decrement":
      return { ...state, count: state.count - state.step };
    case "setStep":
      return { ...state, step: action.payload };
    default:
      throw new Error("Unknown action type");
  }
}

function AdvancedCounter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <>
      <p>Count: {state.count}</p>
      <input
        type="number"
        value={state.step}
        onChange={e => dispatch({ type: "setStep", payload: Number(e.target.value) })}
      />
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
}
```

### 11.2 `useMemo` – memoizing expensive computations

```jsx
const filteredItems = useMemo(
  () => items.filter(item => item.toLowerCase().includes(query.toLowerCase())),
  [items, query]
);
```

Use when:

- Computation is expensive, and dependencies change infrequently.
- Do not use simply to “avoid every re‑render”; that often adds complexity without real gain.

### 11.3 `useCallback` – stable callback identity

```jsx
const handleToggle = useCallback(
  (id) => {
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  },
  [setTodos]
);
```

Useful with:

- `React.memo`‑wrapped children to prevent unnecessary renders.
- Event handlers passed deep down.

### 11.4 `useRef` – DOM refs and instance variables

```jsx
import { useRef, useEffect } from "react";

function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} placeholder="I get focused on mount" />;
}
```

Also for **mutable values** that don’t trigger re‑render:

```jsx
const renderCount = useRef(0);
renderCount.current++;
console.log("Rendered", renderCount.current, "times");
```

***

## 12. Custom Hooks & Reusable Business Logic

Custom hooks let you extract reusable logic that uses hooks internally. [react](https://react.dev/reference/rules/rules-of-hooks)

**Rules of hooks:** [geeksforgeeks](https://www.geeksforgeeks.org/reactjs/what-are-the-rules-of-hooks-in-react-and-why-are-they-important/)

- Call hooks only:
  - At the top level of React function components or other custom hooks.
  - Never inside loops, conditions, or nested functions.
- Always call hooks in the **same order** on every render.
- Custom hooks must start with `use` (e.g., `useAuth`, `useFetch`).

Example: `useLocalStorage` hook

```jsx
import { useEffect, useState } from "react";

export function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const saved = window.localStorage.getItem(key);
    if (saved != null) {
      try {
        return JSON.parse(saved);
      } catch {
        return saved;
      }
    }
    return initialValue;
  });

  useEffect(() => {
    window.localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

Usage:

```jsx
const [theme, setTheme] = useLocalStorage("theme", "light");
```

***

## 13. Routing and SPA Architecture (React Router)

Routing is not built into React; **React Router** is the de‑facto standard for client‑side routing.

### 13.1 Basic SPA routing

```jsx
import { BrowserRouter, Routes, Route, Link } from "react-router-dom";
import Home from "./Home";
import About from "./About";
import User from "./User";

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>{" | "}
        <Link to="/about">About</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users/:id" element={<User />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### 13.2 Route params and navigation

```jsx
import { useParams, useNavigate } from "react-router-dom";

function User() {
  const { id } = useParams();
  const navigate = useNavigate();

  return (
    <>
      <h2>User {id}</h2>
      <button onClick={() => navigate("/")}>Go home</button>
    </>
  );
}
```

***

## 14. State Management Strategies (Local, Context, Redux, Server State)

**Important interview concept:** differentiate **UI state vs server/cache state**.

### 14.1 Local state (Hooks)

- Best for state that concerns a single component or small subtree.

### 14.2 Context

- For relatively stable global values (theme, auth, locale).
- Don’t use context for everything; it might cause wide re‑renders.

### 14.3 Redux / Zustand / MobX (client global state)

Use when:

- Many components need to read/write the same state.
- You want devtools, time‑travel debugging, middlewares, etc.

Redux (with Redux Toolkit) example reducer is beyond scope here, but be able to explain conceptually.

### 14.4 Server state and data‑fetching (React Query etc.)

- Server data is different:
  - Comes from APIs, has caching, refetching, stale‑while‑revalidate patterns.
- Tools like **TanStack Query (React Query)** shine here:
  - Manages cache, background refetching, retries, loading/error states.

Explain that **mixing server state into Redux** is often overkill today.

***

## 15. Rendering Behavior, Reconciliation & Performance Tuning

### 15.1 When do components re‑render?

A component re‑renders when:

- Its own **state** changes.
- Its **props** change (according to `Object.is` shallow comparison).
- Its parent re‑renders (unless memoized with `React.memo`).

### 15.2 Reconciliation & virtual DOM

- React compares the **previous virtual tree** and the **new one**:
  - Same type and key → update existing DOM node.
  - Different type or key → unmount old, mount new.
- Keys allow React to match elements when lists change.

**Common performance issues:**

- Passing inline functions or new objects to memoized children unnecessarily.
- Expensive computations performed directly in render without `useMemo`.
- Huge lists rendered at once (use windowing libraries like `react-window`).

### 15.3 Optimization tools

- `React.memo(Component)` – memoize component:
  ```jsx
  const UserList = React.memo(function UserList({ users }) {
    // ...
  });
  ```
- `useMemo` – memoize computed values.
- `useCallback` – memoize callbacks passed to children.
- Code splitting / lazy loading using `React.lazy` + `Suspense`:

  ```jsx
  const AdminPanel = React.lazy(() => import("./AdminPanel"));

  function App() {
    return (
      <Suspense fallback={<Spinner />}>
        <AdminPanel />
      </Suspense>
    );
  }
  ```

***

## 16. Error Handling & Error Boundaries

Error boundaries catch runtime errors **in their child component tree** and show a fallback UI. [legacy.reactjs](https://legacy.reactjs.org/docs/design-principles.html)

Currently implemented via **class components**.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error("ErrorBoundary:", error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h2>Something went wrong.</h2>;
    }
    return this.props.children;
  }
}
```

Usage:

```jsx
<ErrorBoundary>
  <UserDashboard />
</ErrorBoundary>
```

Know limitations:

- Do not catch:
  - Errors inside event handlers
  - Async errors (e.g., `setTimeout`)
  - Server side rendering errors

***

## 17. React 18+ Concurrent Features & Suspense

React 18 enabled **concurrent rendering** by default with `createRoot`. [dev](https://dev.to/rinonten/understanding-concurrency-in-react-a-guide-to-smoother-and-more-responsive-uis-1p70)

### 17.1 Concurrent rendering (conceptual)

- React can **pause**, **interrupt**, and **resume** renders.
- Urgent updates (typing, clicks) are prioritized over non‑urgent (background data updates). [certificates](https://certificates.dev/blog/react-concurrent-features-an-overview)
- This leads to smoother UI for complex apps.

### 17.2 `startTransition` and `useTransition`

Mark updates as **non‑urgent transitions**:

```jsx
import { useState, useTransition } from "react";

function Search() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    const value = e.target.value;
    setQuery(value);

    startTransition(() => {
      const filtered = expensiveFilter(allItems, value);
      setResults(filtered);
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <p>Updating results…</p>}
      {/* render results */}
    </>
  );
}
```

### 17.3 Suspense (conceptual)

- Lets you **suspend rendering** of a component tree while waiting for data and show a fallback UI.
- Integrated more strongly in data‑fetching frameworks like Next.js.

***

## 18. React Server Components – Conceptual Overview

Server Components (RSC) are an advanced concept mostly surfaced through frameworks like **Next.js (App Router)**. [reddit](https://www.reddit.com/r/react/comments/1ot7smx/what_are_the_most_important_react_concepts_to/)

High level:

- Some components run **only on the server**:
  - They can access DB, filesystem, secrets safely.
  - They render to a serialized tree (not HTML exactly, but a protocol).
- Client receives a mix of:
  - HTML/content from server components.
  - JS bundles for **client components** (those that use hooks, browser APIs).
- Benefits:
  - Less JavaScript shipped to client.
  - Better performance and SEO.

For interviews, keep it conceptual: you understand that React is moving towards **hybrid server+client rendering**, not purely CSR.

***

## 19. Testing React Components

Common stack:

- **Jest** – test runner + assertions.
- **React Testing Library (RTL)** – tests components via DOM, not implementation details.

Example:

```jsx
// Counter.jsx
import { useState } from "react";

export function Counter() {
  const [count, setCount] = useState(0);
  return (
    <>
      <span aria-label="count">{count}</span>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </>
  );
}
```

```jsx
// Counter.test.jsx
import { render, screen, fireEvent } from "@testing-library/react";
import { Counter } from "./Counter";

test("increments counter when button is clicked", () => {
  render(<Counter />);

  const button = screen.getByText(/increment/i);
  fireEvent.click(button);

  expect(screen.getByLabelText("count")).toHaveTextContent("1");
});
```

Focus on:

- Testing what the **user sees and does**.
- Using queries like `getByRole`, `getByText`, `getByLabelText`.

***

## 20. Project Structure, Tooling & Environment

Typical React project (Vite or CRA):

```
src/
  components/
    common/
    layout/
  pages/
  hooks/
  context/
  App.jsx
  main.jsx
public/
```

**Tooling:**

- **Vite** – modern dev server and bundler (faster than CRA).
- **ESLint + Prettier** – linting and formatting.
- **TypeScript** – for type safety in larger apps.

**Environment variables:**

- Vite: `VITE_API_BASE_URL`
- Create React App: `REACT_APP_API_BASE_URL`

***

## 21. Common Interview Questions & How to Answer

Below are concise answer hints; expand them in your own words.

1. **What is React and why is it popular?**  
   - Library for building UI, declarative, component‑based, strong ecosystem, good performance.

2. **What is JSX? Is it mandatory?**  
   - Syntax extension that compiles to `React.createElement`. Not mandatory; you can use plain JS, but JSX is standard.

3. **Props vs State?**  
   - Props: external, read‑only, passed from parent.  
   - State: internal, mutable, managed by component via hooks.

4. **Explain the Virtual DOM.**  
   - React keeps an in‑memory representation of UI; on updates, it diffs old vs new trees and applies minimal DOM changes.

5. **What are Hooks? Why were they introduced?**  
   - Functions that let you “hook into” React features (state, lifecycle) from function components. Introduced to avoid class component complexity and enable better reuse via custom hooks. [geeksforgeeks](https://www.geeksforgeeks.org/reactjs/introduction-to-react-hooks/)

6. **Rules of Hooks?**  
   - Only call hooks at top level of component/custom hook.
   - Only call from React components/custom hooks.
   - Must be called in same order each render. [geeksforgeeks](https://www.geeksforgeeks.org/reactjs/what-are-the-rules-of-hooks-in-react-and-why-are-they-important/)

7. **What is `useEffect` used for? Difference from `useLayoutEffect`?**  
   - `useEffect` for side effects after paint; `useLayoutEffect` runs synchronously after DOM updates but before paint (for measuring layout, etc.).

8. **What are keys in lists? Why not use index?**  
   - Keys help React identify items between renders. Index causes bugs when list order changes or items are inserted/deleted.

9. **What is Context? When would you use Redux instead?**  
   - Context for global-ish static data (theme, auth). Redux (or other libraries) for complex shared state, cross‑cutting concerns, debugging, middleware.

10. **What are error boundaries? Can they be functional components?**  
    - Components that catch JS errors in subtree and show fallback. Currently implemented as class components.

11. **Explain React 18 concurrent features.**  
    - React can interrupt and prioritize renders; `startTransition`/`useTransition`, Suspense improvements, smoother UI for heavy updates. [buttercms](https://buttercms.com/blog/react-concurrent-mode-the-future-of-web-apps/)

12. **What are React Server Components?**  
    - Components rendered on server only, can fetch data directly, reduce bundle size; exposed mostly via frameworks like Next.js.

Prepare short 2–3 minute explanations with **examples** for each.

***

## 22. End‑to‑End Example: Todo App with Filters and Persistence

This example is intentionally “interview‑ish”: uses multiple concepts cleanly.

### 22.1 `useLocalStorage` custom hook

```jsx
// src/hooks/useLocalStorage.js
import { useEffect, useState } from "react";

export function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    if (typeof window === "undefined") return initialValue;
    try {
      const stored = window.localStorage.getItem(key);
      return stored != null ? JSON.parse(stored) : initialValue;
    } catch {
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch {
      // ignore quota or serialization errors
    }
  }, [key, value]);

  return [value, setValue];
}
```

### 22.2 `App.jsx` – main container

```jsx
// src/App.jsx
import { useMemo, useCallback } from "react";
import { useLocalStorage } from "./hooks/useLocalStorage";
import { TodoForm } from "./components/TodoForm";
import { TodoList } from "./components/TodoList";
import { FilterBar } from "./components/FilterBar";

let nextId = 1;

const FILTERS = {
  all: () => true,
  active: todo => !todo.completed,
  completed: todo => todo.completed
};

export default function App() {
  const [todos, setTodos] = useLocalStorage("todos", []);
  const [filterKey, setFilterKey] = useLocalStorage("filter", "all");

  const handleAdd = useCallback((text) => {
    setTodos(prev => [
      ...prev,
      { id: nextId++, text, completed: false }
    ]);
  }, [setTodos]);

  const handleToggle = useCallback((id) => {
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, [setTodos]);

  const handleDelete = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, [setTodos]);

  const filteredTodos = useMemo(() => {
    const predicate = FILTERS[filterKey] || FILTERS.all;
    return todos.filter(predicate);
  }, [todos, filterKey]);

  const remainingCount = useMemo(
    () => todos.filter(t => !t.completed).length,
    [todos]
  );

  return (
    <div className="app">
      <h1>Todo App</h1>

      <TodoForm onAdd={handleAdd} />

      <FilterBar
        value={filterKey}
        onChange={setFilterKey}
        remaining={remainingCount}
      />

      <TodoList
        todos={filteredTodos}
        onToggle={handleToggle}
        onDelete={handleDelete}
      />
    </div>
  );
}
```

Concepts:

- Custom hook (`useLocalStorage`) for persistence.
- State + derived data (`remainingCount`).
- `useMemo`/`useCallback` for optimization.
- Clean separation of concerns.

### 22.3 `TodoForm.jsx`

```jsx
// src/components/TodoForm.jsx
import { useState } from "react";

export function TodoForm({ onAdd }) {
  const [text, setText] = useState("");

  function handleSubmit(e) {
    e.preventDefault();
    const trimmed = text.trim();
    if (!trimmed) return;
    onAdd(trimmed);
    setText("");
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
        placeholder="What needs to be done?"
      />
      <button type="submit">Add</button>
    </form>
  );
}
```

### 22.4 `TodoList.jsx`

```jsx
// src/components/TodoList.jsx
import React from "react";

export const TodoList = React.memo(function TodoList({
  todos,
  onToggle,
  onDelete
}) {
  if (todos.length === 0) {
    return <p>No todos</p>;
  }

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <label>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => onToggle(todo.id)}
            />
            <span style={{
              textDecoration: todo.completed ? "line-through" : "none"
            }}>
              {todo.text}
            </span>
          </label>
          <button onClick={() => onDelete(todo.id)}>X</button>
        </li>
      ))}
    </ul>
  );
});
```

### 22.5 `FilterBar.jsx`

```jsx
// src/components/FilterBar.jsx
const OPTIONS = [
  { key: "all", label: "All" },
  { key: "active", label: "Active" },
  { key: "completed", label: "Completed" }
];

export function FilterBar({ value, onChange, remaining }) {
  return (
    <div className="filters">
      <span>{remaining} items left</span>
      {OPTIONS.map(option => (
        <button
          key={option.key}
          onClick={() => onChange(option.key)}
          style={{
            fontWeight: value === option.key ? "bold" : "normal"
          }}
        >
          {option.label}
        </button>
      ))}
    </div>
  );
}
```

This mini‑app demonstrates:

- Hooks (`useState`, custom hook, `useMemo`, `useCallback`).
- Lists, keys, controlled input.
- Derived state and simple performance considerations.
- Clean separation of UI and logic → exactly the style expected in interviews.

***

These notes are ready to be saved directly as a `.md` file. You can now:

- Add your own sections (e.g., “React + TypeScript”, “React with Next.js”).
- Insert short Q&A sections beneath each heading to practice verbal explanations.
- Build small demo components for tricky concepts (`useEffect` dependencies, context splitting, RSC mental model) to solidify them.