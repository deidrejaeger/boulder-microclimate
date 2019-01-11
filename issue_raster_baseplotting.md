test\_data\_issue\_plotting
================
Deidre Jaeger
11/7/2018

I think I am having some package dependency issues but struggling to know how to check which packages changed - or to know if I need to go back to older versions of packages. I'm working with spatial data in ggplot, and I was working from this tutorial: <https://eriqande.github.io/rep-res-eeb-2017/plotting-spatial-data-with-ggplot.html>

I installed devtools::install\_github("paleolimbot/ggspatial") from <https://paleolimbot.github.io/ggspatial/>

While the installation was running, I noticed a lot of things were updated in the R console messaging, such as packages "raster" "sp" "ggplot2". Then plotting with these packages, as well as with base graphics packages no longer worked the same way.

I am just trying to get my original code to work to produce the image below:

![](issue_raster_baseplotting_files/figure-markdown_github/unnamed-chunk-1-1.png) I was originally working with base graphics for visualization, but was attempting to figure out ggplot to display this data.

I will detail the workflow below just focusing on one of the panels:

``` r
library(rgdal) # needed to read in boulder shapefile
```

    ## Loading required package: sp

    ## rgdal: version: 1.3-6, (SVN revision 773)
    ##  Geospatial Data Abstraction Library extensions to R successfully loaded
    ##  Loaded GDAL runtime: GDAL 2.1.3, released 2017/20/01
    ##  Path to GDAL shared files: /Library/Frameworks/R.framework/Versions/3.5/Resources/library/rgdal/gdal
    ##  GDAL binary built with GEOS: FALSE 
    ##  Loaded PROJ.4 runtime: Rel. 4.9.3, 15 August 2016, [PJ_VERSION: 493]
    ##  Path to PROJ.4 shared files: /Library/Frameworks/R.framework/Versions/3.5/Resources/library/rgdal/proj
    ##  Linking to sp version: 1.3-1

``` r
library(raster) # needed for cdf conversion
library(ggplot2)

# libraries for images
library(png)
library(grid)

sessionInfo()
```

    ## R version 3.5.1 (2018-07-02)
    ## Platform: x86_64-apple-darwin15.6.0 (64-bit)
    ## Running under: macOS Sierra 10.12.6
    ## 
    ## Matrix products: default
    ## BLAS: /Library/Frameworks/R.framework/Versions/3.5/Resources/lib/libRblas.0.dylib
    ## LAPACK: /Library/Frameworks/R.framework/Versions/3.5/Resources/lib/libRlapack.dylib
    ## 
    ## locale:
    ## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
    ## 
    ## attached base packages:
    ## [1] grid      stats     graphics  grDevices utils     datasets  methods  
    ## [8] base     
    ## 
    ## other attached packages:
    ## [1] ggplot2_3.1.0 raster_2.8-4  rgdal_1.3-6   sp_1.3-1      png_0.1-7    
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_0.12.19     compiler_3.5.1   pillar_1.3.0     plyr_1.8.4      
    ##  [5] bindr_0.1.1      tools_3.5.1      digest_0.6.18    evaluate_0.11   
    ##  [9] tibble_1.4.2     gtable_0.2.0     lattice_0.20-35  pkgconfig_2.0.2 
    ## [13] rlang_0.3.0.1    yaml_2.2.0       bindrcpp_0.2.2   withr_2.1.2     
    ## [17] stringr_1.3.1    dplyr_0.7.7      knitr_1.20       rprojroot_1.3-2 
    ## [21] tidyselect_0.2.5 glue_1.3.0       R6_2.3.0         rmarkdown_1.10  
    ## [25] purrr_0.2.5      magrittr_1.5     backports_1.1.2  scales_1.0.0    
    ## [29] codetools_0.2-15 htmltools_0.3.6  assertthat_0.2.0 colorspace_1.3-2
    ## [33] stringi_1.2.4    lazyeval_0.2.1   munsell_0.5.0    crayon_1.3.4

``` r
boulder_outline <- rgdal::readOGR("data/BoulderCityLimits/BoulderCityLimits.shp")
```

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "/Users/deidrejaeger/Documents/Career/CU-Boulder/Research/BoulderMicroclimate/boulder-microclimate-analysis/data/BoulderCityLimits/BoulderCityLimits.shp", layer: "BoulderCityLimits"
    ## with 5 features
    ## It has 2 fields

``` r
plot(boulder_outline)
```

![](issue_raster_baseplotting_files/figure-markdown_github/unnamed-chunk-3-1.png)

``` r
extent(boulder_outline) # check extent
```

    ## class       : Extent 
    ## xmin        : 3055614 
    ## xmax        : 3090086 
    ## ymin        : 1227233 
    ## ymax        : 1277405

``` r
crs(boulder_outline) # check coordinate reference system
```

    ## CRS arguments:
    ##  +proj=lcc +lat_1=39.71666666666667 +lat_2=40.78333333333333
    ## +lat_0=39.33333333333334 +lon_0=-105.5 +x_0=914401.8288999998
    ## +y_0=304800.6096 +ellps=GRS80 +units=us-ft +no_defs

