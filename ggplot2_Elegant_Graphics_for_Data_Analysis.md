Notes on ggplot2 - Elegant Graphics for Data Analysis
================

Chapter 2: Getting started with ggplot2
=======================================

### geom\_smooth()

-   `method = "loess"`, the default for small n, uses a smooth local regression (as described in `?loess`). The wiggliness of the line is controlled by the `span` parameter, which ranges from 0 (exceedingly wiggly) to 1 (not so wiggly).

-   `method = "gam"` fits a generalised additive model provided by the **mgcv** package. You need to first load mgcv, then use a formula like `formula = y ~ s(x)` or `y ~ s(x, bs = "cs")` (for large data). This is what ggplot2 uses when there are more than 1,000 points.

-   `method = "lm"` fits a linear model, giving the line of best fit.

-   `method = "rlm"` works like `lm()`, but uses a robust fitting algorithm so that outliers don't affect the fit as much. It's part of the **MASS** package.

Chapter 3: Toolbox
==================

### Annotations

`geom_vline()`, `geom_hline()` and `geom_abline()` allow you to add reference lines (sometimes called rules), that span the full range of the plot. Typically, you can either put annotations in the foreground (using `alpha` if needed so you can still see the data), or in the background. With the default background, a thick white line makes a useful reference: it's easy to see but it doesn't jump out at you.

``` r
ggplot(economics, aes(date, unemploy)) + 
  geom_line()
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/umep-1.png" width="100%" style='max-width: 6in' />

We can annotate this plot with which president was in power at the time. There is little new in this code - it's a straightforward manipulation of existing geoms. There is one special thing to note: the use of `-Inf` and `Inf` as positions. These refer to the top and bottom (or left and right) limits of the plot.

``` r
presidential <- subset(presidential, start > economics$date[1])

ggplot(economics) + 
  geom_rect(
    aes(xmin = start, xmax = end, fill = party), 
    ymin = -Inf, ymax = Inf, alpha = 0.2, 
    data = presidential) + 
  geom_vline(
    aes(xintercept = as.numeric(start)), 
    data = presidential,
    colour = "grey50", alpha = 0.5) + 
  geom_text(
    aes(x = start, y = 2500, label = name), 
    data = presidential, 
    size = 3, vjust = 0, hjust = 0, nudge_x = 50) + 
  geom_line(aes(date, unemploy)) + 
  scale_fill_manual(values = c("blue", "red"))
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/unemp-pres-1.png" width="100%" style='max-width: 6in' />

You can use the same technique to add a single annotation to a plot, but it's a bit fiddly because you have to create a one row data frame:

``` r
yrng <- range(economics$unemploy)
xrng <- range(economics$date)
caption <- paste(strwrap("Unemployment rates in the US have 
  varied a lot over the years", 40), collapse = "\n")

ggplot(economics, aes(date, unemploy)) + 
  geom_line() + 
  geom_text(
    aes(x, y, label = caption), 
    data = data.frame(x = xrng[1], y = yrng[2], caption = caption), 
    hjust = 0, vjust = 1, size = 4)
```

Annotations, particularly reference lines, are also useful when comparing groups across facets.

``` r
ggplot(diamonds, aes(log10(carat), log10(price))) + 
  geom_bin2d() + 
  facet_wrap(~cut, nrow = 1)
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/reference_line-1.png" width="100%" style='max-width: 6in' />

``` r

mod_coef <- coef(lm(log10(price) ~ log10(carat), data = diamonds))
ggplot(diamonds, aes(log10(carat), log10(price))) + 
  geom_bin2d() + 
  geom_abline(intercept = mod_coef[1], slope = mod_coef[2], 
    colour = "white", size = 1) + 
  facet_wrap(~cut, nrow = 1)
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/reference_line-2.png" width="100%" style='max-width: 6in' />

### Weighted Data

The choice of a weighting variable profoundly affects what we are looking at in the plot and the conclusions that we will draw. There are two aesthetic attributes that can be used to adjust for weights. Firstly, for simple geoms like lines and points, use the size aesthetic:

