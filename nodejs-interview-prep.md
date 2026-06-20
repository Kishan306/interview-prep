# The Complete Node.js Backend Interview Prep Sheet
### Beginner → Advanced+ | Node.js Core, Frameworks & TypeScript | Questions, Answers, Code & Follow-ups

> Built by a senior backend engineer's lens. Covers the Node.js runtime, async internals, the major frameworks (Express, NestJS, Fastify), databases/ORMs, security, scaling, testing, and production — with **TypeScript as the default**, because that's where serious Node backends live now.

**How to read this:**
- Each **main question** is numbered. Try to answer out loud before reading.
- **`↳ Follow-up:`** lines are the small "twist" questions an interviewer fires *right after* you answer — short enough to handle inline. If you can't answer the follow-up, you didn't really know the main answer.
- Bigger twists get promoted to their own numbered main question (cross-referenced).
- Code is TypeScript unless a point is JS-specific.

---

## TABLE OF CONTENTS

1. [Phase 1 — Node.js Fundamentals & Runtime](#p1)
2. [Phase 2 — The Event Loop, Async & Concurrency (Deep)](#p2)
3. [Phase 3 — Modules, npm & Package Management](#p3)
4. [Phase 4 — Asynchronous Patterns](#p4)
5. [Phase 5 — Streams & Buffers](#p5)
6. [Phase 6 — Core Modules: FS, Networking, Crypto, Workers](#p6)
7. [Phase 7 — TypeScript with Node.js](#p7)
8. [Phase 8 — Express.js](#p8)
9. [Phase 9 — NestJS](#p9)
10. [Phase 10 — Fastify, Koa & Framework Comparison](#p10)
11. [Phase 11 — REST API Design](#p11)
12. [Phase 12 — Databases, ORMs & Data Access](#p12)
13. [Phase 13 — Authentication & Security](#p13)
14. [Phase 14 — Error Handling, Logging & Observability](#p14)
15. [Phase 15 — Performance, Scaling & Architecture](#p15)
16. [Phase 16 — Testing](#p16)
17. [Phase 17 — Deployment, DevOps & Production](#p17)
18. [Phase 18 — Tricky / Gotcha & Rapid-Fire](#p18)

---

<a name="p1"></a>
## PHASE 1 — NODE.JS FUNDAMENTALS & RUNTIME

**1. What is Node.js?**
Node.js is a JavaScript **runtime** built on Chrome's **V8** engine that lets you run JavaScript outside the browser, primarily on the server. It pairs V8 (which compiles JS to machine code) with **libuv** (a C library providing the event loop and async I/O) and a set of core APIs (filesystem, networking, streams, crypto). Its defining trait is an **event-driven, non-blocking I/O** model that makes it efficient for I/O-heavy, concurrent workloads (APIs, real-time apps) on a single thread.

`↳ Follow-up: Is Node.js a language, framework, or runtime?` — A **runtime/environment**. JavaScript is the language; Express/Nest are frameworks; Node is the runtime that executes JS server-side.

`↳ Follow-up: Is Node single-threaded?` — Your JavaScript executes on a **single main thread** (one event loop). But Node isn't *entirely* single-threaded: libuv maintains a **thread pool** (default 4 threads) for certain async operations (file I/O, DNS, crypto, zlib), and you can spawn additional threads with `worker_threads`. So "single-threaded JS execution, multi-threaded under the hood."

**2. Why is Node.js good for some workloads and bad for others?**
Node excels at **I/O-bound, high-concurrency** work (REST/GraphQL APIs, proxies, real-time/websocket servers) because non-blocking I/O lets one thread juggle thousands of connections cheaply. It's **poor for CPU-bound** work (image processing, heavy computation, cryptographic mining) because long synchronous computation **blocks the single event loop**, stalling all other requests. For CPU work you offload to worker threads, child processes, a queue, or a different runtime.

`↳ Follow-up: What happens if you run a heavy for-loop in a request handler?` — It blocks the event loop; **every other pending request waits** until the loop finishes. The server appears frozen. (See Q22 on blocking.)

**3. What is V8?**
V8 is Google's open-source JavaScript (and WebAssembly) engine written in C++, used in Chrome and Node. It parses JS, compiles it to native machine code via JIT compilation (using the Ignition interpreter + TurboFan optimizing compiler), and manages memory with a generational garbage collector. Node embeds V8 to execute your JavaScript; libuv handles the async/I/O that V8 itself doesn't provide.

**4. What is libuv?**
libuv is the C library that gives Node its asynchronous superpowers: the **event loop**, a **thread pool**, and cross-platform async I/O (using `epoll` on Linux, `kqueue` on macOS, IOCP on Windows). When you do non-blocking file or network operations, libuv coordinates them and queues their callbacks back onto the event loop. It's the piece that makes "single-threaded but highly concurrent" possible.

**5. What is the difference between blocking and non-blocking code?**
**Blocking** code halts the thread until the operation completes (e.g., `fs.readFileSync`) — nothing else runs meanwhile. **Non-blocking** code initiates the operation and returns immediately, with a callback/promise invoked later when it finishes (e.g., `fs.readFile` / `fs.promises.readFile`), freeing the thread to do other work. Node's model favors non-blocking so the single thread isn't stuck waiting on I/O.

```ts
import { readFileSync } from 'node:fs';
import { readFile } from 'node:fs/promises';

// Blocking — thread waits here
const data1: Buffer = readFileSync('a.txt');

// Non-blocking — thread continues, result arrives later
const data2: Buffer = await readFile('a.txt');
```

`↳ Follow-up: When is sync code acceptable?` — At **startup/CLI scripts** (reading config once before the server accepts traffic), where blocking briefly is fine. Never inside a hot request path on a live server.

**6. What are the global objects in Node.js?**
Unlike browsers (which have `window`), Node provides globals like `global` (the global namespace), `process` (the running process), `Buffer`, `console`, timers (`setTimeout`, `setInterval`, `setImmediate`), and in CommonJS the module-scoped `__dirname`, `__filename`, `require`, `module`, `exports`. Modern Node also exposes web-standard globals like `fetch`, `URL`, `TextEncoder`, and `structuredClone`.

`↳ Follow-up: Are __dirname and __filename available in ES modules?` — **No.** In ESM they're not defined. You reconstruct them with `import.meta.url`:
```ts
import { fileURLToPath } from 'node:url';
import { dirname } from 'node:path';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```
(Recent Node versions also add `import.meta.dirname` and `import.meta.filename` directly.)

**7. What is the `process` object?**
`process` is a global giving information about and control over the current Node process: `process.env` (environment variables), `process.argv` (CLI arguments), `process.cwd()`, `process.pid`, `process.platform`, `process.exit(code)`, `process.nextTick()`, and lifecycle events (`process.on('exit' | 'uncaughtException' | 'unhandledRejection' | 'SIGTERM', ...)`). It's central to config, graceful shutdown, and signal handling.

```ts
const port = Number(process.env.PORT ?? 3000);
process.on('SIGTERM', () => { /* graceful shutdown */ });
```

**8. How do you read environment variables, and how should you manage config?**
Read via `process.env.MY_VAR` (always a `string | undefined`). Best practice: load from a `.env` file in development (Node 20.6+ has built-in `--env-file=.env`, or use `dotenv`), **never commit secrets**, validate and coerce types at startup with a schema (e.g., Zod), and fail fast if required vars are missing.

```ts
import { z } from 'zod';
const Env = z.object({
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  NODE_ENV: z.enum(['development', 'test', 'production']),
});
export const env = Env.parse(process.env); // throws if invalid → fail fast
```

`↳ Follow-up: Why validate env at startup instead of at usage?` — **Fail fast.** A misconfigured deploy crashes immediately on boot with a clear message, instead of throwing a confusing runtime error hours later deep in a request. Also gives you typed, coerced config everywhere downstream.

**9. What is the difference between `process.env.NODE_ENV` values?**
`NODE_ENV` is a convention (not enforced by Node) signaling the environment: `development`, `test`, `production`. Many libraries change behavior based on it — Express disables verbose error pages and enables view caching in `production`; bundlers strip dev code. Set it explicitly in deployment; don't rely on defaults.

**10. What's the difference between `node app.js` and `require`/`import`?**
`node app.js` launches a process with `app.js` as the entry module. `require()` (CommonJS) / `import` (ESM) load *other* modules into the current one. The entry file runs top to bottom; imported modules are evaluated once and **cached** — subsequent imports of the same module return the cached exports, not a fresh execution.

`↳ Follow-up: Is a module's code run every time you require it?` — **No.** It runs **once** on first load; the result is cached in `require.cache` (CJS) / the module registry (ESM). This makes modules effectively singletons — useful (shared DB pool) and occasionally surprising (shared mutable state).

**11. What is the REPL?**
REPL = Read-Eval-Print-Loop, the interactive Node shell you get by running `node` with no arguments. It reads an expression, evaluates it, prints the result, and loops — handy for quick experiments and debugging. Production code doesn't use it, but it's good for exploring APIs.

**12. How does Node handle concurrency without multiple threads for JS?**
Through the **event loop + non-blocking I/O**. Instead of one thread per connection (the classic thread-per-request model), Node uses one thread that registers I/O operations with the OS/libuv and continues; when an operation completes, its callback is queued and the loop runs it. This lets a single thread handle many thousands of concurrent connections with low memory overhead — as long as work stays I/O-bound, not CPU-bound. (Full mechanics in Phase 2.)

---

<a name="p2"></a>
## PHASE 2 — THE EVENT LOOP, ASYNC & CONCURRENCY (DEEP)

**13. Explain the Node.js event loop.**
The event loop is the mechanism that lets single-threaded Node perform non-blocking I/O. It continuously cycles through **phases**, each with its own callback queue, executing ready callbacks. When you call an async API, the operation is handed to libuv/the OS; when it completes, its callback is queued in the appropriate phase, and the loop picks it up on a later iteration ("tick"). Between phases, Node drains the **microtask** queues. This loop is why Node can wait on thousands of I/O operations without blocking.

**14. What are the phases of the event loop, in order?**
Per iteration, libuv processes these phases:
1. **Timers** — callbacks from `setTimeout`/`setInterval` whose threshold elapsed.
2. **Pending callbacks** — certain deferred system callbacks (e.g., some TCP errors).
3. **Idle/prepare** — internal use only.
4. **Poll** — retrieves new I/O events; executes I/O callbacks (the bulk of the work); may block here waiting for I/O.
5. **Check** — `setImmediate` callbacks.
6. **Close callbacks** — e.g., `socket.on('close', ...)`.
Between each phase (and after each callback), Node empties the **microtask queues**: `process.nextTick` queue first, then the Promise microtask queue.

**15. What's the difference between macrotasks and microtasks?**
**Macrotasks** are callbacks scheduled into event-loop phases: timers (`setTimeout`), I/O callbacks, `setImmediate`. **Microtasks** are higher-priority callbacks drained *between* macrotasks and *between phases*: `process.nextTick` (Node-specific, highest priority) and resolved **Promise** callbacks (`.then`/`await` continuations). After every single macrotask, Node fully drains all microtasks before moving on — so microtasks can starve the loop if they keep enqueuing more.

**16. `process.nextTick()` vs `setImmediate()` vs `setTimeout(fn, 0)` — what's the order?**
- `process.nextTick()` — runs **before** the event loop continues, after the current operation, ahead of Promise microtasks. Highest priority.
- Promise callbacks — run after `nextTick`, still as microtasks.
- `setImmediate()` — runs in the **Check** phase, after Poll.
- `setTimeout(fn, 0)` — runs in the **Timers** phase (next iteration); minimum delay is ~1ms.

```ts
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('nextTick'));
// Order: nextTick → promise → (timeout/immediate order varies at top level) 
```

`↳ Follow-up: Inside an I/O callback, which fires first — setTimeout(0) or setImmediate?` — **`setImmediate` always wins** inside an I/O cycle, because after the Poll phase the loop goes straight to the Check phase (setImmediate) before looping back to Timers. At the top level (outside I/O), their order is non-deterministic.

`↳ Follow-up: Why can process.nextTick be dangerous?` — Recursively scheduling `nextTick` callbacks **starves the event loop** — the loop never advances to I/O because it keeps draining the nextTick queue. Prefer `setImmediate` for "run after current operation but yield to the loop."

**17. What is the thread pool and what uses it?**
libuv maintains a thread pool (default **4** threads, set via `UV_THREADPOOL_SIZE`) for operations that can't be done asynchronously at the OS level. These include **file system** operations, **DNS lookups** (`dns.lookup`), **crypto** (`pbkdf2`, `scrypt`), and **zlib** compression. Network I/O (TCP/HTTP) generally does *not* use the pool — it uses the OS's async facilities directly. CPU-heavy crypto/compression on many concurrent requests can exhaust the 4 threads and create latency.

`↳ Follow-up: How do you increase the thread pool size?` — Set `process.env.UV_THREADPOOL_SIZE` **before** the first use (ideally as an env var at launch, since it's read once at startup). Max is 1024. But more threads isn't free — it competes for CPU cores.

**18. What does `async`/`await` actually do under the hood?**
`async` functions return a **Promise**. `await` pauses the async function, schedules the continuation as a **microtask** when the awaited promise settles, and yields control back to the event loop meanwhile (not blocking it). It's syntactic sugar over promises and generators — the function's remaining code becomes a `.then` callback. Crucially, `await` doesn't block the thread; it frees it to handle other work.

`↳ Follow-up: Does await block the event loop?` — **No.** It suspends only the current async function. The event loop continues running other callbacks. (Blocking the loop requires *synchronous* long work, not awaiting.)

**19. What is the difference between concurrency and parallelism in Node?**
**Concurrency** = managing many tasks that make progress over overlapping time (Node's event loop interleaving thousands of I/O operations on one thread). **Parallelism** = literally executing multiple tasks at the same instant on multiple cores (requires `worker_threads`, `cluster`, or child processes). Node gives you cheap concurrency out of the box; parallelism needs extra threads/processes.

**20. How do you run tasks in parallel with promises?**
Use the Promise combinators — they kick off operations concurrently (the I/O overlaps), even though JS execution is single-threaded:
```ts
// All must succeed; rejects on first failure
const [user, orders] = await Promise.all([getUser(id), getOrders(id)]);

// Wait for all, success or failure
const results = await Promise.allSettled([a(), b(), c()]);

// First to settle (resolve or reject)
const fastest = await Promise.race([primary(), fallback()]);

// First to fulfill (ignores rejections unless all reject)
const ok = await Promise.any([mirror1(), mirror2()]);
```

`↳ Follow-up: Difference between Promise.all and Promise.allSettled?` — `all` **rejects immediately** on the first rejection (and you lose the other results); `allSettled` **never rejects** — it resolves with an array of `{status: 'fulfilled', value}` / `{status: 'rejected', reason}` for every input. Use `allSettled` when partial failure is acceptable.

`↳ Follow-up: What's the bug in `await getA(); await getB();` if A and B are independent?` — You made them **sequential** (B waits for A) when they could run **concurrently**. Use `Promise.all([getA(), getB()])` to overlap them and roughly halve latency.

**21. What is an `EventEmitter`?**
`EventEmitter` (from `node:events`) is the core pub/sub primitive underpinning much of Node (streams, HTTP servers, sockets all extend it). You `emit` named events and register `on`/`once` listeners. It's synchronous by default — listeners run in registration order when `emit` is called.
```ts
import { EventEmitter } from 'node:events';
class Orders extends EventEmitter {}
const orders = new Orders();
orders.on('created', (id: string) => console.log('order', id));
orders.emit('created', 'ord_123');
```

`↳ Follow-up: Are EventEmitter listeners async?` — **No, they're synchronous** — `emit` calls each listener inline, in order, before returning. If a listener throws and it's the `error` event with no handler, the process crashes. For async work, the listener can kick off a promise, but `emit` won't await it.

`↳ Follow-up: What's the "max listeners" warning?` — By default an emitter warns at **10** listeners for one event (a leak heuristic). Raise it with `emitter.setMaxListeners(n)` if intentional, or fix the leak if you're adding listeners without removing them.

**22. What does "blocking the event loop" mean and how do you avoid it?**
A long **synchronous** operation (big loop, `JSON.parse` of a huge payload, sync crypto, regex catastrophic backtracking) monopolizes the single thread, so no other callbacks — including other requests — can run until it finishes. Avoid by: offloading CPU work to `worker_threads`/child processes/a queue, chunking work with `setImmediate`, using streaming/async APIs, and validating input sizes. Monitor with event-loop-lag metrics.

`↳ Follow-up: Name a sneaky way to block the loop.` — A pathological **regular expression** (ReDoS) on attacker-controlled input, or `JSON.parse`/`JSON.stringify` on a very large object. Both are synchronous and can freeze the server.

**23. What are `worker_threads` and when do you use them?**
`worker_threads` let you run JavaScript on **separate threads** within the same process, sharing memory via `SharedArrayBuffer` and communicating via message passing. Use them for **CPU-bound** tasks (parsing, image/video processing, encryption) so they don't block the main event loop. Unlike child processes, workers are lighter (same process, no separate V8 startup per task if pooled).
```ts
import { Worker } from 'node:worker_threads';
const worker = new Worker('./heavy-task.js', { workerData: { n: 42 } });
worker.on('message', (result) => console.log(result));
```

`↳ Follow-up: worker_threads vs cluster vs child_process?` — `worker_threads`: multiple threads in one process, shared memory, for CPU work. `cluster`: forks multiple **processes** sharing a server port to use all cores for handling requests. `child_process`: spawns separate programs/processes (`exec`, `spawn`, `fork`), full isolation, good for running external commands or fully separate Node scripts. (More in Phase 15.)

---

<a name="p3"></a>
## PHASE 3 — MODULES, npm & PACKAGE MANAGEMENT

**24. What's the difference between CommonJS and ES Modules?**
**CommonJS (CJS)** — Node's original system: `require()` to import, `module.exports`/`exports` to export. Synchronous, loaded at runtime, supports dynamic `require()`. **ES Modules (ESM)** — the JS standard: `import`/`export`, statically analyzable (enables tree-shaking), asynchronous loading, top-level `await`. ESM is the future and the default for new projects; CJS still dominates legacy code and many npm packages.

| | CommonJS | ESM |
|---|---|---|
| Import | `require()` | `import` |
| Export | `module.exports` | `export` |
| Loading | Synchronous | Asynchronous |
| Resolution | Runtime, dynamic | Static (mostly) |
| Top-level await | No | Yes |
| File hint | `.cjs` / default | `.mjs` / `"type":"module"` |

`↳ Follow-up: How does Node decide if a .js file is CJS or ESM?` — By the nearest `package.json`'s `"type"` field: `"type":"module"` → `.js` is ESM; `"type":"commonjs"` or absent → CJS. Explicit `.mjs` is always ESM; `.cjs` is always CJS.

`↳ Follow-up: Can you require() an ESM module?` — Historically no (you used dynamic `import()`), but recent Node (22+) added experimental `require()` of synchronous ESM. The safe, portable answer: use dynamic `import()` to load ESM from CJS.

**25. What's the difference between `module.exports` and `exports`?**
`exports` is initially a **reference** to `module.exports`. Adding properties (`exports.foo = ...`) works because both point to the same object. But **reassigning** `exports = {...}` breaks the link — only `module.exports` is actually returned by `require`. Rule: to export a single thing, assign `module.exports = ...`; to export multiple named things, attach to `exports` (or `module.exports`).

```js
exports.add = (a, b) => a + b;        // works
module.exports = class Service {};     // works (single export)
exports = { add };                     // BROKEN — reassigns local ref only
```

**26. How does Node's module resolution work?**
For `require('x')` / `import 'x'`: (1) core module? use it. (2) starts with `./` `../` `/`? resolve as a file/directory relative path (trying `.js`, `.json`, `.node`, then `index.*`, then `package.json` `main`/`exports`). (3) otherwise, walk up `node_modules` folders from the current directory to the root, looking for the package. ESM adds the `"exports"` map and stricter rules (file extensions often required).

`↳ Follow-up: What is the "exports" field in package.json?` — It defines a package's **public entry points** and can map different paths/conditions (`import` vs `require`, `node` vs `browser`) to different files, while **encapsulating** internals (consumers can't import unlisted deep paths). It's the modern replacement for relying on `main` + deep `require`.

**27. What is `package.json` and its key fields?**
The manifest describing a Node project: `name`, `version`, `type` (module system), `main`/`exports` (entry points), `scripts` (npm run commands), `dependencies`, `devDependencies`, `peerDependencies`, `engines` (Node version), `bin` (CLI executables). It's the contract npm/yarn/pnpm use to install and run your project.

**28. dependencies vs devDependencies vs peerDependencies vs optionalDependencies?**
- **dependencies** — needed at runtime in production (express, pg).
- **devDependencies** — only for development/build/test (typescript, jest, eslint); not installed with `--production`/`--omit=dev`.
- **peerDependencies** — packages the host app must provide (a plugin declaring it needs a specific React/Nest version), avoiding duplicate installs.
- **optionalDependencies** — install failures are non-fatal; code must handle absence.

`↳ Follow-up: Why does it matter for Docker image size?` — Installing with `--omit=dev` (or a multi-stage build) excludes devDependencies, shrinking the production image and attack surface.

**29. What is semantic versioning (semver)?**
`MAJOR.MINOR.PATCH` (e.g., `4.18.2`): **MAJOR** = breaking changes, **MINOR** = backward-compatible features, **PATCH** = backward-compatible bug fixes. Range operators: `^4.18.2` allows minor+patch up to `<5.0.0`; `~4.18.2` allows patch up to `<4.19.0`; exact `4.18.2` pins. Understanding ranges prevents surprise breakage.

`↳ Follow-up: What does the lockfile (package-lock.json / pnpm-lock.yaml) do?` — It records the **exact resolved versions** (and integrity hashes) of the entire dependency tree, so installs are **reproducible** across machines/CI. Commit it. `npm ci` installs strictly from it.

`↳ Follow-up: Difference between npm install and npm ci?` — `npm install` may update the lockfile and node_modules to satisfy `package.json` ranges; `npm ci` does a clean, **exact** install from the lockfile (deletes node_modules first), failing if lockfile and package.json disagree. Use `npm ci` in CI/CD.

**30. npm vs yarn vs pnpm?**
All install dependencies. **npm** — bundled with Node, ubiquitous. **yarn** — introduced lockfiles/workspaces early, fast, plug-n-play option. **pnpm** — uses a **content-addressable store with hard links/symlinks**, so packages are stored once on disk and shared across projects — dramatically less disk usage and faster, with stricter dependency isolation (no phantom dependencies). pnpm is increasingly the senior choice for monorepos.

**31. What are npm workspaces / monorepos?**
Workspaces let one repository host multiple packages with shared tooling and cross-linked local dependencies (npm/yarn/pnpm workspaces, or tools like Turborepo/Nx). Benefits: code sharing, atomic cross-package changes, unified CI. Common for a backend with shared `types`, `config`, and multiple services.

**32. How do `npm scripts` work?**
The `scripts` field maps names to shell commands run via `npm run <name>` (`start`, `test`, `build` have shortcuts). They get `node_modules/.bin` on the PATH, so you can call locally-installed CLIs (`tsc`, `jest`) without global installs. Pre/post hooks (`prebuild`, `postinstall`) run automatically around the named script.

`↳ Follow-up: Security concern with postinstall scripts?` — Malicious packages can run arbitrary code in `postinstall` during `npm install` — a supply-chain attack vector. Mitigate with `npm install --ignore-scripts`, lockfiles, dependency auditing, and minimal dependencies.

---

<a name="p4"></a>
## PHASE 4 — ASYNCHRONOUS PATTERNS

**33. What is callback hell and how do you escape it?**
Callback hell (the "pyramid of doom") is deeply nested callbacks for sequential async steps, making code unreadable and error handling repetitive. Escapes: **Promises** (flat `.then` chains), **async/await** (synchronous-looking flow), named functions instead of inline anonymous ones, and modularization.
```ts
// Hell
getUser(id, (e, u) => { getOrders(u, (e, o) => { getItems(o, (e, i) => {/*...*/}); }); });
// async/await
const u = await getUser(id);
const o = await getOrders(u);
const i = await getItems(o);
```

**34. What is a Promise and its states?**
A Promise represents the eventual result of an async operation, in one of three states: **pending** → **fulfilled** (resolved with a value) or **rejected** (with a reason). Once settled, it's immutable. You consume it via `.then`/`.catch`/`.finally` or `await`. Promises solve callback composition and error propagation.

`↳ Follow-up: Can a Promise change state after settling?` — **No.** Once fulfilled or rejected, it's locked; further resolve/reject calls are ignored.

**35. How do you handle errors with async/await?**
Wrap awaited calls in `try/catch`. For Express-style handlers, either wrap each handler or use an async-error wrapper (Express 5 handles async rejections automatically; Express 4 doesn't — see Q63).
```ts
try {
  const data = await risky();
} catch (err) {
  // handle / rethrow / log
} finally {
  // cleanup
}
```

`↳ Follow-up: What happens to an unhandled promise rejection?` — Node emits an `unhandledRejection` event and (in modern Node) **terminates the process by default**. Always attach a `.catch` or `try/catch`, and add a top-level `process.on('unhandledRejection')` handler for logging before crash.

`↳ Follow-up: Why is `forEach` with async callbacks a trap?` — `array.forEach(async ...)` **doesn't await** the callbacks — it fires them all and moves on, so you can't sequence or catch errors properly. Use a `for...of` loop with `await` (sequential) or `Promise.all(array.map(async ...))` (concurrent).

**36. What's the difference between sequential and concurrent awaits?**
Sequential `await`s run one after another (total time = sum). Independent operations should run concurrently with `Promise.all` (total time ≈ slowest one). Only keep them sequential when a later call **depends** on an earlier result.
```ts
// Sequential (slow if independent)
const a = await taskA(); const b = await taskB();
// Concurrent
const [a2, b2] = await Promise.all([taskA(), taskB()]);
```

**37. How do you convert a callback-based API to a Promise?**
Use `util.promisify` for standard error-first callbacks, or wrap manually.
```ts
import { promisify } from 'node:util';
import { readFile } from 'node:fs';
const readFileAsync = promisify(readFile);

// Manual
const wait = (ms: number) => new Promise<void>((resolve) => setTimeout(resolve, ms));
```

`↳ Follow-up: What is the "error-first callback" convention?` — Node callbacks take `(err, result)` where `err` is `null` on success and an `Error` on failure: `(err, data) => { if (err) ...; }`. `promisify` relies on this convention.

**38. What is an async iterator / `for await...of`?**
A protocol for iterating over async data sources (streams, paginated APIs) where each step returns a Promise. `for await...of` consumes them with clean syntax and proper backpressure.
```ts
for await (const chunk of readableStream) {
  process(chunk);
}
```

**39. How do you implement a timeout for an async operation?**
Race the operation against a timeout promise, or use `AbortController` (the modern, cancellation-aware approach).
```ts
const controller = new AbortController();
const t = setTimeout(() => controller.abort(), 5000);
try {
  const res = await fetch(url, { signal: controller.signal });
} finally { clearTimeout(t); }
```

`↳ Follow-up: What is AbortController used for?` — A standard way to **cancel** async operations (fetch, streams, timers, custom logic). You pass `controller.signal` into the operation; calling `controller.abort()` rejects/aborts it. Essential for request timeouts and cleanup on client disconnect.

**40. What is a race condition in async Node code, and how do you prevent it?**
When the correctness of the result depends on the unpredictable ordering of concurrent async operations — e.g., two requests both read-then-write the same record, and one overwrites the other ("lost update"). Prevent with: database transactions, optimistic/pessimistic locking, atomic operations, queues/serialization, or idempotency keys.

`↳ Follow-up: Give a concrete example.` — Two concurrent "increment balance" requests both read balance=100, both compute 110, both write 110 — final is 110 instead of 120. Fix with an atomic DB update (`UPDATE ... SET balance = balance + 10`) or a transaction with row locking.

---

<a name="p5"></a>
## PHASE 5 — STREAMS & BUFFERS

**41. What are Streams in Node.js and why use them?**
Streams process data **in chunks** over time rather than loading it all into memory. They're ideal for large files, network transfers, and pipelines — keeping memory flat regardless of data size. Reading a 10GB file with `readFile` would exhaust memory; streaming it processes it piece by piece.

**42. What are the four types of streams?**
- **Readable** — source you read from (`fs.createReadStream`, HTTP request).
- **Writable** — destination you write to (`fs.createWriteStream`, HTTP response).
- **Duplex** — both readable and writable (TCP socket).
- **Transform** — a duplex stream that modifies data passing through (zlib gzip, encryption).

**43. What is `pipe()` and the `pipeline()` utility?**
`pipe()` connects a readable to a writable, automatically managing data flow and backpressure: `readable.pipe(writable)`. `stream.pipeline()` (preferred) chains multiple streams **with proper error propagation and cleanup**, which `pipe()` lacks.
```ts
import { pipeline } from 'node:stream/promises';
import { createReadStream, createWriteStream } from 'node:fs';
import { createGzip } from 'node:zlib';

await pipeline(
  createReadStream('input.txt'),
  createGzip(),
  createWriteStream('input.txt.gz'),
);
```

`↳ Follow-up: Why prefer pipeline() over pipe()?` — `pipe()` doesn't forward errors or destroy streams on failure, leaking file descriptors. `pipeline()` propagates errors, destroys all streams on failure, and (the promise version) gives you a clean `await`. Always use `pipeline` for multi-stage flows.

**44. What is backpressure?**
Backpressure is the feedback mechanism preventing a fast producer from overwhelming a slow consumer. When a writable's internal buffer fills, `write()` returns `false`, signaling the readable to pause until the `drain` event. `pipe`/`pipeline` handle this automatically; manual writing must respect the return value.

`↳ Follow-up: What happens if you ignore backpressure?` — Unbounded memory growth — the producer keeps buffering data the consumer can't keep up with, eventually crashing the process with OOM.

**45. What is a Buffer?**
A `Buffer` is a fixed-length chunk of **raw binary data** outside V8's heap, used for I/O (files, network, crypto). It predates JS's `Uint8Array` (which it now subclasses). You handle Buffers whenever working with binary protocols, file bytes, or encoding conversions.
```ts
const buf = Buffer.from('héllo', 'utf8');
console.log(buf.length);              // byte length (6, not 5 — é is 2 bytes)
console.log(buf.toString('base64'));
```

`↳ Follow-up: Why is `Buffer.from` preferred over `new Buffer()`?` — `new Buffer()` is **deprecated and unsafe** — older signatures could allocate uninitialized memory (leaking old data) or be exploited. Use `Buffer.from`, `Buffer.alloc` (zero-filled), or `Buffer.allocUnsafe` (fast but must be overwritten).

`↳ Follow-up: Difference between Buffer.alloc and Buffer.allocUnsafe?` — `alloc(n)` returns zero-filled memory (safe). `allocUnsafe(n)` skips zeroing for speed but may contain **old memory contents** — you must fully overwrite it before exposing it, or you risk leaking sensitive data.

**46. When would you use streams in a real backend?**
Serving/processing large file uploads/downloads, proxying responses, CSV/log processing, on-the-fly compression (gzip), transcoding, and piping DB query results to the client. Anywhere data is large or unbounded, streaming keeps memory predictable and improves time-to-first-byte.

---

<a name="p6"></a>
## PHASE 6 — CORE MODULES: FS, NETWORKING, CRYPTO, WORKERS

**47. What core modules should every backend dev know?**
`fs`/`fs/promises` (filesystem), `path` (cross-platform paths), `http`/`https`/`http2` (servers/clients), `net` (TCP), `url` (parsing), `crypto` (hashing/encryption), `os` (system info), `events` (EventEmitter), `stream`, `zlib` (compression), `worker_threads`, `cluster`, `child_process`, `util`, and the modern `node:test` runner. Prefix with `node:` (e.g., `import { readFile } from 'node:fs/promises'`) to disambiguate from npm packages.

**48. Why use the `path` module instead of string concatenation?**
File paths differ across OSes (`/` vs `\`). `path.join`, `path.resolve`, `path.basename`, `path.extname` handle separators, normalization, and edge cases correctly and portably. String concatenation breaks on Windows and mishandles `..`/trailing slashes.
```ts
import { join, resolve } from 'node:path';
const file = join(__dirname, 'uploads', filename); // safe & portable
```

`↳ Follow-up: path.join vs path.resolve?` — `join` concatenates segments with the separator. `resolve` produces an **absolute** path, processing from right to left until it builds one (using `cwd` if needed). Use `resolve` when you need an absolute path.

**49. How do you create a basic HTTP server without a framework?**
```ts
import { createServer } from 'node:http';
const server = createServer((req, res) => {
  if (req.url === '/health' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ status: 'ok' }));
  } else {
    res.writeHead(404).end();
  }
});
server.listen(3000);
```
Frameworks (Express/Fastify) build on this, adding routing, middleware, and parsing.

**50. What does the `crypto` module provide, and how do you hash passwords?**
`crypto` offers hashing (`createHash`), HMAC, key derivation (`pbkdf2`, `scrypt`), symmetric/asymmetric encryption, random bytes (`randomBytes`, `randomUUID`), and signing. **Never** hash passwords with plain `sha256`/`md5` — use a slow, salted KDF like **bcrypt** (npm), `scrypt`, or argon2.
```ts
import { randomUUID, scrypt, randomBytes } from 'node:crypto';
const id = randomUUID();
// password hashing with bcrypt (npm) is most common in interviews
```

`↳ Follow-up: Why not use SHA-256 for passwords?` — It's **too fast** — attackers can brute-force billions/sec on GPUs. Password hashes must be deliberately **slow** and salted (bcrypt/scrypt/argon2) to resist brute force and rainbow tables. (More in Phase 13.)

`↳ Follow-up: How do you generate cryptographically secure random values?` — `crypto.randomBytes()` or `crypto.randomUUID()` / `crypto.randomInt()`. **Never** `Math.random()` for tokens, IDs, or secrets — it's predictable and not cryptographically secure.

**51. What is `cluster` and how does it relate to CPU cores?**
The `cluster` module forks the Node process into multiple **worker processes** that share the same server port (the OS/Node load-balances connections across them), letting you utilize all CPU cores despite each process being single-threaded. Common in production to scale across cores (or use a process manager like PM2 that does this for you).
```ts
import cluster from 'node:cluster';
import { cpus } from 'node:os';
if (cluster.isPrimary) {
  for (let i = 0; i < cpus().length; i++) cluster.fork();
} else {
  startServer();
}
```

`↳ Follow-up: cluster vs a load balancer / multiple containers?` — `cluster` scales across cores on **one machine/process group**. In containerized/cloud setups, you often instead run **one process per container** and scale horizontally with an orchestrator (Kubernetes) + load balancer — simpler, more observable, and the modern default. Know both.

`↳ Follow-up: How do clustered workers share state?` — They **don't** share memory — each is a separate process. Shared state must live in an external store (Redis, DB). In-memory caches/sessions per worker cause inconsistency, which is why you externalize sessions.

**52. How do you handle graceful shutdown?**
On `SIGTERM`/`SIGINT`, stop accepting new connections, finish in-flight requests, close DB pools and other resources, then exit. Critical for zero-downtime deploys and not dropping requests.
```ts
process.on('SIGTERM', async () => {
  server.close(async () => {        // stop accepting new connections
    await db.end();                 // close DB pool
    process.exit(0);
  });
  setTimeout(() => process.exit(1), 10_000).unref(); // force-exit safety net
});
```

`↳ Follow-up: Why the forced-exit timeout?` — If a connection hangs, `server.close` may never call back, leaving the process stuck. The timeout guarantees the process eventually exits so the orchestrator can replace it. `.unref()` so the timer itself doesn't keep the process alive.

**53. What is `child_process` and its methods?**
Runs external commands or separate Node scripts as separate processes: `exec` (buffers output, shell), `execFile` (no shell, safer), `spawn` (streams output, good for long/large output), `fork` (spawns a Node child with an IPC channel). Use for shelling out, running CLIs, or isolating work.

`↳ Follow-up: exec vs spawn — which for large output and why?` — `spawn` — it **streams** output incrementally, so it handles large/continuous output without buffering it all in memory. `exec` buffers everything and can blow its `maxBuffer` limit. Also prefer `execFile`/`spawn` (no shell) to avoid **shell injection**.

---

<a name="p7"></a>
## PHASE 7 — TYPESCRIPT WITH NODE.JS

**54. Why use TypeScript for a Node backend?**
TypeScript adds **static typing** to JS, catching errors at compile time (typos, wrong shapes, null misuse), enabling confident refactoring, providing first-class editor autocomplete/IntelliSense, and serving as living documentation. For backends with complex domain models, request/response contracts, and DB schemas, it dramatically reduces runtime bugs and improves maintainability — which is why it's the de-facto standard for serious Node services.

**55. How do you set up a TypeScript Node project (tsconfig essentials)?**
Install `typescript` + `@types/node`, add a `tsconfig.json`. Key options for backends:
```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",          // modern Node module resolution
    "moduleResolution": "NodeNext",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,                 // enable all strict checks
    "esModuleInterop": true,
    "skipLibCheck": true,
    "sourceMap": true,
    "declaration": true
  },
  "include": ["src/**/*"]
}
```

`↳ Follow-up: What does "strict": true actually turn on?` — A bundle including `strictNullChecks` (null/undefined must be handled), `noImplicitAny`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `noImplicitThis`, `useUnknownInCatchVariables`. It's the single most valuable setting — always enable it.

`↳ Follow-up: Why @types/node?` — TypeScript doesn't ship Node's API types. `@types/node` provides type definitions for `fs`, `http`, `process`, etc., so Node's built-ins are typed.

**56. How do you run TypeScript in development vs production?**
- **Dev:** `tsx` or `ts-node` (run `.ts` directly without a separate build step), often with watch mode (`tsx watch src/index.ts`). Recent Node (22.6+) also has experimental built-in `--experimental-strip-types`.
- **Production:** **compile ahead of time** with `tsc` (or `esbuild`/`swc` for speed) to plain JS in `dist/`, then run `node dist/index.js`. Don't run `ts-node` in production — it's slower and adds runtime overhead.

`↳ Follow-up: tsc vs esbuild/swc for building?` — `tsc` does full type-checking but is slower. `esbuild`/`swc` transpile **much** faster but **don't type-check** (they strip types). Common setup: type-check with `tsc --noEmit` in CI, build with esbuild/swc for speed.

**57. What's the difference between `interface` and `type` in TypeScript?**
Both describe object shapes. `interface` supports **declaration merging** and is idiomatic for object/class contracts and public APIs; it can be `extends`-ed. `type` is more flexible — it can represent unions, intersections, tuples, mapped and conditional types, and primitives. Rule of thumb: `interface` for object shapes you might extend; `type` for unions and complex compositions.

```ts
interface User { id: string; email: string; }
type Result<T> = { ok: true; value: T } | { ok: false; error: string };  // union — needs type
```

**58. How do you type Express request/response objects?**
Use the `@types/express` types and generics on `Request`/`Response`. Type params, query, body, and response body:
```ts
import { Request, Response } from 'express';

interface CreateUserBody { email: string; name: string; }
interface UserParams { id: string; }

app.post('/users', (req: Request<{}, {}, CreateUserBody>, res: Response) => {
  const { email, name } = req.body; // typed
  res.status(201).json({ id: '1', email });
});

app.get('/users/:id', (req: Request<UserParams>, res: Response) => {
  const { id } = req.params;        // typed as string
});
```

`↳ Follow-up: How do you safely type req.body when input is untrusted?` — Types are **compile-time only** — they don't validate runtime input. Validate with a schema library (**Zod**, Joi, class-validator) and **infer** the TS type from the schema, so the type and the runtime check stay in sync. (See Q59.)

**59. How do you get runtime validation AND static types from one source?**
Define a schema (Zod) and derive the type with `z.infer`. One source of truth — the validator guarantees the data matches the type at runtime.
```ts
import { z } from 'zod';

const CreateUser = z.object({
  email: z.string().email(),
  age: z.number().int().positive(),
});
type CreateUser = z.infer<typeof CreateUser>;  // { email: string; age: number }

app.post('/users', (req, res) => {
  const result = CreateUser.safeParse(req.body);
  if (!result.success) return res.status(400).json(result.error.issues);
  const data: CreateUser = result.data;  // validated + typed
});
```

`↳ Follow-up: Why is this better than just casting `req.body as CreateUser`?` — A cast is a **lie to the compiler** — it asserts a type without checking, so malformed input flows through as if valid, causing runtime crashes or security holes. Schema validation actually verifies the data.

**60. What are utility types you'll actually use in backends?**
`Partial<T>` (all optional — patch DTOs), `Required<T>`, `Pick<T, K>`/`Omit<T, K>` (derive DTOs from entities, e.g., `Omit<User, 'passwordHash'>` for responses), `Record<K, V>` (maps/dictionaries), `Readonly<T>`, `Awaited<T>` (unwrap a Promise's type), `ReturnType<F>`/`Parameters<F>`. These keep DTOs in sync with domain models without duplication.
```ts
type User = { id: string; email: string; passwordHash: string };
type PublicUser = Omit<User, 'passwordHash'>;       // safe response shape
type UserPatch = Partial<Pick<User, 'email'>>;       // PATCH body
```

**61. What are generics and where do they show up in Node backends?**
Generics parameterize types for reuse while preserving type safety: typed repositories, API response wrappers, service result types, middleware.
```ts
interface Repository<T, ID = string> {
  findById(id: ID): Promise<T | null>;
  create(data: Omit<T, 'id'>): Promise<T>;
}

type ApiResponse<T> = { data: T; meta?: { page: number; total: number } };
```

**62. What is `unknown` vs `any`, and why does it matter for safety?**
`any` **disables** type checking — it propagates and silently defeats TS. `unknown` is the type-safe top type: you can hold any value but **must narrow** it (via type guards) before using it. Use `unknown` for untrusted/external data (parsed JSON, caught errors) to force explicit handling. Modern TS (`strict`) types `catch` variables as `unknown` for this reason.
```ts
try { /* ... */ } catch (err: unknown) {
  if (err instanceof Error) logger.error(err.message); // must narrow
}
```

`↳ Follow-up: Why does TypeScript type the catch variable as unknown now?` — Because a thrown value can be **anything** (you can `throw 'string'` or `throw 42`), not just an `Error`. `unknown` forces you to check before assuming `.message` exists, preventing crashes on non-Error throws.

---

<a name="p8"></a>
## PHASE 8 — EXPRESS.JS

**63. What is Express and what is middleware?**
Express is a minimal, unopinionated web framework for Node — routing + middleware over the `http` module. **Middleware** are functions with the signature `(req, res, next)` that run in order on the request/response cycle; each can read/modify `req`/`res`, end the response, or call `next()` to pass control on. Middleware powers logging, parsing, auth, validation, and error handling.
```ts
import express, { Request, Response, NextFunction } from 'express';
const app = express();

const logger = (req: Request, _res: Response, next: NextFunction) => {
  console.log(`${req.method} ${req.url}`);
  next();                              // pass to next middleware
};
app.use(express.json());               // body parser middleware
app.use(logger);
```

`↳ Follow-up: What happens if you forget to call next() (and don't send a response)?` — The request **hangs** — control never passes on and no response is sent, so the client waits until timeout. Every middleware must either end the response or call `next()`.

`↳ Follow-up: What's the order of middleware execution?` — **Top to bottom, in the order registered.** This is why body parsers and auth must be registered before the routes that need them, and error handlers last.

**64. What's the difference between application-level, router-level, and error-handling middleware?**
- **Application-level:** `app.use(fn)` / `app.METHOD` — runs for matching requests app-wide.
- **Router-level:** same but on an `express.Router()` instance — modularizes routes.
- **Error-handling:** has **four** parameters `(err, req, res, next)` — Express recognizes the arity and routes errors here. Must be registered **last**.
- **Built-in/third-party:** `express.json()`, `helmet`, `cors`, etc.

**65. How does error-handling middleware work?**
An error middleware has four args. You reach it by calling `next(err)` or (Express 5) throwing in an async handler. Centralize error formatting here.
```ts
app.use((err: Error, _req: Request, res: Response, _next: NextFunction) => {
  const status = err instanceof HttpError ? err.status : 500;
  res.status(status).json({ error: err.message });
});
```

`↳ Follow-up: How do async errors differ between Express 4 and 5?` — In **Express 4**, a rejected promise/throw in an `async` handler is **not** caught automatically — it escapes to `unhandledRejection` and won't hit your error middleware unless you wrap handlers (e.g., `express-async-errors` or a `catchAsync` wrapper). **Express 5** automatically forwards rejected async handlers to error middleware.

```ts
// Express 4 wrapper pattern
const catchAsync = (fn: RequestHandler): RequestHandler =>
  (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
app.get('/users/:id', catchAsync(async (req, res) => { /* may throw */ }));
```

**66. What does `express.json()` / body parsing do, and what's a security concern?**
`express.json()` parses incoming JSON request bodies into `req.body`. Security: set a **size limit** (`express.json({ limit: '100kb' })`) to prevent large-payload DoS, and validate the parsed body (types don't validate runtime input).

**67. How do you structure a production Express app?**
Layered separation: **routes** (define endpoints) → **controllers** (handle req/res, no business logic) → **services** (business logic, framework-agnostic) → **repositories/data access** (DB). Plus middleware, config, validation schemas, and error handling. This keeps business logic testable and decoupled from Express.
```
src/
  routes/        controllers/    services/
  repositories/  middleware/     schemas/
  config/        app.ts          server.ts
```

`↳ Follow-up: Why keep business logic out of controllers?` — So it's **testable without HTTP**, reusable across transports (HTTP, queue consumer, CLI), and not coupled to Express. Controllers should just translate HTTP ↔ service calls.

**68. What is `app.use` vs `app.get`/`app.post`?**
`app.use([path], middleware)` mounts middleware for **all** HTTP methods (and matches path prefixes). `app.get`/`app.post`/etc. register handlers for a **specific method + exact route**. Use `app.use` for cross-cutting concerns, method verbs for endpoints.

**69. How do route parameters, query strings, and wildcards work?**
`req.params` (`/users/:id` → `req.params.id`), `req.query` (`?page=2` → `req.query.page`), and route patterns. Note `req.query` values are strings (or arrays) — coerce/validate them.

`↳ Follow-up: What type is req.query.page?` — `string | string[] | ParsedQs | undefined` — **not a number**. You must parse and validate it (`Number(req.query.page)` with a check, or a schema). A classic bug is treating it as a number directly.

**70. How do you handle CORS in Express?**
Use the `cors` middleware. Configure allowed origins explicitly in production — don't reflect all origins with credentials.
```ts
import cors from 'cors';
app.use(cors({ origin: ['https://app.example.com'], credentials: true }));
```

`↳ Follow-up: Why is `cors({ origin: '*', credentials: true })` invalid?` — The CORS spec **forbids** wildcard origin together with credentials (cookies/auth headers). Browsers reject it. You must specify exact origins when `credentials: true`.

---

<a name="p9"></a>
## PHASE 9 — NESTJS

**71. What is NestJS and why use it?**
NestJS is an opinionated, **TypeScript-first** backend framework built on top of Express (or Fastify) that brings **structure** to Node: modular architecture, **dependency injection**, decorators, and clear separation (controllers, providers/services, modules). It's heavily inspired by Angular. Use it for large, team-based applications where consistency, testability, and scalability matter more than minimalism. It bundles solutions for validation, guards, interceptors, microservices, GraphQL, WebSockets, and more.

**72. What are the core building blocks of NestJS?**
- **Modules** (`@Module`) — organize the app into cohesive feature units; the DI boundary.
- **Controllers** (`@Controller`) — handle incoming requests, map routes.
- **Providers/Services** (`@Injectable`) — business logic, injected via DI.
- **Pipes** — transform/validate input.
- **Guards** — authorization/authentication (can this request proceed?).
- **Interceptors** — wrap method execution (logging, transforming responses, caching).
- **Filters** — handle exceptions.

```ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {} // DI

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}
```

**73. What is Dependency Injection and how does Nest implement it?**
DI is a pattern where a class **receives** its dependencies from an external injector rather than constructing them itself — decoupling components and making them swappable/testable. Nest has a built-in **IoC (Inversion of Control) container** that reads constructor parameter types (via metadata), resolves the dependency graph, and injects instances. Providers are registered in modules and (by default) are **singletons** within their scope.

```ts
@Injectable()
export class UsersService {
  constructor(private readonly repo: UsersRepository) {} // injected automatically
}
```

`↳ Follow-up: Why is DI good for testing?` — You can inject **mocks/stubs** for dependencies, testing a service in isolation without real DBs or HTTP. Nest's `Test.createTestingModule` makes overriding providers trivial.

`↳ Follow-up: What are provider scopes in Nest?` — `DEFAULT` (singleton — one instance shared), `REQUEST` (new instance per request — needed for request-specific state, but slower), `TRANSIENT` (new instance per consumer). Default singleton is best for performance; use REQUEST scope only when necessary.

**74. What are Pipes and how do you do validation in Nest?**
Pipes transform or validate arguments before they reach the handler. The common combo: `class-validator` + `class-transformer` decorators on a DTO, enforced globally by `ValidationPipe`.
```ts
import { IsEmail, IsInt, Min } from 'class-validator';
export class CreateUserDto {
  @IsEmail() email: string;
  @IsInt() @Min(0) age: number;
}
// main.ts
app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }));
```

`↳ Follow-up: What does `whitelist: true` do?` — Strips properties **not** declared in the DTO, preventing **mass-assignment / over-posting** attacks where a client sends extra fields (like `isAdmin: true`) hoping they get persisted.

`↳ Follow-up: class-validator vs Zod in Nest?` — `class-validator` (decorator-based on classes) is Nest's native idiom and integrates with DI/Swagger. **Zod** (schema-based) is popular for functional style and type inference; usable via a custom pipe (e.g., `nestjs-zod`). Both are valid; class-validator is the conventional Nest answer.

**75. What are Guards, Interceptors, and Filters — and their order?**
- **Guards** decide if a request may proceed (auth/roles) — run before the handler, return boolean.
- **Interceptors** wrap the handler — transform responses, add logging/timing/caching, can modify the returned stream.
- **Exception filters** catch thrown exceptions and shape the error response.
Execution order: middleware → guards → interceptors (pre) → pipes → **handler** → interceptors (post) → filters (on error).

```ts
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(ctx: ExecutionContext): boolean {
    const req = ctx.switchToHttp().getRequest();
    return Boolean(req.headers.authorization); // simplistic
  }
}
@UseGuards(AuthGuard)
@Get('profile')
getProfile() { /* ... */ }
```

`↳ Follow-up: Guard vs Middleware — when to use which?` — Middleware is Express-level, runs before routing, no access to the execution context (which handler/class is being called). Guards are Nest-aware, run after routing, and can read **metadata** (e.g., `@Roles('admin')` via the `Reflector`) — making them the right tool for role-based authorization.

**76. What are Modules and why does the dependency graph matter?**
A `@Module` groups related controllers/providers and declares what it `imports` (other modules), `provides`, and `exports` (makes available to importers). Providers are only injectable where they're available in scope. This explicit graph enforces boundaries and prevents tangled coupling. The `AppModule` is the root.

`↳ Follow-up: Why might you get "Nest can't resolve dependencies" error?` — The provider isn't in the current module's scope: you forgot to add it to `providers`, or to `exports` it from its module and `imports` that module where it's needed. It's the most common Nest error.

**77. How does Nest handle configuration and async providers?**
`@nestjs/config` loads/validates env config (often with Joi/Zod) and exposes a `ConfigService`. Async providers (`useFactory`) let you create providers that depend on async setup (DB connections) and inject other providers. This keeps config typed, centralized, and testable.

---

<a name="p10"></a>
## PHASE 10 — FASTIFY, KOA & FRAMEWORK COMPARISON

**78. What is Fastify and why might you choose it over Express?**
Fastify is a high-performance, low-overhead web framework focused on **speed** and **developer experience**. Key advantages: significantly faster than Express (efficient routing + a fast JSON serializer), **built-in JSON schema validation and serialization**, a powerful **plugin/encapsulation** system, first-class TypeScript support, and built-in logging (pino). You'd choose it for performance-sensitive services or when you want schema-driven validation/serialization out of the box.

```ts
import Fastify from 'fastify';
const app = Fastify({ logger: true });

app.post('/users', {
  schema: {
    body: {
      type: 'object',
      required: ['email'],
      properties: { email: { type: 'string', format: 'email' } },
    },
  },
}, async (request, reply) => {
  return { id: '1' };   // returning a value sends it; no res.json needed
});
await app.listen({ port: 3000 });
```

`↳ Follow-up: How does Fastify's schema improve performance?` — Beyond validation, Fastify **compiles the response schema into a fast serializer** (via fast-json-stringify), which is much quicker than generic `JSON.stringify`. Schemas do double duty: validate input *and* speed up output.

`↳ Follow-up: What is Fastify's encapsulation model?` — Plugins create **encapsulated contexts** — decorators, hooks, and plugins registered inside one plugin don't leak to siblings unless explicitly shared (`fastify-plugin`). This enforces isolation and prevents the global-middleware sprawl Express is prone to.

**79. What is Koa and how does it differ from Express?**
Koa (by the Express team) is a minimal framework using **async/await middleware** with a **context (`ctx`) object** and an "onion model" — middleware wraps downstream middleware via `await next()`, so code runs on the way down *and* back up. No built-in routing/body parsing (you add modules). It's leaner and more modern in flow control than Express, but smaller ecosystem.
```ts
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();                       // downstream runs here
  ctx.set('X-Response-Time', `${Date.now() - start}ms`); // runs on the way back up
});
```

`↳ Follow-up: What is the "onion model"?` — Each middleware can execute logic **before** and **after** `await next()`, so the request travels "down" through middleware and the response travels "back up" — like layers of an onion. Great for timing, wrapping, and cleanup, which is awkward in Express's linear model.

**80. Express vs Fastify vs NestJS vs Koa — how do you choose?**
- **Express:** mature, huge ecosystem, simple, unopinionated. Default for small/medium apps and when team familiarity matters. Slowest of the four.
- **Fastify:** performance + schema validation + plugins. Choose for speed-sensitive or schema-driven APIs.
- **NestJS:** opinionated structure, DI, TS-first; great for large teams/enterprise apps (runs on Express or Fastify under the hood).
- **Koa:** minimal, modern async middleware; choose when you want a tiny core and full control.

`↳ Follow-up: Can NestJS use Fastify?` — **Yes.** Nest has a Fastify adapter (`@nestjs/platform-fastify`) — you get Nest's architecture with Fastify's performance. Default is Express; switching is mostly a one-line adapter change (with some platform-specific API differences).

**81. What is Hapi, and what's its niche?**
Hapi is a configuration-centric framework emphasizing built-in features (validation via Joi, auth, caching) with minimal external dependencies and a strong security posture. Historically favored where a batteries-included, config-driven approach and enterprise support were priorities. Less common in new projects than Express/Fastify/Nest, but you should recognize it.

---

<a name="p11"></a>
## PHASE 11 — REST API DESIGN

**82. What are the core principles of REST?**
REST (Representational State Transfer) is an architectural style: **resources** identified by URLs, manipulated via standard **HTTP methods**, **stateless** communication (each request carries all needed context — no server-side session dependency), a **uniform interface**, representations (usually JSON), and ideally HATEOAS (hypermedia links). Statelessness is what makes REST horizontally scalable.

**83. What do the HTTP methods mean and which are idempotent/safe?**
- **GET** — read; safe (no side effects) and idempotent.
- **POST** — create/non-idempotent action; **not** idempotent (repeating creates duplicates).
- **PUT** — replace the full resource; idempotent.
- **PATCH** — partial update; not guaranteed idempotent.
- **DELETE** — remove; idempotent.
**Safe** = no state change (GET, HEAD). **Idempotent** = repeating has the same effect as once (GET, PUT, DELETE, HEAD).

`↳ Follow-up: Why does idempotency matter in practice?` — Clients/proxies may **retry** failed requests (network blips). Idempotent methods can be safely retried without side effects. For non-idempotent POSTs (payments!), use an **idempotency key** so retries don't double-charge.

`↳ Follow-up: PUT vs PATCH?` — PUT **replaces** the entire resource (omitted fields are cleared); PATCH applies a **partial** update (only sent fields change). Send a full representation with PUT, a diff with PATCH.

**84. What do the main HTTP status codes mean?**
- **2xx:** 200 OK, 201 Created (+ `Location` header), 202 Accepted (async), 204 No Content.
- **3xx:** 301/302 redirects, 304 Not Modified (caching).
- **4xx (client error):** 400 Bad Request, 401 Unauthorized (not authenticated), 403 Forbidden (authenticated but not allowed), 404 Not Found, 409 Conflict, 422 Unprocessable Entity (validation), 429 Too Many Requests (rate limit).
- **5xx (server error):** 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout.

`↳ Follow-up: 401 vs 403 — what's the difference?` — **401** = "I don't know who you are" (missing/invalid credentials — authenticate). **403** = "I know who you are, but you're not allowed" (authenticated but lacking permission). Mixing these up is a classic mistake.

`↳ Follow-up: 400 vs 422?` — 400 = malformed request (bad JSON, missing required structure). 422 = syntactically valid but **semantically** invalid (failed business/validation rules). Many APIs use 400 for both; 422 is more precise for validation failures.

**85. How do you version an API?**
Common strategies: **URL path** (`/v1/users` — most explicit and common), **header** (`Accept: application/vnd.api.v1+json` — cleaner URLs, harder to test), or **query param** (`?version=1`). Path versioning is the pragmatic default. Version to avoid breaking existing clients when you make incompatible changes.

**86. How do you design pagination?**
- **Offset/limit** (`?page=2&limit=20` or `?offset=40&limit=20`) — simple, supports jumping to pages, but slow on large offsets and can skip/duplicate items if data changes.
- **Cursor/keyset** (`?after=<cursor>`) — uses a stable sort key; efficient and consistent for large/changing datasets, but no random page access.
Return metadata (total, next cursor) alongside data.

`↳ Follow-up: Why is offset pagination slow at high offsets?` — The DB must **scan and discard** all rows before the offset (`OFFSET 100000` reads 100k rows to skip them). Cursor pagination uses an indexed `WHERE id > cursor`, reading only what it returns — constant time regardless of depth.

**87. How should you handle errors in a REST API?**
Return appropriate status codes plus a **consistent error body** (e.g., `{ error: { code, message, details } }`), never leak stack traces/internal details in production, validate input and return 400/422 with field-level messages, and log the full error server-side with a correlation/request ID the client can reference.

`↳ Follow-up: What is a correlation/request ID?` — A unique ID generated per request (or propagated from the client/gateway) attached to logs and returned to the client. It lets you trace a single request across services and logs when debugging — essential in distributed systems.

---

<a name="p12"></a>
## PHASE 12 — DATABASES, ORMs & DATA ACCESS

**88. SQL vs NoSQL — when do you choose which?**
**SQL** (Postgres, MySQL): structured schema, ACID transactions, relations/joins, strong consistency — choose for relational data, complex queries, and financial/transactional integrity. **NoSQL** (MongoDB, DynamoDB, Redis): flexible/schemaless documents or key-value, horizontal scaling, high write throughput — choose for unstructured/evolving data, massive scale, or caching. Many systems use both (Postgres for core data, Redis for cache/sessions).

`↳ Follow-up: Does MongoDB support transactions?` — **Yes**, multi-document ACID transactions since 4.0 (within a replica set/sharded cluster). The old "NoSQL has no transactions" claim is outdated — but transactions are still cheaper/simpler in relational DBs.

**89. What is connection pooling and why is it essential?**
Opening a DB connection is expensive (TCP + auth handshake). A **pool** maintains a set of reusable open connections; requests borrow one, use it, and return it. This avoids per-request connection overhead and caps total connections (protecting the DB from exhaustion). Drivers/ORMs (`pg.Pool`, Prisma, TypeORM) pool by default — you tune min/max size.

`↳ Follow-up: What happens if pool size is too small? Too large?` — **Too small:** requests queue waiting for a free connection → latency, timeouts under load. **Too large:** you exhaust the DB's `max_connections` and starve other clients, or overwhelm the DB. Size it relative to DB limits and concurrency (and remember each clustered process/container has its own pool).

`↳ Follow-up: Why is pooling tricky with serverless?` — Serverless functions scale to many concurrent instances, each with its own pool → connection explosion at the DB. Mitigate with an external pooler (PgBouncer, Prisma Accelerate, RDS Proxy) or serverless-aware drivers.

**90. What is an ORM and what are the trade-offs?**
An ORM (Object-Relational Mapper) maps DB tables to objects/classes, letting you query with code instead of raw SQL, with migrations, relations, and type safety. **Pros:** productivity, type safety, less boilerplate, DB portability. **Cons:** abstraction can hide inefficient queries, harder to express complex/optimized SQL, performance overhead, and the **N+1 problem**. Seniors use ORMs but drop to raw SQL when needed.

**91. Compare Prisma, TypeORM, and Mongoose.**
- **Prisma:** schema-first (own DSL), generates a fully **type-safe** client, excellent DX and migrations; the modern favorite for TS + SQL. Less flexible for very dynamic queries.
- **TypeORM:** decorator/entity-based (Active Record or Data Mapper), mature, integrates tightly with NestJS; types are good but historically leakier than Prisma.
- **Mongoose:** the standard ODM for MongoDB — schemas, validation, middleware/hooks for documents.

```ts
// Prisma — type-safe query
const user = await prisma.user.findUnique({
  where: { id },
  include: { orders: true },     // typed relation
});
```

`↳ Follow-up: Prisma vs raw SQL — when drop to SQL?` — For complex aggregations, window functions, recursive CTEs, or heavily optimized queries the ORM generates poorly. Prisma supports `$queryRaw` (typed, parameterized) for these. Use the ORM for 90% and raw SQL for the hot/complex 10%.

**92. What is the N+1 query problem and how do you fix it?**
N+1 happens when you fetch a list (1 query) then loop and fetch each item's relation separately (N queries) — e.g., 1 query for 100 posts, then 100 queries for each post's author. Fix with **eager loading / joins** (`include`/`join`), **`IN` queries** (fetch all authors in one query), or **DataLoader** (batches + caches per request, common in GraphQL).

```ts
// N+1 (bad)
const posts = await prisma.post.findMany();
for (const p of posts) p.author = await prisma.user.findUnique({ where: { id: p.authorId } });
// Fixed — single query with join
const posts2 = await prisma.post.findMany({ include: { author: true } });
```

`↳ Follow-up: What is DataLoader?` — A utility that **batches** individual load calls made in the same tick into one query and **caches** results per request — turning N lookups into 1. Heavily used in GraphQL resolvers to defeat N+1.

**93. What is a database transaction and what are ACID properties?**
A transaction groups operations so they **all succeed or all fail** (atomicity). ACID: **Atomicity** (all-or-nothing), **Consistency** (valid state to valid state), **Isolation** (concurrent transactions don't interfere), **Durability** (committed data survives crashes). Use transactions for multi-step operations that must stay consistent (transfer money: debit + credit must both happen).
```ts
await prisma.$transaction(async (tx) => {
  await tx.account.update({ where: { id: from }, data: { balance: { decrement: 100 } } });
  await tx.account.update({ where: { id: to },   data: { balance: { increment: 100 } } });
}); // both or neither
```

`↳ Follow-up: What are isolation levels?` — Read Uncommitted, Read Committed (default in Postgres), Repeatable Read, Serializable — increasing isolation prevents anomalies (dirty reads, non-repeatable reads, phantom reads) at the cost of concurrency/performance. Higher isolation = more locking/contention.

**94. What are database migrations and why version them?**
Migrations are versioned, ordered scripts that evolve the DB schema (create tables, add columns) in a repeatable, reviewable way across environments. Tools: Prisma Migrate, TypeORM migrations, Knex. They keep schema changes in source control, enable rollback, and ensure dev/staging/prod stay in sync. Never hand-edit production schemas.

`↳ Follow-up: How do you do a zero-downtime schema change?` — Use **expand/contract**: add the new column/table (backward compatible), deploy code that writes to both, backfill, switch reads, then remove the old column in a later release. Avoid blocking migrations (e.g., adding a NOT NULL column with a default on a huge table) that lock the table.

**95. How do you prevent SQL injection?**
**Never** build queries by string concatenation with user input. Use **parameterized queries / prepared statements** (placeholders bind values safely) — which ORMs and query builders do by default. The DB treats parameters as data, never as executable SQL.
```ts
// SAFE — parameterized
await pool.query('SELECT * FROM users WHERE email = $1', [email]);
// DANGEROUS — never do this
await pool.query(`SELECT * FROM users WHERE email = '${email}'`);
```

`↳ Follow-up: Are ORMs immune to SQL injection?` — Mostly, for normal usage (they parameterize). But **raw query escape hatches** (`$queryRawUnsafe`, string-built `WHERE` clauses) reintroduce the risk. Always parameterize even in raw queries.

---

<a name="p13"></a>
## PHASE 13 — AUTHENTICATION & SECURITY

**96. Authentication vs Authorization?**
**Authentication** = verifying **who** you are (login, credentials, tokens). **Authorization** = determining **what** you're allowed to do (roles, permissions). AuthN comes first, then AuthZ. (401 = failed authentication; 403 = failed authorization.)

**97. How do you store passwords securely?**
**Hash** with a slow, salted algorithm — **bcrypt** (most common), **argon2** (modern, memory-hard), or **scrypt**. Never store plaintext, never use fast hashes (MD5/SHA). Salt is built into bcrypt/argon2 (unique per password, defeats rainbow tables). Use an appropriate cost/work factor.
```ts
import bcrypt from 'bcrypt';
const hash = await bcrypt.hash(plainPassword, 12);     // 12 = cost factor
const ok = await bcrypt.compare(plainPassword, hash);
```

`↳ Follow-up: What does the bcrypt cost factor control?` — The number of hashing rounds (2^cost) — higher = slower = more brute-force-resistant, but more CPU per login. Tune so a hash takes ~100–300ms; raise it over time as hardware improves.

`↳ Follow-up: Why salt if bcrypt is already slow?` — Salt ensures **identical passwords produce different hashes**, so attackers can't precompute rainbow tables or tell when two users share a password. bcrypt embeds a random salt automatically.

**98. JWT vs session-based authentication — trade-offs?**
- **Sessions:** server stores session state (in Redis/DB), client holds a session ID cookie. Easy to **revoke** (delete server-side), but requires server-side storage and is stateful (harder to scale without shared store).
- **JWT:** self-contained signed token holding claims; **stateless** (server verifies signature, no lookup), scales easily. But **hard to revoke** before expiry, and larger per-request. Use short-lived access tokens + refresh tokens to mitigate.

```ts
import jwt from 'jsonwebtoken';
const token = jwt.sign({ sub: user.id, role: user.role }, process.env.JWT_SECRET!, { expiresIn: '15m' });
const payload = jwt.verify(token, process.env.JWT_SECRET!);
```

`↳ Follow-up: How do you revoke a JWT before it expires?` — You can't truly revoke a stateless token, so you add state: a **blocklist/denylist** (store revoked token IDs until expiry), keep access tokens **short-lived** with revocable **refresh tokens**, or bump a per-user token version. Each reintroduces some statefulness.

`↳ Follow-up: Where should you store a JWT on the client?` — Prefer an **httpOnly, Secure, SameSite cookie** (not readable by JS → safer against XSS token theft; needs CSRF protection). `localStorage` is convenient but exposed to any XSS. There's no perfect option — minimize XSS and choose based on threat model.

`↳ Follow-up: What's the difference between an access token and a refresh token?` — **Access token:** short-lived (minutes), sent on every request, grants access. **Refresh token:** long-lived (days), stored securely, used only to obtain new access tokens. Limits the window if an access token leaks, while avoiding frequent re-logins.

**99. What is OAuth 2.0 / OpenID Connect?**
**OAuth 2.0** is an **authorization** framework letting a user grant a third-party app limited access to their resources without sharing credentials (the "Login with Google" delegation). **OpenID Connect (OIDC)** is an **authentication** layer on top of OAuth 2.0 that adds identity (an ID token) — proving who the user is. Use OIDC for "log in with X"; OAuth for delegated API access.

**100. What are the top web security risks (OWASP) for a Node API and mitigations?**
- **Injection** (SQL/NoSQL/command): parameterize queries, validate input, avoid shell with user input.
- **Broken auth:** strong hashing, secure sessions/tokens, rate-limit logins, MFA.
- **XSS:** escape/encode output, sanitize HTML, set CSP (relevant if rendering HTML).
- **Broken access control:** enforce authz on every endpoint/resource, never trust client-supplied IDs without ownership checks.
- **Security misconfiguration:** `helmet`, disable verbose errors, least privilege.
- **Sensitive data exposure:** TLS everywhere, don't log secrets, encrypt at rest.
- **SSRF, insecure deserialization, vulnerable dependencies** (`npm audit`).

`↳ Follow-up: What does helmet do?` — `helmet` sets security-related HTTP **headers** (Content-Security-Policy, X-Frame-Options/frame-ancestors, Strict-Transport-Security, X-Content-Type-Options, etc.) to harden the app against common attacks (clickjacking, MIME sniffing, protocol downgrade). One line of defense-in-depth.

`↳ Follow-up: What is mass assignment and how do you prevent it?` — When a client sends extra fields (`isAdmin: true`) that get blindly saved because you spread `req.body` into your model. Prevent by **whitelisting** allowed fields (DTO + `whitelist: true` in Nest, or explicit field picking) — never trust the whole body.

**101. How do you implement rate limiting?**
Limit requests per client (IP/user/API key) per time window to prevent abuse/DoS and brute force. Use middleware (`express-rate-limit`) backed by a **shared store (Redis)** so the limit is enforced across all instances/processes. Algorithms: fixed window, sliding window, token bucket.

`↳ Follow-up: Why must the rate-limit store be shared (Redis) in a clustered/multi-instance setup?` — In-memory counters are **per process** — with N instances, a client effectively gets N× the limit, and counts reset on restart. A shared Redis store enforces one global limit across all instances.

**102. What is CSRF and when do you need protection?**
Cross-Site Request Forgery tricks a logged-in user's browser into making unwanted authenticated requests using their **cookies**. You need CSRF protection when auth is **cookie-based** (browser auto-sends cookies). Mitigations: `SameSite` cookies, CSRF tokens (synchronizer/double-submit), checking Origin/Referer. **Token-in-header** auth (Authorization: Bearer) isn't vulnerable to classic CSRF because the browser doesn't auto-attach it.

`↳ Follow-up: Why are Bearer-token APIs generally not CSRF-vulnerable?` — CSRF relies on the browser **automatically** sending credentials (cookies). A token sent via a custom `Authorization` header must be added by JS deliberately — a malicious cross-site form can't set it — so classic CSRF doesn't apply (though XSS that steals the token is still a concern).

---

<a name="p14"></a>
## PHASE 14 — ERROR HANDLING, LOGGING & OBSERVABILITY

**103. How should you structure error handling in a Node backend?**
Use a **custom error class hierarchy** (operational errors with status codes), a **centralized error handler** (Express error middleware / Nest exception filter) that maps errors to responses, distinguish **operational** errors (expected: validation, not found) from **programmer** errors (bugs), and never leak internals to clients. Log full details server-side with context.
```ts
class AppError extends Error {
  constructor(message: string, public readonly statusCode = 500, public readonly isOperational = true) {
    super(message);
    Error.captureStackTrace(this, this.constructor);
  }
}
class NotFoundError extends AppError {
  constructor(resource: string) { super(`${resource} not found`, 404); }
}
```

`↳ Follow-up: Operational vs programmer errors — why distinguish?` — **Operational** errors are expected runtime conditions (bad input, network failure) you handle gracefully and respond to. **Programmer** errors are bugs (undefined access, wrong types) — you generally let them crash the process (so a process manager restarts it clean) rather than swallow them, because the process state is now untrustworthy.

`↳ Follow-up: Should you catch every error and keep the process alive?` — No. Swallowing unexpected errors (especially in `uncaughtException`) leaves the process in an unknown/corrupted state. Best practice: log, then **let it crash** and restart (PM2/Kubernetes). `uncaughtException`/`unhandledRejection` handlers should log and exit, not resume.

**104. What's the difference between `uncaughtException` and `unhandledRejection`?**
`uncaughtException` fires for synchronous errors that bubble to the top with no `try/catch`. `unhandledRejection` fires for **promise** rejections with no `.catch`. Both signal a likely bug; the recommended handling is to log the error (and flush logs), then **exit gracefully** and let your supervisor restart the process — not to continue running.

**105. Why use a structured logger instead of `console.log`?**
Structured loggers (**pino**, **winston**) emit **JSON** logs with levels, timestamps, and contextual metadata — machine-parseable for aggregation/searching (ELK, Datadog), with log levels (debug/info/warn/error), redaction of secrets, and far better performance (pino is extremely fast). `console.log` is synchronous, unstructured, and can even block under load.

```ts
import pino from 'pino';
const logger = pino({ level: process.env.LOG_LEVEL ?? 'info' });
logger.info({ userId, route: req.url }, 'request handled');
logger.error({ err }, 'failed to process payment');
```

`↳ Follow-up: Why can console.log be a performance problem?` — Writing to stdout can be **synchronous/blocking** (especially to a file/pipe), so heavy logging in a hot path blocks the event loop. pino offloads/serializes efficiently and can write asynchronously, avoiding this.

`↳ Follow-up: What should you NEVER log?` — Secrets/PII: passwords, tokens, full credit-card/PAN, API keys, session IDs. Use the logger's **redaction** features to scrub sensitive fields, and be mindful of compliance (GDPR/PCI).

**106. What are the "three pillars of observability"?**
- **Logs:** discrete timestamped events (what happened).
- **Metrics:** numeric time-series (request rate, error rate, latency percentiles, event-loop lag, memory) — aggregate health, alerting (Prometheus/Grafana).
- **Traces:** the path of a single request across services with timing per span (distributed tracing, OpenTelemetry) — find *where* latency/errors occur.
Together they let you understand and debug production systems.

`↳ Follow-up: What Node-specific metrics matter most?` — **Event-loop lag** (the key health signal — rising lag means the loop is blocked), heap/memory usage (leak detection), GC pauses, active handles/requests, and the usual RED metrics (Rate, Errors, Duration). Event-loop lag is the one unique to Node.

**107. How do you implement health checks?**
Expose `/health` (liveness — is the process up?) and `/ready` (readiness — can it serve traffic, i.e., DB/cache reachable?). Orchestrators use liveness to restart hung containers and readiness to decide whether to route traffic. Keep liveness cheap; readiness checks dependencies.

`↳ Follow-up: Liveness vs readiness — why separate?` — **Liveness** failing → restart the container (it's broken). **Readiness** failing → stop sending traffic but don't restart (e.g., DB temporarily down, or still warming up). Conflating them causes needless restart loops during transient dependency outages.

---

<a name="p15"></a>
## PHASE 15 — PERFORMANCE, SCALING & ARCHITECTURE

**108. How do you scale a Node.js application?**
- **Vertical:** bigger machine (limited, single point of failure).
- **Horizontal:** multiple instances behind a **load balancer** (the primary strategy) — enabled by statelessness.
- **Multi-core on one box:** `cluster` or PM2 to use all cores.
- **Offload:** caching (Redis), CDN for static assets, queues for async work, read replicas for DB.
- **Decompose:** microservices for independent scaling of hot paths.
Statelessness (externalized sessions/state) is the prerequisite for horizontal scaling.

`↳ Follow-up: What must be true for horizontal scaling to work?` — The app must be **stateless** — no in-memory sessions, caches, or sticky local state that other instances can't see. Shared state lives in Redis/DB. Otherwise requests routed to different instances behave inconsistently.

**109. What caching strategies do you use?**
- **In-memory** (per process) — fastest, but not shared across instances; good for small, hot, non-critical data.
- **Distributed cache (Redis/Memcached)** — shared across instances; sessions, computed results, rate-limit counters.
- **HTTP caching** — `Cache-Control`/`ETag` for client/CDN caching.
- **CDN** — cache static/edge content close to users.
Patterns: cache-aside (lazy), write-through, write-behind. Always set TTLs and plan invalidation.

`↳ Follow-up: What is the cache-aside pattern?` — On read: check cache; on miss, read DB, populate cache, return. On write: update DB and **invalidate/update** the cache. It's the most common pattern. The hard part is **invalidation** — stale cache is a frequent bug source.

`↳ Follow-up: What is a cache stampede / thundering herd, and how do you prevent it?` — When a popular cache key expires and **many** concurrent requests all miss and hit the DB simultaneously. Prevent with locking/single-flight (only one request recomputes while others wait), staggered/jittered TTLs, or serving stale-while-revalidate.

**110. When and why use a message queue?**
A message queue (RabbitMQ, Redis/BullMQ, Kafka, SQS) **decouples** producers from consumers and enables **asynchronous** processing: offload slow work (emails, image processing, report generation) from the request path, smooth load spikes (buffering), retry failed jobs, and build event-driven architectures. The request returns fast (202 Accepted) while work happens in the background.
```ts
// BullMQ producer
await emailQueue.add('welcome', { userId });   // returns immediately
// Worker (separate process) processes it later
```

`↳ Follow-up: What is a dead-letter queue?` — A queue where messages that repeatedly fail processing (after max retries) are sent for inspection instead of being lost or retried forever. It prevents poison messages from blocking the queue and lets you debug failures.

`↳ Follow-up: Difference between a task queue (BullMQ/RabbitMQ) and a log/stream (Kafka)?` — Task queues distribute **discrete jobs** to workers (consumed once, then gone). Kafka is a **durable, ordered, replayable log** — multiple consumers read independently, events are retained, good for event sourcing/streaming analytics. Different tools for different shapes of "async."

**111. How do you handle CPU-intensive tasks without blocking?**
Offload them off the main event loop: **worker_threads** (in-process threads for CPU work), a **message queue + separate worker processes** (best for heavy/long jobs — also scales independently), **child processes**, or an external service. Never run heavy synchronous computation in a request handler.

`↳ Follow-up: worker_threads vs offloading to a queue — how to choose?` — `worker_threads` for **latency-sensitive** CPU work needed within the request (still returns a response). A **queue** for **fire-and-forget / long-running** jobs where the client doesn't wait — more scalable and resilient (retries, separate scaling). Choose by whether the caller needs the result synchronously.

**112. What is the difference between a monolith and microservices?**
A **monolith** is one deployable app — simpler to build, deploy, test, and reason about; great until team/scale pressure. **Microservices** split functionality into independently deployable services — enabling independent scaling/deployment and team autonomy, at the cost of distributed-systems complexity (network failures, data consistency, observability, deployment overhead). Start monolith; extract services when justified.

`↳ Follow-up: What new problems do microservices introduce?` — Network latency/failure between services, distributed transactions/eventual consistency, service discovery, distributed tracing, versioning of inter-service contracts, and operational complexity. "Don't distribute until you must" is the seasoned take.

**113. How do you find and fix a memory leak in Node?**
Symptoms: steadily rising heap/RSS that doesn't drop after GC, eventual OOM crash. Find it by taking **heap snapshots** (Chrome DevTools via `--inspect`, `node --heapsnapshot-signal`, or `clinic.js`/`heapdump`) at intervals and **diffing** to see what's growing. Common causes: unbounded caches/maps, forgotten event listeners, closures capturing large objects, global accumulation, timers not cleared.

`↳ Follow-up: Name common Node memory-leak sources.` — Module-level arrays/maps that only grow, `EventEmitter` listeners added without removal (the "max listeners" warning), closures holding references, `setInterval` never cleared, and caching without size limits/TTL. Bound your caches and remove listeners.

`↳ Follow-up: How do you profile CPU usage?` — `node --prof` (V8 profiler) + `--prof-process`, the `--inspect` flag with Chrome DevTools CPU profiler, `clinic flame` (flamegraphs), or `0x`. Flamegraphs show where CPU time concentrates so you optimize the actual hot path.

---

<a name="p16"></a>
## PHASE 16 — TESTING

**114. What are the levels of testing for a backend?**
- **Unit:** a single function/class in isolation, dependencies mocked — fast, many.
- **Integration:** multiple units together (service + real DB, or route + middleware) — verifies they work together.
- **E2E:** the whole app over HTTP against real(ish) dependencies — highest confidence, slowest.
The "testing pyramid" suggests many unit, fewer integration, fewest E2E — though backend code often benefits from a heavier integration layer.

**115. What testing tools are common in Node/TS?**
**Jest** (all-in-one: runner, assertions, mocking, coverage — most popular), **Vitest** (faster, Vite-native, Jest-compatible API), **Mocha + Chai + Sinon** (modular classic), the **built-in `node:test`** runner (no deps), and **Supertest** for HTTP assertions against Express/Nest/Fastify apps. **Testcontainers** spins up real DBs in Docker for integration tests.

```ts
import request from 'supertest';
import { app } from '../src/app';

describe('POST /users', () => {
  it('creates a user', async () => {
    const res = await request(app).post('/users').send({ email: 'a@b.com' });
    expect(res.status).toBe(201);
    expect(res.body).toHaveProperty('id');
  });
});
```

**116. What's the difference between mocks, stubs, spies, and fakes?**
- **Stub:** replaces a function with a canned return value (no real logic).
- **Spy:** wraps a real/fake function to record how it was called (args, call count) while optionally calling through.
- **Mock:** a stub + built-in expectations/assertions about how it should be called.
- **Fake:** a working lightweight implementation (in-memory DB/repo).
You use these to isolate the unit under test from slow/external dependencies.

`↳ Follow-up: Why is over-mocking a problem?` — Tests that mock everything verify your **mocks**, not real behavior — they pass even when the integration is broken, and they break on every refactor. Prefer testing real collaborations (integration tests, in-memory fakes) where practical; mock only true externals (third-party APIs, time, randomness).

**117. How do you test code that depends on a database?**
Options: an **in-memory/fake repository** (fast unit tests of business logic), a **real DB in Docker** via Testcontainers or a test database (integration tests — highest fidelity), or transactional rollback per test (wrap each test in a transaction and roll back). Avoid mocking the ORM deeply — it tests the mock, not real queries.

`↳ Follow-up: How do you keep DB tests isolated and repeatable?` — Reset state between tests (truncate tables, transaction rollback, or fresh container), seed deterministic fixtures, and avoid shared mutable state/ordering dependencies between tests. Each test should pass in isolation and in any order.

**118. How do you test async code and handle flakiness?**
Use `async/await` in tests and **await** assertions; never forget to return/await the promise (or the test passes before async work finishes). Control non-determinism: **fake timers** (`jest.useFakeTimers`) for setTimeout/intervals, **mock the clock** and `Date.now`, seed randomness, and stub network calls. Flaky tests usually stem from real timers, real network, shared state, or unawaited promises.

`↳ Follow-up: How do you test something that uses Date.now() or setTimeout?` — Inject the clock or use fake timers so time is deterministic. `jest.useFakeTimers()` lets you `advanceTimersByTime(ms)` to fast-forward, and mock `Date` to a fixed instant — making time-dependent logic testable and fast.

**119. How do you test a NestJS service in isolation?**
Use `Test.createTestingModule` to build a module with the service and **mocked providers**, leveraging Nest's DI to override dependencies.
```ts
const module = await Test.createTestingModule({
  providers: [
    UsersService,
    { provide: UsersRepository, useValue: { findById: jest.fn() } },
  ],
}).compile();
const service = module.get(UsersService);
```

---

<a name="p17"></a>
## PHASE 17 — DEPLOYMENT, DEVOPS & PRODUCTION

**120. How do you containerize a Node app well (Docker)?**
Use a **multi-stage build**: a build stage installs all deps and compiles TS; the final stage copies only production deps + compiled `dist` onto a slim base image. Run as a **non-root** user, use `npm ci --omit=dev`, leverage layer caching (copy `package*.json` and install before copying source), and add a `.dockerignore`.
```dockerfile
FROM node:20-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=build /app/dist ./dist
USER node
CMD ["node", "dist/index.js"]
```

`↳ Follow-up: Why copy package.json and install before copying the rest of the source?` — **Layer caching.** Dependencies change less often than source. Installing in its own layer means Docker reuses the cached `npm ci` layer on subsequent builds when only source changed — dramatically faster builds.

`↳ Follow-up: Why run as non-root and use a slim/distroless image?` — Security and size. Non-root limits damage if the container is compromised; slim/distroless images have fewer packages → smaller attack surface, smaller image, faster pulls.

**121. What is PM2 and when do you need it?**
PM2 is a production process manager for Node: it keeps the app **alive** (auto-restart on crash), runs **cluster mode** across cores, handles **zero-downtime reloads**, log management, and monitoring. Useful on VMs/bare metal. In **Kubernetes**, you often skip PM2 — the orchestrator handles restarts/scaling, and one-process-per-container is cleaner.

`↳ Follow-up: PM2 cluster mode vs Kubernetes — do you need both?` — Generally **no**. In K8s, let the orchestrator manage replicas/restarts and run a single Node process per container (simpler, better observability, matches container philosophy). PM2 cluster mode shines on a single VM where you want to use all cores without an orchestrator.

**122. How do you manage configuration and secrets across environments?**
Config via **environment variables** (12-factor), validated at startup. Secrets via a **secrets manager** (AWS Secrets Manager, Vault, Doppler, K8s Secrets) — **never** committed to git or baked into images. Different values per environment injected at deploy time. `.env` files only for local development.

`↳ Follow-up: Why not put secrets in the Docker image?` — Image layers are **persistent and inspectable** — anyone with the image can extract baked-in secrets, and they end up in registries/caches. Inject secrets at runtime (env vars from a secrets manager), not build time.

**123. What does a good CI/CD pipeline for a Node service include?**
On each push/MR: install (`npm ci`), **lint** (ESLint), **type-check** (`tsc --noEmit`), **test** (unit + integration with coverage), **build** (compile + Docker image), **security scan** (`npm audit`, image scanning), and on merge to main: **deploy** (to staging then production, ideally with health-check-gated rollout). Fail fast, keep it fast, and require green checks to merge.

`↳ Follow-up: Why type-check separately if you build with esbuild/swc?` — Fast transpilers **strip types without checking them**, so a type error wouldn't fail the build. Run `tsc --noEmit` as a dedicated CI step to actually catch type errors, while using esbuild/swc for the fast emit.

**124. How do you achieve zero-downtime deployments?**
Run multiple instances behind a load balancer and roll out gradually: **rolling/blue-green/canary** deployments, **readiness probes** so traffic only routes to ready instances, **graceful shutdown** (drain in-flight requests on SIGTERM), and backward-compatible DB migrations (expand/contract). The orchestrator keeps old instances serving until new ones are healthy.

---

<a name="p18"></a>
## PHASE 18 — TRICKY / GOTCHA & RAPID-FIRE

**125. What's the output order? (classic event-loop puzzle)**
```ts
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
process.nextTick(() => console.log('4'));
console.log('5');
```
**Order: 1, 5, 4, 3, 2.** Synchronous (`1`, `5`) first; then microtasks — `process.nextTick` (`4`) before the Promise queue (`3`); then the timer macrotask (`2`) on the next loop iteration.

**126. Why does this leak memory?**
```ts
const cache: Record<string, Buffer> = {};
app.get('/file/:name', (req, res) => {
  cache[req.params.name] = readFileSync(req.params.name); // grows forever
  res.send('ok');
});
```
An **unbounded cache** keyed by user input grows without limit until OOM. Fix with a bounded LRU cache (`lru-cache`) with a max size/TTL.

**127. What's wrong with this?**
```ts
const ids = [1, 2, 3];
ids.forEach(async (id) => { await deleteUser(id); });
console.log('done'); // logs BEFORE deletes finish
```
`forEach` **ignores** the returned promises — it doesn't await them, so `'done'` logs immediately and errors are unhandled. Use `for (const id of ids) await deleteUser(id)` (sequential) or `await Promise.all(ids.map(deleteUser))` (concurrent).

**128. Why might `JSON.parse(req.body)` crash the server?**
On malformed JSON it **throws synchronously**. If unguarded, the throw escapes the handler (Express 4) to `uncaughtException` and can crash the process. Wrap in try/catch or rely on `express.json()` (which handles parse errors → 400). Also, parsing huge bodies blocks the event loop — set size limits.

**129. What's the subtle bug?**
```ts
async function getUser(id: string) {
  const user = await db.findUser(id);
  return user;
}
app.get('/users/:id', async (req, res) => {
  const user = await getUser(req.params.id);
  res.json(user); // if user is null → sends null with 200
});
```
A missing user returns `null` with **status 200** instead of 404, and an unhandled DB rejection (Express 4) escapes. Fix: check for null → 404, wrap in try/catch or `catchAsync`.

**130. Why is `Array(1000000).fill(0).map(heavyCompute)` a problem in a handler?**
It runs a **synchronous** CPU-heavy loop, **blocking the event loop** for its entire duration — every other request stalls. Offload to a worker thread/queue, or chunk it with `setImmediate` to yield to the loop.

**131. What does `0.1 + 0.2 === 0.3` return and why?**
`false` — IEEE-754 floating point can't represent `0.1`/`0.2` exactly, so the sum is `0.30000000000000004`. For money, use **integer minor units** (cents) or a decimal library (`decimal.js`); never floats for currency.

**132. Why shouldn't you trust `req.headers` / client input for security decisions?**
All client-supplied data (headers, body, query, cookies) is **spoofable**. Never make authz decisions on a client-set header like `x-user-role`; derive identity/roles from a **verified** token/session server-side. Validate and sanitize everything.

**133. Rapid-fire one-liners:**
- **Node is** single-threaded for JS, multi-threaded under the hood (libuv pool + workers).
- **The event loop** runs phases: timers → pending → poll → check → close; microtasks drain between.
- **`process.nextTick`** beats Promise microtasks; both beat timers/`setImmediate`.
- **`await`** suspends the function, not the event loop.
- **Streams** keep memory flat; respect **backpressure**.
- **`Promise.all`** fails fast; **`allSettled`** never rejects.
- **Pool** your DB connections; **share** rate-limit/session state via Redis.
- **Parameterize** queries; **hash** passwords with bcrypt/argon2.
- **JWT** = stateless but hard to revoke; **sessions** = revocable but stateful.
- **401** = who are you; **403** = you can't; **404** = not here.
- **Validate at runtime** (Zod), don't trust TS types for input.
- **Let it crash** on programmer errors; handle operational ones.
- **`npm ci`** in CI, **multi-stage Docker**, **non-root**, **`--omit=dev`**.
- **Statelessness** is the prerequisite for horizontal scaling.
- **Type-check with `tsc --noEmit`** even when building with esbuild/swc.

---

## FINAL PREP TIPS (FROM THE OTHER SIDE OF THE TABLE)

1. **The event loop is the #1 differentiator.** If you can confidently explain phases, microtasks vs macrotasks, `nextTick` vs `setImmediate`, and what "blocking the loop" means, you'll clear most senior screens. Practice the output-order puzzles out loud.
2. **Always connect a feature to the problem it solves.** Streams → memory; pooling → connection overhead; queues → decoupling/async; JWT → statelessness. Reasoning chains read as experience.
3. **Lead with TypeScript.** Show you validate runtime input (Zod) separately from compile-time types, and that you know `unknown` > `any`. This is a strong seniority signal.
4. **Know the trade-offs, not just the happy path.** JWT vs sessions, monolith vs microservices, ORM vs raw SQL, worker_threads vs queue — naming costs on both sides beats picking a "winner."
5. **Security questions are seniority filters.** Password hashing, SQL injection, mass assignment, 401 vs 403, secrets management — have crisp answers.
6. **Be production-minded.** Graceful shutdown, health checks (liveness vs readiness), structured logging, event-loop-lag metrics, zero-downtime deploys. Junior devs build features; seniors run them in production.
7. **Code the classics by hand:** an Express error-handling middleware + `catchAsync`, a Zod-validated route, a `Promise.all` refactor, a graceful-shutdown block, a multi-stage Dockerfile, a transaction. Muscle memory beats recall under pressure.
8. **For framework questions, know the "why."** Nest's DI (testability), Fastify's schema (validation + serialization speed), Express's middleware order. Don't just list features — explain the design intent.

> **You've got this.** Drill the event-loop puzzles and the trade-off questions, build the "code the classics" snippets until they're automatic, and you'll walk in able to discuss Node like someone who's shipped and operated it — which is exactly what they're listening for.
