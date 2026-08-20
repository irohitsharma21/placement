# TCR 03 — Full-Stack Development

> The broadest TCR topic. You do not need to build anything — you need crisp one-line explanations of **what each piece is and why it exists**. DevOps questions (Jenkins, Docker, Kubernetes) appeared in the official sample, so they get their own file: [04_DEVOPS.md](04_DEVOPS.md).

---

## 1. The web request lifecycle — the mental model everything hangs off

```
Browser types URL
  -> DNS resolves the domain to an IP
  -> TCP connection (3-way handshake) + TLS handshake for HTTPS
  -> HTTP request sent
  -> [ CDN / Load balancer ]
  -> Web server / application server
  -> Business logic -> Cache (Redis) -> Database
  -> HTTP response (HTML / JSON)
  -> Browser parses HTML, builds the DOM, fetches CSS/JS/images, renders
```
Being able to narrate this chain lets you answer "where would you investigate first?" questions — exactly the style the sample TCR screenshot showed.

---

## 2. HTTP essentials

### Methods

| Method | Purpose | Safe? | Idempotent? | Has body? |
|---|---|---|---|---|
| `GET` | retrieve a resource | Yes | Yes | No |
| `POST` | create a resource / submit data | No | **No** | Yes |
| `PUT` | replace a resource entirely | No | **Yes** | Yes |
| `PATCH` | partially update a resource | No | Not required to be | Yes |
| `DELETE` | remove a resource | No | **Yes** | Optional |
| `HEAD` | like GET but headers only | Yes | Yes | No |
| `OPTIONS` | ask which methods are allowed (CORS preflight) | Yes | Yes | No |

- **Safe** = does not modify server state.
- **Idempotent** = performing it N times has the same effect as performing it once.
- **The classic question, why is PUT idempotent but POST not?** *"PUT supplies the complete target state at a known URI, so repeating it leaves the resource identical. POST creates a new subordinate resource each time, so repeating it creates duplicates."*

### Status codes

| Range | Meaning | Ones to know |
|---|---|---|
| **1xx** | Informational | 101 Switching Protocols (WebSocket upgrade) |
| **2xx** | Success | **200** OK, **201** Created, 202 Accepted, **204** No Content |
| **3xx** | Redirection | 301 Moved Permanently, 302 Found, **304** Not Modified (cache hit) |
| **4xx** | **Client** error | **400** Bad Request, **401** Unauthorized, **403** Forbidden, **404** Not Found, 405 Method Not Allowed, 409 Conflict, 422 Unprocessable Entity, 429 Too Many Requests |
| **5xx** | **Server** error | **500** Internal Server Error, 502 Bad Gateway, **503** Service Unavailable, 504 Gateway Timeout |

**401 vs 403 is asked constantly:** *"401 means the request lacks valid authentication credentials — who are you? 403 means the server knows who you are but you are not permitted — you may not do this."*

**GET vs POST:** GET puts parameters in the URL (visible, bookmarkable, cacheable, length-limited); POST puts them in the body (not logged in URLs, no practical size limit, not cached). Never send credentials via GET.

### Other HTTP facts
- **HTTP is stateless** — each request is independent and the server retains nothing between them. Cookies, sessions and tokens are the mechanisms bolted on to simulate state.
- **HTTPS** = HTTP over TLS: encryption, integrity, and server authentication via certificates.
- **HTTP/2** adds multiplexing (many requests on one connection), header compression, and server push. **HTTP/3** runs over QUIC/UDP and removes head-of-line blocking.
- Headers worth knowing: `Content-Type`, `Authorization`, `Accept`, `Cache-Control`, `Set-Cookie`, `Origin`.

---

## 3. REST

**REST (Representational State Transfer)** is an architectural style for APIs over HTTP.

**Its constraints:**
1. **Client-server** — separated concerns.
2. **Stateless** — the server keeps no client session between requests; each request carries everything it needs.
3. **Cacheable** — responses declare whether they may be cached.
4. **Uniform interface** — resources identified by URIs, manipulated with standard methods.
5. **Layered system** — a client cannot tell whether it is talking to the origin server or an intermediary.
6. **Code on demand** (optional).

