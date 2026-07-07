# React.js Interview Questions & Answers (4 Years Experience — React + Spring Boot Full Stack)

A curated set of 100 questions covering React fundamentals, Hooks, performance, architecture, testing, and Spring Boot backend integration — tailored for a mid-senior (4 YOE) full-stack developer interview.

---

## Section 1: React Fundamentals (Q1–Q15)

**Q1. What is React and why is it used?**
React is a JavaScript library (not a full framework) for building user interfaces using a component-based architecture. It uses a virtual DOM to efficiently update the UI, supports declarative programming, and enables reusable UI components, making large applications easier to maintain.

**Q2. What is the Virtual DOM and how does it work?**
The Virtual DOM is an in-memory, lightweight copy of the real DOM. When state changes, React creates a new virtual DOM tree, diffs it against the previous one (reconciliation), and applies only the minimal set of changes to the real DOM via batched updates — avoiding expensive full re-renders.

**Q3. What is JSX?**
JSX is a syntax extension that lets you write HTML-like code inside JavaScript. It compiles down to `React.createElement()` calls via Babel. JSX is not mandatory but improves readability by co-locating markup and logic.

**Q4. What is the difference between an Element and a Component?**
A React Element is a plain object describing what to render (immutable, cheap to create). A Component is a function or class that returns elements — it's the reusable blueprint, while an element is a single instance/description of the UI at a point in time.

**Q5. What are functional vs class components?**
Functional components are plain JavaScript functions returning JSX, and with Hooks (since React 16.8) they can manage state and side effects. Class components extend `React.Component`, use `this.state`, and lifecycle methods. Modern React strongly favors functional components with Hooks — less boilerplate, easier composition, no `this` binding issues.

**Q6. What is the difference between props and state?**
Props are read-only inputs passed from a parent component to configure a child; they cannot be modified by the child. State is local, mutable data managed within a component that triggers re-render when updated via `setState`/`useState`.

**Q7. What is prop drilling and how do you avoid it?**
Prop drilling is passing props through multiple intermediate components that don't need them, just to reach a deeply nested child. It's avoided using Context API, state management libraries (Redux, Zustand), or component composition (passing children/render props).

**Q8. Explain the component lifecycle in React.**
Class components have three phases: Mounting (`constructor`, `render`, `componentDidMount`), Updating (`shouldComponentUpdate`, `render`, `componentDidUpdate`), and Unmounting (`componentWillUnmount`). In functional components, `useEffect` with different dependency arrays replicates these phases.

**Q9. What are controlled vs uncontrolled components?**
A controlled component's form value is driven by React state (`value` + `onChange`), giving full control and validation. An uncontrolled component stores its own state internally in the DOM and is accessed via `ref` (e.g., `inputRef.current.value`).

**Q10. What is the significance of `key` prop in lists?**
`key` gives React a stable identity for list items so it can efficiently determine which items changed, were added, or removed during reconciliation, instead of re-rendering the entire list. Keys should be unique and stable (avoid using array index if the list can reorder).

**Q11. What is React Fragment and why use it?**
`<React.Fragment>` (or `<>...</>`) lets you group multiple children without adding an extra wrapper DOM node, keeping the DOM tree clean and avoiding invalid HTML nesting (e.g., inside `<table>`).

**Q12. What is the difference between `React.memo`, `PureComponent`, and `shouldComponentUpdate`?**
All three prevent unnecessary re-renders. `PureComponent` (class) and `React.memo` (functional) do a shallow prop comparison automatically. `shouldComponentUpdate` is a manual lifecycle hook in class components giving full control over the comparison logic.

**Q13. What are synthetic events in React?**
SyntheticEvent is React's cross-browser wrapper around native DOM events, normalizing behavior across browsers. It pools events for performance (in legacy React) and is accessed the same way as native events (`e.target`, `e.preventDefault()`).

**Q14. What is the difference between `state` and `ref`?**
State updates trigger a re-render and are asynchronous/batched. Refs (`useRef`) let you hold mutable values or DOM references that persist across renders without causing a re-render when changed.

