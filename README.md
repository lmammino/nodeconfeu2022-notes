# nodeconfeu2022-notes

My notes from NodeConfEU 2022

## My Twitter Summaries:

- [Day 1](https://twitter.com/loige/status/1577203372737929218)
- [Day 2](https://twitter.com/loige/status/1577568572603305986)
- [Day 3](https://twitter.com/loige/status/1577736153184321553)
- [Final summary](https://twitter.com/loige/status/1578043892947099656)

# day 1

## Liz Parody - new features in Node.js
- built in test runner
	- node:test + node:assert
	- --test flag
	- inspired by Deno (hapi, mocha and jest)
	- just run the test file
	- tests can be skipped
	- you can do subtests
	- suggested external libs (proxyquire, esmock, sinon)
- Watch
	- restarts app on file changes
	- inspired by Nodemon
	- node --watch file
	- node --watch-path `path` file
- Stream accessors
	- streams are good for large amounts of data and composition
	- memory efficient and time efficient
	- types of stream (readable, writable, transform, duplex)
	- NEW: map / filter / reduce
- Argument parsers
	- inspired by commander and yargs but much more lightweight and unopinionated
	- commander 93mln downloads weekly
	- does not execute functions
	- does not check types
	- subcommands cannot take options from the parent command
- Undici
	- http 1.1 client
	- fundamental design issues in the core http client that would be hard to fix without breaking compatibility
	- undici is an entirely new design
	- features:
		- keep alive by default
		- lifo scheduling
		- no pipelining by default
		- unlimited connections
		- can follow redirects (opt-in)
	- Mocking
		- Nock wont work, because assumes underlying node http core
		- undici has its own mocking through MockAgent and interceptors
	- Other features
		- async hooks
		- diagnostic channels
		- trace events
		- web crypto api
		- web streams api
		- wasi


Pacman mouth principle: leave room for people to join a grouo conversation (tierny)


## Cody zuschlang - Modern fullstack

- To build modern tech stacks, focus on the that 20% makes the hard part (non functional requirements)
- 1st principle: modularity wins (be able to swap stuff that don't work anymore)
- 2nd principle: DX matters
- 3rd: community first (open source projects with good communities)
- 4th: transparency (abstraction layers don't always work. We should be able to peel abstractions off if they don't work for us)
- the stack
	- Postgres (the elephant in the room, the best relational db)
	- Fastify
	- GraphQL (Mercurius, plugin for fastify. Caching built-in, it can even use redis as a backend, helps you to avoid the N+1 problem using data loaders)
	- URQL: Frontend graphql client with caching support (it can also normalise data and find the same items in different queries, avoiding unnecessary data fetches. Has hooks for React and devtools)
	- React (because it's react! evolving but stable and widely adopted. Use it with Vite for the best experience)
	- SSR: fastify-vite / fastify-dx


## Robin Ricard - Record and Tuple (immutable data structures) - @r_ricard

- references and mutability
	- two strings with the same content are equal
	- two objects with the same content are NOT equal
	- in strict mode you cannot mutate characters in a string (e.g. str[2] = 'c' will fail)
	- objects can be mutate, very common
	- You can Object.freeze if you want to avoid mutations
	- But it is shallow, sub objects are not frozen
	- const and Object.freeze are not deep guarantees
	- trick JSON.parse(JSON.stringify(obj)) - not good for cpu/memory
	- better alternative `structuredClone`
	- When using objects as keys for a Map, then you need to provide exactly the same reference to access the same key
	- object cycles are not serialisable, so that makes things more complicated
- Record and Tuple
	- record #{key: value}
	- tupe #[value1, value2, etc]
	- Can be combined
	- they are immutable by default (TypeError if you try to change them)
	- Deep immutability
	- Equality by value and not by reference
	- cycles cannot exist in records and tuples
	- You cannot put objects inside tuples and records (TypeError). Tradeoff to really keep them immutable
	- You cannot reference functions in a record or tuple (they are effectually objects) ... There are plans to support that though
	- tuple.push() does not exist (would be mutable), but you can use spread #[...tuple1, ...tuple2, ...etc]. Similarly with records: #{...record1, ...record2, ...etc}
	- tuple.reverse() does not exist, but there's tuple.toReversed() which makes a reversed COPY. Similarly there's .toSorted and .with (for assignment). Also toSpliced
	- All these methods will also be added to arrays (as part of another proposal, they should also be interoperable with tuples)
- State of the proposals
	- STAGE 2 at TC39
	- the feature is expected to ship "one day"
	- to get to stage 3:
		- needs review
		- (done) has a babel transform and a polyfill
		- (done) tests are already there
		- (done) Firefox implementation behind a compile-time flag
	- Provide feedback record-tuple-playground by Rick Button: https://rickbutton.github.io/record-tuple-playground
	- new array methods STAGE 3: shipping on the major engines (hopefully available in Node.js soon)


## James Snell - AbortController @jasnell

- Cloneable AbortSignal - creating and fixing a performance bug in Node fetch implementation
- there is always an abort signal created for a WritableStream. Creating that was very very slow
- the issue was related to structured cloning of abort signals (done to support worker threads and be able to send an abort signal across worker threads)
- AbortSignal are implemented as a JS class (which extends from EventTarget), this makes it hard to clone (structured clone does not work with classes)
- structured clones produces plain objects so you'll lose all the class based properties
- Node.js has a JSTransferable helper that helps here, but you need to extend it, and AbortSignal is already extendins EventTarget (JS does not support multiple inheritance)
- hacked a function called makeTransferable that manually sets prototypes with the chain: JSTransferable -> AbortSignal -> EventTarget
- this is slow because it creates C++ backed objects (doing it everytime an AbortSignal is created, everytime a stream is created, everytime a Request is created)
- fix: simple, stop doing it. AbortSignal is not cloneable by default anymore and there are ways to opt-in when needed.
- Fix landed by James son, Aaron which makes Node.js officially a multi-generational open source project!


## Jean Burellier - Heavy computation in Node.js using Rust @shepsheplu

- Addons are dynamically shared objects written in C++ (interface between JS and C++)
- In practice adds support for any other language that can produce C/C++ compatible libraries
- Writing native extensions requires knowledge of libuc, v8, node.js internals
- native extensions  need to be loaded with require
- problem: Node.js evolves and it breaks compatibility with existing native addons
- NaN - Native Abstractions for Node.js (added in 2015), creates a common interface on top of all versions of Node.js. It is more stable than the generci Node.js API
- Node-API (previously called N-API): Stable ABI (Application Binary Interface). Full abstraction of the JS engine
- Id done right, the fact that a module is native, should not impact consumers at all
- how to use Rust for creating native addons: napi-rs, node-bindgen, neon

```
cargo new nodeconf
```

Add node-bindge to dependencies and build dependencies

add

```
[lib]
crate-type = ["cdynlib"]
```

Define build.rs

```
fn main() {
	node_bindgen::build::configure()
}
```

build with

```
nj-cli build .
```

in node:

```
let add = require('./dist')
add.main()
```

- Good for performance oriented applications or for creating bridges between rust apps and node, access to lower level system apis or desktop apps
- examples at node-addon-examples: https://github.com/nodejs/node-addon-examples
- Downsides:
	- it's expensive to transfer data between JS and C layer
	- Do not try to rewrite everything as a native module


## Matteo Collina - I would never use an ORM
- MVC is a bit of an antipattern, it's better to structure stuff by features
- ORMs are generally good at the beggining, but they generate a lot of trouble later down the road
- You end up fighting against the ORM most of the time when trying to build more advanced features
- Platformatic tries to create a very flexible backend that can remove repetitive tasks but still gives you maximum flexibility https://oss.platformatic.dev
- Steps to use platformatic DB:
	- write schemas/migrations
	- apply migrations
	- Setup platformatic db
	- Write your custom sql
- Based on Fastify (plugins)

```
npm i -g platformatic
platformatic db init
# needs more deps to support dbs
```

Creates a  new migration and `platformatic.db.json`

```
platformatic db migrate
```

```
platformatic db # starts the db
```

Starts a graphiql and a swagger ui

Big demo


## Michere Riva - Lyra
- How does elastic search keep being performant even when dealing with billions of records
- Built lyra to learn how it is possible that elastic search is so good
- Some problems with elastic: Hard to setup, upgrade, big memory footprint, costly
- initially tried to build it in rust, then go then settled with JS
- Lyra is all open source - lyrajs.io
- runs everywhere (js, node, edge, serverless, workers, bun, deno, react native, etc)
- very fast (microseconds to query over a million records dataset)
- supports stemming in multiple languages (23 right now)
- You can build your own stemmers
- goal is to empower people to learn more about search engines and their algorithms
- Supports plugins (e.g. data persistace)
- can be used as a static asset behind a CDN like cloudfront (which you can update with a batch process if you have to)
- contribute and join the slack lyrasearch.slack.com


## Liran Tal 

# day 2

## Ethan Arrowood @ArrowoodTech - Making fetch happen
- fetch really starts with ajax (XHR spec) in 2005 (browsers really need to talk with servers somehow)
- 2013 Service Workers - programmable network proxy for browsers
- fetch:
	- modern async patterns
	- class based networking interfaces
	- based on CORS standard
- ~2015-2017 URL + Stream + Abort Controller standards
- isomorphic JavaScript
- so many JS environments...
- Node.js: http module in 0.3.6
- many 3rd party http libs with different tradeoffs (simplicity vs control - got vs undici - vs familiarity, fetch)
- fetch experimental in node.js v18 (after years of work)
- more WHATWG APIs coming / many added but still experimental (WebStreams, structuredClone, FormData, Web Crypto)
- WinterCG (web interoperable runtimes community group): trying to standardise a minimum js API that works everywhere so we can have interoperable apps across runtimes
- fetch is currently being discussed in WinterCG

## Liran Tal - Char wars (traversal attacks)
- path traversal (directory traversal): user supplied filename might escape the current dir and acces files in parent folders. Happens because of lack of (or insufficient) validation
- not in the OWASP top 10.
- it can be used to steal password or keys or to look into dependencies to find other attack vectors
- Zimbra (mail provider) was vulnerable to this attack
- Apache 2.4.49 had this vuln last year (critical). Apparently there was a previous attempt to fix this which wasn't fully successful
- many path traversal attacks can be found easily (even on twitter)
- st package vulnerability analysis (not really vulnerable to simple attacks)
- URL Encoding can be used to trick st though
- fix is better normalisation and use the api in the right order (decode uri first, than normalisation then additional validation)
- in Node.js you can use path.relative()
- Path traversal also in a VS Code extension: a 0 click exploit that leverages a vulnerable local file server (spawn by the vscode extension) to fetch local files from users  (after they click a link)
- Who checks VSCode extension? supply chain security does not even consider this ecosystem
- Path traversal in Node.js
	- simple vulnerable express example
- dotdotpwn: fuzzy test tool that looks for path traversal attacks (exists since 2011)
- snyk vscode extension for in-editor analysis

## Danielle Adams - life and times of a Node.js release - @adamzdanielle (AWS Amplify)
- releasers are not responsible for writing the code that goes into the release but they make sure that everything is good (changes, testing, etc)
- 3 types of releases:
	- Current (in sync with `main` branch)
	- Active LTS (more stable and tested release. Might lack newer features)
	- Maintenance LTS (no more commits from the main branch, only patches: bug fixes, security, etc.)
	- timeline of releases from pre-release to 36months after release
- ~100 commits per release
- maintaining multiple release lines using branches: release and staging
- important to respect semver to avoid unexpected breaking changes
- backport PR: when a release (from main to staging) breaks stuff is used to fix this kind of situations
- git bisect to find which commit broke stuff
- preparing the release: node custom tool to generate the release notes. A human edits that to highlight major changes
- CITGM: test suite that runs a new release against famous libraries (to look for regressions). Prevents disruption to the ecosystem
- every release is GPG signed
- releases are scheduled in advance (months). Release team needs to think ahead. Next releases are scheduled until 2026!!!
- Code names: Fermium, Gallium, etc (alphabetical order).

## Luke Golmquist - Lesser known Node.js modules
- @sienaluke
- "Sometimes tiny doors lead to big opportunities" - Lisa Simpson's cat
- Globals
	- exports
	- require
	-  module
	- \_\_filename
	- \_\_dirname
	- (not actually global, scoped to the current module - in commonJS)
- node: prefix
	- all core modules can be prefixed (e.g "node:fs")
	- explicit about where the module is coming from
	- newer modules will probably be prefix only, for instance, `node:test` is prefix only!
- corepack
	- experimental tools: needs to be enabled
	- supports yarn and pnpm
	- manages versions of yarn and pnpm
	- corepack prepare yarn@x.y.z --activate
- querystring
	- marked as legacy
	- use URLSearchParams
	- `qs` module is a well used alternative too (+71 mln downloads/week!)
- Builtins
	- repl.builtinModules (14.5.0)
	- console.table()
- CLI options
	- --pending-deprecation (early warnings for stuff that will be eventually deprecated)
	- --throw-deprecation (will actually throw an exception after the warning)
- VM module
	- vm.createContext (like eval but with a limited context)
	- NOT A SECURITY MECHANISM: DO NOT USE WITH UNTRUSTED CODE!
- punycode
	- char encoding for intl domains
	- (deprecated!)

## Miroslav - Hold my context @bajtos
- context propagation: 
- app context (e.g. correlation id for log aggregation, user credentials, user permissions)
- implicit context (things you can infer from the code, url paths, etc)
- java/.net: thread per request model (each request gets a thread, so you can use thread local storage to keep an isolate context)
- doesn't work in node.js because of the event loop that interleaves different tasks (across multiple requests) in one single thread
- a tentative approach was domains (2012-2015). You create a domain (like a scope), you run your request in that domain. Context is propagated
- continuation-local-storage was created from it
- it doesn't always work (context might be lost and be undefined generating crashes, sometimes you get incorrect context).
- solution: explicit context passing (like in go), no magic but a lot of work (and cannot propagate implicitly)
- what do we want:
	- a built-in api that solves this problem (many modules can use it consistently)
	- needs to support native promises
	- an API to restore a context (e.g. for a task or a connection pool library)
	- good performance
- 2013-2014 async liteners
	- abandoned
- 2015-2018 Async Wrap
	- undocumented low-level apis
- 2017-now Async hooks
	- Built-in (experimental)
	- matches all the other expectations
	- but not very easy to adopt
- Async Local Storage (2020-now)
	- public and stable, in core
	- easy to use
- @fastify/request-context plugin
- known issues:
	- tricky edge cases (you might still loose context)
	- some user modules are still catching up (e.g. pg-pool does not work with callbacks)
	- async-break-finder: module that helps you to find context propagation problems at runtime

## Marian & Santiago - Observable apps with open source software
- 3 pillars
	- metrics as dots
	- logs as lines
	- traces as traces
	- they create a full picture of your running system
- metrics: aggregation over time of infrastructure measurements
- logs: data emitted by different components (better if correlated)
- span: unit of work (measured over time)
	- building block of a trace (traces are composed of one or more spans)
- Context propagation (storing state across the lifespan of a transaction)
- Async hooks!!!
- trace context specification by the W3C
- traces are essential for a distributed system
- instrumentation (art of adding observability)
- can be automatic or manual
- open telemetry provides a lot of tooling and standard practices (API, SDK, OTLP, COllector, semanting conventions)
- DEMO
	- example that allows to compare traces (slow and fast request) with jaegar
	- checking event loop utilization of the main thread using prometheus (looking for stuff that can block the event loop)
- Takeaways
	- good observability with open telemetry is easy
	- observe individual requests and recurring patterns
	- helps to narrow down the research of specific issues
	- don't overdo with instrumentations because it's not entirely free (so it might hurt performance)


## Paolo Fragomeni - Peer to Peer is the next step for cloud computing @heapwolf
- cloud: "landlord/tenant relationship" ("devops is becoming INSANE")
- pushing from the cloud to p2p (the end of cloud)
- history of p2p
- AWS vs ubiquitus computing (3mln servers vs 15bln mobile devices)
- every device generates data -> data proliferation (e.g. "self driving cars generating 10gb/hour")
- cost & complexity (cloud vs p2p): cloud gets more expensive & complex, p2p gets cheaper and doesn't get more complex
- socket supply company (cloud and p2p tools): allowing apps to communicate directly with each other (no servers)
- modern app runtime (socket), runs signed local code
- `ssc build .` to build a project which is going to be a single binary app (similar to electro or tauri or phonegap)
- it also offers libraries to create p2p swarms
- nat-traversal-spec repo made public (LIVE)
- sockets.sh
- inspired by and mostly compatible with Node.js
- sandboxing built in
- socketsupply/socket runtime (C++) made public too

## tensorflow.js workshop
- Make a smart camera using a pre-trained TensorFlow.js Machine Learning model


# Day 3

## Gil Tayar - Types proposal to ES (TC39) @giltayar
- "If you use TS, you probably wish you did not have to transpile!"
- tc39 on github github.com/tc39
- process for new additions to the spec:
	- stage 0: to be proposed
	- stage 1: accepted for consideration
	- stage 2: spec almost complete
	- stage 3: waiting for implementation
	- stage 4: implemented!
- type annotations to js, they are treated like comments
- the syntax for typing annotations is intentionally loose
- the proposal suggests also `type` keywords
- same syntax as TS
- requires third party type checkers to take advantages of type annotations
- why adding types at all?
	- It's a complex feature, so it needs good reasons
	- most js devs use ts
	- static typing was the most requested feature
	- people want types in js
- We have typescript, why do we need something else?
	- we have forgotten our roots in scripting: "write code, run it"
	- we have become addicted to tooling
	- but tooling adds complexity
- JS needs to be backward compatible (idea "pragma types;")
- runtimes could use these annotations to optimize code execution, but some people disagree that's the case (type annotations don't give any strong guarantee on runtime types after all)
- challenges:
	- how do we find type annotations (where does it start, where does it end)
- Lots of typescript stuff is not going to be in the spec
- It will take years, but it will be worth the wait

## Paolo Insogna - ddos attacks in Node.js @p_insogna
- Web apps are important and must not go down!
- (D)DoS: make a service unavailable by sending a lot of traffic and slowing it down
- slowloris attack
	- DDoS attack with minimal bandwidth
	- Created by Robert RSnake Hansen in 2009
	- it's not just HTTP (it can be used on any connection based protocol)
- Normal HTTP activity:
	- SIMPLE MENTAL MODE: clients: connect/make request/disconnect
	- the server needs to allocate some RAM for every client
	- in normal cases the amount of connected clients remains constant (with clients coming and going) so does the memory
	- retaining a socket is expensive
		- OS level
		- file descriptors
		- the app only manages things at high level but it might manage more data per every connection
- slowloris:
	- clients never disconnect!
	- they intentionally send data very very slowly (e.g. 1byte per minute)
	- avoids timeouts but retains memory for a long time
	- with enough clients this becomes very memory intensive for the server and it might eventually exhaust all its resources
- how do we stop it?
	- never put a node.js in front! Use reverse proxies 
	- mitigation strategies on the software
		- it's hard to distinguish between a slowloris attack and a genuinely slow client
		- but if you could, you can decide to terminate slowloris connections
		- you could limit number of connections per IP, it can help but not a definitive solution. It's easy for attackers to get more IPs
- What do we do with Node.js?
	- First attack in 2009
	- Node.js partially addressed this in 2018
	- `http.Server.headersTimeout` (time needed to parse headers)
	- the mitigation was assuming a good connection will send at least 1 byte every 2 minutes (by default) - not very effective
- Node 13 even disabled this mitigation by default to allow long-lived connection
- `http.Server.requestTimeout` (time needed to parse the entire request, including the body)
	- Disabled by default
	- setting timers on node is very expensive (`setTimeout` allocates resources)
	- the user needs to explicitly enable this defense mechanism
- Are we safe now?
	- YES ALMOST!
	- loose countermeasures
- How to protect Node.js 16 (and below)
	- Make sure sockets cannot be idle using `http.Server.timeout`
	- code example in pic
- Node.js 18 is finally safe by default
	- the values from the snippets are now the default
	- there is a new API `connectionsCheckingInterval` (one interval for ALL the connections)
	- these changes improved performance by 2%
	- It's a BC so it's not going to land in 16
- Take home lessons:
	- security always (FIRST)
	- even if that means sacrificing performance
	- validate your stance regularly
- "Never assume the obvious is true"


## Steve Husak - Best Practices at an Enterprise Level @shusak
- classes of attacks on npm/js
	- cryptojacking (code inserted to do mining)
	- typosquatting (fake packages with similar names to famous packages to install malicious code)
	- dependency confusion: looking for a private package, but pulling a public one with the same name
	- data harvesting
	- ransomware
	- hijacking
	- author's own attack
- supply chain attack / substitution attack
	- attacker publishes publicly a bumped version of a private package. npm will look for updates in both places and ends up installing the fake public package
- use SBOMs (Software Bill Of Materials)
- internal policies to decide which versions of Node.js can be used at CapitalOne
	- consider only LTS versions and prefer the active over the maintenance
	- never use current
- use golden images for docker
	- no known vulnerabilities (at build time)
	- consider distroless images to decrease vulnerability surface
- Scan dependencies with: Mend, Snyk, Socket
- evaluate packages by looking at release cadence, issues, maintainability
- you need to have tooling for all this stuff
- but to use tooling you need to have GOOD TESTS
- OpenSSF best practices guide: https://openssf.org/blog/2022/09/01/npm-best-practices-for-the-supply-chain/
- https://openssf.org/resources/guides/

## Vitaly Aminev - serving data to thousand devices @v_aminev
- Sport events app: very spiky traffic kind of app
- how to build scalable apps?
- Application runtime:
	- Node.js is great when you are not doing CPU bound tasks
	- good ecosystem
- "Last mile" transport
	- how do you deliver content (RESTful, live updates through websockets, gRPC)?
- gRPC:
	- high throughput bi-directional streaming
	- low latency over HTTP/2
	- protobuf
	- language agnostic
- Interservice communication
	- message brokers! Specifically RabbitMQ
- Load balancing
	- don't expose your Node.js apps directly
	- entrypoint into your backend
	- using nginx initially, lack of good gRPC support
	- mesh might be tricky
	- used linkerd, which supports gRPC
- Databases
	- you could start with SQLite in most cases
	- you need to understand what's the "load profile" (read and write)
	- you could often rely on memory databases like Redis
- how do you test if it scales?
	- you don't and trust your design (don't do that!)
	- instrumentation and review and optimize over time (it will work only if the traffic grows slowly over time so you have time for fixing and scaling things as needed)
	- LOAD TESTS!
- Load testing:
	- load producers (autocannong, pandoea, pgbench, etc). They need to saturate the systems to get interesting numbers
	- initially test everything separately
	- then test all together
	- You need to determine the baseline to figure out if a change had any effect
- benchmarks
	- don't trust anything you see on the web, do your own tests
	- don't trust dev machines, do them in the cloud!

## Yagiz Nizipli - Road to a fast url parser in Node.js
- Current bottleneks in Node.js
	- Node use v8: whenever you call a function that calls C++ you use a bridge which needs to go through serialization and deserialization (and that slows thing down)
- URLParser
	- most important functionality for a web server
	- runs all the time (especially in benchmarks generating many many requests)
	- it's a complex component and people feat changing it
	- Correct URLParsing needs a complex state machine
	- The spec is very detailed and describes a process for parsing correctly: https://www.rfc-editor.org/rfc/rfc1738
- It should be possible to implement a url parser in web assembly or try a new pure JavaScript implementation (current implementation uses C++)
	- using rust and compiling to webassembly
	- undici does that behind the scene
	- downsides
		- decoding utf16 to utf8 is costly
		- you need to deal with shared array buffers
		- text encoding/decoding in Node.js relies on C++
- the first implementation in Rust was 86% slower than the current one
- `url-state-machine` library (pure JS re-implementation)
	- ~600% faster than the previous Node.js implementation
- Interesting changes
	- using function vs class (55% faster) - this disparity is fixed in v8 9.7
	- using if vs switch (switch seems slow). In reality if you use more than 10 branches, switch is faster than if
	- String vs number comparison: numbers are faster to compare
	- .slice vs .substring (slice seems faster for relatively short strings)
	- isNan vs undefined (isNaN is expensive, sometimes you can just check for undefined which is faster)
	- memoization: trade memory for cpu
- statistics:
	- 99.9% compliant (1 test failing)
	- 95.88% of coverage (implementing the official test suite)
- performance:
	- still limited by v8

## Zbyszek Tenerowicz - Eval all the strings! - Hardened JavaScript @naugtor
- npm packages are just random string of texts wrapped in a tar.gz
- but... SUPPLY CHAIN SECURITY
- `npm-audit-resolver` & `can-i-ignore-scripts` / `@lavamoat/ignore-scripts`
- dangers of running someone else code:
	- exfiltration (e.g. process.env piped to fetch)
	- remote code execution
	- file system access
- isolation (if we want to execute something we don't trust in an isolated environment)
- Alternative use an alternative to `eval` . This allows us to pass a very specific context to eval (and block access to anything else)
	- uses `with` to take an object and explode it into the global scope
	- scopeguard a proxy object to stop the code to try to access global variables
- lockdown freezed object, array, promises and mocks defineProperty
- SES (secure eval string) is a tc39 proposal
- object capability tries to add capabilities (authority) to certain objects, rather than using a global identity
	- you create an obj (e.g. a linter) and you pass it only the functions it can use (e.g. readFile)
- @endo a runtime that can isolate stuff
- lavamoat: convenience to use SES
- lavamoat can generate a policy based on the apis that a given package currently uses. If you upgrate that package and there are new requirements, lavamot will stop the execution and ask you to review and update the policy

## Kassian Wren - Augmenting Node.js with WebAssembly  @nodebotanist
- webassembly (the tech) allows you to compile some code into a bytecode format
- it's not web (runs in node, iot, dbs, etc) nor is assembly (it does not target a specific chip)
- webassembly is also the specification for the bytecode
- to avoid confusion:
	- wasm the bytecode spec
	- webassembly the tech
- WASI: web assembly system interface for communicating with the underlying system (e.g. even to do console.log because console is system specific)
- WAT: web assembly text format
- why?
	- heavy computation
	- avoid compiling native code (over and over!)
	- wasm can be used for sandboxing (containerless alternative)


## Big themes
- Security
- Performance
- RUST!
