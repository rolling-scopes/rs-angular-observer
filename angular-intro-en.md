# Understanding Angular: eight sketches before the first lecture

If you're just starting an Angular course, you have two roads. The first — open the docs, read about components, directives, services, NgModules, Signals, RxJS, OnPush, DI, lifecycle hooks — and within an hour you'll be drowning in "there's too much here, I'll never get it." The second — spend half an hour on the eight sketches below, and then everything you meet in code and lectures will fall into prepared slots in your head.

These sketches aren't about API. They're about **how Angular thinks** and which decisions it has already made for you. The specific functions and classes change between versions — Signals were an experiment in 2023, became standard in 2024, default in 2025. But the philosophy behind them hasn't changed in ten years.

We're in 2026, current version is Angular 21[^1]. I'll mention API names along the way but not teach you how to use them. That comes on the course.

---

## How to read this document

There are two voices in the text.

**The main voice** — narrative, addressed as "you," through metaphors (where they actually help) and direct explanations (where a metaphor would just get in the way). You can read it on the subway; no IDE needed. The goal is to build the right mental model in your head.

**The engineering voice** — short inserts in this format:

> **⚙ Engineer.** Formal terms, precise APIs, the boundaries of the metaphor. If the narrator says "the electrical grid," the engineer says "Dependency Injection, implemented through hierarchical injectors." If a metaphor breaks somewhere, the engineer says so explicitly.

This voice exists so you don't leave the text feeling "I get it, but I don't know what it's actually called." All the real names and footnotes are its work.

In a couple of places you'll also meet:

> **🎛 Interactive (in the HTML version).** A description of a widget that will live here in the full web publication. In the markdown version it's just a description — because words can't really explain it, but a live graph or animation can.

If you're in a hurry — read only the narrator. If you're prepping for an interview — read both. If you reach a "🎛 Interactive" — know that there's a real conceptual jump there, and in the future HTML version it'll be visual.

---

## 0. Prologue · Why a UI framework at all

Any program that shows something to a user has an unpleasant property: at any one moment, **three things must be kept in sync**.

1. **State** — numbers, lists, flags in memory. "The cart is empty right now," "the user is logged in," "the timer shows 1:47."
2. **Screen** — what's actually drawn in the browser: the DOM tree, styles, input field contents, scroll position.
3. **Events** — everything coming from outside: clicks, keystrokes, API responses, timer ticks, WebSocket messages.

If the program is simple — a counter on a single button — keeping things in sync is easy by hand. Click — increment the number — write it to the DOM. Done.

If the program is more complex — a chat with history, notifications, auto-highlighting of new messages, reconnection when the network drops, draft saving — the volume of "glue" between state and screen grows nonlinearly. For every pair of "something in memory changed ↔ some piece of UI must update," you have to hand-write a rule: "when a message arrives — append it to the list, scroll to the bottom, update the unread counter, but only if the window is minimized; if it's active — don't update." Such rules quickly become hundreds, and **90% of bugs in a live UI are state-screen drift**. Not logic errors, not network errors. Specifically "I clicked but it didn't update" or "it updated, but the wrong thing, or in the wrong place, or twice."

UI frameworks emerged for one main job: to take this synchronization on themselves. You describe **what the UI should look like at each state** (this is called the "declarative approach"), and the framework figures out exactly what to recreate in the DOM, what to update, what to leave alone. You no longer write "if it changed — update this div." You write "this is how it looks," and the framework diffs "what was" against "what should be" and applies the minimal set of changes.

That's half the deal. The other half — frameworks bring along ten more utilities you need in any nontrivial UI: routing (how to move between screens), forms (how to handle complex input), requests (how to talk to the server with retry, interceptors, cancellation), state management (how to share data between parts of the app), tests (how to verify any of this works), build tools, linters, IDE plugins.

You can write all of this yourself, in plain JavaScript. It's a normal engineering task — such people exist, and they don't complain. But that's one or two thousand lines of infrastructure you'll then maintain for the rest of the project's life, alone, without an ecosystem.

There are many frameworks, and they're different. Each has **its own philosophy**: its own view on how best to declare UI, how to handle state, how many decisions to make for the developer, where to draw the line between "magic" and "explicitness." React, Vue, Svelte, Solid, Angular — these aren't "one good and the rest bad," they're different points on a tradeoff map. Choosing a framework means choosing the set of decisions that have already been made for you.

What follows — seven sketches about which decisions Angular made and why these particular ones. And one — about why modern Angular is built differently from its own decade-old version.

---

## 1. A camp with a ready kitchen
### Framework vs library · opinionated all-in-one against "core + ecosystem"

Imagine you decide to go hiking in the mountains. You have two options.

The first — pack the bag yourself. Get a tent from one friend, a sleeping bag from another, find a pot online, order a gas burner, forget the rain cover and buy it on the last day at a roadside store. Along the way it turns out the tent has one type of stake but the mountain ground takes another, and the rainfly is sewn crooked. It's not a catastrophe — you'll figure it out. But you'll spend half the trip fixing the bag instead of walking.

