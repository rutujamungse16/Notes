# React.js Complete Interview Reference Guide 2026
## For Senior-Level Technical Interviews at Top MNCs (Google, Amazon, Microsoft, Flipkart, Uber)

---

# Table of Contents

1. [React Core Architecture & Internals](#1-react-core-architecture--internals)
2. [Virtual DOM & Reconciliation Algorithm](#2-virtual-dom--reconciliation-algorithm)
3. [React Fiber Architecture](#3-react-fiber-architecture)
4. [Component Lifecycle & Rendering](#4-component-lifecycle--rendering)
5. [Hooks Deep Dive](#5-hooks-deep-dive)
6. [State Management Patterns](#6-state-management-patterns)
7. [Context API & Dependency Injection](#7-context-api--dependency-injection)
8. [Performance Optimization](#8-performance-optimization)
9. [Concurrent React & Suspense](#9-concurrent-react--suspense)
10. [Server Components & SSR](#10-server-components--ssr)
11. [Error Boundaries & Error Handling](#11-error-boundaries--error-handling)
12. [React Patterns & Anti-Patterns](#12-react-patterns--anti-patterns)
13. [Testing Strategies](#13-testing-strategies)
14. [Security Best Practices](#14-security-best-practices)
15. [React 18/19 Features](#15-react-1819-features)
16. [Interview Questions Bank](#16-interview-questions-bank)
17. [Cheat Sheet](#17-cheat-sheet)

---

# 1. React Core Architecture & Internals

## 1.1 What is React Under the Hood?

React is a **declarative, component-based JavaScript library** for building user interfaces. At its core, React is:

1. **A Scheduling Library**: Manages when and how UI updates happen
2. **A Diffing Engine**: Computes minimal changes needed to update DOM
3. **A Component Model**: Encapsulates UI + behavior + state

### Core Packages Structure

```
react/
├── react                 # Core API (createElement, hooks, Component)
├── react-dom            # DOM-specific rendering (web)
├── react-dom/client     # Client-side rendering APIs (React 18+)
├── react-dom/server     # Server-side rendering APIs
├── react-reconciler     # The Fiber reconciler (shared)
└── scheduler            # Priority-based task scheduling
```

### How React.createElement Works

```javascript
// JSX
<div className="container">
  <h1>Hello</h1>
  <Button onClick={handleClick}>Click</Button>
</div>

// Compiles to (via Babel/SWC)
React.createElement(
  'div',
  { className: 'container' },
  React.createElement('h1', null, 'Hello'),
  React.createElement(Button, { onClick: handleClick }, 'Click')
);

// Returns a React Element (Plain Object)
{
  $$typeof: Symbol(react.element),  // Security: prevents JSON injection
  type: 'div',                       // String for DOM, Function for components
  key: null,
  ref: null,
  props: {
    className: 'container',
    children: [/* nested elements */]
  },
  _owner: null,                      // Fiber that created this element
}
```

### The $$typeof Symbol - Security Deep Dive

```javascript
// Why Symbol? Prevents XSS via JSON injection
// JSON.parse cannot create Symbols

// Attack scenario WITHOUT $$typeof Symbol:
const maliciousJSON = '{"type":"div","props":{"dangerouslySetInnerHTML":{"__html":"<script>evil()</script>"}}}';
// If React accepted plain objects, this could execute

// With Symbol(react.element), JSON.parse returns undefined for $$typeof
// React rejects the object as invalid element
```

## 1.2 React Element vs Component vs Instance vs Fiber

| Concept | What It Is | Lifespan | Memory |
|---------|-----------|----------|--------|
| **Element** | Plain JS object describing UI tree | Created each render, then discarded | Lightweight (~100 bytes) |
| **Component** | Function or Class that returns Elements | Exists in code, never instantiated directly | N/A |
| **Instance** | Class component's `this` context | Lives between mount/unmount | Has state, refs |
| **Fiber** | Internal work unit with scheduling info | Persists across renders | ~1KB per node |

```javascript
// Element - Immutable description
const element = <Button color="blue" />;

// Component - Blueprint/Template
function Button({ color, children }) {
  return <button className={color}>{children}</button>;
}

// Instance - Only for class components
class Button extends React.Component {
  state = { clicked: false };  // Instance property
  render() { return <button />; }
}

// Fiber - Internal (you don't create these)
// React creates Fibers to track component state, effects, priorities
```

## 1.3 Render vs Commit Phase

```
┌─────────────────────────────────────────────────────────────────┐
│                        RENDER PHASE                              │
│  (Pure, can be paused/aborted, no side effects)                 │
├─────────────────────────────────────────────────────────────────┤
│  1. Begin Work: Process component, call render/function         │
│  2. Reconciliation: Diff new vs old elements                    │
│  3. Create/Update Fiber nodes                                   │
│  4. Calculate effects (mutations, layouts, passive)             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        COMMIT PHASE                              │
│  (Synchronous, cannot be interrupted, has side effects)        │
├─────────────────────────────────────────────────────────────────┤
│  1. Before Mutation: getSnapshotBeforeUpdate                    │
│  2. Mutation: Apply DOM changes                                 │
│  3. Layout Effects: useLayoutEffect, componentDidMount/Update   │
│  4. Passive Effects: useEffect (async, after paint)            │
└─────────────────────────────────────────────────────────────────┘
```

### Key Interview Point: Render Phase is NOT DOM Manipulation

```javascript
function Component() {
  console.log('Render phase'); // Called during render
  
  useEffect(() => {
    console.log('Passive effect - after paint');
  });
  
  useLayoutEffect(() => {
    console.log('Layout effect - before paint, sync');
  });
  
  return <div />;
}

// Output order:
// 1. "Render phase"
// 2. "Layout effect - before paint, sync"
// 3. Browser paints
// 4. "Passive effect - after paint"
```

## 1.4 Interview Questions: Core Architecture

**Q1: Why does React use a Virtual DOM instead of directly manipulating the real DOM?**

**A:** Three primary reasons:
1. **Batching**: Accumulate multiple changes, apply once (minimize reflows/repaints)
2. **Declarative Model**: Describe final state, React computes transitions
3. **Cross-Platform**: Same reconciliation logic works for DOM, Native, Canvas, etc.

**Misconception to Address**: Virtual DOM is NOT faster than manual DOM manipulation. It's an abstraction that makes declarative programming possible while maintaining acceptable performance.

**Q2: What happens when you call setState or a state setter?**

```javascript
// Simplified flow:
setState(newValue)
  → scheduleUpdateOnFiber(fiber, lane)
    → markUpdateLaneFromFiberToRoot(fiber, lane)
      → ensureRootIsScheduled(root)
        → scheduleMicrotask(flushSyncWorkOnAllRoots) // or scheduler
          → performConcurrentWorkOnRoot(root)
            → renderRootConcurrent()
              → workLoopConcurrent()
                → performUnitOfWork() // Process each fiber
                  → beginWork() → reconcileChildren()
                  → completeWork() // Prepare DOM mutations
            → commitRoot()
              → commitMutationEffects() // Apply DOM changes
              → commitLayoutEffects() // useLayoutEffect
              // Browser paints here
              → flushPassiveEffects() // useEffect
```

**Q3: Explain the difference between controlled and uncontrolled components internally.**

| Aspect | Controlled | Uncontrolled |
|--------|-----------|--------------|
| Source of Truth | React state | DOM itself |
| Value Access | Via state variable | Via ref.current.value |
| Updates | setState on every change | Read when needed |
| Validation | Immediate (on each keystroke) | On submit/blur |
| Performance | More re-renders | Fewer re-renders |
| Form Libraries | React Hook Form (perf mode) | React Hook Form (default) |

```javascript
// Controlled - React owns the value
function Controlled() {
  const [value, setValue] = useState('');
  return <input value={value} onChange={e => setValue(e.target.value)} />;
}
// Every keystroke: setState → re-render → React updates DOM

// Uncontrolled - DOM owns the value
function Uncontrolled() {
  const inputRef = useRef();
  const handleSubmit = () => console.log(inputRef.current.value);
  return <input ref={inputRef} defaultValue="" />;
}
// No re-renders on typing, read value imperatively
```

---

# 2. Virtual DOM & Reconciliation Algorithm

## 2.1 What is the Virtual DOM?

The Virtual DOM is a **lightweight JavaScript representation** of the actual DOM. It's essentially a tree of React Elements (plain objects).

```javascript
// Virtual DOM node structure
{
  type: 'div',
  props: {
    className: 'container',
    children: [
      { type: 'h1', props: { children: 'Title' } },
      { type: 'p', props: { children: 'Content' } }
    ]
  }
}

// Corresponding Real DOM
<div class="container">
  <h1>Title</h1>
  <p>Content</p>
</div>
```

## 2.2 Reconciliation Algorithm (Diffing)

React's diffing algorithm operates on two key heuristics:

### Heuristic 1: Different Types = Complete Rebuild

```javascript
// Before
<div>
  <Counter />
</div>

// After
<span>
  <Counter />
</span>

// Result: div tree destroyed, span tree built from scratch
// Counter state is LOST (different parent type)
```

### Heuristic 2: Keys for List Optimization

```javascript
// Without keys (O(n) insertions can become O(n²))
['a', 'b', 'c'].map(item => <li>{item}</li>)

// With keys (O(n) always)
['a', 'b', 'c'].map(item => <li key={item}>{item}</li>)
```

### The Diffing Process

```
┌─────────────────────────────────────────────────────────┐
│ Step 1: Compare Root Elements                           │
├─────────────────────────────────────────────────────────┤
│ Same type? → Update props, recurse into children        │
│ Different type? → Unmount old tree, mount new tree      │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│ Step 2: Compare Children                                │
├─────────────────────────────────────────────────────────┤
│ No keys? → Compare by index (positional)                │
│ Has keys? → Match by key, minimize moves                │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│ Step 3: Generate Effect List                            │
├─────────────────────────────────────────────────────────┤
│ Placement: New elements to insert                       │
│ Update: Changed props/children                          │
│ Deletion: Elements to remove                            │
└─────────────────────────────────────────────────────────┘
```

## 2.3 Key Prop Deep Dive

### Why Index as Key is Dangerous

```javascript
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        // BAD: Using index as key
        <TodoItem key={index} todo={todo} />
      ))}
    </ul>
  );
}

// Scenario: Delete first item from [A, B, C]
// Before: index 0=A, index 1=B, index 2=C
// After:  index 0=B, index 1=C

// React sees: key 0 changed (A→B), key 1 changed (B→C), key 2 deleted
// Reality: We wanted to delete A only
// Result: B and C are unnecessarily re-rendered with wrong state!
```

### Correct Key Usage

```javascript
// GOOD: Stable, unique identifiers
todos.map(todo => <TodoItem key={todo.id} todo={todo} />)

// ACCEPTABLE: Composite keys when no ID exists
items.map(item => <Item key={`${item.category}-${item.name}`} item={item} />)

// ACCEPTABLE: Index ONLY if list is static, never reordered
staticOptions.map((opt, i) => <Option key={i} value={opt} />)
```

### Key-Based State Reset Pattern

```javascript
// Force component remount by changing key
function ProfilePage({ userId }) {
  // When userId changes, entire Profile remounts with fresh state
  return <Profile key={userId} userId={userId} />;
}

// Alternative: Reset state in useEffect (less clean)
function Profile({ userId }) {
  const [data, setData] = useState(null);
  useEffect(() => {
    setData(null);  // Reset on userId change
    fetchProfile(userId).then(setData);
  }, [userId]);
}
```

## 2.4 Reconciliation Complexity

| Operation | Without Keys | With Keys |
|-----------|-------------|-----------|
| Append to end | O(1) | O(1) |
| Prepend to beginning | O(n) | O(1) |
| Insert in middle | O(n) | O(1) |
| Delete from anywhere | O(n) | O(1) |
| Reorder | O(n²) worst case | O(n) |

## 2.5 Interview Questions: Reconciliation

**Q1: What are the two heuristics React uses for O(n) reconciliation?**

**A:**
1. **Type Heuristic**: Elements of different types produce different trees. If a `<div>` becomes a `<span>`, the entire subtree is rebuilt.
2. **Key Heuristic**: Developer-provided keys identify which elements are stable across renders, enabling efficient reordering.

**Q2: Why shouldn't you use Math.random() as a key?**

```javascript
// NEVER DO THIS
items.map(item => <Item key={Math.random()} data={item} />)

// Problems:
// 1. New key every render → component always remounts
// 2. State is lost on every parent re-render
// 3. Performance disaster (no reconciliation benefit)
// 4. Focus, animations, form inputs all break
```

**Q3: Explain what happens internally when a component's parent type changes.**

```javascript
// Before
<div>
  <Form />
</div>

// After
<section>
  <Form />
</section>
```

**Answer:**
1. React compares root element types: `div` ≠ `section`
2. Entire `<div>` subtree is scheduled for unmount
3. `Form` component's `componentWillUnmount` / cleanup effects run
4. `Form` instance and state are destroyed
5. New `<section>` subtree is mounted from scratch
6. `Form` mounts as a new instance with initial state

---

# 3. React Fiber Architecture

## 3.1 What is Fiber?

Fiber is React's **reimplementation of the core reconciliation algorithm** (introduced in React 16). It enables:

1. **Incremental Rendering**: Split work into chunks
2. **Prioritization**: Handle urgent updates first
3. **Pause/Resume**: Stop work to handle higher priority tasks
4. **Concurrency**: Work on multiple state updates simultaneously

### Pre-Fiber (Stack Reconciler) Problems

```javascript
// Long synchronous update blocks the main thread
function BigList({ items }) {
  return (
    <ul>
      {items.map(item => <ComplexItem key={item.id} data={item} />)}
    </ul>
  );
}
// With 10,000 items: Main thread blocked for 100ms+
// User input/animations freeze
```

## 3.2 Fiber Node Structure

Each component/element has a corresponding Fiber node:

```javascript
// Simplified Fiber structure
{
  // Instance
  tag: FunctionComponent | ClassComponent | HostComponent | ...,
  type: function | class | string,  // 'div', Button function, etc.
  stateNode: DOM node | class instance | null,
  
  // Fiber Tree Links (linked list, not tree)
  return: Fiber | null,       // Parent
  child: Fiber | null,        // First child
  sibling: Fiber | null,      // Next sibling
  index: number,              // Position among siblings
  
  // Props & State
  pendingProps: Object,       // New props for this render
  memoizedProps: Object,      // Props from last render
  memoizedState: Object,      // State from last render (hooks linked list)
  
  // Effects
  flags: Flags,               // Placement | Update | Deletion | ...
  subtreeFlags: Flags,        // Aggregated child flags (optimization)
  deletions: Array<Fiber>,    // Children to delete
  
  // Scheduling
  lanes: Lanes,               // Priority lanes for this fiber
  childLanes: Lanes,          // Priority lanes in subtree
  
  // Alternate (Double Buffering)
  alternate: Fiber | null,    // Points to the other tree
}
```

## 3.3 Double Buffering (Current vs Work-in-Progress)

```
┌─────────────────────────────────────────────────────────────────┐
│                      FIBER ROOT                                  │
│                          │                                       │
│        ┌─────────────────┴─────────────────┐                    │
│        │                                    │                    │
│        ▼                                    ▼                    │
│  ┌──────────────┐                    ┌──────────────┐           │
│  │   CURRENT    │   ←── alternate ──→│    WORK IN   │           │
│  │    TREE      │                    │   PROGRESS   │           │
│  │  (on screen) │                    │    TREE      │           │
│  └──────────────┘                    └──────────────┘           │
│                                                                  │
│  On commit: root.current = workInProgress                       │
│  Previous current becomes next workInProgress                   │
└─────────────────────────────────────────────────────────────────┘
```

```javascript
// Accessing current fiber (internal, but useful to understand)
function Component() {
  // React.currentOwner points to current rendering fiber
  // This is how hooks know which component they belong to
}

// After commit, trees swap
root.current = finishedWork;
// Old current tree becomes WIP for next render (reused)
```

## 3.4 Fiber Traversal Algorithm

Fiber uses a **while loop with linked list** (not recursion):

```javascript
// Simplified workLoop
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}

function performUnitOfWork(unitOfWork) {
  const current = unitOfWork.alternate;
  
  // Phase 1: "Begin" - Process this fiber
  const next = beginWork(current, unitOfWork, renderLanes);
  unitOfWork.memoizedProps = unitOfWork.pendingProps;
  
  if (next === null) {
    // No children, complete this fiber
    completeUnitOfWork(unitOfWork);
  } else {
    // Has children, process first child next
    workInProgress = next;
  }
}

function completeUnitOfWork(unitOfWork) {
  let completedWork = unitOfWork;
  
  do {
    // Phase 2: "Complete" - Finalize this fiber
    completeWork(completedWork);
    
    const siblingFiber = completedWork.sibling;
    if (siblingFiber !== null) {
      // Process sibling
      workInProgress = siblingFiber;
      return;
    }
    // No sibling, complete parent
    completedWork = completedWork.return;
    workInProgress = completedWork;
  } while (completedWork !== null);
}
```

### Traversal Visualization

```
        App
       / │ \
      A  B  C
     /|     |
    D E     F

Traversal Order:
Begin: App → A → D → (complete D) → E → (complete E) → (complete A) 
       → B → (complete B) → C → F → (complete F) → (complete C) 
       → (complete App)

Like depth-first with sibling processing
```

## 3.5 Lanes & Priority System

React 18 replaced `expirationTime` with `lanes` (bitfield-based priority):

```javascript
// Lane values (simplified)
const SyncLane             = 0b0000000000000000000000000000001;  // Highest
const InputContinuousLane  = 0b0000000000000000000000000000100;
const DefaultLane          = 0b0000000000000000000000000010000;
const TransitionLane1      = 0b0000000000000000000001000000000;
const IdleLane             = 0b0100000000000000000000000000000;  // Lowest
const OffscreenLane        = 0b1000000000000000000000000000000;

// Multiple lanes can be worked on together
const renderLanes = SyncLane | InputContinuousLane;

// Check if update should be processed
if ((updateLane & renderLanes) !== 0) {
  // Process this update
}
```

### Priority Mapping

| User Action | Lane | Interruptible? |
|-------------|------|----------------|
| Click handler | SyncLane | No |
| Input typing | InputContinuousLane | Partially |
| setTimeout callback | DefaultLane | Yes |
| startTransition | TransitionLane | Yes |
| Offscreen/hidden | IdleLane | Yes |

## 3.6 Interview Questions: Fiber

**Q1: Why did React rewrite the reconciler from Stack to Fiber?**

**A:** The Stack reconciler used recursive tree traversal, which had critical limitations:
1. **Blocking**: Once started, rendering couldn't be paused
2. **No Prioritization**: All updates treated equally
3. **Animation Jank**: Long renders blocked 60fps animations
4. **Input Lag**: User interactions queued behind renders

Fiber solves this with:
- Linked list traversal (pauseable)
- Priority-based scheduling
- Concurrent rendering capabilities

**Q2: Explain the double buffering technique in Fiber.**

**A:** React maintains two fiber trees:
- **Current**: The rendered UI currently on screen
- **Work-in-Progress (WIP)**: The tree being built for next render

Benefits:
1. No partial UI: Updates are prepared "offscreen"
2. Recovery: If render fails, current tree is untouched
3. Concurrent: Multiple WIP trees can exist
4. Reuse: After commit, old current becomes new WIP's base

**Q3: What does shouldYield() check for?**

```javascript
function shouldYield() {
  const currentTime = getCurrentTime();
  // Yield if 5ms have passed (1 frame at 200fps)
  return currentTime >= deadline;
}

// This enables:
// 1. Browser to process user events
// 2. Animations to update
// 3. Higher priority work to interrupt
```

---

# 4. Component Lifecycle & Rendering

## 4.1 Class Component Lifecycle (Complete Reference)

```
┌─────────────────────────────────────────────────────────────────┐
│                         MOUNTING                                 │
├─────────────────────────────────────────────────────────────────┤
│  constructor(props)                                              │
│       │                                                          │
│       ▼                                                          │
│  static getDerivedStateFromProps(props, state) → newState       │
│       │                                                          │
│       ▼                                                          │
│  render()                                                        │
│       │                                                          │
│       ▼                                                          │
│  React updates DOM                                               │
│       │                                                          │
│       ▼                                                          │
│  componentDidMount() ←── Safe for side effects, subscriptions   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                         UPDATING                                 │
├─────────────────────────────────────────────────────────────────┤
│  New props / setState() / forceUpdate()                         │
│       │                                                          │
│       ▼                                                          │
│  static getDerivedStateFromProps(props, state)                  │
│       │                                                          │
│       ▼                                                          │
│  shouldComponentUpdate(nextProps, nextState) → boolean          │
│       │ (skip if returns false)                                  │
│       ▼                                                          │
│  render()                                                        │
│       │                                                          │
│       ▼                                                          │
│  getSnapshotBeforeUpdate(prevProps, prevState) → snapshot       │
│       │                                                          │
│       ▼                                                          │
│  React updates DOM                                               │
│       │                                                          │
│       ▼                                                          │
│  componentDidUpdate(prevProps, prevState, snapshot)             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        UNMOUNTING                                │
├─────────────────────────────────────────────────────────────────┤
│  componentWillUnmount() ←── Cleanup subscriptions, timers       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      ERROR HANDLING                              │
├─────────────────────────────────────────────────────────────────┤
│  static getDerivedStateFromError(error) → newState              │
│  componentDidCatch(error, errorInfo)                            │
└─────────────────────────────────────────────────────────────────┘
```

## 4.2 Function Component "Lifecycle" (Hooks Mapping)

| Class Lifecycle | Hooks Equivalent |
|----------------|-----------------|
| `constructor` | `useState` initial value / `useRef` |
| `getDerivedStateFromProps` | `useState` + update in render |
| `shouldComponentUpdate` | `React.memo` + custom comparison |
| `render` | Function body return |
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [deps])` |
| `componentWillUnmount` | `useEffect(() => () => cleanup, [])` |
| `getSnapshotBeforeUpdate` | No direct equivalent (useLayoutEffect close) |
| `componentDidCatch` | No hooks (must use class Error Boundary) |

```javascript
function LifecycleDemo({ userId }) {
  // "Constructor" - runs once per mount
  const [data, setData] = useState(null);
  const renderCount = useRef(0);
  
  // "getDerivedStateFromProps" - sync state from props
  const [prevUserId, setPrevUserId] = useState(userId);
  if (userId !== prevUserId) {
    setPrevUserId(userId);
    setData(null);  // Reset state when userId changes
  }
  
  // "componentDidMount" + "componentDidUpdate" for userId
  useEffect(() => {
    let cancelled = false;
    fetchUser(userId).then(result => {
      if (!cancelled) setData(result);
    });
    
    // "componentWillUnmount" cleanup
    return () => { cancelled = true; };
  }, [userId]);
  
  // Track renders
  renderCount.current++;
  
  return <div>{data?.name}</div>;
}
```

## 4.3 Render Phase Rules

### What MUST Be Pure in Render

```javascript
function Component({ items }) {
  // ❌ NEVER: Mutate props
  items.push(newItem);
  
  // ❌ NEVER: Mutate existing state
  const [list, setList] = useState([]);
  list.push(newItem);  // Mutation!
  
  // ❌ NEVER: Side effects in render
  localStorage.setItem('key', value);
  
  // ❌ NEVER: Subscriptions in render
  eventEmitter.on('event', handler);
  
  // ✅ OK: Create new objects/arrays
  const newItems = [...items, newItem];
  
  // ✅ OK: Derive values from props/state
  const total = items.reduce((sum, i) => sum + i.price, 0);
  
  // ✅ OK: Lazy state initialization
  const [data] = useState(() => expensiveComputation());
}
```

### Strict Mode Double Rendering

```javascript
// In development with StrictMode, React intentionally:
// 1. Double-invokes function components
// 2. Double-invokes state initializers
// 3. Double-invokes reducers

// This catches impure renders!
function ImpureComponent() {
  // Bug exposed by StrictMode:
  globalCounter++;  // Different results on each render!
  return <div>{globalCounter}</div>;  // Shows 2, not 1
}
```

## 4.4 Re-render Triggers & Bailouts

### When Does React Re-render?

```javascript
// Trigger 1: State change
const [count, setCount] = useState(0);
setCount(1);  // Re-render scheduled

// Trigger 2: Props change (parent re-rendered)
function Parent() {
  const [x, setX] = useState(0);
  return <Child value={x} />;  // Child re-renders when Parent does
}

// Trigger 3: Context change
const ThemeContext = createContext('light');
function Consumer() {
  const theme = useContext(ThemeContext);  // Re-renders on context change
}

// Trigger 4: Parent re-renders (even with same props!)
function Parent() {
  const [, forceUpdate] = useState();
  return <Child />;  // Child re-renders even if Child has no props
}
```

### Bailout Conditions

```javascript
// Bailout 1: Same state value (Object.is)
setCount(0);  // If count is already 0, React bails out early
setState(obj);  // If obj === currentObj, React bails out

// Bailout 2: React.memo with same props
const MemoChild = React.memo(Child);
// If props are shallowly equal, skip re-render

// Bailout 3: useMemo for expensive children
function Parent() {
  const [unrelated, setUnrelated] = useState(0);
  const child = useMemo(() => <ExpensiveChild />, []);
  return <div>{child}</div>;  // ExpensiveChild doesn't re-render
}
```

## 4.5 Interview Questions: Lifecycle

**Q1: Why was componentWillMount deprecated?**

**A:** Multiple reasons:
1. **Concurrent Mode**: Render phase can be called multiple times
2. **SSR Issues**: Called on server but effects expected in browser
3. **Async Rendering**: Side effects in render phase cause bugs
4. **Misleading Name**: Suggested it ran before mount, but render is part of mounting

```javascript
// Old (deprecated)
componentWillMount() {
  this.subscription = subscribe();  // Bug: may run multiple times
}

// New
componentDidMount() {
  this.subscription = subscribe();  // Safe: runs once, after mount
}
```

**Q2: What's the difference between useEffect and useLayoutEffect?**

| Aspect | useEffect | useLayoutEffect |
|--------|-----------|-----------------|
| Timing | After paint (async) | Before paint (sync) |
| Blocking | Non-blocking | Blocks visual updates |
| Use Case | Data fetching, subscriptions | DOM measurements, scroll |
| SSR | Works (no-op) | Warns (can cause mismatch) |

```javascript
// useLayoutEffect for measuring DOM
function Tooltip({ targetRef }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  
  useLayoutEffect(() => {
    // Measure before paint to prevent flicker
    const rect = targetRef.current.getBoundingClientRect();
    setPosition({ top: rect.bottom, left: rect.left });
  }, []);
  
  return <div style={position}>Tooltip</div>;
}
```

**Q3: Why can't you call hooks inside conditions or loops?**

```javascript
// ❌ WRONG
function Component({ condition }) {
  if (condition) {
    const [a, setA] = useState(0);  // Hook #1 (sometimes)
  }
  const [b, setB] = useState(0);  // Hook #2 (or #1?)
}

// React stores hooks in a linked list, indexed by call order:
// Render 1 (condition=true):  [useState#1, useState#2]
// Render 2 (condition=false): [useState#1]
// Now useState for 'b' returns 'a's state!

// ✅ CORRECT
function Component({ condition }) {
  const [a, setA] = useState(0);  // Always #1
  const [b, setB] = useState(0);  // Always #2
  // Use condition in the values, not around hooks
}
```

---

# 5. Hooks Deep Dive

## 5.1 How Hooks Work Internally

```javascript
// Simplified hooks implementation
let currentFiber = null;
let hookIndex = 0;

function useState(initialValue) {
  const fiber = currentFiber;
  const hooks = fiber.memoizedState || [];
  
  // Get existing hook or create new one
  const hook = hooks[hookIndex] || { 
    state: typeof initialValue === 'function' ? initialValue() : initialValue,
    queue: []
  };
  
  // Process queued updates
  hook.queue.forEach(action => {
    hook.state = typeof action === 'function' ? action(hook.state) : action;
  });
  hook.queue = [];
  
  // Store back and advance index
  hooks[hookIndex] = hook;
  fiber.memoizedState = hooks;
  hookIndex++;
  
  // Return state and setter
  const setState = (action) => {
    hook.queue.push(action);
    scheduleRender(fiber);
  };
  
  return [hook.state, setState];
}

// Before each render:
function beginWork(fiber) {
  currentFiber = fiber;
  hookIndex = 0;
  // ... render component
}
```

### Hook Storage Structure

```javascript
// Hooks are stored as a linked list on fiber.memoizedState
fiber.memoizedState = {
  memoizedState: 0,           // useState value
  baseState: 0,
  baseQueue: null,
  queue: { pending: null },   // Update queue
  next: {                     // Next hook (linked list)
    memoizedState: [dep1],    // useEffect deps
    tag: 5,                   // Effect tag
    create: () => {},         // Effect callback
    destroy: undefined,       // Cleanup function
    deps: [dep1],
    next: { ... }             // Next hook
  }
}
```

## 5.2 useState In-Depth

### State Updates Are Queued & Batched

```javascript
function Counter() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    // All three are batched into ONE re-render
    setCount(count + 1);  // Uses stale 'count' = 0
    setCount(count + 1);  // Uses stale 'count' = 0
    setCount(count + 1);  // Uses stale 'count' = 0
    // Result: count = 1 (not 3!)
  };
  
  const handleClickCorrect = () => {
    // Use functional updates for dependent state
    setCount(c => c + 1);  // 0 → 1
    setCount(c => c + 1);  // 1 → 2
    setCount(c => c + 1);  // 2 → 3
    // Result: count = 3
  };
}
```

### Lazy Initialization

```javascript
// ❌ Expensive computation on EVERY render
const [data, setData] = useState(expensiveComputation(props));

// ✅ Expensive computation only on FIRST render
const [data, setData] = useState(() => expensiveComputation(props));

// Note: The initializer function receives no arguments
// If you need props, closure over them:
const [data, setData] = useState(() => computeFromProps(props.id));
// BUT: This only uses props.id from first render!
// For derived state, use useMemo instead
```

### State Updates and Object.is

```javascript
// React uses Object.is for bailout comparison
const [user, setUser] = useState({ name: 'Alice' });

// ❌ This won't trigger re-render (same reference)
user.name = 'Bob';
setUser(user);  // Object.is(user, user) = true, bails out

// ✅ Create new object
setUser({ ...user, name: 'Bob' });  // New reference, re-renders

// Special case: NaN
setCount(NaN);
setCount(NaN);  // Object.is(NaN, NaN) = true, bails out (correct!)
```

## 5.3 useEffect In-Depth

### Effect Cleanup Timing

```javascript
useEffect(() => {
  console.log('Effect runs');
  
  return () => {
    console.log('Cleanup runs');
  };
}, [dep]);

// Timeline for dep change:
// 1. New render starts
// 2. Component function runs
// 3. React updates DOM
// 4. Browser paints
// 5. OLD cleanup runs (with old props/state)
// 6. NEW effect runs (with new props/state)
```

### Dependency Array Gotchas

```javascript
// ❌ Object in deps - new object every render
useEffect(() => {
  fetchData(config);
}, [{ page: 1, limit: 10 }]);  // Always re-runs!

// ✅ Primitive values or memoized objects
const config = useMemo(() => ({ page, limit }), [page, limit]);
useEffect(() => {
  fetchData(config);
}, [config]);

// ❌ Function in deps without useCallback
useEffect(() => {
  onUpdate(data);
}, [onUpdate]);  // onUpdate is new function each render!

// ✅ Use useCallback or include function body dependencies
const onUpdate = useCallback((data) => {
  console.log(data);
}, []);  // Now stable

useEffect(() => {
  onUpdate(data);
}, [onUpdate, data]);
```

### Effect with Async Functions

```javascript
// ❌ WRONG: useEffect callback can't be async
useEffect(async () => {
  const data = await fetchData();  // Returns Promise, not cleanup
}, []);

// ✅ CORRECT: Define async function inside
useEffect(() => {
  const fetchAndSet = async () => {
    const data = await fetchData();
    setData(data);
  };
  fetchAndSet();
}, []);

// ✅ BETTER: With cleanup for race conditions
useEffect(() => {
  let cancelled = false;
  
  const fetchAndSet = async () => {
    const data = await fetchData(userId);
    if (!cancelled) {
      setData(data);
    }
  };
  
  fetchAndSet();
  
  return () => { cancelled = true; };
}, [userId]);

// ✅ BEST: Use AbortController for actual cancellation
useEffect(() => {
  const controller = new AbortController();
  
  fetch(url, { signal: controller.signal })
    .then(res => res.json())
    .then(setData)
    .catch(err => {
      if (err.name !== 'AbortError') throw err;
    });
  
  return () => controller.abort();
}, [url]);
```

## 5.4 useRef Deep Dive

### ref.current Mutations Don't Trigger Re-renders

```javascript
function Timer() {
  const intervalRef = useRef(null);
  const countRef = useRef(0);
  const [, forceUpdate] = useState();
  
  const start = () => {
    intervalRef.current = setInterval(() => {
      countRef.current++;  // No re-render!
      console.log(countRef.current);  // Updates silently
    }, 1000);
  };
  
  const showCount = () => {
    // Need to force re-render to see countRef.current in UI
    forceUpdate({});
  };
  
  const stop = () => clearInterval(intervalRef.current);
  
  return (
    <div>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={showCount}>Show</button>
    </div>
  );
}
```

### Callback Refs vs Object Refs

```javascript
// Object ref - assigned automatically
const inputRef = useRef(null);
<input ref={inputRef} />  // inputRef.current = DOM node

// Callback ref - for dynamic elements or measurements
function MeasuredList({ items }) {
  const [heights, setHeights] = useState({});
  
  const measureRef = useCallback((node, id) => {
    if (node !== null) {
      setHeights(h => ({ ...h, [id]: node.getBoundingClientRect().height }));
    }
  }, []);
  
  return items.map(item => (
    <div key={item.id} ref={node => measureRef(node, item.id)}>
      {item.content}
    </div>
  ));
}

// Cleanup with callback refs
function WithCleanup() {
  const callbackRef = useCallback(node => {
    if (node) {
      // Node mounted - set up
      const observer = new ResizeObserver(...);
      observer.observe(node);
      // Return cleanup isn't supported!
      // Store observer somewhere to disconnect later
    }
  }, []);
}
```

## 5.5 useMemo & useCallback

### When to Use (and When NOT to)

```javascript
// ❌ OVER-OPTIMIZATION: Simple operations
const doubled = useMemo(() => value * 2, [value]);  // Unnecessary

// ✅ CORRECT USE: Expensive computation
const sortedList = useMemo(() => 
  items.slice().sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// ❌ OVER-OPTIMIZATION: Callback not passed to memoized child
const handleClick = useCallback(() => setCount(c => c + 1), []);
return <button onClick={handleClick}>+</button>;  // button isn't memo'd

// ✅ CORRECT USE: Callback passed to memo'd child
const Child = React.memo(({ onClick }) => <button onClick={onClick}>+</button>);
function Parent() {
  const handleClick = useCallback(() => setCount(c => c + 1), []);
  return <Child onClick={handleClick} />;  // Prevents Child re-render
}

// ✅ CORRECT USE: Callback in useEffect dependency
const fetchData = useCallback(() => {
  return fetch(`/api/user/${userId}`);
}, [userId]);

useEffect(() => {
  fetchData().then(setData);
}, [fetchData]);  // Only re-runs when userId changes
```

### useMemo vs Memoization Libraries

| Aspect | useMemo | External (reselect, memoize-one) |
|--------|---------|----------------------------------|
| Cache Size | 1 (last result) | Configurable |
| Scope | Per component instance | Shared across components |
| Dependencies | Array of deps | Argument comparison |
| Persistence | Until deps change | Until cache eviction |

## 5.6 useReducer

### When to Prefer useReducer over useState

```javascript
// Complex state with multiple sub-values
// ❌ Multiple useState - easy to have inconsistent updates
const [name, setName] = useState('');
const [email, setEmail] = useState('');
const [errors, setErrors] = useState({});
const [isSubmitting, setIsSubmitting] = useState(false);

// ✅ useReducer - atomic updates, clear transitions
const initialState = { name: '', email: '', errors: {}, isSubmitting: false };

function reducer(state, action) {
  switch (action.type) {
    case 'SET_FIELD':
      return { ...state, [action.field]: action.value };
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true, errors: {} };
    case 'SUBMIT_SUCCESS':
      return initialState;
    case 'SUBMIT_ERROR':
      return { ...state, isSubmitting: false, errors: action.errors };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

const [state, dispatch] = useReducer(reducer, initialState);
```

### useReducer with Lazy Initialization

```javascript
// init function for expensive initial state
function init(initialCount) {
  // Called once, like useState(() => ...)
  return { count: initialCount, history: [initialCount] };
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  // Third argument is init function, second is argument to init
}
```

## 5.7 Custom Hooks Patterns

### The Extract Hook Pattern

```javascript
// Before: Logic coupled to component
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);
  
  // ... render logic
}

// After: Reusable custom hook
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { user, loading, error } = useUser(userId);
  // ... render logic
}
```

### Hook Composition

```javascript
// Building higher-level hooks from lower-level ones
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });
  
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);
  
  return [value, setValue];
}

function useTheme() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  const toggleTheme = useCallback(() => {
    setTheme(t => t === 'light' ? 'dark' : 'light');
  }, [setTheme]);
  
  return { theme, toggleTheme };
}
```

## 5.8 Interview Questions: Hooks

**Q1: Why do hooks rely on call order?**

**A:** Hooks are stored in a linked list, identified by position (not name). This design choice enables:
1. No need for unique keys/names
2. Simple implementation
3. Works with existing JavaScript (no compiler magic)

The tradeoff: hooks must be called in the same order every render, hence the rules against conditionals/loops.

**Q2: What happens if you forget the dependency array in useEffect?**

```javascript
// Missing deps array = runs after EVERY render
useEffect(() => {
  fetchData();  // Fetches on every state change, even unrelated!
});

// Empty array = runs once on mount
useEffect(() => {
  fetchData();
}, []);

// With deps = runs when deps change
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

**Q3: How would you implement usePrevious?**

```javascript
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  });  // No deps: runs after every render
  
  return ref.current;  // Returns OLD value during render
}

// How it works:
// Render 1: value=A, ref.current=undefined, return undefined, then effect sets ref.current=A
// Render 2: value=B, ref.current=A, return A, then effect sets ref.current=B
// Render 3: value=C, ref.current=B, return B, then effect sets ref.current=C
```

**Q4: Implement useDebounce hook**

```javascript
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(timer);  // Cancel on value change
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);
  
  useEffect(() => {
    if (debouncedQuery) {
      searchAPI(debouncedQuery);
    }
  }, [debouncedQuery]);
}
```

---

# 6. State Management Patterns

## 6.1 Component State vs Application State

| Type | Scope | Examples | Where to Put |
|------|-------|----------|--------------|
| **Local UI State** | Single component | Form inputs, modals, toggles | useState |
| **Shared UI State** | Component subtree | Accordion open state | Lift state / Context |
| **Server Cache State** | App-wide | API responses | React Query / SWR |
| **URL State** | App-wide, persistent | Filters, pagination | Router params |
| **Global App State** | App-wide | User session, theme | Context / Redux / Zustand |

## 6.2 State Lifting Pattern

```javascript
// Problem: Sibling components need shared state
function App() {
  return (
    <>
      <SearchInput />   {/* Has query state */}
      <SearchResults /> {/* Needs query state */}
    </>
  );
}

// Solution: Lift state to common ancestor
function App() {
  const [query, setQuery] = useState('');
  
  return (
    <>
      <SearchInput query={query} onQueryChange={setQuery} />
      <SearchResults query={query} />
    </>
  );
}
```

## 6.3 Props Drilling Problem & Solutions

```javascript
// Problem: Props passed through many levels
function App() {
  const [user, setUser] = useState(null);
  return <Layout user={user} setUser={setUser} />;
}

function Layout({ user, setUser }) {
  return <Sidebar user={user} setUser={setUser} />;  // Just passing through
}

function Sidebar({ user, setUser }) {
  return <UserMenu user={user} setUser={setUser} />;  // Just passing through
}

function UserMenu({ user, setUser }) {
  return <UserAvatar user={user} />;  // Finally used!
}
```

### Solution 1: Component Composition

```javascript
// Pass component as children instead of data
function App() {
  const [user, setUser] = useState(null);
  
  return (
    <Layout>
      <Sidebar>
        <UserMenu user={user} setUser={setUser} />
      </Sidebar>
    </Layout>
  );
}

function Layout({ children }) {
  return <div className="layout">{children}</div>;
}

function Sidebar({ children }) {
  return <aside>{children}</aside>;
}
```

### Solution 2: Context API

```javascript
const UserContext = createContext(null);

function App() {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Layout />
    </UserContext.Provider>
  );
}

function UserMenu() {
  const { user, setUser } = useContext(UserContext);
  return <UserAvatar user={user} />;
}
```

## 6.4 Server State Management

### The Problem with useState for Server Data

```javascript
// ❌ Naive approach - many problems
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);
  
  // Problems:
  // 1. No caching - refetches on every mount
  // 2. No deduplication - multiple components fetch same data
  // 3. No background updates
  // 4. No optimistic updates
  // 5. Manual loading/error state
}
```

### React Query / TanStack Query Pattern

```javascript
// ✅ Server state library approach
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function UserList() {
  const { data: users, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
    staleTime: 5 * 60 * 1000,  // 5 minutes
  });
  
  // Automatic: caching, deduping, background refetch, error retry
}

function AddUserForm() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
    // Or optimistic update:
    onMutate: async (newUser) => {
      await queryClient.cancelQueries({ queryKey: ['users'] });
      const previousUsers = queryClient.getQueryData(['users']);
      queryClient.setQueryData(['users'], old => [...old, newUser]);
      return { previousUsers };
    },
    onError: (err, newUser, context) => {
      queryClient.setQueryData(['users'], context.previousUsers);
    },
  });
}
```

## 6.5 State Machine Pattern

```javascript
// Complex state transitions managed explicitly
const stateMachine = {
  idle: {
    FETCH: 'loading',
  },
  loading: {
    SUCCESS: 'success',
    ERROR: 'error',
  },
  success: {
    FETCH: 'loading',
    RESET: 'idle',
  },
  error: {
    FETCH: 'loading',
    RESET: 'idle',
  },
};

function reducer(state, event) {
  const nextState = stateMachine[state.status]?.[event.type];
  if (!nextState) return state;  // Invalid transition
  
  switch (event.type) {
    case 'FETCH':
      return { status: 'loading', data: null, error: null };
    case 'SUCCESS':
      return { status: 'success', data: event.data, error: null };
    case 'ERROR':
      return { status: 'error', data: null, error: event.error };
    case 'RESET':
      return { status: 'idle', data: null, error: null };
    default:
      return state;
  }
}

function useFetch(fetchFn) {
  const [state, dispatch] = useReducer(reducer, { 
    status: 'idle', 
    data: null, 
    error: null 
  });
  
  const execute = useCallback(async () => {
    dispatch({ type: 'FETCH' });
    try {
      const data = await fetchFn();
      dispatch({ type: 'SUCCESS', data });
    } catch (error) {
      dispatch({ type: 'ERROR', error });
    }
  }, [fetchFn]);
  
  return { ...state, execute };
}
```

## 6.6 Redux vs Context vs Zustand

| Feature | Context API | Redux | Zustand |
|---------|-------------|-------|---------|
| Boilerplate | Low | High | Very Low |
| DevTools | No | Yes | Yes |
| Middleware | No | Yes | Yes |
| Persistence | Manual | Easy | Easy |
| Learning Curve | Low | High | Low |
| Re-render Scope | All consumers | Selector-based | Selector-based |
| Bundle Size | 0 (built-in) | ~7KB | ~1KB |
| Use Case | Simple shared state | Complex app state | Medium complexity |

```javascript
// Zustand example - modern minimal approach
import { create } from 'zustand';

const useStore = create((set, get) => ({
  count: 0,
  users: [],
  
  increment: () => set(state => ({ count: state.count + 1 })),
  
  fetchUsers: async () => {
    const users = await api.getUsers();
    set({ users });
  },
  
  // Computed value (not reactive, but simple)
  getUserById: (id) => get().users.find(u => u.id === id),
}));

// Usage - component only re-renders when selected state changes
function Counter() {
  const count = useStore(state => state.count);
  const increment = useStore(state => state.increment);
  return <button onClick={increment}>{count}</button>;
}
```

---

# 7. Context API & Dependency Injection

## 7.1 Context Internals

```javascript
// createContext returns an object
const ThemeContext = createContext('light');

// ThemeContext structure:
{
  $$typeof: Symbol(react.context),
  _currentValue: 'light',      // Current value (used in render)
  _currentValue2: 'light',     // For concurrent rendering
  Provider: {
    $$typeof: Symbol(react.provider),
    _context: ThemeContext     // Reference back
  },
  Consumer: {
    $$typeof: Symbol(react.context),
    _context: ThemeContext
  }
}
```

## 7.2 Provider Value Stability

```javascript
// ❌ BAD: New object every render
function App() {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {/* All consumers re-render on ANY App re-render */}
      <Children />
    </UserContext.Provider>
  );
}

// ✅ GOOD: Memoize the value
function App() {
  const [user, setUser] = useState(null);
  
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return (
    <UserContext.Provider value={value}>
      <Children />
    </UserContext.Provider>
  );
}

// ✅ BETTER: Split contexts by update frequency
const UserContext = createContext(null);      // Rarely changes
const UserUpdateContext = createContext(null); // Never changes

function App() {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={user}>
      <UserUpdateContext.Provider value={setUser}>
        <Children />
      </UserUpdateContext.Provider>
    </UserContext.Provider>
  );
}

// Components reading only setUser won't re-render when user changes
```

## 7.3 Context Selector Problem

```javascript
// Problem: useContext re-renders for ANY context change
const AppContext = createContext({ theme: 'light', user: null, count: 0 });

function Counter() {
  const { count } = useContext(AppContext);
  // Re-renders when theme or user changes too!
  return <div>{count}</div>;
}

// Solution 1: Multiple contexts
const ThemeContext = createContext('light');
const UserContext = createContext(null);
const CountContext = createContext(0);

// Solution 2: use-context-selector library
import { createContext, useContextSelector } from 'use-context-selector';

const AppContext = createContext({ theme: 'light', user: null, count: 0 });

function Counter() {
  const count = useContextSelector(AppContext, ctx => ctx.count);
  // Only re-renders when count changes
  return <div>{count}</div>;
}
```

## 7.4 Context vs Props

| Use Context When | Use Props When |
|-----------------|----------------|
| Data needed by many components at different levels | Data needed by direct children |
| Data changes infrequently | Data changes frequently |
| Provider is near root | Explicit data flow is clearer |
| Avoiding prop drilling is priority | Component reusability is priority |

## 7.5 Interview Questions: Context

**Q1: Why doesn't Context have a selector API like Redux?**

**A:** By design, Context is meant for low-frequency updates (theme, locale, auth). Adding selectors would:
1. Increase complexity
2. Duplicate Redux functionality
3. Encourage using Context for all state

For selector-based updates, use external state libraries or multiple contexts.

**Q2: What's the difference between Context and Redux?**

```javascript
// Context: React feature for prop drilling avoidance
// - No middleware
// - No devtools
// - Re-renders all consumers on any value change
// - Good for: theme, locale, auth status

// Redux: External state management library
// - Middleware (thunks, sagas, logging)
// - Powerful devtools (time travel)
// - Selector-based re-renders
// - Good for: complex app state, debugging needs
```

**Q3: How do you test components that use Context?**

```javascript
// Wrap component with test provider
import { render, screen } from '@testing-library/react';

const TestProvider = ({ children, value = mockValue }) => (
  <UserContext.Provider value={value}>
    {children}
  </UserContext.Provider>
);

test('displays user name', () => {
  render(
    <TestProvider value={{ user: { name: 'Alice' } }}>
      <UserProfile />
    </TestProvider>
  );
  
  expect(screen.getByText('Alice')).toBeInTheDocument();
});

// Create custom render function
function renderWithProviders(ui, { initialState = {} } = {}) {
  return render(
    <AllProviders initialState={initialState}>
      {ui}
    </AllProviders>
  );
}
```

---

# 8. Performance Optimization

## 8.1 Performance Mental Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    PERFORMANCE HIERARCHY                         │
├─────────────────────────────────────────────────────────────────┤
│  1. MEASURE FIRST (React DevTools Profiler, Lighthouse)         │
│         │                                                        │
│         ▼                                                        │
│  2. FIX DATA FETCHING (waterfall, over-fetching)                │
│         │                                                        │
│         ▼                                                        │
│  3. REDUCE BUNDLE SIZE (code splitting, tree shaking)           │
│         │                                                        │
│         ▼                                                        │
│  4. AVOID UNNECESSARY RENDERS (memo, useMemo, useCallback)      │
│         │                                                        │
│         ▼                                                        │
│  5. VIRTUALIZE LONG LISTS (react-window, react-virtualized)     │
│         │                                                        │
│         ▼                                                        │
│  6. WEB WORKERS (heavy computation off main thread)             │
└─────────────────────────────────────────────────────────────────┘
```

## 8.2 React.memo Deep Dive

```javascript
// Default: Shallow comparison of all props
const MemoizedComponent = React.memo(Component);

// Custom comparison function
const MemoizedComponent = React.memo(Component, (prevProps, nextProps) => {
  // Return true if props are "equal" (skip re-render)
  // Return false if props are "different" (re-render)
  return prevProps.id === nextProps.id;
});

// Common gotcha: inline objects/functions break memoization
function Parent() {
  return (
    // ❌ New object every render
    <MemoChild style={{ color: 'red' }} />
    
    // ❌ New function every render
    <MemoChild onClick={() => handleClick(id)} />
  );
}

// ✅ Fix with useMemo/useCallback
function Parent() {
  const style = useMemo(() => ({ color: 'red' }), []);
  const handleClick = useCallback(() => onClick(id), [id, onClick]);
  
  return <MemoChild style={style} onClick={handleClick} />;
}
```

### When NOT to Use memo

```javascript
// ❌ Primitive props only - memo overhead > benefit
const Button = memo(({ label, disabled }) => (
  <button disabled={disabled}>{label}</button>
));

// ❌ Component almost always receives new props
const ListItem = memo(({ item, index, onSelect }) => (
  // If parent remaps array every render, item is new reference
));

// ❌ Very cheap to render
const Divider = memo(() => <hr />);  // Overhead not worth it
```

## 8.3 Avoiding Re-renders

### Pattern: Move State Down

```javascript
// ❌ BAD: Entire app re-renders on input change
function App() {
  const [query, setQuery] = useState('');
  
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <ExpensiveComponent />  {/* Re-renders on every keystroke! */}
    </div>
  );
}

// ✅ GOOD: Isolate state to where it's needed
function SearchInput() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}

function App() {
  return (
    <div>
      <SearchInput />
      <ExpensiveComponent />  {/* Doesn't re-render */}
    </div>
  );
}
```

### Pattern: Lift Content Up

```javascript
// ❌ BAD: Children re-render when scroll state changes
function ScrollContainer() {
  const [scrollY, setScrollY] = useState(0);
  
  return (
    <div onScroll={e => setScrollY(e.target.scrollTop)}>
      <Header scrollY={scrollY} />
      <ExpensiveContent />  {/* Re-renders! */}
    </div>
  );
}

// ✅ GOOD: Pass children, they're already rendered
function ScrollContainer({ children }) {
  const [scrollY, setScrollY] = useState(0);
  
  return (
    <div onScroll={e => setScrollY(e.target.scrollTop)}>
      <Header scrollY={scrollY} />
      {children}  {/* Doesn't re-render */}
    </div>
  );
}

function App() {
  return (
    <ScrollContainer>
      <ExpensiveContent />
    </ScrollContainer>
  );
}
```

## 8.4 List Virtualization

```javascript
// Without virtualization: 10,000 items = 10,000 DOM nodes
// With virtualization: Only visible items + buffer

import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>{items[index].name}</div>
  );
  
  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={35}
      width={300}
    >
      {Row}
    </FixedSizeList>
  );
}

// For variable height items
import { VariableSizeList } from 'react-window';

function getItemSize(index) {
  return items[index].expanded ? 100 : 35;
}

<VariableSizeList
  height={400}
  itemCount={items.length}
  itemSize={getItemSize}
  estimatedItemSize={35}
>
  {Row}
</VariableSizeList>
```

## 8.5 Code Splitting

### Route-Based Splitting

```javascript
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Dynamic imports - separate chunks
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

### Component-Level Splitting

```javascript
// Heavy component loaded on demand
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}

// Named exports require wrapper
const MyComponent = lazy(() => 
  import('./components').then(module => ({ default: module.MyComponent }))
);
```

## 8.6 React DevTools Profiler

### Key Metrics to Watch

| Metric | What It Means | Target |
|--------|---------------|--------|
| Render Duration | Time to render component | < 16ms for 60fps |
| Commit Duration | Time to apply DOM changes | < 10ms |
| Render Count | Times component rendered | Minimize unnecessary |
| Why Did This Render | Trigger for re-render | Should be intentional |

### Profiler API

```javascript
import { Profiler } from 'react';

function onRenderCallback(
  id,                   // Profiler tree id
  phase,                // "mount" | "update"
  actualDuration,       // Time rendering committed update
  baseDuration,         // Estimated time without memoization
  startTime,            // When React began rendering
  commitTime,           // When React committed
  interactions          // Set of tracked interactions
) {
  // Log to analytics
  analytics.track('render', {
    component: id,
    duration: actualDuration,
    phase,
  });
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Component />
    </Profiler>
  );
}
```

## 8.7 Interview Questions: Performance

**Q1: How do you identify performance problems in a React app?**

**A:** Systematic approach:
1. **User reports**: Slow interactions, janky scrolling
2. **React DevTools Profiler**: Identify slow components, unnecessary re-renders
3. **Chrome DevTools Performance**: Check main thread blocking, long tasks
4. **Lighthouse**: Overall performance score, Core Web Vitals
5. **React.StrictMode**: Double-renders expose side effects and impurity

**Q2: Explain the difference between useMemo and React.memo**

| useMemo | React.memo |
|---------|------------|
| Memoizes a **value** | Memoizes a **component** |
| Called inside component | Wraps component definition |
| Deps array comparison | Props shallow comparison |
| Returns computed value | Returns component |

```javascript
// useMemo - avoid expensive recomputation
const sortedData = useMemo(() => data.sort(compareFn), [data]);

// React.memo - avoid re-rendering when props unchanged
const MemoComponent = React.memo(({ data }) => {
  return <ExpensiveRender data={data} />;
});
```

**Q3: What is React.lazy and when would you use it?**

**A:** React.lazy enables code-splitting by loading components dynamically:

```javascript
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

// Use cases:
// 1. Route-based splitting (load pages on navigation)
// 2. Modal/dialog content (load when opened)
// 3. Feature flags (load only if feature enabled)
// 4. Below-the-fold content (load after initial render)

// Must be wrapped in Suspense
<Suspense fallback={<Loading />}>
  <HeavyComponent />
</Suspense>
```

---

# 9. Concurrent React & Suspense

## 9.1 What is Concurrent Rendering?

Concurrent React allows React to:
1. **Pause** rendering to handle urgent updates
2. **Abandon** incomplete renders if state changes
3. **Render multiple versions** of UI simultaneously
4. **Keep UI responsive** during expensive renders

```javascript
// Before Concurrent React:
// Render A → Render B → Render C (sequential, blocking)

// With Concurrent React:
// Render A... (urgent update arrives) → Pause A
// → Handle urgent update → Resume or restart A
```

## 9.2 Transitions

### startTransition

```javascript
import { startTransition, useState } from 'react';

function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    // Urgent: Update input immediately
    setQuery(value);
    
    // Non-urgent: Can be interrupted
    startTransition(() => {
      setResults(filterResults(value));  // Heavy computation
    });
  };
  
  return (
    <>
      <input value={query} onChange={handleChange} />
      <ResultsList results={results} />
    </>
  );
}
```

### useTransition Hook

```javascript
import { useTransition, useState } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();
  
  const selectTab = (nextTab) => {
    startTransition(() => {
      setTab(nextTab);  // Render new tab content (may be slow)
    });
  };
  
  return (
    <>
      <TabButtons onClick={selectTab} />
      {isPending && <Spinner />}  {/* Show while transitioning */}
      <TabContent tab={tab} />
    </>
  );
}
```

## 9.3 useDeferredValue

```javascript
import { useDeferredValue, useMemo, useState } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  
  // This is "stale" during transition
  const isStale = query !== deferredQuery;
  
  // Heavy filtering uses deferred value
  const results = useMemo(
    () => filterItems(deferredQuery),
    [deferredQuery]
  );
  
  return (
    <>
      <input 
        value={query} 
        onChange={e => setQuery(e.target.value)} 
      />
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <ResultsList results={results} />
      </div>
    </>
  );
}
```

### startTransition vs useDeferredValue

| startTransition | useDeferredValue |
|-----------------|------------------|
| Wraps state updates | Wraps values |
| You control the setter | You receive a derived value |
| For: Your own state updates | For: Values from props or external sources |
| Returns nothing (or isPending) | Returns deferred version of value |

## 9.4 Suspense Deep Dive

### Data Fetching with Suspense

```javascript
// Suspense-enabled data source (e.g., React Query, Relay, or custom)
function createResource(promise) {
  let status = 'pending';
  let result;
  let suspender = promise.then(
    (data) => { status = 'success'; result = data; },
    (error) => { status = 'error'; result = error; }
  );
  
  return {
    read() {
      if (status === 'pending') throw suspender;
      if (status === 'error') throw result;
      return result;
    }
  };
}

// Usage
const userResource = createResource(fetchUser(userId));

function UserProfile() {
  const user = userResource.read();  // Suspends until resolved
  return <h1>{user.name}</h1>;
}

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <UserProfile />
    </Suspense>
  );
}
```

### Nested Suspense Boundaries

```javascript
function Dashboard() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Header />
      
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />
      </Suspense>
      
      <Suspense fallback={<ContentSkeleton />}>
        <MainContent />
      </Suspense>
    </Suspense>
  );
}

// Loading sequence:
// 1. PageSkeleton shows
// 2. Header loads, replaces header portion
// 3. SidebarSkeleton and ContentSkeleton show (paralell)
// 4. Components load as they resolve
```

### SuspenseList (Experimental)

```javascript
import { SuspenseList, Suspense } from 'react';

function Feed() {
  return (
    <SuspenseList revealOrder="forwards" tail="collapsed">
      <Suspense fallback={<PostSkeleton />}>
        <Post id={1} />
      </Suspense>
      <Suspense fallback={<PostSkeleton />}>
        <Post id={2} />
      </Suspense>
      <Suspense fallback={<PostSkeleton />}>
        <Post id={3} />
      </Suspense>
    </SuspenseList>
  );
}

// revealOrder options:
// "forwards" - reveal in order (1, then 2, then 3)
// "backwards" - reveal in reverse order
// "together" - reveal all at once when all ready
```

## 9.5 useId Hook

```javascript
// Generates stable IDs that match between server and client
import { useId } from 'react';

function FormField({ label }) {
  const id = useId();  // e.g., ":r1:"
  
  return (
    <>
      <label htmlFor={id}>{label}</label>
      <input id={id} />
    </>
  );
}

// Multiple IDs from same base
function PasswordField() {
  const id = useId();
  
  return (
    <>
      <label htmlFor={`${id}-password`}>Password</label>
      <input id={`${id}-password`} type="password" />
      
      <label htmlFor={`${id}-confirm`}>Confirm</label>
      <input id={`${id}-confirm`} type="password" />
    </>
  );
}
```

## 9.6 Interview Questions: Concurrent React

**Q1: What problem does Concurrent Rendering solve?**

**A:** Blocking renders that cause UI jank:
- Large list updates blocking input
- Navigation transitions freezing UI
- Heavy computations on main thread

Concurrent React makes renders interruptible, so urgent updates (typing, clicking) aren't blocked by non-urgent updates (search results, page transitions).

**Q2: When should you use useTransition vs useDeferredValue?**

```javascript
// useTransition: When YOU control the state update
function Search() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    startTransition(() => {
      setQuery(e.target.value);  // You're calling the setter
    });
  };
}

// useDeferredValue: When value comes from OUTSIDE (props, context)
function Results({ searchQuery }) {  // Prop, you don't control it
  const deferredQuery = useDeferredValue(searchQuery);
  // Use deferredQuery for heavy operations
}
```

**Q3: Explain how Suspense coordinates loading states**

**A:** Suspense works by catching thrown promises:

1. Component throws a promise (from data-fetching library)
2. Nearest `<Suspense>` boundary catches it
3. Suspense renders fallback
4. When promise resolves, React re-renders subtree
5. Component now returns actual content

```javascript
// Pseudo-implementation
function Suspense({ children, fallback }) {
  try {
    return children;
  } catch (thrown) {
    if (thrown instanceof Promise) {
      thrown.then(() => scheduleRerender());
      return fallback;
    }
    throw thrown;  // Not a Promise, re-throw
  }
}
```

---

# 10. Server Components & SSR

## 10.1 Rendering Strategies Comparison

| Strategy | Where Rendered | Hydration | Use Case |
|----------|---------------|-----------|----------|
| **CSR** | Browser only | N/A | SPAs, authenticated apps |
| **SSR** | Server → Client | Full hydration | SEO, initial load |
| **SSG** | Build time | Full hydration | Static content |
| **ISR** | Build + revalidate | Full hydration | Semi-static content |
| **RSC** | Server only | Partial (client components) | Mixed content |
| **Streaming SSR** | Server, streamed | Progressive | Large pages |

## 10.2 Server Components (RSC)

### Server vs Client Components

```javascript
// server-component.jsx (default in app/ directory)
// - Can read files, query databases directly
// - No hooks (no useState, useEffect)
// - No browser APIs
// - Zero JS shipped to client

async function ServerComponent() {
  const data = await db.query('SELECT * FROM posts');  // Direct DB access!
  
  return (
    <article>
      <h1>{data.title}</h1>
      <ClientButton />  {/* Client component for interactivity */}
    </article>
  );
}

// client-component.jsx
'use client';  // Directive marks as Client Component

import { useState } from 'react';

function ClientButton() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Clicked {count} times
    </button>
  );
}
```

### RSC Mental Model

```
┌─────────────────────────────────────────────────────────────────┐
│                        SERVER                                    │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Server Components (RSC)                                    ││
│  │  - Render to JSON-like payload                             ││
│  │  - Can be async                                            ││
│  │  - Direct backend access                                   ││
│  │  - No interactivity                                        ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ RSC Payload (not HTML)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT                                    │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Client Components                                          ││
│  │  - Traditional React                                       ││
│  │  - useState, useEffect work                                ││
│  │  - Event handlers, interactivity                           ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Composition Rules

```javascript
// ✅ OK: Server component renders client component
// ServerParent.jsx
import ClientChild from './ClientChild';

function ServerParent() {
  const data = await fetchData();
  return <ClientChild initialData={data} />;
}

// ✅ OK: Client component receives server component as children
// ClientWrapper.jsx
'use client';
function ClientWrapper({ children }) {
  const [show, setShow] = useState(true);
  return show ? children : null;
}

// ServerParent.jsx
<ClientWrapper>
  <ServerComponent />  {/* Pre-rendered, passed as serialized props */}
</ClientWrapper>

// ❌ NOT OK: Client component imports server component
'use client';
import ServerComponent from './ServerComponent';  // Error!
function ClientParent() {
  return <ServerComponent />;
}
```

## 10.3 Streaming SSR

```javascript
// React 18+ streaming with renderToPipeableStream
import { renderToPipeableStream } from 'react-dom/server';

app.get('/', (req, res) => {
  let didError = false;
  
  const stream = renderToPipeableStream(
    <App />,
    {
      bootstrapScripts: ['/client.js'],
      
      onShellReady() {
        // Shell (everything outside Suspense) is ready
        res.statusCode = didError ? 500 : 200;
        res.setHeader('Content-Type', 'text/html');
        stream.pipe(res);
      },
      
      onShellError(error) {
        res.statusCode = 500;
        res.send('<h1>Error</h1>');
      },
      
      onError(error) {
        didError = true;
        console.error(error);
      }
    }
  );
});
```

### Streaming Timeline

```
Browser receives:
├── HTML shell (header, layout)          ← onShellReady
│
├── <Suspense fallback="Loading...">
│
│   ... time passes, data loads ...
│
├── <script>$RS(Suspense content)</script>  ← Streamed in place
│
├── <Suspense fallback="Loading...">
│
│   ... more time passes ...
│
└── <script>$RS(More content)</script>    ← Streamed in place
```

## 10.4 Hydration

### Full Hydration Process

```
1. Server renders HTML
2. Browser displays HTML (non-interactive)
3. JavaScript loads
4. React "hydrates" - attaches event handlers to existing DOM
5. App becomes interactive
```

### Hydration Mismatch Errors

```javascript
// Common causes of hydration mismatch:

// 1. Date/time rendering
// Server: "March 23, 2026 10:00 UTC"
// Client: "March 23, 2026 15:30 IST"
const BadDate = () => <span>{new Date().toLocaleString()}</span>;

// Fix: Use useEffect for client-only values
const GoodDate = () => {
  const [date, setDate] = useState(null);
  useEffect(() => {
    setDate(new Date().toLocaleString());
  }, []);
  return <span>{date}</span>;
};

// 2. Random values
const BadRandom = () => <span>{Math.random()}</span>;  // Different each render

// 3. Browser-only APIs
const BadWindow = () => <span>{window.innerWidth}</span>;  // window undefined on server

// 4. Invalid HTML nesting
<p>
  <div>This causes mismatch</div>  {/* Invalid: div inside p */}
</p>
```

### Selective Hydration (React 18)

```javascript
// Suspense boundaries enable selective hydration
<Suspense fallback={<Spinner />}>
  <Comments />  {/* Can hydrate independently */}
</Suspense>

// User interaction prioritizes hydration
// If user clicks on unhydrated <Comments>,
// React prioritizes hydrating it immediately
```

## 10.5 Interview Questions: SSR/RSC

**Q1: What's the difference between SSR and RSC?**

| SSR | RSC |
|-----|-----|
| Server renders to HTML string | Server renders to serialized component tree |
| All components run on both server and client | Server Components run only on server |
| Full hydration required | Selective hydration, less JS |
| Limited to initial render | Can refetch RSC on navigation |

**Q2: What are the benefits of streaming SSR?**

**A:**
1. **Faster Time to First Byte**: Shell renders before data
2. **Progressive Loading**: Content appears as it's ready
3. **Improved Core Web Vitals**: FCP/LCP happen earlier
4. **Better Error Recovery**: Errors isolated to Suspense boundaries

**Q3: When would you choose CSR over SSR?**

**A:** CSR is preferable when:
1. **Authenticated dashboards**: No SEO needed, user must be logged in
2. **Complex interactivity**: Heavy client-side state from the start
3. **Offline-first PWAs**: Service worker caches client app
4. **Real-time apps**: Constant WebSocket updates
5. **Privacy concerns**: No server-side rendering of sensitive data

---

# 11. Error Boundaries & Error Handling

## 11.1 Error Boundaries

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    // Update state to show fallback UI
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    // Log to error reporting service
    errorService.log({
      error: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack
    });
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback || <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary fallback={<ErrorPage />}>
      <Dashboard />
    </ErrorBoundary>
  );
}
```

### What Error Boundaries DON'T Catch

```javascript
// 1. Event handlers (use try-catch)
function Button() {
  const handleClick = () => {
    try {
      doSomething();
    } catch (error) {
      // Handle manually
    }
  };
  return <button onClick={handleClick}>Click</button>;
}

// 2. Async code (use error state or error boundaries in data libs)
useEffect(() => {
  fetchData().catch(error => setError(error));
}, []);

// 3. Server-side rendering
// Use try-catch in renderToString

// 4. Errors in the error boundary itself
```

### Error Boundary Placement Strategy

```javascript
// Granular boundaries for isolation
<ErrorBoundary fallback={<WidgetError />}>
  <WeatherWidget />
</ErrorBoundary>

<ErrorBoundary fallback={<WidgetError />}>
  <StockWidget />
</ErrorBoundary>

// Route-level boundaries
<Routes>
  <Route 
    path="/dashboard" 
    element={
      <ErrorBoundary fallback={<PageError />}>
        <Dashboard />
      </ErrorBoundary>
    } 
  />
</Routes>

// App-level boundary as final catch-all
<ErrorBoundary fallback={<CatastrophicError />}>
  <App />
</ErrorBoundary>
```

## 11.2 Error Handling Patterns

### Reset After Error

```javascript
class ResettableErrorBoundary extends React.Component {
  state = { hasError: false, errorKey: 0 };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  resetErrorBoundary = () => {
    this.setState({ hasError: false, errorKey: this.state.errorKey + 1 });
  };
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Something went wrong</h1>
          <button onClick={this.resetErrorBoundary}>Try Again</button>
        </div>
      );
    }
    return <React.Fragment key={this.state.errorKey}>{this.props.children}</React.Fragment>;
  }
}
```

### react-error-boundary Library

```javascript
import { ErrorBoundary, useErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // Reset app state
      }}
      resetKeys={[someKey]}  // Reset when keys change
    >
      <Dashboard />
    </ErrorBoundary>
  );
}

// Programmatic error throwing
function ChildComponent() {
  const { showBoundary } = useErrorBoundary();
  
  const handleClick = async () => {
    try {
      await riskyOperation();
    } catch (error) {
      showBoundary(error);  // Propagate to boundary
    }
  };
}
```

## 11.3 Interview Questions: Error Handling

**Q1: Why can't hooks implement error boundaries?**

**A:** Error boundaries require:
1. `static getDerivedStateFromError` - static methods don't exist in functions
2. `componentDidCatch` - no equivalent hook exists
3. Class instance to store error state between catch and render

React team's stance: Error boundaries are a specific pattern that works well with classes. A hook like `useErrorBoundary` might be added in future.

**Q2: How do you handle async errors in React?**

```javascript
// Pattern 1: Error state in component
function Component() {
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchData()
      .then(setData)
      .catch(setError);
  }, []);
  
  if (error) throw error;  // Propagate to boundary
  // or
  if (error) return <ErrorMessage error={error} />;
}

// Pattern 2: Data fetching library (React Query)
const { data, error } = useQuery(['key'], fetchData, {
  useErrorBoundary: true  // Throw to nearest error boundary
});

// Pattern 3: Suspense with Error Boundary
<ErrorBoundary fallback={<Error />}>
  <Suspense fallback={<Loading />}>
    <DataComponent />
  </Suspense>
</ErrorBoundary>
```

---

# 12. React Patterns & Anti-Patterns

## 12.1 Composition Patterns

### Compound Components

```javascript
// Example: Tabs component with shared state
const TabsContext = createContext();

function Tabs({ children, defaultValue }) {
  const [activeTab, setActiveTab] = useState(defaultValue);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ value, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  
  return (
    <button
      className={activeTab === value ? 'active' : ''}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

function TabPanels({ children }) {
  return <div className="tab-panels">{children}</div>;
}

function TabPanel({ value, children }) {
  const { activeTab } = useContext(TabsContext);
  return activeTab === value ? <div>{children}</div> : null;
}

// Attach sub-components
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panels = TabPanels;
Tabs.Panel = TabPanel;

// Usage - clean, declarative API
<Tabs defaultValue="tab1">
  <Tabs.List>
    <Tabs.Tab value="tab1">Tab 1</Tabs.Tab>
    <Tabs.Tab value="tab2">Tab 2</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panels>
    <Tabs.Panel value="tab1">Content 1</Tabs.Panel>
    <Tabs.Panel value="tab2">Content 2</Tabs.Panel>
  </Tabs.Panels>
</Tabs>
```

### Render Props

```javascript
// Pattern: Pass render function as prop
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);
  
  return render(position);
}

// Usage
<MouseTracker 
  render={({ x, y }) => (
    <div>Mouse is at ({x}, {y})</div>
  )}
/>

// Alternative: children as function
<MouseTracker>
  {({ x, y }) => <div>Mouse is at ({x}, {y})</div>}
</MouseTracker>
```

### Higher-Order Components (HOC)

```javascript
// HOC: Function that takes a component, returns enhanced component
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { user, loading } = useAuth();
    
    if (loading) return <Loading />;
    if (!user) return <Redirect to="/login" />;
    
    return <WrappedComponent {...props} user={user} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);

// HOC conventions:
// 1. Don't mutate the original component
// 2. Pass through unrelated props
// 3. Wrap display name for debugging
AuthenticatedComponent.displayName = `withAuth(${WrappedComponent.displayName || WrappedComponent.name})`;
```

## 12.2 Anti-Patterns

### Anti-Pattern: Derived State from Props

```javascript
// ❌ BAD: Copying props to state
function BadComponent({ initialValue }) {
  const [value, setValue] = useState(initialValue);
  // value never updates when initialValue changes!
  
  return <input value={value} onChange={e => setValue(e.target.value)} />;
}

// ✅ GOOD: Use key to reset
<GoodComponent key={userId} initialValue={initialValue} />

// ✅ GOOD: Control completely from parent
function ControlledInput({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}

// ✅ GOOD: Fully uncontrolled with key reset
function UncontrolledInput({ defaultValue }) {
  const [value, setValue] = useState(defaultValue);
  return <input value={value} onChange={e => setValue(e.target.value)} />;
}
<UncontrolledInput key={itemId} defaultValue={item.value} />
```

### Anti-Pattern: Overusing useEffect

```javascript
// ❌ BAD: Unnecessary effect for derived data
function Component({ items }) {
  const [filteredItems, setFilteredItems] = useState([]);
  
  useEffect(() => {
    setFilteredItems(items.filter(i => i.active));
  }, [items]);
  // Causes extra render: items change → render → effect → setState → render again
  
  return <List items={filteredItems} />;
}

// ✅ GOOD: Calculate during render
function Component({ items }) {
  const filteredItems = useMemo(
    () => items.filter(i => i.active),
    [items]
  );
  // Or just: const filteredItems = items.filter(i => i.active);
  
  return <List items={filteredItems} />;
}
```

### Anti-Pattern: Prop Spreading Without Control

```javascript
// ❌ RISKY: Spreading all props
function Input(props) {
  return <input {...props} />;  // Could pass invalid DOM attributes
}
<Input onCustomEvent={handler} />  // onCustomEvent goes to DOM, warning!

// ✅ SAFE: Destructure known props
function Input({ label, error, ...inputProps }) {
  return (
    <div>
      <label>{label}</label>
      <input {...inputProps} />  // Only valid input props
      {error && <span className="error">{error}</span>}
    </div>
  );
}
```

## 12.3 Component Design Principles

### Single Responsibility

```javascript
// ❌ BAD: Component does too much
function UserDashboard({ userId }) {
  // Fetches data
  // Handles authentication
  // Manages form state
  // Renders complex UI
  // 500 lines...
}

// ✅ GOOD: Split responsibilities
function UserDashboard({ userId }) {
  return (
    <RequireAuth>
      <DashboardLayout>
        <UserDataProvider userId={userId}>
          <UserStats />
          <UserActivityFeed />
          <UserSettingsForm />
        </UserDataProvider>
      </DashboardLayout>
    </RequireAuth>
  );
}
```

### Inversion of Control

```javascript
// ❌ RIGID: Component controls everything
function Dropdown({ options, onSelect }) {
  return (
    <ul>
      {options.map(opt => (
        <li key={opt.id} onClick={() => onSelect(opt)}>
          {opt.label}  {/* Fixed rendering */}
        </li>
      ))}
    </ul>
  );
}

// ✅ FLEXIBLE: Let parent control rendering
function Dropdown({ options, renderOption, onSelect }) {
  return (
    <ul>
      {options.map(opt => (
        <li key={opt.id} onClick={() => onSelect(opt)}>
          {renderOption ? renderOption(opt) : opt.label}
        </li>
      ))}
    </ul>
  );
}

// Usage: Full control over option rendering
<Dropdown
  options={users}
  renderOption={(user) => (
    <div>
      <Avatar src={user.avatar} />
      <span>{user.name}</span>
    </div>
  )}
  onSelect={handleSelect}
/>
```

---

# 13. Testing Strategies

## 13.1 Testing Library Principles

```javascript
// Core principle: Test behavior, not implementation
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// ❌ BAD: Testing implementation details
expect(component.state.count).toBe(1);
expect(wrapper.find('Button').props().disabled).toBe(true);

// ✅ GOOD: Testing what user sees/does
expect(screen.getByText('Count: 1')).toBeInTheDocument();
expect(screen.getByRole('button', { name: 'Submit' })).toBeDisabled();
```

## 13.2 Query Priority

```javascript
// Priority order (most to least preferred):
// 1. Accessible queries (how users find elements)
screen.getByRole('button', { name: 'Submit' });
screen.getByLabelText('Email');
screen.getByPlaceholderText('Enter email');
screen.getByText('Welcome');
screen.getByDisplayValue('current value');

// 2. Semantic queries
screen.getByAltText('User avatar');
screen.getByTitle('Close');

// 3. Test IDs (last resort)
screen.getByTestId('custom-element');
```

## 13.3 Testing Patterns

### Testing Async Behavior

```javascript
test('loads and displays user data', async () => {
  // Mock API
  server.use(
    rest.get('/api/user', (req, res, ctx) => {
      return res(ctx.json({ name: 'Alice' }));
    })
  );
  
  render(<UserProfile userId="123" />);
  
  // Assert loading state
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  
  // Wait for content
  await waitFor(() => {
    expect(screen.getByText('Alice')).toBeInTheDocument();
  });
  
  // Or use findBy (combines getBy + waitFor)
  expect(await screen.findByText('Alice')).toBeInTheDocument();
});
```

### Testing Custom Hooks

```javascript
import { renderHook, act } from '@testing-library/react';

test('useCounter increments', () => {
  const { result } = renderHook(() => useCounter(0));
  
  expect(result.current.count).toBe(0);
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});

// With wrapper for context
const wrapper = ({ children }) => (
  <ThemeProvider theme="dark">{children}</ThemeProvider>
);

const { result } = renderHook(() => useTheme(), { wrapper });
```

### Testing Forms

```javascript
test('submits form with entered data', async () => {
  const handleSubmit = jest.fn();
  const user = userEvent.setup();
  
  render(<ContactForm onSubmit={handleSubmit} />);
  
  // Fill form
  await user.type(screen.getByLabelText('Name'), 'Alice');
  await user.type(screen.getByLabelText('Email'), 'alice@example.com');
  await user.selectOptions(screen.getByLabelText('Topic'), 'support');
  await user.click(screen.getByLabelText('Subscribe'));
  
  // Submit
  await user.click(screen.getByRole('button', { name: 'Submit' }));
  
  // Assert
  expect(handleSubmit).toHaveBeenCalledWith({
    name: 'Alice',
    email: 'alice@example.com',
    topic: 'support',
    subscribe: true
  });
});
```

## 13.4 Component Integration Testing

```javascript
// Test component tree, not just units
test('checkout flow completes successfully', async () => {
  const user = userEvent.setup();
  
  render(
    <CartProvider>
      <Router>
        <CheckoutPage />
      </Router>
    </CartProvider>
  );
  
  // Step 1: Shipping
  await user.type(screen.getByLabelText('Address'), '123 Main St');
  await user.click(screen.getByRole('button', { name: 'Continue' }));
  
  // Step 2: Payment (new screen)
  await screen.findByText('Payment Details');
  await user.type(screen.getByLabelText('Card Number'), '4111111111111111');
  await user.click(screen.getByRole('button', { name: 'Place Order' }));
  
  // Step 3: Confirmation
  expect(await screen.findByText('Order Confirmed')).toBeInTheDocument();
  expect(screen.getByText('Order #')).toBeInTheDocument();
});
```

## 13.5 Mocking Strategies

```javascript
// Module mocking
jest.mock('./api', () => ({
  fetchUser: jest.fn().mockResolvedValue({ name: 'Alice' })
}));

// MSW for API mocking (recommended)
import { setupServer } from 'msw/node';
import { rest } from 'msw';

const server = setupServer(
  rest.get('/api/user/:id', (req, res, ctx) => {
    return res(ctx.json({ id: req.params.id, name: 'Alice' }));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Override for specific test
test('handles error', async () => {
  server.use(
    rest.get('/api/user/:id', (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );
  // Test error handling...
});
```

---

# 14. Security Best Practices

## 14.1 XSS Prevention

### dangerouslySetInnerHTML

```javascript
// ❌ DANGEROUS: User input directly rendered as HTML
function Comment({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
  // If html = '<script>stealCookies()</script>', it executes!
}

// ✅ SAFE: Sanitize before rendering
import DOMPurify from 'dompurify';

function Comment({ html }) {
  const sanitized = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
    ALLOWED_ATTR: ['href']
  });
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}

// ✅ BETTER: Use markdown renderer with safe options
import ReactMarkdown from 'react-markdown';

function Comment({ markdown }) {
  return <ReactMarkdown>{markdown}</ReactMarkdown>;
}
```

### URL Handling

```javascript
// ❌ DANGEROUS: Unvalidated URLs
<a href={userProvidedUrl}>Click here</a>
// If url = 'javascript:stealCookies()', XSS!

// ✅ SAFE: Validate URL protocol
function SafeLink({ href, children }) {
  const url = new URL(href, window.location.origin);
  const isAllowed = ['http:', 'https:', 'mailto:'].includes(url.protocol);
  
  return isAllowed ? <a href={href}>{children}</a> : null;
}
```

## 14.2 CSRF Protection

```javascript
// Include CSRF token in forms/requests
function Form() {
  const csrfToken = document.querySelector('meta[name="csrf-token"]').content;
  
  return (
    <form method="POST" action="/api/submit">
      <input type="hidden" name="_csrf" value={csrfToken} />
      {/* form fields */}
    </form>
  );
}

// For fetch requests
async function submitData(data) {
  const csrfToken = document.querySelector('meta[name="csrf-token"]').content;
  
  await fetch('/api/submit', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken
    },
    credentials: 'same-origin',
    body: JSON.stringify(data)
  });
}
```

## 14.3 Sensitive Data Handling

```javascript
// ❌ BAD: Exposing sensitive data in client state
function UserProfile({ user }) {
  // user.ssn, user.creditCard visible in React DevTools
  return <div>{user.name}</div>;
}

// ✅ GOOD: Only fetch/store necessary data
async function fetchUserForDisplay(userId) {
  const response = await fetch(`/api/users/${userId}/public`);
  return response.json();  // Server returns only public fields
}

// Never log sensitive data
console.log(user);  // Could include passwords in dev!
```

## 14.4 Content Security Policy

```html
<!-- In HTML head or server response headers -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline'; 
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:;">
```

---

# 15. React 18/19 Features

## 15.1 React 18 Features

### Automatic Batching

```javascript
// React 17: Only batched in React event handlers
// React 18: Batches everywhere

// Before React 18
setTimeout(() => {
  setCount(c => c + 1);  // Re-render
  setFlag(f => !f);      // Re-render
  // 2 renders
}, 100);

// React 18
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // 1 render (batched!)
}, 100);

// Opt-out of batching (rare)
import { flushSync } from 'react-dom';

flushSync(() => {
  setCount(c => c + 1);
});
// DOM updated here
flushSync(() => {
  setFlag(f => !f);
});
// DOM updated here
```

### useTransition & useDeferredValue

(Covered in Section 9)

### Suspense Improvements

```javascript
// React 18: Suspense works with SSR, streaming, selective hydration
// React 17: Suspense only worked with React.lazy

// Suspense for data fetching (with compatible libraries)
<Suspense fallback={<Loading />}>
  <ProfilePage />  {/* Uses Suspense-enabled data fetching */}
</Suspense>
```

### New Client/Server Rendering APIs

```javascript
// Client (React 18)
import { createRoot, hydrateRoot } from 'react-dom/client';

// SPA rendering
const root = createRoot(document.getElementById('root'));
root.render(<App />);

// SSR hydration
hydrateRoot(document.getElementById('root'), <App />);

// Server streaming
import { renderToPipeableStream } from 'react-dom/server';
const stream = renderToPipeableStream(<App />, options);
```

## 15.2 React 19 Features

### use() Hook

```javascript
// use() can read context and promises
import { use } from 'react';

function Comments({ commentsPromise }) {
  // Suspends until promise resolves
  const comments = use(commentsPromise);
  
  return comments.map(c => <Comment key={c.id} data={c} />);
}

// use() with context (can be called conditionally!)
function Panel({ theme }) {
  if (theme === 'dark') {
    const settings = use(DarkThemeContext);  // OK in React 19!
  }
}
```

### Actions

```javascript
// Form actions - run async functions on submit
async function submitForm(formData) {
  'use server';  // Server Action
  const name = formData.get('name');
  await db.insert({ name });
}

function Form() {
  return (
    <form action={submitForm}>
      <input name="name" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### useActionState (formerly useFormState)

```javascript
import { useActionState } from 'react';

async function createUser(prevState, formData) {
  const name = formData.get('name');
  if (!name) return { error: 'Name required' };
  
  await db.insert({ name });
  return { success: true };
}

function Form() {
  const [state, formAction, isPending] = useActionState(createUser, null);
  
  return (
    <form action={formAction}>
      <input name="name" disabled={isPending} />
      <button disabled={isPending}>
        {isPending ? 'Submitting...' : 'Submit'}
      </button>
      {state?.error && <p className="error">{state.error}</p>}
    </form>
  );
}
```

### useOptimistic

```javascript
import { useOptimistic } from 'react';

function TodoList({ todos, addTodo }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, { ...newTodo, pending: true }]
  );
  
  const handleSubmit = async (formData) => {
    const newTodo = { text: formData.get('text'), id: Date.now() };
    addOptimisticTodo(newTodo);  // Immediately show
    await addTodo(newTodo);      // Server call
    // When complete, 'todos' prop updates, removing 'pending'
  };
  
  return (
    <form action={handleSubmit}>
      <input name="text" />
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
            {todo.text}
          </li>
        ))}
      </ul>
    </form>
  );
}
```

### Document Metadata

```javascript
// React 19: Components can render to <head>
function BlogPost({ post }) {
  return (
    <article>
      <title>{post.title}</title>
      <meta name="description" content={post.excerpt} />
      <h1>{post.title}</h1>
      {/* content */}
    </article>
  );
}
// <title> and <meta> automatically hoisted to <head>
```

## 15.3 Version Comparison

| Feature | React 17 | React 18 | React 19 |
|---------|----------|----------|----------|
| Concurrent Features | No | Yes | Enhanced |
| Automatic Batching | Event handlers only | Everywhere | Everywhere |
| Suspense for Data | No | Yes (limited) | Full support |
| SSR Streaming | No | Yes | Improved |
| use() hook | No | No | Yes |
| Actions | No | No | Yes |
| useOptimistic | No | No | Yes |
| Server Components | Experimental | Stable | Enhanced |

---

# 16. Interview Questions Bank

## 16.1 Conceptual Questions

**Q: What is the difference between Real DOM and Virtual DOM?**

| Real DOM | Virtual DOM |
|----------|-------------|
| Direct representation of UI in browser | JavaScript object representation |
| Updates are expensive (reflow/repaint) | Updates are cheap (just objects) |
| Can update any part | Changes batched and optimized |
| Used by browser | Used by React internally |

**Q: Explain React's unidirectional data flow.**

Data flows down from parent to children via props. Children communicate up via callbacks. This makes data flow predictable and debugging easier.

```
Parent State
    ↓ props
   Child
    ↓ callback