**Q15. What is reconciliation in React?**
Reconciliation is the diffing algorithm React uses to compare the new virtual DOM tree with the previous one and compute the minimal number of real DOM mutations needed. It assumes elements of different types produce different trees and uses keys to match elements within lists.

---

## Section 2: Hooks (Q16–Q30)

**Q16. What is `useState` and how does it work internally?**
`useState` lets functional components hold local state. It returns a state value and a setter function. Internally, React maintains a linked list of hooks per fiber node, tied to call order, which is why hooks can't be called conditionally.

**Q17. What is `useEffect` and how do you control when it runs?**
`useEffect` handles side effects (API calls, subscriptions, DOM manipulation) after render. Its second argument (dependency array) controls execution: no array = runs every render, empty array `[]` = runs once on mount, array with values = runs when those values change. The returned function is the cleanup, run before the next effect or on unmount.

**Q18. Difference between `useEffect` and `useLayoutEffect`?**
`useEffect` runs asynchronously after the browser paints, so it doesn't block visual updates. `useLayoutEffect` runs synchronously after DOM mutations but before the browser paints — used when you need to measure/mutate the DOM before the user sees a flicker (e.g., reading layout dimensions).

**Q19. What is `useMemo` and when should you use it?**
`useMemo` memoizes the result of an expensive computation, recalculating only when its dependencies change. Use it to avoid re-computing costly derived values (e.g., filtering/sorting large lists) on every render — but don't overuse it since memoization itself has overhead.

**Q20. What is `useCallback` and how does it differ from `useMemo`?**
`useCallback` memoizes a function reference so it doesn't get recreated on every render (useful when passing callbacks to memoized child components to prevent unnecessary re-renders). `useMemo` memoizes a computed value; `useCallback(fn, deps)` is essentially `useMemo(() => fn, deps)`.

**Q21. What is `useRef` used for?**
`useRef` returns a mutable object (`{ current: value }`) that persists across renders without causing re-renders when updated. Common uses: accessing DOM nodes directly, storing previous values, or holding timers/interval IDs.

**Q22. What is the Context API and `useContext`?**
Context provides a way to share values (theme, auth user, locale) across the component tree without prop drilling. `createContext` creates a context object, a `Provider` supplies the value, and `useContext(MyContext)` consumes it in any descendant component.

**Q23. What is `useReducer` and when would you prefer it over `useState`?**
`useReducer` manages complex state logic via a reducer function `(state, action) => newState`, similar to Redux. Prefer it when state transitions are complex, involve multiple sub-values, or the next state depends on the previous one in a structured way.

**Q24. What are custom hooks? Give an example.**
Custom hooks are reusable functions prefixed with `use` that encapsulate stateful logic using built-in hooks. Example: a `useFetch(url)` hook that manages `data`, `loading`, and `error` state internally, reused across multiple components calling different Spring Boot REST endpoints.

**Q25. What are the rules of hooks?**
1) Only call hooks at the top level (not inside loops, conditions, or nested functions). 2) Only call hooks from React function components or custom hooks. These rules ensure hooks are called in the same order on every render, which React relies on to associate state correctly.

**Q26. What is `useImperativeHandle`?**
It customizes the instance value exposed when using `ref` with `forwardRef`, letting a parent call specific imperative methods on a child (e.g., `inputRef.current.focus()`) while hiding internal DOM details.

**Q27. What is `useTransition` and `useDeferredValue` (Concurrent React)?**
Both help keep UIs responsive during expensive updates. `useTransition` marks state updates as non-urgent ("transitions"), letting urgent updates (typing) interrupt them. `useDeferredValue` defers re-rendering a value until more urgent renders complete — useful for search-as-you-type with large filtered lists.

