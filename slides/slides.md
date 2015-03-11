# RethinkDB:
### Bringing the lessons of Node to the database

----

### RethinkDB is the first open-source scalable database for the realtime web

----

## We're open source

- Open sourced AGPL 2 years ago
- Over 100 contributors to the project
- Small team of core developers in Mountain View

----

## We're scalable

- Sharding so you can scale queries to more machines <!-- .element: class="fragment" -->
- Replication so your data is never lost <!-- .element: class="fragment" -->
- Queries are automatically converted to map/reduce so no changes are needed when you scale up <!-- .element: class="fragment" -->

----

## So, what is the realtime web?

<video data-autoplay class="fragment" src="movies/rta.webm">

----

## Slack

<img src="images/slack_rta.png">

----

## Where do we need the realtime web?

- modern marketplaces  <!-- .element: class="fragment" -->
- streaming analytics apps  <!-- .element: class="fragment" -->
- multiplayer games  <!-- .element: class="fragment" -->
- collaborative web and mobile apps.  <!-- .element: class="fragment" -->

All of these applications require sending data directly to the client in realtime. <!-- .element: class="fragment" -->

----

## Web browsers have it solved

- Long-polling
- WebSockets <!-- .element: class="fragment" -->

----

## Web application servers have it solved

- Node.js was designed with realtime in mind
- Other languages have have bolted these features on, but they work  <!-- .element: class="fragment" -->

----

## What about the database?

Adapting databases to the realtime web still presents a huge challenge.

RethinkDB is the first database designed from the ground up for the realtime web.  <!-- .element: class="fragment" -->

We make building these kinds of apps dramatically easier.  <!-- .element: class="fragment" -->

----

## A bit about RethinkDB

----

## JSON document database
- Lock-free MVCC <!-- .element: class="fragment" -->
- Distributed joins <!-- .element: class="fragment" -->
- Queries automatically distributed across cluster <!-- .element: class="fragment" -->
- Built-in web admin interface <!-- .element: class="fragment" -->

----

## Easy sharding and replication

<video data-autoplay src="movies/reconfigure.webm"></video>

----

## Intuitive query language

- Official drivers for Node, Ruby & Python <!-- .element: class="fragment" data-fragment-index="1" -->
- Excellent community drivers for Go, PHP, Clojure and more <!-- .element: class="fragment" data-fragment-index="2"-->
- Integrated directly into the host language <!-- .element: class="fragment" data-fragment-index="3" -->

### Node.js:
<!-- .element: class="fragment" data-fragment-index="4" -->

```js
r.table('foo').filter(function(row){ return row('val').gt(23) })
```
<!-- .element: class="fragment" data-fragment-index="4" -->

### Python:
<!-- .element: class="fragment" data-fragment-index="4" -->

```py
r.table('foo').filter(lambda row: row['val'] > 23)
```
<!-- .element: class="fragment" data-fragment-index="4" -->

### Ruby:
<!-- .element: class="fragment" data-fragment-index="4" -->

```rb
r.table('foo').map{|row| row['val'] > 23}
```
<!-- .element: class="fragment" data-fragment-index="4" -->
----
<!-- .slide: data-transition="none" -->

## Demo

----

## Why Node.js is great for the realtime web

- Node is event driven

- Uses coroutines, not threads or processes <!-- .element: class="fragment" -->

- Designed for thousands of simultaneous connections <!-- .element: class="fragment" -->

----

## Event driven

Node.js was originally created with push capabilities in mind  <!-- .element: class="fragment" -->

When an event occurs, callbacks are run in response.  <!-- .element: class="fragment" -->

But if nothing is happening, Node is happy to quietly wait.  <!-- .element: class="fragment" -->

----

## Node uses coroutines

Coroutines are "cooperative multitasking". <!-- .element: class="fragment" -->

Unlike threads, coroutines don't need locks or mutexes to protect memory <!-- .element: class="fragment" -->

Unlike threads, you don't pay the cost of context switching back to the operating system between tasks. <!-- .element: class="fragment" -->

They're great if you do a lot of I/O and don't need a lot of CPU time  <!-- .element: class="fragment" -->

----

## Node is designed for thousands of connections

How do you have 10,000 connections to your web server open at once?

You can't use threads, it takes up too much memory! <!-- .element: class="fragment" -->

Co-routines solve this. <!-- .element: class="fragment" -->

Web servers do a ton of I/O, so they're a perfect match. <!-- .element: class="fragment" -->

----

### So, what makes RethinkDB useful for realtime?

----

## To answer that question, let's build a realtime app

The natural choice for the frontend is to talk to the browser over Websockets <!-- .element: class="fragment" data-fragment-index="2" -->

<img class="fragment movie_img" data-fragment-index="2" src="movies/01_last_frame.png">

----
<!-- .slide: data-transition="fade" -->

## This works great

As Node becomes aware of relevant events, it pushes them to the clients that need them <!-- .element: class="fragment" -->

Without threads, we can have thousands of connections simultaneously <!-- .element: class="fragment" -->

Since everything is push, no unnecessary work is going on <!-- .element: class="fragment" -->

<img class="movie_img" src="movies/01_last_frame.png">

----

<!-- .slide: data-transition="fade" -->

## Well, almost no unnecessary work

How does Node find out about relevant events?

<video data-autoplay src="movies/02_what_pushes.webm"></video>

----

<!-- .slide: data-transition="fade" -->

## Option 1: Polling the database

1. Query the database every 500ms
2. Calculate the diff
3. Push the results to the client
4. Optionally, push everything and have the client do the diff

<img class="movie_img" src="movies/02_last_frame.png">

----
<!-- .slide: data-transition="fade" -->

## Polling isn't great :/