``` r
# Unweighted
ggplot(midwest, aes(percwhite, percbelowpoverty)) + 
  geom_point()

# Weight by population
ggplot(midwest, aes(percwhite, percbelowpoverty)) + 
  geom_point(aes(size = poptotal / 1e6)) + 
  scale_size_area("Population\n(millions)", breaks = c(0.5, 1, 2, 4))
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/miss-basic-1.png" width="50%" style='max-width: 3in' /><img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/miss-basic-2.png" width="50%" style='max-width: 3in' />

For more complicated grobs which involve some statistical transformation, we specify weights with the `weight` aesthetic. These weights will be passed on to the statistical summary function. Weights are supported for every case where it makes sense: smoothers, quantile regressions, boxplots, histograms, and density plots. The following code shows how weighting by population density affects the relationship between percent white and percent below the poverty line.

``` r
# Unweighted
ggplot(midwest, aes(percwhite, percbelowpoverty)) + 
  geom_point() + 
  geom_smooth(method = lm, size = 1)

# Weighted by population
ggplot(midwest, aes(percwhite, percbelowpoverty)) + 
  geom_point(aes(size = poptotal / 1e6)) + 
  geom_smooth(aes(weight = poptotal), method = lm, size = 1) +
  scale_size_area(guide = "none")
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/weight-lm-1.png" width="50%" style='max-width: 3in' /><img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/weight-lm-2.png" width="50%" style='max-width: 3in' />

When we weight a histogram or density plot by total population, we change from looking at the distribution of the number of counties, to the distribution of the number of people. The following code shows the difference this makes for a histogram of the percentage below the poverty line:

``` r
ggplot(midwest, aes(percbelowpoverty)) +
  geom_histogram(binwidth = 1) + 
  ylab("Counties")

ggplot(midwest, aes(percbelowpoverty, weight = poptotal)) +
  geom_histogram(binwidth = 1) +
  ylab("Population (1000s)")
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/weight-hist-1.png" width="50%" style='max-width: 3in' /><img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/weight-hist-2.png" width="50%" style='max-width: 3in' />

Chapter 4: Mastering the Grammar
================================

### Scales

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/scale-legends-1.png" alt="Examples of legends from four different scales. From left to right: continuous variable mapped to size, and to colour, discrete variable mapped to shape, and to colour. The ordering of scales seems upside-down, but this matches the labelling of the $y$-axis: small values occur at the bottom." width="100%" style='max-width: 6in' />
<p class="caption">
Examples of legends from four different scales. From left to right: continuous variable mapped to size, and to colour, discrete variable mapped to shape, and to colour. The ordering of scales seems upside-down, but this matches the labelling of the *y*-axis: small values occur at the bottom.
</p>

Chapter 5: Build a plot layer by layer
======================================

### Geoms

Geometric objects, or **geoms** for short, perform the actual rendering of the layer, controlling the type of plot that you create. For example, using a point geom will create a scatterplot, while using a line geom will create a line plot.

-   Graphical primitives:
    -   `geom_blank()`: display nothing. Most useful for adjusting axes limits using data.
    -   `geom_point()`: points.
    -   `geom_path()`: paths.
    -   `geom_ribbon()`: ribbons, a path with vertical thickness.
    -   `geom_segment()`: a line segment, specified by start and end position.
    -   `geom_rect()`: rectangles.
    -   `geom_polyon()`: filled polygons.
    -   `geom_text()`: text.
-   One variable:
    -   Discrete:
        -   `geom_bar()`: display distribution of discrete variable.
    -   Continuous
        -   `geom_histogram()`: bin and count continuous variable, display with bars.
        -   `geom_density()`: smoothed density estimate.
        -   `geom_dotplot()`: stack individual points into a dot plot.
        -   `geom_freqpoly()`: bin and count continuous variable, display with lines.
-   Two variables:
    -   Both continuous:
        -   `geom_point()`: scatterplot.
        -   `geom_quantile()`: smoothed quantile regression.
        -   `geom_rug()`: marginal rug plots.
        -   `geom_smooth()`: smoothed line of best fit.
        -   `geom_text()`: text labels.
    -   Show distribution:
        -   `geom_bin2d()`: bin into rectangles and count.
        -   `geom_density2d()`: smoothed 2d density estimate.
        -   `geom_hex()`: bin into hexagons and count.
    -   At least one discrete:
        -   `geom_count()`: count number of point at distinct locations
        -   `geom_jitter()`: randomly jitter overlapping points.
    -   One continuous, one discrete:
        -   `geom_bar(stat = "identity")`: a bar chart of precomputed summaries.
        -   `geom_boxplot()`: boxplots.
        -   `geom_violin()`: show density of values in each group.
    -   One time, one continuous
        -   `geom_area()`: area plot.
        -   `geom_line()`: line plot.
        -   `geom_step()`: step plot.
    -   Display uncertainty:
        -   `geom_crossbar()`: vertical bar with center.
        -   `geom_errorbar()`: error bars.
        -   `geom_linerange()`: vertical line.
        -   `geom_pointrange()`: vertical line with center.
    -   Spatial
        -   `geom_map()`: fast version of `geom_polygon()` for map data.
-   Three variables:
    -   `geom_contour()`: contours.
    -   `geom_tile()`: tile the plane with rectangles.
    -   `geom_raster()`: fast version of `geom_tile()` for equal sized tiles.

Chapter 6: Scales, axes and legends
===================================

### Guides: legends and axes

You might find it surprising that axes and legends are the same type of thing, but while they look very different there are many natural correspondences between the two, as shown in table and figure below.

<img src="scale-guides.png" width="100%" style='max-width: 6in' />

| Axis              | Legend    | Argument name |
|-------------------|-----------|---------------|
| Label             | Title     | `name`        |
| Ticks & grid line | Key       | `breaks`      |
| Tick label        | Key label | `labels`      |

#### Relabel the axes of categorical scale

If you want to relabel the breaks in a categorical scale, you can use a named labels vector:

``` r
df2 <- data.frame(x = 1:3, y = c("a", "b", "c"))

