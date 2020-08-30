# Dağılımlar
:label:`sec_distributions`

Artık hem kesikli hem de sürekli ortamda olasılıkla nasıl çalışılacağını öğrendiğimize göre, şimdi karşılaşılan yaygın dağılımlardan bazılarını öğrenelim. Makine öğrenmesi alanına bağlı olarak, bunlardan çok daha fazlasına aşina olmamız gerekebilir; derin öğrenmenin bazı alanları için potansiyel olarak hiç kullanılmıyorlar. Ancak bu, aşina olunması gereken iyi bir temel listedir. Önce bazı ortak kütüphaneleri içeri aktaralım.

```{.python .input}
%matplotlib inline
from d2l import mxnet as d2l
from IPython import display
from math import erf, factorial
import numpy as np
```

```{.python .input}
#@tab pytorch
%matplotlib inline
from d2l import torch as d2l
from IPython import display
from math import erf, factorial
import torch

torch.pi = torch.acos(torch.zeros(1)) * 2  # Define pi in torch
```

## Bernoulli

Bu, genellikle karşılaşılan en basit rastgele değişkendir. Bu rastgele değişken, $p$ olasılıkla $1$ ve $1-p$ olasılıkla $0$ gelen bir yazı tura atmayı kodlar. Bu dağılımla rastgele bir değişkenimiz $X$ varsa, sunu yazacağız:

$$
X \sim \mathrm{Bernoulli}(p).
$$

Birikimli dağılım fonksiyonu şöyledir:

$$F(x) = \begin{cases} 0 & x < 0, \\ 1-p & 0 \le x < 1, \\ 1 & x >= 1 . \end{cases}$$
:eqlabel:`eq_bernoulli_cdf`

Olasılık kütle fonksiyonu aşağıda çizilmiştir.

```{.python .input}
#@tab all
p = 0.3

d2l.set_figsize()
d2l.plt.stem([0, 1], [1 - p, p], use_line_collection=True)
d2l.plt.xlabel('x')
d2l.plt.ylabel('p.m.f.')
d2l.plt.show()
```

Şimdi, birikimli dağılım fonksiyonunu çizelim :eqref:`eq_bernoulli_cdf`.

```{.python .input}
x = np.arange(-1, 2, 0.01)

def F(x):
    return 0 if x < 0 else 1 if x > 1 else 1 - p

d2l.plot(x, np.array([F(y) for y in x]), 'x', 'c.d.f.')
```

```{.python .input}
#@tab pytorch
x = torch.arange(-1, 2, 0.01)

def F(x):
    return 0 if x < 0 else 1 if x > 1 else 1 - p

d2l.plot(x, torch.tensor([F(y) for y in x]), 'x', 'c.d.f.')
```

Eğer $X \sim \mathrm{Bernoulli}(p)$ ise, o zaman:

* $\mu_X = p$,
* $\sigma_X^2 = p(1-p)$.

Bir Bernoulli rastgele değişkeninden keyfi bir şekil dizisini aşağıdaki gibi örnekleyebiliriz.

```{.python .input}
1*(np.random.rand(10, 10) < p)
```

```{.python .input}
#@tab pytorch
1*(torch.rand(10, 10) < p)
```

## Ayrık Tekdüze Dağılım

Karşılaşılan bir sonraki yaygın karşılaşılan rastgele değişken, ayrık bir tekdüzedir. Buradaki tartışmamız için, $\{1, 2, \ldots, n\}$ tam sayılarında desteklendiğini varsayacağız, ancak herhangi bir tüm değerler kümesi serbestçe seçilebilir. Bu bağlamda *tekdüze* kelimesinin anlamı, olabilir her değerin eşit derecede olası olmasıdır. $i \in \{1, 2, 3, \ldots, n\}$ değerinin olasılığı $p_i = \frac{1}{n}$'dir. Bu dağılımla $X$ rastgele değişkenini şu şekilde göstereceğiz:

$$
X \sim \mathrm{Uniform}(n).
$$

Birikimli dağılım fonksiyonunu böyledir:

$$F(x) = \begin{cases} 0 & x < 1, \\ \frac{k}{n} & k \le x < k+1 \text{ with } 1 \le k < n, \\ 1 & x >= n . \end{cases}$$
:eqlabel:`eq_discrete_uniform_cdf`

