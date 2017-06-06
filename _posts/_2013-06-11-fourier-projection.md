---
title: "Fourier Projection"
---

# So You Want to Fourier-Approximate a Sequence...

This is a tutorial about using Fourier analysis to approximate a finite sequence of values; this has practical applications for forecasting time series that have a periodic component (e.g. weekly, monthly, or annual effects).

# Explanation (Theory)

(Warning: Some linear algebra to follow.) The big secret to finite Fourier analysis is that it's just linear algebra. (At least before one starts getting cute by moving into representation theory or something like that.) The setup is as follows: You have a function from a finite set \\(S\\) into the complex numbers: \\(f:S \to \mathbb{C}\\). Without loss of generality, we'll let \\(S\\) be the size-\\(N\\) set of integers from \\(0\\) to \\(N-1\\) for some \\(N\\). Going slightly further, we'll make our lives even easier by replacing \\(\{0, 1, \ldots, N-1\}\\) with \\(\mathbb{Z}/N\\), the integers modulo \\(N\\). (The convenience here is in being able to refer to \\(f(-n)\\).)

The sequence \\(\\{a\_k\\}\_{k=0}^{N-1}\\) that we want to Fourier-approximate is a function \\(f:\mathbb{Z}/N \to \mathbb{C}\\); we just let \\(f(k) = a\_k\\). So, for example, if our sequence is `[1,2,3,2]`, then the related function \\(f\\) is given by \\(f(0) = 1, f(1) = 2, f(2) = 3, f(3) = 2\\). So we can talk about sequences \\(\{a\_0, \ldots, a\_{N-1}\}\\), and functions \\(f:\mathbb{Z}/N \to \mathbb{C}\\) interchangeably.

Let's look at the set of <em>all</em> functions \\(f:\mathbb{Z}/N \to \mathbb{C}\\); we'll call this set \\(V\\). We have two big observations to make about \\(V\\). <strong>First:</strong> \\(V\\) is a vector space over \\(\mathbb{C}\\). Just define \\((\alpha f + \beta g)(k) = \alpha f(k) + \beta g(k)\\). Since \\(\mathbb{Z}/N\\) is a finite set, \\(V\\) is finite dimensional. To get a basis, we can use the indicator functions \\(\chi\_m\\), where 

\\[\chi\_m(k) = \left\\{ \begin{array}{ll} 1 & \text{if } m = k, \\\ 0 & \text{if } m \neq k \end{array} \right. \\] 

This may seem a little strange, so here's an example: Let \\(f:\mathbb{Z}/4 \to \mathbb{C}\\) be given by the rule \\(f(k) = k^2 + 1\\). Then \\(f(0) = 1, f(1) = 2, f(2) = 5, f(3) = 10\\), and thus: \\[f = \chi\_0 + 2\chi\_1 + 5\chi\_2 + 10\chi\_3.\\] (So under the basis \\(\{\chi\_m\}\\), \\(f\\) is the vector \\((1,2,5,10)\\). Notice that this is just the sequence of \\(f\\)'s values.)

<strong>Second:</strong> \\(V\\) is not just a vector space, it's an [inner product space](https://en.wikipedia.org/wiki/Inner_product_space)inner product space. The inner product is given by \\[\langle f, g \rangle = \frac{1}{N}\sum\_{k \in \mathbb{Z}/N} f(k)\overline{g(k)}\\] (Example / Exercise: If \\(N = 3, f(k) = k^2, g(k) = 3k + i\\), then \\(\langle f, g \rangle = (27 - 5i)/3\\).) Like any finite-dimensional inner product space, there is an [orthonormal basis](http://en.wikipedia.org/wiki/Orthonormal_basis). The one I have in mind is \\(\epsilon\_0, \epsilon\_1, \ldots, \epsilon\_{N-1}\\) where \\[\epsilon\_k(m) = e^{\frac{-2\pi i k m}{N}}.\\] (That's \\(\exp(-2 \pi i k m / N)\\) for those of you in the back row.)

Armed with these two observations, we can define the Fourier transform. Given a function \\(f:\mathbb{Z}/N \to \mathbb{C}\\), we denote it's kth Fourier coefficient by \\(\widehat{f}(k)\\) and define it by the rule \\[\widehat{f}(k) = \langle f, \epsilon\_k \rangle.\\] So the kth Fourier coefficient is nothing more than the inner product of \\(f\\) and \\(\epsilon\_k\\). You may have seen the coefficient defined by the rule \\[\widehat{f}(k) = \frac{1}{N}\sum\_{m=0}^{N-1} f(k)e^{-2\pi i km / N},\\] which is just \\( \langle f, \epsilon\_k \rangle\\) expanded out.  Notice that \\(\langle f, \epsilon\_m \rangle \epsilon\_m\\) is just the <em>projection</em> of \\(f\\) onto \\(\epsilon\_m\\). So you should see the taking of Fourier coefficients as no different than [normal linear projection](http://en.wikipedia.org/wiki/Projection_(linear_algebra).