ggplot(df2, aes(x, y)) + 
  geom_point()

ggplot(df2, aes(x, y)) + 
  geom_point() + 
  scale_y_discrete(labels = c(a = "apple", b = "banana", c = "carrot"))
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/unnamed-chunk-2-1.png" width="50%" style='max-width: 3in' /><img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/unnamed-chunk-2-2.png" width="50%" style='max-width: 3in' />

#### Scales package

The scales package provides a number of useful labelling functions. See the documentation of the scales package for more details.

``` r
df <- data.frame(x = c(1, 3, 5) * 1000, y = 1)
axs <- ggplot(df, aes(x, y)) + 
  geom_point() + 
  labs(x = NULL, y = NULL)
leg <- ggplot(df, aes(y, x, fill = x)) + 
  geom_tile() + 
  labs(x = NULL, y = NULL)

axs + scale_y_continuous(labels = scales::percent_format())
axs + scale_y_continuous(labels = scales::dollar_format("$"))
leg + scale_fill_continuous(labels = scales::unit_format("k", 1e-3))
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/breaks-functions-1.png" width="33.3%" style='max-width: 2in' /><img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/breaks-functions-2.png" width="33.3%" style='max-width: 2in' /><img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/breaks-functions-3.png" width="33.3%" style='max-width: 2in' />

### Limits

Because modifying the limits is such a common task, ggplot2 provides some helper to make this even easier: `xlim()`, `ylim()` and `lims()` These functions inspect their input and then create the appropriate scale, as follows:

-   `xlim(10, 20)`: a continuous scale from 10 to 20
-   `ylim(20, 10)`: a reversed continuous scale from 20 to 10
-   `xlim("a", "b", "c")`: a discrete scale
-   `xlim(as.Date(c("2008-05-01", "2008-08-01")))`: a date scale from May 1 to August 1 2008.

``` r
df <- data.frame(x = 1:3, y = 1:3, z = c("a", "b", "c"))
base <- ggplot(df, aes(x, y)) + 
  geom_point(aes(colour = z), size = 3) + 
  xlab(NULL) + 
  ylab(NULL)

base + xlim(0, 4)
base + xlim(4, 0)
base + lims(x = c(0, 4))
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/unnamed-chunk-3-1.png" width="33.3%" style='max-width: 2in' /><img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/unnamed-chunk-3-2.png" width="33.3%" style='max-width: 2in' /><img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/unnamed-chunk-3-3.png" width="33.3%" style='max-width: 2in' />

Chapter 8: Themes
=================

### Examples