Ilk olarak olasılık kütle fonksiyonu çizelim.

```{.python .input}
#@tab all
n = 5

d2l.plt.stem([i+1 for i in range(n)], n*[1 / n], use_line_collection=True)
d2l.plt.xlabel('x')
d2l.plt.ylabel('p.m.f.')
d2l.plt.show()
```

Şimdi, birikimli dağılım fonksiyonunu çizelim :eqref:`eq_discrete_uniform_cdf`.

```{.python .input}
x = np.arange(-1, 6, 0.01)

def F(x):
    return 0 if x < 1 else 1 if x > n else np.floor(x) / n

d2l.plot(x, np.array([F(y) for y in x]), 'x', 'c.d.f.')
```

```{.python .input}
#@tab pytorch
x = torch.arange(-1, 6, 0.01)

def F(x):
    return 0 if x < 1 else 1 if x > n else torch.floor(x) / n

d2l.plot(x, torch.tensor([F(y) for y in x]), 'x', 'c.d.f.')
```

Eğer $X \sim \mathrm{Uniform}(n)$ ise, o zaman:

* $\mu_X = \frac{1+n}{2}$,
* $\sigma_X^2 = \frac{n^2-1}{12}$.

Aşağıdaki gibi, ayrık bir tekdüze rastgele değişkenden keyfi bir şekil dizisini örnekleyebiliriz.

```{.python .input}
np.random.randint(1, n, size=(10, 10))
```

```{.python .input}
#@tab pytorch
torch.randint(1, n, size=(10, 10))
```

## Sürekli Tekdüze Dağılım

Şimdi, sürekli tekdüze dağılımı tartışalım. Bu rastgele değişkenin arkasındaki fikir şudur: Ayrık tekdüze dağılımdaki $n$'yi arttırırsak ve bunu $[a, b]$ aralığına sığacak şekilde ölçeklendirirsek; sadece $[a, b]$ aralığında, hepsi eşit olasılıkla, keyfi bir değer seçen sürekli bir rastgele değişkene yaklaşacağız. Bu dağılımı şu şekilde göstereceğiz

$$
X \sim \mathrm{Uniform}([a, b]).
$$

Olasılık yoğunluk fonksiyonu şöyledir: 

$$p(x) = \begin{cases} \frac{1}{b-a} & x \in [a, b], \\ 0 & x \not\in [a, b].\end{cases}$$
:eqlabel:`eq_cont_uniform_pdf`

Birikimli dağılım fonksiyonunu şöyledir:

$$F(x) = \begin{cases} 0 & x < a, \\ \frac{x-a}{b-a} & x \in [a, b], \\ 1 & x >= b . \end{cases}$$
:eqlabel:`eq_cont_uniform_cdf`

Önce, olasılık yoğunluk dağılım fonksiyonunu çizelim :eqref:`eq_cont_uniform_pdf`.

```{.python .input}
a, b = 1, 3

x = np.arange(0, 4, 0.01)
p = (x > a)*(x < b)/(b - a)

d2l.plot(x, p, 'x', 'p.d.f.')
```

```{.python .input}
#@tab pytorch
a, b = 1, 3

x = torch.arange(0, 4, 0.01)
p = (x > a).type(torch.float32)*(x < b).type(torch.float32)/(b-a)
d2l.plot(x, p, 'x', 'p.d.f.')
```

Şimdi, birikimli dağılım fonksiyonunu çizelim :eqref:`eq_cont_uniform_cdf`.

```{.python .input}
def F(x):
    return 0 if x < a else 1 if x > b else (x - a) / (b - a)

d2l.plot(x, np.array([F(y) for y in x]), 'x', 'c.d.f.')
```

```{.python .input}
#@tab pytorch
def F(x):
    return 0 if x < a else 1 if x > b else (x - a) / (b - a)

d2l.plot(x, torch.tensor([F(y) for y in x]), 'x', 'c.d.f.')
```

Eğer $X \sim \mathrm{Uniform}([a, b])$ ise, o zaman:

* $\mu_X = \frac{a+b}{2}$,
* $\sigma_X^2 = \frac{(b-a)^2}{12}$.

