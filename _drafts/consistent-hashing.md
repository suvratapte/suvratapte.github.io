---
layout: post
title:  Consistent Hashing with Clojure
author: Suvrat Apte
date:   2020-12-28 14:39:40 +0530
categories: consistent-hashing, distributed-caches, clojure
comments: true
---

In this post, I have tried explaining what *Consistent Hashing* is and why it is
needed.

## Caching

Almost all applications today use some kind of caching. Caches are required to
save your databases from having to serve huge amount of requests. Caches are
usually used as follows:

**Insert path**: Information of user X is to be inserted in a database.

```
1. Update your database with user X data.
2. Populate your cache with the same data with cache key (say, user ID).
   (Optionally with some TTL)
```

**Retrieve path**: Information of user X is to be retrieved from a database.

```
1. Check if cache has data for user X. This will be checked with the
   cache key (user ID).
2a. If found, serve the data.
2b. If not found, fetch it from the database, populate the cache and
    serve the data.
```

**Update path**: Information of user X is to be updated in a database.
(Same as the insert path.)

**Delete path**: Information of user X is to be deleted from a database.

```
1. Invalidate cache entry for user X. This will be done with the cache
   key (user ID).
2. Delete data from the databse.
```

But as your application usage and data grows, your cache node is also going to
get overwhelmed and soon enough, you will need multiple cache nodes. When you
have multiple nodes, you will need to decide how you are going to divide data
between these nodes.

## Distributed Caching

One very simple strategy to divide data between cache nodes is to take hash
(integer) of the cache key and then take mod by the number of cache nodes.

For example, if hash of the cache key came out to be 86696499 and if we have 4
servers, then `(86696499 mod 4) = 3`. So the data should be in the node with
index 3 (0 based indexes).

This is very simple to implement. For simplicity, we will use names of users as
cache keys.

{% highlight clojure %}