The second — sign up for a camp where identical tents are already pitched, the kitchen is ready, the guide knows the route, and there's an evacuation plan if someone twists an ankle. You don't need to pack a bag — you need to show up and walk. The camp answers 50 small "how" questions for you and leaves you the one big "where to" question.

**React is closer to the first. Angular — to the second.**

With React you build your stack out of independent parts: a router (react-router, tanstack-router), state management (redux, zustand, jotai), forms (react-hook-form, formik), an HTTP client (axios or fetch), tests (jest, vitest). Each has its own API, docs, release cycle, and community. That's freedom: take exactly what you like, swap it when you want. It's also responsibility: when ecosystem leaders shift, parts of the stack eventually need rethinking. The price is known — it just helps to see it ahead of time.

Angular comes as the "camp." In the box: Router, Forms, HTTP, Animations, a Service Worker for PWA, Vitest tests, an AOT compiler, an esbuild-based build system, version-update tools, IDE plugins. All consistent in style. All updated by one command `ng update`. All maintained by one Google team.

The price — you have less freedom of choice. If you like green tents and the camp's are all yellow — you live with yellow. That's a fair price for a 5+ person team working on a project for 5+ years. Less fair for a solo developer building a weekend landing page. Hence the typical geography: Angular is a frequent choice in banks, airlines, healthcare, government, and inside Google itself. In early-stage startups it's picked less often: there, stack plasticity matters more than predictability. It's not "good vs bad" — it's two different optimizations for different contexts.

There's another flip side to "the camp": **you eat what's served**. If a hot new state-management library appears tomorrow (Zustand, Jotai, valtio — that happens in React every year or so), nothing formally stops you from plugging it into an Angular project. But you'll be carrying your own dishes past the shared kitchen — over Signals, over RxJS, against the framework's idioms. In React, "your own dishes" is normal — that's what it was designed for. In Angular it's an exception, and the tooling ecosystem doesn't expect it.

> **⚙ Engineer.** Formally, Angular is an **opinionated full-featured framework**. All official packages (`@angular/core`, `@angular/router`, `@angular/forms`, `@angular/common/http`, `@angular/animations`, `@angular/cdk`, `@angular/material`, `@angular/ssr`) live in one monorepo[^2] and release in sync. This doesn't mean "everything ends up in the bundle": tree-shaking[^3] strips unused code at build time. In practice: import only from `@angular/*` at the start; reach for third-party ecosystem only when needed, not to plug architectural gaps.

> **One shift that changes everything:** When you first open a fresh Angular project, don't look for "missing" libraries. They're not there because they're already built in. What you imported in React from ten different npm packages, in Angular you import from one — `@angular/...`.

---

## 2. The template is TypeScript, just with brackets
### Templates as typed programs · AOT + strictTemplates

In the prologue we said: the framework's main job is to take state-screen synchronization off your hands, so you describe _how_ the UI looks given the data, not _what_ to do to the DOM when something changes.

This "descriptive layer" exists in every UI framework, and it's called roughly the same everywhere — **template** (or "template," "view," "render function" — the essence is the same). It's a piece of your code that says: "given such-and-such state, draw such-and-such structure." It looks different in different frameworks. In React — it's a function returning JSX: a mix of HTML-like syntax and JavaScript expressions. In Vue and Svelte — a separate markup block in the `.vue`/`.svelte` file, next to `<script>` and `<style>`. In Angular — a separate HTML file (or inline string) next to the component class.

That's the shared part. Then differences begin. And **the way Angular processes its template is one of the most characteristic decisions of the framework**. We'll start there: if you understand this one thing, a lot becomes immediately clear about why Angular code looks the way it does.

The first thing that surprises people moving from React to Angular — the HTML template stops being HTML.

```html
<input [value]="user().name" (input)="onChange($event)">
```

`[value]="user().name"` isn't an HTML attribute, it's an expression. Square brackets — "compute and substitute." Round ones — "this is an event handler." Inside — real TypeScript: a call to the signal `user()`, accessing the `.name` field. Angular reads your template not as markup but as **a TypeScript program** embedded in tags.

Before the file reaches the browser, the Angular compiler walks every template and checks: does `user` exist on your component, does its call return an object with a `name` field, is the type of that field compatible with the `value` attribute. If you've made a typo — `user().nme` instead of `user().name` — the project won't build. The error appears in the terminal and IDE, not in the browser console a week after deploy.

This is called **AOT compilation** (ahead-of-time): the compiler translates your HTML into optimized TypeScript code before the browser even downloads it. The browser doesn't get strings with `{{ … }}` placeholders — it gets ready-made function calls that draw the necessary DOM nodes.

Three practical consequences:

**Types reach into HTML.** Change a field's type in the component — every template using it lights up red in the IDE immediately. It's not a separate validation system, it's the same TypeScript as in `.ts` files.

**Templates are fast.** At runtime the browser has nothing to parse — the compiler has already analyzed and optimized everything. On a large app this is tens of milliseconds saved per render.

**Templates are disciplined.** In an Angular template you can't write arbitrary JavaScript — only expressions. No raw `for` or `if` — only declarative `@for`, `@if`, `@switch`. This is annoying until you notice that templates across the entire project read the same way, because writing "your own way" is physically impossible.

