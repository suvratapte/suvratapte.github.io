---
layout: post
title:  Simulating Prisoner's Dilemma
author: Suvrat Apte
date:   2024-06-01 08:45:40 +0200
categories: clojure, game-theory
comments: true
---

A recent [Veritasium video](https://www.youtube.com/watch?v=mScpHTIi-kM) got me
interested in Game Theory and I decided to try to implement some of the
experiments in game theory.

The word "game" in game theory makes it seem that it is related to (possibly
silly) computer or board games. But that is not true. In game theory, we create
models which attempt to replicate real life situations in terms of rules and
some scoring system to measure the outcomes. A game has rules and also a scoring
system. That's why the name.

Prisoner's Dilemma is one of the famous games in game theory. You can read about
it [here](https://en.wikipedia.org/wiki/Prisoner%27s_dilemma).

We are going to implement the inverted version of this game. The rules of our
game are as follows:

There are two players, they can either cooperate with each other or defect.  If
both cooperate, they get 3 points each. If one cooperates and the other defects,
they defector gets 5 points and the other gets 0. If both defect, both get 1
point each.

|-------------------------|-------------------------|-------------------------|
|                         | Player 1 cooperates     | Player 1 defects        |
|-------------------------|-------------------------|-------------------------|
| **Player 2 cooperates** | P1 - **3**   P2 - **3** | P1 - **5**   P2 - **0** |
| **Player 2 deffects**   | P1 - **0**   P2 - **5** | P1 - **1**   P2 - **1** |
|-------------------------|-------------------------|-------------------------|

My plan is to implement a two player version first and then implement a
multi-player version (which will be a new post).

<!---excerpt-break-->

## Implementation

This game will be played between two players and they will play many rounds. So
we will need to store the moves that each player has played in all the rounds
and the scores of the players. So let's define the `state` of the game:

{% highlight clojure %}

(def initial-state
  {:first-player-moves []
   :second-player-moves []
   :score {:first-player 0
           :second-player 0}})

{% endhighlight %}

Now let's see how the players can make a move. For this, we need to a function
for each player which will return whether the player wants to cooperate or
defect. Let's represent cooperation with `:co` and defection with `:de`.

For the players to decide their next moves, we need to provide them with the
history of their previous moves. So that they can devise a strategy which takes
into account the previous moves.

{% highlight clojure %}

(defn- tit-for-tat
  [player-moves opponent-moves]
  (or (last opponent-moves) :co))

(defn- devil
  [_player-moves _opponent-moves]
  :de)

{% endhighlight %}

The `tit-for-tat` strategy just does what the other player has done in the last
round. And if it is the first round, it cooperates.
The `devil` strategy just defects all the time.

Now that we know how to write strategy functions which give us moves, let's
compute the score based on the moves.

{% highlight clojure %}

(defn- compute-score
  "Returns the score in this format:
   [<first-player-score> <second-player-score>]"
  [first-player-move second-player-move]
  (cond
      ;; Both cooperate
      (and (= :co first-player-move)
           (= :co second-player-move))
      [3 3]

      ;; Both defect
      (and (= :de first-player-move)
           (= :de second-player-move))
      [1 1]

      ;; One cooperatiion one defection
      (and (= :co first-player-move)
           (= :de second-player-move))
      [0 5]

      (and (= :de first-player-move)
           (= :co second-player-move))
      [5 0]))

{% endhighlight %}

`compute-score` as the name suggests, computes the score given the moves of the
first and the second player.

Now to play this game over and over again for many rounds, we have to update the
state of the game and keep track of the total score. Let's do that:

{% highlight clojure %}

(defn- simulate-game
  [rounds-number first-player-stragey second-player-stragey]
  (loop [state initial-state
         n rounds-number]
    (let [first-player-moves (:first-player-moves state)
          second-player-moves (:second-player-moves state)
          first-player-move (first-player-stragey first-player-moves
                                                  second-player-moves)
          second-player-move (second-player-stragey second-player-moves
                                                    first-player-moves)
          [first-player-score second-player-score]
          (compute-score first-player-move
                         second-player-move)

          new-state (-> state
                        (update :first-player-moves conj first-player-move)
                        (update :second-player-moves conj second-player-move)
                        (update-in [:score :first-player] + first-player-score)
                        (update-in [:score :second-player] + second-player-score))

          n (dec n)]
      (if (pos? n)
        (recur new-state n)
        new-state))))

{% endhighlight %}

This is the complete implementation of the two player version of this
game. Now we can simulate this game between two strategies:

{% highlight clojure %}

(:score (simulate-game 100 devil tit-for-tat))

{% endhighlight %}

I have some more strategies
[here](https://github.com/suvratapte/game-theory/blob/main/src/game_theory/single_player.clj).

Play around and find out which is the best strategy! :-)

I have also written a multi player version [here](https://github.com/suvratapte/game-theory/blob/main/src/game_theory/multi_player.clj).
I will soon write a post about it.

:-)