It's slow, and it puts a massive load on the database and the web server as more clients connect.

We don't want the webserver doing a bunch of busywork <!-- .element: class="fragment" data-fragment-index="2" -->

Busywork means we can't handle as many request <!-- .element: class="fragment" data-fragment-index="3" -->

<img class="movie_img" src="movies/02_last_frame.png">

----

<!-- .slide: data-transition="fade" -->

## Option 2: Message queues

- A message queue will wake Node up only when there's an event to be
processed.

- This is a push architecture, and it ensures Node isn't doing busywork. <!-- .element: class="fragment" -->

<video data-autoplay src="movies/03_mq.webm"></video>

----

<!-- .slide: data-transition="fade" -->

## How do we know what to send to Node?

We could try just sending everything.

The app can sort out what's relevant. <!-- .element: class="fragment" -->

This is the "firehose strategy" <!-- .element: class="fragment" -->

<video data-autoplay src="movies/04_firehose.webm"></video>

----

<!-- .slide: data-transition="fade" -->

## And when we scale out?

Well, we have to send all events to all webservers

<video data-autoplay src="movies/05_scaled_firehose.webm"></video>

----

<!-- .slide: data-transition="fade" -->

## This is almost as bad as polling

Instead of asking for events when nothing is happening, we're
processing tons of events that aren't relevant.

<img class="movie_img" src="movies/05_last_frame.png">

----

<!-- .slide: data-transition="fade" -->

## Time to get smart?

- We can set up different queues for different kinds of events... <!-- .element: class="fragment" -->
- We can subscribe to topics... <!-- .element: class="fragment" -->
- We can divide all our queries up into categories... <!-- .element: class="fragment" -->

<video data-autoplay src="movies/06_smart_mq.webm"></video>

----

<!-- .slide: data-transition="fade" -->

## This is complicated

You can get this to be efficiently, but it's a lot of work

The solution will be different for every app <!-- .element: class="fragment" -->

As your app evolves, you'll need to add new queues to keep up <!-- .element: class="fragment" -->

<img class="movie_img" src="movies/06_last_frame.png">

----

<!-- .slide: data-transition="fade" -->

## But before we do all that

Let's zoom out a bit more.

Who's the source of all these events, and what are they doing? <!-- .element: class="fragment" -->

<video data-autoplay src="movies/07_change_sources.webm"></video>

----

<!-- .slide: data-transition="fade" -->

## Sources of events can be anything

- Could be the web app itself, getting input from users <!-- .element: class="fragment" -->
- Could be webhooks from external APIs <!-- .element: class="fragment" -->
- Could be the collar on a shark somewhere in the Pacific... <!-- .element: class="fragment" -->

<img class="movie_img" src="movies/07_last_frame.png">

----

<!-- .slide: data-transition="fade" -->

## Can we make the change sources smarter?

It's probably not a great idea.  <!-- .element: class="fragment" -->

Your shark collar shouldn't need to know how your entire app works.  <!-- .element: class="fragment" -->

<img class="movie_img" src="movies/07_last_frame.png">

----

<!-- .slide: data-transition="fade" -->

## Can the database do it?

Aren't we sending all these events to the database anyway? <!-- .element: class="fragment" -->

Can the database just tell us when something happens that we care about? <!-- .element: class="fragment" -->

<img class="movie_img" src="movies/07_last_frame.png">

----

<!-- .slide: data-transition="fade" -->

## Yes it can.

We call them changefeeds.

<video data-autoplay src="movies/08_changefeeds.webm"></video>

----

## Why is the database the best place to handle this?

- The database knows how your queries work  <!-- .element: class="fragment" -->
- It knows what parts of the query depend on what data  <!-- .element: class="fragment" -->
- It knows which shards the data is on <!-- .element: class="fragment" -->
- And it can save a bunch of processing time because of it <!-- .element: class="fragment" -->

----

## How you do it in RethinkDB

We already wrote a query describing what was relevant:
<!-- .element: class="fragment" data-fragment-index="1" -->

<pre data-fragment-index="1" class="highlight js fragment">
r.table('players').orderBy({index: r.desc('score')}).limit(5)
</pre>

Just tack ".changes()" on the end:
<!-- .element: class="fragment" data-fragment-index="2" -->

<pre data-fragment-index="2" class="highlight js fragment">
r.table('players').orderBy({index: r.desc('score')}).limit(5)<span data-fragment-index="2" class="fragment highlight-green">.changes()</span>
</pre>

Now whenever the results of this query change, they'll be pushed to us in realtime
<!-- .element: class="fragment" data-fragment-index="3"-->

----

## Using changefeeds demo

----

## Aren't DB connections expensive?

Normally, they are. <!-- .element: class="fragment" -->

Most databases use a process or a thread per connection. <!-- .element: class="fragment" -->

But RethinkDB uses coroutines for everything, so connections are cheap. <!-- .element: class="fragment" -->

----

## Coroutines in the database

Often aren't worth the trouble. <!-- .element: class="fragment" -->

Queries are commonly CPU bound, which is where cooperative multitasking isn't much benefit. <!-- .element: class="fragment" -->

But it is very useful in the case where we have many "mostly idle" connections listening for changes. <!-- .element: class="fragment" -->

----

## The event-driven stack

Changefeeds enable a front-to-back push architecture.  <!-- .element: class="fragment" -->

Write your queries once, get the snapshot of the data you want, and then get updates as they happen in real time.  <!-- .element: class="fragment" -->

----

## The future

- Right now, changefeeds work on the "map" step, but not the "reduce" step
- Our goal is to make all queries changefeedable <!-- .element: class="fragment" -->

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

### We're shipping 2.0 in a few weeks! Now is a great time to get started. <!-- .element: class="fragment" -->

----

# Questions?
