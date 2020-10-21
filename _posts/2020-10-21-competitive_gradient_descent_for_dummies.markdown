---
layout: post
title:      "Competitive Gradient Descent for Dummies"
date:       2020-10-21 07:10:55 -0400
permalink:  competitive_gradient_descent_for_dummies
---


First thing's first: for all intents and purposes, I am a "dummy" in this case.  But, in general, this article is intended to be read by "non-technical stakeholders".  I will have done my job if my technologically-averse father can understand it.

With that said, allow me to launch into some context: *Generative Adversarial Networks* (or GANs, for short).  I learned about Competitive Gradient Descent just recently when I attended the virtual [GANs for Good](http://post.spmailtechnol.com/f/a/BV4eH2zDtxdcB13cDpyjow~~/AAQxAQA~/RgRhVgybP0QcaHR0cHM6Ly95b3V0dS5iZS85ZDRqbVBtVFdtY1cDc3BjQgoASBvZdF91agPnUhtzdGV2ZS5hLmNvbnRyZXJhc0BnbWFpbC5jb21YBAAAAAA~) event.

So let's first talk about GANs...

## GANs: High-level Overview by Analogy
Probably the easiest way to think of what a *Generative Adversarial Networks* is, is to think of two separate computers playing a two-player game against each other.  Here, the players are the computers.  Each computer is programmed to play this "game", which has not yet been named, and to beat the other "player" (computer).  There can only be one winner in this game.  The player that doesn't win the game, obviously loses the game.  So far so good, right?  Sounds obvious, right?

### The *Counterfeit Currency* Game
Well, it will make more sense if we begin to discuss the *nature* of the game: the rules and objectives of each player.  For lack of a better term, let's call this game "Counterfeit Currency".  You can already see how this is going to go.  GOOD.

So, in the game of *Counterfeit Currency*, there is a "good" and an "evil" side.  

#### The Counterfeiter
One player (computer) in *Counterfeit Currency* will be the *Forger* or *Counterfeiter* (is that a word? hint: it is... I just looked it up).  The *Counterfeiter* is not just *called* a "*Counterfeitor*".  A *Counterfeiter* is a player that *should* have the skills to carry out the act of counterfeiting currency.  In this case, the *Counterfeiter* is a computer, so it should be programmed with at least the goal of creating "fake" currency.  But to what end?

#### The Banker
Well, this is where the second player (computer) comes in to play.  Let's call this player the *Banker*.  I am sure by now you can really see where this is going.  I am no Economist and I wil not pretend to really know what I am talking about when it comes to banking systems.  So let's just keep this to the bare minimum, shall we?  The *Banker*, in this case, has the basic interest to keep the "real" value of the currency high.  If currency is not backed by some relative "real" value, then terrible things like inflation happen, not to mention the fact that his (the *Banker*'s) job gradually becomes more and more difficult with an influx of more currency (less value).

#### The Crux of GANs
Let's just say the goal of the *Counterfeiter* is to trade counterfeit currency for (real) Gold bars, from the *Banker*'s vault.  The *Banker*'s goal, relative to the *Counterfeiter*, is to detect when someone tries to trade-in counterfeit currency.  Thus, **the two players have competing goals, in direct opposition of the other's**.

The thing is, our players were "noobies" when they both started out in their respective careers, as *Counterfeiter* and *Banker*, respectively.

So, it would be reasonable for the *Counterfeiter* to use *real* currency as a template, a source that he should try to copy when he attempts to create his own *new* counterfeit currency.  At first he's not going to be very good at it.  He may use inferior tools.  Maybe he tries to create his first few copies with crayons...  Who knows?!  He's a "noob", remember?!

In any case, at first, the *Banker* has a very easy time spotting these fakes.  The *Banker* has the advantage... at first.

At a minimum, the *Counterfeiter* knows when his tricks work - when he fools the *Banker* and trades in some fake currency for real gold - and when they don't, since he gets no gold!  When his tricks don't work, the *Counterfeiter*
meticulously goes over his prior attempt, to inspect the minor differences between the details of his work and real currency.  With this knowledge, he aims to learn from the mistakes he made in the past, to improve his work on his attempts at counterfeit currency in the future.

Meanwhile, a similar learning process occurs with the banker.  As the *Counterfeiter* gains skill, some of his work will be detected as counterfeit by the *Banker*.  But it will become increasingly difficult for the *Banker* to do so if the *Counterfeiter* is learning and gaining skill.  Nevertheless, the times when the *Banker* does detect counterfeit currency, the *Banker* improves his skill at counterfeit-currency detection.

**By virtue of direct competition, with diametrically opposing goals, the two players in this game learn from and improve each other's skills, or knowledge of how to play the game, *strategically***.

From a very high-level, we have just explored the most basic concepts of how GANs work.  Now we can change up terms a little bit.  In GAN's, the *Counterfeiter* is called the *Generator* and the *Banker* is called the *Discriminator*.

**The *Generator*'s function in GANs is to *create* new, "fake" copies of the original content, whatever that may be, with the express purpose of fooling the the *Discriminator*.  Meanwhile, the *Discriminator*'s function in GANs is to differentiate authentic/original content from fake copies produced by the *Generator*.**

### The Cost of Losing: *Gradient Descent* in GANs
At each turn, one player's win is the other player's loss.    A player's win equates to a cost to the other player.  

In our example, the cost to the *Banker* when the *Counterfeiter* wins is real-valued gold bars!  When the *Banker* catches the *Counterfeiter*'s deception, he can't pay for groceries since he has no gold bars!  (These are some very, VERY expensive groceries!)

**The "cost" to each player can be represented as a mathematical formula specific to each.  This is called - go figure - the "Cost Function".  In GAN's, under the hood, each player - the *Generator* or *Discriminator* - learns or improves by minimizing the cost of each "move" it makes; that is, by minimizing the value output by the "Cost Function".  In general, the act of minimizing the Cost Function is what we call "Gradient Descent" in Deep Learning but is only applicable to "single-agent" optimization.**

### Convergence vs. Divergence
Sometimes, though, games are very loosely defined and don't really allow for minimizing the cost of losing at each turn.

Consider another example: the game of "Rock, Paper, Scissors".  Unless players have predictable habits, any move (Rock, Paper, or Scissors) is just as good as another.  A player can't really formulate a "strategy", per se.  Again, unless a player is predictable or manifesting some "tell" to indicate what his or her move is going to be, the game is essentially left up to chance.

There is no chance for improvement or learning in this game.  It is completely random, all things being equal.  This means that the so-called "Cost Function" for a player cannot be minimized over time.  It never "converges" to some minimal cost, to arrive at some optimal strategy.  This is known as "Divergence" for all intents and purposes.  And when a player's cost function can be minimized over time, it is said to "Converge" and corresponds to some optimal strategy.

### Competitive Gradient Descent
When a "game" only involves one player, the description of minimizing cost is refered to as *Gradient Descent*.  Having described this in a fair amount of high-level detail, now, we can now easily augment this definition.  From a VERY high level, therefore, ***Competitive Gradient Descent* can be thought of as  *Gradient Descent* with more than one player, wherein the act of minimizing one player's cost function depends on the moves other players in the game have recently made**.

Simple, right?

Yeah, right.  Not.  But I hope I have given at least an inkling of what it might resemble.


