[![Build Status](https://travis-ci.org/andrewzm/FRK.svg)](https://travis-ci.org/andrewzm/FRK)
[![codecov.io](http://codecov.io/github/andrewzm/FRK/coverage.svg?branch=master)](http://codecov.io/github/andrewzm/FRK?branch=master)

Fixed Rank Kriging 
================

<img src="./man/figures/FRK_logo.svg" width=100>



Installation 
------------

The package `FRK` is now at v1.0.0 and available on CRAN! To install, please type

```r
install.packages("FRK")
```

To install the most recent (development) version, first please install `INLA` from `http://www.r-inla.org/download`, then please load `devtools` and type

```r
install_github("andrewzm/FRK", dependencies = TRUE, build_vignettes = TRUE)
```

A document containing a description, details on the underlying maths and computations, as well as several examples, is available as a vignette. To load this vignette please type

```r
library("FRK")
vignette("FRK_intro")
```

A draft paper with more details, currently in press, is available from [here](https://arxiv.org/abs/1705.08105).

Description
------------

Package: FRK

Type: Package

Title: Fixed Rank Kriging

Version: 1.0.0

Date: 2020-03-25

Author: Andrew Zammit-Mangion, Matthew Sainsbury-Dale

Maintainer: Andrew Zammit-Mangion <andrewzm@gmail.com>

Description: Fixed Rank Kriging is a tool for spatial/spatio-temporal modelling and prediction with large datasets. The approach, discussed in Cressie and Johannesson (2008), decomposes the field, and hence the covariance function, using a fixed set of *n* basis functions, where *n* is typically much smaller than the number of data points (or polygons) *m*. The method naturally allows for non-stationary, anisotropic covariance functions and the use of observations with varying support (with known error variance). The projected field is a key building block of the Spatial Random Effects (SRE) model, on which this package is based. The package FRK provides helper functions to model, fit, and predict using an SRE with relative ease. 
Reference: Cressie, N., & Johannesson, G. (2008). Fixed rank kriging for very large spatial data sets. Journal of the Royal Statistical Society: Series B, 70, 209-226.

License: GPL (>= 2)


Quick start
------------

```r
library("FRK")
library("sp")
library("ggplot2")
library("ggpubr")

## Setup
m <- 1000                                                 # Sample size
RNGversion("3.6.0"); set.seed(1)                          # Fix seed
zdf <- Z <- data.frame(x = runif(m), y= runif(m))         # Generate random locs
zdf$z <- Z$z <- sin(8*Z$x) + cos(8*Z$y) + 0.5*rnorm(m)    # Simulate Gaussian data
coordinates(Z) = ~x+y                                     # Turn into sp object

## Run FRK
S <- FRK(f = z ~ 1,                           # Formula to FRK
         list(Z),                             # All datasets are supplied in list
         n_EM = 10)                           # Max number of EM iterations
Pred <- predict(S)                            # Prediction stage

xy <- data.frame(coordinates(Pred))           # Extract info from predictions
xy$mu <- Pred$mu
xy$se <- Pred$sd

## Plotting
data_plot <- ggplot(zdf) + geom_point(aes(x,y,colour=z)) + 
  scale_colour_distiller(palette="Spectral") + theme_bw() + coord_fixed()
pred_plot <- ggplot(xy) + geom_raster(aes(x,y,fill=mu)) + 
  scale_fill_distiller(palette="Spectral") + theme_bw() + coord_fixed()
se_plot <- ggplot(xy) + geom_tile(aes(x,y,fill=se)) + 
  geom_point(data=zdf,aes(x,y),pch=46) +
  scale_fill_distiller(palette = "BrBG", n.breaks = 3) + theme_bw() + coord_fixed() 
ggarrange(data_plot, pred_plot, se_plot, nrow = 1, align = "hv", legend = "top")  

```

<!---
ggsave( 
  filename = "Gaussian_data.png", device = "png", 
  width = 10, height = 4,
  path = "~/Desktop/"
)
--->

![(Left) Gaussian data. (Centre) Predictions. (Right) Standard errors.](/man/figures/Gaussian_data?raw=true)


```r
# Simulate Poisson data, using the previous example to construct a mean 
RNGversion("3.6.0"); set.seed(1)                          
zdf$z <- Z$z <- rpois(m, lambda = Z$z^2)

## Run FRK
S <- FRK(f = z ~ 1,                           # Formula to FRK
         list(Z),                             # All datasets are supplied in list
         response = "poisson",                # Poisson data model
         link = "square-root")                # square-root link function
Pred <- predict(S)                            # Prediction stage

xy <- data.frame(coordinates(Pred$newdata))   # Extract info from predictions
xy$mu <- Pred$newdata$p_mu
xy$se <- Pred$newdata$RMSPE_mu

## Plotting
data_plot <- ggplot(zdf) + geom_point(aes(x,y,colour=z)) + 
  scale_colour_distiller(palette="Spectral") + theme_bw() + coord_fixed()
pred_plot <- ggplot(xy) + geom_raster(aes(x,y,fill=mu)) + 
  scale_fill_distiller(palette="Spectral") + theme_bw() + coord_fixed()
se_plot <- ggplot(xy) + geom_tile(aes(x,y,fill=se)) + 
  geom_point(data=zdf,aes(x,y),pch=46) +
  scale_fill_distiller(palette = "BrBG", n.breaks = 4) + theme_bw() + coord_fixed() 
ggarrange(data_plot, pred_plot, se_plot, nrow = 1, align = "hv", legend = "top")  
             
```    
<!---
ggsave( 
  filename = "Poisson_data.png", device = "png", 
  width = 10, height = 4,
  path = "~/Desktop/"
)
--->

![(Left) Poisson data. (Centre) Prediction of the mean response. (Right) Standard error of the mean response.](/man/figures/Poisson_data.png?raw=true)


[//]: # (Currently `FRK` is not installing on OSX with `build_vignettes=TRUE` as it fails to find `texi2dvi`. Set `build_vignettes=FALSE` to ensure installation. Then download the `.Rnw` file in the `vignettes` folder and compile the pdf file separately in `RStudio` with `knitr`. )


Demonstrations
--------------

The package `FRK` is currently being used to generate spatio-temporal animations of fields observed by satellite data. [Here](https://www.youtube.com/watch?v=_kPa8VoeSdM) we show a daily prediction of CO2 using data from the NASA OCO-2 between September 2014 and June 2016.

[![alt tag](https://img.youtube.com/vi/ENx4CIZdoQk/0.jpg)](https://www.youtube.com/watch?v=ENx4CIZdoQk)

Acknowledgements
--------------

Thanks to [Michael Bertolacci](https://mbertolacci.github.io/) for designing the FRK hex logo!