**Good REST design:**
```
GET    /api/users             list users
GET    /api/users/42          fetch one user
POST   /api/users             create a user
PUT    /api/users/42          replace user 42
PATCH  /api/users/42          partially update user 42
DELETE /api/users/42          delete user 42
GET    /api/users/42/orders   nested resource
```
Rules: **nouns not verbs** (`/getUsers` is wrong), plural collection names, hierarchy through nesting, filtering and pagination via the query string (`?page=2&limit=20&sort=name`), and versioning (`/api/v1/...`).

**Why statelessness matters:** *"Because no session lives on any particular server, any instance can serve any request — which is what makes horizontal scaling and load balancing possible."* That single sentence answers a whole family of questions.

### REST vs GraphQL vs SOAP

| | REST | GraphQL | SOAP |
|---|---|---|---|
| Style | resources + HTTP verbs | one endpoint, a query language | XML protocol with strict contracts |
| Data fetching | fixed response per endpoint | the client requests exactly the fields it needs | fixed, contract-driven |
| Over/under-fetching | common | avoided | common |
| Round trips | often several | usually one | several |
| Caching | easy (HTTP caching) | harder (POST to one URL) | hard |
| Best for | most public and CRUD APIs | complex nested data, mobile clients on slow networks | enterprise systems needing WS-Security and formal contracts |

---

## 4. State, sessions, cookies and tokens

| | Cookie | Session | JWT |
|---|---|---|---|
| Stored where | browser | **server** (memory/Redis/DB), with an id in a cookie | **client** — the token carries the data |
| Server state | none required | required | **none** (stateless) |
| Scales horizontally | yes | needs sticky sessions or shared storage | **yes, easily** |
| Revocation | delete the cookie | delete server-side — **easy** | **hard** — valid until expiry unless you keep a blocklist |
| Size limit | about 4 KB | unlimited server-side | keep it small; it travels on every request |

**JWT structure:** `header.payload.signature`, base64url-encoded and dot-separated. The signature proves integrity.
> **Critical point examiners test:** a JWT payload is **encoded, not encrypted** — anyone can read it. Never put secrets inside. The signature prevents **tampering**, not **reading**.

**Cookie security flags:** `HttpOnly` (JavaScript cannot read it, mitigating XSS theft), `Secure` (HTTPS only), `SameSite` (mitigates CSRF).

**Authentication vs Authorization:** *"Authentication establishes who you are; authorization determines what you are allowed to do. Authentication always comes first."*
**OAuth 2.0** is a delegated **authorization** framework ("let this app read my Google contacts") — not an authentication protocol. **OpenID Connect** is the authentication layer built on top of it.

---

## 5. Web security — the ones that appear in MCQs

| Attack | What it is | Defence |
|---|---|---|
| **SQL Injection** | user input concatenated into a query changes the query meaning | **parameterised queries / prepared statements**, least-privilege DB accounts |
| **XSS** (Cross-Site Scripting) | attacker-supplied script runs in another user's browser | escape and encode output, Content-Security-Policy, `HttpOnly` cookies |
| **CSRF** | a logged-in user's browser is tricked into sending an authenticated request | anti-CSRF tokens, `SameSite` cookies, verify the `Origin` header |
| **Man-in-the-middle** | traffic intercepted in transit | HTTPS/TLS, HSTS, certificate pinning |
| **Broken authentication** | weak passwords, session fixation | hash with **bcrypt/argon2 plus a salt**, MFA, rotate the session id on login |
| **Sensitive data exposure** | secrets in source code or logs | environment variables, secret managers, encryption at rest |

**Never store passwords encrypted or in plain text — store them hashed with a salt, using a deliberately slow algorithm (bcrypt, scrypt, argon2).** Encryption is reversible, which is exactly what you do not want here. This distinction is a frequent exam question.

**CORS (Cross-Origin Resource Sharing):** browsers enforce a **same-origin policy** (same scheme, host and port). CORS is how a server *relaxes* it, via `Access-Control-Allow-Origin` headers. A "CORS error" is the browser blocking a response the **server** never authorised — so the fix belongs on the **server**, not in client code. It protects browsers only; it is not a server-side access control.

---

## 6. Frontend: HTML & CSS

