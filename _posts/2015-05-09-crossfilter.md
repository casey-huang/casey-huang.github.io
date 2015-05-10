---
layout: post
title:  "Crossfilter for NYC school-wise SAT and survey results"
date:   2015-05-09 22:34:25
categories: jekyll update
tags: featured
image: /assets/article_images/2015-05-09-crossfilter/desktop.JPG
---

This is a part of my data visualization class project that I find very rewarding. The result of this simple visualization is applicable for both exploratory and explanation purpose of various survey data, when there's a target metrics in which we are interested in finding the relationship with survey answers.

The purpose of the whole project is to explore the potential factors with strong correlation to schools' academic performance, which is measured by SAT scores. We considered school logistic factors (class size, gender ratio, number of AP classes, etc), environmental factors (income/poverty, crime rate, etc), and how students/parents/teachers' feedback of their experience with the school (mainly by survey data). 

###The datasets

> - NYC school survey dataset of year 2013 ([here](http://schools.nyc.gov/Accountability/tools/survey/2013.htm)). It's nice that the students and teachers' response rate is over 83%, and the parents' 66%. The scoring scale&method is also on this site.
> - NYC High school SAT score dataset of year 2012 ([here](https://data.cityofnewyork.us/Education/SAT-Results/f9bf-2cp4))

The cleaned datasets are joined on dbn, which is a unique identifier for schools. Since our main interest here is to study SAT performance, I didn't include the schools that didn't report SAT scores in the graph.

###The idea

I was thinking "Is there a way to visualize the distribution of certain sub-group of schools?" say, what's the distribution of the answer to 1st question by the school students whose is school average math score is in the lowest 10% percentile? We did cluster schools into high-scored, medium-scored, low-scored, and study other factors based on such binning. But I feel like want to play with a more flexible cutting point, and want the resulted distribution to be shown immediately. So I came up with the idea of doing it by crossfilter.

One more thing that's good about crossfilter is that we can condition on more than one subset, and see the effect of two factors together on the SAT score.

However, we don't want hundreds of independent variables on the screen, and the inter-correlation among questions are large, so I want to pick several "important" questions to have a closer look. The feature selection work is then done by fitting a LASSO logistic regression and picking the 6 factors with the largest absolute coefficients.

###The graph

It should be 3*2 so that everything is in one screen at the same time, but limited by the width of the blogpost, I changed the layout to 2*3, so you may want to zoom-out to view the whole visualization.

<iframe src="/assets/crossfilter/" width="650" height="1010" marginwidth="0" marginheight="0" scrolling="no" align="left"></iframe>


###Several findings at first sight

> - The students' survey has much stronger correlation with SAT performance than teachers' and parents'.
> - The schools in the upper tail of math score must have few gang activities, and students have to feel safe in class to get a high score.
> - Schools that are more friendly to disabled students get higher score.
> - Surprisingly, the schools where students' name are less known by the adults get higher score. This may be reasonable with one of our other findings, that school performance is better with larger class size.
> - The schools where the stuedents think the teaching staff don't expect them to get a higher education don't have good SAT performance.