**Q28. How do you fetch data using hooks, and how do you avoid race conditions?**
Use `useEffect` with an async function inside (since the effect callback itself can't be async), and a cleanup flag/AbortController to cancel or ignore stale responses:
```jsx
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/products/${id}`, { signal: controller.signal })
    .then(res => res.json())
    .then(setData)
    .catch(err => { if (err.name !== 'AbortError') setError(err); });
  return () => controller.abort();
}, [id]);
```

**Q29. Can you call hooks conditionally? Why or why not?**
No. React tracks hooks by call order using an internal array/linked list per component instance. Conditionally calling hooks would shift indices between renders, causing state to be associated with the wrong hook and leading to bugs or crashes.

**Q30. How do you share stateful logic between components without HOCs or render props?**
Custom hooks are the modern preferred approach — extract the reusable stateful logic into a `useXyz` function and call it in each component, avoiding the wrapper-hell of HOCs and the nesting of render props.

---

## Section 3: State Management (Q31–Q40)

**Q31. When would you choose Redux/Redux Toolkit over Context API?**
Context is fine for low-frequency updates (theme, auth). Redux is better for complex, frequently-changing global state, when you need time-travel debugging, middleware (logging, async flows), or predictable state updates across a large app with many components subscribing to slices of state — Context re-renders all consumers on any change, whereas Redux with selectors avoids that.

**Q32. What is Redux Toolkit (RTK) and why is it preferred over classic Redux?**
RTK is the official, opinionated way to write Redux logic. It reduces boilerplate via `createSlice` (combines actions + reducers), uses Immer internally for "mutable-looking" immutable updates, and includes `configureStore` with sensible defaults (DevTools, thunk middleware) built in.

**Q33. What is RTK Query and how have you used it with Spring Boot APIs?**
RTK Query is a data-fetching/caching layer built into Redux Toolkit. I define an `apiSlice` with a `baseUrl` pointing to the Spring Boot backend, declare endpoints (`getUsers`, `createOrder`) with automatic caching, re-fetching, and invalidation tags — eliminating manual `useEffect`/loading-state boilerplate for REST calls.

**Q34. What is the difference between local state, lifted state, and global state?**
Local state lives in one component (`useState`). Lifted state is moved up to the nearest common ancestor when siblings need to share it. Global state (Context, Redux, Zustand) is accessible app-wide, used for cross-cutting concerns like auth, cart, or user preferences.

**Q35. What is Zustand/Recoil and how do they compare to Redux?**
Zustand is a minimal state management library using hooks with no boilerplate (no actions/reducers required), storing state in a simple store outside React. Recoil introduces atoms and selectors for fine-grained, derived reactive state. Both are lighter-weight alternatives to Redux for many use cases, trading off some of Redux's structure/tooling ecosystem.

**Q36. How do you handle server state vs client state differently?**
Server state (data from Spring Boot APIs) is asynchronous, can go stale, and needs caching/refetching/invalidation — best handled by React Query or RTK Query. Client state (UI toggles, form inputs, modals) is synchronous and owned entirely by the frontend — handled with `useState`/Context/Redux.

**Q37. What is React Query (TanStack Query) and its core benefits?**
React Query manages server-state: caching, background refetching, deduplication of identical requests, retries, and pagination/infinite scroll support. It removes the need to manually manage loading/error/data state for every API call to the backend.

**Q38. How do you implement optimistic updates when calling a Spring Boot API?**
Update the UI immediately assuming success (e.g., add the new item to local/cache state), fire the API request, and roll back to the previous cached state if the request fails. RTK Query and React Query both provide built-in optimistic update patterns via `onMutate`/cache manipulation.

**Q39. How do you structure global state for a mid-large scale application?**
Split state by domain/feature (auth slice, cart slice, notifications slice) rather than one monolithic store. Keep server-state (API data) separate from UI state. Normalize nested/related entities (e.g., using `createEntityAdapter` in RTK) to avoid duplication and ease updates.

**Q40. What is state normalization and why is it important?**
Normalization stores related data in flat, keyed structures (like a database table) instead of deeply nested objects/arrays — e.g., `{ users: { byId: {...}, allIds: [...] } }`. It avoids data duplication, simplifies updates, and prevents unnecessary re-renders from deep object comparisons.

---

## Section 4: Performance Optimization (Q41–Q50)

**Q41. How do you identify performance bottlenecks in a React app?**
Use React DevTools Profiler to record renders and see which components re-render and why (highlighted with render duration). Also use Chrome Performance tab, Lighthouse, and `why-did-you-render` library to catch unnecessary re-renders.

**Q42. What causes unnecessary re-renders and how do you prevent them?**
Common causes: passing new object/array/function references as props on every render, missing `React.memo`, context value changing on every render, or state lifted too high. Fixes: `useMemo`/`useCallback` for stable references, `React.memo`, splitting context providers, and colocating state closer to where it's used.

**Q43. What is code-splitting and how do you implement it in React?**
Code-splitting breaks the bundle into smaller chunks loaded on demand, reducing initial load time. Implemented via dynamic `import()` combined with `React.lazy()` and `<Suspense>`:
```jsx
const Dashboard = React.lazy(() => import('./Dashboard'));
<Suspense fallback={<Spinner />}><Dashboard /></Suspense>
```

**Q44. What is windowing/virtualization and when do you use it?**
Virtualization (via `react-window` or `react-virtualized`) renders only the visible portion of a large list/table in the DOM, recycling DOM nodes as the user scrolls. Essential for rendering thousands of rows (e.g., a large transaction table from the backend) without freezing the UI.

**Q45. How does React batch state updates?**
React groups multiple `setState` calls within the same event handler/synchronous block into a single re-render for performance. Since React 18, automatic batching extends to updates inside promises, timeouts, and native event handlers too (not just React event handlers as in React 17).

**Q46. What is lazy loading of images/components and how do you implement it?**
Deferring the load of off-screen resources until needed — using `loading="lazy"` for images, or `React.lazy`/dynamic imports plus Intersection Observer for components, reducing initial payload and improving perceived performance.

**Q47. How do you optimize a large form with many controlled inputs?**
Avoid a single state object causing the whole form to re-render on every keystroke; use field-level state, libraries like `react-hook-form` (uncontrolled by default, minimal re-renders), or split the form into memoized sub-components.

**Q48. What is memoization and where can it hurt rather than help?**
Memoization caches results to avoid recomputation. It hurts when the computation is cheap (the comparison overhead exceeds the recomputation cost) or when dependencies change on nearly every render anyway, making the cache ineffective while still adding overhead.

**Q49. How do you reduce bundle size in a production React app?**
Tree-shaking (ES modules), code-splitting by route, analyzing bundles with `webpack-bundle-analyzer`, avoiding heavy libraries (e.g., moment.js → date-fns), lazy-loading rarely used features, and enabling gzip/Brotli compression on the server.

**Q50. How do you debounce/throttle API calls triggered by user input (e.g., search-as-you-type against a Spring Boot search endpoint)?**
Debounce the input using `lodash.debounce` or a custom hook so the API call fires only after the user pauses typing (e.g., 300ms), reducing unnecessary requests to the backend and improving both UX and server load.

---

## Section 5: React Router (Q51–Q55)

**Q51. How does client-side routing work in a React SPA?**
React Router intercepts navigation, manipulating the browser's History API (`pushState`/`popState`) instead of triggering full page reloads, and renders matching components based on the current URL — giving an app-like navigation experience.

**Q52. How do you implement protected/private routes with JWT auth from Spring Boot?**
Create a wrapper component that checks authentication state (e.g., a valid JWT in memory/context, validated against `/api/auth/me`), and either renders the child route or redirects to `/login` using `<Navigate />`.
```jsx
function PrivateRoute({ children }) {
  const { isAuthenticated } = useAuth();
  return isAuthenticated ? children : <Navigate to="/login" />;
}
```

**Q53. What is the difference between `BrowserRouter` and `HashRouter`?**
`BrowserRouter` uses the HTML5 History API for clean URLs (requires server-side configuration to serve `index.html` for all routes). `HashRouter` uses the URL hash (`#`) so routing is handled entirely client-side without server configuration — useful for static hosting without rewrite rules.

