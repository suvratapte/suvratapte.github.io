---
layout: post
title:  nREPL Middleware
author: Suvrat Apte
date:   2020-07-31 18:37:40 +0530
categories: emacs
---

In this post, I will try to cover what <a href="https://nrepl.org/"
target="_blank">nREPL</a> is, what _nREPL Middleware_ are and why we need them. We will
also look at the middleware provided by <a href="https://github.com/clojure-emacs/cider-nrepl"
target="_blank">`cider-nrepl`</a>.

If you write Clojure, you are most probably using nREPL as that is the most popular REPL
out there. But it is not the only REPL out there.

## The Clojure REPL

Clojure itself has its own REPL. To start the default Clojure REPL, just run the `clojure` on
your terminal. You will see a REPL prompt. This is the default Clojure REPL.
<br>Try `(println "Hello, world!")` and you will see it works just like any other REPL.

But while typing that, if you try to autocomplete `println`, it would not work. Now if you
wanted to execute the same command again, you would usually use the up arrow key or
`control-p` to re-visit previously typed text. But when you do that, you will notice that
this REPL will just put a `^P` on the REPL prompt.

> This will work if you use the `clj` command, but it is just a wrapper over `clojrue`
> with `rlwrap`.

If you read `man clojure`, you will also notice that there is no easy way to start this
REPL on a socket. So if you are using this REPL, you cannot connect to it from a remote
machine.

So the default REPL clearly cannot be used as your daily REPL. That is where the need for
other types of REPLs comes in.

## nREPL

Network REPL (nREPL) is a huge improvement over the default Clojure REPL. nREPL has a
client server design. As the name suggests, an nREPL server can be started on a socket so
that remote clients can connect to it. It also gives you support for autocompletion symbol
lookups and many more things.

<!---excerpt-break-->

If you use Emacs, you can see the communication between the client and the server. To see
this, start a REPL in any of your projects. Then execute the following Elisp code:

{% highlight elisp %}
(setq nrepl-log-messages t)
{% endhighlight %}

> If you don't know how to run the above code, just press `M-:` (which runs
> `eval-expression`) in any buffer, paste the above code in the input minibuffer and press
> `RET`.

This will tell the nREPL client in Emacs to log the messages passed between the client and
the server. Now run `(println "Hello, world!")` in the REPL and then switch buffer with
this name:

`*nrepl-messages <projectname>:<host>:<port>*`

This buffer lists all the messages sent by the client and the replies sent by the
server. Messages starting with `-->` are from client to server. And those starting with
`<--` are from server to client.
<br>When we executed `(println "Hello, world!")`, the client sent this message:

{% highlight clojure %}
(-->
  id                                 "9"
  op                                 "eval"
  session                            "ed9decc8-200a-4a80-aec7-025050a8ff4b"
  time-stamp                         "2020-07-31 20:09:42.624467000"
  code                               "(println \"Hello, world!\")"
  column                             1
  file                               "*cider-repl workspace/cider-nrepl:localhost:57021(clj)*"
  line                               2
  nrepl.middleware.print/buffer-size 4096
  nrepl.middleware.print/options     (dict ...)
  nrepl.middleware.print/print       "cider.nrepl.pprint/pprint"
  nrepl.middleware.print/quota       1048576
  nrepl.middleware.print/stream?     "1"
  ns                                 "user"
)
{% endhighlight %}

> The messages buffer will contain someo ther messages as well. But we will focus on the
> messages that I've mentioned here.

So what does this tell us?

The most important field in any message sent by the client is the `op` field. This field
tells the server what operation is to be performed. Depending on the `op`, there will be
other fields that that particular `op` depends on.

In the above message, the `op` is `eval`. From `eval`, the server understands that it has
to evaluate some code. It expects the client to send the code to be evaluated in `code`
field. You can see that the `code` field contains the code which we had run: `"(println \"Hello, world!\")`.

Another important field in all the messages is the `id` field. The server will use the
same `id` in its replies to client requests. So to look at the replies from server, let's
use the `id` field (at your end, `id` will be a different number). Here are the replies
from the server:

{% highlight clojure %}
(<--
  id         "9"
  session    "ed9decc8-200a-4a80-aec7-025050a8ff4b"
  time-stamp "2020-07-31 20:09:43.156831000"
  out        "Hello, world!
"
)
(<--
  id         "9"
  session    "ed9decc8-200a-4a80-aec7-025050a8ff4b"
  time-stamp "2020-07-31 20:09:43.184731000"
  value      "nil"
)
{% endhighlight %}

> There will be another 3 messages with the same `id`, but those are not important for
> this discussion.

The first message tells the client that `"Hello, world!"` is to be printed on `out`
(`stdout`). The second message says that the return value of the executed expression was
`nil` (`println` returns `nil`).

This feature of being able to look at client server communication comes in handy at times.

Now that we have looked at basics of client server communication, we can jump to the
biggest feature that nREPL provides - support to add _middleware_.

## nREPL Middleware

nREPL Middleware gives you the ability to add your own middleware to support your own
operations. What this essentially means is that you can write code (which will run on the
server side) which can read requests from clients and then send appropriate responses.

