---
layout: post
title: NB1 and NB2 Dispersion
date:   2016-12-03 10:50:00
tags: R simulation
subclass: 'post tag-fiction'
navigation: True
logo: 'assets/images/ghost.png'
cover: 'assets/images/cover1.jpg'
author: hackz
categories: hackz
---

The other day I started working with a colleague on propagating uncertainty for
a negative binomial generalized linear model. To my surprise I found that there
was a couple of different ways to define the probability distribution that led
to more than a bit of confusion on my end, but got me messing around with how
you could simulate over-dispersed count data in a couple of ways. It also got me
working with TMB again and the woes of dealing with nonlinear optimization.

The first model is a mixture distribution model which incorporates a poisson
distribution but also a gamma distribution to account for the extra variance.
from what I have seen this is referred to as an NB1 model and looks like this.

$$
y \sim Poisson(Gamma(\mu / \alpha, \alpha))
$$

Our distribution has a mean of $$\mu$$ but that persists through both
distributions but the variance is greater than just a regular poisson
distribution because of the randomness introduced by the Gamma distribution
which itself has a variance of $$\mu * \alpha$$. Getting this to work for a
linear model with a log link required me to introduce a new random
variable $$\nu$$ which would account for the over-dispersion and a full log link
model follows the form

$$
\nu \sim gamma(1/\alpha, \alpha)
$$

$$
\mu = exp(X \dot \beta + \nu)
$$

$$
y \sim Poisson(\mu)
$$

where $$X$$ is my matrix of covariate data and $$\beta$$ are their coefficients
for the linear model.

Simulating this data was cake but when I used TMB to fit the model I found
that my model often failed to converge depending on what the start values of my
parameters were. I could get around it by fitting the model with out $$\nu$$
first to get good start values for $$\beta$$ but even so the model was slow to
fit.

Alternatively we could model the extra variance directly without any extra term
$$\nu$ by using a negative binomial function that comes built in with both R and
TMB. This way saves us the hassle of having to simulate from two different
distributions and the two parameter model can easily be set up so that it has
a mean setting parameter and a variance parameter. The NB2 as well call it looks
like this.

$$
y \sim NB2(\mu, \theta)
$$

$$
E(y) = \mu
$$

$$
Var(y) = \mu + (\theta / \mu^{2})
$$

Fitting a log linear model into here just means estimating $$log(\mu)$$ with
the same $$X \dot \beta$$ without the $$\nu$$ parameter. The nice thing is we
still get the same distribution as the first model where $$\alpha$$ corresponds
to $$1/\theta$$. The even nicer thing is that I didn't have any problems fitting
this model and it was way faster for TMB due to not having a whole other set of
parameters to estimate.

A working version of the code exists [here](https://github.com/nmmarquez/re_simulations/blob/master/nb_compare/nb_tmb.R) 