**Q54. How do you handle nested routes and layouts?**
Define parent routes with an `<Outlet />` placeholder where child routes render, allowing shared layouts (nav bars, sidebars) while only the nested content changes based on the URL.

**Q55. How do you implement route-based code splitting?**
Wrap each route's component in `React.lazy()` combined with `<Suspense>` at the router level, so each page's bundle is only downloaded when the user navigates to it.

---

## Section 6: Testing (Q56–Q60)

**Q56. What testing libraries have you used with React?**
Jest as the test runner/assertion library, React Testing Library (RTL) for component testing focused on user behavior rather than implementation details, and Cypress/Playwright for end-to-end tests covering full user flows including real API calls to Spring Boot.

**Q57. What's the philosophy behind React Testing Library?**
"Test software the way users use it" — query elements by role/label/text rather than internal component state or implementation details, making tests resilient to refactors.

**Q58. How do you mock API calls in tests?**
Use `msw` (Mock Service Worker) to intercept network requests at the network level, or Jest mocks (`jest.mock('axios')`) to simulate Spring Boot API responses without hitting a real backend, keeping tests fast and deterministic.

**Q59. How do you test a component that uses `useEffect` to fetch data?**
Render the component wrapped in `act()` (RTL does this automatically), mock the fetch/axios call, then use `findBy*` queries (which wait for async updates) or `waitFor` to assert the UI updates once data resolves.

