---
title: "Redlining"
---

In data privacy debates, I sometimes hear a distinction between "personal" data and other types of information tied to a user account. The feeling is that some sensitive types of personal information -- attributes like race, annual income, gender, sexual preference, religious beliefs, etc. -- should be a special, protected class of data, deserving of particular care and stewardship. By exclusion, other forms of data would have less stringent storage and sharing rules. As a hypothetical example: It would clearly be bad if every employee at TurboTax had full access to the annual income and social security number of each tax filer. We'd want to protect that sensitive information. But maybe we're less worried about TurboTax employees having access to less sensitive information, such as the dates on which tax returns were filed.

I don't think it's bad to be especially vigilent of "personal" data, but I do think it's overly simplistic to think that there's a clear dividing line between what's personal and what's not. The reality is that "innocuous" data is often enough to predict extremely "personal" data. Even with simple machine learning and analytics, you can often deduce "sensitive" data from "non sensitive" data.

[Redlining](https://en.wikipedia.org/wiki/Redlining) is a prominent historical example. In some cities, home address can be used to accurately predict race. You can forbid banks from denying loans based on race, but if banks can deny loans based on geographic location, there's a backdoor to discrimination via a hidden variable. So to protect race as sensitive data, we might need to protect physical address as sensitive data too.

A more recent example is the idea of [using NYC's open taxi cab data to predict which cab drivers are Muslim](https://www.reddit.com/r/dataisbeautiful/comments/2t201h/identifying_muslim_cabbies_from_trip_data_and/). There isn't a source of truth here, so we don't know how accurate this prediction is -- but I find it convincing as a proof of concept. Similarly to the last example: We want to predict religious beliefs as personal data, so perhaps we also need to protect the pick-up/drop-off data timestamps for cab drivers.

The point is that data which doesn seem personal or sensitive at first could, in fact, be used to accurately predict data that is **very** personal or sensitive. If you want to effectively protect the variable `y`, then you may also need to protect any set of variables `x1`, `x2`, ..., `xn` that can be used to **predict** `y`. This is a hard problem, because there are lots and lots of things that might predict `y`.

In short, protecting personal data is a hard problem, because "non personal" data leaks personal data.

