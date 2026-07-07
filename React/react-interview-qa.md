# React.js Interview Questions & Answers
### For 4 Years Experience — React Frontend + Spring Boot Backend

> 100 Questions Total: 50 Conceptual + 50 Coding Questions

---

## Table of Contents

**Conceptual Questions**
- [Section A: Core React Concepts (Q1–12)](#section-a-core-react-concepts)
- [Section B: Hooks Deep Dive (Q13–22)](#section-b-hooks-deep-dive)
- [Section C: Performance Optimization (Q23–30)](#section-c-performance-optimization)
- [Section D: State Management (Q31–38)](#section-d-state-management)
- [Section E: React Router & Navigation (Q39–42)](#section-e-react-router--navigation)
- [Section F: Testing (Q43–46)](#section-f-testing)
- [Section G: React + Spring Boot Integration (Q47–50)](#section-g-react--spring-boot-integration)

**Coding Questions**
- [Section H: Custom Hooks (Q51–60)](#section-h-custom-hooks-coding)
- [Section I: Component Design Patterns (Q61–70)](#section-i-component-design-patterns-coding)
- [Section J: Lists, Arrays & Data Manipulation (Q71–75)](#section-j-lists-arrays--data-manipulation-coding)
- [Section K: Performance Coding (Q76–80)](#section-k-performance-coding)
- [Section L: State Management Coding (Q81–83)](#section-l-state-management-coding)
- [Section M: Async / API Handling (Q84–88)](#section-m-async--api-handling-coding)
- [Section N: JavaScript Fundamentals Often Asked in React Interviews (Q89–93)](#section-n-javascript-fundamentals-coding)
- [Section O: React + Spring Boot Integration Coding (Q94–97)](#section-o-react--spring-boot-integration-coding)
- [Section P: Miscellaneous (Q98–100)](#section-p-miscellaneous-coding)

---

## Section A: Core React Concepts

### Q1. What is React and why is it used?
React is a JavaScript library for building user interfaces using a component-based architecture. It uses a Virtual DOM to efficiently update the UI, supports declarative programming, and enables reusable UI components — making large, dynamic applications easier to build and maintain.

### Q2. What is the Virtual DOM and how does it work?
The Virtual DOM is an in-memory, lightweight copy of the real DOM. When state changes, React creates a new Virtual DOM tree, diffs it against the previous one (reconciliation), and calculates the minimal set of changes needed. It then applies only those changes to the real DOM, which is expensive to manipulate directly.

### Q3. What is JSX?
JSX (JavaScript XML) is a syntax extension that lets you write HTML-like code inside JavaScript. It gets compiled (via Babel) into `React.createElement()` calls. JSX makes component structure more readable and allows embedding JS expressions using `{}`.

### Q4. What is the difference between functional and class components?
Functional components are plain JS functions that return JSX and use Hooks for state/lifecycle. Class components extend `React.Component`, use `this.state`, and lifecycle methods like `componentDidMount`. Since React 16.8, functional components with Hooks are the standard — they're simpler, avoid `this` binding issues, and support better code reuse.

### Q5. What are props and how do they differ from state?
Props (properties) are read-only data passed from a parent to a child component; they cannot be modified by the child. State is local, mutable data managed within a component that can change over time and trigger re-renders when updated via `setState` or a Hook's setter.

### Q6. Explain the component lifecycle in React.
For class components: **Mounting** (`constructor` → `render` → `componentDidMount`), **Updating** (`shouldComponentUpdate` → `render` → `componentDidUpdate`), **Unmounting** (`componentWillUnmount`). In functional components, `useEffect` covers all three phases based on its dependency array and cleanup function.

### Q7. What is the difference between controlled and uncontrolled components?
A controlled component's form value is driven by React state (`value` + `onChange`), giving full control and validation ability. An uncontrolled component stores its own state internally in the DOM and is accessed via `ref`s — simpler but harder to validate/control programmatically.

### Q8. What are keys in React lists and why are they important?
Keys are unique identifiers given to list items so React can track which items changed, were added, or removed during reconciliation, instead of re-rendering the entire list. Using array indexes as keys is discouraged when list order can change, since it can cause incorrect UI state association.

### Q9. What is prop drilling and how can it be avoided?
Prop drilling is passing data through multiple layers of components that don't need it themselves, just to reach a deeply nested child. It can be avoided using the Context API, state management libraries (Redux, Zustand), or component composition.

### Q10. What is the difference between `React.Fragment` and a `div` wrapper?
`React.Fragment` (or `<>...</>`) groups a list of children without adding an extra node to the actual DOM, avoiding unnecessary wrapper elements that could break CSS layouts (e.g., flex/grid) or add invalid nesting. A `div` wrapper actually renders as a DOM node.

### Q11. What are synthetic events in React?
SyntheticEvent is React's cross-browser wrapper around the browser's native event system. It normalizes event behavior across browsers and pools event objects for performance (in React <17). It provides the same interface as native events (`stopPropagation`, `preventDefault`, etc.).

### Q12. What is the difference between `useEffect` cleanup and `componentWillUnmount`?
Both handle cleanup logic, but `useEffect`'s cleanup function runs before every re-execution of the effect **and** on unmount (based on dependency changes), while `componentWillUnmount` only runs once, right before a class component is removed from the DOM.

---

## Section B: Hooks Deep Dive

### Q13. What are Hooks and why were they introduced?
Hooks are functions (like `useState`, `useEffect`) that let functional components use state and lifecycle features without writing a class. They were introduced to solve problems like reusing stateful logic between components (avoiding wrapper hell from HOCs/render props), simplifying complex components, and avoiding confusing `this` behavior.

### Q14. Explain `useState` vs `useReducer` — when would you choose one over the other?
`useState` is ideal for simple, independent state values. `useReducer` is better when state logic is complex, involves multiple sub-values, or the next state depends on the previous one in complicated ways (e.g., a shopping cart with add/remove/update actions) — it centralizes state transitions in a reducer function, similar to Redux.

### Q15. What is the dependency array in `useEffect` and what happens if you omit it?
The dependency array tells React when to re-run the effect — it re-runs whenever any listed value changes. Omitting it entirely causes the effect to run after **every** render. An empty array `[]` runs the effect only once, after the initial mount.

### Q16. What is the difference between `useMemo` and `useCallback`?
`useMemo` memoizes a **computed value**, recalculating it only when its dependencies change — useful for expensive calculations. `useCallback` memoizes a **function reference** itself, preventing unnecessary re-creation of functions on every render — useful when passing callbacks to memoized child components.

### Q17. What are custom hooks and what rules must they follow?
Custom hooks are JS functions whose names start with `use` and that call other Hooks inside them, allowing you to extract and reuse stateful logic across components. They must follow the **Rules of Hooks**: only call Hooks at the top level (not inside loops/conditions) and only call them from React function components or other custom Hooks.

### Q18. What is `useRef` used for besides accessing DOM nodes?
`useRef` creates a mutable object (`{ current: ... }`) that persists across renders without causing a re-render when changed. Besides DOM references, it's commonly used to store previous values, timers/interval IDs, or any mutable value that shouldn't trigger a re-render.

### Q19. What is `useLayoutEffect` and how does it differ from `useEffect`?
`useLayoutEffect` runs synchronously **after DOM mutations but before the browser paints**, making it useful for reading/measuring layout (like element dimensions) and making synchronous DOM updates to avoid visual flicker. `useEffect` runs asynchronously after paint and is preferred for most side effects (data fetching, subscriptions) since it doesn't block rendering.

### Q20. What is the Context API and when should you avoid it?
Context provides a way to pass data through the component tree without manual prop drilling, via `createContext`, a `Provider`, and `useContext`. It should be avoided for very frequently-changing, high-frequency data (like animation values) since every consumer re-renders on context change — a dedicated state library or memoized providers may perform better in those cases.

### Q21. What is the stale closure problem in Hooks and how do you fix it?
It occurs when a function (e.g., inside `useEffect` or `setInterval`) captures an outdated version of state/props from a previous render because it wasn't included in the dependency array. Fixes include adding the correct dependencies, using the functional form of `setState` (`setCount(c => c + 1)`), or storing the latest value in a `ref`.

### Q22. What is `useImperativeHandle` used for?
It customizes the instance value exposed to parent components when using `ref` with `forwardRef`, letting you expose only specific methods/properties from a child component (like `focus()` or `reset()`) instead of the whole DOM node — useful for building reusable input/form components.

---

## Section C: Performance Optimization

### Q23. How does `React.memo` improve performance?
`React.memo` is a higher-order component that memoizes a functional component, skipping re-render if its props haven't changed (shallow comparison). This is useful for pure presentational components that receive the same props frequently but whose parent re-renders often.

### Q24. What causes unnecessary re-renders in React and how do you prevent them?
Common causes: parent re-renders passing new object/array/function references as props each time, unnecessary state updates, and context value changes. Prevent them using `React.memo`, `useMemo`/`useCallback` to stabilize references, splitting state/context to narrower scopes, and avoiding inline object/function literals in JSX where it matters.

### Q25. What is code-splitting and how do you implement it in React?
Code-splitting breaks the app bundle into smaller chunks loaded on demand rather than one large bundle upfront, improving initial load time. It's implemented via dynamic `import()`, `React.lazy()` for component-level splitting, combined with `Suspense` to show a fallback while the chunk loads.

### Q26. What is list virtualization / windowing and when is it needed?
Virtualization renders only the visible portion of a large list (plus a small buffer) instead of the entire dataset, drastically reducing DOM nodes and improving performance for lists with hundreds/thousands of items. Libraries like `react-window` or `react-virtualized` implement this.

### Q27. How does React 18's automatic batching improve performance?
Before React 18, state updates were only batched inside React event handlers. React 18 batches multiple `setState` calls together even inside promises, timeouts, and native event handlers, reducing the number of re-renders triggered by grouped updates.

### Q28. What is `useTransition` and how does it help with performance?
`useTransition` lets you mark certain state updates as **non-urgent ("transitions")**, so React can prioritize more urgent updates (like typing in an input) and render the transition update without blocking the UI, keeping the app responsive during expensive re-renders.

### Q29. How would you profile and identify performance bottlenecks in a React app?
Using React DevTools' **Profiler tab** to record renders and see which components re-rendered and why (highlighting wasted renders), the browser's Performance tab for overall runtime cost, and tools like `why-did-you-render` to flag unnecessary re-renders during development.

### Q30. What is the difference between `shouldComponentUpdate` and `PureComponent`?
`shouldComponentUpdate` is a lifecycle method you implement manually to control whether a class component re-renders, giving full custom comparison logic. `PureComponent` automatically implements a shallow prop/state comparison for you, so you don't have to write it yourself — but it can't do deep comparisons.

---

## Section D: State Management

### Q31. When would you choose Context API over Redux (or vice versa)?
Context API is suitable for simple, low-frequency global state (theme, auth user, locale) in small-to-medium apps without needing middleware or dev tools. Redux (or Redux Toolkit) is preferable for large apps with complex state interactions, need for time-travel debugging, middleware (like async logic via thunks/sagas), and predictable state updates across many features.

### Q32. What problem does Redux Toolkit solve compared to classic Redux?
Redux Toolkit (RTK) reduces Redux boilerplate by providing `configureStore` (with good defaults like Redux DevTools and thunk middleware built in), `createSlice` (combines actions + reducers), and `createAsyncThunk` for async logic — while using Immer internally so you can write "mutating" logic that's actually immutable under the hood.

### Q33. Explain the unidirectional data flow in Redux.
Data flows in one direction: a component dispatches an **action** → the **reducer** (pure function) computes a new state based on the action and current state → the **store** updates → subscribed components re-render with the new state. This predictability makes debugging and testing easier compared to two-way binding.

### Q34. What are selectors and why use libraries like `reselect`?
Selectors are functions that extract/derive specific pieces of data from the store. `reselect` memoizes selectors so that expensive derived-data computations (e.g., filtering/sorting large lists) only re-run when their specific input state actually changes, preventing unnecessary recalculations and re-renders.

### Q35. How do you handle server state (API data) differently from client/UI state?
Server state (data fetched from APIs) is asynchronous, can become stale, and needs caching/invalidation — best handled by libraries like **React Query (TanStack Query)** or **RTK Query**, which manage caching, background refetching, and loading/error states out of the box. Client/UI state (like modal open/closed, form inputs) is typically simpler and handled with `useState`/Context/Redux.

### Q36. What is the difference between local component state and lifted state?
Local state lives and is only used within a single component. "Lifting state up" means moving shared state to the closest common ancestor of the components that need it, then passing it down via props (and callbacks to update it) — this is the standard React pattern before reaching for global state tools.

### Q37. How does `useReducer` combined with Context replace simple Redux use cases?
You can create a Context that provides both the state and a `dispatch` function from a `useReducer` hook at a high level in the tree, letting any nested component read state via `useContext` and dispatch actions — mimicking Redux's pattern without adding an external library, suitable for medium-complexity global state.

### Q38. What are the tradeoffs of using too many nested Context providers?
Excessive nested providers ("provider hell") hurt readability, make debugging harder, and each context triggers re-renders in all its consumers when its value changes — even if a consumer only cares about part of that value. Solutions include combining related contexts, splitting contexts by concern, or migrating to a dedicated state library.

---

## Section E: React Router & Navigation

### Q39. What is the difference between `BrowserRouter` and `HashRouter`?
`BrowserRouter` uses the HTML5 History API to keep the UI in sync with clean URLs (e.g., `/dashboard`), requiring server-side configuration to handle direct navigation/refreshes. `HashRouter` uses a URL hash (`/#/dashboard`) so the server always serves `index.html`, avoiding server config, but with less clean URLs — usually reserved for static hosting without server-side routing support.

### Q40. How do you implement protected/private routes in React Router?
Create a wrapper component (e.g., `<ProtectedRoute>`) that checks authentication state (from Context, Redux, or a token check) and either renders the requested route's element or redirects to a login page using `<Navigate to="/login" />` from `react-router-dom`.

### Q41. What are nested routes and layout routes used for?
Nested routes let child routes render inside a parent route's `<Outlet />`, enabling shared layouts (like a sidebar/header) across multiple pages without duplicating that layout code — the matched child route content is injected into the `Outlet` placeholder.

### Q42. How do you handle 404 / not-found pages and dynamic route parameters?
A catch-all route (`path="*"`) at the end of the route list renders a NotFound component for unmatched URLs. Dynamic parameters are defined with a colon (e.g., `path="/users/:id"`) and read in a component via the `useParams()` hook.

---

## Section F: Testing

### Q43. What is the difference between unit testing and integration testing in a React app?
Unit tests verify a single component or function in isolation (often mocking dependencies), while integration tests verify that multiple components/modules work correctly together (e.g., a form component correctly updating state and calling an API mock) — React Testing Library encourages integration-style tests that resemble real user interaction.

### Q44. What is React Testing Library's philosophy compared to Enzyme?
React Testing Library encourages testing components the way a user interacts with them — via visible text, roles, and labels — rather than testing internal implementation details (like component instance state), which Enzyme historically allowed. This makes tests more resilient to refactors that don't change user-facing behavior.

### Q45. How do you mock an API call in a Jest/RTL test?
Using `jest.mock()` to mock a module (like an Axios wrapper or service file) and have it return a resolved/rejected promise with test data, or using **MSW (Mock Service Worker)** to intercept actual network requests at the network level for more realistic integration tests without changing app code.

### Q46. How would you test a custom hook?
Using `@testing-library/react-hooks` (or `renderHook` from `@testing-library/react` in newer versions) to render the hook in a test harness, then asserting on its returned state/values and using `act()` to wrap state-updating calls so React processes updates correctly before assertions.

---

## Section G: React + Spring Boot Integration

### Q47. How do you handle CORS issues when a React app (e.g., on `localhost:3000`) calls a Spring Boot API (e.g., on `localhost:8080`)?
Configure CORS on the Spring Boot side — either via `@CrossOrigin(origins = "http://localhost:3000")` on controllers, or globally via a `WebMvcConfigurer` bean overriding `addCorsMappings`, specifying allowed origins, methods, and headers. In production, this is typically restricted to the actual deployed frontend domain.

### Q48. How do you manage JWT-based authentication between React and Spring Boot?
The Spring Boot backend issues a JWT on login (via Spring Security + a JWT filter), which React stores (in memory, or `httpOnly` cookie for better security than `localStorage`) and attaches to subsequent requests as an `Authorization: Bearer <token>` header — often centralized via an Axios interceptor so every request automatically includes it, and a response interceptor handles 401s by redirecting to login or refreshing the token.

### Q49. How do you handle file uploads from React to a Spring Boot REST endpoint?
On the frontend, build a `FormData` object appending the file (from an `<input type="file">`), and POST it via Axios/fetch with `Content-Type: multipart/form-data` (letting the browser set the boundary automatically). On the backend, the Spring controller accepts it via `@RequestParam("file") MultipartFile file`.

### Q50. How do you handle global error responses (e.g., validation errors, 500s) consistently between Spring Boot and React?
On Spring Boot, use a `@ControllerAdvice` with `@ExceptionHandler` methods to return a consistent error response shape (e.g., `{ status, message, errors: [...] }`) instead of a default stack trace. On React, centralize handling in an Axios response interceptor that inspects this shape and dispatches consistent UI feedback (toast/snackbar) or redirects, so individual components don't need repetitive try/catch logic for common cases.

---

## Section H: Custom Hooks (Coding)

### Q51. Implement a `useDebounce` custom hook.
```jsx
import { useState, useEffect } from "react";

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer); // cleanup on value/delay change or unmount
  }, [value, delay]);

  return debouncedValue;
}

// Usage: debounced search input
function Search() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 400);

  useEffect(() => {
    if (debouncedQuery) console.log("API call with:", debouncedQuery);
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

### Q52. Implement a `useFetch` custom hook with loading/error states.
```jsx
import { useState, useEffect } from "react";

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);
    fetch(url, { signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((json) => setData(json))
      .catch((err) => {
        if (err.name !== "AbortError") setError(err.message);
      })
      .finally(() => setLoading(false));

    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}
```

### Q53. Implement a `usePrevious` hook.
```jsx
import { useRef, useEffect } from "react";

function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value; // set after render, so it holds previous value during current render
  });
  return ref.current;
}

// Usage
function Counter({ count }) {
  const prevCount = usePrevious(count);
  return <p>Now: {count}, Before: {prevCount}</p>;
}
```

### Q54. Implement a `useLocalStorage` hook that syncs state with localStorage.
```jsx
import { useState, useEffect } from "react";

function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const stored = window.localStorage.getItem(key);
      return stored !== null ? JSON.parse(stored) : initialValue;
    } catch {
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (err) {
      console.error("Failed to save to localStorage", err);
    }
  }, [key, value]);

  return [value, setValue];
}
```

### Q55. Implement a `useToggle` hook.
```jsx
import { useState, useCallback } from "react";

function useToggle(initial = false) {
  const [state, setState] = useState(initial);
  const toggle = useCallback(() => setState((s) => !s), []);
  return [state, toggle];
}

// Usage
function Modal() {
  const [isOpen, toggleOpen] = useToggle(false);
  return (
    <>
      <button onClick={toggleOpen}>{isOpen ? "Close" : "Open"}</button>
      {isOpen && <div className="modal">Modal Content</div>}
    </>
  );
}
```

### Q56. Implement a `useInterval` hook (Dan Abramov's pattern) that handles changing callbacks correctly.
```jsx
import { useEffect, useRef } from "react";

function useInterval(callback, delay) {
  const savedCallback = useRef(callback);

  useEffect(() => {
    savedCallback.current = callback; // always keep latest callback, avoid stale closures
  }, [callback]);

  useEffect(() => {
    if (delay === null) return;
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}
```

### Q57. Implement a `useOnClickOutside` hook (used for closing dropdowns/modals).
```jsx
import { useEffect } from "react";

function useOnClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      if (!ref.current || ref.current.contains(event.target)) return;
      handler(event);
    };
    document.addEventListener("mousedown", listener);
    return () => document.removeEventListener("mousedown", listener);
  }, [ref, handler]);
}

// Usage
function Dropdown({ onClose }) {
  const ref = useRef(null);
  useOnClickOutside(ref, onClose);
  return <div ref={ref} className="dropdown">...</div>;
}
```

### Q58. Implement a `useWindowSize` hook.
```jsx
import { useState, useEffect } from "react";

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () =>
      setSize({ width: window.innerWidth, height: window.innerHeight });
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return size;
}
```

### Q59. Implement a `useForm` hook for basic form state + validation.
```jsx
import { useState } from "react";

function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (onSubmit) => (e) => {
    e.preventDefault();
    const validationErrors = validate(values);
    setErrors(validationErrors);
    if (Object.keys(validationErrors).length === 0) onSubmit(values);
  };

  return { values, errors, handleChange, handleSubmit };
}

// Usage
function LoginForm() {
  const validate = (v) => {
    const errs = {};
    if (!v.email) errs.email = "Email is required";
    if (!v.password) errs.password = "Password is required";
    return errs;
  };
  const { values, errors, handleChange, handleSubmit } = useForm(
    { email: "", password: "" },
    validate
  );

  return (
    <form onSubmit={handleSubmit((v) => console.log("Submit:", v))}>
      <input name="email" value={values.email} onChange={handleChange} />
      {errors.email && <span>{errors.email}</span>}
      <input name="password" type="password" value={values.password} onChange={handleChange} />
      {errors.password && <span>{errors.password}</span>}
      <button type="submit">Login</button>
    </form>
  );
}
```

### Q60. Implement a `usePagination` hook for client-side pagination.
```jsx
import { useState, useMemo } from "react";

function usePagination(data, pageSize = 10) {
  const [page, setPage] = useState(1);

  const totalPages = Math.ceil(data.length / pageSize);

  const currentData = useMemo(() => {
    const start = (page - 1) * pageSize;
    return data.slice(start, start + pageSize);
  }, [data, page, pageSize]);

  const nextPage = () => setPage((p) => Math.min(p + 1, totalPages));
  const prevPage = () => setPage((p) => Math.max(p - 1, 1));

  return { currentData, page, totalPages, nextPage, prevPage, setPage };
}
```

---

## Section I: Component Design Patterns (Coding)

### Q61. Implement a Compound Component pattern (e.g., `Tabs`).
```jsx
import { createContext, useContext, useState } from "react";

const TabsContext = createContext();

function Tabs({ children, defaultIndex = 0 }) {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);
  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ index, children }) {
  const { activeIndex, setActiveIndex } = useContext(TabsContext);
  return (
    <button
      className={activeIndex === index ? "active" : ""}
      onClick={() => setActiveIndex(index)}
    >
      {children}
    </button>
  );
}

function TabPanel({ index, children }) {
  const { activeIndex } = useContext(TabsContext);
  return activeIndex === index ? <div>{children}</div> : null;
}

Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

// Usage
// <Tabs>
//   <Tabs.List>
//     <Tabs.Tab index={0}>Profile</Tabs.Tab>
//     <Tabs.Tab index={1}>Settings</Tabs.Tab>
//   </Tabs.List>
//   <Tabs.Panel index={0}>Profile Content</Tabs.Panel>
//   <Tabs.Panel index={1}>Settings Content</Tabs.Panel>
// </Tabs>
```

### Q62. Implement a Higher-Order Component (HOC) that adds a loading spinner.
```jsx
function withLoading(WrappedComponent) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) return <div className="spinner">Loading...</div>;
    return <WrappedComponent {...props} />;
  };
}

// Usage
const UserListWithLoading = withLoading(UserList);
// <UserListWithLoading isLoading={loading} users={users} />
```

### Q63. Implement the Render Props pattern for a mouse tracker.
```jsx
import { useState } from "react";

function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return <div onMouseMove={handleMouseMove}>{render(position)}</div>;
}