Aşağıdaki gibi tekdüze bir rastgele değişkenden keyfi bir şekil dizisini örnekleyebiliriz. Bunun $\mathrm{Uniform}([0,1])$'den varsayılan örnekler olduğuna dikkat edin, bu nedenle farklı bir aralık istiyorsak, onu ölçeklendirmemiz gerekir.

```{.python .input}
(b - a) * np.random.rand(10, 10) + a
```

```{.python .input}
#@tab pytorch
(b - a) * torch.rand(10, 10) + a
```

## Binomial

Let us make things a little more complex and examine the *binomial* random variable.  This random variable originates from performing a sequence of $n$ independent experiments, each of which has probability $p$ of succeeding, and asking how many successes we expect to see.

Let us express this mathematically.  Each experiment is an independent random variable $X_i$ where we will use $1$ to encode success, and $0$ to encode failure.  Since each is an independent coin flip which is successful with probability $p$, we can say that $X_i \sim \mathrm{Bernoulli}(p)$.  Then, the binomial random variable is

$$
X = \sum_{i=1}^n X_i.
$$

In this case, we will write

$$
X \sim \mathrm{Binomial}(n, p).
$$

To get the cumulative distribution function, we need to notice that getting exactly $k$ successes can occur in $\binom{n}{k} = \frac{n!}{k!(n-k)!}$ ways each of which has a probability of $p^k(1-p)^{n-k}$ of occurring.  Thus the cumulative distribution function is

$$F(x) = \begin{cases} 0 & x < 0, \\ \sum_{m \le k} \binom{n}{m} p^m(1-p)^{n-m}  & k \le x < k+1 \text{ with } 0 \le k < n, \\ 1 & x >= n . \end{cases}$$
:eqlabel:`eq_binomial_cdf`

Let us first plot the probability mass function.

```{.python .input}
n, p = 10, 0.2

# Compute binomial coefficient
def binom(n, k):
    comb = 1
    for i in range(min(k, n - k)):
        comb = comb * (n - i) // (i + 1)
    return comb

pmf = np.array([p**i * (1-p)**(n - i) * binom(n, i) for i in range(n + 1)])

d2l.plt.stem([i for i in range(n + 1)], pmf, use_line_collection=True)
d2l.plt.xlabel('x')
d2l.plt.ylabel('p.m.f.')
d2l.plt.show()
```

```{.python .input}
#@tab pytorch
n, p = 10, 0.2

# Compute binomial coefficient
def binom(n, k):
    comb = 1
    for i in range(min(k, n - k)):
        comb = comb * (n - i) // (i + 1)
    return comb

pmf = torch.tensor([p**i * (1-p)**(n - i) * binom(n, i) for i in range(n + 1)])

d2l.plt.stem([i for i in range(n + 1)], pmf, use_line_collection=True)
d2l.plt.xlabel('x')
d2l.plt.ylabel('p.m.f.')
d2l.plt.show()
```

Now, let us plot the cumulative distribution function :eqref:`eq_binomial_cdf`.

```{.python .input}
x = np.arange(-1, 11, 0.01)
cmf = np.cumsum(pmf)

def F(x):
    return 0 if x < 0 else 1 if x > n else cmf[int(x)]

d2l.plot(x, np.array([F(y) for y in x.tolist()]), 'x', 'c.d.f.')
```

```{.python .input}
#@tab pytorch
x = torch.arange(-1, 11, 0.01)
cmf = torch.cumsum(pmf, dim=0)

def F(x):
    return 0 if x < 0 else 1 if x > n else cmf[int(x)]

d2l.plot(x, torch.tensor([F(y) for y in x.tolist()]), 'x', 'c.d.f.')
```

While this result is not simple, the means and variances are.  If $X \sim \mathrm{Binomial}(n, p)$, then:

* $\mu_X = np$,
* $\sigma_X^2 = np(1-p)$.

This can be sampled as follows.

```{.python .input}
np.random.binomial(n, p, size=(10, 10))
```

```{.python .input}
#@tab pytorch
m = torch.distributions.binomial.Binomial(n, p)
m.sample(sample_shape=(10, 10))
```

