## Project 1: Quantifying and mapping plant trait limits at climate extremes
 
**Project description:** This project shows how to combine quantile regressions with plant trait data and climate variables to better understand species geographic ranges. Data are from Van Nuland et al. (2020) "Intraspecific trait variation across elevation predicts a widespread tree species' climate niche and range limits" in <em>Ecology and Evolution</em>.
 
Briefly, this study leveraged the natural variation across plant trait measuresments (e.g., leaf area, tree diameter) to predict where climates may be too difficult for a tree species to grow and thrive. The ecological idea behind this project is relatively straightforward: if climate stress leaves a consistent signature on plant trait variation, then trait distributions should be informative for predicting the temperature and precipitation extremes that define species range limits. Here, using quantile regression is helpful because you might expect that a given trait-climate relationship could differ between the upper 95th percentile, median 50th percentile, and lower 5th percentile of the climate gradient. For example, leaf traits might respond differently to temperature extremes at the upper warm edge vs. the lower cold edge of the species climate range, and quantile regressions can be useful for teasing apart these differences. 

Below is an overview of the approach I used to sample plant traits across elevation gradients (which act as natural climate gradients) to capture the necessary variation in trait-climate relationships in order to test this idea. 
 
See the full project and results from the paper here for more information: https://doi.org/10.1002/ece3.5969.
<p align="center"><img src="images/TraitClimateOverview.png?" alt="drawing" width="600"/></p>
 
 
### 1. Getting started
Load the relevant R libraries, plant data, and climate data. 

```javascript
# Load libraries, trait data, and climate data

# Quantile regression
library(quantreg)

# Spatial analysis and mapping
library(maptools)
library(gpclib)
library(rgdal)
library(maps)
library(raster)
library(sp)
library(plyr)
library(plotrix)
library(ggsn)
library(rgeos)
library(ggplot2)

# Load plant trait data 
plant.dat <- read.csv(file="Plant_traits.csv")

# Load 19 'Bioclim' variables of climate data from worldclim website (https://www.worldclim.org/data/worldclim21.html)
climate.dat <- getData("worldclim", var="bio", res=2.5) # normally you can download from the server like this, but not working at the moment

# Work-around is to download the geotif layers .zip file and import (using 10min resolution here for speed)
climate.list <- list.files(path="wc2.1_10m_bio", pattern = '.tif$', all.files=TRUE, full.names=TRUE)
climate.stack <- lapply(climate.list, raster)

# Extract temperature and precipitation data for the lat/long coordinates and add to plant trait dataset
plant.temp.dat <- raster::extract(climate.stack[[1]], plant.dat[2:3]) # bio1 is mean annual temperature (degrees C)
plant.precip.dat <- raster::extract(climate.stack[[4]], plant.dat[2:3]) # bio12 is annual precipitation (mm)

plant.climate.dat <- cbind.data.frame(plant.dat, Temperature = plant.temp.dat, Precipitation = plant.precip.dat)
```

### 2 Quantile Regressions with Temperature and Precipitation
The following code models the 5th, 50th, and 95th quantile regressions for each trait-temperature and trait-precipitation relationship
```javascript
# Split dataset into list by each plant trait type
plant.climate.ls <- split(plant.climate.dat, f = plant.climate.dat$Trait_id)

# Create for loop to generate quantile regressions for each trait-climate relationship
temperature.quantregs <- list()
temperature.qr.summary <- list()
precipitation.quantregs <- list()
precipitation.qr.summary <- list()

for (i in seq_along(names(plant.climate.ls))){
  temperature.quantregs[[i]] <- rq(Temperature ~ Trait_value, tau = c(0.05, 0.5, 0.95), data = plant.climate.ls[[i]])
  precipitation.quantregs[[i]] <- rq(Precipitation ~ Trait_value, tau = c(0.05, 0.5, 0.95), data = plant.climate.ls[[i]])
  
  temperature.qr.summary[[i]] <- summary(temperature.quantregs[[i]], se="boot", bsmethod= "xy")
  precipitation.qr.summary[[i]] <- summary(precipitation.quantregs[[i]], se="boot", bsmethod= "xy")
}

names(temperature.quantregs) <- names(precipitation.quantregs) <- names(plant.climate.ls)
names(temperature.qr.summary) <- names(precipitation.qr.summary) <- names(plant.climate.ls)


# Visualizing the quantile regression differences across traits and climate variables

ggplot(plant.climate.dat, aes(x = Trait_value, y = Temperature)) +
  geom_point(size=1, na.rm=TRUE) +
  stat_quantile(geom="quantile", position="identity", quantiles=c(0.95), method="rq", color="red", size=0.7, formula = y ~ x) +
  stat_quantile(geom="quantile", position="identity", quantiles=c(0.5), method="rq", color="grey45", size=0.7,formula = y ~ x) +
  stat_quantile(geom="quantile", position="identity", quantiles=c(0.05), method="rq", color="blue", size=0.7, formula = y ~ x) +
  facet_wrap(~Trait_id, scales = "free_x") +
  labs(x="Trait value", y="Temperature (C)") +
  theme_bw() +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank()) +
  theme(legend.position="none")

ggplot(plant.climate.dat, aes(x = Trait_value, y = Precipitation)) +
  geom_point(size=1, na.rm=TRUE) +
  stat_quantile(geom="quantile", position="identity", quantiles=c(0.95), method="rq", color="red", size=0.7, formula = y ~ x) +
  stat_quantile(geom="quantile", position="identity", quantiles=c(0.5), method="rq", color="grey45", size=0.7,formula = y ~ x) +
  stat_quantile(geom="quantile", position="identity", quantiles=c(0.05), method="rq", color="blue", size=0.7, formula = y ~ x) +
  facet_wrap(~Trait_id, scales = "free_x") +
  labs(x="Trait value", y="Precipitation (mm)") +
  theme_bw() +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank()) +
  theme(legend.position="none")
```
<p align="center"><img src="images/traits_quantreg.png?" alt="drawing" width="900"/></p>


### 3

```javascript

```
<p align="center"><img src="images/growthplots.jpeg?" alt="drawing" width="700"/></p>


### 3. 3-D response surface plots
Visualizing plant growth simultaneously across the two-dimensional axes of critical soil resources (nitrogen and phosphorus).

```javascript

```


### 4. Provide a basis for further data collection through surveys or experiments