// Usage
// <MouseTracker render={({ x, y }) => <p>Mouse at {x}, {y}</p>} />
```

### Q64. Build a controlled search input with debounced filtering of a list.
```jsx
import { useState, useMemo } from "react";
import useDebounce from "./useDebounce"; // from Q51

function SearchableList({ items }) {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 300);

  const filtered = useMemo(
    () =>
      items.filter((item) =>
        item.name.toLowerCase().includes(debouncedQuery.toLowerCase())
      ),
    [items, debouncedQuery]
  );

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {filtered.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Q65. Implement infinite scroll using `IntersectionObserver`.
```jsx
import { useEffect, useRef, useState, useCallback } from "react";

function useInfiniteScroll(fetchMore) {
  const [isFetching, setIsFetching] = useState(false);
  const observerRef = useRef(null);

  const lastElementRef = useCallback(
    (node) => {
      if (observerRef.current) observerRef.current.disconnect();
      observerRef.current = new IntersectionObserver((entries) => {
        if (entries[0].isIntersecting) {
          setIsFetching(true);
        }
      });
      if (node) observerRef.current.observe(node);
    },
    []
  );

  useEffect(() => {
    if (!isFetching) return;
    fetchMore().finally(() => setIsFetching(false));
  }, [isFetching, fetchMore]);

  return { lastElementRef, isFetching };
}
```

### Q66. Build a reusable Modal component using a portal.
```jsx
import { createPortal } from "react-dom";
import { useEffect } from "react";

function Modal({ isOpen, onClose, children }) {
  useEffect(() => {
    const handleEsc = (e) => e.key === "Escape" && onClose();
    if (isOpen) document.addEventListener("keydown", handleEsc);
    return () => document.removeEventListener("keydown", handleEsc);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <button onClick={onClose}>X</button>
        {children}
      </div>
    </div>,
    document.getElementById("modal-root")
  );
}
```

### Q67. Build an Accordion component where only one panel can be open at a time.
```jsx
import { useState } from "react";

function Accordion({ items }) {
  const [openIndex, setOpenIndex] = useState(null);

  const toggle = (index) =>
    setOpenIndex((prev) => (prev === index ? null : index));

  return (
    <div>
      {items.map((item, index) => (
        <div key={item.id}>
          <button onClick={() => toggle(index)}>{item.title}</button>
          {openIndex === index && <div className="panel">{item.content}</div>}
        </div>
      ))}
    </div>
  );
}
```

### Q68. Build a Todo app supporting Add, Toggle Complete, Delete, and Filter (all/active/completed).
```jsx
import { useState, useMemo } from "react";

function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [text, setText] = useState("");
  const [filter, setFilter] = useState("all");

  const addTodo = () => {
    if (!text.trim()) return;
    setTodos((prev) => [...prev, { id: Date.now(), text, completed: false }]);
    setText("");
  };

  const toggleTodo = (id) =>
    setTodos((prev) =>
      prev.map((t) => (t.id === id ? { ...t, completed: !t.completed } : t))
    );

  const deleteTodo = (id) =>
    setTodos((prev) => prev.filter((t) => t.id !== id));

  const filteredTodos = useMemo(() => {
    if (filter === "active") return todos.filter((t) => !t.completed);
    if (filter === "completed") return todos.filter((t) => t.completed);
    return todos;
  }, [todos, filter]);

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={addTodo}>Add</button>
      <div>
        {["all", "active", "completed"].map((f) => (
          <button key={f} onClick={() => setFilter(f)}>{f}</button>
        ))}
      </div>
      <ul>
        {filteredTodos.map((todo) => (
          <li key={todo.id}>
            <span
              style={{ textDecoration: todo.completed ? "line-through" : "none" }}
              onClick={() => toggleTodo(todo.id)}
            >
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Q69. Implement a multi-step form wizard with state persisted across steps.
```jsx
import { useState } from "react";

function useWizard(steps) {
  const [stepIndex, setStepIndex] = useState(0);
  const [formData, setFormData] = useState({});

  const next = (data) => {
    setFormData((prev) => ({ ...prev, ...data }));
    setStepIndex((i) => Math.min(i + 1, steps.length - 1));
  };
  const back = () => setStepIndex((i) => Math.max(i - 1, 0));

  const StepComponent = steps[stepIndex];

  return { StepComponent, next, back, formData, stepIndex, isLast: stepIndex === steps.length - 1 };
}

// Usage: const { StepComponent, next, back, formData } = useWizard([Step1, Step2, Step3]);
// <StepComponent data={formData} onNext={next} onBack={back} />
```

### Q70. Implement a drag-and-drop reorderable list (using native HTML5 drag events).
```jsx
import { useState } from "react";

function ReorderableList({ initialItems }) {
  const [items, setItems] = useState(initialItems);
  const [draggedIndex, setDraggedIndex] = useState(null);

  const handleDragStart = (index) => setDraggedIndex(index);

  const handleDrop = (index) => {
    if (draggedIndex === null || draggedIndex === index) return;
    const updated = [...items];
    const [removed] = updated.splice(draggedIndex, 1);
    updated.splice(index, 0, removed);
    setItems(updated);
    setDraggedIndex(null);
  };

  return (
    <ul>
      {items.map((item, index) => (
        <li
          key={item.id}
          draggable
          onDragStart={() => handleDragStart(index)}
          onDragOver={(e) => e.preventDefault()}
          onDrop={() => handleDrop(index)}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

---

## Section J: Lists, Arrays & Data Manipulation (Coding)

### Q71. Given a flat list of comments with `parentId`, build a nested comment tree and render it recursively.
```jsx
function buildTree(comments, parentId = null) {
  return comments
    .filter((c) => c.parentId === parentId)
    .map((c) => ({ ...c, children: buildTree(comments, c.id) }));
}

function CommentNode({ comment }) {
  return (
    <li>
      {comment.text}
      {comment.children.length > 0 && (
        <ul>
          {comment.children.map((child) => (
            <CommentNode key={child.id} comment={child} />
          ))}
        </ul>
      )}
    </li>
  );
}

function CommentTree({ comments }) {
  const tree = buildTree(comments);
  return (
    <ul>
      {tree.map((c) => (
        <CommentNode key={c.id} comment={c} />
      ))}
    </ul>
  );
}
```

### Q72. Group an array of objects by a key (e.g., group employees by department) and render each group.
```jsx
function groupBy(array, key) {
  return array.reduce((acc, item) => {
    const groupKey = item[key];
    if (!acc[groupKey]) acc[groupKey] = [];
    acc[groupKey].push(item);
    return acc;
  }, {});
}

function GroupedEmployeeList({ employees }) {
  const grouped = groupBy(employees, "department");

  return (
    <div>
      {Object.entries(grouped).map(([dept, list]) => (
        <div key={dept}>
          <h3>{dept}</h3>
          <ul>
            {list.map((emp) => (
              <li key={emp.id}>{emp.name}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

### Q73. Implement a sortable table (click column header to sort ascending/descending).
```jsx
import { useState, useMemo } from "react";

function SortableTable({ data, columns }) {
  const [sortConfig, setSortConfig] = useState({ key: null, direction: "asc" });

  const sortedData = useMemo(() => {
    if (!sortConfig.key) return data;
    return [...data].sort((a, b) => {
      const valA = a[sortConfig.key];
      const valB = b[sortConfig.key];
      if (valA < valB) return sortConfig.direction === "asc" ? -1 : 1;
      if (valA > valB) return sortConfig.direction === "asc" ? 1 : -1;
      return 0;
    });
  }, [data, sortConfig]);

  const handleSort = (key) => {
    setSortConfig((prev) => ({
      key,
      direction: prev.key === key && prev.direction === "asc" ? "desc" : "asc",
    }));
  };

  return (
    <table>
      <thead>
        <tr>
          {columns.map((col) => (
            <th key={col.key} onClick={() => handleSort(col.key)}>
              {col.label}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {sortedData.map((row) => (
          <tr key={row.id}>
            {columns.map((col) => (
              <td key={col.key}>{row[col.key]}</td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### Q74. Deduplicate an array of objects by a specific field (e.g., unique users by email).
```jsx
function dedupeByField(array, field) {
  const seen = new Set();
  return array.filter((item) => {
    if (seen.has(item[field])) return false;
    seen.add(item[field]);
    return true;
  });
}
```

### Q75. Implement a multi-select checkbox list with "select all" functionality.
```jsx
import { useState } from "react";

function MultiSelectList({ items }) {
  const [selected, setSelected] = useState(new Set());

  const toggleItem = (id) => {
    setSelected((prev) => {
      const next = new Set(prev);
      next.has(id) ? next.delete(id) : next.add(id);
      return next;
    });
  };

  const toggleAll = () => {
    setSelected((prev) =>
      prev.size === items.length ? new Set() : new Set(items.map((i) => i.id))
    );
  };

  return (
    <div>
      <label>
        <input
          type="checkbox"
          checked={selected.size === items.length && items.length > 0}
          onChange={toggleAll}
        />
        Select All
      </label>
      {items.map((item) => (
        <label key={item.id}>
          <input
            type="checkbox"
            checked={selected.has(item.id)}
            onChange={() => toggleItem(item.id)}
          />
          {item.name}
        </label>
      ))}
    </div>
  );
}
```

---

## Section K: Performance Coding

### Q76. Demonstrate correct usage of `React.memo` with a custom comparison function.
```jsx
import { memo } from "react";

function UserCard({ user }) {
  console.log("Rendering:", user.name);
  return <div>{user.name} - {user.email}</div>;
}

// Only re-render if user.id or user.name actually changes, ignore other prop changes
export default memo(UserCard, (prevProps, nextProps) => {
  return (
    prevProps.user.id === nextProps.user.id &&
    prevProps.user.name === nextProps.user.name
  );
});
```

### Q77. Fix a component that re-renders unnecessarily due to inline functions passed to a memoized child.
```jsx
// Problem: new function reference created every render, breaking React.memo on Child
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  // BAD: handleClick recreated every render
  // const handleClick = () => console.log("clicked");

  // GOOD: stable reference via useCallback
  const handleClick = useCallback(() => console.log("clicked"), []);

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <MemoizedChild onClick={handleClick} />
    </>
  );
}

const MemoizedChild = memo(function Child({ onClick }) {
  console.log("Child rendered");
  return <button onClick={onClick}>Click</button>;
});
```

### Q78. Implement a simple virtualized list (windowing) from scratch without a library.
```jsx
import { useState, useRef } from "react";

function VirtualList({ items, itemHeight = 40, containerHeight = 400 }) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef(null);

  const totalHeight = items.length * itemHeight;
  const startIndex = Math.floor(scrollTop / itemHeight);
  const visibleCount = Math.ceil(containerHeight / itemHeight) + 1;
  const endIndex = Math.min(items.length, startIndex + visibleCount);
  const visibleItems = items.slice(startIndex, endIndex);

  return (
    <div
      ref={containerRef}
      style={{ height: containerHeight, overflowY: "auto" }}
      onScroll={(e) => setScrollTop(e.target.scrollTop)}
    >
      <div style={{ height: totalHeight, position: "relative" }}>
        {visibleItems.map((item, i) => (
          <div
            key={startIndex + i}
            style={{
              position: "absolute",
              top: (startIndex + i) * itemHeight,
              height: itemHeight,
              width: "100%",
            }}
          >
            {item.label}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Q79. Use `useMemo` to avoid recomputing an expensive filtered/sorted derived list on every render.
```jsx
import { useMemo, useState } from "react";

function ProductList({ products }) {
  const [minPrice, setMinPrice] = useState(0);
  const [unrelatedState, setUnrelatedState] = useState(0); // e.g., a UI toggle

  const filteredSorted = useMemo(() => {
    console.log("Recomputing filtered/sorted list...");
    return products
      .filter((p) => p.price >= minPrice)
      .sort((a, b) => a.price - b.price);
  }, [products, minPrice]); // does NOT recompute when unrelatedState changes

  return (
    <div>
      <input
        type="number"
        value={minPrice}
        onChange={(e) => setMinPrice(Number(e.target.value))}
      />
      <button onClick={() => setUnrelatedState((s) => s + 1)}>Toggle UI</button>
      <ul>
        {filteredSorted.map((p) => (
          <li key={p.id}>{p.name} - ${p.price}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Q80. Implement `React.lazy` + `Suspense` for route-based code splitting.
```jsx
import { lazy, Suspense } from "react";
import { Routes, Route } from "react-router-dom";

const Dashboard = lazy(() => import("./pages/Dashboard"));
const Settings = lazy(() => import("./pages/Settings"));

function App() {
  return (
    <Suspense fallback={<div>Loading page...</div>}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

---

## Section L: State Management Coding

### Q81. Build a global counter using Context API + `useReducer`.
```jsx
import { createContext, useContext, useReducer } from "react";

const CounterContext = createContext();

function counterReducer(state, action) {
  switch (action.type) {
    case "increment": return { count: state.count + 1 };
    case "decrement": return { count: state.count - 1 };
    case "reset": return { count: 0 };
    default: throw new Error(`Unknown action: ${action.type}`);
  }
}

export function CounterProvider({ children }) {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  return (
    <CounterContext.Provider value={{ state, dispatch }}>
      {children}
    </CounterContext.Provider>
  );
}

export function useCounter() {
  const context = useContext(CounterContext);
  if (!context) throw new Error("useCounter must be used within CounterProvider");
  return context;
}

// Usage
function CounterDisplay() {
  const { state, dispatch } = useCounter();
  return (
    <div>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
    </div>
  );
}
```

### Q82. Implement a shopping cart with `useReducer` (add, remove, update quantity, total).
```jsx
import { useReducer } from "react";

function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM": {
      const existing = state.items.find((i) => i.id === action.payload.id);
      if (existing) {
        return {
          items: state.items.map((i) =>
            i.id === action.payload.id ? { ...i, qty: i.qty + 1 } : i
          ),
        };
      }
      return { items: [...state.items, { ...action.payload, qty: 1 }] };
    }
    case "REMOVE_ITEM":
      return { items: state.items.filter((i) => i.id !== action.payload) };
    case "UPDATE_QTY":
      return {
        items: state.items.map((i) =>
          i.id === action.payload.id ? { ...i, qty: action.payload.qty } : i
        ),
      };
    case "CLEAR":
      return { items: [] };
    default:
      return state;
  }
}

function useCart() {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });
  const total = state.items.reduce((sum, i) => sum + i.price * i.qty, 0);
  return { items: state.items, total, dispatch };
}
```

### Q83. Write a Redux Toolkit slice for a "posts" feature with async data fetching.
```jsx
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";
import axios from "axios";

export const fetchPosts = createAsyncThunk("posts/fetchPosts", async () => {
  const response = await axios.get("/api/posts");
  return response.data;
});

const postsSlice = createSlice({
  name: "posts",
  initialState: { items: [], status: "idle", error: null },
  reducers: {
    postAdded: (state, action) => {
      state.items.push(action.payload); // Immer allows "mutation" syntax
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state) => {
        state.status = "loading";
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = "succeeded";
        state.items = action.payload;
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = "failed";
        state.error = action.error.message;
      });
  },
});

export const { postAdded } = postsSlice.actions;
export default postsSlice.reducer;
```

---

## Section M: Async / API Handling (Coding)

### Q84. Set up an Axios instance with request/response interceptors for JWT auth (attach token, handle 401).
```jsx
import axios from "axios";

const api = axios.create({ baseURL: "https://api.example.com" });

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("accessToken");
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem("accessToken");
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);

export default api;
```

### Q85. Implement fetch cancellation on component unmount using `AbortController`.
```jsx
import { useEffect, useState } from "react";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    fetch(`/api/users/${userId}`, { signal: controller.signal })
      .then((res) => res.json())
      .then(setUser)
      .catch((err) => {
        if (err.name !== "AbortError") console.error(err);
      });

    return () => controller.abort(); // cancel if userId changes or component unmounts
  }, [userId]);

  return user ? <div>{user.name}</div> : <div>Loading...</div>;
}
```

### Q86. Implement a retry mechanism for a failed API call with exponential backoff.
```jsx
async function fetchWithRetry(url, options = {}, retries = 3, delay = 500) {
  try {
    const res = await fetch(url, options);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    if (retries === 0) throw err;
    await new Promise((resolve) => setTimeout(resolve, delay));
    return fetchWithRetry(url, options, retries - 1, delay * 2); // exponential backoff
  }
}
```

### Q87. Implement server-side pagination (fetch a new page from API on button click) with loading state.
```jsx
import { useState, useEffect } from "react";

function PaginatedList() {
  const [page, setPage] = useState(1);
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [totalPages, setTotalPages] = useState(1);

  useEffect(() => {
    let isCurrent = true;
    setLoading(true);
    fetch(`/api/items?page=${page}&size=10`)
      .then((res) => res.json())
      .then((json) => {
        if (isCurrent) {
          setData(json.content);
          setTotalPages(json.totalPages);
        }
      })
      .finally(() => isCurrent && setLoading(false));

    return () => { isCurrent = false; }; // prevent stale response overwrite
  }, [page]);

  return (
    <div>
      {loading ? <p>Loading...</p> : (
        <ul>{data.map((item) => <li key={item.id}>{item.name}</li>)}</ul>
      )}
      <button disabled={page === 1} onClick={() => setPage((p) => p - 1)}>Prev</button>
      <span> Page {page} of {totalPages} </span>
      <button disabled={page === totalPages} onClick={() => setPage((p) => p + 1)}>Next</button>
    </div>
  );
}
```

### Q88. Implement a hook that runs parallel API requests and combines the results.
```jsx
import { useState, useEffect } from "react";

function useParallelFetch(urls) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    Promise.all(urls.map((url) => fetch(url).then((res) => res.json())))
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [urls]);

  return { data, error, loading };
}

// Usage: const { data } = useParallelFetch(["/api/users", "/api/orders"]);
// data => [usersResponse, ordersResponse]
```

---

## Section N: JavaScript Fundamentals (Coding)

### Q89. Implement `debounce` and `throttle` functions from scratch (frequently asked alongside React hooks).
```javascript
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

function throttle(fn, limit) {
  let inThrottle = false;
  return function (...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}
```

### Q90. Implement a deep clone function without using `structuredClone` or JSON methods.
```javascript
function deepClone(obj, seen = new WeakMap()) {
  if (obj === null || typeof obj !== "object") return obj;
  if (seen.has(obj)) return seen.get(obj); // handle circular references

  const clone = Array.isArray(obj) ? [] : {};
  seen.set(obj, clone);

  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      clone[key] = deepClone(obj[key], seen);
    }
  }
  return clone;
}
```

### Q91. Implement a function to flatten a deeply nested array.
```javascript
function flattenArray(arr) {
  return arr.reduce((flat, item) => {
    return flat.concat(Array.isArray(item) ? flattenArray(item) : item);
  }, []);
}

// Alternative using built-in:
// arr.flat(Infinity)
```

### Q92. Implement a simple custom event emitter (pub-sub pattern).
```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, listener) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(listener);
    return () => this.off(event, listener); // returns unsubscribe function
  }

  off(event, listener) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter((l) => l !== listener);
  }

  emit(event, ...args) {
    if (!this.events[event]) return;
    this.events[event].forEach((listener) => listener(...args));
  }
}
```

### Q93. Implement your own version of `Function.prototype.bind` (a classic polyfill question).
```javascript
Function.prototype.myBind = function (context, ...boundArgs) {
  const originalFn = this;
  return function (...callArgs) {
    return originalFn.apply(context, [...boundArgs, ...callArgs]);
  };
};

// Usage
function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}
const bound = greet.myBind({ name: "Alice" }, "Hello");
bound(); // "Hello, Alice"
```

---

## Section O: React + Spring Boot Integration Coding

### Q94. Write an Axios call to a Spring Boot REST endpoint with JWT auth and typed error handling.
```jsx
import api from "./api"; // Axios instance from Q84

async function fetchOrders() {
  try {
    const response = await api.get("/api/orders");
    return response.data;
  } catch (err) {
    if (err.response) {
      // Backend returned an error shape from @ControllerAdvice
      console.error("Server error:", err.response.data.message);
    } else {
      console.error("Network error:", err.message);
    }
    throw err;
  }
}
```

### Q95. Build a file upload component that sends a file to a Spring Boot `MultipartFile` endpoint with a progress bar.
```jsx
import { useState } from "react";
import api from "./api";

function FileUpload() {
  const [progress, setProgress] = useState(0);

  const handleUpload = async (e) => {
    const file = e.target.files[0];
    const formData = new FormData();
    formData.append("file", file);

    await api.post("/api/files/upload", formData, {
      headers: { "Content-Type": "multipart/form-data" },
      onUploadProgress: (event) => {
        setProgress(Math.round((event.loaded * 100) / event.total));
      },
    });
  };

  return (
    <div>
      <input type="file" onChange={handleUpload} />
      <progress value={progress} max="100" />
    </div>
  );
}
```

### Q96. Implement a WebSocket/STOMP connection to a Spring Boot backend for real-time updates (e.g., notifications).
```jsx
import { useEffect, useState } from "react";
import { Client } from "@stomp/stompjs";
import SockJS from "sockjs-client";

function useNotifications(userId) {
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
    const client = new Client({
      webSocketFactory: () => new SockJS("http://localhost:8080/ws"),
      onConnect: () => {
        client.subscribe(`/topic/notifications/${userId}`, (message) => {
          setNotifications((prev) => [...prev, JSON.parse(message.body)]);
        });
      },
    });
    client.activate();

    return () => client.deactivate(); // cleanup on unmount
  }, [userId]);

  return notifications;
}
```

### Q97. Implement token refresh logic: when a Spring Boot API returns 401, silently refresh the JWT and retry the original request.
```jsx
import axios from "axios";

const api = axios.create({ baseURL: "https://api.example.com" });
let isRefreshing = false;
let queue = [];

api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        // queue requests until refresh completes
        return new Promise((resolve) => {
          queue.push((token) => {
            originalRequest.headers.Authorization = `Bearer ${token}`;
            resolve(api(originalRequest));
          });
        });
      }

      originalRequest._retry = true;
      isRefreshing = true;
      try {
        const { data } = await axios.post("/api/auth/refresh", {
          refreshToken: localStorage.getItem("refreshToken"),
        });
        localStorage.setItem("accessToken", data.accessToken);
        queue.forEach((cb) => cb(data.accessToken));
        queue = [];
        originalRequest.headers.Authorization = `Bearer ${data.accessToken}`;
        return api(originalRequest);
      } catch (refreshError) {
        window.location.href = "/login";
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }
    return Promise.reject(error);
  }
);
```

---

## Section P: Miscellaneous (Coding)

### Q98. Implement a class-based Error Boundary component.
```jsx
import { Component } from "react";

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error("Error caught by boundary:", error, errorInfo);
    // Could also log to a monitoring service here
  }

  render() {
    if (this.state.hasError) {
      return <h2>Something went wrong. Please try again later.</h2>;
    }
    return this.props.children;
  }
}