- **Semantic HTML** (`<header>`, `<nav>`, `<main>`, `<article>`, `<footer>`) improves accessibility and SEO compared with generic `<div>`s.
- **The CSS box model:** content -> padding -> border -> margin. `box-sizing: border-box` makes `width` include padding and border, which is why nearly every project sets it globally.
- **Specificity**, highest wins: inline style > id > class/attribute/pseudo-class > element. `!important` overrides everything and should be a last resort.
- **Position:** `static` (default), `relative` (offset from its normal spot), `absolute` (relative to the nearest positioned ancestor), `fixed` (relative to the viewport), `sticky` (relative until a scroll threshold, then fixed).
- **Flexbox** is one-dimensional (a row or a column); **Grid** is two-dimensional. Flexbox for a toolbar, Grid for a page layout.
- **Responsive design:** media queries, relative units (`rem`, `%`, `vw`), `max-width: 100%` on images, mobile-first.
- `display: none` removes the element from layout entirely; `visibility: hidden` hides it but keeps its space.

---

## 7. JavaScript — the highest-yield frontend topic

### var vs let vs const

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | function | block | block |
| Hoisted | yes, initialised as `undefined` | yes, but in the temporal dead zone | same as `let` |
| Redeclarable | yes | no | no |
| Reassignable | yes | yes | **no** |

`const` prevents **rebinding**, not mutation: `const a = [1]; a.push(2);` is legal, `a = [2]` is not. Expect this exact question.

### Hoisting
Declarations move to the top of their scope. `var` is initialised to `undefined`; `let` and `const` exist but are unusable until declared — the **temporal dead zone**, which throws a `ReferenceError`. Function declarations are fully hoisted; function expressions are not.

### Closures
A function retains access to the variables of the scope in which it was created, even after that scope has returned.
```js
function counter() {
  let count = 0;                 // private state
  return function () { return ++count; };
}
const c = counter();
c(); // 1
c(); // 2
```
**Why they matter:** data privacy, function factories, the module pattern.
*"A closure is a function bundled with its lexical environment, which is why count survives after counter() has returned."*

### The event loop — a guaranteed question
JavaScript is **single-threaded** with a **non-blocking** concurrency model. The **call stack** runs synchronous code. Asynchronous work is handed off to Web APIs (or libuv in Node); completed callbacks queue up, and the **event loop** moves them onto the stack once it is empty.
**The microtask queue (promises) is drained completely before the macrotask queue (setTimeout, setInterval, I/O).**
```js
console.log('1');
setTimeout(() => console.log('2'), 0);            // macrotask
Promise.resolve().then(() => console.log('3'));   // microtask
console.log('4');
// Output: 1 4 3 2
```
If you can explain that output, you can answer any event-loop question they ask.

### Promises and async/await
```js
fetch('/api/users')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err))
  .finally(() => console.log('done'));

async function getUsers() {
  try {
    const res = await fetch('/api/users');
    if (!res.ok) throw new Error(res.status);
    return await res.json();
  } catch (err) { console.error(err); }
}
```
A Promise is **pending**, **fulfilled**, or **rejected**. `async/await` is syntactic sugar over promises that lets asynchronous code read sequentially, with errors handled by `try/catch` instead of `.catch()`. `Promise.all` fails fast if any promise rejects; `Promise.allSettled` waits for every outcome; `Promise.race` settles with the first to finish.

### Equality
`==` coerces types before comparing; `===` compares value **and** type with no coercion. **Always use `===`.**
```js
0 == '';             // true  (coerced)
0 === '';            // false
null == undefined;   // true
null === undefined;  // false
NaN === NaN;         // false — use Number.isNaN()
```
Falsy values, worth memorising: `false, 0, -0, 0n, "", null, undefined, NaN`. Everything else is truthy — including `[]` and `{}`.

### this
`this` depends on **how a function is called**, not where it is written.
- Method call `obj.f()` gives `obj`
- Plain call `f()` gives `undefined` in strict mode, otherwise the global object
- `new F()` gives the newly created object
- `call` / `apply` / `bind` give whatever you specify
- **Arrow functions have no `this` of their own** — they inherit it lexically from the enclosing scope, which is why they are the standard fix for callbacks that lose `this`.