Parent Handler → Updates State → Re-renders
```

**Q: What is reconciliation?**

Process of comparing the new element tree with the previous one to determine minimum changes needed. Uses two heuristics:
1. Different element types create different trees
2. Keys identify stable elements across renders

## 16.2 Hooks Questions

**Q: Implement useDebounce**

```javascript
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}
```

**Q: Implement useThrottle**

```javascript
function useThrottle(value, limit) {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastRan = useRef(Date.now());
  
  useEffect(() => {
    const handler = setTimeout(() => {
      if (Date.now() - lastRan.current >= limit) {
        setThrottledValue(value);
        lastRan.current = Date.now();
      }
    }, limit - (Date.now() - lastRan.current));
    
    return () => clearTimeout(handler);
  }, [value, limit]);
  
  return throttledValue;
}
```

**Q: Implement usePrevious**

```javascript
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => { ref.current = value; });
  return ref.current;
}
```

## 16.3 Performance Questions

**Q: How would you optimize a slow React application?**

1. **Profile first**: Use React DevTools Profiler, identify slow components
2. **Reduce renders**: React.memo, useMemo, useCallback where appropriate
3. **Virtualize lists**: react-window for long lists
4. **Code split**: React.lazy for route-based splitting
5. **Optimize data**: Move to server state libraries (React Query)
6. **Reduce bundle**: Tree shaking, dynamic imports

**Q: When would React.memo be counterproductive?**

- Component already renders fast
- Props change on almost every parent render
- The comparison function is more expensive than re-rendering
- Simple primitive props (comparison overhead > render time)

## 16.4 Architecture Questions

**Q: How do you decide between Context, Redux, and local state?**

- **Local state**: UI-specific, doesn't need sharing
- **Lifted state**: Shared by nearby siblings
- **Context**: Infrequent changes, prop drilling avoidance (theme, auth)
- **Redux/Zustand**: Complex state logic, many consumers, devtools need

**Q: Design a notification system component**

```javascript
const NotificationContext = createContext();

