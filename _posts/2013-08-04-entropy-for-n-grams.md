---
title: "Entropy for N-Grams"
---

#Entropy

Let \\(P\\) be a probability distribution on a finite set \\(\Omega\\). Then the <strong>entropy</strong> of \\(P\\) is defined as \\[H(P) = -\sum\_{\omega \in \Omega} P(\omega)\log\_2(P(\omega)) = \sum\_{\omega \in \Omega} P(\omega)\log\_2(1/P(\omega)) = \mathbb{E}(1/P).\\] The entropy of a random variable \\(X\\) is defined similarly: \\[H(X) = -\sum\_{\omega} \mathbb{P}(X = \omega) \log\_2(\mathbb{P}(X=\omega)) = \mathbb{E}(1/\mathbb{P}(X)).\\]

One sometimes hears that \\(H(X)\\) is supposed to measure the amount of "information" in a sample from \\(X\\). This sort of language can really only be justified if one takes an extremely restricted view on what counts as "information" and on what it means to "communicate information". So instead, here's what I consider a less controversial interpretation of Shannon entropy: Suppose that you express all the possible values of \\(X\\) using a <a href="http://en.wikipedia.org/wiki/Prefix_code">prefix-free</a> binary encoding, and suppose further that you do this in a manner that minimizes the expected length of the encoding of a sample from \\(X\\). Then the expected length of a sample from \\(X\\) is lower-bounded by \\(H(X)\\). In fact, up to discretization (replacing \\(\log\_2(1/P(X=a))\\) with \\(\lceil \log\_2(1/P(X=a))\rceil\\)), the expected length of the sample is exactly \\(H(X)\\); see Theorem 1.3 in Mumford and Desolneux for the exact result. 

Let's make this clearer with a drawn-out example. Suppose that \\(X\\) is a random variable modeling some sort of random emitter; \\(X\\) emits the signal \\(\alpha\\) with probability \\(1/2\\), the signal \\(\beta\\) with probability \\(1/4\\) and the signal \\(\gamma\\) with probability \\(1/4\\). (That is: \\(X\\) is a random variable with range \\(\{\alpha, \beta, \gamma\}\\), and \\(\mathbb{P}(X=\alpha) = 1/2\\), \\(\mathbb{P}(X=\beta) = \mathbb{P}(X = \gamma) = 1/4\\).) We want to encode the output of \\(X\\) using only sequences of the symbols <code>0</code> and <code>1</code>. So, for example, we might try setting \\(\alpha \mapsto 00, \beta \mapsto 001, \gamma \mapsto 100\\). If the stream of outputs of \\(X\\) were \\(\alpha\beta\beta\alpha\gamma\\), we would encode that as <code>0000100100100</code>.

Only this doesn't quite work; look at the beginning of the encoding, i.e. <code>0000100</code>. Is that \\(\alpha\alpha\gamma\\) or is it \\(\alpha\beta\alpha\\)? Our encoding is ambiguous.

To make it unambiguous, we need it to be prefix-free; that means that if we represent \\(\alpha, \beta, \gamma\\) by the <code>0,1</code> strings \\(\sigma\_1, \sigma\_2, \sigma\_3\\) no \\(\sigma\_i\\) is a prefix of some other \\(\sigma\_j\\). The encoding \\(\alpha \mapsto 00, \beta \mapsto 001, \gamma \mapsto 100\\) was not prefix free; <code>00</code> is a prefix of <code>001</code>.

So let's try again: We can use the encoding \\(\alpha \mapsto 0000, \beta \mapsto 0001, \gamma \mapsto 0010\\). This is a prefix-free, and thus unambiguous encoding; you know that <code>0000000100010010</code> is \\(\alpha\beta\beta\gamma\\), and that it couldn't possibly be anything else.

