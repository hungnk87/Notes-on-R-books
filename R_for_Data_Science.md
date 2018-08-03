R for Data Science
================

Chapter 1: Data visualization with ggplot2
==========================================

### Organization of a typical data science project

<img src="organization.png" width="500px" style="display: block; margin: auto;" />

### facet

``` r
ggplot(data = mpg) +
      geom_point(mapping = aes(x = displ, y = hwy)) +
      facet_wrap(~ class, nrow = 2)

ggplot(data = mpg) +
      geom_point(mapping = aes(x = displ, y = hwy)) +
      facet_grid(drv ~ cyl)
```

### show.legend

``` r
ggplot(data = mpg) +
      geom_smooth(
        mapping = aes(x = displ, y = hwy, color = drv),
        show.legend = FALSE )
```

### Overides the global data

``` r
 ggplot(data = mpg, mapping = aes(x = displ, y = hwy)) +
      geom_point(mapping = aes(color = class)) +
      geom_smooth(
        data = filter(mpg, class == "subcompact"),
        se = FALSE )
```

    ## `geom_smooth()` using method = 'loess'

<img src="R_for_Data_Science_files/figure-markdown_github/overide-1.png" style="display: block; margin: auto;" />

### geom\_smooth

``` r
ggplot(data = mpg, mapping = aes(x = displ, y = hwy)) +
      geom_point(mapping = aes(color = class)) +
      geom_smooth(
        data = filter(mpg, class == "subcompact"),
        method = "loess", se = FALSE)
```

### geom\_bar

``` r
p1 <- ggplot(data = diamonds) +
      geom_bar(mapping = aes(x = cut))
p2 <- ggplot(data = diamonds) +
        geom_bar(mapping = aes(x = cut, y = ..prop.., group = 1))
grid.arrange(p1,p2,nrow=1)
```

<img src="R_for_Data_Science_files/figure-markdown_github/geom_bar1-1.png" style="display: block; margin: auto;" />

### position

-   `position = "identity"` will place each object exactly where it falls in the context of the graph
-   `position = "fill"` works like stacking, but makes each set of stacked bars the same height
-   `position = "dodge"` places overlapping objects directly beside one another
-   `position = "jitter"` adds a small amount of random noise to each point.

### Cordinate Systems

-   `coord_flip()` switches the x- and y-axes.
-   `coord_quickmap()` sets the aspect ratio correctly for maps

``` r
nz <- map_data("nz")

p1 <- ggplot(nz, aes(long, lat, group = group)) +
        geom_polygon(fill = "white", color = "black")

p2 <- ggplot(nz, aes(long, lat, group = group)) +
        geom_polygon(fill = "white", color = "black") +
        coord_quickmap()

grid.arrange(p1,p2,nrow=1)
```

<img src="R_for_Data_Science_files/figure-markdown_github/map-1.png" style="display: block; margin: auto;" />

### Shortcut

In Console, Cmd + Up Arrow to list all the commands typed that start with those letter

### near

``` r
1/49*49==1 ## FALSE
near(1/49*49,1) ## TRUE
```
