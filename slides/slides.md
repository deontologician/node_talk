# RethinkDB:
### Bringing the lessons of Node to the database



## Node.js has shown that

### event driven = scalable



## RethinkDB:
### a database that has taken Node's lessons to heart



## What do I mean by that?

### &#x2713; We have v8 as a dependency. <!-- .element: class="fragment" -->



### OK, that's not actually what I was talking about



## About RethinkDB:



## JSON Document database
- Lock-free MVCC <!-- .element: class="fragment" -->
- Joins <!-- .element: class="fragment" -->
- Queries automatically distributed across cluster <!-- .element: class="fragment" -->
- Built-in web admin interface <!-- .element: class="fragment" -->



## Easy sharding and replication

<video data-autoplay src="movies/reconfigure.webm"></video>



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


<!-- .slide: data-transition="none" -->

## Here's how you get a table

<pre class="highlight js" data-trim>
> <span class="fragment highlight-current-green">r.table('players')</span>
</pre>

```js
{login: "KittyBot", score: 67}
{login: "coffeemug", score: 129}
{login: "mlucy", score: 140}
{login: "segphault", score:91}
{login: "JoeyZwicker", score:147}
{login: "VeXocide", score:120}
{login: "deontologician", score:125}
{login: "gchpaco", score:141}
{login: "AtnNn", score:129}
{login: "mglukhovsky", score:38}
{login: "timmaxw", score:43}
{login: "neumino", score:98}
{login: "Thinker-Bot", score:5}
{login: "Tryneus", score:148}
{login: "chipotle", score:189}
{login: "larkost", score:7}
{login: "thejsj", score:37}
{login: "danielmewes", score:104}
{login: "ahruygt", score:109}
{login: "jessskuo", score:198}
```


<!-- .slide: data-transition="none" -->

## Order the results by score

<pre class="highlight js">
> r.table('players')<span class="fragment highlight-current-green">.orderBy({index: 'score_desc'})</span>
</pre>

```js
{login: "jessskuo", score: 198}
{login: "chipotle", score: 189}
{login: "Tryneus", score: 148}
{login: "JoeyZwicker", score: 147}
{login: "gchpaco", score: 141}
{login: "mlucy", score: 140}
{login: "coffeemug", score: 129}
{login: "AtnNn", score: 129}
{login: "deontologician", score: 125}
{login: "VeXocide", score: 120}
{login: "ahruygt", score: 109}
{login: "danielmewes", score: 104}
{login: "neumino", score: 98}
{login: "segphault", score:91}
{login: "KittyBot", score: 67}
{login: "timmaxw", score: 43}
{login: "mglukhovsky", score: 38}
{login: "thejsj", score: 37}
{login: "larkost", score: 7}
{login: "Thinker-Bot", score:5}
```


<!-- .slide: data-transition="none" -->

## Now just the top 5

<pre class="highlight js">
> r.table('players').orderBy({index: 'score_desc'})<span class="fragment highlight-current-green">.limit(5)</span>
</pre>

```js
{login: "jessskuo", score: 198}
{login: "chipotle", score: 189}
{login: "Tryneus", score: 148}
{login: "JoeyZwicker", score: 147}
{login: "gchpaco", score: 141}
.
.
.
.
.
.
.
.
.
.
.
.
.
.
.
```



## Let's build an event-driven app

We want to take maximum advantage of Node's event-driven architecture.<!-- .element: class="fragment" data-fragment-index="1" -->

The natural choice is to talk to the browser over websockets <!-- .element: class="fragment" data-fragment-index="2" -->

<img class="fragment movie_img" data-fragment-index="2" src="movies/01_last_frame.png">
<!-- <video data-autoplay class="fragment" src="movies/01_websocket.webm"></video> -->


<!-- .slide: data-transition="fade" -->

## This is great

As Node becomes aware of relevant events, it pushes them to the clients that need them <!-- .element: class="fragment" -->

Without threads, we can have thousands of connections simultaneously <!-- .element: class="fragment" -->

Since everything is push, no unnecessary work is going on <!-- .element: class="fragment" -->

<img class="movie_img" src="movies/01_last_frame.png">


<!-- .slide: data-transition="fade" -->

## Well, that's not entirely true

How does Node find out about relevant events?

<video data-autoplay src="movies/02_what_pushes.webm"></video>


<!-- .slide: data-transition="fade" -->

## Polling the database is bad

That's much too slow. <!-- .element: class="fragment" data-fragment-index="1" -->

We don't want the webserver doing a bunch of busywork <!-- .element: class="fragment" data-fragment-index="2" -->

Busywork means we can handle less requests <!-- .element: class="fragment" data-fragment-index="3" -->

<img class="movie_img" src="movies/02_last_frame.png">


<!-- .slide: data-transition="fade" -->

## We need a message queue!

A message queue will wake Node up only when there's an event to be
processed

<video data-autoplay src="movies/03_mq.webm"></video>


<!-- .slide: data-transition="fade" -->

## How do we know what to send to Node?

Simple: just send everything.