**Q60. What is snapshot testing and what are its pitfalls?**
Snapshot testing captures a rendered component's output and compares it against a saved reference on subsequent runs. Pitfalls: snapshots can become large/brittle, easily "updated" blindly by developers without real review, and don't test actual behavior — best used sparingly for stable, simple UI.

---

## Section 7: Advanced Patterns & Architecture (Q61–Q70)

**Q61. What are Higher-Order Components (HOCs)?**
A HOC is a function that takes a component and returns a new enhanced component, used to share cross-cutting logic (e.g., `withAuth(Component)` injecting user data). Largely superseded by custom hooks for cleaner composition.

**Q62. What is the render props pattern?**
A pattern where a component takes a function as a prop (often `children`) that it calls with internal state/data, letting the consumer decide how to render it. Less common now that hooks solve most of the same problems more simply.

**Q63. What is the Compound Component pattern?**
A pattern where multiple components work together sharing implicit state via Context, giving a flexible, declarative API — e.g., `<Tabs><Tabs.List><Tabs.Tab>...` — similar to how native `<select>`/`<option>` work together.

**Q64. What is the Container/Presentational component pattern?**
Separates logic (data fetching, state) into "container" components from pure UI rendering in "presentational" components. With hooks, this is now often achieved by extracting logic into custom hooks instead of separate container components.

**Q65. How do you structure a large React project (folder architecture)?**
Feature-based (a.k.a. "vertical slice") structure is preferred over type-based: group by feature/domain (`/features/orders/{components, hooks, api, slice}`) rather than by file type (`/components`, `/reducers`), which scales better as the app grows and keeps related backend-integration code together.

**Q66. What is error boundary and how do you implement one?**
An Error Boundary is a class component implementing `static getDerivedStateFromError()` and/or `componentDidCatch()` to catch JS errors in its child tree during render, log them, and show a fallback UI instead of crashing the whole app. (Functional components can't yet be error boundaries natively.)

**Q67. What is `Suspense` used for beyond lazy loading?**
Suspense lets components "wait" for something (data, code) before rendering, showing a fallback meanwhile. It's being extended (React 18+) to support data fetching frameworks (like Relay, or React Query's suspense mode) so components can suspend while awaiting API responses.

**Q68. How do you handle global error/loading states across many API-driven components?**
Centralize via an API client interceptor (Axios interceptors) that catches errors globally (e.g., 401 → redirect to login, 500 → toast notification), combined with a shared loading/error UI pattern (spinners, skeleton loaders) driven by React Query/RTK Query status flags.

**Q69. What is micro-frontend architecture and have you worked with it?**
Micro-frontends split a large frontend into independently deployable pieces owned by different teams (e.g., via Module Federation in Webpack), each potentially talking to different Spring Boot microservices — useful for large organizations wanting independent release cycles.

**Q70. How do you handle feature flags in React?**
Fetch flag configuration from a backend endpoint or a service (LaunchDarkly, Unleash) on app load, store in Context, and conditionally render features: `{flags.newCheckout ? <NewCheckout /> : <OldCheckout />}` — allowing gradual rollouts without redeploying.

---

## Section 8: React + Spring Boot Integration (Q71–Q85)