### Other essentials
- **Prototypal inheritance:** objects delegate to a prototype object; `class` syntax is sugar over it.
- **Shallow vs deep copy:** the spread operator `{...obj}` copies one level, so nested objects stay shared. Use `structuredClone(obj)` for a deep copy.
- `null` is an intentional empty value; `undefined` means never assigned.
- **Event bubbling** propagates from the target up through its ancestors; **capturing** goes the other way. **Event delegation** attaches one listener to a parent and inspects `event.target` — efficient for long, dynamic lists.
- **Debounce** waits until activity stops before firing (search-as-you-type); **throttle** fires at most once per interval (scroll handlers).

---

## 8. React

### Core concepts
- **Component** — a reusable, self-contained piece of UI. Modern React uses function components.
- **Props** — data passed *in* from a parent. **Read-only** inside the child.
- **State** — data owned and mutated by the component itself; changing it triggers a re-render.

> **Props vs state, the most common React question:** *"Props are passed down from a parent and are immutable within the receiving component; state is internal, owned by the component, and updating it triggers a re-render. Props enable reuse, state enables interactivity."*

- **Virtual DOM** — an in-memory representation of the UI. On a state change React builds a new tree, **diffs** it against the previous one (reconciliation), and applies only the minimal set of real DOM mutations. Direct DOM manipulation is expensive, so minimising and batching those mutations is the performance win.
- **Keys** — a stable identity for each list item, so React can tell which items moved, changed or were removed. **Using the array index as a key is an anti-pattern** when the list can be reordered or filtered: React then reuses the wrong DOM nodes and component state attaches to the wrong row.
- **JSX** — HTML-like syntax compiled into `React.createElement()` calls.
- **One-way data flow** — data flows from parent to child. To share state between siblings, **lift it up** to the nearest common ancestor.

### Hooks
```jsx
const [count, setCount] = useState(0);        // local state

useEffect(() => {                              // side effects
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);              // CLEANUP: runs on unmount / before re-run
}, [dependency]);                              // dependency array

const value = useMemo(() => expensive(a), [a]);      // memoise a VALUE
const fn    = useCallback(() => doThing(a), [a]);    // memoise a FUNCTION
const ref   = useRef(null);                           // a mutable box that survives renders
const theme = useContext(ThemeContext);               // consume context, avoiding prop drilling
```
**The `useEffect` dependency array decides when it re-runs:**
- omitted -> after **every** render
- `[]` -> **once**, after the first render (mount)
- `[a, b]` -> whenever `a` or `b` changes

**Rules of hooks:** call them only at the top level — never inside conditions, loops or nested functions — and only from React function components or custom hooks. React tracks hooks **by call order**, so a conditional hook desynchronises that order and corrupts state.

**Controlled vs uncontrolled inputs:** a controlled input takes its value from React state and updates through `onChange`, making React the single source of truth. An uncontrolled input keeps its value in the DOM and is read with a ref. Controlled is the default recommendation because validation and conditional logic become straightforward.

**State management:** `useState` for local state; Context for low-frequency global values (theme, current user); Redux or Zustand for large, frequently-updated shared state. Redux's parts: **store** (single source of truth), **action** (a plain object describing what happened), **reducer** (a pure function `(state, action) => newState`).

**Why React state updates look asynchronous:** React batches updates within an event handler for performance, so reading the state variable immediately after `setCount` still yields the old value. Use the functional form `setCount(c => c + 1)` whenever the new value depends on the previous one.

---

## 9. Node.js and Express

**Node.js** is a JavaScript runtime built on the V8 engine that runs JavaScript outside the browser. It is **single-threaded, event-driven and non-blocking**, which makes it excellent for I/O-bound work (APIs, real-time apps) and poor for CPU-bound work — a heavy computation blocks the event loop and stalls every other request. The escape hatches are worker threads, clustering, or offloading to another service.

**npm** is the package manager. `package.json` declares dependencies and scripts; `package-lock.json` pins exact resolved versions so installs are reproducible. `dependencies` ship to production; `devDependencies` (test runners, bundlers) do not.

