---
layout: post
title: How to do differential abundance in 10 MINUTES!
usemathjax: true
comments: true
categories: linear-regression data analysis statistics computer-science data-science
---

If you are reading this page, you may be shopping around for different differential abundance tools to determine what microbes have been altered across treatment conditions.

If you are working on RNASeq or microbiome, you may want to explore limma / voom, edgeR, DESeq2 or aldex2.
If you follow my work, you may have noticed that I developed my own differential abundance software tool [songbird](https://github.com/biocore/songbird).
How can you pick?  There are so many options!?!

One thing that is worth realizing is that all of the tools mentioned above rely on one core concept : generalized linear models.
My philosophy is that one of the best ways to understand a concept is to implement it yourself.
Here, I'm going to try to demonstate how you can build your own differential abundance tool and run it in just a few minutes.
We'll break this up into 3 sections.

1. The negative binomial distribution
2. Compositional transforms
3. Piecing it altogether in Stan

# The negative binomial distribution

This is one of the standard distributions used to count sequences.  This the distribution choosen for edgeR and DESeq2 and there are
quite a few resources on it (see [here](https://en.wikipedia.org/wiki/Ladislaus_Bortkiewicz),
[here](https://divingintogeneticsandgenomics.rbind.io/post/negative-binomial-distribution-in-scrnaseq/) and
[here](http://www.nxn.se/valent/2018/1/30/count-depth-variation-makes-poisson-scrna-seq-data-negative-binomial).
If there is still widespread confusion on this topic, maybe I'll go into more detail behind this in a future blog post.
For now, we will treat this as a means to model counts with extra technical variability.

Here, we will model each microbe as a negative binomial distribution given by

$$
y_{ij} \sim NB(\mu_{ij}, \phi_i)
$$

where $y_ij$ denotes the counts for microbe $j$ and sample $j$ that we are trying to model. $mu$ denotes the expected abundance for microbe $j$ and sample $j$ and $\phi_j$ denotes the dispersion parameter for microbe $j$. This dispersion parameter enables us to allow learn the variance of microbe $j$, since $V(y_{ij}) = \mu_{ij} + \frac{1}{\phi_j}\mu_{ij}$.  This will ultimately allow us to build a more robust / flexible model, which is really important given how high variance many of these sequencing datasets are (see paper [here](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003531)).

While understanding $\phi$ is important, the actual gold in the model is contained in $\mu_{ij}$.  If you are trying to infer differences between two treatment groups, you likely
will want to do something that can take the difference between $\mu_{ij}$ and $\mu_{kj}$ for samples $i$ and $k$.  To do this, we will need to carefully break it down, leading us to the next section on compositionality

# Compositionality

The consensus that has been made in the field is that sequencing measurements are relative, not absolute (see paper [here](https://www.ncbi.nlm.nih.gov/pubmed/29143816) and our paper [here](https://www.nature.com/articles/s41467-019-10656-5)). This means that the total microbial counts do not reflect upon the true microbial counts in the environment, and that the only information obtained from sequencing are microbial proportions.  This means that there are two things to consider, namely the sequencing depth and the proportions.  Since the sequencing depth does not correlate with the microbial biomass, we will treat it as a nuissance variable, leaving us to only infer the microbial proportions.  In other words, we will break up $\mu_{ij}$ into two parts as

$$
\mu_{ij} = n_i \times p_{ij}
$$

where $n_i$ is the observed sequencing depth for sample $i$ and $p_{ij}$ are the proportions of microbe $i$ in sample $j$ that we would like to model.
For the compositional skeptics, we can model the microbial abundances as independent negative binomial distributions, due to the connection between the Poisson and the Multinomial distribution (see [here](https://arxiv.org/abs/1311.6139)).  So as long as we can control for the observed sequencing depth, we can model the proportions.

The next question is, how do we model the proportions?  A sure fire way to do this is to use the alr transform (see this wikipedia page [here](https://en.wikipedia.org/wiki/Compositional_data#Linear_transformations)).  Specifically this will give us

$$
alr(p_{i}) = x_i \cdot \beta
$$

Where $x_i$ specifies the experimental conditions you wish to investigate and $\beta_j$ are the *log fold change* values of microbial abundances across the experimental conditions.
These are the values that you want to estimate. We put quotes around the log fold change since you have to be careful about how to interpret it.
See our paper [here](https://www.nature.com/articles/s41467-019-10656-5)).

Taking together everything that we discussed, the generative model can be built as follows

$$
\beta_j \sim N(0, 5)
1/\phi_j \sim Cauchy(0, 5)
p_{i} = x_i \cdot \beta
\mu_{i} = n_i \times alr^{-1}(p_{i})
y_{ij} \sim NB(\mu_{ij}, \phi_j)
$$

We specified relatively uninformed priors for $\beta_j$ and $\phi_j$ - from previous observations most microbes fluctuate within 5 orders of magnitude.

# Piecing it altogether in Stan

We can code this up with a few lines of Stan code.  This will also give us posterior estimates of the parameters, which can help us quantify the
certainity of the parameters we estimated.

The stan code is given as follows (which can also be found [here](https://gist.github.com/mortonjt/227a0058194c2fa0ecf06807e1315d35)).
To run this later, we will save this code under `model.stan`.

```
data {
  int<lower=0> N;    // number of samples
  int<lower=0> D;    // number of dimensions
  int<lower=0> p;    // number of covariates
  real depth[N];     // sequencing depths of microbes
  matrix[N, p] x;    // covariate matrix
  int y[N, D];       // observed microbe abundances
}

parameters {
  // parameters required for linear regression on the species means
  matrix[p, D-1] beta;
  real reciprocal_phi;
}

transformed parameters {
  matrix[N, D-1] lam;
  matrix[N, D] lam_clr;
  matrix[N, D] prob;
  vector[N] z;
  real phi;

  phi = 1. / reciprocal_phi;

  z = to_vector(rep_array(0, N));
  lam = x * beta;
  lam_clr = append_col(z, lam);

}

model {
  // setting priors ...
  reciprocal_phi ~ cauchy(0., 5.);
  for (i in 1:D-1){
    for (j in 1:p){
      beta[j, i] ~ normal(0., 5.); // uninformed prior
    }
  }
  // generating counts
  for (n in 1:N){
    for (i in 1:D){
      target += neg_binomial_2_log_lpmf(y[n, i] | depth[n] + lam_clr[n, i], phi);
    }
  }
}
```

If you have a biom table of abundances, you can run it through pystan as follows

```python
from biom import load_table
import pandas as pd
from patsy import dmatrix
import pystan

table = load_table('<your table>')
metadata = pd.read_table('<your sample metadata for experimental conditions>', index_col=0)
formula = '<your formula>'  # this can be something like 'C(Treatment) + C(Gender) + age'
X = dmatrix(formula, metadata, return_type='dataframe')
Y = table.to_dataframe().to_dense().T

dat = {
    'N' : X.shape[0],
    'D' : table.shape[0],
    'p' : X.shape[1],
    'depth' : np.log(table.sum(axis='sample').values),
    'x' : X.values,
    'y' : table.values.astype(np.int64)
}

code = open('model.stan', 'r').read()
sm = pystan.StanModel(model_code=code)

# Run Hamiltonian Monte Carlo
fit = sm.sampling(data=dat, iter=1000, chains=4)
```

Depending on the size of your dataset, it may take some time to run the model (it took 2 hours to run on a dataset with 200 samples and 1000 microbes on my laptop).
Once the model is fitted, you can extract your parameter estimates as follows

```
res =  fit.extract(permuted=True)
```

This will give you draws from the posterior distribution, so we will want to compute means and variances to directly interpret our estimate parameters.
I'll provide some code used for a recent case study below.  We can pull out the log fold changes from $\beta$.  Here, $\beta$ is in alr coordinates, meaning that
we chose the first microbe to be the reference frame in this example.  To help visualize these results, we can shift the reference frame to the average, hence
converting to clr coordinates. There is more discussion about this in our [reference frames paper](https://www.nature.com/articles/s41467-019-10656-5)), but
I can have another blog post further clarifying this if necessary.  For now, we can build an `alr2clr` function and visualize these results.

```python
def alr2clr(x):
    d = x.shape[0]
    x_clr = np.hstack((np.zeros((d, 1)), x))
    x_clr = x_clr - x_clr.mean(axis=1).reshape(-1, 1)
    return x_clr
```
In the original study, there were 6 covariates.  Here, we will only focus on the 4th covariate, which gives us information about the microbial fold change across disease type.

```python
beta = alr2clr(la['beta'][:, 3, :])
beta_df = pd.DataFrame(beta, columns=table.ids('observation'))
```
Here, the dimensions of beta are `(number of microbes ) x (posterior samples)`.  We can summarize these samples in terms of means and standard deviations as follows

```python
std = beta_df.std(axis=0)
mean = beta_df.mean(axis=0)
mean = mean.sort_values()
std = std.loc[mean.index]
```

We can visualize the estimated log fold change for each microbe (i.e. the microbial differentials) as follows

```python
plt.figure(figsize=(10, 5))
i = np.arange(beta_df.shape[1])
plt.errorbar(i, np.array(mean.values), yerr=np.ravel(std.values))
#plt.ylabel('Log fold change', fontsize=14)
plt.ylabel(r'log(Control / Disease)+K', fontsize=14, rotation=0, labelpad=90)
plt.xticks([])
plt.xlabel('Microbes', fontsize=14)
plt.axhline(0, c='r')
```

![differentials plot](https://github.com/mortonjt/probable-bug-bytes/raw/master/images/microbe_fold_change_11252019.png "Differentials")

And now we can see all of the log fold change estimates for each microbe along with confidence intervals.
There are a few things to note here.

First, this is a visualization of the microbe differentials in clr coordinates.  This means that zero here does not imply no change, but rather the *average*.
We cannot infer from this which microbes have changed, we can only infer the ranking of the microbes (i.e. which microbes increased / decreased the most relative to the other microbes).

Second, we don't have proper model diagnostics here.  So we don't know how well this model generalizes across different studies.  Posterior predictive checks can
help with evaluate how well the model can predict unobserved samples as a form of cross-validation.  Wee this stan writeup [here](microbe_fold_change_11252019.png) for more details.
The stan code will need to be augmented to enable this.

Third, the posterior samples from this model can serve as a drop-in replacement for commonly used significance tests such as t-tests. For instance, one can convert these differentials
to log ratios (i.e. balances) to test how different ratios of microbes change across conditions.  The statistical test criteria can be applied to each of the posterior samples,
and the results can be counted up to determine statistical significance. Another blog post on this maybe necessary.

Maybe you can read all of this in 10 minutes.  But chances are, it'll take some time to digest all of this material.
Hopefully this serves as a guide towards how you can build your own statistical models from scratch, opening the doors for more statistical insights.