// Usage: <ErrorBoundary><App /></ErrorBoundary>
```

### Q99. Implement client-side form validation (manual, without a library) for a signup form.
```jsx
import { useState } from "react";

function validate(values) {
  const errors = {};
  if (!values.email) {
    errors.email = "Email is required";
  } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(values.email)) {
    errors.email = "Invalid email format";
  }
  if (!values.password || values.password.length < 8) {
    errors.password = "Password must be at least 8 characters";
  }
  if (values.confirmPassword !== values.password) {
    errors.confirmPassword = "Passwords do not match";
  }
  return errors;
}

function SignupForm() {
  const [values, setValues] = useState({ email: "", password: "", confirmPassword: "" });
  const [errors, setErrors] = useState({});

  const handleSubmit = (e) => {
    e.preventDefault();
    const validationErrors = validate(values);
    setErrors(validationErrors);
    if (Object.keys(validationErrors).length === 0) {
      console.log("Submitting:", values);
    }
  };

  const handleChange = (e) =>
    setValues((prev) => ({ ...prev, [e.target.name]: e.target.value }));

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" value={values.email} onChange={handleChange} placeholder="Email" />
      {errors.email && <p>{errors.email}</p>}
      <input name="password" type="password" value={values.password} onChange={handleChange} placeholder="Password" />
      {errors.password && <p>{errors.password}</p>}
      <input name="confirmPassword" type="password" value={values.confirmPassword} onChange={handleChange} placeholder="Confirm Password" />
      {errors.confirmPassword && <p>{errors.confirmPassword}</p>}
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

### Q100. Implement a simple hash-based client-side router from scratch (to demonstrate understanding of how React Router works internally).
```jsx
import { useState, useEffect } from "react";

function useHashRoute() {
  const [route, setRoute] = useState(window.location.hash.slice(1) || "/");

  useEffect(() => {
    const handleHashChange = () => setRoute(window.location.hash.slice(1) || "/");
    window.addEventListener("hashchange", handleHashChange);
    return () => window.removeEventListener("hashchange", handleHashChange);
  }, []);

  return route;
}

function MiniRouter({ routes }) {
  const route = useHashRoute();
  const Component = routes[route] || routes["/404"];
  return <Component />;
}

// Usage
// const routes = { "/": Home, "/about": About, "/404": NotFound };
// <MiniRouter routes={routes} />
// Navigate with: <a href="#/about">About</a>
```

---

## Quick Reference: Suggested Prep Strategy

1. **Week 1**: Sections A–D (Core concepts, Hooks, Performance, State Management) — be ready to explain trade-offs, not just definitions.
2. **Week 2**: Sections E–G (Router, Testing, Spring Boot integration) — focus on real integration scenarios you've likely handled (JWT, CORS, file uploads).
3. **Week 3**: Sections H–L (Custom hooks + component patterns) — practice writing these from memory, not just reading them.
4. **Week 4**: Sections M–P (Async handling, JS fundamentals, Spring Boot coding, misc) — these are the ones most likely to appear as live coding rounds.

**Interview tip for 4 YOE**: Interviewers at this level expect you to justify *why* you'd choose one approach over another (e.g., Context vs Redux, useMemo vs premature optimization) and to relate frontend decisions to how your Spring Boot backend is designed (pagination shape, error format, auth flow). Practice narrating your coding answers out loud as you write them.
