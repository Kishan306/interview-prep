# The Complete React Interview Prep Sheet
### Beginner → Advanced+ | Questions & Detailed Answers

---

## TABLE OF CONTENTS

1. [Phase 1 — Fundamentals (Beginner)](#phase-1)
2. [Phase 2 — Components, Props & State](#phase-2)
3. [Phase 3 — Hooks Deep Dive](#phase-3)
4. [Phase 4 — Intermediate Patterns](#phase-4)
5. [Phase 5 — Performance & Optimization](#phase-5)
6. [Phase 6 — Internals: Reconciliation, Fiber & Rendering](#phase-6)
7. [Phase 7 — Concurrent React & React 18/19](#phase-7)
8. [Phase 8 — SSR, RSC, Hydration & Frameworks](#phase-8)
9. [Phase 9 — State Management Ecosystem](#phase-9)
10. [Phase 10 — Routing, Data Fetching & Forms](#phase-10)
11. [Phase 11 — TypeScript with React](#phase-11)
12. [Phase 12 — Testing](#phase-12)
13. [Phase 13 — Tooling, Build & Security](#phase-13)
14. [Phase 14 — Design Patterns & Architecture](#phase-14)
15. [Phase 15 — Tricky/Gotcha & Rapid-Fire Round](#phase-15)

---

<a name="phase-1"></a>
## PHASE 1 — FUNDAMENTALS (BEGINNER)

**1. What is React and what problem does it solve?**
React is an open-source JavaScript library (not a full framework) for building user interfaces, maintained by Meta. Its core idea is to let you build UIs as a composition of **components** whose output is a pure function of their **state** and **props**. The central problem it solves is keeping the UI in sync with constantly changing data without manual, error-prone DOM manipulation. Instead of telling the browser *how* to update the DOM step by step (imperative), you declare *what* the UI should look like for a given state (declarative), and React figures out the minimal set of DOM operations needed.

**2. Is React a library or a framework? Why does the distinction matter?**
React is a **library** focused on the view layer. It deliberately leaves routing, data fetching, global state, and build tooling to the ecosystem (React Router, TanStack Query, Redux, Vite, etc.). This matters in interviews because it shows you understand React makes few decisions for you — you assemble a stack. Frameworks like Next.js wrap React to provide those missing pieces (routing, SSR, bundling) opinionatedly.

**3. What is the Virtual DOM and how does it work?**
The Virtual DOM (VDOM) is a lightweight in-memory JavaScript representation of the actual DOM — a tree of plain objects describing what the UI should look like. When state changes, React builds a new VDOM tree, **diffs** it against the previous one (reconciliation), computes the minimal set of changes, and applies only those to the real DOM in a batch. The real DOM is slow to mutate; comparing JS objects is cheap. Important nuance: the VDOM isn't "faster than the DOM" in absolute terms — it's a means to *minimize and batch* expensive real-DOM writes, and to give React a declarative programming model.

**4. What is the difference between the Real DOM and the Virtual DOM?**
The Real DOM is the browser's object representation of the page; mutating it triggers reflow/repaint and is comparatively expensive. The Virtual DOM is a JS object tree React keeps in memory. React mutates the VDOM freely, diffs old vs new, then flushes the smallest possible set of mutations to the Real DOM. The VDOM never "replaces" the real DOM — it's a staging layer.

**5. What is JSX? Is it required?**
JSX is a syntax extension to JavaScript that lets you write HTML-like markup inside JS. It is **not** required — it's syntactic sugar. Under the hood, `<div className="x">Hi</div>` compiles (via Babel or the modern JSX transform) to `React.createElement('div', { className: 'x' }, 'Hi')` (or `_jsx(...)` with the automatic runtime in React 17+). JSX is preferred because it's far more readable and keeps markup and logic colocated.

**6. Why `className` instead of `class`, and `htmlFor` instead of `for`?**
Because JSX compiles to JavaScript, and `class` and `for` are reserved words in JS. React uses the DOM property names (`className`, `htmlFor`) instead of the HTML attribute names. Most other attributes use camelCase (`onClick`, `tabIndex`, `readOnly`).

**7. What are the rules of JSX?**
- Must return a **single root element** (use a Fragment `<>...</>` to avoid extra DOM nodes).
- All tags must be **closed** (`<img />`, `<br />`).
- Use camelCase for most attributes.
- Embed JS expressions with `{}` (expressions only — no `if` statements or loops directly, though ternaries and `.map()` work).
- Comments inside JSX use `{/* ... */}`.

**8. What's the difference between an expression and a statement in JSX?**
Only **expressions** (things that produce a value) can go inside `{}`. `cond ? a : b`, `arr.map(...)`, `a && b`, function calls, and variables are expressions. `if`, `for`, `switch`, and variable declarations are statements and cannot be inlined — you handle those before the `return` or extract them into helper functions/variables.

**9. What is a React component?**
A component is a reusable, self-contained piece of UI — a JavaScript function (or class) that returns React elements describing what should appear on screen. Components can accept inputs (props) and manage their own internal data (state). The two types are **functional components** (the modern standard) and **class components** (legacy).

**10. What's the difference between a component and an element?**
An **element** is a plain immutable object describing what you want to see (`{ type: 'div', props: {...} }`) — the smallest building block, the return of `createElement`. A **component** is a function/class that *returns* elements. Elements are cheap to create and are what React diffs; components are the templates that produce them.

**11. Functional vs Class components — what's the difference?**
Class components extend `React.Component`, use `this.state`/`this.setState`, lifecycle methods (`componentDidMount`, etc.), and a `render()` method. Functional components are plain functions that return JSX and use **Hooks** for state and side effects. Since React 16.8, functional components can do everything class components can, and they're now the recommended default: less boilerplate, no `this` confusion, better logic reuse via custom hooks, and better compatibility with future React features.

**12. What are props?**
Props ("properties") are read-only inputs passed from a parent to a child component, like function arguments. They enable data flow and configuration. Props are **immutable** within the receiving component — a child must never mutate its props. To change data, the owner of the state updates it and re-passes new props down.

**13. What is the children prop?**
`props.children` is a special prop containing whatever you nest between a component's opening and closing tags: `<Card>Hello</Card>` makes `"Hello"` the `children`. It enables composition — wrapper/layout components that don't know their content ahead of time. `children` can be a string, element, array of elements, a function (render props), or `null`.

**14. What is state?**
State is data that is private to a component and changes over time, usually in response to user interaction or async events. Unlike props (passed in, read-only), state is owned and managed internally. When state changes, React re-renders the component. In functional components you create state with `useState`/`useReducer`.

**15. Props vs State — summarize the differences.**
Props are passed in from a parent and are read-only to the receiver; state is internal and mutable (via setters). Props enable parent→child communication; state enables a component to remember and react to changes. A change to either triggers a re-render. A common rule: if a value can be derived from props or other state, it probably shouldn't be its own state.

**16. How do you render a list in React, and why are keys needed?**
You map over an array and return an element per item: `items.map(item => <li key={item.id}>{item.text}</li>)`. **Keys** give each list item a stable identity so React can match elements between renders during reconciliation — knowing which items were added, removed, or reordered. Without correct keys, React may re-use the wrong DOM nodes, causing bugs (e.g., input values attaching to the wrong row) and unnecessary re-renders.

**17. Why is using array index as a key problematic?**
Index keys are fine *only* for static lists that never reorder, filter, or change length. If the list can change order or have items inserted/removed, the index no longer identifies the *same item* across renders — React reuses state/DOM for what it thinks is the same position, causing subtle bugs (wrong checkbox checked, stale input text) and broken animations. Always prefer a stable unique ID from your data.

**18. How do you conditionally render in React?**
Several idioms: a ternary `{isLoggedIn ? <Dashboard/> : <Login/>}`; logical AND `{hasError && <Error/>}` (careful — `0 && <X/>` renders `0`!); early `return null` to render nothing; or assigning JSX to a variable with `if/else` before the return. Returning `null`, `false`, `undefined`, or `true` renders nothing.

**19. What's the gotcha with `{count && <Component/>}`?**
If `count` is `0`, `0 && <Component/>` evaluates to `0`, and React **renders the number 0** on screen (because `0` is a valid renderable value, unlike `false`/`null`). Fix it with an explicit boolean: `{count > 0 && <Component/>}` or `{!!count && ...}`.

**20. How does event handling work in React?**
You attach handlers via camelCase props: `<button onClick={handleClick}>`. You pass a **function reference**, not a call (`onClick={handleClick}` not `onClick={handleClick()}`). React uses a **synthetic event** system — a cross-browser wrapper around native events — and (in React 17+) attaches a single delegated listener at the root container rather than on each node, for performance.

**21. What is a SyntheticEvent?**
`SyntheticEvent` is React's normalized wrapper around the browser's native event, giving a consistent API across browsers. It exposes `preventDefault()`, `stopPropagation()`, `target`, etc. You can reach the raw native event via `e.nativeEvent`. (Pre-React 17, synthetic events were pooled and reused, so you had to call `e.persist()` for async access — that pooling was removed in React 17.)

**22. How do you pass arguments to an event handler?**
Wrap it in an arrow function: `onClick={() => handleClick(id)}`. Avoid `onClick={handleClick(id)}` — that *calls* the function during render. Be aware inline arrows create a new function each render (usually negligible, but relevant when passing to memoized children).

**23. Controlled vs Uncontrolled components — what's the difference?**
A **controlled** component has its value driven by React state: `<input value={text} onChange={e => setText(e.target.value)} />`. React is the single source of truth. An **uncontrolled** component lets the DOM hold its own state, and you read it via a `ref` when needed (`<input ref={inputRef} defaultValue="x" />`). Controlled is preferred for validation, conditional disabling, and predictable data flow; uncontrolled is simpler for basic forms and integrating non-React widgets.

**24. What is one-way data binding / unidirectional data flow?**
Data in React flows in **one direction**: from parent down to child via props. State lives at some level and flows downward; children communicate upward by calling callback functions passed as props. This makes data flow predictable and debuggable — you always know where state lives and what can change it. (Two-way binding, as in Angular, is simulated via the value+onChange pattern.)

**25. What does it mean that React is "declarative"?**
You describe *what* the UI should look like for the current state, and React handles *how* to update the DOM to match. Contrast with imperative code where you manually query and mutate DOM nodes step by step. Declarative code is easier to reason about, test, and maintain because you remove the "transition" logic.

**26. What is "lifting state up"?**
When two sibling components need to share or sync state, you move ("lift") that state to their closest common ancestor and pass it down as props, with callbacks to update it. This keeps a single source of truth and avoids trying to sync duplicated state.

**27. What is prop drilling and why is it a problem?**
Prop drilling is passing props through many intermediate components that don't use them, just to reach a deeply nested consumer. It bloats component signatures, couples components, and makes refactoring painful. Solutions: Context API, composition (passing components as children), or a state-management library.

**28. What is a Fragment and why use it?**
A Fragment (`<React.Fragment>` or shorthand `<>...</>`) lets you group multiple children without adding an extra DOM node. Useful when a component must return multiple elements but you don't want a wrapper `<div>` polluting the DOM (which can break flex/grid layouts or table structure). Only the long form `<React.Fragment key={...}>` accepts a `key`.

**29. What are keys and where else besides lists do they matter?**
Beyond lists, changing a component's `key` forces React to **unmount and remount** it (discarding its state) instead of updating in place. This is a powerful trick to reset a component's state — e.g., `<Form key={userId} />` gives a fresh form when the user changes.

**30. What happens when you call a component vs render it as JSX?**
`<MyComp />` tells React to treat it as a component instance — React manages its lifecycle, state, and hooks. Calling `MyComp()` directly just runs the function and inlines its output as if it were part of the parent — it gets no own state, no hooks isolation, no reconciliation identity. Always render components as elements.

---

<a name="phase-2"></a>
## PHASE 2 — COMPONENTS, PROPS & STATE

**31. How does `useState` work and what does it return?**
`useState(initialValue)` returns a pair: `[currentValue, setterFunction]`. Calling the setter with a new value schedules a re-render; on the next render `useState` returns the updated value. State is preserved across re-renders because React stores it internally, keyed by the order hooks are called.

**32. Why is state update asynchronous / "batched"?**
`setState` doesn't mutate immediately; it schedules an update. React **batches** multiple state updates that occur in the same event handler into a single re-render for performance. So reading state right after setting it gives the old value. In React 18, batching was extended to apply everywhere (promises, setTimeout, native handlers) via automatic batching.

**33. What is the functional updater form of a setter and when must you use it?**
`setCount(prev => prev + 1)`. You pass a function that receives the latest pending state. Use it whenever the new state depends on the previous state, especially with multiple updates in one tick: `setCount(c => c+1); setCount(c => c+1)` increments by 2, whereas `setCount(count+1); setCount(count+1)` increments by 1 (both read the same stale `count`).

**34. Why should you never mutate state directly?**
React decides whether to re-render by comparing references (`Object.is`). If you mutate an existing object/array (`state.x = 1; setState(state)`), the reference is unchanged, so React may skip the re-render, and you also break time-travel/debugging guarantees. Always create a **new** object/array: `setItems([...items, newItem])`, `setUser({...user, name})`.

**35. How do you update nested state immutably?**
Spread at each level you change: `setState(s => ({ ...s, address: { ...s.address, city } }))`. For deep structures this gets verbose — use Immer (`produce`) or `useImmer` to write "mutating" code that produces immutable updates under the hood.

**36. What is lazy initialization of state?**
If computing the initial state is expensive, pass a **function** to `useState`: `useState(() => expensiveCompute())`. React calls it only on the first render, not on every render. Passing the value directly (`useState(expensiveCompute())`) would run the expensive computation on every render even though only the first one is used.

**37. Can you have multiple `useState` calls? Should you combine state?**
Yes — multiple independent `useState` calls are idiomatic and often clearer than one big object. Combine related values that change together; keep unrelated values separate. Over-combining forces you to spread on every update and can cause unnecessary coupling; over-splitting can scatter related logic. Use `useReducer` when several state values change together with complex logic.

**38. What is `useReducer` and when do you prefer it over `useState`?**
`useReducer(reducer, initialState)` returns `[state, dispatch]`. You centralize update logic in a pure reducer `(state, action) => newState` and trigger updates via `dispatch({ type, payload })`. Prefer it when: state is complex/nested, the next state depends on intricate logic, multiple values update together, or you want to decouple *what happened* (action) from *how state changes* (reducer). It also makes state transitions testable in isolation.

**39. What are the lifecycle phases of a React component?**
Three phases: **Mounting** (component created and inserted into the DOM), **Updating** (re-renders due to prop/state changes), and **Unmounting** (removed from the DOM). In class components these map to `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`. In functional components, `useEffect` covers all three depending on its dependency array and cleanup function.

**40. Map class lifecycle methods to hooks.**
- `componentDidMount` → `useEffect(() => {...}, [])`
- `componentDidUpdate` → `useEffect(() => {...}, [deps])`
- `componentWillUnmount` → cleanup function returned from `useEffect`
- `getDerivedStateFromProps` → compute during render or adjust state with the key trick / derived value
- `shouldComponentUpdate` → `React.memo` + `useMemo`/`useCallback`
- `getSnapshotBeforeUpdate` → `useLayoutEffect` (reading layout before paint)

**41. What were the legacy/deprecated lifecycle methods and why?**
`componentWillMount`, `componentWillReceiveProps`, and `componentWillUpdate` were deprecated (prefixed `UNSAFE_`) because they were unreliable with async/concurrent rendering — they could be called multiple times or with stale assumptions, leading to bugs. They were replaced by `getDerivedStateFromProps` and `getSnapshotBeforeUpdate`.

**42. What is `defaultProps` and how is it handled now?**
`defaultProps` provided fallback values when a prop wasn't passed. For functional components, the modern approach is **default parameter values**: `function Btn({ size = 'md' }) {...}`. `Component.defaultProps` on function components is deprecated in React 19 in favor of default parameters.

**43. What is `PropTypes`?**
`PropTypes` is a runtime type-checking utility (separate `prop-types` package) that validates the types of props in development, warning in the console on mismatches. It's largely superseded by **TypeScript**, which checks types at compile time with far better coverage. Mentioning you prefer TS here signals seniority.

**44. What does it mean for a component to be "pure"?**
A pure component renders the same output given the same props and state, and has no side effects during rendering. React assumes purity — rendering should be a pure calculation. Side effects (fetching, subscriptions, timers, DOM mutations) belong in event handlers or effects, never in the render body. `React.StrictMode` double-invokes renders in dev to surface impurities.

**45. What is `React.PureComponent` / `React.memo`?**
`PureComponent` (class) implements `shouldComponentUpdate` with a **shallow** comparison of props and state, skipping re-render when nothing shallowly changed. `React.memo` is the functional equivalent — it memoizes a component, re-rendering only when props shallowly change. Both help avoid unnecessary re-renders but rely on stable prop references (hence `useCallback`/`useMemo`).

**46. What's a "render" vs a "commit" in React?**
The **render phase** is when React calls your components to compute the new VDOM and diffs it — pure, no side effects, can be paused/restarted/aborted (in concurrent mode). The **commit phase** is when React applies the computed changes to the real DOM and runs layout effects — synchronous and cannot be interrupted. Understanding this split is key to explaining `useEffect` vs `useLayoutEffect` and concurrent features.

**47. What causes a component to re-render?**
A component re-renders when: (1) its own state changes, (2) its parent re-renders (by default, regardless of prop changes — unless memoized), (3) a context it consumes changes value, or (4) a `useSyncExternalStore`/subscription it reads updates. Note: props changing doesn't *directly* trigger a re-render; the parent re-rendering does.

**48. If a parent re-renders, do all children re-render?**
By default, yes — React re-renders the entire subtree below a re-rendering component, even children whose props didn't change. This is usually fine (rendering is cheap; only DOM diffs commit). To skip it for expensive children, wrap them in `React.memo` and ensure stable prop references, or restructure with composition (`children`) so the expensive subtree isn't re-created.

**49. How can passing `children` avoid re-renders?**
If a component holds frequently-changing state but receives an expensive subtree as `children`, that subtree is created by the *parent* and passed in as a stable element. When the wrapper re-renders due to its own state, the `children` element reference is unchanged, so React skips re-rendering it. This is the "components as props" optimization, no memo needed.

**50. What is a "controlled vs uncontrolled" component at the form level — and how do you read uncontrolled values?**
Controlled: every input's value lives in state. Uncontrolled: inputs keep their own DOM state; you attach a `ref` and read `ref.current.value` on submit, optionally seeding with `defaultValue`/`defaultChecked`. Uncontrolled is lighter (no re-render per keystroke) and good for simple or performance-sensitive forms and file inputs (`<input type="file">` is always uncontrolled).

---

<a name="phase-3"></a>
## PHASE 3 — HOOKS DEEP DIVE

**51. What are Hooks and why were they introduced?**
Hooks are functions (prefixed `use`) that let functional components "hook into" React features — state, lifecycle, context — without classes. Introduced in React 16.8 to solve: (1) reusing stateful logic was hard (HOCs/render-props caused "wrapper hell"), (2) complex components became unwieldy with related logic split across lifecycle methods, and (3) classes confused both people and machines (`this`, binding, no good minification/hot-reload). Hooks let you organize code by concern and extract reusable logic into custom hooks.

**52. What are the Rules of Hooks?**
(1) **Only call hooks at the top level** — never inside loops, conditions, or nested functions. (2) **Only call hooks from React functions** — components or custom hooks, not regular JS functions. The reason: React tracks hooks by **call order**, not by name. If the order changes between renders (e.g., a conditional hook), React mismatches state to hooks, corrupting everything. The ESLint plugin `eslint-plugin-react-hooks` enforces these.

**53. Why can't hooks be called conditionally?**
React maintains an internal list of hooks per component, indexed by the order they're called. On each render it walks that list in the same order to return the right state. A conditional hook would shift subsequent hooks' positions, so React would hand the wrong state to the wrong hook. Consistent call order every render is mandatory.

**54. Explain `useEffect` in depth.**
`useEffect(setup, deps)` runs side effects *after* render is committed to the screen. The `setup` function can return a `cleanup` function. Behavior by deps:
- No deps array → runs after **every** render.
- `[]` → runs **once** after mount; cleanup runs on unmount.
- `[a, b]` → runs after mount and whenever `a` or `b` change (by `Object.is`); cleanup runs before the next effect and on unmount.
Use it for synchronizing with external systems: subscriptions, timers, manual DOM, network requests, logging.

**55. What is the cleanup function and when does it run?**
The function returned from `useEffect` cleans up the previous effect. It runs (1) before the effect re-runs due to changed deps, and (2) when the component unmounts. Use it to unsubscribe, clear timers, abort fetches, remove listeners — preventing memory leaks and stale subscriptions.

**56. What's the difference between `useEffect` and `useLayoutEffect`?**
`useEffect` runs **asynchronously after the browser paints** — non-blocking, good for most side effects. `useLayoutEffect` runs **synchronously after DOM mutations but before paint** — it can read layout (measurements) and mutate the DOM without the user seeing a flicker. Use `useLayoutEffect` only when you must measure or adjust DOM before the browser paints (tooltips positioning, scroll restoration); otherwise prefer `useEffect` to avoid blocking paint.

**57. When does `useEffect` run relative to the browser paint?**
Render → commit (DOM updated) → browser paints → `useEffect` runs. `useLayoutEffect` runs between commit and paint. This is why a layout effect can prevent visual flicker but can also delay paint if it does heavy work.

**58. What are common `useEffect` mistakes?**
- Missing dependencies → stale closures (reads old props/state).
- Putting derived data in effects instead of computing during render.
- Forgetting cleanup → leaks, duplicate listeners.
- Using effects to sync state that could be derived → unnecessary renders and bugs.
- Object/function/array deps recreated every render → effect re-runs every time.
- Fetching without an abort/ignore flag → race conditions where a stale response overwrites a newer one.

**59. How do you avoid race conditions in data-fetching effects?**
Use an `ignore`/`isMounted` flag or an `AbortController`:
```js
useEffect(() => {
  let ignore = false;
  fetchData(id).then(res => { if (!ignore) setData(res); });
  return () => { ignore = true; };
}, [id]);
```
This ensures a late-arriving response for an old `id` doesn't overwrite the current one. In real apps, prefer a data library (TanStack Query) that handles this for you.

**60. What is a stale closure and how does it bite you?**
A closure captures variables from the render in which it was created. If an effect/callback with an empty deps array references state, it forever "sees" the initial value. Example: a `setInterval` set up once that logs `count` always logs the initial `count`. Fixes: include the value in deps, use the functional updater (`setCount(c => c+1)`), or store the latest value in a ref.

**61. What is `useRef` and what are its two main uses?**
`useRef(initial)` returns a mutable object `{ current: initial }` that persists across renders **without** triggering a re-render when changed. Two uses: (1) **Accessing DOM nodes** — attach via `ref={myRef}` then read `myRef.current`. (2) **Storing mutable instance values** — timers, previous values, any mutable data you want to survive renders but not render from (like an instance variable in a class).

**62. What's the difference between `useRef` and `useState` for storing values?**
Updating `useState` triggers a re-render and the value is part of the render output. Updating `useRef.current` does **not** trigger a re-render and isn't reflected in the UI until something else renders. Use state for values the UI displays; use refs for values you need to remember but don't render (e.g., a `setInterval` ID, whether it's the first render).

**63. How do you access a child's DOM node from a parent?**
Pass a ref down. Pre-React 19 you needed `forwardRef`: `const Input = forwardRef((props, ref) => <input ref={ref} {...props} />)`. In **React 19**, `ref` can be passed as a regular prop to function components, so `forwardRef` is largely unnecessary.

**64. What is `forwardRef`?**
`forwardRef` lets a component receive a `ref` from its parent and forward it to a DOM element or another component inside it. Needed (pre-19) because refs aren't normal props. Common for reusable input/button libraries where consumers need direct DOM access. React 19 deprecates the need by treating `ref` as a normal prop.

**65. What is `useImperativeHandle`?**
Used with `forwardRef`, it lets a child **customize the ref value** exposed to the parent — exposing specific imperative methods instead of the raw DOM node: `useImperativeHandle(ref, () => ({ focus: () => inputRef.current.focus(), reset: () => ... }))`. Use sparingly; it's an escape hatch from declarative data flow, appropriate for focus/scroll/animation APIs.

**66. What is `useContext` and how does it work?**
`useContext(MyContext)` reads the current value of a context provided by the nearest `<MyContext.Provider value={...}>` above. It avoids prop drilling for "global-ish" data (theme, auth, locale). Any component calling `useContext` re-renders when the provider's `value` changes (by `Object.is`). Create context with `createContext(defaultValue)`.

**67. What's the performance pitfall of Context, and how do you fix it?**
**All** consumers re-render whenever the provider's `value` changes — and if you pass an inline object `value={{ user, setUser }}`, a new reference is created every render, re-rendering every consumer each time the provider renders. Fixes: memoize the value with `useMemo`; split contexts so unrelated data lives separately (e.g., separate "state" and "dispatch" contexts); or use a selector-based store (Zustand, Redux with `useSelector`, or `useSyncExternalStore`) for fine-grained subscriptions.

**68. What is `useMemo`?**
`useMemo(() => computeExpensive(a, b), [a, b])` memoizes the **result** of an expensive computation, recomputing only when a dependency changes. Use it to (1) avoid recomputing expensive derived values each render, and (2) keep a **stable reference** to objects/arrays passed as props to memoized children or used in other hooks' deps. Don't overuse it — memoization has its own cost and most computations are cheap.

**69. What is `useCallback`?**
`useCallback(fn, deps)` memoizes a **function reference**, returning the same function identity until deps change. `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`. Its main purpose is keeping callback identity stable so that `React.memo`-wrapped children don't re-render and so the function can be a stable dependency in other hooks/effects. Useless if the child isn't memoized.

**70. `useMemo` vs `useCallback` — the one-liner.**
`useMemo` memoizes a **value** (the result of calling the function); `useCallback` memoizes the **function itself**. `useCallback(fn, d)` === `useMemo(() => fn, d)`.

**71. Will `useMemo`/`useCallback` always prevent re-renders?**
No. They only provide stable references. A memoized callback prevents a child re-render *only if* that child is wrapped in `React.memo` and all its other props are also stable. By themselves they don't stop the current component from re-rendering. Overusing them can even hurt performance (extra bookkeeping). React 19's compiler aims to auto-memoize, reducing the need.

**72. What is a custom hook?**
A custom hook is a function whose name starts with `use` that calls other hooks to encapsulate and reuse stateful logic across components — e.g., `useFetch`, `useLocalStorage`, `useDebounce`, `useWindowSize`. Each component using the hook gets its **own isolated state**; hooks share logic, not state. This is React's primary mechanism for logic reuse, replacing HOCs/render props for most cases.

**73. Do two components using the same custom hook share state?**
No. Each call to a custom hook runs its internal hooks independently, creating separate state per component instance. Hooks share *behavior/logic*, not *data*. To share data, lift the state up or use context/a store.

**74. What is `useId`?**
`useId()` (React 18) generates a unique, stable ID that matches between server and client renders — essential for accessibility attributes (`htmlFor`/`id`, `aria-describedby`) without hydration mismatches. Don't use it for list keys.

**75. What is `useSyncExternalStore`?**
`useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot)` (React 18) is the official way to subscribe to **external stores** (Redux, Zustand, browser APIs) safely with concurrent rendering, avoiding tearing (inconsistent reads across components in one render). State libraries use it internally. You'd use it directly when integrating a custom external data source.

**76. What is `useDeferredValue`?**
`useDeferredValue(value)` (React 18) returns a possibly-deferred copy of a value, letting React keep showing the old value while a heavy re-render (driven by the new value) happens in the background at lower priority. Great for keeping an input responsive while an expensive filtered list lags slightly behind.

**77. What is `useTransition`?**
`useTransition()` returns `[isPending, startTransition]`. Wrapping a state update in `startTransition(() => setState(...))` marks it as a **non-urgent transition** — React can interrupt it to keep urgent updates (typing, clicks) responsive, and `isPending` lets you show a subtle loading state. Used for expensive UI updates like filtering large lists or tab switches.

**78. What is `useDebugValue`?**
`useDebugValue(value)` lets custom hooks display a label in React DevTools for easier debugging. Purely a developer-experience tool; no runtime effect in production.

**79. What is `useInsertionEffect`?**
`useInsertionEffect` (React 18) fires **before** any layout effects, intended for CSS-in-JS libraries to inject `<style>` tags before the DOM is read for layout. Library-author territory — you almost never use it in app code.

**80. Can you call hooks inside loops or conditions if you're "careful"?**
No — never. Even if you think you control the path, varying hook call order between renders breaks React's index-based hook tracking. Instead, lift the condition inside the hook (`useEffect(() => { if (cond) {...} })`) or split into separate components. This is non-negotiable and lint-enforced.

---

<a name="phase-4"></a>
## PHASE 4 — INTERMEDIATE PATTERNS

**81. What is a Higher-Order Component (HOC)?**
A HOC is a function that takes a component and returns a new enhanced component: `const withAuth = Comp => props => <Comp {...props} extra={...} />`. It's a pattern for reusing cross-cutting logic (auth, logging, data injection). Examples: `connect` from React-Redux, `withRouter`. Downsides: wrapper hell, prop collisions, indirection. Mostly replaced by custom hooks, but you should know it for legacy codebases.

**82. What is the render props pattern?**
A component takes a function as a prop (often `children`) and calls it with internal data, letting the consumer control rendering: `<Mouse>{({x, y}) => <p>{x},{y}</p>}</Mouse>`. It shares stateful logic without HOC nesting. Like HOCs, largely superseded by custom hooks, but still seen in libraries (older React-Router, Formik, Downshift).

**83. HOC vs Render Props vs Custom Hooks — which and when?**
All three solve logic reuse. **Custom hooks** are the modern default — no wrapper components, no prop collisions, clean composition. **Render props** are still useful when the reused logic must control *what gets rendered* in a flexible spot. **HOCs** persist in legacy code and some libs. In an interview, say: "I reach for custom hooks first; render props for render-controlling APIs; HOCs mainly for legacy or when wrapping is genuinely needed."

**84. What is the Compound Component pattern?**
Related components that share implicit state via context to work as a unit: `<Tabs><Tabs.List><Tabs.Tab/></Tabs.List><Tabs.Panel/></Tabs>`. The parent manages state; children consume it through context. It gives a flexible, declarative API (like HTML's `<select>`/`<option>`). Used by Radix, Reach UI, Headless UI.

**85. What is the Provider pattern?**
Wrapping part of the tree in a context provider to supply shared data/services (theme, auth, i18n, a store) to all descendants without prop drilling. The backbone of Redux's `<Provider>`, theme providers, etc.

**86. What is the Container/Presentational pattern?**
Separating components into **containers** (handle data/logic/state) and **presentational** components (pure UI, receive props, no logic). It improves testability and reuse. With hooks this split is less rigid — logic often lives in custom hooks instead — but the principle of separating concerns still applies.

**87. What are React Portals and when do you use them?**
`createPortal(child, domNode)` renders children into a DOM node **outside** the parent's DOM hierarchy while keeping them in the React tree (so context and events still work — events bubble through the React tree, not the DOM tree). Use for modals, tooltips, popovers, toasts that must escape parent `overflow:hidden`/`z-index`/`transform` stacking contexts.

**88. Do events bubble through portals normally?**
Yes — surprisingly. Even though the portal's DOM node is elsewhere, React propagates events along the **React component tree**, so a click inside a portal bubbles to ancestors in the JSX tree, not the DOM tree. This is a common gotcha and a great interview detail.

**89. What are Error Boundaries?**
Error boundaries are components that **catch JavaScript errors in their child tree during rendering, lifecycle, and constructors**, log them, and render a fallback UI instead of crashing the whole app. They're implemented with class components using `static getDerivedStateFromError` (to render fallback) and `componentDidCatch` (to log). There is **no hook equivalent yet** — you use a class or the `react-error-boundary` library.

**90. What do error boundaries NOT catch?**
They don't catch errors in: event handlers (use try/catch there), asynchronous code (`setTimeout`, promises), server-side rendering, and errors thrown in the boundary itself. They only catch errors during rendering of their descendants.

**91. What is `React.lazy` and code splitting?**
`React.lazy(() => import('./Comp'))` lets you load a component's code **on demand** (dynamic import), splitting it into a separate bundle. Combined with `<Suspense fallback={...}>`, React shows a fallback while the chunk loads. This reduces initial bundle size and improves load time — split by route or heavy feature.

**92. What is `Suspense`?**
`<Suspense fallback={<Spinner/>}>` declaratively handles components that "suspend" (aren't ready to render yet) — lazy-loaded components, and (with frameworks/React 18+) async data fetching. While a child suspends, the nearest Suspense boundary shows the fallback, then swaps in real content when ready. It enables coordinated loading states and streaming SSR.

**93. How do you reset an error boundary?**
Give it a `key` that changes (remounting it), or use `react-error-boundary`'s `resetKeys`/`onReset` to clear the error state and retry. A bare error boundary stays in the fallback state until remounted.

**94. What is the difference between composition and inheritance in React?**
React strongly favors **composition** over inheritance. Instead of extending components, you compose them via `children`, props, and slots. Need to specialize a component? Pass different props/children. React's docs explicitly recommend never using inheritance for component reuse — composition covers all cases more flexibly.

**95. What are "slots" in React?**
Passing elements as named props to fill specific regions: `<Layout header={<Header/>} sidebar={<Nav/>}>{content}</Layout>`. It's composition for multi-region layouts — analogous to Vue/Web Components slots. Keeps layout components flexible and reusable.

**96. What is `cloneElement` and when is it used?**
`React.cloneElement(element, newProps)` copies an element and merges new props — used to inject props into children you receive (e.g., a parent giving each child an `onClick` or index). Powerful but somewhat magical; prefer context or explicit APIs when possible. Seen in compound components and form libraries.

**97. What are `Children` utilities?**
`React.Children.map/forEach/count/toArray/only` safely iterate over `props.children`, which can be a single node, array, or undefined. `Children.toArray` also assigns stable keys. Useful when building components that manipulate their children (tabs, carousels, lists).

**98. What is the "state colocation" principle?**
Keep state as close as possible to where it's used. Don't hoist state higher than necessary — global/high state causes broader re-renders and coupling. Lift only when multiple components genuinely need to share it. Colocation improves performance and maintainability.

**99. What is "derived state" and the anti-pattern around it?**
Derived state is data computed from props/other state. The anti-pattern is **copying props into state** (`useState(props.value)`) and trying to keep them synced via effects — it causes bugs and stale data. Instead, compute derived values during render, or if you truly need to reset on prop change, use the `key` remount trick. Rule: "Don't sync state — derive it."

**100. How do you share logic between class and functional components today?**
Extract the logic into a **custom hook** for functional components; for the remaining class components, wrap them or use a small HOC/render-prop adapter that calls the hook. Long-term, migrate classes to functions. In interviews, emphasize that new code should be functional + hooks.

---

<a name="phase-5"></a>
## PHASE 5 — PERFORMANCE & OPTIMIZATION

**101. What are the main causes of performance problems in React?**
(1) Unnecessary re-renders (parent re-renders cascading, unstable props, context changes). (2) Expensive computations on every render. (3) Large lists rendering all items at once. (4) Large bundle sizes / no code splitting. (5) Unmemoized callbacks/objects breaking memoization. (6) Heavy synchronous work blocking the main thread. (7) Frequent layout thrashing from `useLayoutEffect` or DOM reads.

**102. How do you prevent unnecessary re-renders?**
- Wrap pure expensive children in `React.memo`.
- Stabilize prop references with `useMemo`/`useCallback`.
- Lift expensive subtrees out via `children` composition.
- Split contexts and memoize context values; use selector-based stores.
- Colocate state so updates affect the smallest subtree.
- Use proper keys to avoid remounts.
In React 19, the compiler auto-memoizes much of this.

**103. How does `React.memo` work and when does it backfire?**
It shallow-compares props and skips re-render if unchanged. It backfires when props include inline objects/arrays/functions (new reference each render → comparison always fails → memo does nothing but adds cost), or when comparison is more expensive than the render it saves. Use it on genuinely expensive components with stable props, or pass a custom comparison function as the second arg.

**104. What is list virtualization / windowing?**
Rendering only the items currently visible in the viewport (plus a small buffer) instead of the entire list — drastically reducing DOM nodes for long lists/tables. Libraries: `react-window`, `react-virtualized`, TanStack Virtual. Essential for thousands of rows; otherwise you create huge DOM and slow scrolling.

**105. How do you optimize large lists besides virtualization?**
Stable keys; memoize row components; avoid inline functions/objects in rows; paginate or infinite-scroll; debounce filters; move heavy computation out of render; and use `content-visibility` CSS where applicable. Virtualization is usually the biggest win.

**106. What is React Profiler and how do you use it?**
The React DevTools **Profiler** records renders, showing which components rendered, how often, and how long, plus *why* they rendered. The `<Profiler>` component API (`onRender` callback) lets you measure programmatically. Use it to find the actual bottleneck before optimizing — never optimize blind.

**107. How do you debounce/throttle in React?**
Wrap the handler with a debounced/throttled function (lodash `debounce`, or a custom `useDebounce`/`useThrottle` hook). Keep the debounced function stable across renders (store in a ref or `useMemo`) so you don't recreate it each render and lose the timer. Clean it up on unmount. Common for search inputs, resize, scroll.

**108. What is `startTransition`'s role in performance?**
It deprioritizes non-urgent state updates so urgent ones (typing, clicking) stay snappy. React can interrupt and re-do the transition work. Combined with `useDeferredValue`, it keeps interactive UI responsive while expensive renders catch up — concurrency as a performance tool.

**109. What causes "wasted renders" with context and how to fix?**
A context provider whose `value` is a new object each render forces all consumers to re-render. Fix by memoizing the value, splitting frequently- vs rarely-changing data into separate contexts, or using a store with selectors so components subscribe only to the slices they use.

**110. How do you reduce bundle size?**
Code splitting (`React.lazy`, route-based chunks), tree shaking (ESM imports, avoid `import *`), import only what you use from libraries (`import debounce from 'lodash/debounce'`), analyze with `webpack-bundle-analyzer`/`vite` visualizer, lazy-load heavy deps (charts, editors), compress (gzip/brotli), and prefer lighter alternatives (date-fns over moment).

**111. What is the difference between `useMemo` for performance vs correctness?**
Usually `useMemo` is a performance hint React may ignore. But sometimes you need a stable reference for *correctness* — e.g., an object in a `useEffect` dependency array that would otherwise re-run the effect infinitely. In those cases the memoization is structurally necessary, not just an optimization.

**112. What is "tearing" and why does it matter?**
Tearing is when, during a single render pass, different components read **different values** from an external store because the store updated mid-render — producing an inconsistent UI. Concurrent rendering makes this possible. `useSyncExternalStore` exists specifically to prevent tearing by giving React a consistent snapshot.

**113. How do you measure and improve initial load (Core Web Vitals)?**
Measure LCP, CLS, INP via Lighthouse/Web Vitals. Improve with SSR/SSG for fast first paint, code splitting, preloading critical resources, image optimization, avoiding layout shifts (reserve space), reducing JS on the main thread, and streaming/Suspense to show content progressively.

**114. When should you NOT optimize?**
Premature optimization adds complexity and bugs. Don't memoize everything reflexively — `useMemo`/`useCallback`/`React.memo` have costs. Profile first, optimize the proven bottleneck, and measure the improvement. Most components are cheap; re-rendering isn't inherently bad — only *committing unnecessary DOM changes* or doing heavy work is.

**115. What is `key`-based remounting as a performance/correctness tool?**
Changing a `key` unmounts and remounts a component, resetting all its state. Use it to reset forms/animations when an identity changes, or to force a clean slate. It's both a correctness tool (clean reset) and occasionally a simplification over manual state-clearing effects.

---

<a name="phase-6"></a>
## PHASE 6 — INTERNALS: RECONCILIATION, FIBER & RENDERING

**116. What is reconciliation?**
Reconciliation is React's algorithm for figuring out what changed between the previous and next render. It diffs the new element tree against the previous one and computes the minimal DOM operations to update. React uses heuristics (O(n) instead of a general O(n³) tree diff) based on two assumptions: (1) elements of different types produce different trees, and (2) keys identify stable children across renders.

**117. What are React's diffing heuristics?**
(1) **Different element types** → React tears down the old subtree and builds a new one (no diffing across types). (2) **Same type** → React keeps the DOM node, updates only changed attributes, and recurses into children. (3) **Lists** → keys match children between renders so React reorders/reuses rather than recreating. These assumptions let diffing run in linear time.

**118. What is React Fiber?**
Fiber is React's reconciliation engine (since React 16), a complete rewrite that makes rendering **interruptible**. Each element is represented by a "fiber" node — a unit of work holding component type, state, props, and pointers to parent/child/sibling. Fiber lets React split rendering into chunks, pause, prioritize, abort, and resume work — the foundation for concurrent features. The old "stack reconciler" was synchronous and couldn't be interrupted.

**119. What problem did Fiber solve?**
The pre-Fiber stack reconciler rendered the whole tree synchronously and recursively — long renders blocked the main thread, causing dropped frames and janky input. Fiber breaks work into small units that can be paused/resumed/prioritized, so React can yield to the browser for high-priority work (input, animation) and keep the UI responsive. It enables time-slicing and concurrent rendering.

**120. Explain the two phases of Fiber: render and commit.**
**Render/reconcile phase** ("work-in-progress"): React builds a new fiber tree, calling components and computing changes. This phase is asynchronous, interruptible, and side-effect-free — React may even discard it. **Commit phase**: React applies the computed effects to the DOM in one synchronous, uninterruptible pass, then runs layout effects and (asynchronously) passive effects. This split is why render must be pure.

**121. What is "double buffering" in Fiber?**
React maintains two fiber trees: the **current** tree (what's on screen) and the **work-in-progress** tree (being built). When the WIP tree is complete and committed, the pointers swap — the WIP becomes current. This double-buffering lets React build the next UI without affecting the live one and allows discarding incomplete work cleanly.

**122. What are "lanes" in React's scheduler?**
Lanes are React 18's priority model — a bitmask representing different priority levels (e.g., sync/urgent input, default, transitions, idle). Each update is assigned to a lane; the scheduler processes higher-priority lanes first and can batch updates in the same lane. Lanes replaced the older "expiration time" model and enable fine-grained concurrency.

**123. How does React schedule and prioritize work?**
React's scheduler assigns priorities (lanes) to updates: discrete events (clicks, typing) are urgent and processed synchronously; transitions and deferred values are lower priority and interruptible. React uses cooperative scheduling — periodically yielding to the browser (via a scheduler that mimics `requestIdleCallback`) so high-priority work and rendering frames aren't blocked.

**124. What happens during the "begin work" and "complete work" phases?**
For each fiber, **beginWork** processes the component (runs it, reconciles children) and returns the next unit. When a subtree's children are done, **completeWork** builds/updates the corresponding DOM nodes (off-screen) and bubbles up effects. Once the whole WIP tree completes, React commits. This work loop is what can be paused and resumed.

**125. What is an "effect list" in Fiber?**
During the render phase, React collects fibers that have side effects (DOM insertions, updates, deletions, lifecycle/effect callbacks) into an effect list. The commit phase walks this list to apply changes efficiently, rather than re-traversing the entire tree. (Newer versions use subtree flags rather than a linked effect list, but the concept holds.)

**126. Why must the render phase be pure / side-effect free?**
Because React may call components multiple times, pause, abort, or restart the render phase in concurrent mode. If rendering had side effects (mutating external state, network calls, DOM writes), those could run multiple times or partially, causing bugs. Side effects belong in event handlers or `useEffect`/commit phase, which run predictably once committed.

**127. How does React decide to reuse vs recreate a component instance?**
By position in the tree and `type` (and `key`). Same type + same position (or same key) → reuse the instance and its state, update props. Different type or changed key → unmount the old instance (losing state) and mount a new one. This is why changing a key resets state, and why conditionally rendering different component types at the same position discards state.

**128. What is "bailout" in React rendering?**
A bailout is when React skips re-rendering a component because nothing relevant changed — e.g., a `memo` component with equal props, or a component whose state setter was called with the same value (`Object.is` equal). React can reuse the previous fiber/output and skip the subtree, saving work.

**129. If you call a setter with the same value, does it re-render?**
React may **bail out** of the re-render if the new state is `Object.is`-equal to the current state. However, React sometimes still renders that component once before bailing (it has to call it to compare), but it won't go deeper or commit. So same-value sets are cheap but not always a guaranteed zero-render.

**130. What is the significance of `Object.is` in React?**
React uses `Object.is` for comparisons in state bailouts, `useMemo`/`useEffect` dependency checks, `React.memo` shallow comparison (per-prop), and context value changes. `Object.is` is like `===` but treats `NaN` as equal to `NaN` and distinguishes `+0`/`-0`. Understanding it explains why reference equality matters so much for memoization.

---

<a name="phase-7"></a>
## PHASE 7 — CONCURRENT REACT & REACT 18/19

**131. What is Concurrent React / Concurrent Mode?**
Concurrent React is a set of capabilities (opt-in via `createRoot` in React 18) that let React prepare multiple versions of the UI at the same time and interrupt, pause, or abandon a render. It's not a feature you "turn on" globally but a foundation enabling `useTransition`, `useDeferredValue`, streaming SSR, and Suspense for data. The key shift: rendering became interruptible and prioritized.

**132. What changed in React 18?**
Key additions: `createRoot` (new root API enabling concurrent features), **automatic batching** everywhere, `useTransition` and `useDeferredValue`, `useId`, `useSyncExternalStore`, `useInsertionEffect`, improved Suspense + streaming SSR (`renderToPipeableStream`), and stricter `StrictMode` double-effect behavior in dev. It set the stage for React Server Components.

**133. What is automatic batching and how did it change in React 18?**
Batching groups multiple state updates into one re-render. Before React 18, batching only happened inside React event handlers — updates in promises, `setTimeout`, or native events each caused separate renders. React 18 **automatically batches everywhere** (via `createRoot`), so all updates in the same tick are batched regardless of origin. You can opt out with `flushSync` when you need a synchronous DOM update.

**134. What is `flushSync`?**
`ReactDOM.flushSync(() => setState(...))` forces React to flush updates synchronously and update the DOM immediately, opting out of batching. Use rarely — e.g., when you must read the updated DOM right after (measuring layout) before the browser paints. Overuse hurts performance.

**135. How do `useTransition` and `useDeferredValue` differ?**
Both keep the UI responsive during expensive updates. `useTransition` wraps the **state update** you control (`startTransition(() => setX(...))`) and gives `isPending`. `useDeferredValue` wraps a **value** you receive (often a prop you don't control the update of), returning a lagging copy. Use transitions when you trigger the update; use deferred value when you only have the value.

**136. What are React Server Components (RSC)?**
RSCs render **on the server only** and never ship their JS to the client, reducing bundle size. They can directly access server resources (DB, filesystem, secrets) and `await` data. Their output is serialized and streamed to the client, where Client Components (`"use client"`) handle interactivity. RSCs can't use state/effects/browser APIs. They're a foundational architecture in Next.js App Router (and React 19 stabilizes the model).

**137. What's the difference between Server and Client Components?**
**Server Components** (default in RSC frameworks): run on server, no hooks/state/effects, no event handlers, can be async and fetch data directly, ship zero JS. **Client Components** (`"use client"` directive): run on client (and during SSR), support state, effects, event handlers, browser APIs, and ship JS. You compose them — server components render client components and pass serializable props.

**138. What are Server Actions / `"use server"`?**
Server Actions are async functions marked with `"use server"` that run on the server but can be called from client components (e.g., form submissions) without manually building an API endpoint. React/Next handles the RPC. They simplify mutations and progressive enhancement (forms work without JS). Stabilized in React 19.

**139. What new hooks/APIs did React 19 add?**
`useActionState` (manage form action state + pending), `useFormStatus` (read parent form's submission status), `useOptimistic` (optimistic UI updates), the `use` API (read promises/context during render, conditionally), built-in form `action` prop support, `ref` as a regular prop (no `forwardRef`), document metadata hoisting (`<title>`, `<meta>` in components), and stylesheet/async script support. Plus the React Compiler maturing.

**140. What is the `use` API in React 19?**
`use(promise)` lets a component **read the value of a promise** during render, suspending until it resolves (integrating with Suspense). `use(context)` reads context — and unlike `useContext`, `use` **can be called conditionally** (inside `if`). It's a more flexible primitive for consuming resources. It must still follow some placement rules but relaxes the "top level only" constraint for context/promises.

**141. What is `useOptimistic`?**
`useOptimistic(state, updateFn)` (React 19) lets you show an **optimistic** UI immediately (e.g., a sent message appears instantly) while the real async action (server mutation) is in flight, automatically reverting/reconciling when it resolves or fails. It makes responsive, optimistic UIs much simpler than manual rollback logic.

**142. What is `useActionState`?**
`useActionState(action, initialState)` (React 19) wires a form/async action to state: it returns `[state, formAction, isPending]`, manages the result of the action and pending status, and integrates with form `action`. It standardizes the common "submit → pending → result/error" pattern.

**143. What is the React Compiler (formerly React Forget)?**
An optimizing build-time compiler that **automatically memoizes** components and values — inserting the equivalent of `useMemo`/`useCallback`/`React.memo` where beneficial — so you rarely hand-write them. It analyzes your code's data flow to skip unnecessary re-renders while preserving correctness. It aims to make "just write straightforward components" performant by default.

**144. What is streaming SSR with Suspense?**
With `renderToPipeableStream`, the server can **stream HTML in chunks** as parts of the page become ready, wrapping slow parts in `<Suspense>`. The shell loads fast; slower sections stream in with their fallbacks replaced as data resolves. Combined with **selective hydration**, React can hydrate ready parts independently and prioritize where the user interacts. Big improvement over all-or-nothing SSR.

**145. What is selective / progressive hydration?**
React 18 can hydrate parts of the page **independently and out of order**, prioritizing components the user interacts with (clicking an un-hydrated section makes React hydrate it first). This avoids the old requirement that the entire page hydrate before anything is interactive, improving Time-to-Interactive.

**146. What is the difference between `createRoot` and `hydrateRoot`?**
`createRoot(container).render(<App/>)` creates a fresh client-rendered root (for SPAs). `hydrateRoot(container, <App/>)` attaches React to **server-rendered HTML**, reusing the existing DOM and only attaching event handlers/state rather than recreating nodes. Both enable concurrent features (they replaced `ReactDOM.render`/`hydrate`).

**147. Why does `StrictMode` double-invoke things in development?**
React 18 `StrictMode` intentionally **double-invokes** render functions, state updaters, and mounts/unmounts effects twice in development to surface impure renders and missing effect cleanups (preparing your code for concurrent features and future reusable state). It only happens in dev, not production. If your effect breaks when run twice, it has a cleanup bug.

**148. What does it mean that "state may be reused" in future React?**
React is moving toward features (like offscreen rendering / `<Activity>`) where component state can be **preserved while hidden and restored**, or where mounting/unmounting happens more freely. Writing resilient effects (idempotent setup + proper cleanup) prepares your components for this — which is exactly what StrictMode's double-mount tests.

**149. What is the `<Activity>` (formerly Offscreen) component?**
An API to render parts of the UI in the background or keep them mounted but hidden, preserving their state and DOM while deprioritizing their rendering — useful for instant tab switches or pre-rendering likely-next views. It lets you hide UI without losing state, unlike conditional unmounting.

**150. Summarize the evolution: 16 → 17 → 18 → 19.**
**16**: Fiber rewrite, Hooks (16.8), error boundaries, portals, fragments. **17**: no new features — event delegation moved to root, gradual-upgrade support, new JSX transform (no React import needed). **18**: concurrent rendering, `createRoot`, automatic batching, transitions, Suspense streaming SSR, new hooks. **19**: Server Components/Actions stabilized, `use` API, form hooks, `useOptimistic`, ref-as-prop, metadata support, React Compiler.

---

<a name="phase-8"></a>
## PHASE 8 — SSR, RSC, HYDRATION & FRAMEWORKS

**151. What is CSR, SSR, SSG, and ISR?**
- **CSR (Client-Side Rendering):** browser downloads a minimal HTML + JS bundle, React renders everything in the browser. Fast navigation, slow first paint, weaker SEO.
- **SSR (Server-Side Rendering):** server renders HTML per request, browser shows it fast, then hydrates. Better SEO/first paint, more server cost.
- **SSG (Static Site Generation):** HTML pre-rendered at **build time**, served from CDN. Fastest, but content is fixed until rebuild.
- **ISR (Incremental Static Regeneration):** SSG + background regeneration on a schedule/on-demand, so static pages can update without a full rebuild (Next.js).

**152. What is hydration?**
Hydration is the process where React takes server-rendered static HTML and "attaches" to it on the client — reconstructing the component tree, wiring up event handlers and state — making the static markup interactive without re-creating the DOM. The server HTML and the client's first render **must match**, or you get hydration errors.

**153. What causes hydration mismatches and how do you fix them?**
Causes: rendering different content on server vs client — using `Date.now()`/`Math.random()`, `window`/`localStorage` during render, locale/timezone differences, or invalid HTML nesting. Fixes: ensure deterministic render; defer client-only content with a mounted flag/`useEffect`; use `useId` for IDs; guard browser APIs; or use `suppressHydrationWarning` for known-unavoidable diffs (timestamps).

**154. Why is hydration sometimes considered wasteful?**
Traditional hydration re-runs component logic on the client to attach handlers, effectively doing the render work twice (once on server for HTML, once on client to hydrate) and shipping all that JS. This delays interactivity (TTI). Solutions in the ecosystem: streaming + selective hydration (React 18), React Server Components (ship less JS), islands architecture (Astro), and resumability (Qwik) which skips hydration entirely.

**155. What is the "islands architecture"?**
Ship a mostly-static HTML page with isolated interactive "islands" that hydrate independently; everything else is static, zero-JS. Reduces JS sent and hydration cost. Astro popularized it. Conceptually related to React's selective hydration and RSC, but more aggressive about defaulting to static.

**156. What problem do React Server Components solve over SSR?**
Plain SSR still ships all component JS to the client for hydration. RSCs render on the server and **ship no JS** for those components — only their serialized output. They reduce bundle size, allow direct backend access, and keep sensitive logic server-side. SSR is about *when/where HTML is generated*; RSC is about *which components never reach the client at all*. They're complementary.

**157. How does data fetching differ in Server Components?**
Server Components can be `async` and directly `await` data (DB queries, `fetch`) in the render function — no `useEffect`, no client loading state needed for that data. The result streams to the client. Client Components still fetch via hooks/libraries (TanStack Query, SWR) or receive data as props from a server parent.

**158. What is Next.js and what does it add over plain React?**
Next.js is a React framework providing file-based routing, SSR/SSG/ISR, the App Router with RSC/Server Actions, API routes, image/font optimization, code splitting, middleware, and production-grade build tooling out of the box. It turns React (a view library) into a full-stack framework with sensible defaults.

**159. App Router vs Pages Router (Next.js)?**
**Pages Router** (`/pages`): the original model — file-based routes, `getServerSideProps`/`getStaticProps` for data, client components by default. **App Router** (`/app`): newer — built on React Server Components, layouts, nested routing, streaming, Server Actions, `loading.js`/`error.js` conventions, components are server by default with `"use client"` opt-in. App Router is the future-facing model.

**160. What is Remix and how does it compare?**
Remix is a React framework emphasizing web fundamentals: nested routes with per-route `loader` (read) and `action` (write) functions, progressive enhancement (forms work without JS), and tight server/client data flow. It leans on SSR and standard web APIs (Request/Response, FormData). Compared to Next.js, it's more focused on data loading/mutations via the platform and less on static generation (though it supports it).

**161. How do you do SEO in a React SPA?**
Pure CSR SPAs are weak for SEO since crawlers may not run JS well. Solutions: SSR/SSG (Next.js, Remix) so crawlers get full HTML; manage meta tags (React 19 native metadata, or `react-helmet`); generate sitemaps; ensure semantic HTML and proper status codes; pre-render with services if needed. Server rendering is the robust answer.

**162. What is `renderToString` vs `renderToPipeableStream`?**
`renderToString` synchronously renders the whole app to an HTML string — simple but blocking and no streaming/Suspense data support. `renderToPipeableStream` (React 18, Node) **streams** HTML as it's ready, supports Suspense boundaries and selective hydration, and doesn't block on slow data. For edge runtimes there's `renderToReadableStream`. Modern SSR uses the streaming APIs.

---

<a name="phase-9"></a>
## PHASE 9 — STATE MANAGEMENT ECOSYSTEM

**163. When do you actually need a state management library?**
When state is genuinely **global** and shared across many distant components (auth/user, theme, cart, notifications), when you have complex update logic, need predictable debugging/time-travel, or want to avoid prop drilling/context performance issues at scale. For most apps, local state + context + a data-fetching library (TanStack Query) covers a lot; reach for Redux/Zustand only when shared client state grows complex.

**164. What is Redux and its core principles?**
Redux is a predictable state container built on: (1) **single source of truth** — one store holding the whole app state tree; (2) **state is read-only** — you change it only by dispatching **actions**; (3) **changes via pure reducers** — `(state, action) => newState`. This unidirectional flow (action → reducer → new state → UI) makes state changes traceable, testable, and time-travelable via DevTools.

**165. What is Redux Toolkit (RTK) and why is it preferred?**
RTK is the official, opinionated, batteries-included way to write Redux. It reduces boilerplate with `createSlice` (auto-generates actions + reducers, uses Immer for "mutating" syntax), `configureStore` (sane defaults + DevTools + thunk), `createAsyncThunk` for async, and **RTK Query** for data fetching/caching. Modern Redux = RTK; raw Redux boilerplate is discouraged.

**166. What is the Redux data flow?**
UI dispatches an **action** (a plain object `{ type, payload }`) → the **store** runs **reducers** which compute the new state purely → subscribed components re-render with new state via `useSelector`. Middleware (thunk/saga) sits between dispatch and reducer to handle async/side effects. Strictly unidirectional.

**167. What is middleware in Redux? Thunk vs Saga?**
Middleware intercepts dispatched actions to add capabilities (logging, async). **Redux Thunk** lets action creators return functions (dispatching async logic imperatively) — simple, most common. **Redux Saga** uses generator functions to model complex async flows declaratively (`takeLatest`, `call`, `put`), better for intricate side-effect orchestration (debounced/cancelable flows) at the cost of a learning curve. RTK includes thunk by default.

**168. What is `useSelector` and `useDispatch`?**
`useSelector(state => state.x)` subscribes a component to a slice of the Redux store, re-rendering when that selected value changes (compared with `===` by default). `useDispatch()` returns the store's `dispatch` to send actions. Selectors should return the minimal needed data and may use `reselect`'s `createSelector` to memoize derived computations.

**169. What is reselect / memoized selectors?**
`reselect` creates memoized selectors that recompute derived data only when their inputs change, avoiding expensive recalculation and unnecessary re-renders. Crucial when selecting derived/filtered data so `useSelector` doesn't return a new reference each call (which would re-render every time).

**170. What is the Context API vs Redux?**
Context is a **dependency-injection** mechanism to pass data down without prop drilling — not a state management solution per se (no built-in updates optimization, devtools, middleware). Redux is a full state container with predictable updates, middleware, devtools, and selective subscriptions. Context is great for low-frequency global values (theme, locale, auth). For high-frequency or complex state, Redux/Zustand scale better (Context re-renders all consumers on value change).

**171. What is Zustand?**
Zustand is a minimal, hook-based state management library: create a store with `create(set => ({...}))`, consume with a selector `useStore(s => s.count)`. It's tiny, has no boilerplate/providers, supports selective subscriptions (only re-render on selected slice), middleware (persist, immer, devtools), and works outside React. Popular as a lighter Redux alternative.

**172. What is Jotai / Recoil (atomic state)?**
Atomic libraries model state as small **atoms** that components subscribe to individually; derived atoms compute from others. Components re-render only when atoms they use change — fine-grained reactivity without selectors. **Jotai** is minimal and actively maintained; **Recoil** (Meta) pioneered the model but is now largely inactive. Good when state is highly granular and interdependent.

**173. Redux vs Zustand vs Jotai vs Context — how do you choose?**
- **Context:** simple, low-frequency global values; built-in.
- **Redux Toolkit:** large apps, complex flows, team conventions, strong devtools/middleware, RTK Query for data.
- **Zustand:** want global state with minimal boilerplate and good performance; pragmatic default for many apps.
- **Jotai:** highly granular, derived, interdependent state (forms, editors).
Pick based on complexity, team familiarity, and whether the state is client state vs server cache.

**174. What's the difference between "server state" and "client state"?**
**Server state** is data owned by the backend, fetched async, shared, and can become stale (users, products) — best handled by caching libraries (TanStack Query, RTK Query, SWR) that manage fetching, caching, revalidation, dedup. **Client state** is UI/local state (modals open, form inputs, theme) — handled by `useState`/Context/Zustand/Redux. A common architectural mistake is shoving server data into Redux manually; a query library does it better.

**175. What is TanStack Query (React Query) and what does it give you?**
A server-state library that handles fetching, **caching, background revalidation, deduplication, pagination/infinite queries, retries, and stale-while-revalidate** out of the box. You declare queries (`useQuery`) and mutations (`useMutation`); it manages loading/error/cache states, `staleTime`/`gcTime`, and refetch triggers. It eliminates tons of manual `useEffect` fetching boilerplate and race-condition handling.

---

<a name="phase-10"></a>
## PHASE 10 — ROUTING, DATA FETCHING & FORMS

**176. What is React Router and how does client-side routing work?**
React Router maps URLs to components without full page reloads. It intercepts navigation, updates the URL via the History API, and renders the matching route's component — giving SPA navigation. Core pieces (v6): `<BrowserRouter>`, `<Routes>`/`<Route>`, `<Link>`/`<NavLink>`, `useNavigate`, `useParams`, `useSearchParams`, nested routes with `<Outlet>`, and (v6.4+) data routers with `loader`/`action`.

**177. What's the difference between `BrowserRouter` and `HashRouter`?**
`BrowserRouter` uses the HTML5 History API for clean URLs (`/about`) but needs server config to serve `index.html` for all routes (else refresh 404s). `HashRouter` uses the URL hash (`/#/about`) — works without server config since the hash isn't sent to the server, but uglier URLs and worse SEO. Use `BrowserRouter` for real apps; `HashRouter` for static hosts you can't configure.

**178. What are nested routes and `<Outlet>`?**
Nested routes let child routes render inside a parent layout. The parent renders shared UI (nav/sidebar) plus an `<Outlet/>` placeholder where the matched child route renders. This models layouts and shared chrome cleanly without repeating wrappers in each route.

**179. How do you do dynamic routes and read params?**
Define a param with a colon: `<Route path="/user/:id" .../>`, then read it via `useParams()` → `{ id }`. Query strings use `useSearchParams()` returning a `URLSearchParams` and a setter. Use these for detail pages, filters, and pagination state in the URL.

**180. What are React Router loaders and actions (v6.4+ data APIs)?**
**Loaders** fetch data *before* a route renders (no loading-spinner-on-mount waterfall), accessible via `useLoaderData`. **Actions** handle form submissions/mutations server-side-style and trigger revalidation. Together with `<Form>`, deferred data, and error elements, they bring Remix-style data flow into React Router, reducing `useEffect` fetching.

**181. How do you protect routes (auth guards)?**
Create a wrapper that checks auth and either renders the children/`<Outlet/>` or redirects (`<Navigate to="/login" replace />`). In data routers, a loader can check auth and `redirect()` before rendering. Centralize the check so protected routes stay declarative.

**182. How do you lazy-load routes?**
Combine `React.lazy` with route definitions and a `<Suspense>` boundary (or React Router's `lazy` route property in data routers) so each route's code loads only when navigated to. This is the highest-impact code-splitting strategy for SPAs.

**183. What are the patterns for data fetching in React?**
- **`useEffect` + fetch:** basic, manual loading/error/race handling (avoid for complex needs).
- **Data libraries (TanStack Query/SWR/RTK Query):** caching, revalidation, dedup — recommended for client components.
- **Route loaders (React Router/Remix):** fetch before render.
- **Server Components:** `await` directly on the server.
- **Suspense for data:** declarative loading via `use()`/framework integration.

**184. What is the "fetch waterfall" problem and how do you avoid it?**
A waterfall is sequential dependent fetches where each request waits for the previous (often caused by fetching in nested components' effects), slowing load. Avoid by: fetching in parallel (`Promise.all`), hoisting fetches to route loaders/parents, prefetching, using libraries that dedupe and parallelize, or RSC where you can structure parallel awaits. Suspense + parallel data helps too.

**185. Controlled forms vs form libraries — when to use what?**
Plain controlled state is fine for small forms. For complex forms (many fields, validation, arrays, performance), use **React Hook Form** (uncontrolled-first, minimal re-renders, great perf, schema validation via Zod/Yup) or **Formik** (controlled, more re-renders, older). React Hook Form is the modern favorite for performance and DX.

**186. Why is React Hook Form performant?**
It uses **uncontrolled inputs with refs**, so typing doesn't re-render the whole form on every keystroke — it subscribes only what needs updating and validates on demand. Contrast with fully controlled forms (or Formik) where each keystroke can re-render many components. It also integrates schema validators (Zod/Yup) cleanly.

**187. How do you handle form validation?**
Inline (per-field) or schema-based (Zod/Yup) validation; validate on change/blur/submit. Show errors near fields with accessible `aria` attributes. Libraries (RHF + Zod) give typed, declarative schemas and consolidated error handling. For server-driven validation, surface server errors back into the form state.

**188. How do you handle file uploads in React?**
File inputs are inherently **uncontrolled** — read files from `e.target.files` (a `FileList`). Use `FormData` to send multipart requests, show progress via `XMLHttpRequest`/`fetch` streams or an upload library, and validate type/size client-side. For previews, `URL.createObjectURL` (revoke it on cleanup to avoid leaks).

**189. How do you debounce a search input that triggers fetches?**
Debounce the query value (custom `useDebounce` hook or library), trigger the fetch on the debounced value, and cancel stale requests (AbortController or a query library's built-in cancellation). TanStack Query + a debounced key handles caching, dedup, and cancellation cleanly.

**190. What is optimistic UI and how do you implement it?**
Update the UI immediately as if the mutation succeeded, then reconcile when the server responds (rollback on failure). Implement with TanStack Query's `onMutate`/`onError`/`onSettled`, RTK Query optimistic updates, or React 19's `useOptimistic`. It makes apps feel instant for likely-successful actions (likes, sends, toggles).

---

<a name="phase-11"></a>
## PHASE 11 — TYPESCRIPT WITH REACT

**191. How do you type a functional component's props?**
Define an interface/type and annotate the props parameter:
```tsx
interface ButtonProps { label: string; onClick: () => void; disabled?: boolean; }
function Button({ label, onClick, disabled = false }: ButtonProps) { ... }
```
Prefer typing props directly over `React.FC` (see next). Use `?` for optional props and default parameters for defaults.

**192. Should you use `React.FC`? Why is it sometimes discouraged?**
`React.FC<Props>` types a component and implicitly includes `children`. It's discouraged by many because: it forces an implicit `children` even when the component takes none, historically complicated generics and `defaultProps`, and adds little value over typing the props parameter directly. Modern guidance: type props explicitly and add `children: React.ReactNode` only when needed.

**193. How do you type `children`?**
Use `React.ReactNode` for "anything renderable" (elements, strings, numbers, arrays, null). Use `React.ReactElement` for a single element, or a function type for render props: `children: (data: T) => React.ReactNode`. `ReactNode` is the usual default for a `children` prop.

**194. How do you type `useState`?**
TypeScript usually infers it from the initial value: `useState(0)` → `number`. When the initial value doesn't capture the full type (e.g., starts `null` but later holds an object), annotate the generic: `useState<User | null>(null)`. For arrays/unions, be explicit: `useState<string[]>([])`.

**195. How do you type `useRef`?**
For DOM refs: `useRef<HTMLInputElement>(null)` → `ref.current` is `HTMLInputElement | null`. For mutable instance values you write to: `useRef<number>(0)` (note: passing an initial value makes it `MutableRefObject`). DOM refs initialized with `null` are read-only `RefObject` intended for the `ref` attribute.

**196. How do you type event handlers?**
Use React's synthetic event types: `React.ChangeEvent<HTMLInputElement>`, `React.MouseEvent<HTMLButtonElement>`, `React.FormEvent<HTMLFormElement>`, `React.KeyboardEvent`. Example: `const onChange = (e: React.ChangeEvent<HTMLInputElement>) => setValue(e.target.value)`. Let inline handlers infer when possible.

**197. How do you type `useReducer`?**
Type the state and a discriminated-union action type:
```tsx
type State = { count: number };
type Action = { type: 'inc' } | { type: 'set'; payload: number };
function reducer(state: State, action: Action): State { ... }
```
The discriminated union gives exhaustive, type-safe `switch` handling on `action.type`.

**198. How do you type custom hooks?**
Type parameters and return value; for tuple returns use `as const` or an explicit tuple type so TS preserves the order/types instead of widening to a union array: `return [value, setValue] as const`. Generics make hooks reusable: `function useFetch<T>(url: string): { data: T | null; loading: boolean }`.

**199. What are generic components and when are they useful?**
Components parameterized by a type, e.g., a typed `<List<T> items={T[]} renderItem={(item: T)=>...} />`. Useful for reusable data components (tables, selects, lists) that should preserve item types end-to-end. Written as generic functions: `function List<T>(props: ListProps<T>) {...}`.

**200. What's the difference between `interface` and `type` for props?**
Both work. `interface` supports declaration merging and is conventional for object/props shapes; `type` is more flexible (unions, intersections, mapped/conditional types). A common convention: `interface` for public component props, `type` for unions and complex compositions. Functionally interchangeable for most prop definitions.

**201. How do you type a component that forwards refs?**
Pre-19: `forwardRef<HTMLInputElement, InputProps>((props, ref) => <input ref={ref} {...props}/>)` — first generic is the ref element type, second the props. In React 19, type `ref` as a normal prop: `function Input({ ref, ...props }: InputProps & { ref?: React.Ref<HTMLInputElement> })`.

**202. How do you extend native HTML element props?**
Use `React.ComponentProps<'button'>` or `React.ButtonHTMLAttributes<HTMLButtonElement>` and intersect with your own: `type Props = React.ComponentProps<'button'> & { variant: 'primary' | 'ghost' }`. This lets your component accept all standard button attributes plus custom ones — the idiomatic way to build typed UI primitives.

**203. What is `as const` and why does it matter in React?**
`as const` makes a value deeply readonly and narrows literals to their exact type (e.g., `'primary'` instead of `string`). Useful for action types, config objects, and tuple returns from hooks so TypeScript keeps precise types rather than widening. Key for discriminated unions and stable hook return tuples.

---

<a name="phase-12"></a>
## PHASE 12 — TESTING

**204. What's the testing philosophy of React Testing Library (RTL)?**
"Test behavior, not implementation." RTL encourages querying the DOM the way a **user** would (by role, label, text) and asserting on visible output rather than internal state or instance methods. This makes tests resilient to refactors — you can change internals freely as long as user-facing behavior holds. Guiding principle: "The more your tests resemble the way your software is used, the more confidence they give."

**205. RTL vs Enzyme — why did the ecosystem move to RTL?**
Enzyme tested implementation details (shallow rendering, accessing state/instances), making tests brittle and coupled to internals; it also lagged on hooks/concurrent features. RTL tests user-visible behavior, works naturally with hooks and modern React, and is officially recommended. Enzyme is effectively legacy.

**206. What are RTL queries and their priority order?**
Queries by accessibility/semantics first: `getByRole` (best — tests accessibility too), then `getByLabelText` (forms), `getByPlaceholderText`, `getByText`, `getByDisplayValue`, and last resort `getByTestId`. Variants: `getBy*` (throws if not found), `queryBy*` (returns null — for asserting absence), `findBy*` (async, waits for element). Prefer accessible queries to also validate a11y.

**207. How do you test user interactions?**
Use `@testing-library/user-event` (preferred over `fireEvent`) to simulate realistic interactions (`await user.click(...)`, `await user.type(...)`) including focus, hover, and key sequences. Then assert on the resulting DOM. `user-event` better mirrors real browser behavior than the lower-level `fireEvent`.

**208. How do you test async behavior?**
Use `findBy*` queries (which retry until found) or `waitFor(() => expect(...))` for assertions that become true after async work. For loading→loaded flows, await the loaded content. Mock network with **MSW (Mock Service Worker)** — it intercepts at the network level so your code's real fetch logic runs against mocked responses, which is more realistic than stubbing `fetch`.

**209. How do you test custom hooks?**
Use `renderHook` from `@testing-library/react` (it renders the hook in a test component) and `act()` to wrap state updates: `const { result } = renderHook(() => useCounter()); act(() => result.current.increment());` then assert on `result.current`. Wrap with providers via the `wrapper` option when the hook needs context.

**210. What is `act()` and why does it matter?**
`act()` ensures all updates (state, effects) are processed and applied before you assert, mirroring how React batches and flushes in the browser — preventing "state update not wrapped in act" warnings and flaky tests. RTL's render/user-event wrap interactions in `act` for you; you mainly need it manually with `renderHook` or direct state pokes.

**211. How do you mock modules, API calls, and context?**
Mock modules with `jest.mock('./module')` / `vi.mock` (Vitest). For APIs, prefer **MSW** over mocking `fetch` directly. For context, wrap the component under test in the real provider (via RTL's `wrapper`) with test values rather than mocking the context, keeping tests realistic.

**212. What are snapshot tests and their pitfalls?**
Snapshot tests serialize a component's rendered output and compare it to a saved snapshot, failing on changes. Useful for catching unintended markup changes, but pitfalls: large snapshots become noise, devs blindly update them (defeating the purpose), and they don't assert *behavior*. Use sparingly, keep them small/focused, and prefer explicit behavioral assertions.

**213. What's the difference between unit, integration, and e2e tests in React?**
- **Unit:** a single function/hook/small component in isolation.
- **Integration:** multiple components working together (a form with validation and submission) — RTL shines here and gives the best confidence-per-effort.
- **E2E:** the whole app in a real browser (Cypress, Playwright), testing real user flows across pages/network. The "testing trophy" (Kent C. Dodds) favors heavy integration testing.

**214. What is Jest vs Vitest?**
Both are test runners with assertion/mocking APIs. **Jest** is the long-standing standard (great with CRA/Babel). **Vitest** is a Vite-native runner — faster, ESM-first, near Jest-compatible API, and the default in Vite-based projects. For modern Vite apps, Vitest is increasingly preferred; Jest remains widely used.

**215. How do you test components that use routing or Redux?**
Wrap the component under test in the needed providers: a `<MemoryRouter>` (with `initialEntries`) for routing, and a real Redux `<Provider>` with a test store for Redux. Create a custom `render` helper that injects all app providers so tests stay concise and realistic.

---

<a name="phase-13"></a>
## PHASE 13 — TOOLING, BUILD & SECURITY

**216. What is the difference between Webpack, Vite, and other bundlers?**
**Webpack** bundles everything upfront — mature, configurable, but slower dev startup. **Vite** uses native ES modules + esbuild for near-instant dev server start and HMR, bundling with Rollup for production — much faster DX, now the default for most new React apps. Others: **esbuild**/**SWC** (ultra-fast transpilers used under the hood), **Parcel** (zero-config), **Turbopack** (Next.js's Rust bundler).

**217. Why did Create React App fall out of favor?**
CRA became slow, under-maintained, and hid configuration behind `react-scripts`, lagging on modern needs. The community (and React's own docs) now recommend **Vite** for SPAs or a framework like **Next.js**/**Remix** for full-stack — faster, better defaults, and actively maintained. CRA is effectively deprecated.

**218. What is Babel and the JSX transform?**
Babel transpiles modern JS/JSX into browser-compatible code. The **JSX transform** converts JSX into function calls. The **new transform** (React 17+) compiles JSX to `_jsx(...)` from `react/jsx-runtime` automatically — so you no longer need `import React` just to use JSX. (SWC/esbuild can do this faster than Babel.)

**219. What is tree shaking?**
Tree shaking is dead-code elimination during bundling — removing exports that are never imported. It relies on ES module `import`/`export` static analysis. To benefit: use ESM, avoid side-effectful imports, import specific functions (`import { x } from 'lib'` or `lib/x`), and mark packages `"sideEffects": false`. Reduces bundle size.

**220. What is the most common security vulnerability in React and how does React mitigate it?**
**Cross-Site Scripting (XSS).** React mitigates it by **auto-escaping** all values embedded in JSX — `{userInput}` is rendered as text, not HTML, so injected `<script>` won't execute. The escape hatch `dangerouslySetInnerHTML` bypasses this protection (hence the scary name) and must only be used with sanitized HTML (e.g., via DOMPurify).

**221. What is `dangerouslySetInnerHTML` and when is it safe?**
It sets raw HTML directly: `<div dangerouslySetInnerHTML={{ __html: html }} />`, bypassing React's escaping. It's only safe with **trusted or sanitized** HTML — always run user/third-party HTML through a sanitizer like **DOMPurify** first to strip scripts/handlers. Otherwise it's an XSS vector.

**222. How do you prevent other common React security issues?**
- **XSS:** rely on JSX escaping; sanitize any `dangerouslySetInnerHTML`.
- **Injection via URLs:** validate `href`/`src` (block `javascript:` URLs).
- **Sensitive data:** never embed secrets in client bundles (they're public).
- **Dependencies:** audit (`npm audit`), pin versions, watch for supply-chain attacks.
- **CSRF/auth:** handle on the server; use httpOnly cookies or secure token storage (avoid localStorage for tokens when possible).
- **CSP:** set a Content Security Policy header.

**223. Why shouldn't you store JWTs in localStorage?**
`localStorage` is accessible to any JS on the page, so an XSS bug can exfiltrate the token. **httpOnly, Secure, SameSite cookies** aren't readable by JS, mitigating token theft via XSS (at the cost of needing CSRF protection). It's a trade-off, but for sensitive tokens, httpOnly cookies are generally safer. At minimum, keep the XSS surface minimal.

**224. What are environment variables in React and the key caveat?**
Build tools inject env vars at build time (Vite: `import.meta.env.VITE_*`; CRA: `REACT_APP_*`). The **critical caveat**: anything bundled into client code is **public** — never put secrets (API keys with privileges, DB creds) in client env vars. Keep secrets server-side; only expose safe, publishable values to the client.

**225. What is ESLint and Prettier's role, and the React-specific plugins?**
**ESLint** catches bugs/anti-patterns; **Prettier** auto-formats. React-specific: `eslint-plugin-react` (JSX rules), `eslint-plugin-react-hooks` (enforces Rules of Hooks + exhaustive-deps — catches stale-closure bugs), and `jsx-a11y` (accessibility). The `react-hooks/exhaustive-deps` rule is especially valuable for correct effect dependencies.

**226. How do you handle accessibility (a11y) in React?**
Use semantic HTML (`<button>`, `<nav>`, `<main>`), proper ARIA roles/attributes only when semantics fall short, label form controls (`htmlFor`/`aria-label`), manage focus (especially in modals/route changes), ensure keyboard navigability, sufficient color contrast, and test with `jsx-a11y` lint, axe DevTools, and screen readers. `useId` helps wire labels/aria without collisions.

**227. What is hot module replacement (HMR) / Fast Refresh?**
HMR swaps changed modules in a running app without a full reload; React **Fast Refresh** preserves component state across edits where possible, so you don't lose your place while developing. Vite and modern setups provide near-instant Fast Refresh, a major DX improvement.

---

<a name="phase-14"></a>
## PHASE 14 — DESIGN PATTERNS & ARCHITECTURE

**228. How do you structure a large React application?**
Two common approaches: **feature-based** (group by domain: `/features/auth`, `/features/cart`, each with its own components/hooks/state/api) — scales best for large apps; or **type-based** (`/components`, `/hooks`, `/services`) — fine for small apps. Add shared `/components/ui`, `/lib`, `/hooks`. Keep features cohesive and loosely coupled, colocate related code, and define clear public APIs per feature (index barrels). Feature-based + colocation is the senior-recommended default.

**229. What is "separation of concerns" in modern React?**
Rather than splitting purely by file type, separate by **responsibility**: UI rendering (presentational components), business/stateful logic (custom hooks), data access (API/service modules), and global state (store). Custom hooks are the key tool — they pull logic out of components so components stay focused on rendering.

**230. What is the "smart vs dumb" / container-presentational pattern's modern form?**
The modern form replaces container components with **custom hooks**: a hook encapsulates data/logic (`useUserProfile()`), and a presentational component consumes it and renders. You keep the separation (logic vs presentation) without the extra wrapper component layer. Still valuable conceptually for testability and reuse.

**231. How do you handle cross-cutting concerns (auth, logging, theming)?**
Combine: **Context/Providers** for app-wide values (auth, theme, i18n), **custom hooks** for reusable logic (`useAuth`, `useAnalytics`), **wrapper components/route guards** for gating, and **interceptors** (in your API layer) for auth headers/error handling. Keep these centralized so feature code stays clean.

**232. What is a "barrel file" and its trade-off?**
A barrel (`index.ts` re-exporting a module's public API) simplifies imports (`import { Button } from '@/components'`). Trade-off: barrels can hurt tree-shaking and slow builds/cold starts if they pull in large module graphs, and can cause circular imports. Use them judiciously for clear public APIs; avoid giant root barrels.

**233. What are micro-frontends and when are they justified?**
Micro-frontends split a frontend into independently developed/deployed apps (via Module Federation, single-spa, or iframes), letting teams own slices end-to-end. Justified for large orgs with many teams needing independent deploys and tech autonomy. Costs: shared-dependency duplication, integration complexity, consistency challenges. For most apps a well-structured monolith is simpler and better.

**234. What is Module Federation?**
A Webpack (and now Vite plugin) feature letting separate builds **share code and load each other's modules at runtime** — the backbone of many micro-frontend setups. App A can consume a component from independently-deployed App B without bundling it at build time, with shared dependency negotiation. Powerful but adds operational complexity.

**235. How do you design a reusable component library?**
Build accessible, unstyled-or-themeable primitives; expose props that extend native elements (`ComponentProps<'button'>`); forward refs; support composition (compound components, `children`, slots); document with Storybook; type everything; keep components controlled/uncontrolled-flexible; and avoid baking in app-specific logic. Consider headless libraries (Radix, Headless UI) as a base for behavior + your own styling.

**236. What is Storybook and why use it?**
Storybook is a tool for developing and documenting UI components in **isolation**, rendering each component in various states ("stories") outside the app. Benefits: visual development, design-system documentation, visual regression testing, and a living component catalog. Great for component libraries and team collaboration.

**237. How do you manage theming and design tokens in React?**
Use CSS custom properties (variables) for tokens (colors, spacing) toggled via a `data-theme` attribute or class, optionally driven by a theme Context; or CSS-in-JS theme providers (styled-components/emotion `ThemeProvider`); or utility frameworks (Tailwind with a token config). Prefer CSS variables for performance (no JS re-render to switch themes) and design-system consistency.

**238. CSS approaches in React — what are the options and trade-offs?**
- **CSS Modules:** scoped class names, zero runtime, simple.
- **CSS-in-JS (styled-components/emotion):** dynamic styling via props, colocated, but runtime cost and SSR complexity (improving with `useInsertionEffect`).
- **Utility-first (Tailwind):** fast, consistent, no naming, larger markup; great DX, very popular.
- **Zero-runtime CSS-in-JS (vanilla-extract, Linaria):** type-safe styles compiled at build time.
Choose based on team preference, dynamic-styling needs, and performance.

**239. How do you handle error handling and resilience at the app level?**
Layer it: **error boundaries** around routes/features for render errors with fallback UI; try/catch in event handlers/async; a global API error interceptor; toast/notification system for user-facing errors; logging/monitoring (Sentry); retry logic for transient failures (query libraries); and graceful degradation. Combine error boundaries with `react-error-boundary` for reset/retry UX.

**240. How do you approach performance budgets and monitoring in production?**
Set budgets (bundle size, LCP/INP targets) enforced in CI (bundle analyzers, Lighthouse CI). Monitor real-user metrics (Web Vitals via analytics, Sentry performance). Track render performance with the Profiler in dev. Lazy-load, code-split, and audit dependencies. Treat performance as a continuous, measured discipline, not a one-time pass.

---

<a name="phase-15"></a>
## PHASE 15 — TRICKY / GOTCHA & RAPID-FIRE ROUND

**241. Why does my `useEffect` run twice on mount in development?**
React 18 `StrictMode` intentionally mounts, unmounts, and remounts components once in **development** to surface effects that aren't cleanup-safe. It does **not** happen in production. If running twice breaks things, your effect is missing proper cleanup or isn't idempotent — fix the effect, don't remove StrictMode.

**242. Why is my state "one step behind"?**
Because state updates are asynchronous/batched — reading state right after `setState` gives the value from the current render, not the pending one. The component re-renders with the new value next. If you need the latest value for further updates, use the functional updater (`setX(prev => ...)`) or compute the value separately.

**243. Why does my list lose input focus / show wrong data after reordering?**
Almost always a **key** problem — using array index as key (or non-stable keys) so React reuses the wrong DOM nodes/state when the list reorders. Use stable unique IDs from your data as keys.

**244. Why does my child component re-render even though its props didn't change?**
Because its **parent re-rendered** — by default React re-renders the whole subtree, regardless of prop changes. It's usually harmless. To skip it, wrap the child in `React.memo` and ensure stable prop references (`useCallback`/`useMemo`), or restructure with `children` composition.

**245. Why does `setState` in a loop only update once?**
If you call `setCount(count + 1)` multiple times, each reads the same stale `count` from the closure, so they all set the same value → one increment. Use the functional form `setCount(c => c + 1)` so each update builds on the previous pending state.

**246. Why does my `onClick={handleClick()}` fire immediately on render?**
You **called** the function during render instead of passing a reference. Use `onClick={handleClick}` or `onClick={() => handleClick(arg)}`. The parentheses invoke it right away and pass its return value as the handler.

**247. Why does `0` or `NaN` show up in my JSX?**
The `&&` short-circuit gotcha: `{count && <X/>}` renders `0` when `count` is `0` because `0` is a renderable value. Use an explicit boolean: `{count > 0 && <X/>}` or `{Boolean(count) && <X/>}`.

**248. Why is my `useEffect` causing an infinite loop?**
Typically you set state in the effect, and the effect's dependency is an object/array/function recreated every render (so it always "changes"), re-running the effect → setState → re-render → repeat. Fix: stabilize deps (`useMemo`/`useCallback`), use primitive deps, or move the value out. Also check you're not setting state unconditionally every run.

**249. Why does my fetch sometimes show stale data when switching items quickly?**
A **race condition**: an earlier request resolves after a later one and overwrites it. Guard with an `ignore` flag or `AbortController` in the effect cleanup, or use a query library that cancels/dedupes. Always key cleanup to the changing dependency (e.g., `id`).

**250. What's the difference between `e.preventDefault()` and `e.stopPropagation()`?**
`preventDefault()` stops the browser's **default behavior** (form submit navigation, link follow). `stopPropagation()` stops the event from **bubbling** up to parent handlers. They're independent — you might use one, both, or neither.

**251. Why doesn't my ref value show up immediately / cause a re-render?**
Mutating `ref.current` doesn't trigger a re-render by design — refs are for values that persist without rendering. If the UI must reflect the value, use state instead. Refs are also `null` until after the DOM commits, so reading a DOM ref during render gives `null` — read it in effects/handlers.

**252. Why are my context consumers all re-rendering on every provider render?**
The provider's `value` is a new object/array each render (`value={{a, b}}`), so `Object.is` sees a change and re-renders all consumers. Memoize the value (`useMemo`), split contexts, or use a selector-based store for fine-grained subscriptions.

**253. What happens if you forget the dependency array in `useEffect`?**
The effect runs after **every** render. If it sets state, you can get an infinite loop or excessive work. Always pass a deps array (`[]` for mount-only, `[deps]` for specific triggers) and let `exhaustive-deps` lint guide you.

**254. Can you use `async` directly as the `useEffect` callback?**
No — an `async` function returns a Promise, but `useEffect` expects the return to be a cleanup function (or nothing). Define an async function **inside** and call it: `useEffect(() => { const run = async () => {...}; run(); }, [])`. Or use an IIFE.

**255. What is the difference between `useEffect(fn)`, `useEffect(fn, [])`, and `useEffect(fn, [dep])`?**
No array → runs after every render. `[]` → runs once on mount (cleanup on unmount). `[dep]` → runs on mount and whenever `dep` changes (cleanup before each re-run and on unmount). The dependency array controls *when* the effect re-synchronizes.

**256. Why might two components with the same custom hook behave independently?**
Because custom hooks share **logic, not state** — each invocation runs its own hooks and gets isolated state. There's no shared instance unless you explicitly share via context or a store.

**257. What's the difference between mounting and rendering?**
**Rendering** is React calling your component to compute output (happens many times). **Mounting** is the first time a component is inserted into the DOM (happens once per instance, until unmounted). Effects with `[]` run on mount; effects with deps run on relevant renders.

**258. Why does changing a `key` reset my component?**
React identifies components by position + key. A changed key signals "this is a different component," so React unmounts the old instance (discarding its state) and mounts a fresh one. It's a deliberate way to reset state.

**259. What is the difference between `React.Fragment` and a `<div>`?**
A Fragment groups children **without** adding a DOM node; a `<div>` adds an extra element. Use Fragments to avoid breaking layouts (fl/grid/table) or polluting the DOM. Only `React.Fragment` (long form) accepts a `key`; the `<>` shorthand cannot.

**260. Rapid-fire one-liners:**
- **Keys must be:** stable, unique among siblings, derived from data (not index for dynamic lists).
- **Props are:** immutable, passed down, read-only in the child.
- **State updates are:** asynchronous and batched.
- **Hooks must be called:** at the top level, in the same order, from React functions only.
- **`useMemo` memoizes:** a value; **`useCallback`:** a function.
- **`useEffect`:** after paint; **`useLayoutEffect`:** before paint.
- **Render phase:** pure & interruptible; **commit phase:** side effects & synchronous.
- **Context change re-renders:** all consumers.
- **`React.memo`:** shallow-compares props to skip re-renders.
- **Server Components:** no state/effects, ship zero JS, run on server only.
- **Reconciliation diff is:** O(n) via type + key heuristics.
- **`dangerouslySetInnerHTML`:** bypasses XSS protection — sanitize first.

---

## FINAL PREP TIPS (FROM THE OTHER SIDE OF THE TABLE)

1. **Explain the "why," not just the "what."** Anyone can say "useMemo caches a value." Seniors explain *when reference stability matters* and *when it's wasted effort*.
2. **Know the render→commit model cold.** Half of React's "magic" (effects timing, concurrency, purity rules) follows from it.
3. **Have opinions, lightly held.** "I default to Zustand for client state and TanStack Query for server state, because..." reads as experience.
4. **Connect features to problems.** Fiber → interruptibility; keys → identity; Suspense → coordinated loading. Show the chain of reasoning.
5. **Admit trade-offs.** Every tool has costs. Naming them (memoization overhead, context re-renders, SSR complexity) signals maturity.
6. **Code the classics by hand:** a custom `useFetch`/`useDebounce`/`useLocalStorage`, a controlled form, a modal with a portal, an error boundary, a memoized list. These come up constantly.
7. **Stay current but grounded:** know React 18/19 (concurrency, RSC, the compiler) conceptually, but don't oversell features you haven't shipped.

> **Good luck - you've got this.** 
