---
title: "Kelly Criterion II"
---

Let's generalize from last time to betting on a subset of mutually exclusive events `E_1, ... E_n`. Event `E_i` occurs with (true) probability `p_i`, and the market thinks that it occurs with (possibly untrue) probability `l_i`, and thus a long contract costs `l_i`. If we invest `d_i` dollars in `E_i` occurring, we get `d_i/l_i` shares. These shares resolve to being worth `$1` if `E_i` occurs, and thus our `d_i/l_i` shares resolve to be worth `d_i/l_i` dollars.

Our total cost is `d = d_1 + d_2 + ... + d_n`, so in net, if `E_i` occurs, our bankroll changes from (wlog) `$1` to `$1 + d_i/l_i - d`. Knowing this, we can write out the expected log value:

$$\mathbb{E}(\log(X)) = \sum_i p_i \log\left( 1 + \frac{d_i}{l_i} - d\right).$$

The Kelly-Optimal bet is the set of values $$d_1, \ldots, d_n$$ that maximize this expression, subject to the constraint that $$0 \leq \sum_i d_i \leq 1$$.

Let's try this out with a numerical solver in Excel:

![link](http://i.imgur.com/uCAoYpj.png){: .center-image}

Here we're using solver to find the value of the yellow cells that maximizes `E(log(X))`. Excel is telling us to put 22% of our bankroll into Event 2 and 38% of our bankroll into Event 3. Notice that Event 3 has a 50% chance of occurring, though the market believes the chance is only 30%. Using the formula from our last post, we know that if we could bet in this contract alone, Kelly would have us wager (50% - 30%)/70% = 28.6 of our payroll. In this case, having more contracts allows us to increase our return.