> **⚙ Engineer.** Technically, the template is a DSL called the **Angular Template Language**. The compiler (current engine — **Ivy**[^4], since Angular 9, View Engine removed in v13) turns it into a series of low-level render instructions (`ɵɵelement`, `ɵɵproperty`, `ɵɵadvance` and so on), which are AOT-optimized and tree-shakable. In Angular 21 the standard builder is esbuild-based. Type-checking template expressions is enabled via `strictTemplates: true` in `tsconfig.app.json`[^5]; without it, some errors slip to runtime. JIT rendering (runtime compilation) is disabled in production bundles — production is always AOT.

> **🎛 Interactive (in the HTML version).** Two-pane live playground: on the left, an editable template with a typo `<input [value]="user.nme">`, on the right a live compiler log: `Property 'nme' does not exist on type 'User'. Did you mean 'name'?`. Fix the typo — the error disappears. 30 seconds of interaction conveys "typed template" better than any paragraph. Budget: ~0.5 day.

> **One shift that changes everything:** Don't read an Angular template as HTML. It's a small language designed specifically for UI, checked by the compiler as strictly as the rest of TypeScript. A typo in a template is a build error, not a production surprise.

---

## 3. The electrical grid in the walls + branch panels on each floor
### Dependency Injection · inversion of control, injector hierarchy, swapping in tests

In most programs, when one piece of code needs another, it grabs it directly: imports, instantiates, uses. It's like a battery store: a lamp needs a battery — you go, buy it, plug it in. With 5 lamps, fine. With 500 — you spend your life buying and replacing batteries.

Angular doesn't work that way. In Angular, **the electrical grid runs through the walls**, and lamps connect via outlets. This gives three different things that are hard to organize separately but come naturally together. Layer by layer.

**Layer 1 — inversion of control.** The lamp doesn't buy batteries. It says: "I need electricity, 220 volts." Where it comes from isn't its concern. Could be the city grid, a generator at a country house, a solar panel, a battery in a van. The lamp works with any source as long as the interface fits.

In code, the same: a component doesn't create `new HttpClient()`. It tells the framework: "I need an `HttpClient`." The framework finds the right service and supplies it. The component does its job — drawing UI; the service does its — talking to the server. Their connection is described through an interface, not through `new`.

**Layer 2 — hierarchy.** A large building doesn't have a single grid. In the basement — the building's substation. On the fifth floor — a separate UPS for the server room: if the building loses power, servers keep running off it. And in a particular meeting room — its own power strip with a noise filter for an important video conference.

When a lamp in the meeting room asks for current, the search goes from bottom up: first the strip (does it have power?), then the floor's UPS, then the building substation. Stops at the first match.

In Angular it works literally the same way: the application has a root injector (the building substation), a lazy route has its own injector (the floor UPS), a component has its own (the meeting-room outlet). A service can be "wired in" at any level, and all descendants will get exactly that version, not the shared one.

This means: if your admin area needs a special logger with extended permissions, you declare it at the admin route level, and all components inside admin get exactly it — without touching the components themselves and without affecting the rest of the application.

**Layer 3 — substitution for tests.** You don't modify the lamp in a test. You feed it current not from the grid but from a lab power supply. From outside, the lamp doesn't notice the difference: same plug, same 220 volts.

In Angular component tests, the same. You don't modify the component to "check it without real HTTP." You tell the test injector: "when someone asks for `HttpClient`, hand them this stub." The component receives the stub through the same DI mechanism, knowing nothing about it.

> **⚙ Engineer.** The pattern is called **Inversion of Control**, the specific implementation — **Dependency Injection**[^6]. In Angular there's a four-level injector hierarchy: `NullInjector → PlatformInjector → EnvironmentInjector → ElementInjector`. The lookup goes from bottom up, stops at the first matching `Provider`. The modern way to obtain a dependency is the `inject()` function, available in injection context (constructor, field initializer, factory provider, functional `CanActivateFn`/`HttpInterceptorFn`/`ResolveFn`). Providers are declared via `@Injectable({ providedIn: 'root' })` (application root), `providers: [...]` in `Route` (route scope), or `@Component({ providers })` (component scope). For non-standard types (interfaces are erased at runtime) use `InjectionToken<T>`. In tests, substitution — via `TestBed.overrideProvider()` or `{ provide: X, useValue: mock }`.

DI exists elsewhere too: Spring in Java, NestJS in Node, .NET in C#. It's not unique to Angular. But among frontend frameworks of 2026, Angular is the only one with DI built into the core and used at every step. React and Vue don't have it; Svelte doesn't. This is one of the main differences — and one of the reasons Angular is chosen for large long-lived systems: there DI pays off in year three of the project's life.

> **🎛 Interactive (in the HTML version).** An interactive injector tree with three levels: root → "Admin" route with an overridden `LoggerService` → `UserList` component. Click on a component — the lookup path lights up the tree, showing where the specific instance of each service came from. Useful for "aha!" moments about scoped overrides. Budget: 0.5–1 day.

> **One shift that changes everything:** When you see `inject(SomeService)` in Angular code, or a constructor with typed parameters — don't look for where this service is created. It's not created by you. It's brought to you. Your job is to ask and use.

---

