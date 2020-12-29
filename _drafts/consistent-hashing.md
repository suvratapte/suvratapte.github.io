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

We need to find a strategy will not directly depend on the _number of nodes_ that we have.

## Consistent Hashing









