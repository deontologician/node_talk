# RethinkDB
### polling: there is a cure

----

### RethinkDB is the first open-source scalable database for the realtime web

- JSON document store <!-- .element: class="fragment" -->
- Queries in your language, not in string literals/json <!-- .element: class="fragment" -->
- AGPLv3: not going anywhere! <!-- .element: class="fragment" -->

----

## Scalability built in

- Sharding so you can scale queries to more machines <!-- .element: class="fragment" -->
- Replication so your data is never lost <!-- .element: class="fragment" -->
- Queries are automatically converted to map/reduce so you can effortlessly scale <!-- .element: class="fragment" -->

----

## The realtime web

You've seen first hand how the realtime web works

"Push, don't Poll"

----

## Web browsers have it solved

- Server Sent Events (SSE)
- WebSockets

----

## Web application servers have it solved

- Node.js and Go were designed with realtime in mind
- Tornado and Twisted in Python  <!-- .element: class="fragment" -->
- EventMachine in Ruby  <!-- .element: class="fragment" -->
- Pretty much every language now has a push web framework <!-- .element: class="fragment" -->

----

## The database is the final piece

Can we end polling in our lifetimes? <!-- .element: class="fragment" -->

----

## Adapting databases to the realtime web presents a huge challenge.

Ever tried creating 1000 persistent connections to your DB? <!-- .element: class="fragment" -->

----

## RethinkDB is the first database designed from the ground up for the realtime web.

----

## Because of this we make building realtime apps dramatically easier.

----

## Where do web servers like Node get their scalability?

----

## #1: The kernel

- `epoll` on linux, `kqueue` on BSD/Mac
- These interfaces let you keep thousands of sockets open and poll them in in O(1) time, instead of O(n)

----

## #2: Coroutines instead of threads

This one takes a bit more explaining.  <!-- .element: class="fragment" -->

----

## The old model for web servers

Open a thread or process for each incoming request

----

### What threads and processes are good for.

Threads simplify your code if you're doing a lot of CPU heavy tasks.

You don't need to worry about being fair to other jobs, the OS does it for you.<!-- .element: class="fragment" -->

----

### The cost of processes and threads

Each process gets a complete copy of the memory space

Threads use much less memory, but...

----

### Switching between threads is slow

The OS has to do task switching:

  1. You need to jump to kernel space (small cost) <!-- .element: class="fragment" -->
  2. The kernel needs to decide which thread/process to run next (big cost) <!-- .element: class="fragment" -->
  3. The kernel needs to jump back to your process (small cost) <!-- .element: class="fragment" -->

----

### In a web app

Most time is spent waiting for I/O, not doing cpu heavy computations

Ever wonder how "slow" languages like Ruby and Python got so big on the web? <!-- .element: class="fragment" -->

This is how they get away with it <!-- .element: class="fragment" -->

----

### Threads and processes don't buy us much in web apps

We don't spend much time in the app itself using cpu <!-- .element: class="fragment" -->

Instead we spend most of our time reading and writing on sockets <!-- .element: class="fragment" -->

----

## Coroutines avoid paying context switch costs

No jumping to the kernel and back, we switch ourselves.

Our cpu usage is light, so it isn't long before we will let another coroutine run. <!-- .element: class="fragment" -->

<span>This is called **cooperative multitasking**</span> <!-- .element: class="fragment" -->

----

## Side benefit of cooperative multitasking

No locks/mutexes

You just don't yield control until you're ready. <!-- .element: class="fragment" -->

Nobody will pre-empt you. <!-- .element: class="fragment" -->

----

## epoll/kqueue + coroutines

One process listens on the epoll/kqueue socket

<span>Called an **io loop** or **reactor**</span> <!-- .element: class="fragment" -->

It reads data from the socket, then passes it to the right coroutine <!-- .element: class="fragment" -->

When the coroutine completes, go back to waiting on the socket <!-- .element: class="fragment" -->

----

## The critical part:

No busy looping. No polling.

There's a flurry of activity when data comes in, but otherwise we're sitting around waiting. <!-- .element: class="fragment" -->

This is "Push, don't Poll" <!-- .element: class="fragment" -->

----

## So coroutines work great for web apps but...

Are they good for databases?

It depends <!-- .element: class="fragment" -->

----

## Coroutines in the database

Often aren't worth the trouble.

Queries are commonly CPU bound, which is where cooperative multitasking isn't much benefit. <!-- .element: class="fragment" -->

Most databases use a process or thread per connection. <!-- .element: class="fragment" -->

----

## Why RethinkDB is different

Coroutines are very useful in the case where there are many mostly idle connections listening for changes. <!-- .element: class="fragment" -->

This is how changefeeds work  <!-- .element: class="fragment" -->

We can keep thousands of database connections open and push changes to them <!-- .element: class="fragment" -->

----

## Coroutines let us do things other databases can't.

This isn't something you can bolt on afterwards. <!-- .element: class="fragment" -->

Cooperative multitasking has to be integrated into every part of the database. <!-- .element: class="fragment" -->

----

## The bottom line

Connections are cheap in RethinkDB.

Make as many as you need.

----

## Switching gears

Why is the database in the best place to push changes to your app?

----

## You're already storing incoming events there

When a user makes a change, the database is where you store it

The database can push changes to other users notifying them of the change

----

## The database knows how your queries work

It's already parsed the queries and turned them into its native data structures in memory

----

## It knows what parts of the query depend on what data

Some parts of a query are constant and won't change depending on changes in the data.  <!-- .element: class="fragment" -->

Some parts of a query can only change in specific ways we can optimize for. <!-- .element: class="fragment" -->

----

## It knows where the data you're watching is located

We only listen to shards that actually hold your data.  <!-- .element: class="fragment" -->

----

## The event-driven stack

Changefeeds enable a front-to-back push architecture.  <!-- .element: class="fragment" -->

Write your queries once, get the snapshot of the data you want, and then get updates as they happen in real time.  <!-- .element: class="fragment" -->

----

### RethinkDB is the first open-source scalable database for the realtime web

### It makes building realtime app dramatically easier <!-- .element: class="fragment" -->

----

## Learn more
- Check out [rethinkdb.com](http://rethinkdb.com)
- Download the product and try it out.
  - It's available on most platforms
  - very easy to get started
- Ask us questions and give us feedback [@rethinkdb](http://twitter.com/rethinkdb) on twitter

----

# Questions?