``` r
# load raster from daymet, tile 11738 minimum temperature for 2017
tile_tmin17_st <- raster::stack("data/daymet/V3/CF_tarred/tars_2017/11738_2017/tmin.nc")
```

    ## Loading required namespace: ncdf4

    ## Warning in cbind(m[i, ], vals): number of rows of result is not a multiple
    ## of vector length (arg 2)

``` r
# convert raster stack to brick
tile_tmin17_br <- raster::brick(tile_tmin17_st)

extent(tile_tmin17_br) # tile is much larger than the City of Boulder
```

    ## class       : Extent 
    ## xmin        : -488750 
    ## xmax        : -315750 
    ## ymin        : -257500 
    ## ymax        : -35500

``` r
crs(tile_tmin17_br) # both lambert conical cooordinate system but slightly different projection dataum than City of Boulder shapefile
```

    ## CRS arguments:
    ##  +proj=lcc +lon_0=-100 +lat_0=42.5 +x_0=0 +y_0=0 +lat_1=25
    ## +ellps=WGS84 +lat_2=45

``` r
# Reproject the city polygons onto the raster brick's coord ref system
boulder_outline_reproj <- spTransform(boulder_outline,
                                crs(tile_tmin17_br))


extent(boulder_outline_reproj) 
```

    ## class       : Extent 
    ## xmin        : -447153.3 
    ## xmax        : -436416.5 
    ## ymin        : -267961 
    ## ymax        : -253158.7

``` r
crs(boulder_outline_reproj)
```

    ## CRS arguments:
    ##  +proj=lcc +lon_0=-100 +lat_0=42.5 +x_0=0 +y_0=0 +lat_1=25
    ## +ellps=WGS84 +lat_2=45

Before I installed the ggspatial package devtools::install\_github("paleolimbot/ggspatial") I was able to plot the temperature raster within the extent of the City of Boulder.

``` r
plot(tile_tmin17_br[[305]], ext=extent(boulder_outline_reproj), 
                                     main = "Daymet maximum temperature (C) in Boulder, CO \n November 1, 2016")
plot(boulder_outline_reproj, add= T)
```

![](issue_raster_baseplotting_files/figure-markdown_github/unnamed-chunk-5-1.png)

After installing devtools::install\_github("paleolimbot/ggspatial") I noticed a lot of things were updated in the R console messaging, such as packages "raster" "sp" "ggplot2".

### BASE GRAPHICS

When I tried to use the base graphics plot, nothing shows up anymore

``` r
plot(tile_tmin17_br[[305]], ext=extent(boulder_outline_reproj), 
                                     main = "Daymet maximum temperature (C) in Boulder, CO \n November 1, 2016")
plot(boulder_outline_reproj, add= T)
```

![](issue_raster_baseplotting_files/figure-markdown_github/unnamed-chunk-6-1.png)

If I just specify base graphics to view the full tile I get an error message

``` r
graphics::plot(tile_tmin17_br[[32]])
```

> graphics::plot(tile\_tmin17\_br\[\[32\]\]) Error in as.double(y) : cannot coerce type 'S4' to vector of type 'double'

If I specify plotting from the raster package I notice that the resulting image looks different: it is tilted, with more jagged pixel edges than it had before.

``` r
raster::plot(tile_tmin17_br[[32]])
```

![](issue_raster_baseplotting_files/figure-markdown_github/unnamed-chunk-8-1.png)

If I add the city of boulder, I see that the outline no longer overlaps any part of the tile.

``` r
raster::plot(tile_tmin17_br[[32]])
plot(boulder_outline_reproj, add = T)
```

![](issue_raster_baseplotting_files/figure-markdown_github/unnamed-chunk-9-1.png)

### GGPLOT

When I tried to use ggplot2 to plot tile\_tmin17\_br\[\[305\]\], I got the following error message: Error in get(Info\[i, 1\], envir = env) : lazy-load database '/Library/Frameworks/R.framework/Versions/3.5/Resources/library/ggplot2/R/ggplot2.rdb' is corrupt In addition: Warning message: In get(Info\[i, 1\], envir = env) : internal error -3 in R\_decompress1

I then uninstalled ggplot, and reinstalled it and the problem went away. I tried to remove.packages(ggspatial) but it still seems to be an available package. In the meantime, I am working to try and use the ggspatial package

``` r
library(ggspatial)

ggplot() +
  layer_spatial(tile_tmin17_br[[32]])
```

![](issue_raster_baseplotting_files/figure-markdown_github/unnamed-chunk-10-1.png)

``` r
ggplot() +
    layer_spatial(boulder_outline) 
```

> ggplot() + + layer\_spatial(boulder\_outline) Error in grid.Call(C\_textBounds, as.graphicsAnnot(x*l**a**b**e**l*),*x*x, x$y, : polygon edge not found

``` r
ggplot() +
  layer_spatial(boulder_outline) +
  layer_spatial(tile_tmin17_br[[32]]) 
```

![](issue_raster_baseplotting_files/figure-markdown_github/unnamed-chunk-12-1.png)