### Express and middleware
```js
const express = require('express');
const app = express();

app.use(express.json());                       // built-in body parser

app.use((req, res, next) => {                  // custom middleware
  console.log(req.method, req.url);
  next();                                       // pass control onward
});

app.get('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) return res.status(404).json({ error: 'Not found' });
  res.json(user);
});

app.use((err, req, res, next) => {             // error handler: FOUR parameters
  console.error(err);
  res.status(500).json({ error: 'Internal Server Error' });
});

app.listen(3000);
```
**Middleware** is a function with access to `req`, `res` and `next`, executed **in the order it is registered**. It can run code, modify the request/response, end the cycle, or pass control on. Typical uses: logging, body parsing, authentication, CORS, error handling.
**Two things they test:** forgetting to call `next()` leaves the request hanging forever, and an **error-handling middleware is identified by having four parameters** `(err, req, res, next)` and must be registered **last**.

**Request data lives in three places:** `req.params` (route parameters, `/users/:id`), `req.query` (query string, `?page=2`), and `req.body` (the request body, needing a parser).

---

## 10. Databases in an application context

**SQL vs NoSQL**

| | SQL (MySQL, PostgreSQL) | NoSQL (MongoDB, Redis, Cassandra) |
|---|---|---|
| Schema | fixed, defined up front | flexible / schema-less |
| Structure | tables, rows, relations | documents, key-value, wide-column, graph |
| Scaling | vertical (bigger machine); sharding is hard | **horizontal** (add nodes) |
| Transactions | strong **ACID** | often **BASE** / eventual consistency |
| Joins | native and efficient | limited; data is usually denormalised or embedded |
| Best for | financial and transactional data, complex relationships | high write volume, evolving schemas, caching, catalogues |

**CAP theorem:** a distributed system can guarantee at most two of **Consistency**, **Availability** and **Partition tolerance**. Since network partitions are unavoidable in practice, the real choice is **CP** (refuse requests to stay consistent — MongoDB, HBase) versus **AP** (stay available and reconcile later — Cassandra, DynamoDB).

**ORM/ODM** (Sequelize, TypeORM, Mongoose, Hibernate) maps objects to database rows or documents. It gives productivity, portability and safety from injection, at the cost of some control and the risk of inefficient generated queries — most notoriously the **N+1 query problem**, where fetching a list then lazily loading each item's relation issues one query per row. The fix is eager loading or a join.

**Connection pooling** reuses a fixed set of open database connections instead of opening one per request, because the TCP and authentication handshake is expensive relative to the query itself.

---

## 11. Architecture & scaling

### Monolith vs Microservices

| | Monolith | Microservices |
|---|---|---|
| Deployment | one unit | many independent services |
| Scaling | scale the whole application | scale only the busy service |
| Technology choice | one stack | per-service freedom |
| Failure blast radius | one bug can down everything | isolated, if boundaries are right |
| Complexity | simpler to build and debug | distributed tracing, networking, data consistency |
| Best for | small teams, early products | large teams, independently-evolving domains |

**The honest answer to "which is better?" is "it depends on team size and domain complexity"** — start monolithic and extract services when a specific boundary demonstrably needs independent scaling or deployment. That nuance scores better than picking a side.

### SPA vs MPA, CSR vs SSR

| | Client-Side Rendering (SPA) | Server-Side Rendering |
|---|---|---|
| First paint | slower (download and run the JS bundle) | **faster** (HTML arrives ready) |
| Later navigation | **fast** (no full reload) | slower (a round trip per page) |
| SEO | historically weaker | **better** — crawlers get real HTML |
| Server load | lower | higher |
| Examples | React, Angular, Vue SPAs | Next.js SSR, traditional PHP/JSP |

**Static Site Generation (SSG)** pre-renders pages at build time — the fastest option for content that rarely changes. Frameworks like Next.js mix all three per route.

### Scaling & performance vocabulary
- **Vertical scaling** = a bigger machine; simple but capped and a single point of failure. **Horizontal scaling** = more machines; needs statelessness and a load balancer.
- **Load balancer** distributes traffic across servers (round-robin, least-connections, IP-hash) and performs health checks so dead instances stop receiving traffic.
- **CDN** caches static assets at edge locations near users, cutting latency and origin load.
- **Caching layers:** browser cache -> CDN -> application cache (Redis/Memcached) -> database query cache. Eviction policies: **LRU**, LFU, FIFO, TTL. The hard part is **invalidation** — serving stale data is the usual failure mode.
- **Message queue** (RabbitMQ, Kafka, SQS) decouples producers from consumers, absorbs traffic spikes, and enables asynchronous work such as sending email or generating reports.
- **Rate limiting** protects an API from abuse; the usual response code is **429 Too Many Requests**.
- **WebSockets** provide a persistent, full-duplex connection for real-time features (chat, live dashboards). Compare with **polling** (repeated requests, wasteful), **long polling**, and **Server-Sent Events** (one-way server-to-client).
- **Horizontal scaling requires stateless servers** — which is precisely why REST's statelessness constraint matters.

