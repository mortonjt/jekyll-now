---
layout: post
title: 7 tips on how to collaborate with your fellow data analyst.
usemathjax: true
comments: true
categories: collaboration data analysis statistics computer-science data-science
---

Do you have a data problem? Are you in need of a data analyst to
generate polished gorgeous figures, or just a informatician to run some pipelines for you?

At this point in my career, I have worked with a number of collaborators across multiple
biological discliplines. While I have blessed with the opportunity to work with
truly awesome people, there are definitely scenarios that I recommend to avoid in the
future, or at least guard against.  If you are reading this and are in the process
of working with a statistician / data scientist / computer scientist / analyst,
some of the tips below could help tighten your communication with them and enable
a productive working relationship.


## Rule 1: Try to learn about the details behind the analysis.

You didn't take computer science / statistics / mathematics / physics as a major in college?
Don't fret about it!  Part of doing cutting edge research is about getting into the deep end of
the pool.  But if you are asking someone to run an analysis for you, you should try to obtain
a high level overview of what you are asking them to do. It is definitely ok to ask questions here.

The reverse is also true -- the analyst should try to obtain some background behind the problem.
You should also be prepared to carefully motivate the problem at hand as well.
Love goes both ways here.

## Rule 2: Be open minded.

You ideas may suck. This does happen.  A lot.
There are many reasons why you cannot answer particular questions from your data.
For instance,  your data may not be normally distributed, complicating statistical tests.
Or you may have extremely unbalanced classes, complicating classification.
Or god forbid, you are trying run ttest / Pearson correlation to test for differences across proportions - in which case, you would be better off throwing away your data, and instead flip coins (but seriously, ttest / Pearson perform worse than random on proportions - there's over 40 years of mathematical research that shows this).

Be mindful of data scientists, and avoid micromanaging them.  When they tell you that something is
not possible, chances are they are up to something.  They aren't acting like a bunch of grammar Nazis,
its worth hearing our what they have to say.

## Rule 3 : Don't tell your collaborator to Google it.

There are definitely scenarios where it is warranted to send a [letmegooglethat](http://letmegooglethat.com) link to someone.  Google search has been a huge enabler of searching new content.
However, it is still not quite up to research standards yet, and can be very challenging to learn
new content.  For instance, if someone asks you about the background behind your experimental design
or or recommended papers on microbial exometabolites, telling them to google it is not only
completely unreasonable.  It is fucking rude. You wouldn't want someone to tell you to google
VAEs / GLMs without any context. Would you?

Rather than being a total asshole, try to follow up papers to provide motivation to your problem.
Review papers are great for this.

For the analysts reading this, if your collaborator tells you google something without any context -- GTOF!

## Rule 4 : If possible consult with analysts _before_ you run an experiment.

There is a rather large graveyard of botched experiments, some of them costing up to
millions of dollars. You don't want your experiment to join that club.

Not having the proper statistical expertise could cause biases / contamination to crept
in your data, drastically reducing the questions that you can answer.

Its really in your best interest to get a statistician / data scientist on board
before you even start collecting data.

## Rule 5 : Clean up your data before you pass it off.

Try to clean up your data to the best of your abilities before you pass it off.
This will save the analyst a lot of headache, which in turn will save you a lot of headache.
This means trying to avoid ambiguity in your files and having a well-defined file format.
If you have a procedure that wouldn't make sense to your grandma without any context -
you probably need to document it.  In my field, much of the experimental information
ends up being written on the side of a DNA extraction tube, so often times the sample name
has weird esoteric numbers and letters on it.  Make sure to have this well spelled out
either in a README and (preferrably) spreadsheet.

If an analyst has had experience with collaborations, they probably have nailed down
a procedure for file formats and pipelines.  The first few times maybe a learning curve,
but it is definitely worthwhile to try to get a feel for the workflows
and understanding what file formats are required. And trust me, you don't want to
end up in a situation where in order for the analyst to run the analysis, they
have to read your mind in order to figure out what experiment you ran.
That just sucks.

## Rule 6 : Don't expect analysts to clean up your mess.

If you botched your experiment or passed over sloppy data, don't expert analysts
to whip up some mathemagic to solve your problems.  Own up to your problems -
fix what you can to get the analysts the data that they need to make sensible conclusions.

Furthermore, you shouldn't expect the analyst to generate beautiful figures without any feedback.
You should be prepared to spend time understanding the analyses and providing some domain specific
context. If in the offchance the analyst was able to design the experiment, generate polished figures
with a complete story - then there needs to be a serious discussion on
co-first authorship / promotion. Don't be that asshole to minimizes your analysts contributions to
live in the acknowledgements section.

## Rule 7 : Figures, figures, figures

If you got this far, you should be excited - your work is already shaping up into a story
and it is getting close to share.

If you are the main author piecing together the story, then it is in your best interests to
minimize the amount of back and forth.  What I mean by this is try to minimize the number
of figure requests.  You don't want your email to blow up over requests to change
the colors, font size and styles. Hundreds of emails can easily be wasted during this time
and will honestly create more confusion. Slack can help mitigate this issue,
but this won't solve the problem.

Rather than micromanaging your collaborator and expect them to read your mind and poop out a figure
exactly how you envisioned it, this is an opportunity for you to take some initiative.
If at all possible try to get SVGs - this can allow you to change the fonts and colors
how ever you like in your favorite editor (photoshop, inkscape, gimp, just pick one!).
If you can get the raw data / code used to generate the plot, even better -- just regenerate those results in your favorite environment (R / Python, or if you really hate yourself, Excel).

I know how being the primarily coordinator can be overwhelming at times, but the
more things that are within your control the better off you are, and the more likely
the project will actually finish.

### The takeaway

Collaborations can be tricky - it brings in a human component into the equation.
But they can also be huge enablers to new and exciting problems that couldn't have been realized
otherwise! Don't be discouraged, it is a learning experience.