## 4. Angular as Google Sheets
### Fine-grained reactivity · Signals, change detection, zoneless

Open Google Sheets (or Excel, if you used it back in 2005 — same idea).

In cell B1 — the number **10**. In cell A1 — the formula `=B1*2`, showing **20**. Change B1 to **15** — A1 recomputes itself to **30**. You didn't write code "when B1 changes, recompute A1"; the spreadsheet itself knows the dependency graph and knows that when B1 changes, exactly A1 needs updating, and the neighbors C1, D1 — leave alone.

**Signals in Angular are exactly a spreadsheet for your application state.**

`signal(0)` — a cell with a number. `computed(() => a() * 2)` — a formula. The template — a cell that shows the value to the user. When the value in any "cell" (signal) changes, Angular knows the exact graph — who depends on it — and updates only those places, not the whole sheet.

In code it looks almost like a regular variable:

```typescript
const counter = signal(0);                          // a cell with the number 0
counter();                                          // read it — got 0
counter.set(1);                                     // wrote — now it's 1
const doubled = computed(() => counter() * 2);      // formula: a cell that recomputes itself
```

That's the short explanation; now the long story, **how Angular got here**. It helps you understand why modern Angular has the mechanisms it does and not others.

**Era one — the watchman with a flashlight (Zone.js, 2014–2024).** Angular originally solved the "data ↔ screen" sync problem like this: let there be a special watchman who, after every async event (click, server response, timer), walks the **entire component tree** and checks whether anything has changed. If so — redraw it. It's reliable: you won't miss anything. It's slow: on a large app the watchman tires. It's "magical": you don't have to mark anything, it all works on its own — but it's also unclear why it stops working at unexpected moments. The mechanism relies on the Zone.js library, which "patches" standard browser APIs to catch any async activity.

**Era two — switches in every room (OnPush, 2016–2024).** Angular added the option to tell each component: "watchman, only visit this component if its inputs changed or an event happened inside." This cut the watchman's work tenfold, but added a developer obligation: think about when to "mark a component as dirty" by hand. Hence the famous `OnPush` mode, and along with it the famous bug class "the screen isn't updating because I forgot to call `markForCheck()`."

**Era three — the cells know themselves (Signals + zoneless, 2024–2026).** The idea is simple: instead of a watchman who walks the whole house after every sneeze, let every variable that has dependents keep a list of "who reads me." When the value changes — it pings everyone who depends on it. No traversal, no "wash me," no Zone.js. That's Signals.

And this isn't "the future we're heading toward," it's **the present that's already here**. Signals stabilized in Angular 17 (May 2024), zoneless mode became stable in 20.2 and **the default for new projects in Angular 21 (November 2025)**[^7]. When you run a fresh `ng new` in 2026 — there's no `zone.js` in the dependencies. None.

> **⚙ Engineer.** The model is called **fine-grained reactivity**. The idea isn't new: it exists in Knockout (2010), MobX (2015), Vue 3 (2020), SolidJS (2021). Angular got there via RFC 012[^8] in 2023, stabilized in v17, made it default in v21. Primitives: `signal()` — writable, `computed()` — derived read-only, `effect()` — side effect (reactively re-runs when read signals change), `linkedSignal()` — writable computed with access to the previous value. Zone.js is disabled with the `provideZonelessChangeDetection()` provider; in new v21 projects it's absent from `dependencies`. Important rule: mutating an object without calling `set()` / `update()` on a signal does **not** trigger a redraw — immutability discipline remains mandatory.

> **🎛 Interactive (in the HTML version) · widget A.** A live dependency graph. Three `signal`s — `firstName`, `lastName`, `age` (editable inputs). Two `computed`s: `fullName = firstName + lastName`, `isAdult = age >= 18`. One `effect` that updates `document.title`. Graph nodes are circles, dependencies are arrows. Change a value in an input — only the nodes that actually recomputed light up, in propagation order. Budget: 1–2 days.

> **🎛 Interactive (in the HTML version) · widget B.** A 3×4 component tree, one button "change a value in component X." A toggle "Zone.js / Signals." In Zone.js mode — after a click the entire tree lights up (the watchman effect), counter "checked 12 nodes." In Signals+zoneless mode — only X and the path to root light up, counter "checked 2 nodes." This conveys the idea of fine-grained reactivity in 3 seconds of observation better than any paragraph. Budget: 1 day.

> **One shift that changes everything:** When you read modern Angular code and see `signal()`, `computed()`, `effect()` — don't try to figure out "when this runs." Think backwards: "what depends on what." Reactivity is about connections, not timing.

---

## 5. Recipe vs ready dish + factory conveyor
### Reactive streams · RxJS Observables and data spread over time

The previous sketch was about **state that lives in memory right now**: selected symbol, cart, timer. Signals are the perfect tool for this. But there's another kind of data — the kind **spread over time**.

You clicked a button — that's an event. 200 milliseconds later — another click. A second later, the server sent a response. Two seconds later, a second response, already stale. Five minutes later, the user left the page — and the subscription should detach. This is a stream of events: not an array, not a variable, but a chain of things, each with its own moment in time.

