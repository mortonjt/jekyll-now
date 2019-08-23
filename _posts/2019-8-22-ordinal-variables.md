---
layout: post
title: How to account for ordering of categories in your differiential abundance analysis
usemathjax: true
comments: true
categories: statsmodels differential-abundance ordinal-variables
---
What are ordinal variables?  Ordinal variables are really just categorical variables with an ordering.
This can be especially useful for disease categorization, sometime patients are recorded to have a more severe onset of disease compared to others.
While the measurements are categorical, it would be ideal if some ordering information was enforced.

The main idea behind ordinal variables is that you can enforce this ordering in your design matrix.
I wrote about design matrices and catergorical variables on my old blog [here](http://mortonjt.blogspot.com/2018/05/encoding-design-matrices-in-patsy.html).
This ordering is ca be enforced using a __backwards difference encoding__.  Here, variables are only compared against adjacent levels.

For instance, if we have 3 levels (i.e. healthy, mild and severe), a backwards difference would first compare severe to mild, then mild to healthy.
This willl ultimately give you two dummy variables : severe vs mild and mild vs healthy.

Fortunately, there are quite a few resources that can enable this sort of analysis with basically zero overhead.  Patsy, the formula backend engine behind statsmodels
supports this sort of analysis - see [here](https://www.statsmodels.org/dev/contrasts.html#backward-difference-coding).

Since [Songbird](https://github.com/biocore/songbird) relies on patsy for formulas, it too can do ordinal differential abundance analysis.
Here is an example command that you can run in songbird using a biom table of microbe abundances `microbes.biom` with covariates saved under `sample-metadata.txt`

```bash
songbird multinomial \
    --input-biom microbes.biom \
    --metadata-file sample-metadata.txt \
    --formula 'sample_site + antibiotics + C(symptom, Diff, levels=["healthy", "mild", "severe"])' \
    --summary-dir summary
```

Of course, most major statistical software packages support this sort of analysis - so this can be done using [scikit-learn](https://contrib.scikit-learn.org/categorical-encoding/backward_difference.html) and [R](https://stats.idre.ucla.edu/r/library/r-library-contrast-coding-systems-for-categorical-variables/#backward)!