Let the application sort out what's relevant. <!-- .element: class="fragment" -->

This is the "firehose strategy" <!-- .element: class="fragment" -->

<video data-autoplay src="movies/04_firehose.webm"></video>


<!-- .slide: data-transition="fade" -->

## And when we scale out?

Just send all events to all of your webservers

<video data-autoplay src="movies/05_scaled_firehose.webm"></video>


<!-- .slide: data-transition="fade" -->

## This is almost as bad as polling

Instead of asking for events when nothing is happening, we're
processing tons of events that aren't relevant.

<img class="movie_img" src="movies/05_last_frame.png">


<!-- .slide: data-transition="fade" -->

## Time to get smart

- We can set up different queues for different kinds of events... <!-- .element: class="fragment" -->
- We can subscribe to topics... <!-- .element: class="fragment" -->
- We can divide all our queries up into categories... <!-- .element: class="fragment" -->

<video data-autoplay src="movies/06_smart_mq.webm"></video>


<!-- .slide: data-transition="fade" -->

## But before we do all that

Let's zoom out a bit more.

Who's the source of all these events, and what are they doing? <!-- .element: class="fragment" -->

<video data-autoplay src="movies/07_change_sources.webm"></video>


<!-- .slide: data-transition="fade" -->

## Sources of change can be anything

- Could be Node itself, getting events from other users <!-- .element: class="fragment" -->
- Could be webhooks from external APIs <!-- .element: class="fragment" -->
- Could be the collar on a shark somewhere in the Pacific... <!-- .element: class="fragment" -->

<img class="movie_img" src="movies/07_last_frame.png">


<!-- .slide: data-transition="fade" -->

## Should we make the change emitters smarter?

No, we can't pass the buck any more.

There's no piece of the stack more qualified to decide what events are relevant than the web server<!-- .element: class="fragment" -->

<img class="movie_img" src="movies/07_last_frame.png">


<!-- .slide: data-transition="fade" -->

## Well, almost...

Aren't we sending all these events to the database anyway? <!-- .element: class="fragment" -->

Can the database just tell us when something happens that we care about? <!-- .element: class="fragment" -->

<img class="movie_img" src="movies/07_last_frame.png">


<!-- .slide: data-transition="fade" -->

## Yes. Yes it can.

<video data-autoplay src="movies/08_changefeeds.webm"></video>



## Why is the database the best place to handle this?

You already wrote those queries describing what was relevant:
<!-- .element: class="fragment" data-fragment-index="1" -->

<pre data-fragment-index="1" class="highlight js fragment">
r.table('players').orderBy({index: 'score_desc'}).limit(5)
</pre>

Just tack ".changes()" on the end:
<!-- .element: class="fragment" data-fragment-index="2" -->

<pre data-fragment-index="2" class="highlight js fragment">
r.table('players').orderBy({index: 'score_desc'}).limit(5)<span data-fragment-index="2" class="fragment highlight-current-green">.changes()</span>
</pre>

Now whenever the results of this query change, they'll be pushed to us in realtime
<!-- .element: class="fragment" data-fragment-index="3"-->



## Changefeeds in RethinkDB

You can watch changes on a table, or an individual document:
<!-- .element: class="fragment" data-fragment-index="1"-->

```js
r.table('foo').changes()
r.table('foo').get(documentId).changes()
```
<!-- .element: class="fragment" data-fragment-index="1"-->

You can watch for changes on transformations of documents
<!-- .element: class="fragment" data-fragment-index="2"-->

```js
r.table('foo').map(function(x){return x + 1}).changes()
```
<!-- .element: class="fragment" data-fragment-index="2"-->

You can watch for changes on filtered selections of documents
<!-- .element: class="fragment" data-fragment-index="3"-->

```js
r.table('foo').filter(r.row('name').contains('bob')).changes()
```
<!-- .element: class="fragment" data-fragment-index="3"-->

You can watch for changes on queries that are ordered and limited
<!-- .element: class="fragment" data-fragment-index="4"-->

```js
r.table('foo').orderBy({index: 'score'}).limit(10)
```
<!-- .element: class="fragment" data-fragment-index="4"-->



## Aren't DB connections expensive?

Normally, they are. <!-- .element: class="fragment" -->

Most databases use a process or a thread per connection. <!-- .element: class="fragment" -->

But RethinkDB uses coroutines for everything, so connections are cheap. <!-- .element: class="fragment" -->



## Database coroutines

Often aren't worth the trouble. <!-- .element: class="fragment" -->

Queries are commonly CPU bound, which is where cooperative multitasking isn't much benefit. <!-- .element: class="fragment" -->

But it is very useful in the case where we have many "mostly idle" connections listening for changes. <!-- .element: class="fragment" -->



## The event-driven stack

Changefeeds enable a front-to-back push architecture.  <!-- .element: class="fragment" -->

Write your queries once, get the snapshot of the data you want, and then get updates as they happen in real time.  <!-- .element: class="fragment" -->



## The future

- Right now, changefeeds don't work on queries requiring a "reduce" step
- Our goal is to make all queries changefeedable



# Questions?