In a regular program, this is handled with handlers (`addEventListener`), timers (`setInterval`), promises, and a heap of manual flags. Subscription didn't detach — leak. Two requests came in the wrong order — yesterday's search results on screen. The handler doesn't remember the old timer — the button fires twice. These are hand-stitched patches, and this is exactly where the nastiest UI bugs live.

RxJS is a different approach. The event stream is treated as **a first-class object**: it has a name, you can combine it with others, filter, transform, subscribe, and unsubscribe. The key operation is subscription, and two kinds of streams come with it.

**A Cold Observable is a recipe.** You give the recipe to people. Everyone who reads it cooks their own portion from scratch: buys ingredients, puts it on the stove, waits five minutes. One recipe — any number of portions, each separate.

In code, an HTTP request works this way: `http.get('/users')` — that's a recipe. Every subscriber triggers **their own** request to the server. Five subscribers = five requests.

**A Hot Observable is a ready dish on a buffet.** It's already cooking (or already hot on the counter), no matter how many people show up — they get from the current moment, the same dish. Missed the first three minutes — that's lost.

In code, this is how button clicks or WebSocket messages work: the stream flows independently of whether you're subscribed or not. All subscribers get **the same** events, from the moment of subscription onward.

This core distinction — through the river metaphor ("you can't step in the same river twice") doesn't come across — a river has no explicit difference between "the source appeared when you walked up" and "the source was always there." Here the recipe/dish metaphor is more precise.

Now **operators** — the second strong idea of RxJS. An operator is **a station on a conveyor belt**: a stream comes in, a stream comes out, just changed. Each station has one narrow job.

- `filter` — a sieve: passes only what fits, drops the rest.
- `map` — a transform station: turns each element into another (e.g., a JSON object into a string).
- `debounceTime(300)` — a timer-buffer: lets an element through only if 300 milliseconds of silence followed it. Classic pattern for "as the user types" search.
- `switchMap` — a station with a job swap: got a new element — drop the old work, start the new. Classic for "cancel the previous request, send a new one."
- `merge`, `combineLatest` — mixers: combine multiple streams into one by different rules.

This chain "stream → operator → operator → subscription" — that's a typical RxJS pipeline. Instead of 20 lines with manual `setTimeout`, `clearTimeout`, `isAborted`, `lastRequest = currentRequest` — one chain:

```typescript
searchQuery.valueChanges.pipe(        // stream of values from the input field (each keystroke)
  debounceTime(300),                  // wait 300ms while the user finishes typing
  distinctUntilChanged(),             // don't bother if the same text was entered
  switchMap(q => http.get(`/search?q=${q}`)),  // cancel the old request, send the new one
).subscribe(result => render(result));              // each server response — render
```

This is a **declarative description of behavior over time**, in one construct. Stations can be rearranged, added, removed — and the whole pipeline changes consistently.

> **⚙ Engineer.** RxJS is the implementation library for the **Reactive Extensions** pattern, invented by Erik Meijer in a Microsoft team in 2009 on .NET[^9], ported to dozens of languages (Rx.Java, Rx.Swift, RxJS). In Angular it's built in as a required dependency: `HttpClient.get()` returns an `Observable`, `Router.events` — `Observable`, `FormControl.valueChanges` — `Observable`. It's not a choice, it's a fact. Core primitives: `Observable` — a source of events over time; `Observer` — the one who subscribed; `Operator` — a "stream-to-stream" function; `Subject` — both Observable and Observer at once, used for multicasting and bridges to non-reactive code. More on cold/hot — in the article by RxJS author Ben Lesh[^10]: the producer is created inside the subscription (cold) or outside (hot) — that's the formal distinction.

