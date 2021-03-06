---
title: "Reading ggplot2" 
subtitle: "Elegant Graphics for Data Analysis"
author: "ND"
date: '2018-09-07'
output: 
  html_document:
    keep_md: true
---



## Ch 2: Getting started with qplot

* Designed to resemble base plot but has some notable differences.
* Not generic. It needs vectors as data input, ideally organised in a dataframe. So `qplot(my_mod)`, where `my_mod` is some fitted model, does not work.
* Aesthetic attributes (color, size, shape, ...) expect, by default, data (e.g. a categorical variable) and therefore adds an informative legend. One needs to use the `I()` function to suppress this behavior. As an example, compare `qplot(carat, price, data = diamonds, col = "red")` and `qplot(carat, price, data = diamonds, col = I("red"))`.
* Multiple 'geoms' can be used in one call.
* You can't use base graphic commands (`lines()`, `points()`, ...) to add elements. You need to add layers according to the ggplot2 syntax.
* Uses non-standard evaluation

**All in all, it seems to be better to avoid `qplot` and use ggplot directly.**

## Ch 3: Mastering the grammar

A plot according to the layered grammar of ggplot2 is the combination of (*nearly verbatim from the book*):

* Default **dataset** + set of mappings from variables to aesthetics.
* One or more **layers**. Each layer is composed of a **geom** (geometric object), a **statistical transformation** (stat for short, e.g. data -> spline smoothing line) and a **position adjustment** (adjustments to avoid overplotting). Optionally, a different dataset and/or aesthetic mappings can be provided. So, in effect, each different layer can have a different dataset and a different aesthetic mapping.
* One **scale** for each aesthetic mapping. A scale maps data to aesthetic attributes (like color, size, ...) and every attribute requires a scale. The scale operates on all the data in the plot, thus ensuring consistency. A scale is a function and its inverse and parameters. The inverse is used to read values from the graph (axes for position scales or legends for other scales). Note that the inverse is not necessarily unique (i.e. the mapping is not one-to-one). Scaling is done before the statistical transformation.
* The **coordinate system**, or coord (e.g. Cartesian, polar, ...). The coordinate system affect all posision variables. They differ from scales as they change the appearance of geoms. Coordinate transformations are done after the statistical transformations.
> * The faceting specs. These are used for conditional graphs. 

The way these elements are encoded in R is with a list with components `data`, `mapping` (for the aesthetics), `layers`, `scales`, `coordinates`, `facet`, ... This implies that the data is effectively stored in the plot object (as a copy, not a reference!). This also implies that a plot object can be printed (`print()`), saved (`ggsave` or `save`) and summarized (`summary`).

## Ch 4: Build a plot layer by layer

* A plot is initialised with the `ggplot` function. This function has two arguments, `data` (data.frame) and `mapping` (list of aesthetic mappings, created with the `aes` function). These two arguments set up the defaults for the entire plot. If they are not provided, each additional layer needs to have these arguments. 
* The `data` must be a dataframe!
* Layers are then added with `+ layer()`. The `layer` function requires quite a lot of parameters. As each geom has a default statistic and position, and every statistic has a default geom, shortcuts are available as `geom_xxx(mapping, data, ...)` or `stat_xxx(mapping, data, ...)`. Note that the default stat or geom can be overruled. 
* Note that the order of the two main arguments of `ggplot`and `geom_xxx` (or `stat_xxx`) is reversed.
* The data of an existing plot can be changed with `p %+% new_data`.

* In the `aes`function, never refer to data outside the data frame from the data argument (e.g. `diamonds$carat`). Functions of the variables can be used (e.g. log).
* Aesthetic mappings specified in a layer only pertain to the aesthetics of that layer (so not the entire plot).
* There is a difference between **mapping** and **setting** aesthetics. Mapping refers to mapping an aesthetic to a variable. Setting refers to setting an aesthetic equal to a constant (e.g. `color = "red"`). Mapping is done in the `aes` function whereas setting is done as a parameter in a layer (so outside of `aes()`).
* An important aesthetic is the **group**. Some geoms operate on multiple rows simultaneously (e.g. line, boxplot, ...). If this geom should be done separately for each group, you need to specify the group aesthetic. A good example is for longitudinal data were one draws a line, per subject. One can use `interaction()` to specify a grouping based on multiple variables. The default of group is the interaction of all discrete variables in the plot. Use `group = 1` to 'ungroup'.
* For these collective geoms the individual aesthetics can't be used all together. For lines (defined by 2 points) the aesthetics of the first point are used. For other collective geoms the default aesthetics are used unless all components share the same aesthetics.

