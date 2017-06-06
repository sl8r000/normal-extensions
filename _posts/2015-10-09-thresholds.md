---
title: "Using ML Thresholds to Make Money"
---

> This post was written for [Insight Data Labs](http://blog.insightdatalabs.com/visualizing-classifier-thresholds/). Check them out! If you're a Ph.D. student interested in a career in data science or data engineering, check out [Insight's Fellows Program](http://insightdatascience.com/).

As data scientists, when we build a machine learning model our ultimate goal is to create value: We want to leverage our model's predictions to do something *better* than we were doing it before, when we didn't have a model or when our model was more primitive. Focusing on outcomes means that our final measure of a model's performance is how useful it was, measured as the amount of value it created in the application for which it was used. In this post, we'll use data visualization as a powerful tool in choosing and understanding the modeling decisions that maximize business value.

For classification algorithms, one of the most common usage patterns is **thresholding**: All cases with a model score above the threshold get some sort of special treatment. For example:

* **Fraud Prevention**: You're at a social network company, and you'd like to delete fake accounts. You build a classifier that assigns a "fraud score" between 0 and 1 to each account and, after some research, decide that all accounts whose score is above 0.9 should be sent to your fraud team, which will review each case and delete the accounts that are actually fake.
* **Response Modeling**: You're at an enterprise software company, and you'd like to improve your outbound sales program. You buy a large database of potential customers, and build a classifier to predict which ones are likely buy your product if contacted by your sales team. You decide that all customers with a "response score" of above the threshold 0.7 should get a call.
* **Shopping Cart Abandonment**: You're at an e-commerce company, and you'd like to email a 10% off coupon to users who abandoned their shopping carts and won't return organically. (You don't want to give a 10% off coupon to users who are going to return anyway, since then you'd be losing 10% of the sale price.) You build a classifier to predict which users will never return to their carts, and decide that all users with a "abandonment score" above 0.85 should get the 10% off coupon.
* Etc.

Thresholding is popular because of its simplicity and ease of implementation: We translate a continuous score to a binary yes/no decision, and act on it in a predetermined way.

The biggest question for the thresholding pattern is: **Where should I set the threshold point?** This is a business decision rather than a prediction problem, so there's no universal answer; in this post, however, we'll show the visualization methods that will help you make the right decision for yourself.

## Visualizing Thresholding Decisions

There are three factors to consider when choosing a threshold point:

* **Queue Rate:** How many cases do you want to act on? This will depend on the cost of treating an individual case, as well as your overall capacity. If you're asking a team of reviewers to look at potential fake accounts, then the number of cases you can flag is limited by the resources of that team: How many cases do they have the ability to review within your timeframe? If, on the other hand, you're sending out mass emails, your variable cost is nearly zero, and you can afford to queue a lot of cases. Etc.
* **Precision:** What's the downside when you erroneously queue a case? If you're reviewing fraudulent accounts, the cost is wasted reviewer time. If you mistakenly send a 10% off coupon to someone who would have completed his or her purchase anyway, your cost is 10% of their shopping cart. Etc.
* **Recall:** What's the downside when you fail to queue a case that should have been queued? If you're queueing sales leads, you're losing the value of a missed opportunity. If you're reviewing fake accounts, the cost is tied to the cost of having fraudulent accounts in your network. Etc.

To understand all these factors at once, I like to draw a chart that shows queue rate, precision, and recall as a function of classifier threshold. This allows you to easily see the tradeoffs you face for different thresholds, and to make the optimal choice for your specific situation.

To see this in action, let's look at the famous [telecom churn data set](https://bigml.com/user/francisco/gallery/dataset/5163ad540c0b5e5b22000383). This data set consists of information about phone plan subscribers; the goal is to predict which users will churn before the next billing period. (By now we should be asking: What is the *value* of being able to predict churn? What can we *do* with these predictions? The answer depends on company priorities and the value of churn prevention, but some ideas are: send a $10 off coupon to high-risk users, offer high-risk users a discount if they sign a new 12 month contract now, offer to subsidize a new phone purchase to such users if they remain in contract for another 12 months, etc.)

Let's build a classifier to solve this task. We'll use an out-of-the-box random forest model:

```python
# Imports
import pandas as pd
import numpy as np

from sklearn.cross_validation import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import precision_recall_curve
from sklearn.preprocessing import LabelEncoder


# Retrieve Data Set
df = pd.read_csv('http://www.dataminingconsultant.com/data/churn.txt')


# Some Prepreocessing
df.columns = [c.lower().replace(' ', '_').replace('?', '').replace("'", "") for c in df.columns]

state_encoder = LabelEncoder()
df.state = state_encoder.fit_transform(df.state)

del df['phone']

binary_columns = ['intl_plan', 'vmail_plan', 'churn']
for col in binary_columns:
    df[col] = df[col].map({
            'no': 0
        ,   'False.': 0
        ,   'yes': 1
        ,   'True.': 1
    })


# Build the classifier and get the predictions
clf = RandomForestClassifier(n_estimators=50, oob_score=True)
test_size_percent = 0.1

signals = df[[c for c in df.columns if c != 'churn']]
labels = df['churn']

train_signals, test_signals, train_labels, test_labels = train_test_split(signals, labels, test_size=test_size_percent)
clf.fit(train_signals, train_labels)
predictions = clf.predict_proba(test_signals)[:,1]
```

Now let's see what the performance of this model is as a function of the threshold. We'll use the handy `precision_recall_curve` function from `sklearn`

```python
precision, recall, thresholds = precision_recall_curve(test_labels, predictions)
thresholds = np.append(thresholds, 1)

queue_rate = []
for threshold in thresholds:
    queue_rate.append((predictions >= threshold).mean())

plt.plot(thresholds, precision, color=sns.color_palette()[0])
plt.plot(thresholds, recall, color=sns.color_palette()[1])
plt.plot(thresholds, queue_rate, color=sns.color_palette()[2])

leg = plt.legend(('precision', 'recall', 'queue_rate'), frameon=True)
leg.get_frame().set_edgecolor('k')
plt.xlabel('threshold')
plt.ylabel('%')
```

![single_curve](http://i.imgur.com/HLYU9Sh.png){: .center-image}

Here's how we can read this chart: For a given threshold on the x-axis, we can see queue rate, precision, and recall expressed as percentages against the y-axis. For example, if we choose a threshold of 0.4 (all cases with a score above 0.4 get reviewed / sent an offer / whatever treatment we have at hand), then:

* About 14% of cases will be queued. So if 10k accounts are scored for churn each day, about 1400 of them will be selected for special treatment.
* Our precision will be about 92%. So of the cases we queue, 92% will really go on to churn if we don't intervene.
* Our recall will be about 80%. Of all users who will churn before the next billing period, we'll catch 80% with our model.

Of course, 0.4 might not be the best threshold value for this particular company. If, for example, you only have the capacity to act on 500 cases (5% of the hypothetical 10k scored cases) per period, then you're bound to choose a threshold around 0.8, in which case your precision jumps to 100% but your recall drops to ~35%. The point is that with this chart, you are now equipped to make informed decisions about where to set your threshold.

## Dealing with Model Variability

The chart we produced above only shows the performance of a single train/test split. While we'd hope that this sample is indicative of likely model performance, it's better to visualize many train/test splits to get an idea of the range of possible performance outcomes. One of the simplest ways to do this is to perform many random train/test splits, and plot the curves for each. Here's what that looks like for 50 random splits on the telecom churn model:

```python
clf = RandomForestClassifier(n_estimators=50, oob_score=True)

n_trials = 50
test_size_percent = 0.1

signals = df[[c for c in df.columns if c != 'churn']]
labels = df['churn']

plot_data = []

for trial in range(n_trials):
    train_signals, test_signals, train_labels, test_labels = train_test_split(signals, labels, test_size=test_size_percent)
    clf.fit(train_signals, train_labels)
    predictions = clf.predict_proba(test_signals)[:,1]

    precision, recall, thresholds = precision_recall_curve(test_labels, predictions)
    thresholds = np.append(thresholds, 1)

    queue_rate = []
    for threshold in thresholds:
        queue_rate.append((predictions >= threshold).mean())

    plot_data.append({
            'thresholds': thresholds
        ,   'precision': precision
        ,   'recall': recall
        ,   'queue_rate': queue_rate
    })

for p in plot_data:
    plt.plot(p['thresholds'], p['precision'], color=sns.color_palette()[0], alpha=0.5)
    plt.plot(p['thresholds'], p['recall'], color=sns.color_palette()[1], alpha=0.5)
    plt.plot(p['thresholds'], p['queue_rate'], color=sns.color_palette()[2], alpha=0.5)

leg = plt.legend(('precision', 'recall', 'queue_rate'), frameon=True)
leg.get_frame().set_edgecolor('k')
plt.xlabel('threshold')
plt.ylabel('%')
```

![curves](http://i.imgur.com/e3g3Ec1.png){: .center-image}

As expected, there's some variation. Earlier, when we were just looking at a single train/test split, we said that with a threshold of 0.4, we'd expect:

* Queue rate of about 14%.
* Precision of about 92%.
* Recall of about 80%.

Now we see that there is more uncertainty around these numbers. Looking at the new chart, which shows 50 train/test splits, we might now say instead that a threshold of 0.4 would produce:

* A queue rate of between 10% and 20%
* Precision of between 70% and 100%
* Recall of between 60% and 90%.

We can make more precise statements with a statistical visualization. Let's plot the median curves, along with a 90% central interval for each threshold:

```python
import bisect
from scipy.stats import mstats

uniform_thresholds = np.linspace(0, 1, 101)

uniform_precision_plots = []
uniform_recall_plots= []
uniform_queue_rate_plots= []

for p in plot_data:
    uniform_precision = []
    uniform_recall = []
    uniform_queue_rate = []
    for ut in uniform_thresholds:
        index = bisect.bisect_left(p['thresholds'], ut)
        uniform_precision.append(p['precision'][index])
        uniform_recall.append(p['recall'][index])
        uniform_queue_rate.append(p['queue_rate'][index])

    uniform_precision_plots.append(uniform_precision)
    uniform_recall_plots.append(uniform_recall)
    uniform_queue_rate_plots.append(uniform_queue_rate)

quantiles = [0.1, 0.5, 0.9]
lower_precision, median_precision, upper_precision = mstats.mquantiles(uniform_precision_plots, quantiles, axis=0)
lower_recall, median_recall, upper_recall = mstats.mquantiles(uniform_recall_plots, quantiles, axis=0)
lower_queue_rate, median_queue_rate, upper_queue_rate = mstats.mquantiles(uniform_queue_rate_plots, quantiles, axis=0)

plt.plot(uniform_thresholds, median_precision)
plt.plot(uniform_thresholds, median_recall)
plt.plot(uniform_thresholds, median_queue_rate)

plt.fill_between(uniform_thresholds, upper_precision, lower_precision, alpha=0.5, linewidth=0, color=sns.color_palette()[0])
plt.fill_between(uniform_thresholds, upper_recall, lower_recall, alpha=0.5, linewidth=0, color=sns.color_palette()[1])
plt.fill_between(uniform_thresholds, upper_queue_rate, lower_queue_rate, alpha=0.5, linewidth=0, color=sns.color_palette()[2])

leg = plt.legend(('precision', 'recall', 'queue_rate'), frameon=True)
leg.get_frame().set_edgecolor('k')
plt.xlabel('threshold')
plt.ylabel('%')
```

![intervals](http://i.imgur.com/xKHb6c0.png){: .center-image}

Now we can be considerably more precise by saying that for a threshold of 0.4, we expect:

* A 90% chance that our queue rate will be between ~11% and ~15%, with median value ~13%.
* A 90% chance that our precision will be between ~80% and ~95%, with median value ~90%.
* A 90% chance that our recall will be between ~70% and ~85% with a median value ~80%.

With this chart, we can say quite a lot about the likely outcomes of different threshold choices under realistic conditions.

## Thresholding Like an Owner

One of the things I love about the last chart is that it empowers you to make business decisions based on your ML work. This gives you the opportunity to move beyond the technical role of building a model into the role of deciding what to actually do with your model. All you need to get started is the chart above and some back of the envelope numbers for cost/benefit analysis.

As a simple example, let's say that your company's churn prevention strategy is to make direct phone calls to each churn-risky user (defined as users whose churn score is above the threshold you set). To keep things easy, we'll assume unrealistically that this strategy always works: If a user who was going to churn gets a phone call, he or she will not churn. Your retention team has 3 members, each of whom can make 333 calls per period; so you can make a total of 1000 calls per period. Let's say again that you get 10k cases to score each period, that the cost of labor in making a phone call is $20, and that if a user churns, you lose $100. These estimates need not be perfect for our purposes; we're looking for a back of the envelope calculation.

So right now, on the curve, you're going to set a threshold of about 0.57: That will give you a queue rate of 10.03% = 1003 cases, and that's all you can handle. The chart also shows you uncertainty. You know, e.g., that with a threshold of 0.57, you'll end up between about 8.4% = 840 and 11.4% = 1140 cases. If you know that your team absolutely cannot handle more than 1000 cases, then you should leave some room: Choose a lower threshold (e.g. 0.65) so that the 90% interval lies below your 1000 case limit.

Even more importantly: Because you have your chart, you can see that having a 1000 call capacity is suboptimal for your business; the clue is the fact that recall is so low. Right now, your total cost is around $20 x 1000 = $20,000 and, since your precision is 96%, you're making about 0.96 x $100 x 1000 = $96,000 in saved account "revenue", making the total profit of your operation about $76,000. But what if you doubled your budget? Then you could review 2k cases, your cost would be $40,000, your precision would be about 61% and you'd make 0.61 x $100 x 2000 = $122,000 in revenue, making your profit around $82,000. Your chart just helped you make $6,000 *per period*.

This was just an educated guess about a better threshold; if we poke around a bit more, we find that the optimal threshold for these specific cost estimates is 0.33, which gives us a median profit of about $90,700.

```python
uniform_thresholds = np.linspace(0, 1, 101)

uniform_payout_plots = []

n = 10000
success_payoff = 100
case_cost = 20

for p in plot_data:
    uniform_payout = []
    for ut in uniform_thresholds:
        index = bisect.bisect_left(p['thresholds'], ut)
        precision = p['precision'][index]
        queue_rate = p['queue_rate'][index]
        
        payout = n*queue_rate*(precision*100 - case_cost)
        uniform_payout.append(payout)

    uniform_payout_plots.append(uniform_payout)

quantiles = [0.1, 0.5, 0.9]
lower_payout, median_payout, upper_payout = mstats.mquantiles(uniform_payout_plots, quantiles, axis=0)

plt.plot(uniform_thresholds, median_payout, color=sns.color_palette()[4])
plt.fill_between(uniform_thresholds, upper_payout, lower_payout, alpha=0.5, linewidth=0, color=sns.color_palette()[4])

max_ap = uniform_thresholds[np.argmax(median_payout)]
plt.vlines([max_ap], -100000, 150000, linestyles='--')
plt.ylim(-100000, 150000)

leg = plt.legend(('payout ($)', 'median argmax = {:.2f}'.format(max_ap)), frameon=True)
leg.get_frame().set_edgecolor('k')
plt.xlabel('threshold')
plt.ylabel('$')
plt.title("Payout as a Function of Threshold")
```

![payout](http://i.imgur.com/zcFEAtV.png){: .center-image}

We can also see that there's a lot of uncertainty in the profitability of our operation; knowing this ahead of time let's you move more confidently when natural variation occurs.

(As a side note, it's worth pointing out too that if we had used the naive value of 0.5 (i.e. queue cases with churn score above 0.5, ignore cases below 0.5), which is a common default value for classifiers, we would have created far less value than we were able to do do by using our thresholding chart. With a threshold of 0.5 and review capacity to match, we would have made about $84,000 rather than $89,200, more than a $5,000 difference per period!)

In general, using the queue rate / precision / recall graph is an easy way to perform "what if" analysis on the operational and strategic decision of how your model can be best used.

## Bottom Line

Thresholding is a simple and effective strategy for creating value from a machine learning classifier. Once you've decided to threshold, data visualization techniques like the ones in this post will let you understand the tradeoffs you face for each possible threshold choice -- and thus help you choose the most value-creating threshold for your particular business application.

*An IPython notebook for this post is [available here](https://github.com/InsightDataLabs/ipython-notebooks/blob/master/classifier_thresholds.ipynb).*