**Q71. How did you integrate a React frontend with a Spring Boot backend in your projects?**
I built the React app as a separate SPA (via Create React App/Vite) consuming REST APIs exposed by Spring Boot. In development, I proxied API calls (`"proxy"` in `package.json` or Vite's `server.proxy`) to the Spring Boot server (e.g., `localhost:8080`) to avoid CORS issues. In production, the React build (`npm run build`) was either served as static resources from Spring Boot's `src/main/resources/static` folder, or deployed separately (Nginx/S3/CDN) and configured to call the API via an environment-specific base URL, with Spring Boot exposing `@RestController` endpoints returning JSON (using Jackson serialization).

**Q72. How do you handle CORS issues between React (localhost:3000) and Spring Boot (localhost:8080)?**
On the Spring Boot side, I configure CORS globally via a `WebMvcConfigurer` bean (`addCorsMappings`) or per-controller with `@CrossOrigin(origins = "http://localhost:3000")`, specifying allowed methods, headers, and credentials. For production, I restrict `allowedOrigins` to the actual deployed frontend domain rather than using a wildcard, especially when credentials/cookies are involved.

**Q73. How do you structure API calls from React to Spring Boot (Axios vs Fetch)?**
I typically create a centralized Axios instance with a `baseURL` (from environment variables like `REACT_APP_API_URL`), request/response interceptors for attaching JWT tokens and handling global errors, and organize API calls into service modules per domain (`userService.js`, `orderService.js`) that map directly to Spring Boot's `@RestController` endpoints.

**Q74. How do you handle authentication between React and Spring Boot (JWT flow)?**
On login, Spring Boot (using Spring Security) validates credentials and returns a JWT access token (and often a refresh token). React stores the access token in memory (or a secure httpOnly cookie set by the backend, preferably over localStorage to reduce XSS risk), attaches it as `Authorization: Bearer <token>` via an Axios interceptor on every request, and Spring Security's `OncePerRequestFilter`/JWT filter validates it on each incoming request before reaching the controller.

**Q75. How do you handle token refresh when the JWT expires?**
An Axios response interceptor catches 401 responses, calls a `/api/auth/refresh` endpoint (using the refresh token) to get a new access token, retries the original failed request, and if refresh also fails, logs the user out and redirects to `/login`. Care is taken to queue concurrent failed requests during the refresh so multiple simultaneous refresh calls aren't fired.

**Q76. How do you handle validation errors returned from a Spring Boot backend (e.g., `@Valid` DTO validation failures)?**
Spring Boot returns a structured error response (often via a `@ControllerAdvice`/`@ExceptionHandler` for `MethodArgumentNotValidException`) with field-level messages and a 400 status. In React, I parse this structured error object in the API call's catch block and map field errors back to the corresponding form fields (e.g., using `react-hook-form`'s `setError`) so users see inline validation messages matching backend rules.

**Q77. How do you handle pagination when Spring Boot uses `Pageable`/`Page<T>`?**
Spring Data's `Page<T>` response includes `content`, `totalElements`, `totalPages`, and `number`. React sends `page` and `size` (and `sort`) as query params, and the frontend table/list component reads `totalPages`/`totalElements` from the response to render pagination controls, often integrated with React Query for caching each page.

**Q78. How do you handle file uploads from React to a Spring Boot endpoint?**
Use `FormData` in React to append the file and any metadata, send via Axios with `Content-Type: multipart/form-data` (Axios sets this automatically for FormData), and Spring Boot receives it via `@RequestParam("file") MultipartFile file` in the controller, often showing upload progress in React using Axios's `onUploadProgress` callback.

**Q79. How do you handle real-time updates (e.g., notifications) between React and Spring Boot?**
Using WebSockets — Spring Boot exposes a STOMP endpoint via `spring-boot-starter-websocket` (`@EnableWebSocketMessageBroker`), and React connects using `sockjs-client` + `@stomp/stompjs`, subscribing to topics (e.g., `/topic/notifications`) and updating component state as messages arrive. For simpler one-way updates, Server-Sent Events (SSE) via `SseEmitter` on the backend and `EventSource` in React is a lighter alternative.

