---
layout: post
title: "Generating random variables"
author: "MMA"
comments: true
---
# Transformation of Random Variables

Let's consider how to take the transformation of a random variable $X$ with cumulative distribution function $F_{X}(x)$. Let $Y=t(X)$, that is, $Y$ is the transformation of $X$ via function $t(\cdot)$.

In order to get the CDF of $Y$ we use the definition of CDFs:

$$
F_{Y}(y) = P(Y \leq y) = P(t(X) \leq y)
$$

We have $F_{X}(x)$ and want to know how to compute $F_{Y}(y)$ in terms of $F_{X}(x)$. To get there we can take the inverse of $t(x)$ on both sides of the inequality:

$$
F_{Y}(y) = P(Y \leq y) = P(t(X) \leq y) = P(X \leq t^{-1}(y))
$$

This is the CDF of $X$:

$$
P(X \leq t^{-1}(y)) = F_{X}(t^{-1}(y))
$$

and that's how we get $F_{Y}(y)$ in terms of $F_{X}(x)$. We can compute the density function $f_{Y}(y)$ by differentiating $F_{Y}(y)$, applying the chain rule:

$$
f_{Y}(y) = f_{y}(t^{-1}(y)) \times \frac{d}{dy} t^{-1}(y) dy
$$

Note that it is only this simple if $t(x)$ is one-to-one and strictly monotone increasing; it gets more complicated to reason about the regions where $Y$ is defined otherwise.

How do we use this result?

Let $U \sim U(0, 1)$. Then $F(X) = U$ means that the random variable $F^{−1}(U)$ has the same distribution as $X$.

# Inverse transform sampling

It is a basic method for pseudo-random number sampling, i.e. for generating sample numbers at random from any probability distribution given its cumulative distribution function. The basic principle is to find the inverse function of F, $F^{-1}$ such that $F~ F^{-1} = F^{-1} ~ F = I$.

The problem that the inverse transform sampling method solves is as follows:

* Let $X$ be a random variable whose distribution can be described by the cumulative distribution function $F_{X}$.
* We want to generate values of $X$ which are distributed according to this distribution.

The inverse transform sampling method works as follows:

* Generate a random number $u$ from the standard uniform distribution in the interval $[0,1]$, e.g. from $U\sim Unif [0,1]$.
* Find the inverse of the desired CDF, e.g. $F_{X}^{-1}(x)$. Inverse cumulative distribution function is also called quantile function.
* Compute $x = F_{X}^{-1}(u)$ (Solve the equation $F_{X}(x) = u$ for $X$). The computed random variable $X$ has distribution $F_{X}(x)$.

Expressed differently, given a continuous uniform variable $U$ in $[0,1]$ and an invertible cumulative distribution function $F_{X}$, the random variable $X=F_{X}^{-1}(U)$ has distribution $F_{X}$ (or, $X$ is distributed $F_{X}$).

$$
\begin{split}
F_{X}(x) = P(X \leq x) &= P(F_{X}^{-1}(U)\leq x)\\
&=P(U \leq F_{X}(x))\\
&= F_{U}(F_{X}(x))\\
&= F_{X}(x) 
\end{split}
$$

Remember that the cumulative distribution function of continuous uniform distribution on the interval $[0,1]$ is $F_{U}(u)=u
$.

Computationally, this method involves computing the quantile function of the distribution — in other words, computing the cumulative distribution function (CDF) of the distribution (which maps a number in the domain to a probability between 0 and 1) and then inverting that function many times. This is the source of the term "inverse" or "inversion" in most of the names for this method. Note that for a discrete distribution, computing the CDF is not in general too difficult: we simply add up the individual probabilities for the various points of the distribution. For a continuous distribution, however, we need to integrate the probability density function (PDF) of the distribution, which is impossible to do analytically for most distributions (including the normal distribution). As a result, this method may be computationally inefficient for many distributions and other methods are preferred; however, it is a useful method for building more generally applicable samplers such as those based on rejection sampling.

