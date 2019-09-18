---
layout: post
title: Making biom functional
usemathjax: true
comments: true
categories: functional-programming biom microbiology
---

If you are reading this page, chances are you haven't heard about functional programming.

And that's ok! I'm by no means and expert in it either - but having some functional skills up your sleeve is very much worthwhile to learn as a data scientist.

Wikipedia has a more formal definition here [Functional programming](https://en.wikipedia.org/wiki/Functional_programming).  The main takeaway here is that in a functional programming paradigm, the functions themselves can act like inputs.  The top 3 python blogposts that came up on my google search go into quite a bit of detail on the benefits of this (see [here](https://maryrosecook.com/blog/post/a-practical-introduction-to-functional-programming) [here](https://www.dataquest.io/blog/introduction-functional-programming-python/) and [here](https://kite.com/blog/python/functional-programming/)) so feel free to take a look at those.

The main purpose of this blog post is to show that some of these concepts can be directly applied out of the box to the [biom table](https://www.ncbi.nlm.nih.gov/pubmed/23587224).

The biom format has the reputation of being a black box object that microbiologists shuttle back and forth from program to program.  But actually, it far from black box - it is just a sparse matrix that can easily be saved to disk (see an old blog post on this [here]).

One of the cool things about this object is that you can relatively easily query this object, even if you have thousands of samples and tens of thousands of species.

Here's an example, suppose that I want to identify the most abundant microbe within each sample.
The easiest way to do this is to convert the table into a dense pandas table and just argmax.
To see for yourself, try pulling down a biom table [here](https://qiita.ucsd.edu/study/description/10863) and play around with it.  I've renamed this file to `otu_nts.biom`, so this command will look like as follows

```python
>>> from biom import load_table
>>> import numpy as np
>>> table = load_table('otus_nt.biom')
>>> sparse_table = table.to_dataframe()
>>> dense_table = sparse_table.to_dense()
>>> most_abundance_microbes = dense_table.apply(np.argmax, axis=0)
```

Let's break this down. `load_table` loads the biom table into memory as a sparse matrix.
The `to_dataframe` method converts this biom table into a sparse pandas DataFrame.
As the moment of this writing, it isn't clear to me how to manipulate the sparse pandas dataframe.
So we'll conver this into a dense matrix representation using `to_dense` method.

Finally, we will use the `apply` method to get the most abundant microbes.
This method will apply a function (here, we are using the `np.argmax` function) to each row of the table.

If you peek into the most_abundant_microbes, you'll get a list of sequences per sample like as follows
```python
>>> most_abundance_microbes.head()
10863.259.BC.5     TACGTAGGTGGCAAGCGTTGTCCGGATTTATTGGGCGTAAAGCGAG...
10863.157.1.a      TACGAAGGGTGCAAGCGTTAATCGGAATTACTGGGCGTAAAGCGCG...
10863.148.BC.10    TACGTATGTCGCGAGCGTTATCCGGAATTATTGGGCATAAAGGGCA...
10863.148.BC.1     TACGAAGGGTGCAAGCGTTAATCGGAATTACTGGGCGTAAAGCGCG...
10863.243.TB.5     TACGGAAGGTCCAGGCGTTATCCGGATTTATTGGGTTTAAAGGGAG...
dtype: object
```

If you try this out - you'll notice that (1) it takes a while and (2) the memory footprint is huge compared to the biom table.

```
>>> import sys
>>> sys.getsizeof(dense_table)
40019634
>>> sys.getsizeof(table)
>>> 56
```


That's because you have a few dense tables that are storing a ton of zeros.  As far as we are concerned,
that is useless / redundant information.

You can get away with storing the exact same amount of information using biom sparse representation.
Plus, it'll be much faster.

Below is how to can do this with biom

```python
from biom import load_table
import numpy as np
otu_ids = table.ids(axis='observation')
sample_ids = table.ids(axis='sample')

def argmax_f(x):
    v, i, m = x
    return otu_ids[np.argmax(v)]

gen = map(argmax_f, table.iter(axis='sample'))
most_abundant_microbes = pd.Series(list(gen), index=sample_ids)
```

Here, I build a function `argmax_f` that unpacks each element from `iter` (see [docs](http://biom-format.org/documentation/generated/biom.table.Table.iter.html#biom.table.Table.iter)) for more details.  The ordering is important, the first element v corresponds to the values of that column (sample), i corresponds to the column id and m corresponds to the metadata corresponding to that column.  Once we have these values, we can look up the most abundant microbe using `np.argmax`.

And you can see, this will return the exact results

```python
>>> most_abundance_microbes.head()
10863.259.BC.5     TACGTAGGTGGCAAGCGTTGTCCGGATTTATTGGGCGTAAAGCGAG...
10863.157.1.a      TACGAAGGGTGCAAGCGTTAATCGGAATTACTGGGCGTAAAGCGCG...
10863.148.BC.10    TACGTATGTCGCGAGCGTTATCCGGAATTATTGGGCATAAAGGGCA...
10863.148.BC.1     TACGAAGGGTGCAAGCGTTAATCGGAATTACTGGGCGTAAAGCGCG...
10863.243.TB.5     TACGGAAGGTCCAGGCGTTATCCGGATTTATTGGGTTTAAAGGGAG...
dtype: object
```

If we compare the timings of conversion and argmax with the pandas version, we can see it is quite slow compared to the biom version.
```python
%timeit table.to_dataframe().to_dense().apply(np.argmax, axis=0)
22.4 s ± 104 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

```python
%timeit list(map(argmax_f, table.iter(axis='sample')))
60.2 ms ± 302 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

Besides, if your table is sufficiently big, you may not even be able to load it - in those scenarios working directly with biom tables can be very useful.

We just showcase this with finding the most abundant microbes, but you can imagine this can be applied to a wide variety of settings, including log transforming abundances or data subsampling.  Looking forward to seeing more applications!