``` r
base <- ggplot(mpg, aes(cty, hwy, color = factor(cyl))) +
  geom_jitter() + 
  geom_abline(colour = "grey50", size = 2)
base
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/motivation-1-1.png" width="75%" style='max-width: 4.5in' style="display: block; margin: auto;" />

``` r
labelled <- base +
  labs(
    x = "City mileage/gallon",
    y = "Highway mileage/gallon",
    colour = "Cylinders",
    title = "Highway and city mileage are highly correlated"
  ) +
  scale_colour_brewer(type = "seq", palette = "Spectral")
labelled
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/motivation-2-1.png" width="75%" style='max-width: 4.5in' style="display: block; margin: auto;" />

``` r
styled <- labelled +
  theme_bw() + 
  theme(
    plot.title = element_text(face = "bold", size = 12),
    legend.background = element_rect(fill = "white", size = 4, colour = "white"),
    legend.justification = c(0, 1),
    legend.position = c(0, 1),
    axis.ticks = element_line(colour = "grey70", size = 0.2),
    panel.grid.major = element_line(colour = "grey70", size = 0.2),
    panel.grid.minor = element_blank()
  )
styled
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/motivation-3-1.png" width="75%" style='max-width: 4.5in' style="display: block; margin: auto;" />

### Theme elements

#### Plot elements

| Element         | Setter           | Description         |
|-----------------|------------------|---------------------|
| plot.background | `element_rect()` | plot background     |
| plot.title      | `element_text()` | plot title          |
| plot.margin     | `margin()`       | margins around plot |

#### Axis elements

| Element           | Setter           | Description                                      |
|-------------------|------------------|--------------------------------------------------|
| axis.line         | `element_line()` | line parallel to axis (hidden in default themes) |
| axis.text         | `element_text()` | tick labels                                      |
| axis.text.x       | `element_text()` | x-axis tick labels                               |
| axis.text.y       | `element_text()` | y-axis tick labels                               |
| axis.title        | `element_text()` | axis titles                                      |
| axis.title.x      | `element_text()` | x-axis title                                     |
| axis.title.y      | `element_text()` | y-axis title                                     |
| axis.ticks        | `element_line()` | axis tick marks                                  |
| axis.ticks.length | `unit()`         | length of tick marks                             |

#### Panel elements

| Element            | Setter           | Description                   |
|--------------------|------------------|-------------------------------|
| panel.background   | `element_rect()` | panel background (under data) |
| panel.border       | `element_rect()` | panel border (over data)      |
| panel.grid.major   | `element_line()` | major grid lines              |
| panel.grid.major.x | `element_line()` | vertical major grid lines     |
| panel.grid.major.y | `element_line()` | horizontal major grid lines   |
| panel.grid.minor   | `element_line()` | minor grid lines              |
| panel.grid.minor.x | `element_line()` | vertical minor grid lines     |
| panel.grid.minor.y | `element_line()` | horizontal minor grid lines   |
| aspect.ratio       | numeric          | plot aspect ratio             |

#### Legend elements

| Element            | Setter           | Description                                  |
|--------------------|------------------|----------------------------------------------|
| legend.background  | `element_rect()` | legend background                            |
| legend.key         | `element_rect()` | background of legend keys                    |
| legend.key.size    | `unit()`         | legend key size                              |
| legend.key.height  | `unit()`         | legend key height                            |
| legend.key.width   | `unit()`         | legend key width                             |
| legend.margin      | `unit()`         | legend margin                                |
| legend.text        | `element_text()` | legend labels                                |
| legend.text.align  | 0--1             | legend label alignment (0 = right, 1 = left) |
| legend.title       | `element_text()` | legend name                                  |
| legend.title.align | 0--1             | legend name alignment (0 = right, 1 = left)  |

#### Facetting elements

| Element          | Setter           | Description                        |
|------------------|------------------|------------------------------------|
| strip.background | `element_rect()` | background of panel strips         |
| strip.text       | `element_text()` | strip text                         |
| strip.text.x     | `element_text()` | horizontal strip text              |
| strip.text.y     | `element_text()` | vertical strip text                |
| panel.margin     | `unit()`         | margin between facets              |
| panel.margin.x   | `unit()`         | margin between facets (vertical)   |
| panel.margin.y   | `unit()`         | margin between facets (horizontal) |

Chapter 9: Data Analysis
========================

``` r
ec2
#> # A tibble: 12 x 11
#>   month `2006` `2007` `2008` `2009` `2010` `2011` `2012` `2013` `2014`
#>   <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
#> 1     1    8.6    8.3    9     10.7   20     21.6   21     16.2   15.9
#> 2     2    9.1    8.5    8.7   11.7   19.9   21.1   19.8   17.5   16.2
#> 3     3    8.7    9.1    8.7   12.3   20.4   21.5   19.2   17.7   15.9
#> 4     4    8.4    8.6    9.4   13.1   22.1   20.9   19.1   17.1   15.6
#> 5     5    8.5    8.2    7.9   14.2   22.3   21.6   19.9   17     14.5
#> 6     6    7.3    7.7    9     17.2   25.2   22.3   20.1   16.6   13.2
#> # ... with 6 more rows, and 1 more variable: `2015` <dbl>
```

### Gather

`gather()` has four main arguments:

-   `data`: the dataset to translate.

-   `key` & `value`: the key is the name of the variable that will be created from the column names, and the value is the name of the variable that will be created from the cell values.

-   `...`: which variables to gather. You can specify individually, `A, B, C, D`, or as a range `A:D`. Alternatively, you can specify which columns are *not* to be gathered with `-`: `-E, -F`.

``` r
gather(ec2, key = year, value = unemp, `2006`:`2015`)
#> # A tibble: 120 x 3
#>   month year  unemp
#>   <dbl> <chr> <dbl>
#> 1     1 2006    8.6
#> 2     2 2006    9.1
#> 3     3 2006    8.7
#> 4     4 2006    8.4
#> 5     5 2006    8.5
#> 6     6 2006    7.3
#> # ... with 114 more rows
```

Alternatively, we could gather all columns except `month`:

``` r
gather(ec2, key = year, value = unemp, -month)
```

To be most useful, we can provide two extra arguments:

``` r
economics_2 <- gather(ec2, year, rate, `2006`:`2015`, 
  convert = TRUE, na.rm = TRUE)