* A stat computes, from the input data, a different dataset. This produced dataset can be used for plotting. The derived dataset can be used for plotting and variables must be between two sets of `..` to distinguish them from the original variables (with possibly the same name). Example: `ggplot(diamonds, aes(carat)) + geom_histogram(aes = ..density..)` where `..density`` is computed by the histogram geom.

## Ch 6: Scales, axes and legends

Without a scale, there can't be a transformation from data to aesthetics. Scales also produce a guide (axes and/or legends) to allow for inverse mapping.
Scales have a domain, continuous (interval on the number line) or discrete (factor, character, logical). Scales have a range, which can also be divided in discrete and continuous. The process in going from the domain (obtained from the data) to the range (the aesthetics) has 3 stages:

1. Transformation:
  * Continuous domain.
  * Example: log transformation.
  * Note that statistical summaries in layers are computed on the transformed data.
2. Training:
  * The domain is learned. 
  * This can be complex if there are multiple layers, accross multiple datasets in multiple panels.
  * Can be overruled by manually setting limits in the scale layer. Values outside the domain are then mapped to `NA`.
3. Mapping: Effectly apply the mapping from data to aesthetic values.

Scales have the following naming scheme: `scale_<aesthetic_name>_<scale_name>`, e.g. `scale_color_continuous`.

Common arguments to all scales are:

* name: Labels for axes and legend. First argument of the scale layer. Can be a mathematical expression. Shortcuts: `xlab`, `ylab`, `labs`.
* limits: Sets domain of scale. For continuous scales a length 2 vector. For discrete scales a character vector. Data outside the domain are not plotted.
* breaks and labels: Determine the segmentation of an axis or legend and the corresponding labels.

Scales come, roughly, in 4 categories: 

1. Position: Map data to plot region and construct axes.
  * 2 scales for each plot, horizontal and vertical.
  
    * limits: These are **not zooms** as in base plots. Data outside the limits are not plotted and are **not included in the statistical transformations**. Use `coord_cartesian()` for zooming.
    * trans: Transform scale. This is equivalent to plotting the transformed input directly but the breaks and labels will be different. With transformed scales the labelling will be in the original data space. 
2. Color: Map variables to colors.
  * Continuous: 3 types of color gradients, `scale_color(or fill)_gradient`, `scale_color(or fill)_gradient2` and `scale_color(or fill)_gradientn`.
  * Discrete: `scale_color(or fill)_hue` (default, picks evenly spaced on color wheel) or `scale_color(or fill)_brewer`. 
3. Manual: Map discrete variables to symbol size, line type, shape, ..., and create the legend.
  * Only discrete: `scale_shape_manual`, `scale_linetype_manual` and `scale_color_manual`.
4. Identity: Map data directly to aesthetics (e.g. your data already has a size variable).

# Ch 7: Positioning

## Faceting

There are 2 types: `facet_grid` and `facet_wrap`.

1. `facet_grid`:
  + Lays plots out in 2D grid.
  + Layout syntax: `rows ~ cols`. Use `.` for the default 1, e.g. `. ~ x` has a one row layout with multiple columns as specified by `x`. You can use `x + y` syntax to facet over multiple variables at once.
  + The `margins` argument allows the addition of marginal plots.
  + All panels in a single column need to have the same x scale and equivalently for rows and y scales.
  + `space`: if `"free"` the columns (rows) will have proportional widths (heights).
      
2. `facet_wrap`:
  + Creates a list of plots, which is subsequently wrapped into 2D.
  + Will try to layout as a square, with a bias to more plots in the vertical direction.
  
The `scales` parameter determines if the scales are fixed for all panels or are free. Note also that faceting only works with variables in the data, not transformations.
    
## Coordinate systems:

* There are 3 types of coordinate systems: the Cartesian (with the following flavors: `cartesian`, `equal`, `flip` and `trans`), map and polar. 
* All can be called by `coord_xxx`. 
* Transformations of the data can (will) change the geom's appearances as geoms are first represented as a combination of points, lines and polygons, which are then subsequently transformed to the new coordinate system. 
* Statistics are calculated based on the Cartesian coordinates. 
* There are arguments `xlim` and `ylim`. These arguments change the limits of the plot but do not alter the data that is displayed or used to calculate statistics. E.g. a smoother will be computed based on all the data, even when zoomed in on a particular region of the dataspace. Using limits in the scales will discard data such that the derived smoother is calculated based on the reduced dataset. 
* Transformations work similarly as the limits. A common usage is to transform in a scale layer, compute statistics, backtransform with a coordinate layer for interpretation. 

# Ch 8: Polishing your plots for publication

* Themes: these control plot aspects that are not data related. Examples are titles, fonts, labels, ... 

  + Themes can be set globally (`theme_set`, the theme affects the plot when it is 
  drawn, not when it is created) or by a layer (`+ theme`). 
  + Themes consist of elements (axis, legend, panel, plot, strip): `element.xxx`.
  + These elements can be set by the elementary types (blank, line, rect, text): `element_typexxx`.
  + If elements are left out (`element_blank`), 'real estate' might be reclaimed by other elements. To stop this use `NA` as an option.
  + Use `theme_update` to change multiple settings at once. 
  + The book is seriously outdated with respect to the current (and future) ggplot2 implementation ([Link](https://github.com/wch/ggplot2/wiki/New-theme-system))

* Customising: note that `set_default_scale` has been deprecated. 

* Saving: `ggsave` saves 1 plot. Multiple plots to 1 file need to be saved the conventional R way (`pdf();` `print(plot);` `dev.off()`)

# Ch 10: Reducing duplication

1. Iteration: The last lot is automatically copied every time a new plot is plotted or modified. This plot can be called and then modified by `last_plot()`. This should obviously only be used in interactive mode. 
2. Plot templates: Every component of a ggplot plot is its own object. These objects can be stored and added to different plots. This can help if specific elements are reused often. This can be extended to using vectors of components.
3. Plot functions: 
  
  
  
   