## Poisson
Let us now perform a thought experiment.  We are standing at a bus stop and we want to know how many buses will arrive in the next minute.  Let us start by considering $X^{(1)} \sim \mathrm{Bernoulli}(p)$ which is simply the probability that a bus arrives in the one minute window.  For bus stops far from an urban center, this might be a pretty good approximation.  We may never see more than one bus in a minute.

However, if we are in a busy area, it is possible or even likely that two buses will arrive.  We can model this by splitting our random variable into two parts for the first 30 seconds, or the second 30 seconds.  In this case we can write

$$
X^{(2)} \sim X^{(2)}_1 + X^{(2)}_2,
$$

where $X^{(2)}$ is the total sum, and $X^{(2)}_i \sim \mathrm{Bernoulli}(p/2)$.  The total distribution is then $X^{(2)} \sim \mathrm{Binomial}(2, p/2)$.

Why stop here?  Let us continue to split that minute into $n$ parts.  By the same reasoning as above, we see that

$$X^{(n)} \sim \mathrm{Binomial}(n, p/n).$$
:eqlabel:`eq_eq_poisson_approx`

Consider these random variables.  By the previous section, we know that :eqref:`eq_eq_poisson_approx` has mean $\mu_{X^{(n)}} = n(p/n) = p$, and variance $\sigma_{X^{(n)}}^2 = n(p/n)(1-(p/n)) = p(1-p/n)$.  If we take $n \rightarrow \infty$, we can see that these numbers stabilize to $\mu_{X^{(\infty)}} = p$, and variance $\sigma_{X^{(\infty)}}^2 = p$.  This indicates that there *could be* some random variable we can define in this infinite subdivision limit.  

This should not come as too much of a surprise, since in the real world we can just count the number of bus arrivals, however it is nice to see that our mathematical model is well defined.  This discussion can be made formal as the *law of rare events*.

Following through this reasoning carefully, we can arrive at the following model.  We will say that $X \sim \mathrm{Poisson}(\lambda)$ if it is a random variable which takes the values $\{0,1,2, \ldots\}$ with probability

$$p_k = \frac{\lambda^ke^{-\lambda}}{k!}.$$
:eqlabel:`eq_poisson_mass`

The value $\lambda > 0$ is known as the *rate* (or the *shape* parameter), and denotes the average number of arrivals we expect in one unit of time.  

We may sum this probability mass function to get the cumulative distribution function.

$$F(x) = \begin{cases} 0 & x < 0, \\ e^{-\lambda}\sum_{m = 0}^k \frac{\lambda^m}{m!} & k \le x < k+1 \text{ with } 0 \le k. \end{cases}$$
:eqlabel:`eq_poisson_cdf`

Let us first plot the probability mass function :eqref:`eq_poisson_mass`.

```{.python .input}
lam = 5.0

xs = [i for i in range(20)]
pmf = np.array([np.exp(-lam) * lam**k / factorial(k) for k in xs])

d2l.plt.stem(xs, pmf, use_line_collection=True)
d2l.plt.xlabel('x')
d2l.plt.ylabel('p.m.f.')
d2l.plt.show()
```

```{.python .input}
#@tab pytorch
lam = 5.0

xs = [i for i in range(20)]
pmf = torch.tensor([torch.exp(torch.tensor(-lam)) * lam**k
                    / factorial(k) for k in xs])

d2l.plt.stem(xs, pmf, use_line_collection=True)
d2l.plt.xlabel('x')
d2l.plt.ylabel('p.m.f.')
d2l.plt.show()
```

Now, let us plot the cumulative distribution function :eqref:`eq_poisson_cdf`.

```{.python .input}
x = np.arange(-1, 21, 0.01)
cmf = np.cumsum(pmf)
def F(x):
    return 0 if x < 0 else 1 if x > n else cmf[int(x)]

d2l.plot(x, np.array([F(y) for y in x.tolist()]), 'x', 'c.d.f.')
```

```{.python .input}
#@tab pytorch
x = torch.arange(-1, 21, 0.01)
cmf = torch.cumsum(pmf, dim=0)
def F(x):
    return 0 if x < 0 else 1 if x > n else cmf[int(x)]

d2l.plot(x, torch.tensor([F(y) for y in x.tolist()]), 'x', 'c.d.f.')
```

