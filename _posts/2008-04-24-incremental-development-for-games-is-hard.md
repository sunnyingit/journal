---
layout: post
title: "Incremental Development for Games (Is Hard)"
categories: code game-dev
---
For those who don't know, the game industry (or at least my chunk of it) is
generally about 10 years behind the rest of the software world. We're still
leery of crazy ivory tower concepts like "memory management", "testing", and
"going home at 5:00 PM". Where your hip Web 2.0 company is all aflutter about
[DSLs](http://www.ceteva.com/nutshell.html), [duck typing](http://docs.python.org/tut/node18.html#l2h-46), and [continuations](http://www.seaside.st/), my company is a little
more:

> **Brown:** I have just received a telegraph from Jenkins discussing something
he calls, "[the Ess Tee Ell](http://www.sgi.com/tech/stl/)".
>
> **Watson:** Desist your crazy Moon Man talk or I shall be forced to give you a
drubbing!

It's not that we aren't trying. We're just a little more [East Berlin](http://www.galenfrysinger.com/east_berlin.htm)
circa 1950 than the rest of the software world. One concept that *has*
[crossed](http://www.agilegamedevelopment.com/) our Iron Curtain recently is this thing called "[Agile
Development](http://agilemanifesto.org/)". I can't say we really know what it means, but it is very
interesting to us.

## Games Are Hard

You see, developing a new game is really hard. On top of the technical reasons
(pushing the graphics hardware, strict quality control) and the business ones
(cutthroat competition, [difficult profitability](http://news.bbc.co.uk/2/hi/technology/6397527.stm)), there's this other
little thing called "fun".

If you're making [business software](http://www.microsoft.com/) and it's a [pain to use](http://office.microsoft.com/), you can
still be totally successful because people need to use your stuff to get their
job done, like it or not. And your software can be [totally unoriginal](http://en.wikipedia.org/wiki/Apple_v._Microsoft)
because people don't want to have to learn new things. Unoriginality is
actually [a bonus](http://notebook.arkane-systems.net/index.php/Jakob's_Law_of_the_Web_User_Experience).

Over in Magical Game Land, though, we aren't so lucky. We're trying to get
people to shell out hard-earned cash for something that, by definition, they
*don't need*. So we have to make sure it's more fun to play than whatever else
they could be doing. We have to make sure we're offering them a *new*
experience, otherwise they'll just play an older game. At the same time, our
new game has to be easy enough to learn that it's enjoyable within the first
30 seconds.

On top of this, budgets and player expectations continue to rise, so the
chance and cost of failure keeps going up. Because of this, if a game's going
to suck, we need to know as soon as possible so we can correct course or [bail
out](http://en.wikipedia.org/wiki/List_of_cancelled_video_games).

## Enter Agile Development

Agile development means a lot of things to different people, but the key part
I want to talk about is the idea of **building a game incrementally through a
series of small changes**. This idea that a month into the cycle you have a
*fun* playable game already and you just build onto that is very appealing
because it means we can immediately get feedback on what parts work and what
don't.

For a micro example, imagine you're building a [chess-like](http://en.wikipedia.org/wiki/Fairy_chess_piece) game. In a
typical [waterfall](http://en.wikipedia.org/wiki/Waterfall_model)-like model, you won't be able to actually play the
game until all the different pieces are designed and implemented. And then you
can play the whole thing. Sometime in alpha, after you've burned most of your
budget. Better hope the game is fun!

With a more agile method, you'd be able to play the game early in the cycle
with a couple of pieces, and then gradually add in more pieces throughout.
You'll be testing some core mechanics early in and can change things if they
don't work.

## Uh-Oh

Even that tiny example hints at the pitfall ahead: games are all about
*balance*. Chess is unplayable with just two kings and some pawns. Without the
other pieces, it's a fundamentally different game.

So now we've gone from "nothing to play" to "playing a totally different game
that may or may not reflect the final game". Ouch.

This is where games are different than other software. If you're making a word
processor, adding right-justification doesn't somehow "unbalance" left-
justification. Features are fairly isolated from each other and don't compete.
In a game (in the [game theory](http://en.wikipedia.org/wiki/Game_theory) sense), every feature or capability
affects the overall strategy and equilibrium.

Building a game incrementally is like trying to build a [hanging mobile](http://www.sfmoma.org/espace/calder/calder_windmobiles.html)
by hanging it up *first* and then hanging successive pieces onto it.

## Solutions?

I suspect this is one of the fundamentally hard parts of designing a new game.
That sense of "everything works in harmony" is irreducible to some extent.
I'll point out a couple of ideas to at least carve at the problem a bit.

### Separate the Core from the Peripheral

If we're designing minimal abstract board games, there aren't a lot of
"peripheral" features. In computer/video-games it's a bit different. Non-
interactive set decoration, some UI, extra games modes, etc. all surround the
core "mechanics" of the game. So, even if you can't do the core mechanics
incrementally, you can at least do them in one lump and then tack on the
additional stuff afterwards.

Be careful here, though. Even seemingly trivial things like UI or [VFX](http://en.wikipedia.org/wiki/Particle_system)
can have a massive impact on the usability of your game.

### Matched Pairs

If you're starting with a balanced game and you need to add features while
keeping it balanced, one obvious solution is to try to add them in opposing
pairs. For symmetric games like chess, this is fairly simple: implement the
same piece for both players. For other games, it gets trickier.

If you're adding speed boosts to a driving game, you might be able to match it
by also adding slowdown and damage for hitting the walls at the same time. For
an RPG, giving the player a new attack may require also adding monsters that
resist it.

### Accepted Retuning

Most games, on top of the feature set itself, have a lot of precise tuning of
numbers that happens to make it fun. Those numbers have to be tuned relative
to the current feature set. Adding a new feature can essentially detune them.
If you give your hero a shotgun, all of the sudden it may be time to double
the enemy's health. Make the AI smarter and your hero is toast.

If a fun playable game at all points through the cycle is important, it may be
worth accepting that you will need to frequently retune the same values as the
feature set changes. To some degree, this is "throwaway" work, but if it gives
you a better ability to evaluate your game, it may be worth the effort.

### Reduce Innovation

Of course, all of the above assumes you're making a game with actually new
mechanics. If you're making chess on the other hand, it doesn't matter if the
game sucks with just kings and pawns. You know by the time the rest of the
pieces are in it will be cool. So the safest option is just to rely on the
established formula of your game's genre, and save your innovation for
peripheral features.