As for the Fourier transform \\(T(f)\\) of \\(f\\), it's the function \\(T(f):\mathbb{Z}/N \to \mathbb{C}\\) given by the rule \\[T(f)(k) = \widehat{f}(k).\\] So \\(T(f)\\) just takes \\(k\\) to the \\(k^{\text{th}}\\) Fourier coefficient of \\(f\\). We define the "inverse Fourier transform", denoted \\(T^{-1}\\),  by the rule \\[T^{-1}(g)(m) = \sum\_{k \in \mathbb{Z}/N} g(k)\epsilon\_k(m).\\]<em>
</em>

Now it's time to notice something interesting: Remember that \\(\{\epsilon\_k\}\\) is an orthonormal basis. Since any vector is the sum of it's orthogonal projections, we know that for any function \\(f:\mathbb{Z}/N \to \mathbb{C}\\) we have the identity \\[f = \sum\_{k \in \mathbb{Z}/N} \langle f, \epsilon\_k \rangle \epsilon\_k.\\] That is, \\[f = \sum\_{k \in \mathbb{Z}/N}T(f)(k) \epsilon\_k = T^{-1}(T(f)),\\] so \\(T^{-1} \circ T\\) is the identity and thus \\(T^{-1}\\) lives up to its name. This result is sometimes called the [fourier inversion theorem](http://en.wikipedia.org/wiki/Fourier_inversion_theorem#Properties_of_inverse_transform); but for finite Fourier analysis, it's just a statement about orthogonal bases!

Let's look at an example. Suppose that \\(N = 6\\) and we're interested in the sequence <code>[1,2,3,4,3,2]</code> The first Fourier coefficient, \\(\widehat{f}(0)\\), is given by \\[\frac{1}{6}\sum\_{k=0}^5 f(k)e^{-2\pi i 0 k /N} = \frac{1}{6}\sum\_{k=0}^5 f(k) = (1 + 2 + 3 + 4 + 3 + 2)/6 = 15/6.\\] Similarly, the coefficients \\(\widehat{f}(1), \widehat{f}(2), \widehat{f}(3), \widehat{f}(4), \widehat{f}(5)\\) are \\(-2/3, 0, -1/6, 0, -2/3\\). Let's put these values into a list: <code>[5/2, -2/3, 0, -1/6, 0, -2/3]</code>. The Fourier inversion theorem tells us that \\[f(m) = \sum\_{k \in \mathbb{Z}/N}T(f)(k) \epsilon\_k(m),\\] and indeed we see that, e.g., \\[2 = \frac{5}{2} e^{-2\pi i0\cdot 1/6} - \frac{2}{3} e^{-2\pi i1 \cdot 1/6} - \frac{1}{6}e^{-2\pi i 1 \cdot 3/6} -\frac{2}{3}e^{-2\pi i 1 \cdot 5/6}.\\]

Let's leave the theory at that. There was one miracle that occurred during the explanation: The choice of the basis \\(\{\epsilon\_k\}\\). It's not clear why we made that particular choice of basis. The intuition is that \\(\epsilon\_k\\) is a function that 'goes around' the unit circle; the larger \\(k\\) is, the faster the function 'goes around'. The idea is that the projection onto \\(\epsilon\_k\\) picks out the 'rate \\(k\\) periodic part' of \\(f\\). This explanation is almost hopelessly vague, but I hope that it at least gives some idea about why \\(\epsilon\_k\\) is the 'obvious' choice (there are no miracles in mathematics; everything is 'obvious' once you've spent enough time groping around for understanding). For a more detailed account, see [Stein's excellent book]("http://www.amazon.com/dp/069111384X">Stein's excellent book).

(P.S. You may have noticed that the Fourier transform \\(T\\) is a linear operation; as such, it is given by a matrix \\(M\\) relative to the bases \\(\{\chi\_k\}\\) and \\(\{\epsilon\_k\}\\). In fact, this matrix is given by \\[M = \left\[\begin{array}{ccccc}1 & 1 & 1 & \cdots & 1 \\\ 1 & \omega & \omega^2 & \cdots & \omega^{N-1} \\\ 1 & \omega^2 & \omega^4 & \cdots & \omega^{2(N-1)} \\\ \vdots & \vdots & \vdots & \ddots & \vdots \\\ 1 & \omega^{N-1} & \omega^{2(N-1)} & \cdots & \omega^{(N-1)^2}\end{array}\right\].\\] This matrix (a special case of a [Vandermonde matrix](http://en.wikipedia.org/wiki/Vandermonde_matrix), has a lot of structure, and one can utilize this structure to compute \\(Mx\\) in faster than \\(O(N^2)\\) time. In fact, that's what the [Fast Fourier Transform](http://en.wikipedia.org/wiki/Fast_Fourier_transform) is all about!)


# Using this in Practice with Python

Now let's actually Fourier-approximate some sequences using [Numpy](http://www.numpy.org/) and [Pandas](http://pandas.pydata.org/). For our data set, we'll use [this time series](http://datamarket.com/data/set/22q8/monthly-sales-of-new-one-family-houses-sold-in-th-e-usa-since-1973#!display=line&ds=22q8) representing the number of one-family houses sold each month from 1973 to 1995.

~~~
import pandas as pd
import numpy as np
from Pandas import Series, DataFrame
data = pd.read_csv('monthly-sales-of-new-onefamily-h.csv')
print data[:10]
     Month  Monthly sales of new one-family houses sold in th e USA since 1973
0  1973-01                                                 55                 
1  1973-02                                                 60                 
2  1973-03                                                 68                 
3  1973-04                                                 63                 
4  1973-05                                                 65                 
5  1973-06                                                 61                 
6  1973-07                                                 54                 
7  1973-08                                                 52                 
8  1973-09                                                 46                 
9  1973-10                                                 42  
data.columns = ['month', 'sales']
data = Series(data['sales'])
data.index = data['month'].values
data.plot(rot=10, ylim=(0,100), xlim=(0,23*12 - 1))
~~~


![image](http://newdatascientist.com/blog/wp-content/uploads/2013/06/graph2.png)

As you can see, there's some pretty clear periodicity to house sales. For the curious:

~~~
data['month'] = data.index
def get_month(x):
   numeric = int(x[-2:])
   months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
   return(months[numeric -1])
data['month'] = data['month'].map(get_month)
data[:10]
 
         sales month
1973-01     55   Jan
1973-02     60   Feb
1973-03     68   Mar
1973-04     63   Apr
1973-05     65   May
1973-06     61   Jun
1973-07     54   Jul
1973-08     52   Aug
1973-09     46   Sep
1973-10     42   Oct

by_month = data.groupby('month').mean().reindex(['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'])
by_month 
         sales
Jan  45.347826
Feb  51.130435
Mar  61.391304
Apr  60.260870
May  59.608696
Jun  57.000000
Jul  53.782609
Aug  55.130435
Sep  50.826087
Oct  49.739130
Nov  43.130435
Dec  39.545455

by_month.plot(kind='bar')
plt.show()
~~~


![image](http://newdatascientist.com/blog/wp-content/uploads/2013/06/by_month.png)

Even more interestingly, there seems to be a roughly 9 year cycle. But let's not get too distracted. We want to approximate the time series of house sales by it's Fourier series. Well, actually: From what we said earlier, we know that if we use the <em>full</em> Fourier series, the 'approximation' will actually be the original series again. (How's that for an accurate approximation!) So what we really want to do is use just part of the Fourier series. 

~~~
def truncated_approximation(series, order):
    fourier = np.fft.fft(series)
    truncated_fourier = [x if i <= order or len(fourier)-i <= order else 0 for i,x in enumerate(fourier)]
    return np.real(np.fft.ifft(truncated_fourier))
data['30th'] = truncated_approximation(data['sales'], order=30)
data['20th'] = truncated_approximation(data['sales'], order=20)
data['10th'] = truncated_approximation(data['sales'], order=10)
data.plot(x='month', y=['10th', '20th', '30th'], rot=10, ylim=(0,100), xlim=(0,23*12 - 1), linewidth=5)
data.plot(x='month', y='sales', rot=10, ylim=(0,100), xlim=(0,23*12 - 1))
plt.show()
~~~


![image](http://newdatascientist.com/blog/wp-content/uploads/2013/06/approx.png)

The 20th order approximation (green) is not too shabby; and as you can see, the 30th order approximation (red) gets quite close to the true values. 

So what's going on? To get the \\(n^{\text{th}}\\) order truncated approximation to \\(f\\), we use the sum \\[\sum\_{-n \leq k \leq n} \widehat{f}(k)\epsilon\_k.\\] (Hence the \\(n^{\text{th}}\\) order approximation actually uses \\(2n + 1\\) summands.) We're using the numpy fft (Fast Fourier Transform) library to accomplish this.

And that's pretty much it! There are more advanced tweaks that one can make (e.g. it's sometimes advantageous to 'periodicize' a series by adding taking some values from its beginning and adding them to its end, and a truncated approximation may not be the most appropriate if your data's period is very short), but we'll let them rest for now.