So for example, if you wanted nREPL to show you <a href="https://clojuredocs.org/"
target="_blank">ClojureDocs</a>, you could write your own operation, say `clojure-docs`
which would take a `symbol` to be searched, as input (in the client message) and return
the doc in the server reply.

This is exactly how <a href="https://github.com/clojure-emacs/cider-nrepl"
target="_blank">`cider-nrepl`</a> adds a whole lot of functionality to
nREPL. `cider-nrepl` is a collection of middleware.

### cider-nrepl

If you are using `cider` in Emacs, you are already using `cider-nrepl` middleware.
Let's dig deeper into an operation provided by `cider-nrepl` - _autocomplete_.

If you type `print`, you will see autocompletions similar to this (depending on your autocomplete package):

<p align="center">
<img src="/resources/nREPL--autocompletion.jpg">
</p>
<p align="center">
<i>Cider autocompletion</i>
</p>

Let's look at the messages for this. To get completions for `print`, the client sends this message:

{% highlight clojure %}
(-->
  id                        "58"
  op                        "complete"
  session                   "ed9decc8-200a-4a80-aec7-025050a8ff4b"
  time-stamp                "2020-07-31 22:31:59.822565000"
  context                   ":same"
  enhanced-cljs-completion? "t"
  ns                        "user"
  prefix                    "print"
)
{% endhighlight %}

The `op` is `complete`, and `op` specific keys are `prefix` and `ns`. `prefix` is the text
to be completed and `ns` is the current namespace in the REPL. This is important because
completions are depedent on the current namespace.

Here is the message from the server:

{% highlight clojure %}
(<--
  id          "58"
  session     "ed9decc8-200a-4a80-aec7-025050a8ff4b"
  time-stamp  "2020-07-31 22:31:59.826961000"
  completions ((dict "candidate" "print" "ns" "clojure.core" "type" "function")
 (dict "candidate" "printf" "ns" "clojure.core" "type" "function")
 (dict "candidate" "println" "ns" "clojure.core" "type" "function")
 (dict "candidate" "print-dup" "ns" "clojure.core" "type" "var")
 (dict "candidate" "print-str" "ns" "clojure.core" "type" "function")
 (dict "candidate" "print-ctor" "ns" "clojure.core" "type" "function")
 (dict "candidate" "println-str" "ns" "clojure.core" "type" "function")
 (dict "candidate" "print-method" "ns" "clojure.core" "type" "var")
 (dict "candidate" "print-simple" "ns" "clojure.core" "type" "function"))
  status      ("done")
)
{% endhighlight %}

The server is providing detailed information about the completions in the `completions`
key. For every completion, it is telling what the completion text is, which namespace it
comes from and the type of completion candidate (`var` or `function`). Now if you look at the
screenshot above, you can see that `print-method` has a `<v>` at the end - signifying that
it is a `var`. This is based on the `type` information sent by the server.

These two messages give us a fair idea about how the operation must be implemented. So now
let's take an overview of the implementation.

### Overview of the Implementation

In `cider-nrepl`, the namespace <a
href="https://github.com/clojure-emacs/cider-nrepl/blob/8ea426e62b02a55f177a625361edbe132030f781/src/cider/nrepl/middleware.clj"
target="_blank">`cider.nrepl.middleware`</a> lists all the middleware and their order.

the namespace <a href="https://github.com/clojure-emacs/cider-nrepl/blob/8ea426e62b02a55f177a625361edbe132030f781/src/cider/nrepl.clj" target="_blank">`cider.nrepl`</a> describes all the middleware.

{% highlight clojure %}
(def-wrapper wrap-complete cider.nrepl.middleware.complete/handle-complete
  (cljs/requires-piggieback
   {:doc "Middleware providing completion support."
    :requires #{#'session}
    :handles {"complete"
              {:doc "Return a list of symbols matching the specified (partial) symbol."
               :requires {"ns" "The namespace is which to look for completions (falls back to *ns* if not specified)"
                          "prefix" "The prefix for completion candidates"
                          "session" "The current session"}
               :optional {"context" "Completion context for compliment."
                          "extra-metadata" "List of extra-metadata fields. Possible values: arglists, doc."}
               :returns {"completions" "A list of possible completions"}}
              "complete-doc"
              {:doc "Retrieve documentation suitable for display in completion popup"
               :requires {"ns" "The symbol's namespace"
                          "sym" "The symbol to lookup"}
               :returns {"completion-doc" "Symbol's documentation"}}
              "complete-flush-caches"
              {:doc "Forces the completion backend to repopulate all its caches"}}}))
{% endhighlight %}

<a href="https://github.com/clojure-emacs/cider-nrepl/blob/8ea426e62b02a55f177a625361edbe132030f781/src/cider/nrepl.clj#L57" target="_blank">`def-wrapper`</a> is a macro defined in the same file. The important thing here is the handler of this operation: `cider.nrepl.middleware.complete/handle-complete`. Let's look at this <a href="https://github.com/clojure-emacs/cider-nrepl/blob/8ea426e62b02a55f177a625361edbe132030f781/src/cider/nrepl/middleware/complete.clj#L63" target="_blank">function</a>:

{% highlight clojure %}
(defn handle-complete [handler msg]
  (with-safe-transport handler msg
    "complete" complete-reply
    "complete-doc" doc-reply
    "complete-flush-caches" flush-caches-reply))
{% endhighlight %}