As we saw above, the means and variances are particularly concise.  If $X \sim \mathrm{Poisson}(\lambda)$, then:

* $\mu_X = \lambda$,
* $\sigma_X^2 = \lambda$.

This can be sampled as follows.

```{.python .input}
np.random.poisson(lam, size=(10, 10))
```

```{.python .input}
#@tab pytorch
m = torch.distributions.poisson.Poisson(lam)
m.sample((10, 10))
```

## Gaussian
Now Let us try a different, but related experiment.  Let us say we again are performing $n$ independent $\mathrm{Bernoulli}(p)$ measurements $X_i$.  The distribution of the sum of these is $X^{(n)} \sim \mathrm{Binomial}(n, p)$.  Rather than taking a limit as $n$ increases and $p$ decreases, Let us fix $p$, and then send $n \rightarrow \infty$.  In this case $\mu_{X^{(n)}} = np \rightarrow \infty$ and $\sigma_{X^{(n)}}^2 = np(1-p) \rightarrow \infty$, so there is no reason to think this limit should be well defined.

However, not all hope is lost!  Let us just make the mean and variance be well behaved by defining

$$
Y^{(n)} = \frac{X^{(n)} - \mu_{X^{(n)}}}{\sigma_{X^{(n)}}}.
$$

This can be seen to have mean zero and variance one, and so it is plausible to believe that it will converge to some limiting distribution.  If we plot what these distributions look like, we will become even more convinced that it will work.

```{.python .input}
p = 0.2
ns = [1, 10, 100, 1000]
d2l.plt.figure(figsize=(10, 3))
for i in range(4):
    n = ns[i]
    pmf = np.array([p**i * (1-p)**(n-i) * binom(n, i) for i in range(n + 1)])
    d2l.plt.subplot(1, 4, i + 1)
    d2l.plt.stem([(i - n*p)/np.sqrt(n*p*(1 - p)) for i in range(n + 1)], pmf,
                 use_line_collection=True)
    d2l.plt.xlim([-4, 4])
    d2l.plt.xlabel('x')
    d2l.plt.ylabel('p.m.f.')
    d2l.plt.title("n = {}".format(n))
d2l.plt.show()
```

```{.python .input}
#@tab pytorch
p = 0.2
ns = [1, 10, 100, 1000]
d2l.plt.figure(figsize=(10, 3))
for i in range(4):
    n = ns[i]
    pmf = torch.tensor([p**i * (1-p)**(n-i) * binom(n, i)
                        for i in range(n + 1)])
    d2l.plt.subplot(1, 4, i + 1)
    d2l.plt.stem([(i - n*p)/torch.sqrt(torch.tensor(n*p*(1 - p)))
                  for i in range(n + 1)], pmf,
                 use_line_collection=True)
    d2l.plt.xlim([-4, 4])
    d2l.plt.xlabel('x')
    d2l.plt.ylabel('p.m.f.')
    d2l.plt.title("n = {}".format(n))
d2l.plt.show()
```

One thing to note: compared to the Poisson case, we are now dividing by the standard deviation which means that we are squeezing the possible outcomes into smaller and smaller areas.  This is an indication that our limit will no longer be discrete, but rather a continuous.

A derivation of what occurs is beyond the scope of this document, but the *central limit theorem* states that as $n \rightarrow \infty$, this will yield the Gaussian Distribution (or sometimes normal distribution).  More explicitly, for any $a, b$:

$$
\lim_{n \rightarrow \infty} P(Y^{(n)} \in [a, b]) = P(\mathcal{N}(0,1) \in [a, b]),
$$

where we say a random variable is normally distributed with given mean $\mu$ and variance $\sigma^2$, written $X \sim \mathcal{N}(\mu, \sigma^2)$ if $X$ has density

$$p_X(x) = \frac{1}{\sqrt{2\pi\sigma^2}}e^{-\frac{(x-\mu)^2}{2\sigma^2}}.$$
:eqlabel:`eq_gaussian_pdf`

Let us first plot the probability density function :eqref:`eq_gaussian_pdf`.