### The MERN stack (and friends)
**MongoDB** (database) + **Express** (backend framework) + **React** (frontend) + **Node.js** (runtime). MEAN swaps React for Angular; MEVN uses Vue. The attraction is one language, JavaScript, across the whole stack.

### Environments
`development` -> `testing/staging` -> `production`. Configuration differs per environment through environment variables, never hard-coded — a `.env` file is git-ignored and secrets live in a secret manager.

---

## 12. Testing

| Level | What it tests | Tools |
|---|---|---|
| **Unit** | one function or component in isolation | Jest, Mocha, Google Test |
| **Integration** | several units working together, e.g. an API route hitting a database | Supertest, Jest |
| **End-to-end (E2E)** | a real user journey through the whole system | Cypress, Playwright, Selenium |

**The testing pyramid:** many fast unit tests, fewer integration tests, very few slow E2E tests. Inverting it (mostly E2E) gives a slow, flaky suite that nobody trusts.
**TDD** = write the failing test first, make it pass, then refactor. **Mocking** replaces a real dependency (a database, a payment API) with a controllable stand-in so the test stays fast and deterministic.

---

## 13. Reasoning phrases you can reuse verbatim

- *"HTTP is stateless, so each request must carry its own context; sessions and tokens are the mechanisms used to simulate continuity."*
- *"401 indicates missing or invalid authentication, whereas 403 means the identity is known but lacks permission."*
- *"REST requires statelessness so that any server instance can handle any request, which is what enables horizontal scaling behind a load balancer."*
- *"The virtual DOM lets React compute the minimal set of real DOM mutations, because direct DOM manipulation is the expensive operation."*
- *"Keys give React a stable identity per list item; using the index breaks that identity when the list is reordered, so state attaches to the wrong element."*
- *"Node is single-threaded and non-blocking, which suits I/O-bound workloads but means a CPU-heavy task blocks the event loop and stalls all other requests."*
- *"A JWT is signed, not encrypted — the signature guarantees integrity but the payload remains readable, so it must never carry secrets."*
- *"Middleware executes in registration order and must call next() to pass control on; an error handler is distinguished by taking four parameters and is registered last."*
- *"Parameterised queries prevent SQL injection because the input is transmitted as data, never parsed as part of the SQL statement."*
- *"A CORS failure originates from the server not declaring the requesting origin as allowed, so the fix belongs in the server's response headers."*

---

## 14. Self test (cover the answers)

1. Why is PUT idempotent but POST not? -> *PUT specifies the full target state at a known URI, so repeats are identical; POST creates a new subordinate resource each time.*
2. 401 vs 403? -> *Not authenticated vs authenticated but not permitted.*
3. What does the virtual DOM actually save you? -> *Real DOM mutations; the diff computes the minimum set of changes to apply.*
4. Output of `console.log(1); setTimeout(()=>console.log(2),0); Promise.resolve().then(()=>console.log(3)); console.log(4);` -> *1 4 3 2 — microtasks drain before macrotasks.*
5. Why not use the array index as a React key? -> *It is not a stable identity; reordering or filtering makes React reuse the wrong nodes and misattach state.*
6. Session vs JWT for a horizontally-scaled API? -> *JWT, because it is stateless and needs no shared session store or sticky sessions; the trade-off is difficult revocation.*
7. Where do you fix a CORS error? -> *On the server, by returning the appropriate Access-Control-Allow-Origin headers.*
8. Why is Node poor at CPU-bound work? -> *A long computation occupies the single thread and blocks the event loop, delaying every other request.*
9. `const` — can you mutate the object it points to? -> *Yes. `const` prevents rebinding the identifier, not mutating the value.*
10. Hashing vs encryption for passwords? -> *Hash with a salt using a slow algorithm. Encryption is reversible, so a key compromise would expose every password.*

Next: [04_DEVOPS.md](04_DEVOPS.md)
