---
title: "Kelly Criterion I"
---

The [Kelly Criterion](https://www.wikiwand.com/en/Kelly_criterion) tells you how much to bet so that your long-run IRR is maximized. It's useful when you believe you have an information edge over other market participants. In this post, we'll look at a simple but revealing example.

Let's say that we're betting on heads for a coin flip; if the coin comes up heads, you win $1, and if it comes up tails, you lose whatever you paid to play the game. You can play this game as many times as you like, as long as you don't run out of money. The market believes that the probability of heads is `p_market`, and thus the game is priced at `$1 x p_market` to play. But you know the probability of heads is really `p_true`. With $100 in your pocket, how much should you bet on each flip?

Without going into the [proof](http://www.herrold.com/brokerage/kelly.pdf), it turns out that the Kelly Criterion is equivalent to maximizing the expected log value of our bankroll. In our case, if we bet the fraction `f` of our bankroll on each coin flip, we're seeking to maximize

```
E(log(X)) = p_true x log(1 + f x ((1 - p_market)/p_market))
     + (1 - p_true)x log(1 - f)
```

After a derivative and some arithmetic: The `f` that maximizes this is `f = (p_true - p_market)/(1 - p_market)`.

Let's plug in some numbers as an example. Suppose that the market thinks the probability of heads is `0.6`. However, we know that the true probability of heads is `0.8`. Then we should bet `0.2/0.4 = 50%` of our bankroll on each coin flip. Similarly for other values of `p_market` and `p_true`:

![link](http://i.imgur.com/Ky9wItz.png){: .center-image}

Continuing the `p_market = 0.6` and `p_true = 0.8` example, we have one last thing to calculate, which is our IRR. Using the same `E(log(X))` reasoning, our IRR 'in the long run' should be

```
-1 + (1 + 0.5x(0.4/0.6))^(0.8) x (1 - 0.5)^(0.2).
```

Namely, `9.58%`. This is IRR "per flip", i.e. if we flip the coin 100 times, we expect to make `(1.0958)^(100) = 9,400` times our money.

