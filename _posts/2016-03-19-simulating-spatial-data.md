---
layout: post
title: Simulating Spatially Correlated Data
date:   2016-03-19 21:47:00
tags: spatial R simulation
subclass: 'post tag-fiction'
navigation: True
logo: 'assets/images/ghost.png'
cover: 'assets/images/spatial_autocorrelation.jpg'
author: hackz
categories: hackz
---

A couple of weeks ago I ran into a problem where I needed to create some spatially correlated data for a project and I
realized that I didn't know the first thing about how to go about doing it. I was lucky to find that the
[paper](http://arxiv.org/abs/1601.01180) that I was trying to replicate did a great job of detailing their methods in
such a way that I was able to create some spatial data of my own. The idea is that you need to create a covariance
matrix of the relationships of your data to pull form a multivariate random normal distribution.

$$
\mathcal{N}(\mu, \Sigma)
$$

$$\mu$$ is just the scalar mean and can be set to any real number but defining a sensible covariance matrix was
originally difficult. The way that Riebler et al. did it was by using a matrix Q

$$
Q_{i,j}=
\begin{cases}
    n_{\delta_{i}},& \text{if  } i = j \\
    -1,              & \text{if  } i \sim j \\
    0, & \text{otherwise}

\end{cases}
$$

where $$\delta_{i}$$ is the number of neighbors a location has and $$i \sim j$$ means that the two locations are
adjacent. Im used to working with spatial polygons data frames in R and found that its pretty easy to create this
matrix with the right packages. Here's an example with US state shape file.

    library(sp)
    library(surveillance)
    library(spdep)
    library(MASS)

    # load in some USA data to work with
    load(url("http://biogeo.ucdavis.edu/data/gadm2/R/USA_adm2.RData"))
    cont_usa_locs <- c("Texas", "Louisiana")

    # spatial polygons data frame to simulate from
    cont_usa <- gadm[(gadm@data$NAME_1 %in% cont_usa_locs),]
    cont_usa$ID <- 1:length(cont_usa)

    # adjacency matrix for spatial data 1's if adjacent 0 otherwise
    adjmat <- poly2adjmat(cont_usa)

    # Create Q from an adjacency matrix
    n_delta_i <- rowSums(adjmat)
    Q <- adjmat * -1
    diag(Q) <- n_delta_i


Using the inverse of Q as the covariance matrix creates a smooth spatial dispersion, that you can see below, when
compared to just simple dispersion at random. A more complete example can be found
[here](https://github.com/nmmarquez/spatial_epi/blob/master/project/project_outlay.r).

![Alt text](/assets/images/spatial_sim_plot.png "If it fits I sits.")