economics_2
#> # A tibble: 112 x 3
#>   month  year  rate
#> * <dbl> <int> <dbl>
#> 1     1  2006   8.6
#> 2     2  2006   9.1
#> 3     3  2006   8.7
#> 4     4  2006   8.4
#> 5     5  2006   8.5
#> 6     6  2006   7.3
#> # ... with 106 more rows
```

We use `convert = TRUE` to automatically convert the years from character strings to numbers, and `na.rm = TRUE` to remove the months with no data.

When the data is in this form, it's easy to visualise in many different ways. For example, we can choose to emphasise either long term trend or seasonal variations:

``` r
ggplot(economics_2, aes(year + (month - 1) / 12, rate)) +
  geom_line()

ggplot(economics_2, aes(month, rate, group = year)) +
  geom_line(aes(colour = year), size = 1)
```

<img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/ec2-plots-1.png" width="50%" style='max-width: 3in' /><img src="ggplot2_Elegant_Graphics_for_Data_Analysis_files/figure-markdown_github/ec2-plots-2.png" width="50%" style='max-width: 3in' />

### Spread

`spread()` is the opposite of `gather()`. For example, the following example dataset contains three variables (`day`, `rain` and `temp`), but `rain` and `temp` are stored in indexed form.

``` r
weather <- dplyr::data_frame(
  day = rep(1:3, 2),
  obs = rep(c("temp", "rain"), each = 3),
  val = c(c(23, 22, 20), c(0, 0, 5)))
weather
#> # A tibble: 6 x 3
#>     day obs     val
#>   <int> <chr> <dbl>
#> 1     1 temp     23
#> 2     2 temp     22
#> 3     3 temp     20
#> 4     1 rain      0
#> 5     2 rain      0
#> 6     3 rain      5
```

You'll need to supply the `data` to translate, as well as the name of the `key` column which gives the variable names, and the `value` column which contains the cell values. Here the key is `obs` and the value is `val`:

``` r
spread(weather, key = obs, value = val)
#> # A tibble: 3 x 3
#>     day  rain  temp
#>   <int> <dbl> <dbl>
#> 1     1     0    23
#> 2     2     0    22
#> 3     3     5    20
```
