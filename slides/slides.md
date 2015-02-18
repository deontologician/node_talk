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

![shard and replicate](images/1.16-reconfigure.gif)



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

- We want to take maximum advantage of Node's event-driven architecture.<!-- .element: class="fragment" -->
- Node.js webserver talking to a client over websockets:<!-- .element: class="fragment" -->

![websocket_image]() <!-- .element: class="fragment" -->


## This is great

- As Node becomes aware of relevant events, it pushes them to the clients that need them <!-- .element: class="fragment" -->
- Without threads, we can have thousands of connections simultaneously <!-- .element: class="fragment" -->
- Since everything is push, no unnecessary work is going on <!-- .element: class="fragment" -->


## Well, that's not entirely true

- How does Node find out about relevant events? <!-- .element: class="fragment" data-fragment-index="1" -->
- It polls the database of course! <!-- .element: class="fragment" data-fragment-index="2" -->

![database_polling_image]() <!-- .element: class="fragment" data-fragment-index="2" -->


## Ha. No really, nobody does that

Of course not. <!-- .element: class="fragment" data-fragment-index="1" -->

We don't want Node up doing a bunch of busywork. <!-- .element: class="fragment" data-fragment-index="2" -->

Busywork means less requests handled. <!-- .element: class="fragment" data-fragment-index="2" -->


## What we need is a message queue!

A push solution like a message queue will wake Node up only when
there's an event to be processed: <!-- .element: class="fragment" data-fragment-index="3" -->

![message_queue_image]() <!-- .element: class="fragment" data-fragment-index="3"-->


## How do we know what to send to Node?

Simple: just send everything.

Let the application sort out what's relevant. <!-- .element: class="fragment" -->

I call this the firehose strategy: <!-- .element: class="fragment" -->

![node_firehose_image]() <!-- .element: class="fragment" -->


## This is almost as bad as polling

Instead of asking for events when nothing is happening, we're
processing tons of events that aren't relevant.

![node_sad_face]() <!-- .element: class="fragment" -->


## Time to get smart

- We can set up different queues for different kinds of events... <!-- .element: class="fragment" -->
- We can subscribe to topics... <!-- .element: class="fragment" -->
- We can divide all our queries up into categories... <!-- .element: class="fragment" -->

![MQ_complicated_image]() <!-- .element: class="fragment" -->


## Before we do that, let's zoom out a bit more

Who's the source of all these events, and what are they doing?

![change_source]()


## The sources of the changes could be anything

- Could be Node itself, getting events from other users <!-- .element: class="fragment" -->
- Could be webhooks from external APIs <!-- .element: class="fragment" -->
- Could be the collar on a shark somewhere in the Pacific... <!-- .element: class="fragment" -->

The point being, we can't pass the buck any more: there's no-one more qualified to decide what events are relevant than the application <!-- .element: class="fragment" -->


## Well, almost...

Aren't we sending all these events to the database anyway? <!-- .element: class="fragment" -->

Can the database just tell us when something happens that we care about? <!-- .element: class="fragment" -->

Yes. Yes it can. <!-- .element: class="fragment" -->

![database_changefeed_image]() <!-- .element: class="fragment" -->


## Why is the database the best place?

You already wrote those queries telling it what was relevant:
<!-- .element: class="fragment" data-fragment-index="1" -->

```js
r.table('players').orderBy({index: 'score_desc'}).limit(5)
```
<!-- .element: class="fragment" data-fragment-index="1" -->

Just tack ".changes()" to the end:
<!-- .element: class="fragment" data-fragment-index="2" -->

```js
r.table('players').orderBy({index: 'score_desc'}).limit(5).changes()
```
<!-- .element: class="fragment" data-fragment-index="2" -->

Now whenever the results of this query change, they'll be pushed to us in realtime
<!-- .element: class="fragment" data-fragment-index="3"-->



## Changefeeds in RethinkDB



## Changefeeds in RethinkDB
