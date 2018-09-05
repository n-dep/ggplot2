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