```{.python .input}
mu, sigma = 0, 1

x = np.arange(-3, 3, 0.01)
p = 1 / np.sqrt(2 * np.pi * sigma**2) * np.exp(-(x - mu)**2 / (2 * sigma**2))

d2l.plot(x, p, 'x', 'p.d.f.')
```

```{.python .input}
#@tab pytorch
mu, sigma = 0, 1

x = torch.arange(-3, 3, 0.01)
p = 1 / torch.sqrt(2 * torch.pi * sigma**2) * torch.exp(
    -(x - mu)**2 / (2 * sigma**2))

d2l.plot(x, p, 'x', 'p.d.f.')
```

Now, let us plot the cumulative distribution function.  It is beyond the scope of this appendix, but the Gaussian c.d.f. does not have a closed-form formula in terms of more elementary functions.  We will use `erf` which provides a way to compute this integral numerically.

```{.python .input}
def phi(x):
    return (1.0 + erf((x - mu) / (sigma * np.sqrt(2)))) / 2.0

d2l.plot(x, np.array([phi(y) for y in x.tolist()]), 'x', 'c.d.f.')
```

```{.python .input}
#@tab pytorch
def phi(x):
    return (1.0 + erf((x - mu) / (sigma * torch.sqrt(torch.tensor(2.))))) / 2.0

d2l.plot(x, torch.tensor([phi(y) for y in x.tolist()]), 'x', 'c.d.f.')
```

Keen-eyed readers will recognize some of these terms.  Indeed, we encountered this integral in :numref:`sec_integral_calculus`.  Indeed we need exactly that computation to see that this $p_X(x)$ has total area one and is thus a valid density.

Our choice of working with coin flips made computations shorter, but nothing about that choice was fundamental.  Indeed, if we take any collection of independent identically distributed random variables $X_i$, and form

$$
X^{(N)} = \sum_{i=1}^N X_i.
$$

Then

$$
\frac{X^{(N)} - \mu_{X^{(N)}}}{\sigma_{X^{(N)}}}
$$

will be approximately Gaussian.  There are additional requirements needed to make it work, most commonly $E[X^4] < \infty$, but the philosophy is clear.

The central limit theorem is the reason that the Gaussian is fundamental to probability, statistics, and machine learning.  Whenever we can say that something we measured is a sum of many small independent contributions, we can assume that the thing being measured will be close to Gaussian.  

There are many more fascinating properties of Gaussians, and we would like to discuss one more here.  The Gaussian is what is known as a *maximum entropy distribution*.  We will get into entropy more deeply in :numref:`sec_information_theory`, however all we need to know at this point is that it is a measure of randomness.  In a rigorous mathematical sense, we can think of the Gaussian as the *most* random choice of random variable with fixed mean and variance.  Thus, if we know that our random variable has some mean and variance, the Gaussian is in a sense the most conservative choice of distribution we can make.

To close the section, Let us recall that if $X \sim \mathcal{N}(\mu, \sigma^2)$, then:

* $\mu_X = \mu$,
* $\sigma_X^2 = \sigma^2$.

We can sample from the Gaussian (or standard normal) distribution as shown below.

```{.python .input}
np.random.normal(mu, sigma, size=(10, 10))
```

```{.python .input}
#@tab pytorch
torch.normal(mu, sigma, size=(10, 10))
```

## Summary
* Bernoulli random variables can be used to model events with a yes/no outcome.
* Discrete uniform distributions model selects from a finite set of possibilities.
* Continuous uniform distributions select from an interval.
* Binomial distributions model a series of Bernoulli random variables, and count the number of successes.
* Poisson random variables model the arrival of rare events.
* Gaussian random variables model the result of adding a large number of independent random variables together.

## Exercises

1. What is the standard deviation of a random variable that is the difference $X-Y$ of two independent binomial random variables $X, Y \sim \mathrm{Binomial}(16, 1/2)$.
2. If we take a Poisson random variable $X \sim \mathrm{Poisson}(\lambda)$ and consider $(X - \lambda)/\sqrt{\lambda}$ as $\lambda \rightarrow \infty$, we can show that this becomes approximately Gaussian.  Why does this make sense?
3. What is the probability mass function for a sum of two discrete uniform random variables on $n$ elements?


:begin_tab:`mxnet`
[Discussions](https://discuss.d2l.ai/t/417)
:end_tab: