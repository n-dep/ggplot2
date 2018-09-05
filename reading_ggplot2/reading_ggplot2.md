---
title: "Reading ggplot2" 
subtitle: "Elegant Graphics for Data Analysis"
author: "ND"
date: '2018-09-05'
output: 
  html_document:
    keep_md: true
---



## Getting started with qplot

* Designed to resemble base plot but has some notable differences.
* Not generic. It needs vectors as data input, ideally organised in a dataframe. So `qplot(my_mod)`, where `my_mod` is some fitted model, does not work.
* Aesthetic attributes (color, size, shape, ...) expect, by default, data (e.g. a categorical variable) and therefore adds an informative legend. One needs to use the `I()` function to suppress this behavior. As an example, compare `qplot(carat, price, data = diamonds, col = "red")` and `qplot(carat, price, data = diamonds, col = I("red"))`.
* Multiple 'geoms' can be used in one call.
* You can't use base graphic commands (`lines()`, `points()`, ...) to add elements. You need to add layers according to the ggplot2 syntax.
* Uses non-standard evaluation

> All in all, it seems to be better to avoid `qplot` and use ggplot directly

## Mastering the grammar

A plot according to the layered grammar of ggplot2 is the combination of (*nearly verbatim from the book*):

> * Default **dataset** + set of mappings from variables to aesthetics.
> * One or more **layers**. Each layer is composed of a **geom** (geometric object), a **statistical transformation** (e.g. data -> spline smoothing line) and a **position adjustment** (position in queue of layers). Optionally, a different dataset and/or aesthetic mappings can be provided.
> * One **scale** for each aesthetic mapping. A scale maps data to aesthetic attributes (like color, size, ...) and every attribute requires a scale. The scale operates on all the data in the plot, thus ensuring consistency. A scale is a function and its inverse and parameters. The inverse is used to read values from the graph (axes for position scales or legends for other scales). Note that the inverse is not necessarily unique (i.e. the mapping is not one-to-one). Scaling is done before the statistical transformation.
> * The **coordinate system**, or coord (e.g. Cartesian, polar, ...). The coordinate system affect all posision variables. They differ from scales as they change the appearance of geoms. Coordinate transformations are done after the statistical transformations.
> * The faceting specs. These are used for conditional graphs. 

The way these elements are encoded in R is with a list with components `data`, `mapping` (for the aesthetics), `layers`, `scales`, `coordinates`, `facet`, $\dots$. This implies that the data is effectively stored in the plot object. This also imlies that a plot object can be printed (`print()`), saved (`ggsave` or `save`) and summarized (`summary`).