function NotificationProvider({ children }) {
  const [notifications, setNotifications] = useState([]);
  
  const addNotification = useCallback((message, type = 'info') => {
    const id = Date.now();
    setNotifications(prev => [...prev, { id, message, type }]);
    setTimeout(() => removeNotification(id), 5000);
  }, []);
  
  const removeNotification = useCallback((id) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  }, []);
  
  return (
    <NotificationContext.Provider value={{ addNotification }}>
      {children}
      <NotificationContainer 
        notifications={notifications} 
        onRemove={removeNotification} 
      />
    </NotificationContext.Provider>
  );
}

const useNotification = () => useContext(NotificationContext);
```

---

# 17. Cheat Sheet

## Quick Reference: Hooks

```javascript
// State
const [state, setState] = useState(initialValue);
const [state, setState] = useState(() => computeExpensive());

// Effects
useEffect(() => { /* effect */ }, [deps]);        // After paint
useEffect(() => () => cleanup, [deps]);           // With cleanup
useLayoutEffect(() => { /* sync effect */ }, []); // Before paint

// Refs
const ref = useRef(initialValue);
const inputRef = useRef(null); <input ref={inputRef} />

// Performance
const memoized = useMemo(() => compute(a, b), [a, b]);
const handler = useCallback((e) => handle(e, id), [id]);

