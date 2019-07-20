---
layout: post
title: Thoughts on reference frames
usemathjax: true
comments: true
categories: reference-frame microbial-ecology
---

We just had a [paper](https://www.nature.com/articles/s41467-019-10656-5) come out about how to make sense of differential abundance in the context of microbial sequencing data.  The basic premise behind this work is that inferring differences (between sick/healthy people for instance) in microbial abundances from sequencing data can be counter intuitive.  We confirmed that fluctuations in total microbial biomass is a major confounder, and we have proposed a couple of different solutions.

Since the release of this paper, there has been some discussion surrounding this topic (which is great!). Recently, there was a blog post [here](http://dalmug.org/reference-frames/) that highlighted the major points in the paper, and also noted some points of confusion.  Here, we'll try to answer some of these questions.


> The major question we had was how reproducibly would different researchers select reference frames and to what degree would that alter the results. The authors give some suggestions for how to choose reference frames, but we are concerned that figuring out the ideal reference frame will be impossible in most cases and would also be susceptible to p-hacking.


This question is really important, since it can be abused without careful thought.  

There are a couple of good questions embedded here, namely how reference frames could alter results, and how to decide on a reference frame. The best analogy for thinking about reference frames is in terms of trying to measure velocity.  If you are on a train headed north going 100 k/h, and you are running from the back to the front of the train at 10 k/h, how fast are you traveling?

This will depend on the reference from which you are measuring.  Someone on the train may conclude that you are traveling at 10 k/h, but someone at the train station may concluding that you are traveling at 100 k/h.  Furthermore, someone on the opposite edge of the galaxy may give a completely different answer.

The point is, as long as you can agree on a consistent reference frame, the results should be consistent.  This was one of the points of Figure 3 on atopic dermitis - we postulated that one of the major reasons why different researchers couldn't agree on the role of Malassezia globosa in the context of atopic dermitis is because they are all using different reference frames.  By agreeing on using _P. acnes_ as a reference, we can start to see consistent results.

The next question is, how do you select the correct reference frame?  In some sense, this is the question that every differential abundance tool is trying to answer (i.e. aldex2 chooses the average, DESeq2 choses the median, phylofactor/philr chose based on phylogeny, ...).  What the ideal reference frame is likely going to depend on the problem of interest.  This is mainly because it isn't clear what exactly _ideal_ means.  Is it trying to find taxa that never change in abundance?  Or is it a matter of choosing microbes to maximize a certain classification criteria (see the paper on sebal [here](https://msystems.asm.org/content/3/4/e00053-18)).

The remark on p-hacking is adapt, and this warrants careful thought.  There are a couple of things that could help with this.  First, 
one can think of differential abundance as a measure of feature importance - differential abundance best investigated if there is a global change detected via beta diversity. For example, if you detect a notable global shift in the communities, you can say with high confidence that either the microbe with the lowest rank is changing, or the microbe with the highest rank is changing. This won't give a yes/no answer to determine if a microbe is differential, but this can help prioritze the most obvious players.

Second, cross-validation can help gauge the generalizability of a good reference frame - if one learns a model with a given reference, how predictive is it on data that it hasn't seen before?  Results that are p-hacked likely do not generalize across studies.

>It’s unclear what the consensus in the field is regarding measuring microbial load to convert relative abundances to absolute abundances. The authors make a strong argument for the need for improved CoDa methods especially for environments where determining microbial load is difficult and for pre-existing samples. However, for most future projects determining the total microbial load per sample would avoid the problems described in this paper.

This is an awesome question, definitely one worth more discussion.  Properly answering this questions means that we need to be on the same page about what it means to measure the total microbial load.  Within a given microbiome survey experiment, there are multiple stages of subsampling -- for the sake of simplication, we'll just note both sequencing and sample collection are both subsampling procedures.

This implies that there are a few layers of ___absolute abundances___.  You have the total number of microbial cells within a sample, and then you have the total number of microbial cells that are in the entire environment.  If we look at classic examples in ecology, an ecologist in the rain forest maybe able to count the number of jaguars within a square kilometer radius in the rainforest, but will have trouble counting all of the jaguars in the rainforest.  In microbial ecology, we'll expect the same to hold - even if we are able to count all of the microbes in a fecal sample, measuring the total number of microbes in the environment maybe impractical (i.e. requiring one to cutdown the rainforest / dice up a patient / sequence the entire ocean to count every microbial cell in said envrionment).  


> We’re not sure why measuring salivary flow rate was needed to estimate microbial load if flow cytometry was also performed.

This is very relevant to the previous question.  Techniques that measure absolute abundances (i.e. qPCR, Flow cytometry), are in reality measuring the cell concentration within a sample, not the absolute abundances.  In some cases, we can use these measurements to obtain an estimate of the true absolute abundance in the environment.  In the oral time series study, the saliva flow rate can serve as a proxy for the total volume in the oral cavity. With this in mind we can obtain an estimate of cells since

$$
\frac{\#cells}{ml} \times ml = #cells
$$

> We were confused why Propionibacterium acnes was chosen as a reference frame in the atopic dermatitis example - wouldn’t it make more sense to use a prevalent species with a low log ratio?

_P. acnes_ was chosen for two reasons : it was a highly abundant species (i.e. found in most samples) and it had a high rank (i.e. it was highly associated with the samples after flare).