(require '[clojure.pprint :as pp])

(def names ["Alice" "Bob" "Carry" "Daisy" "George" "Heidi"
            "Kate" "Mark" "Steve" "Tony" "Wendy" "Zack"])

(defn- get-node
  "Returns which node shall be used for the given `object` when `n`
  cache nodes are there."
  [n object]
  (-> object hash (mod n)))

(defn get-distribution
  "Returns distribution of `objects` over `n` nodes."
  [n objects]
  (reduce (fn [acc object]
            (let [node (get-node n object)]
              (update acc
                      (str "node-" node)
                      (fnil conj [])
                      object)))
          {}
          objects))

;; For printing maps in sorted order of keys
(->> names (get-distribution 4) (into (sorted-map)) pp/pprint)

{% endhighlight %}

Let's go through the code above.

`names` is just a collection of names.

`get-node` tells us which object should go on which node. This was the logic
that we discussed previously. We are first taking a hash and then taking a `mod n`
of that hash.

`get-distribution` just runs `get-node` on `names` and returns a map which has
node names as keys and corresponding values as list of keys which would reside
on those nodes.

The last line prints how `names` will be divided on cache nodes if we had 4
cache nodes. It prints the following map:

{% highlight clojure %}
{"node-0" ["Steve" "Zack"],
 "node-1" ["Alice" "George"],
 "node-2" ["Bob"],
 "node-3" ["Carry" "Daisy" "Heidi" "Kate" "Mark" "Tony" "Wendy"]}
{% endhighlight %}

Okay so this seems to be working well!  Now every time we have to fetch some
data, we will just check on which node that data is expected and then we will
try to fetch it from that node. So far so good!

So why do we need consistent hashing if mod n hashing is working well?

In today's distributed infrastructure, the total number of cache nodes can
easily change. There could be multiple reasons such as, needing a few extra
nodes when more traffic is expected, or a node going down due to some error.

We can simulate the above 2 situations.

Let's say we add another cache node in our cluster. We can simulate this by running:

{% highlight clojure %}
(->> names (get-distribution 5) (into (sorted-map)) pp/pprint)
{% endhighlight %}

We will get the following:

{% highlight clojure %}
{"node-0" ["George" "Mark"],
 "node-1" ["Kate"],
 "node-2" ["Carry" "Heidi" "Steve" "Zack"],
 "node-3" ["Bob"],
 "node-4" ["Alice" "Daisy" "Tony" "Wendy"]}
{% endhighlight %}

If we compare the distribution of keys for 4 nodes vs 5 nodes, we can see that
literally *all* the keys have different nodes now.

The same will happen if a node goes down. Let's simulate this by running:

{% highlight clojure %}
(->> names (get-distribution 3) (into (sorted-map)) pp/pprint)
{% endhighlight %}

This produces:

{% highlight clojure %}
{"node-0" ["Heidi" "Mark"],
 "node-1" ["Carry" "Daisy" "George" "Steve" "Tony" "Wendy"],
 "node-2" ["Alice" "Bob" "Kate" "Zack"]}
{% endhighlight %}

In this case as well, 10 out of 12 keys now have a different node.

Both these situations create a really bad situation for our databases. Our data
is going to be residing on 4 nodes initially. Once we add or remove nodes,
almost all of our cache keys are going to have different nodes. This is going to
result into a flurry of cache misses and all the missed requests will go to our
databases, creating a hotspot.

This is clearly undesirable and in extreme situations, this could even take our
entire system down.

Let's understand why this is happening. We need to remember that our logic to
select nodes (`get-node`) for data includes the _number of nodes_ as a
parameter. So when our _number of nodes_ changes, clearly the output of
`get-node` is most likely to change.

We need to find a strategy which will not directly depend on the _number of
nodes_ that we have.

## Consistent Hashing

TODO: Ref paper

Consistent hashing is a simple method of hashing which does not depend on the
number of nodes we have. In consistent hashing, we imagine our nodes to be
placed on a ring. The ring is made up of the range of our hash function. For
example, if our hash function produced hashes over the entire range of integers,
then the ring would go from the minimum integer to the maximum integer.

We will generate hashes for nodes using some property of nodes, say the IP
addresses. These will be the locations of our nodes on the ring.

<p align="center">
<img src="/resources/consistent-hashing-ring.png" style="height: 55%; width: 55%;">
</p>

To insert or retrieve data, we will hash the caching key and use the node which
is closest to the caching key hash in the clockwise (you can choose
anti-clockwise as well) direction.

<p align="center">
<img src="/resources/consistent-hashing-ring-key-hash.png" style="object-position: 70px 0; height: 75%; width: 75%;">
</p>

What benefit has this given us?

Well, so far it does not look like this is useful. In fact, we are doing more
work to find out which data goes to which node. We will now consider the 2 cases
that we discussed for mod n hashing.

Let's say we add a 5<sup>th</sup> node to our ring.

Since the 5<sup>th</sup> node got placed between the 1<sup>st</sup> and the 2<sup>nd</sup>
node, think about which keys will get re-allocated. Only the keys between
1<sup>st</sup> and 5<sup>th</sup> node will be
re-allocated to the 5<sup>th</sup> node. All the keys on the rest of the ring will remain
where they were.

<p align="center">
<img src="/resources/consistent-hashing-node-added.png" style="height: 60%; width: 60%;">
</p>

Similarly, if one of our nodes, say the 4<sup>th</sup> node, goes down; then
only the keys between the 3<sup>rd</sup> and 4<sup>th</sup> will get rebalanced.

<p align="center">
<img src="/resources/consistent-hashing-node-removed.png" style="object-position: -10px; height: 65%; width: 65%;">
</p>

This will reduce the number of cacahe misses by a huge amount and save our
databases from hotspots!

## Implementation in Clojure

We will write a simple and clean API for consistent hashing. This will include
functions to manage the ring: `create-ring`, `add-node`, `remove-node`. And like
before, we will have a `get-node` function which will tell us which node is to
be used.

{% highlight clojure %}

(defn create-ring
  "Creates a ring for `nodes`.

  `nodes` should be a coll of property of nodes which should be used
  for placing the nodes on the ring.
  Returns a ring data structure. It is a map which looks like this:

  {:node-hashes <a list containing hashes of `nodes` (order is preserved) >

   :hash->node <a map which is a look up table which gives node descriptor
                given a hash to a hash> }

  The return value of this function should be preserved by the caller as it is
  required as input to other functions in this API.
  "
  [nodes]
  (let [nodes (set nodes)
        node-hashes (->> nodes
                         (map hash)
                         sort)
        hash->node (reduce (fn [acc node]
                             (assoc acc
                                    (hash node)
                                    node))
                           {}
                           nodes)]
    {:node-hashes node-hashes
     :hash->node hash->node}))

{% endhighlight %}

We are taking the `set` of nodes to make sure there are no duplicate
entries. Then we get the hash values for all the nodes and sort them in
ascending order. This sorted list will be used to find the closest node in the
ring. We also create a lookup table `hash->node` which gives us the node
corresponding to a given hash. Finally, we return both of these so that they can
be passed to other functions in our API.

Let's see how `add-node` and `remove-node` work.

{% highlight clojure %}

(defn add-node
  "Adds `node` to existing `ring-state`.

  `ring-state` is expected to be a ring data structure (see doc string of
  `create-ring`.)

  `node` is expected to be a node descriptor (see doc string of `create-ring`).

  Returns a ring data structure (see doc string of `create-ring`).
  "
  [ring-state node]
  (let [current-nodes (-> ring-state :hash->node vals)]
    (-> current-nodes
        (conj node)
        create-ring)))

(defn remove-node
  "Removes `node` from existing `ring-state`.

  `ring-state` is expected to be a ring data structure (see doc string of
  `create-ring`.)

  `node` is expected to be a node descriptor (see doc string of `create-ring`).

  Returns a ring data structure (see doc string of `create-ring`).
  "
  [ring-state node]
  (let [current-nodes (-> ring-state :hash->node vals)]
    (->> current-nodes
         (remove #(= % node))
         create-ring)))

{% endhighlight %}

As you can see, these are very simple functions which just get `current-nodes`
from `ring-state` and then add or remove a node and create the ring again. This
works because our hash function is repeatable. So creating the ring again
generates the same hash values for existing nodes.

Let's look at the last function in our API, `get-node`.

{% highlight clojure %}

(defn get-node
  "Returns node to be used for cache-key `k`."
  [ring-state k]
  (let [node-hashes (:node-hashes ring-state)
        key-hash (hash k)
        closest-hash (or (->> node-hashes
                              (drop-while #(< % key-hash))
                              first)
                         (first node-hashes))]
    (get (:hash->node ring-state) closest-hash)))

{% endhighlight %}

The main logic in this function is to find the closest node to the cache-key, in
clockwise direction.

{% highlight clojure %}

(or (->> node-hashes
         (drop-while #(< % key-hash))
         first)
    (first node-hashes))

{% endhighlight %}

`node-hashes` is the sorted list of hashes of our nodes. We are dropping the
nodes which have hashes lesser than the hash of the cache-key. We will stop
dropping once we find a value greater than `key-hash`. This value will be the
hash of the closest node in clockwise direction. If `key-hash` is greater then
all the values in `node-hashes`, we wrap around and select `(first
node-hashes)`.

Now our API for consistent hashing is complete and ready for use!

Let's use it for the same example scenarios that we used for mod n hashing.