For the normal distribution, the lack of an analytical expression for the corresponding quantile function means that other methods (e.g. the Box–Muller transform) may be preferred computationally. It is often the case that, even for simple distributions, the inverse transform sampling method can be improved on.

(Note: technically this only works when the CDF has a closed form inverse function)

![](https://github.com/mmuratarat/mmuratarat.github.io/blob/master/_posts/images/Screen%20Shot%202019-10-06%20at%2019.45.52.png?raw=true)

# Continuous Example: Exponential Distribution

The exponential distribution has CDF:

$$
F_X(x) = 1 - e^{-\lambda x}
$$

for $x \geq 0$ (and $0$ otherwise). By solving $u=F(x)$ we obtain the inverse function

$$
\begin{split}
1 - e^{-\lambda x} &= u\\
x &= \frac{-1}{\lambda}ln(1 - y)
\end{split}
$$

so

$$
F^{-1}_X(x) = \frac{-1}{\lambda}ln(1 - u)
$$

It means that if we draw some $u$ from $U \sim Unif(0,1)$ and compute $x = F^{-1}_X(x) = \frac{-1}{\lambda}ln(1 - u)$, this $X$ has exponential distribution.

Note that in practice, since both $u$ AND $1-u$ are uniformly distributed random number, so the calculation can be simplified as:

$$
x = F^{-1}_X(x) = \frac{-1}{\lambda}ln(u)
$$

{% highlight python %} 
import math
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

def inverse_exp_dist(lmbda=1.0):
    return (-1 / lmbda)*math.log(1 - np.random.random())

plt.hist([inverse_exp_dist() for i in range(10000)], 50)
plt.title('Samples from an exponential function')
plt.savefig('inverse_pdf_exp_dist')
plt.show()
{% endhighlight %}

![](https://github.com/mmuratarat/mmuratarat.github.io/blob/master/_posts/images/inverse_pdf_exp_dist.png?raw=true)

and just to make sure this looks right, let's use numpy's exponential function and compare:

{% highlight python %} 
plt.hist([np.random.exponential() for i in range(10000)], 50)   
plt.title('Samples from numpy.random.exponential')
plt.savefig('numpy_random_exponential_dist')
plt.show()
{% endhighlight %}

![](https://github.com/mmuratarat/mmuratarat.github.io/blob/master/_posts/images/numpy_random_exponential_dist.png?raw=true)

# Functions with no inverses

In general, there are no inverses for functions that can return same value for different inputs, for example density functions (e.g., the standard normal density function is symmetric, so it returns the same values for −2 and 2 etc.). The normal distribution is an interesting example for one more reason—it is one of the examples of cumulative distribution functions that do not have a closed-form inverse. Not every cumulative distribution function has to have a closed-form inverse! Therefore,  the inverse transform method is not efficient. Hopefully in such cases the inverses can be found using numerical methods.

# Normal Distribution

There's no closed form expression for the inverse cdf of a normal distributio (a.k.a. the quantile function of a normal distribution). This is often a problem with the inverse transform method. There are various ways to express the function and numerous approximations.

Let's think of a standard normal distribution. The drawback of using inverse CDF method is that it relies on calculation of the probit function $\Phi^{-1}$, which cannot be done analytically (Note that In probability theory and statistics, the probit function is the quantile function associated with the standard normal distribution, which is commonly denoted as N(0,1)). Some approximate methods are described in the literature. One of the easiest way is to do a table lookup. E.g., If $U = 0.975$, then $Z = \Phi^{-1}(U) = 1.96$ because z-table gives $\Phi(Z)$.

![](https://github.com/mmuratarat/mmuratarat.github.io/blob/master/_posts/images/z_table.png?raw=true)

If we are willing to accept numeric solution, inverse functions can be found. One of the inverse c.d.f. of the standard normal distribution was proposed by Schmeiser:

$$
Z = \Phi^{-1}(U) \approx \frac{U^{0.135} - (1 - U)^{0.135}} {0.1975}
$$

for $0.0013499 \le U \le 0.9986501$ which matches the true normal distribution with one digit after decimal point. 

There is one another approximation. The following approximation has absolute error $\leq 0.45 \times 10^{−3}$:

$$
Z = sign(U − 1/2) \left(t - \frac{c_{0} + c_{1}t + c_{2} t^{2}}{1 + d_{1}t + d_{2} t^{2} + d_{3}t^{3}} \right)
$$

where sign(x) = 1, 0, −1 if $X$ is positive, zero, or negative, respectively,

$$
t = \left\{- \ln \left[min (U, 1-U) \right]^{2} \right\}^{1/2}
$$

and $c_{0} = 2.515517, c_{1} = 0.802853, c_{2} = 0.010328, d_{1} = 1.432788, d_{2} = 0.189269, d_{3} = 0.001308$.

In any case, rather than sampling x directly, we could instead sample $Z \sim N(0, 1)$ and transform samples of $Z$ into samples of $X$. If $Z \sim N(0, 1)$ and you want $X \sim N(\mu, \sigma^{2})$, just take $X \leftarrow \mu + \sigma Z$. Suppose you want to generate $X \sim N(3, 16)$, and you start with $U = 0.59$. Then,

$$
X = \mu + \sigma Z = 3 + 4 \Phi^{-1}(0.59) = 3 + 4(0.2275) = 3.91
$$

because $\Phi^{-1}(0.59) = Z \rightarrow \Phi(Z) = P(Z \leq Z) = 0.59$. What is this $Z$? Using a [online calculator](https://stattrek.com/online-calculator/normal.aspx){:target="_blank"}, it is $0.2275$.

Let's see an example in Python. 

{% highlight python %} 
n = 10000  # Samples to draw
mean = 3
variance = 16
Z = np.random.normal(loc=0, scale=1.0, size=(n,))
X = mean + (np.sqrt(variance) * Z)

print(np.mean(X))
#3.0017206638273097

print(np.std(X))
#4.022342597707669

count, bins, ignored = plt.hist(X, 30, normed=True)
plt.plot(bins, univariate_normal(bins, mean, variance),linewidth=2, color='r')
plt.savefig('generated_normal_dist.png')
plt.show()
{% endhighlight %}

![](https://github.com/mmuratarat/mmuratarat.github.io/blob/master/_posts/images/generated_normal_dist.png?raw=true)

### The Box–Muller method 
Now let's consider a more direct and exact transformation. Let $Z_1, Z_2$ be two standard normal random variates. Plot the two as a point in the plane and represent them in a polar coordinate system as $Z_1 = B \cos \theta$ and $Z_2 = B \sin \theta$.

It is known that $B^2 = {Z_1}^2 + {Z_2}^2$ has the chi-square distribution with 2 degrees of freedom, which is equivalent to an exponential distribution with mean 2 (this comes from the fact that if one has $k$ i.i.d normal random variables where $X_i\sim N(0,\sigma^2)$, sum of squares of those random variables, $X_1^2+X_2^2+\dots+X_k^2\sim\sigma^2\chi^2_k$):

$$
Y = \lambda e^{-\lambda t},\,\,\,\,\, t \geq 0
$$

where $E[Y] = 2 = \lambda$. Thus, the raidus $B$ can be generated using $B = \sqrt{-2\ln U}$. 

Note that here, we use alternative formulation of Exponential distribution, where:

$$
f(x) = \frac{1} {\lambda} e^{-x/\lambda},\,\,\,\,\, x \geq 0; \lambda > 0
$$

with mean $E(X) = \lambda$ and variance $Var(X)=\lambda^{2}$

$$
F(x) = 1 - e^{-x/\lambda},\,\,\,\,\, x \ge 0; \lambda > 0
$$

So, the formula for inverse of CDF (quantile function or the percent point function) of the exponential distribution is

$$
F^{-1}_{X}(x) = -\lambda\ln(1 - u)
$$

Again, in practice, since both $u$ AND $1-u$ are uniformly distributed random number.

So a standard normal distribution can be generated by any one of the following.

$$
Z_1 = \sqrt{-2 \ln U_1} \cos (2\pi U_2)
$$

and

$$
Z_2 = \sqrt{-2 \ln U_1} \sin (2\pi U_2)
$$

where $U_1$ and $U_2$ are uniformly distributed over $(0,1)$ and they will be independent. In order to obtain normal variates $X_i$ with mean $\mu$ and variance $\sigma^2$, transform $X_i = \mu + \sigma Z_i$.

```python
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline
plt.style.use('ggplot')

# uniformly distributed values between 0 and 1
u1 = np.random.rand(1000)
u2 = np.random.rand(1000)

# transformation function
def box_muller(u1,u2):
    z1 = np.sqrt(-2*np.log(u1))*np.cos(2*np.pi*u2)
    z2 = np.sqrt(-2*np.log(u1))*np.sin(2*np.pi*u2)
    return z1,z2

# Run the transformation
z1 = box_muller(u1, u2)
z2 = box_muller(u1, u2)

# plotting the values before and after the transformation
plt.figure(figsize = (20, 10))
plt.subplot(221) # the first row of graphs
plt.hist(u1)     # contains the histograms of u1 and u2 
plt.subplot(222)
plt.hist(u2)
plt.subplot(223) # the second contains
plt.hist(z1)     # the histograms of z1 and z2
plt.subplot(224)
plt.hist(z2)
plt.show()
```

![](https://github.com/mmuratarat/mmuratarat.github.io/blob/master/_posts/images/box_muller.png?raw=true)

There is also The Marsaglia polar method which is a modification of the Box–Muller method which does not require computation of the sine and cosine functions

However, in this computer age, most statistical software would provide you with quantile function for normal distribution already implemented. The inverse of the normal CDF is know and given by:

$$
F^{-1}(Z)\; =\; \sqrt2\;\operatorname{erf}^{-1}(2Z - 1), \quad Z\in(0,1).
$$

Hence:

$$
Z = F^{-1}(U)\; =\; \sqrt2\;\operatorname{erf}^{-1}(2U - 1), \quad U\in(0,1)
$$

where erf is error function.

# Characterization method (Convolution Method)

This method is another approach to sample from a distribution. In some cases, $X$ can be expressed as a sum of independent random variables $Y_{1}, Y_{2}, \ldots , Y_{n}$ where $Y_{j}$'s are iid and n is fixed and finite:

$$
X = Y_{1} + Y_{2} + \ldots + Y_{n}
$$

called n-fold convolution of distribution $Y_{j}$. Here, $Y_{j}$'s are generated more easilt.

**Algorithm**:
* Generate independent $Y_{1}, Y_{2}, \ldots , Y_{n}$ each with distribution function $F_{Y}(y)$ using the inverse transform method.
* Return $X = Y_{1} + Y_{2} + \ldots + Y_{n}$.

For example, an Erlang random variable $X$ with parameters $(n, \lambda)$ can be shown to be the sum of $n$ independent exponential random variables $Y_{i}, i=1,2, \ldots ,n$, each having a mean of $\frac{1}{\lambda}$.

$$
X = \sum_{i=1}^{n} Y_{i}
$$

Using inverse CDF method that can generate an exponential variable, an Erlang variate can be generated:

$$
X = \sum_{i=1}^{n}  \frac{-1}{\lambda}ln(u_{i}) = \frac{-1}{\lambda} ln \left(\prod_{i=1}^{n} u_{i} \right)
$$

Other examples:

* If $X_{1}, \ldtos , X_{n}$ are i.i.d. Geometric(p), then $\sum_{i=1}^{n} X_{i} \sim NegBin(n, p)$
* If $X_{1}, \ldtos , X_{n}$ are i.i.d. Normal(0,1), then $\sum_{i=1}^{n} X_{i} \sim \chi_{n}^{2}$
* If $X_{1}, \ldtos , X_{n}$ are i.i.d. Bernoulli(p), then $\sum_{i=1}^{n} X_{i} \sim Binomial(n, p)$

# Composition Method

This method applies when the distribution function $F$ can be expressed as a mixture of other distribution functions $F_{1}, F_{2}, \ldots$:

$$
F(x) = \sum_{j=1}^{\infty} p_{j}F_{j}(x),
$$

where $p_{j} \geq 0$ and $\sum_{j=1}^{\infty} p_{j} =1$, meaning that the $p_{j}$ form a discrete probability distribution

Equivalently, we can decompose the density function $f(x)$ or mass function $p(x)$ into convex combination of other density or mass functions. This method is useful if it is easier to sample from $F_{j}$'s than from $F$.

**Algorithm**:
* Generate a discrete random variable $j$ such that $P(J = j) = p_{j}$.
* Return $X$ with CDF $F_{J}(x)$ (given $J=j$, $x$ is generated independent of $J$).

For fixed $x$:
$$
\begin{split}
P(X \leq x) &= \sum_{j} P(X \leq x \mid J = j)P(J = j) \text{ (condition on } J=j\text{)}\\
&= \sum_{j} P(X \leq x \mid J = j)p_{j}\text{ (distribution of J)}\\
&= \sum_{j} F_{j}(x)p_{j} \text{ (given } J = j, X \sim F_{j}\text{)}\\
&=F_{X}(x) \text{ (decomposition of F)}
\end{split}
$$

The trick is to find $F_{j}$’s from which generation is easy and fast.

#### REFERENCES

1. [https://www.quora.com/What-is-an-intuitive-explanation-of-inverse-transform-sampling-method-in-statistics-and-how-does-it-relate-to-cumulative-distribution-function/answer/Amit-Sharma-2?srid=X8V](https://www.quora.com/What-is-an-intuitive-explanation-of-inverse-transform-sampling-method-in-statistics-and-how-does-it-relate-to-cumulative-distribution-function/answer/Amit-Sharma-2?srid=X8V){:target="_blank"}
2. [http://www.eg.bucknell.edu/~xmeng/Course/CS6337/Note/master/node48.html](http://www.eg.bucknell.edu/~xmeng/Course/CS6337/Note/master/node48.html){:target="_blank"}
3. [http://www.columbia.edu/~ks20/4404-Sigman/4404-Notes-ITM.pdf](http://www.columbia.edu/~ks20/4404-Sigman/4404-Notes-ITM.pdf){:target="_blank"}
4. [http://people.duke.edu/~ccc14/sta-663-2016/15A_RandomNumbers.html](http://people.duke.edu/~ccc14/sta-663-2016/15A_RandomNumbers.html){:target="_blank"}
5. [http://karlrosaen.com/ml/notebooks/simulating-random-variables/](http://karlrosaen.com/ml/notebooks/simulating-random-variables/){:target="_blank"}
6. [https://stephens999.github.io/fiveMinuteStats/inverse_transform_sampling.html](https://stephens999.github.io/fiveMinuteStats/inverse_transform_sampling.html){:target="_blank"}
7. [https://stats.stackexchange.com/a/236157/16534](https://stats.stackexchange.com/a/236157/16534){:target="_blank"}
8. [https://www2.isye.gatech.edu/~sman/courses/6644/Module07-RandomVariateGenerationSlides_171116.pdf](https://www2.isye.gatech.edu/~sman/courses/6644/Module07-RandomVariateGenerationSlides_171116.pdf){:target="_blank"}
9. [http://bjlkeng.github.io/posts/sampling-from-a-normal-distribution/](http://bjlkeng.github.io/posts/sampling-from-a-normal-distribution/){:target="_blank"}
10. [https://en.wikipedia.org/wiki/Normal_distribution#Generating_values_from_normal_distribution](https://en.wikipedia.org/wiki/Normal_distribution#Generating_values_from_normal_distribution){:target="_blank"}
11. [https://web.ics.purdue.edu/~hwan/IE680/Lectures/Chap08Slides.pdf](https://web.ics.purdue.edu/~hwan/IE680/Lectures/Chap08Slides.pdf){:target="_blank"}
12. [https://www.win.tue.nl/~marko/2WB05/lecture8.pdf](https://www.win.tue.nl/~marko/2WB05/lecture8.pdf){:target="_blank"}