// Reducer
const [state, dispatch] = useReducer(reducer, initialState);

// Context
const value = useContext(MyContext);

// Concurrent
const [isPending, startTransition] = useTransition();
const deferredValue = useDeferredValue(value);

// Identity
const id = useId();

// React 19
const value = use(promiseOrContext);
const [state, action, pending] = useActionState(fn, initialState);
const [optimistic, addOptimistic] = useOptimistic(state, reducer);
```

## Quick Reference: Component Patterns

```javascript
// Memo
const MemoComp = React.memo(Component);
const MemoComp = React.memo(Component, (prev, next) => prev.id === next.id);

// Lazy
const LazyComp = React.lazy(() => import('./Component'));
<Suspense fallback={<Loading />}><LazyComp /></Suspense>

// Error Boundary
class ErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) { return { hasError: true }; }
  componentDidCatch(error, info) { logError(error, info); }
  render() { return this.state.hasError ? <Fallback /> : this.props.children; }
}

// Portal
ReactDOM.createPortal(children, domNode);

// Forward Ref
const Input = forwardRef((props, ref) => <input ref={ref} {...props} />);
```

## Quick Reference: Performance

| Problem | Solution |
|---------|----------|
| Expensive computation | `useMemo` |
| Callback causing child re-render | `useCallback` + `React.memo` |
| Large list | `react-window` virtualization |
| Large bundle | `React.lazy` + route splitting |
| Prop drilling | Context or composition |
| Server data re-fetching | React Query / SWR |

## Mnemonics

**REPS for Hook Dependencies:**
- **R**efs don't need to be deps (stable)
- **E**xternal constants don't need deps
- **P**rops usually need to be deps
- **S**etters from useState don't need deps (stable)

**FISH for Re-render Causes:**
- **F**orce update called
- **I**nternal state changed (setState)
- **S**uper (parent) re-rendered
- **H**ook value changed (useContext)

**CUP for Error Boundary Gaps:**
- **C**lick handlers (event handlers)
- **U**nresolved promises (async code)
- **P**roblem in boundary itself

---

## Version History Quick Reference

| React Version | Release | Key Features |
|---------------|---------|--------------|
| 16.0 | Sep 2017 | Fiber, Fragments, Portals, Error Boundaries |
| 16.3 | Mar 2018 | Context API, createRef, forwardRef |
| 16.6 | Oct 2018 | React.lazy, React.memo, Suspense (lazy) |
| 16.8 | Feb 2019 | Hooks |
| 17.0 | Oct 2020 | No new features (gradual upgrades) |
| 18.0 | Mar 2022 | Concurrent Features, useTransition, Suspense (data) |
| 19.0 | 2024 | use(), Actions, useOptimistic, Server Components stable |

---

## Common Interview Traps

1. **"Virtual DOM is faster than real DOM"** → Wrong. It's an abstraction for declarative programming, not a performance optimization.

2. **"useEffect runs before render"** → Wrong. Effect runs after paint. useLayoutEffect runs after DOM mutation but before paint.

3. **"React re-renders when props change"** → Partially wrong. React re-renders when PARENT re-renders, regardless of prop changes.

4. **"memo prevents all re-renders"** → Wrong. Only prevents re-renders from unchanged props. Context/internal state still cause re-renders.

5. **"Functional components are faster"** → No inherent difference. Class and function components reconcile the same way.

---

*Document Version: March 2026*
*Target: Mid-Senior React/Full-Stack Interview Preparation*
*Coverage: React 18/19, Modern Patterns, Performance, Testing*
