---
layout: post
title: Move that ELBO! Providing intuition behind Variational Bayes
usemathjax: true
comments: true
categories: elbo bayesian-inference variational-inference statistics computer-science data-science
---

Here I'm going to try to another perspective on variational inference, in particular some intuition behind the mysteriour ELBO.
Chances are you stumbled on variational inference through [VAEs](https://arxiv.org/pdf/1312.6114.pdf) introduced by Max Welling, or David Blei's work with [LDA](http://jmlr.org/papers/volume3/blei03a/blei03a.pdf) and other applications.

The derivations in these works are typically hinged on the task of estimating a joint distribution of observed variables.  Below is an example from the VAE paper

$$
p(x) \geq \mathbb{E}_{q(z|x)}[\log p(x|z)] - KL(q(x|z)||p(z))
$$

I've removed the extra notation to simplify things.  But still, if this is the first time you've seen this, you may have a number of questions.  There are many funky things going on with the notation.  First off, what is going on with the expectation and how does $$q(z\|x)$$ play a role in this? Why does $$p(x)$$ play a role here? And exactly how is this inequality derived?

The most simplified attempt that I've seen at explaining variational inference is David Blei's review [here](https://amstat.tandfonline.com/doi/pdf/10.1080/01621459.2017.1285773?needAccess=true).  Note that there is nothing wrong in these derivations, but I argue that an even simpler derivation can be obtained directly from Baye's Theorem.
n
# Deriving that ELBO from Bayes Theorem.

Bayes Theorem is given as follows

$$
p(z | x) =  K \; p(x|z) \; p(z)
$$

This is as simple as it looks.  $$p(z)$$ is the prior distribution of $$z$$, an unknown parameter of interest. The prior can be thought of as the assumed distribution of $$z$$, we don't necessary know exactly what value $$z$$ has, but often we can place some bounds on where $$z$$ is likely to exist. This is where model assumptions can be enforced.
The middle term, $$p(x|z)$$, is the likelihood.  In other words the probability of the data being generated from the distribution governed by $$z$$.
The far left term, $$p(z|x)$$ is what is called the posterior distribution.  This distribution reflects the new belief of where $$z$$ is likely to be found.
We have included a normalization constant $$K$$ to make this posterior distribution a true probability distribution. In practice, this can be difficult to calculate, but we will show shortly how this drops out. Altogether, this equation showcases how information can be propagated with incoming data. Given some idea of what the unknown parameter is, we can update our belief of what this parameter is given more data.  This process is known as a Bayesian update.

In a perfect world, we would be able to just apply Bayesian updates seamlessly, calculating posterior distributions with just addition and multiplication without the need for fancy methods.  In fact, with certain distributions, this can be done.  These are known as [conjugate distributions](https://en.wikipedia.org/wiki/Conjugate_prior).

OK, then where do problems arise?

The problem arises when you try to obtain a closed form solution for the distribution $$p(z|x)$$, but solving those integrals just becomes too hard.
How hard is too hard?  It is actually [NP-hard](https://www.sciencedirect.com/science/article/abs/pii/000437029390036B) - trying to solve any arbitrary integral can lead to a combinatorial explosion of possibilities.

OK, so what do we do?

One possibility is to try to simplify the problem, and make it easy enough for us to solve. We assume a much simpler form of the posterior distribution and try to solve for that instead.
In other words, we introduce a variational distribution $$q(z|x)$$ and try to match it as best as we can against the posterior distribution. As Blei notes in his review, we want to try to minimize the following objective

$$
argmin_q KL(q(z|x) || p(z|x))
$$

where the [KL divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence) measures the difference between the posterior distribution $$p(z|x)$$ and the approximate distribution $$q(z|x)$$.  In the variational inference literature, it is more commom to maximize negative KL divergence, which is equivalent to minimizing the KL divergence.

If you write out the form of the KL divergence, and squint at it, you may realize that it is actually an expectation with respect to $$q(z|x)$$.

$$
- KL(q(z|x) || p(z|x)) = \int{q(z|x) \log \frac{p(z|x)}{q(z|x)} dx = \mathbb{E}_{q(z|x)}\bigg[\log \frac{p(z|x)}{q(z|x)}\bigg]
$$
This is the shorthand for what these expectations are actually referring to.

If we expand the KL divergence term with respect to the likelihood and the prior, we will get the following
b
$$
- KL(q(z|x) || p(z|x)) = \mathbb{E}_{q(z|x)}\bigg[\log \frac{p(x|z)p(z)}{q(z|x)} + \log K\bigg] \geq \mathbb{E}_{q(z|x)}\bigg[\log \frac{p(x|z)p(z)}{q(z|x)}\bigg]
$$

This inequality is true because the expectation of K will be stricly greater than zero. That quantity we just derived is the evidence lower bound (ELBO), which we can directly maximize to find the approximate posterior distribution.  If we factor this further, we can obtain the results as presented in Welling et al
$$
\mathbb{E}_{q(z|x)}\bigg[\log \frac{p(x|z)p(z)}{q(z|x)}\bigg] &= \mathbb{E}_{q(z|x)}\bigg[\log \frac{p(x|z)}{q(z|x)} \bigg]+ \mathbb{E}_{q(z|x)}\bigg[\log \frac{p(z)}{q(z|x)}\bigg]
&= \mathbb{E}_{q(z|x)}[\log p(x|z)] - KL(q(x|z)||p(z))
$$

In short, you don't need to understand the joint distribution to understand what variational inference is actually optimizing. The only thing needed here is to understand Bayes Theorem, and the appropriate objective functions can be directly obtained form there.