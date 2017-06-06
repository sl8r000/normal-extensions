---
title: "Author Identification with Markov Models"
---

# How it Works

A couple of  weeks ago, I had a conversation with a colleague at <a href="http://www.elasticbox.com/">ElasticBox</a> about machine-generated papers that were successfully submitted to research conferences. (Definitely check out the awesome <a href="http://thatsmathematics.com/mathgen/">Mathgen engine</a>; while it's not completely convincing -- i.e. I hope that any reviewing mathematician would immediately identify the papers as gibberish -- it's still very, very fun.) As a side-note, he mentioned an application of textual Markov models that I hadn't heard about before: Author identification. More specifically, he told me that someone had used a Markov model to find out who had written the anonymous portions of the Federalist Papers.

This was too interesting to ignore, so when I got home that night, I read up. The work on the Federalist Papers appears in a <a href="http://www.philol.msu.ru/~lex/khmelev/published/llc/khmelev.html">2001 paper by Khmelev and Tweedie</a>; they attribute the idea to Milov (1994) and Holmes (1998).

Stripped of a few complications, the procedure is refreshingly simple: Say you have two authors, \\(A\\) and \\(B\\). You have a large body of text \\(T\_A\\) that you know was written by \\(A\\) and a large body of text \\(T\_B\\) that you know was written by \\(B\\). For some small value of \\(n\\), you build the \\(n\\)-gram Markov models for \\(M\_A, M\_B\\) from \\(T\_A\\) and \\(T\_B\\) respectively. Once you have these models, you can calculate the probability that \\(M\_A\\) and \\(M\_B\\) generate a given text sample. More precisely, let \\(P\_A\\) be the \\(n\\)-gram probability distribution for \\(M\_A\\); then the probability that \\(M\_A\\) produces some string \\(\sigma = a\_1 a\_ 2a\_ 3\cdots a\_{n+k}\\) (\\(k \geq 0\\)), is \\[P\_A(\sigma) = P\_A(a\_1 a\_2 \cdots a\_n) \prod\_{i=1}^{k} P\_A(a\_{n+i} \mid a\_{i+1} a\_{i+2} \cdots a\_{n+i -1}).\\] Similarly for \\(B\\).

To predict the author of an unknown text, we compute the probabilities that \\(M\_A\\) and \\(M\_B\\) produce that text (or a sample from it); i.e. we compute \\(P\_A(\sigma)\\) and \\(P\_B(\sigma)\\) for the sample \\(\sigma\\). Then we just choose the author whose model had the highest probability.
<h1>Let's Try It!</h1>
Let's see if we can actually get this to work. I wrote a small <a href="https://github.com/sl8r000/stat_text">"statistical text" helper class</a> to make our lives a bit easier; we'll use it to build the Markov models.

First, let's pick two authors, and get two texts (for training and testing) for each from Project Gutenberg. I chose Dickens and London; for Dickens, we'll grab <emph>David Copperfield</emph> and <emph>Oliver Twist</emph>, and for London, we'll use <emph>Call of the Wild</emph> and <emph>White Fang</emph>.

<pre class="prettyprint"><code class="language-python">In [4]: dickens_training_text = StatText.simplify(requests.get('http://www.gutenberg.org/cache/epub/766/pg766.txt').text)

In [5]: dickens_testing_text = StatText.simplify(requests.get('http://www.gutenberg.org/cache/epub/730/pg730.txt').text)

In [6]: london_training_text = StatText.simplify(requests.get('http://www.gutenberg.org/cache/epub/215/pg215.txt').text)

In [7]: london_testing_text = StatText.simplify(requests.get('http://www.gutenberg.org/cache/epub/910/pg910.txt').text)
</code></pre>

(To clarify, <code>dickens_training_text</code> is <emph>David Copperfield</emph>, <code>dickens_testing_text</code> is <emph>Oliver Twist</emph>, <code>london_training_text</code> is <emph>Call of the Wild</emph>, and <code>london_testing_text</code> is <emph>White Fang</emph>.)

It turns out that <code>dickens_training_text</code> is about ten times longer than <code>london_training_text</code>:

<pre class="prettyprint"><code class="language-python">In [10]: len(dickens_training_text)
Out[10]: 1852406

In [11]: len(london_training_text)
Out[11]: 188181
</code></pre>

This will mess up our predictions a bit: An unlikely sequence of, e.g., \\(5\\) characters from <code>london_testing_text</code> has a decent chance of appearing in <code>dickens_training_text</code> simply by volume. <strong>So let's cut off <code>dickens_training_text</code> at the length of <code>london_training_text</code></strong>:

<pre class="prettyprint"><code class="language-python">In [12]: dickens_training_text = dickens_training_text[:len(london_training_text)]
</code></pre>

This is a crude way to solve the problem, but (as we'll see) an effective one. (By the way: Notice that other than calling <code>StatText.simplify</code>, which just removes punctuation and converts everything to lowercase, we haven't done any cleaning of our texts. In particular, the 'junk' appearing at the beginning of Project Gutenberg texts is still there. Because the text sample is large enough, though, this won't really affect our work -- a testament to the robustness of the model.) 

Now let's make the Markov \\(n\\)-gram models for the two training texts; I'm going to set \\(n\\) to \\(5\\) (because, heuristically, this value seemed to work well):

<pre class="prettyprint"><code class="language-python">In [13]: dickens_training_stat_text = StatText(dickens_training_text)  

In [14]: dickens_model = dickens_training_stat_text.markov(5)

In [15]: london_training_stat_text = StatText(london_training_text)

In [16]: london_model = london_training_stat_text.markov(5)
In [17]: %paste
def predict_author(text_sample):
    dickens_prob = dickens_model.string_probability(text_sample, log=True)
    london_prob = london_model.string_probability(text_sample, log=True)
    winner = 'Dickens' if dickens_prob >= london_prob else 'London'

    print 'Log Probabilities:'
    print '-'*80
    print 'Dickens:        {}'.format(dickens_prob)
    print 'London:         {}'.format(london_prob)
    print '-'*80
    print 'Prediction:     {}'.format(winner)

## -- End pasted text --
</code></pre>

Now let's try it out on some samples from <code>dickens_testing_text</code> and <code>london_testing_text</code>. We'll do <code>dickens_testing_text</code> first:

<pre class="prettyprint"><code class="language-python">In [18]: predict_author(dickens_testing_text[10000:11000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -2770.08036008
London:         -2968.86657606
--------------------------------------------------------------------------------
Prediction:     Dickens

In [19]: predict_author(dickens_testing_text[20000:21000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -3028.45651154
London:         -3906.15127969
--------------------------------------------------------------------------------
Prediction:     Dickens

In [20]: predict_author(dickens_testing_text[30000:31000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -2821.27034536
London:         -3011.14706353
--------------------------------------------------------------------------------
Prediction:     Dickens

In [22]: predict_author(dickens_testing_text[40000:41000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -3606.7132184
London:         -4154.55552308
--------------------------------------------------------------------------------
Prediction:     Dickens

In [23]: predict_author(dickens_testing_text[50000:51000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -2939.59767322
London:         -3407.584657
--------------------------------------------------------------------------------
Prediction:     Dickens
</code></pre>

Nice! All five tests on a \\(1000\\)-character sample were correctly identified! But, of course, <code>lambda x: 'Dickens'</code> would perform just as well on those particular inputs. So let's look at London too:

<pre class="prettyprint"><code class="language-python">In [24]: predict_author(london_testing_text[10000:11000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -3756.01646651
London:         -3470.92831315
--------------------------------------------------------------------------------
Prediction:     London

In [25]: predict_author(london_testing_text[20000:21000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -3843.20020185
London:         -3774.82346719
--------------------------------------------------------------------------------
Prediction:     London

In [26]: predict_author(london_testing_text[30000:31000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -3294.64847267
London:         -3184.97689782
--------------------------------------------------------------------------------
Prediction:     London

In [27]: predict_author(london_testing_text[40000:41000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -2757.06312231
London:         -2475.24610685
--------------------------------------------------------------------------------
Prediction:     London

In [28]: predict_author(london_testing_text[50000:51000])
Log Probabilities:
--------------------------------------------------------------------------------
Dickens:        -3578.49149961
London:         -2905.8822397
--------------------------------------------------------------------------------
Prediction:     London
</code></pre>

That's a bit more convincing! As a final, fuller test, let's test our method out on \\(50\\) samples of length \\(1000\\) from both texts:

<pre class="prettyprint"><code class="language-python">In [34]: %paste
def predict_author(text_sample):
    dickens_prob = dickens_model.string_probability(text_sample, log=True)
    london_prob = london_model.string_probability(text_sample, log=True)
    winner = 'Dickens' if dickens_prob >= london_prob else 'London'
    return winner

def tally_predictions(large_text_sample, author):
    current_index = 10000
    step_size = 1000

    predictions = {'correct': 0, 'incorrect': 0}
    while current_index < 60000:
        predicted_author = predict_author(large_text_sample[current_index:current_index + step_size])
        if predicted_author == author:
            predictions['correct'] += 1
        else:
            predictions['incorrect'] += 1

        current_index += step_size

    return predictions
## -- End pasted text --

In [35]: london_record = tally_predictions(london_testing_text, 'London')

In [36]: london_record
Out[36]: {'correct': 49, 'incorrect': 1}

In [37]: dickens_record = tally_predictions(dickens_testing_text, 'Dickens')

In [38]: dickens_record
Out[38]: {'correct': 48, 'incorrect': 2}
</code></pre>

So \\(98\%\\) of the London samples were correctly classified, and \\(96\%\\) of the Dickens samples were correctly classified. Remember, the testing texts were never seen by the Markov model during training; we're identifying, e.g., <emph>Oliver Twist</emph> passages as being authored by Dickens because we 'read' <emph>David Copperfield</emph>. I think that the overall result is pretty impressive, especially for such a simple model.