**Q80. How do you keep DTOs/response shapes in sync between Spring Boot and React (avoiding type mismatches)?**
Options include: generating TypeScript types from OpenAPI/Swagger specs that Spring Boot auto-generates (`springdoc-openapi`) using tools like `openapi-typescript-codegen`, or manually maintaining TypeScript interfaces mirroring backend DTOs, reviewed during PRs whenever a DTO changes on the backend.

**Q81. How do you handle global error responses (500s, business exceptions) consistently across the app?**
Standardize the Spring Boot error contract via a `@ControllerAdvice` returning a consistent shape (e.g., `{ timestamp, status, error, message, path }`), then handle it uniformly in React via an Axios response interceptor that maps status codes to UI behavior — toast for 4xx business errors, global error page for 5xx, redirect-to-login for 401/403.

**Q82. How do you test the integration between React and Spring Boot end-to-end?**
Backend: Spring Boot integration tests using `@SpringBootTest` + `MockMvc`/`WebTestClient` to verify controller contracts. Frontend-to-backend: Cypress/Playwright E2E tests running against a real (or Dockerized/Testcontainers-backed) Spring Boot instance, verifying full user flows like login → create order → see it reflected in the UI.

**Q83. How do you handle environment-specific API URLs (dev/staging/prod) in React talking to different Spring Boot deployments?**
Use `.env` files (`.env.development`, `.env.production`) with variables like `REACT_APP_API_BASE_URL` (or Vite's `VITE_API_BASE_URL`), injected at build time, so the same codebase points to different Spring Boot backend hosts per environment without code changes.

**Q84. How would you design the React-Spring Boot communication for a form with dependent dropdowns (e.g., State → City) backed by REST endpoints?**
The State dropdown fetches from `/api/states` on mount; on selecting a state, a `useEffect` (keyed on the selected state ID) fetches `/api/cities?stateId=X` from Spring Boot and populates the City dropdown, with loading states shown and prior city selection cleared/reset when the state changes to avoid stale/mismatched data.

**Q85. What security considerations do you keep in mind when integrating React with a Spring Boot backend?**
Never trust client-side validation alone (Spring Boot must re-validate everything server-side); avoid storing JWTs in `localStorage` where possible (XSS risk) in favor of httpOnly cookies with `SameSite`/CSRF protection; sanitize any user-generated content rendered in React to avoid XSS; ensure Spring Security enforces authorization (`@PreAuthorize`) server-side rather than just hiding UI elements; and always use HTTPS in production.

---

## Section 9: Security (Q86–Q90)

**Q86. How do you prevent XSS attacks in React?**
React escapes values embedded in JSX by default, preventing most injection. Risk arises when using `dangerouslySetInnerHTML` — any HTML rendered this way must be sanitized (e.g., with `DOMPurify`) since React does not escape it.

**Q87. What is CSRF and how do you protect against it when using cookie-based auth with Spring Boot?**
CSRF tricks a logged-in user's browser into making unwanted requests. When using cookie-based sessions/JWTs, Spring Security's CSRF protection (double-submit token pattern) requires the frontend to read a CSRF token (from a cookie or a dedicated endpoint) and send it back in a custom header on state-changing requests.

**Q88. How do you securely store authentication tokens in a React app?**
Prefer httpOnly, Secure, SameSite cookies set by Spring Boot (inaccessible to JS, mitigating XSS token theft) over `localStorage`/`sessionStorage`. If tokens must be stored client-side (e.g., for a mobile-like SPA), keep them in memory (a variable/Context) rather than persistent storage, accepting the trade-off of needing re-login on refresh, or pair with a short-lived access token + secure refresh token pattern.

**Q89. How do you handle role-based UI rendering (e.g., admin-only buttons) while keeping it secure?**
Conditionally render UI elements based on roles/permissions decoded from the JWT or fetched from `/api/auth/me`, but always enforce the same authorization server-side via Spring Security (`@PreAuthorize("hasRole('ADMIN')")`) since hiding a button client-side is only a UX convenience, not real security.

**Q90. What is Content Security Policy (CSP) and how does it help a React app?**
CSP is an HTTP header (often set by the backend/reverse proxy) restricting which sources scripts, styles, and other resources can load from, significantly reducing the impact of XSS by blocking unauthorized inline scripts or third-party script injection.

---

## Section 10: Ecosystem, Tooling & Miscellaneous (Q91–Q100)

**Q91. What is the difference between Create React App, Vite, and Next.js?**
CRA is a webpack-based zero-config bundler/scaffold for SPAs (now largely deprecated in favor of Vite). Vite offers much faster dev-server startup and HMR using native ES modules and esbuild/Rollup. Next.js is a full framework on top of React adding file-based routing, SSR/SSG, and API routes — used when SEO or server rendering is needed, beyond a pure SPA talking to Spring Boot.

**Q92. What is Server-Side Rendering (SSR) and would you use it with a Spring Boot backend?**
SSR renders React components to HTML on the server before sending to the client, improving SEO and initial paint time. It can be used alongside Spring Boot (Spring Boot serves as the pure API layer, while a Node.js-based Next.js server handles SSR and calls Spring Boot's REST APIs) — the two are typically deployed as separate services.

**Q93. What is Hydration in the context of SSR?**
Hydration is the process where React attaches event listeners and internal state to server-rendered static HTML on the client, "activating" it into a fully interactive React app without re-rendering the DOM from scratch.

**Q94. What is TypeScript's benefit in a React + Spring Boot project?**
TypeScript catches type mismatches at compile time — especially valuable at the API boundary, where frontend interfaces can mirror Spring Boot DTOs, catching bugs like a renamed backend field before they reach production, and improving IDE autocomplete/refactoring safety across a large codebase.

**Q95. How do you handle environment configuration and secrets in a React app (given everything shipped to the browser is public)?**
Only non-sensitive, build-time config (API base URLs, feature flags) belongs in React's `.env` files, since anything bundled is visible to users. True secrets (DB credentials, third-party API keys) must stay server-side in Spring Boot and never be exposed to the frontend bundle.

**Q96. What is the purpose of a `.env` file and its limitations in Create React App/Vite?**
`.env` files inject build-time environment variables (must be prefixed `REACT_APP_` or `VITE_` to be exposed to client code). Limitation: values are baked into the static bundle at build time, so switching environments requires a rebuild (or runtime config injection via a small `config.js` fetched at startup for true "build once, deploy anywhere" setups).

**Q97. How do you set up CI/CD for a React app that consumes a Spring Boot API?**
A typical pipeline (GitHub Actions/Jenkins/GitLab CI): lint + unit test (Jest) + build the React app, run E2E tests against a deployed/Dockerized Spring Boot backend (using Testcontainers or a staging environment), then deploy the static build to a CDN/S3/Nginx (or into Spring Boot's static resources) while Spring Boot deploys independently as a Docker image/JAR.

**Q98. How do you monitor and debug production issues in a React app?**
Integrate error tracking (Sentry/LogRocket) to capture uncaught exceptions and component stack traces, use source maps for readable stack traces, correlate frontend errors with backend logs via a shared request/correlation ID passed in headers to Spring Boot, and use browser performance tools (Web Vitals) to track real-user metrics.

**Q99. What are Web Vitals and why do they matter?**
Core Web Vitals (LCP, FID/INP, CLS) measure real-world loading performance, interactivity, and visual stability. They matter for both user experience and SEO ranking; tools like `web-vitals` npm package let you measure and report these metrics from a production React app.

**Q100. How do you approach a full-stack feature end-to-end (React + Spring Boot), from requirement to deployment?**
1) Clarify requirements and define the API contract (request/response DTOs) collaboratively with backend. 2) Implement/verify the Spring Boot endpoint (with validation, tests). 3) Build the React feature: API service layer, hooks/RTK Query endpoint, UI components, form validation mirroring backend rules, loading/error states. 4) Write unit tests (Jest/RTL) and integration/E2E tests covering the real flow. 5) Handle edge cases (auth, empty states, pagination, errors). 6) Code review, then deploy both via CI/CD, verifying in staging before production rollout, often behind a feature flag for safer release.

---

*Good luck with your interview! Focus especially on Sections 2, 4, and 8 — Hooks, Performance, and Spring Boot integration are the most commonly probed areas for a 4-year full-stack React profile.*