> **🎛 Interactive (in the HTML version) · rxmarbles.** An embedded `<iframe>` from [rxmarbles.com](https://rxmarbles.com/) showing operators `map`, `filter`, `debounceTime`, `switchMap`, `merge`. Marble diagrams are the canonical way to visualize RxJS — Ben Lesh himself uses them in talks. In markdown it's just a link, in the HTML version — a live widget right in the text. Budget: 1 hour.

> **🎛 Interactive (in the HTML version) · hot vs cold.** Two animations side by side. Left, cold: every time you click "subscribe," a fresh timer-counter starts from zero. Right, hot: a single timer is already ticking, the subscription just connects at the current moment — whoever subscribed earlier sees more. This conveys the part the recipe/dish metaphor explains in text, and the interactive — locks in. Budget: 1 day.

What about Signals? Signals appeared and closed 80% of what RxJS used to be needed for inside components — storing local state. But RxJS **isn't going away**: it remains "at the edges of the application" — where events come from outside and are spread over time. HTTP, WebSocket, router events, reactive form changes, timers.

The bridge between them — two functions from `@angular/core/rxjs-interop`:

- `toSignal(obs$)` — turns a stream into a signal: takes the latest value.
- `toObservable(sig)` — turns a signal into a stream: emits on every change.

Rule of thumb: **HTTP/WS/events → RxJS. UI state → Signals. At the boundary — interop.**

One caveat. RxJS isn't a simple tool. There are 150+ operators, of which truly useful — about 20, but understanding _which_ 20 and _when_ each — that's half a year of practice. It's beautiful when you've mastered it, and torturous when you haven't. A separate week of the course is dedicated specifically to it — for good reason.

> **One shift that changes everything:** Don't think of RxJS as the "Angular way of events." Think of it as a new data type — "a value stretched over time." Promise — one moment in the future. Observable — many moments in the future. Once that shift happens, operators start composing in your head on their own.

---

## 6. Angular as Ubuntu on a train schedule
### Predictable releases · semver, LTS, and `ng update` as a culture of migrations

Forget about "fast-running frameworks where everything breaks next year." Angular runs on a train schedule.

**A new major comes out every six months.** May and November. Since 2017 — not a single miss. That means an upgrade isn't a fire, it's a planned event you can put on the team calendar.

**Each major has exactly 18 months of life:** 6 months of active support (bugfixes, improvements, security) and 12 months of LTS (security and critical regressions only). When the next major releases, the previous one automatically moves to LTS. At any moment there are roughly three supported major versions.

**Breaking changes are prepared in advance.** Six months before something actually breaks, it's marked `@deprecated` and a warning appears in your IDE. At the moment of the break — almost always shipped with an automatic migration script (`schematic`) that rewrites your code itself. The command `ng update @angular/core` isn't just "change the version number in package.json"; it's "run an AST refactoring that turns deprecated patterns into new ones." The transition from NgModules to standalone (v14→v17), from Zone.js to zoneless (v18→v21) — both done through this mechanism, not through mass app rewrites.

If you know Ubuntu — you already understand Angular's model; it's built on very similar principles:

| Parameter | Ubuntu | Angular |
|----------|--------|---------|
| Release cadence | Every 6 months (X.04, X.10) | Every 6 months (spring, fall) |
| LTS support | Each even .04 — 5 years standard | Each major — 12 months after the active phase |
| Auto-migration | `do-release-upgrade`, `apt` | `ng update`, `schematics` |
| Policy | [ubuntu.com/about/release-cycle](https://ubuntu.com/about/release-cycle) | [angular.dev/reference/releases](https://angular.dev/reference/releases) |
| Community | Canonical + community | Google + community |

The differences are in the details: Ubuntu holds LTS for five years, Angular — one year in the LTS phase (so the total support window per version is ~18 months vs Ubuntu's 5 years). But **the skeleton is the same**: predictable cadence, overlapping support windows, a package manager with migrations. It's a passenger train on a schedule, not "we'll fly if the weather permits."

> **⚙ Engineer.** As of April 2026: Angular 21 — active support (until May 2026), Angular 20 — LTS (until November 2026), Angular 19 — LTS (until May 2026, near EOL)[^11]. Release policy is fixed in the official docs: strict **semver**, two minor versions between majors. Migrations are implemented through `@angular/cli` schematics — versioned scripts that modify source code through AST refactoring (not regex). It was schematics that made the painless transition from decorator-based NgModules to standalone components possible, and from Zone.js to zoneless.

> **🎛 Interactive (in the HTML version).** Horizontal timeline of Angular versions 17–22. Green windows — active support, yellow — LTS, gray — EOL. The current moment ("you are here" — April 2026) is marked with a vertical line. Hover on a version — popup details: release date, key features, EOL date. In markdown it's a table; in HTML — SVG. Budget: 0.5 day.

In practical terms: an Angular project written in 2019 can be updated to the latest version in 2026 — and it'll work. This is the reality for dozens of enterprise teams, not marketing. In ecosystems with faster evolution this is harder: code five to seven years old often needs noticeable repair before upgrading to a modern stack. That's not a sign of a "bad" ecosystem — it's a different contract between framework author and consumer: more iteration speed in exchange for more effort keeping code current.

The price of Angular's stability — it sometimes looks conservative. When a hot new pattern appears, it lands here not immediately but a year or two later, when the team is convinced it's truly needed by the majority and finds a way to add it without breaking old code. That's a deliberate choice: you pay with the speed of getting novelties and you get the invariant "what worked in v17 works in v21."

> **One shift that changes everything:** If you're used to thinking "a framework is something that goes obsolete every year" — start treating Angular as a skill you learn once and exploit for ten years. Updates are a train schedule, not a lottery.

---

## 7. Explicit Angular
### Standalone and the philosophy of "explicit over implicit"

This is a short but important sketch — about the fact that **Angular 2026 is built fundamentally differently from Angular 2016**, and it's exactly this new version you're about to enter on the course.

Early Angular was full of magic — in a good sense, but magic. Zone.js silently watched any async, so you wouldn't have to think about change detection. `NgModule`s collected declarations, imports, and providers, and this "furniture assembly in the room" had to happen before a component could be placed. The `async` pipe subscribed and unsubscribed itself. Half of what the framework did was implicit: code "just worked," and if something broke — it was unclear where.

Angular 2026 is **explicit**. The main shift: everything the framework does with your state and your components is now visible in code.

**Example 1 — state.** Before: you have a variable, the framework compares it with the old value after any async event. Now: `const counter = signal(0)` — explicitly "this is a reactive value." `counter()` — explicitly "I'm reading it." `counter.set(1)` — explicitly "I'm changing it." Not a single implicit operation.

**Example 2 — components.** Before: `@NgModule({ declarations: [...], imports: [...], providers: [...] })` — the furniture was assembled in a separate room. Now: `@Component({ imports: [ButtonComponent, UserPipe] })` — the component itself says what it needs. No intermediaries. That's **standalone**: with Angular 19 — standard, with Angular 21 — default in `ng new`. `NgModule` doesn't appear in new code at all.

**Example 3 — change detection.** Before: `bootstrapApplication(AppComponent)` — and somewhere in the depths of `zone.js` a background mechanism starts, re-checking the tree after every sneeze. Now: `bootstrapApplication(AppComponent, { providers: [provideZonelessChangeDetection()] })` — and you've explicitly said "we work without Zone.js, all changes — through signals and events." In Angular 21 this is the default for new projects.

The general principle: **explicit over implicit**. The framework does less magic, but you see what it does. This cost the Angular team several years of gradual migrations (each — with its own schematics), but the result — an Angular where it's easier for a newcomer to reason about code. You need to remember less about "how this works under the hood"; more about "what I wrote explicitly."

A short metaphor, if needed: **modern Angular is furniture on wheels, not nailed to the floor**. It's explicitly moved, explicitly assembled, explicitly arranged. More hand movement — but much less "why did my table disappear, I didn't touch anything."

> **⚙ Engineer.** Key transition dates: Signals stable in v17 (May 2024), standalone components default in v19 (November 2024), standalone in `ng new` — v17, zoneless stable in v20.2 (June 2025), zoneless default in v21 (November 2025)[^7]. `NgModule` isn't removed, remains for backward compatibility, but doesn't appear in new code. Migration of existing code — through `ng generate @angular/core:standalone`[^12], which converts NgModule-based projects step by step (components → NgModule removal → bootstrapApplication).

> **One shift that changes everything:** If you've read old Angular tutorials (2018–2022) — forget about `NgModule` and `zone.js` when working with new code. They still work, but in 2026 you don't write them. You write standalone, you write signals, you write explicit providers — and it becomes pleasant.

---

## Epilogue · All eight parts as one system
### Cohesion · why these eight decisions assemble together

The eight sketches look like eight separate ideas. In reality, they're a single design, in which each decision was made with the others in mind. Remove any one — the neighbors start crumbling. To see this, let's trace one ordinary scenario: a user clicks "Buy" in a tiny Angular app. And watch how the eight sketches meet in one click.

**Step 1. The template catches the event.** The button `<button (click)="buy(item)">Buy</button>`. `(click)` isn't an HTML attribute, it's a typed handler binding (sketch 2): the compiler verified that the `buy` method exists on the component and accepts an `Item`. If there's a typo in the template or a type mismatch — the project won't build.

**Step 2. The component receives dependencies through DI.** In the component class — the line `private cart = inject(CartService)`. No `new CartService()` (sketch 3) — Angular climbs the injector hierarchy and supplies the right instance. If `CartService` was substituted with a mock in the test — the component doesn't know.

**Step 3. The component updates a signal.** `this.cart.add(item)` calls a method on `CartService` that writes to a signal: `this.items.update(list => [...list, item])`. This is an explicit state change (sketch 4, 7). No Zone.js magic is involved — there isn't any in this app.

**Step 4. Everyone who depends on the signal finds out.** In the same service lives `total = computed(() => this.items().reduce(...))`. The dependency graph knows: `items` changed — recompute `total`. At the same time `badge = computed(() => this.items().length)` also changes — the counter in the header.

**Step 5. The template re-renders automatically.** In the header sits `<span>{{ badge() }}</span>`. The template reads the signal — the framework knows: when `badge` changes, redraw exactly this node (sketch 4). Not the whole tree, not its parents — only this `<span>`.

**Step 6. If a server is needed — RxJS steps in.** Suppose adding to the cart should be persisted on the backend. `CartService` injects `HttpClient` (sketch 3 again), calls `this.http.post('/cart', item)` — gets an `Observable` (sketch 5). Through `pipe(retry(3), catchError(...))` it handles network failures; through `toSignal` the response comes back as a signal. RxJS — at the system boundary, signals — inside. They **don't duplicate** each other, they complement: each does what it's strong at.

**Step 7. The test reproduces the same without a server.** In a unit test, `HttpClient` is substituted via DI with a stub that returns a pre-recorded `Observable`. The component doesn't know there's no real server — for it the interface is the same. This works because DI is a shared system across all layers, and the test can "intercept" any seam.

**Step 8. Tomorrow Angular 22 ships.** Six months later — the next major. `ng update @angular/core@22` — schematics will automatically rewrite whatever's needed (sketch 6). Standalone components, signals, injectors — formally won't change, because these primitives are stabilized and supported under semver. The camp changes (sketch 1) — but without moving its residents.

Eight steps — eight sketches, each kicking in at its moment. Now, to see the system from above, let's mark **three through-lines** that pass through all these steps.

**Line 1: Dependency Injection as the spine.** Through it move almost all objects in the system: `CartService`, `HttpClient`, `Router`, your own services. Once you understand how DI works — you understand how all the other parts connect. That's why DI gets a whole sketch (sketch 3), although "technically" it's just a way to create objects.

**Line 2: Types as glue at every seam.** The template checks that the handler matches the component. The component checks that DI will deliver the right type. The signal knows its type. The RxJS Observable knows the type of its events. Through `toSignal/toObservable` types are also preserved. It's all the same TypeScript, and not a single seam through which the type would be lost.

**Line 3: Reactivity in two layers.** Inside the application — Signals (synchronous state, recomputation graph). At the boundaries — RxJS (asynchronous streams over time). They don't compete, they share territory. Each — better on its turf; together — cover everything a live UI needs.

These three lines make Angular **a coherent whole**, not an assembly kit. Angular isn't "a framework that has DI, signals, and RxJS"; it's **a framework in which these three things are designed under a shared contract**. That's why you can't "take a piece of Angular," the way you take Zustand or axios separately from React. You agree to the whole set — or you don't. And when you do, it works smoothly precisely because **the eight decisions are chosen with each other in mind**.

> **⚙ Engineer.** In architectural terms, this is called **opinionated cohesion** — high cohesion at the cost of less universality. The sign: migration between majors is a series of coordinated steps, not episodes of "update React, update redux, update react-router separately." The price of cohesion for the Angular team — every new major is prepared with compatibility for all other parts in mind (the RFC process[^8], schematics[^12] — technical instruments of this coordination).

> **One shift that changes everything:** When learning Angular, don't read the docs for one part (DI, signals, RxJS, templates) in isolation. Read through scenarios: "what happens when…", and watch how the sketches meet. Understanding the system is understanding its **connections**, not the individual parts.

---

## What's next

If even one of these eight sketches landed — you're already on the right path. Don't memorize details. Memorize directions:

0. A UI framework takes state↔screen sync on itself plus brings a dozen adjacent utilities. Each framework is a coherent set of answers, not a separate tool.
1. Angular gives a lot, but requires you to play by its rules. It's a deal, not a library.
2. The template is a TypeScript program in HTML brackets. Trust it like you trust functions.
3. Services are brought to you through the injector hierarchy; in tests you change not the component but what comes from the outlet.
4. Signals are Google Sheets for your state: cells, formulas, recomputation graph.
5. RxJS is streams over time: recipes (cold) and dishes (hot), with a conveyor of operators. Signals aren't its competitor, they're its neighbor.
6. Releases are a train schedule, like Ubuntu's: predictable, with LTS windows, with `ng update` like `apt`.
7. Modern Angular is explicit. Signals, standalone, zoneless — all through providers and imports, without magic.

On the course all of this turns into code. You'll see `inject()`, `signal()`, `computed()`, `provideRouter()`, `bootstrapApplication()`, `@for`, `@if`, `OnPush`, `switchMap`, `takeUntilDestroyed`, `toSignal` — dozens of names. Each will fall into one of these eight slots. If the slots are prepared — learning will be easy.

One more short note — on AI. Angular 21 introduced a built-in MCP server for AI assistants and improved editor integrations: the IDE suggests not just "this is a function," but "you're mutating an object under OnPush — there won't be a redraw." Convenient, but doesn't replace understanding. The hint will work only if you yourself know what OnPush is. That's why the prologue is so long, and the AI paragraph — one.

Good luck on your first hike.

---

## Footnotes

[^1]: Angular Releases and current version — https://angular.dev/reference/releases

[^2]: Angular monorepo: https://github.com/angular/angular — one repository with all official packages; `packages/` shows the structure.

[^3]: Tree-shaking in Angular is possible thanks to `providedIn: 'root'` on services (tree-shakable providers) and ES modules as the distribution format.

[^4]: Ivy — codename for the current Angular render engine (since v9, 2020). Before Ivy was View Engine, and before that — the original Angular 2 compiler. https://angular.dev/tools/cli/aot-compiler

[^5]: `strictTemplates` — part of "Strict mode" in TypeScript/Angular: https://angular.dev/tools/cli/template-typecheck

[^6]: Martin Fowler's original article on DI and IoC (2004), the basis for much of the modern understanding of the pattern: https://martinfowler.com/articles/injection.html

[^7]: Zoneless in Angular 21 as default — official guide: https://angular.dev/guide/zoneless and the push-based.io breakdown: https://push-based.io/article/angular-v21-goes-zoneless-by-default-what-changes-why-its-faster-and-how-to

[^8]: Angular RFC 012: Signals — https://github.com/angular/angular/discussions/49685. A short design document that described the motivation and API even before stabilization.

[^9]: Erik Meijer — creator of Reactive Extensions at Microsoft. His lectures and articles ("Your Mouse is a Database," 2012) are one of the best introductions to the topic. ReactiveX repository: https://reactivex.io

[^10]: Ben Lesh (RxJS author and maintainer) on hot vs cold Observables: https://benlesh.medium.com/hot-vs-cold-observables-f8094ed53339. Marble diagrams for interactive practice: https://rxmarbles.com/

[^11]: Current EOL dates for each Angular version — https://endoflife.date/angular. For comparison with Ubuntu: https://ubuntu.com/about/release-cycle

[^12]: Migration to standalone: https://angular.dev/reference/migrations/standalone — step-by-step automated refactoring through Angular CLI.