This isn't the most efficient encoding, though. Here, by efficient, I meaning minimizing the expected length of the encoding. In the above, encoding, the expected length of the encoding of a symbol emitted by \\(X\\) is: \\[\frac{1}{4}4 + \frac{1}{4}4 + \frac{1}{2}4 = 1.\\] We could make the expected length shorter by using the following encoding instead: \\(\alpha \mapsto 00, \beta \mapsto 01, \gamma \mapsto 10\\). Now the expected length is \\[\frac{1}{4}2 + \frac{1}{4}2 + \frac{1}{2}2 = 2.\\] This is better, but it still isn't the most efficient encoding; a more efficient encoding is given by \\(\alpha \mapsto 0, \beta \mapsto 10, \gamma \mapsto 11\\). Now the expected length is \\[\frac{1}{4}2 + \frac{1}{4}2 + \frac{1}{2}1 = \frac{3}{2}.\\] (The reason it's better to map \\(\alpha\\) to \\(0\\) instead of, say, mapping \\(\beta\\) to \\(0\\) is that \\(X\\) emits \\(\alpha\\) with higher probability.) In fact, you can't do any better than this encoding; no other encoding scheme is expected to be shorter.

This statement, and its aforementioned generalization, deserve a proof. In an act of gross negligence, though, we're not going to provide one here; please refer to the excellent book by Mumford and Desolneux. (The intuition is that if a symbol occurs with probability \\(p\\), then it will get an encoding of length roughly \\(\log\_2(1/p)\\).)

However, if we're willing to take on faith that \\(H(X)\\) is a lower bound for the expected length of the most efficient encoding of \\(X\\), then we can check directly that \\(\alpha \mapsto 0, \beta \mapsto 10, \gamma \mapsto 11\\) is an optimal encoding: What's the expected length of the encoding of a sample from \\(X\\)? We've already computed it to be \\(3/2\\). And what's \\(H(X)\\)? It's \\[\frac{1}{4}\log\_2(4) + \frac{1}{4}\log\_2(4) + \frac{1}{2}\log\_2(2) = \frac{3}{2}.\\]

#Entropy for n-grams
Let's apply the definition of entropy to some English text. We'll imagine that length-\\(n\\) substrings (called "\\(n\\)-grams") of a text sample were produced from a random emitter, and then count the number of occurrences of these substrings to estimate the "probability" that our emitter produces them.

For example, let's take this snippet from <em>Beyond Good and Evil</em>:
<blockquote>Having kept a sharp eye on philosophers, and having read between their lines long enough, I now say to myself that the greater part of conscious thinking must be counted among the instinctive functions, and it is so even in the case of philosophical thinking; one has here to learn anew, as one learned anew about heredity and "innateness." As little as the act of birth comes into consideration in the whole process and procedure of heredity, just as little is "being-conscious" OPPOSED to the instinctive in any decisive sense; the greater part of the conscious thinking of a philosopher is secretly influenced by his instincts, and forced into definite channels. And behind all logic and its seeming sovereignty of movement, there are valuations, or to speak more plainly, physiological demands, for the maintenance of a definite mode of life. For example, that the certain is worth more than the uncertain, that illusion is less valuable than "truth" such valuations, in spite of their regulative importance for US, might notwithstanding be only superficial valuations, special kinds of niaiserie, such as may be necessary for the maintenance of beings such as ourselves. Supposing, in effect, that man is not just the "measure of things."</blockquote>
&nbsp;

Let's simplify our job by removing all punctuation, and making everything lowercase:

~~~
def simplify_text(text):
    text = text.replace('-', ' ')
    text = text.translate(string.maketrans('',''), string.punctuation)
    text = text.lower()
    return text
~~~

The result is:
<blockquote>having kept a sharp eye on philosophers and having read between their lines long enough i now say to myself that the greater part of conscious thinking must be counted among the instinctive functions and it is so even in the case of philosophical thinking one has here to learn anew as one learned anew about heredity and innateness as little as the act of birth comes into consideration in the whole process and procedure of heredity just as little is being conscious opposed to the instinctive in any decisive sense the greater part of the conscious thinking of a philosopher is secretly influenced by his instincts and forced into definite channels and behind all logic and its seeming sovereignty of movement there are valuations or to speak more plainly physiological demands for the maintenance of a definite mode of life for example that the certain is worth more than the uncertain that illusion is less valuable than truth such valuations in spite of their regulative importance for us might notwithstanding be only superficial valuations special kinds of niaiserie such as may be necessary for the maintenance of beings such as ourselves supposing in effect that man is not just the measure of things</blockquote>
&nbsp;

Let's start with the 1-grams: We need to count the number of times each character ('a' through 'z', along with the space character ' ') occurs. Python's <code>collections</code> module makes this easy:

~~~
In [1]: from collections import Counter

In [2]: length_1_substrings = [text[i:i+1] for i in range(len(text)-1)]

In [3]: length_1_substring_counts = Counter(length_1_substrings)

In [4]: length_1_substring_counts
Out[4]: Counter({' ': 203, 'e': 122, 'n': 96, 'i': 93, 't': 90, 's': 80, 'a': 76, 'o': 75, 'h': 53, 'r': 45, 'l': 39, 'c': 37, 'u': 28, 'f': 27, 'd': 26, 'p': 24, 'g': 21, 'm': 19, 'v': 14, 'y': 14, 'b': 11, 'w': 7, 'k': 6, 'j': 2, 'x': 1})
~~~

Great! Now we have a probability distribution (after normalizing), so let's compute the entropy of 1-grams:

~~~
In [5]: normalizer = sum(length_1_substring_counts.values())

In [6]: length_1_probabilities = dict((k, float(v)/normalizer) for k,v in length_1_substring_counts.items())

In [7]: length_1_probabilities
Out[7]: {' ': 0.16790736145574855,
 'a': 0.06286186931348221,
 'b': 0.009098428453267164,
 'c': 0.030603804797353185,
 'd': 0.021505376344086023,
 'e': 0.10090984284532671,
 'f': 0.022332506203473945,
 'g': 0.017369727047146403,
 'h': 0.043837882547559964,
 'i': 0.07692307692307693,
 'j': 0.0016542597187758478,
 'k': 0.004962779156327543,
 'l': 0.03225806451612903,
 'm': 0.015715467328370553,
 'n': 0.0794044665012407,
 'o': 0.062034739454094295,
 'p': 0.019851116625310174,
 'r': 0.03722084367245657,
 's': 0.06617038875103391,
 't': 0.07444168734491315,
 'u': 0.02315963606286187,
 'v': 0.011579818031430935,
 'w': 0.005789909015715467,
 'x': 0.0008271298593879239,
 'y': 0.011579818031430935}

In [8]: entropy = -sum(p*np.log2(p) for p in length_1_probabilities.values())

In [9]: entropy
Out[9]: 4.05801165066
</code></pre>
We can compute the entropy of 2-grams (length-\\(2\\) subsequences) similarly:
<pre class="prettyprint"><code class="language-python">In \[10]: length_2_substrings = \[text\[i:i+2] for i in range(len(text)-2)]

In [11]: length_2_substring_counts = Counter(length_1_substrings)

In [12]: normalizer = sum(length_2_substring_counts.values())

In [13]: length_2_probabilities = dict((k, float(v)/normalizer) for k,v in length_2_substring_counts.items())

In [14]: -sum(p*np.log2(p) for p in length_2_probabilities.values())
Out[14]: 7.11390041737
~~~

Let's write \\(H\_n\\) for the entropy of \\(n\\)-grams in our text. We've already computed \\(H\_1 \approx 4.06\\) and \\(H\_2 \approx 7.11\\). Let's compute these values up to \\(H\_{10}\\):

\\[
\begin{array}{c\|c}
H\_1 & 4.06 \\\
H\_2 & 7.11 \\\
H\_3 & 8.76 \\\
H\_4 & 9.41 \\\
H\_5 & 9.74 \\\
H\_6 & 9.91 \\\
H\_7 & 10.01 \\\
H\_8 & 10.05 \\\
H\_9 & 10.09 \\\
H\_{10} & 10.12
\end{array}
\\]

Notice that the difference between \\(H\_5\\) and \\(H\_{10}\\) is only about \\(0.38\\). Intuitively, this means that once we know the first \\(5\\) characters of a length-\\(10\\) substring, then there is little uncertainty about the remaining \\(5\\) characters. This is easier to see if we use a simpler passage of text. For example, let's look at the lyrics to Daft Punk's <em>Around the World</em>:
<blockquote>around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world around the world</blockquote>
&nbsp;

The values of \\(H\_n\\) for this text are:

\\[
\begin{array}{c\|c}
H\_1 & 3.46 \\\
H\_2 & 3.97 \\\
H\_3 & 4.09 \\\
H\_4 & 4.09 \\\
H\_5 & 4.09 \\\
H\_6 & 4.09 \\\
H\_7 & 4.09 \\\
H\_8 & 4.09 \\\
H\_9 & 4.09 \\\
H\_{10} & 4.09 \\\
\end{array}
\\]

To see the situation even more clearly, imagine a "random" emitter \\(X\\) that emits \\(\alpha\\), then \\(\beta\\), then \\(\alpha\\), then \\(\beta\\), then \\(\alpha\\), ... forever. For any \\(n\\), there are only two length-\\(n\\) substrings that appear in the output of \\(X\\), both with probability \\(1/2\\). So \\(H\_n = \log\_2(2)/2 + \log\_2(2)/2 = 1\\) for \\(X\\). Once we know which character \\(X\\) emitted at time \\(t\\), we know exactly what \\(X\\) will emit at time \\(t+1\\); there is no "information gain" from reading the next emission of \\(X\\). To express this more clearly, we need the notion of conditional entropy.

#Conditional Entropy

Suppose that you have two random variables \\(X\\) and \\(Y\\) with finite images \\(\Omega\_X\\) and \\(\Omega\_Y\\) respectively. We define the <strong>conditional entropy</strong> of \\(X\\) given \\(Y\\) to be:
\\[H(X \mid Y) = \sum\_{y \in \Omega\_Y} \mathbb{P}(Y=y) H(X \mid Y = y)\\]
\\[= \sum\_{y \in \Omega\_Y} \mathbb{P}(Y=y) \sum\_{x \in \Omega\_X} \mathbb{P}(X=x \mid Y = y)\log\_2(1/\mathbb{P}(X=x \mid Y = y))\\]
\\[= \sum\_{x,y} \mathbb{P}(X=x, Y=y) \log\_2\left(\frac{\mathbb{P}(X=x)}{\mathbb{P}(X=x, Y=y)}\right)\\]

A short computation yields the 'chain rule' for conditional entropies:

\\[H(X,Y) = H(X \mid Y) + H(Y)\\]

(Here \\(H(X,Y)\\) is shorthand for \\(H((X,Y))\\).) Intuitively, \\(H(X \mid Y)\\) is supposed to be the average number of bits needed to encode \\(X\\) once \\(Y\\) is known.

#Back to n-grams

Let \\(\sigma = a\_1 a\_2 \cdots a\_n\\) be a string of length \\(n\\) drawn from our text sample. For each letter \\(a\\) in our alphabet, we have a probability \\[\mathbb{P}(a\_1 a\_2\cdots a\_n a \mid a\_1 a\_2 \cdots a\_n) := \frac{\mathbb{P}(a\_1 a\_2 \cdots a\_n a)}{\mathbb{P}(a\_1 a\_2 \cdots a\_n)}.\\] This is often abbreviated as \\(\mathbb{P}(a \mid a\_1 \cdots a\_n)\\). We're interested in the entropy of this distribution. Or, if one prefers: Let \\(Y\\) the \\(n\\)-gram random variable furnished by our text, and let \\(X\\) be the \\((n+1)\\)-gram random variable. Then we're interested in the entropy of \\(Z = (X \mid Y = a\_1\cdots a\_n)\\).

Let's compute this for some samples from <em>Beyond Good and Evil</em>:

~~~
In [97]: def conditional_probabilities(k, prefix, text):
    n = k + len(prefix)
    length_n_substrings = \[text\[i:i+n] for i in range(len(text)-n)]
    prefix_matching_substrings = \[x for x in length_n_substrings if x\[0:len(prefix)] == prefix]
    counts = collections.Counter(prefix_matching_substrings)
    normalizer = sum(counts.values())
    return dict((k, float(v)/normalizer) for k,v in counts.items())

In [98]: conditional_probabilities(1, 'philo', nietzsche)
Out[98]: {'philos': 1.0}

In [99]: conditional_probabilities(5, 'philo', nietzsche)
Out[99]: {'philosophe': 0.6666666666666666, 'philosophi': 0.3333333333333333}

In [100]: -sum(p*np.log2(p) for p in conditional_probabilities(1, 'philo', nietzsche).values())
Out[100]: -0

In [101]: -sum(p*np.log2(p) for p in conditional_probabilities(5, 'philo', nietzsche).values())
Out[101]: 0.91829583405448956
~~~

One can interpret the entropy of \\(X \mid Y = a\_1 \cdots a\_n\\) as the number of additional bits one needs to encode \\(X\\) given that \\(Y = a\_1\cdots a\_n\\). (Again, and as always, assuming an optimal coding and ignoring discretization.) So what the numbers above are saying is that given that a string starts with 'philo', one doesn't need any additional bits to encode the next character; but one does need (an expected) <code>0.92</code> bits to encode the next character.

We can also compute the conditional entropy \\(H(X mid Y)\\). (Remember, our running notational assumption is that \\(X\\) is the \\((n+1)\\)-gram r.v. and \\(Y\\) is the \\(n\\)-gram r.v. built from our text.) We can use the chain rule to our profit here:

\\[H(X \mid Y) = H(X,Y) - H(Y) = H(X) - H(Y).\\]

(How does one know that \\(H(X,Y) = H(X)\\)? You can do it via direct computation, but a much slicker way is to use <a href="http://en.wikipedia.org/wiki/Coupling_(probability)">coupling</a>; remember that \\(X\\) is an extension of \\(Y\\).) 

![image](http://newdatascientist.com/blog/wp-content/uploads/2013/07/Screen-Shot-2013-07-27-at-1.29.34-AM.png)

So for \\(n &gt; 1\\), the conditional entropy of the \\((n+1)\\)-gram model given the \\(n\\)-gram one is just \\(H\_{n+1} - H\_n\\). It is traditional to write \\(F\_n = H\_n - H\_{n-1}\\), with the convention that \\(F\_1 = H\_1\\). Earlier, we noticed that for both <em>Beyond Good and Evil</em> and for <em>Around the World</em>, the values of \\(H\_n\\) seemed to stabilize; with conditional entropy, one can re-express this by saying that the values of \\(F\_n\\) started tending towards zero.

#Repeating Shannon's English Language Computation

In his classic paper <a href="http://languagelog.ldc.upenn.edu/myl/Shannon1950.pdf"><em>Prediction and Entropy of Written English</em></a>, Shannon made the following estimates for \\(F\_i\\) on the English language as a whole:

\\[F\_1 \approx 4.75, \quad F\_2 \approx 4.03, \quad F\_3 \approx 3.1, \quad F\_4 \approx 2.8\\]

Since \\(H\_n = \sum\_{i=1}^n F\_i\\), these estimates imply the following ones for the \\(H\_n\\) (and vice versa):

\\[H\_1 \approx 4.75, \quad H\_2 \approx 7.35, \quad H\_3 \approx 10.45, \quad H\_4 \approx 13.25.\\]

Let's see if we can arrive at similar figures by using Peter Norvig's <a href="http://norvig.com/big.txt">big.txt</a>. First, we're going to need to do some text munging:

~~~
In[1]: def clean_text(text):
    delete_punctuation_table = dict((c, None) for c in xrange(sys.maxunicode)
                                    if unicodedata.category(unichr(c)).startswith('P'))
    delete_digits_table = dict((c, None) for c in xrange(sys.maxunicode)
                               if unicodedata.category(unichr(c)).startswith('N'))
    delete_signs_table = dict((c, None) for c in xrange(sys.maxunicode)
                              if unicodedata.category(unichr(c)).startswith('S'))

    no_punctuation = text.translate(delete_punctuation_table)
    no_digits = no_punctuation.translate(delete_digits_table)
    no_signs = no_digits.translate(delete_signs_table)
    single_spaces = ' '.join(no_signs.split())
    lowercase = single_spaces.lower()

    return str(lowercase)
~~~

Now let's fetch big.txt and clean it. Just to be safe, let's also double check that every character in the resulting text is in 'abcdefghijklmnopqrstuvwxyz ' (lowercase letters and the space string):

~~~
In [2]: big_text = requests.get('http://norvig.com/big.txt').text

In [3]: clean_big_text = clean_text(big_text)

In [4]: for c in clean_big_text:
    if c not in string.lowercase + ' ':
        print c

In [5]: clean_big_text\[:1000]
Out[5]: 'the project gutenberg ebook of the adventures of sherlock holmes by sir arthur conan doyle in our series by sir arthur conan doyle copyright laws are changing all over the world be sure to check the copyright laws for your country before downloading or redistributing this or any other project gutenberg ebook this header should be the first thing seen when viewing this project gutenberg file please do not remove it do not change or edit the header without written permission please read the legal small print and other information about the ebook and project gutenberg at the bottom of this file included is important information about your specific rights and restrictions in how the file may be used you can also find out about how to make a donation to project gutenberg and how to get involved welcome to the world of free plain vanilla electronic texts ebooks readable by both humans and by computers since these ebooks were prepared by thousands of volunteers title the adventures of sherloc'
~~~

The text seems to look alright. So let's compute its entropy!

~~~
In[6]: def entropy(n_gram_lengths, text):
    entropies = {}
    for n in n_gram_lengths:
        length_n_substrings = \[text\[i:i+n] for i in range(len(text)-n)]
        length_n_substring_counts = collections.Counter(length_n_substrings)
        normalizer = float(len(text) - n)

        distribution_values = np.array(length_n_substring_counts.values()) / normalizer
        entropy = -np.sum(p*np.log2(p) for p in distribution_values)

        entropies\[n] = entropy

    return entropies

In [7]: entropy([1, 2, 3, 4], clean_big_text)
Out[7]: 
{1: 4.1036633920946164,
 2: 7.4632277520451424,
 3: 10.227874216807905,
 4: 12.431460731880753}
~~~

Our numbers are close, but not identical with, Shannon's:

\\[\begin{array}{c\|cc}
& \text{Us} & \text{Shannon} \\\
\hline \\\
H\_1 & 4.10 & 4.75 \\\
H\_2 & 7.46 & 7.35 \\\
H\_3 & 10.22 & 10.45 \\\
H\_4 & 12.43 & 13.25 \\\ 
\end{array}\\]

<h1>A Fun Application</h1>

Here's a toy application of entropy calculations in cryptography: Suppose that Alice scrambles a message using a simple substitution cipher:

~~~
In [91]: cipher_range = [c for c in string.lowercase + string.uppercase]

In [92]: random.shuffle(cipher_range)

In [93]: cipher = dict((k, cipher_range[i]) for i,k in enumerate([c for c in string.lowercase]))

In [94]: cipher[' '] = ' '

In [95]: cipher
Out[95]: 
{' ': ' ',
 'a': 'D',
 'b': 't',
 'c': 'E',
 'd': 'C',
 'e': 'I',
 'f': 'Q',
 'g': 'p',
 'h': 'n',
 'i': 'J',
 'j': 'u',
 'k': 'q',
 'l': 'W',
 'm': 'H',
 'n': 'M',
 'o': 'w',
 'p': 'G',
 'q': 'k',
 'r': 'y',
 's': 'l',
 't': 'O',
 'u': 'g',
 'v': 's',
 'w': 'j',
 'x': 'Z',
 'y': 'o',
 'z': 'i'}

In [96]: nietzsche
Out[96]: 'having kept a sharp eye on philosophers and having read between their lines long enough i now say to myself that the greater part of conscious thinking must be counted among the instinctive functions and it is so even in the case of philosophical thinking one has here to learn anew as one learned anew about heredity and innateness as little as the act of birth comes into consideration in the whole process and procedure of heredity just as little is being conscious opposed to the instinctive in any decisive sense the greater part of the conscious thinking of a philosopher is secretly influenced by his instincts and forced into definite channels and behind all logic and its seeming sovereignty of movement there are valuations or to speak more plainly physiological demands for the maintenance of a definite mode of life for example that the certain is worth more than the uncertain that illusion is less valuable than truth such valuations in spite of their regulative importance for us might notwithstanding be only superficial valuations special kinds of niaiserie such as may be necessary for the maintenance of beings such as ourselves supposing in effect that man is not just the measure of things'

In [97]: ciphered_nietzsche = ''.join(cipher\[c] for c in nietzsche)

In [98]: ciphered_nietzsche
Out[98]: 'nDsJMp qIGO D lnDyG IoI wM GnJWwlwGnIyl DMC nDsJMp yIDC tIOjIIM OnIJy WJMIl WwMp IMwgpn J Mwj lDo Ow HolIWQ OnDO OnI pyIDOIy GDyO wQ EwMlEJwgl OnJMqJMp HglO tI EwgMOIC DHwMp OnI JMlOJMEOJsI QgMEOJwMl DMC JO Jl lw IsIM JM OnI EDlI wQ GnJWwlwGnJEDW OnJMqJMp wMI nDl nIyI Ow WIDyM DMIj Dl wMI WIDyMIC DMIj DtwgO nIyICJOo DMC JMMDOIMIll Dl WJOOWI Dl OnI DEO wQ tJyOn EwHIl JMOw EwMlJCIyDOJwM JM OnI jnwWI GywEIll DMC GywEICgyI wQ nIyICJOo uglO Dl WJOOWI Jl tIJMp EwMlEJwgl wGGwlIC Ow OnI JMlOJMEOJsI JM DMo CIEJlJsI lIMlI OnI pyIDOIy GDyO wQ OnI EwMlEJwgl OnJMqJMp wQ D GnJWwlwGnIy Jl lIEyIOWo JMQWgIMEIC to nJl JMlOJMEOl DMC QwyEIC JMOw CIQJMJOI EnDMMIWl DMC tInJMC DWW WwpJE DMC JOl lIIHJMp lwsIyIJpMOo wQ HwsIHIMO OnIyI DyI sDWgDOJwMl wy Ow lGIDq HwyI GWDJMWo GnolJwWwpJEDW CIHDMCl Qwy OnI HDJMOIMDMEI wQ D CIQJMJOI HwCI wQ WJQI Qwy IZDHGWI OnDO OnI EIyODJM Jl jwyOn HwyI OnDM OnI gMEIyODJM OnDO JWWglJwM Jl WIll sDWgDtWI OnDM OygOn lgEn sDWgDOJwMl JM lGJOI wQ OnIJy yIpgWDOJsI JHGwyODMEI Qwy gl HJpnO MwOjJOnlODMCJMp tI wMWo lgGIyQJEJDW sDWgDOJwMl lGIEJDW qJMCl wQ MJDJlIyJI lgEn Dl HDo tI MIEIllDyo Qwy OnI HDJMOIMDMEI wQ tIJMpl lgEn Dl wgylIWsIl lgGGwlJMp JM IQQIEO OnDO HDM Jl MwO uglO OnI HIDlgyI wQ OnJMpl'
~~~

Now for the fun: Suppose further that Alice decides to pepper the text with a random character, in order to throw off frequency analysis. The letter 'r' is not used anywhere in the range of the encrypted text above, so let's suppose that Alice randomly inserts 'r' into various places in the text:

~~~
In [109]: ciphered_nietzsche_chars = [c for c in ciphered_nietzsche]

In [110]: for i in range(30):
   .....:     random_insert = random.randint(0, len(ciphered_nietzsche_chars)]
   .....:     ciphered_nietzsche_chars.insert(random_insert, 'r')

In [111]: inserted_nietzsche = ''.join(ciphered_nietzsche_chars)

In [112]: inserted_nietzsche
Out[112]: 'nDrsJMp qIGO D lnDyG IoI wM GnJrWwlwGnIyl DMC nrDsJMp yIDC tIOrjIIM OnIJy WJMIrl WwMp IMwgpn J Mwrj lDo Ow HolIWQ OnDO OnI pyIDOIy GDyO wQr EwMlEJwgl OnJMqJMrp HglO tI EwgMOIC DHwMp OrnI JMlOJMEOJsI QgMEOJwMl DMC JO Jl lw IsIM JM OnI EDlI wQ GnJWwlwGnJEDW OnJMqJMp wMI nDl nIyI Ow WrIDyM DMIj Dl wMI WIDyMIC DMIj DtwgO nIyIrCJOor DMC JMMDOrIMIll Dl WJOOWI Dl OnI DEO wQ tJyOn EwHIl JMOw EwMlrJCIyDOJwM JM OnrI jnwWI GywEIll DMC GywEICgyI wQ nIyICJOo uglO Dl WJOOWI Jl tIJMpr EwMlEJwgl wGGwlIC Ow OnI JMlOJMEOJsI JM DMo CIEJlJsrI lIMlI OnI pyIDOIy GDyO wQ OnI EwMlErJwgl OnJMqJMp wQ D GnJWwlwGnIy Jl lIEyIOWo JMQWgrIMEIC to nJl JMlOJMEOl DMC QwyEIC JMOw CIQJMJOI EnDMMIWl DMC tInJMC DWW WwpJE DMC JOl lIIHJMp lwsrIyIJpMOo wQ HwsrIHIMO OnIyI DyI sDWgDOJwMl wy Orw lGIDq HwyI GWDJMWo GnolJwWwpJEDW CIHDMCl Qwy OnI HDJMOIMDrMEI wQ D CIQJMJOI HwCI wQ WJQI Qwy IZDHGWI OnDO OnIr EIyODJM Jl jwyOn HwyI OnDM OnI gMEIyODJM OnDO JWWglJwM Jl WIll sDWgDtWI OnDM OygOn lgEn srDWgDOJwMl JM lGJOI wQ OnIJy yIpgWDOJsI JHrGwyODMEI Qwy gl HJpnO MwOjJOnlODMCJMp tI wMWo lgGIyQJEJrDW sDWgDOJwMl lGIEJDW qJMCl wQ MJDJlIyJI lgErn Dl HDo tI MIEIllDyo Qwy OnI HDJMOIMDMEI wQ tIJMpl lgEn Dl wgylIWsIl lgGGwlJMp JM IQQIEO OnDO HDM Jl MwO uglO rOnI HIDlgryI wQ OnJMpl'
~~~

Now suppose that we, Eve, intercept this text, and that we know about Alice's scheme. What we'd like to do is figure out which character is the nonsense one (since then we can remove it and decrypt the remaining text using frequency analysis). Entropy can help us: If \\(c\\) is the nonsense character, then the distribution of characters that occur immediately after \\(c\\) should be the same as the distribution of \\(1\\) grams; so, letting \\(X\\) be the \\(2\\)-gram random variable, we should look for the character \\(c\\) such that \\(H(X \mid c) \approx H_1\\). Let's try it! First we compute that \\(H\_1 \approx 4.12\\). Then we work as follows:

~~~
In [118]: conditional_distros = dict()

In [119]: for c in inserted_nietzsche:
    relevant_substrings = \[inserted_nietzsche\[i:i+2] for i in range(len(inserted_nietzsche) - 2) if inserted_nietzsche\[i] == c]
    counts = collections.Counter(relevant_substrings)
    normalizer = sum(counts.values())       
    probs = dict((k, float(v)/normalizer) for k,v in counts.items())
    conditional_distros\[c] = probs
   .....: 

In [120]: entropies = dict()

In [121]: for k in conditional_distros:
    entropies\[k] = -np.sum(p*np.log2(p) for p in conditional_distros\[k].values())
   .....:  

In [122]: entropies
Out[122]: 
{' ': 3.9919105737129881,
 'C': 1.7939923560142899,
 'D': 3.3130196578979176,
 'E': 2.8484196781616191,
 'G': 3.0365766899407327,
 'H': 2.6703397002311071,
 'I': 3.3896122709458085,
 'J': 2.9627249567996148,
 'M': 3.5530078672922678,
 'O': 3.0080588691771255,
 'Q': 2.2930266017029326,
 'W': 3.25884933579411,
 'Z': -0,
 'g': 3.0391486719030705,
 'j': 2.1280852788913944,
 'l': 2.7927417755930053,
 'n': 2.4349278460745629,
 'o': 1.2916919971380596,
 'p': 2.3609640474436815,
 'q': 1.2516291673878228,
 'r': 3.6266781970562949,
 's': 1.9241743523004413,
 't': 1.6729330318733675,
 'u': -0,
 'w': 3.3723172682565332,
 'y': 2.8813169999429058}

In [123]: sorted_entropies = Series(entropies)

In [124]: sorted_entropies.sort()

In [125]: sorted_entropies
Out[125]: 
Z   -0.000000
u   -0.000000
q    1.251629
o    1.291692
t    1.672933
C    1.793992
s    1.924174
j    2.128085
Q    2.293027
p    2.360964
n    2.434928
H    2.670340
l    2.792742
E    2.848420
y    2.881317
J    2.962725
O    3.008059
G    3.036577
g    3.039149
W    3.258849
D    3.313020
w    3.372317
I    3.389612
M    3.553008
r    3.626678
     3.991911
dtype: float64
~~~

It worked! Amongst the non-space characters, 'r' had the conditional entropy closest to \\(H\_1 \approx 4.12\\). So using our method, we would correctly conclude that 'r' is the nonsense character. To be fair: We were a little lucky. The character with the second highest entropy, 'M', was pretty close to 'r'. Perhaps to be slightly safer with this method, one ought to consider the top 2 or 3 closest entropy-characters as contenders. Still, it's satisfying to see that the method is sound, and that this approach actually works!
