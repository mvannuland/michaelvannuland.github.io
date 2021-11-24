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

# Anlaysis
library(quantreg)
library(FactoMineR)
library(factoextra)

# Spatial tools and mapping
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

# Plotting
library(ggplot2)
library(RColorBrewer)

# Load plant trait data (found here: https://github.com/mvannuland/DataSci_portfolio_datasets)
plant.dat <- read.csv(file="Plant_traits.csv")

# Load 19 'Bioclim' variables of climate data from worldclim website (https://www.worldclim.org/data/worldclim21.html)
climate.dat <- getData("worldclim", var="bio", res=2.5) # normally you can download from the server like this, but not working at the moment

# Work-around is to download the geotif layers .zip file and import (using 10min resolution here for speed)
climate.list <- list.files(path="wc2.1_10m_bio", pattern = '.tif$', all.files=TRUE, full.names=TRUE)
climate.list <- lapply(climate.list, raster)
climate.stack <- stack(climate.list)

# rename climate layers with shorter identifiers
climate.names <- c('bio1', 'bio10', 'bio11', 'bio12', 'bio13', 'bio14', 'bio15', 'bio16', 'bio17', 'bio18', 'bio19', 'bio2', 'bio3', 'bio4', 'bio5', 'bio6', 'bio7', 'bio8', 'bio9')

names(climate.list) <- names(climate.stack) <- climate.names

# Extract temperature and precipitation data for the lat/long coordinates and add to plant trait dataset
plant.temp.dat <- raster::extract(climate.list[[1]], plant.dat[2:3]) # bio1 is mean annual temperature (degrees C)
plant.precip.dat <- raster::extract(climate.list[[4]], plant.dat[2:3]) # bio12 is annual precipitation (mm)

plant.climate.dat <- cbind.data.frame(plant.dat, Temperature = plant.temp.dat, Precipitation = plant.precip.dat)
```

### 2. Quantile Regressions with Temperature and Precipitation
The following code models the 5th, 50th, and 95th quantile regressions for each trait-temperature and trait-precipitation relationship. In particular, the lower and upper quantiles show where the extent of plant trait variation predicts the species' climate range limits. Areas beyond the outermost regression lines indicate "No-go zones" where there are no plant trait values that allow the tree species to exist in those climate conditions.
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


### 3. Multivariate analysis of climate variables
Plants respond to more than just temperature and precipitation. Here, I show an example of how to use Principle Component Analysis to summarize the complex climate environment (i.e., combining dimensionality from all 19 climate variables) and model the subsequent plant trait-climate relationships using the new multivariate climate axes.
```javascript
# Extract full set of climate variables  
plant.climate.full <- raster::extract(climate.stack, plant.dat[2:3])

# Create PCA of climate variables
# (NOTE: because the climate data is repeated for each trait id, do this for only one trait in rows 1:48)
Climate.pca <- PCA(plant.climate.full[1:48,])

# Here we can look at which climate variables have the largest influence on the first and second PCA axis
# The red dashed line indicates the expected average percent variance explained. This means that climate variables explaining more variation can be interpreted as having a larger contribution to the princple component dimension).
fviz_contrib(Climate.pca, choice = "var", axes = 1, top = 19, fill="black", color="black") +
  labs(x="Climate variable", y="Contributions\nto PCA1 (%)", title="") +
  theme_bw() +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), axis.text.x = element_text(angle=60, hjust=0.975))

fviz_contrib(Climate.pca, choice = "var", axes = 2, top = 19, fill="black", color="black") +
  labs(x="Climate variable", y="Contributions\nto PCA2 (%)", title="") +
  theme_bw() +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), axis.text.x = element_text(angle=60, hjust=0.975))


# Plot PCA biplot
Climate.ind <- get_pca_ind(Climate.pca)

fviz_pca_biplot(Climate.pca, 
                label = "var", pointshape=16, pointsize = 1.5, col.ind="black", mean.point=F, title = "",
                col.var = "black", repel = TRUE) +
  theme_bw() +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank()) +
  labs(x = "PCA1 (46.1%)", y = "PCA2 (25.5%)")


# Extract PCA coordinates and add to plant trait dataset
PCA.coords <- as.data.frame(Climate.pca$ind$coord)

plant.climate.pca.ls <- list()

for (i in names(plant.climate.ls)){
  plant.climate.pca.ls[[i]] <- cbind.data.frame(plant.climate.ls[[i]], 'PCA1' = PCA.coords$Dim.1, 'PCA2' = PCA.coords$Dim.2)
}

# Analyzing and plotting quantile regressions with the multivariate climate dimensions

# Create for loop to generate quantile regressions for each trait-PCA1 relationship
PCA1.quantregs <- list()
PCA1.qr.summary <- list()

for (i in seq_along(names(plant.climate.pca.ls))){
  PCA1.quantregs[[i]] <- rq(PCA1 ~ Trait_value, tau = c(0.05, 0.5, 0.95), data = plant.climate.pca.ls[[i]])
  PCA1.qr.summary[[i]] <- summary(PCA1.quantregs[[i]], se="boot", bsmethod= "xy")
}

names(PCA1.quantregs) <- names(PCA1.qr.summary) <- names(plant.climate.pca.ls)

# Combine dataframes in list into single dataframe
plant.climate.pca.dat <- do.call("rbind", plant.climate.pca.ls)

ggplot(plant.climate.pca.dat, aes(x = Trait_value, y = PCA2)) +
  geom_point(size=1, na.rm=TRUE) +
  stat_quantile(geom="quantile", position="identity", quantiles=c(0.95), method="rq", color="red", size=0.7, formula = y ~ x) +
  stat_quantile(geom="quantile", position="identity", quantiles=c(0.5), method="rq", color="grey45", size=0.7,formula = y ~ x) +
  stat_quantile(geom="quantile", position="identity", quantiles=c(0.05), method="rq", color="blue", size=0.7, formula = y ~ x) +
  facet_wrap(~Trait_id, scales = "free_x") +
  labs(x="Trait value", y="Climate PCA2") +
  theme_bw() +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank()) +
  theme(legend.position="none")
```
<p align="center"><img src="images/PCA_QuantReg.png?" alt="drawing" width="900"/></p>


### 4. Mapping climate constraints on plant trait variation
In sections 2 and 3 I showed how variation in plant traits can predict the climatic limits a species can tolerate. Now, we can reverse this by applying the quantial regression equations to spatial layers of climate data. This approach generates maps that show how the extent of plant trait variation differs across the landscape. Specifically, the color gradients shows changes in projected trait diversity - darker areas are climates that can contain a broad range of plant trait values (high functional diversity), whereas lighter areas are climates that can only support a limited range of trait values (low functional diversity).
```javascript
# Download and crop US data to set map extent to western US states
US1 <- getData('GADM', country='USA', level=1)
CONUS <- c("Arizona", "California", "Colorado","Idaho","Montana","Nevada","New Mexico","Oregon","Utah","Washington","Wyoming")
state.sub <- US1[as.character(US1@data$NAME_1) %in% CONUS, ]

Climate.stack.clip  <- crop(climate.stack, extent(state.sub))
Climate.stack.clip.mask <- mask(Climate.stack.clip, state.sub)

# Pull out temperature and precipitation layers
Temperature.layer <- Climate.stack.clip.mask$bio1
Precipitation.layer <- Climate.stack.clip.mask$bio12

# Limit gridded climate data within range of sampled data (min/max values)
Temperature.layer[Temperature.layer <= min(plant.climate.pca.dat$Temperature)] <- NA
Temperature.layer[Temperature.layer >= max(plant.climate.pca.dat$Temperature)] <- NA
Precipitation.layer[Precipitation.layer <= min(plant.climate.pca.dat$Precipitation)] <- NA
Precipitation.layer[Precipitation.layer >= max(plant.climate.pca.dat$Precipitation)] <- NA

#   Solve for the trait values predicted by climate data using the trait-climate quantile regression coefficients
#   y = B0 + B1*x    =     Climate = B0 + B1*Trait     =     Trait = (Climate - B0)/B1
# See summary objects created in the quantile regression step for coefficient values used here

LeafArea.temperature.05 <- (Temperature.layer - (-1.9))/0.37
LeafArea.temperature.95 <- (Temperature.layer - 1.3)/1.14
LeafCN.temperature.05 <- (Temperature.layer - (-8.57))/0.55
LeafCN.temperature.95 <- (Temperature.layer - 1.03)/0.42
LeafPhenology.temperature.05 <- (Temperature.layer - 9.9)/-0.10
LeafPhenology.temperature.95 <- (Temperature.layer - 28.7)/-0.18
TreeDBH.temperature.05 <- (Temperature.layer - (-0.38))/0.02
TreeDBH.temperature.95 <- (Temperature.layer - 13.9)/-0.14

LeafArea.precipitation.05 <- (Precipitation.layer - 439.5)/-27.1
LeafArea.precipitation.95 <- (Precipitation.layer - 961)/-36.5
LeafCN.precipitation.05 <- (Precipitation.layer - 700)/-17.9
LeafCN.precipitation.95 <- (Precipitation.layer - 1139)/-27.0
LeafPhenology.precipitation.05 <- (Precipitation.layer - 1130)/-8.8
LeafPhenology.precipitation.95 <- (Precipitation.layer - 576)/2.0
TreeDBH.precipitation.05 <- (Precipitation.layer - 161.8)/4.3
TreeDBH.precipitation.95 <- (Precipitation.layer - 776)/0.05

# Stack trait-climate layers and clip values > 0
LeafArea.stack <- stack(LeafArea.temperature.05, LeafArea.temperature.95, LeafArea.precipitation.05, LeafArea.precipitation.95)
LeafCN.stack <- stack(LeafCN.temperature.05, LeafCN.temperature.95, LeafCN.precipitation.05, LeafCN.precipitation.95)
LeafPhenology.stack <- stack(LeafPhenology.temperature.05, LeafPhenology.temperature.95, LeafPhenology.precipitation.05, LeafPhenology.precipitation.95)
TreeDBH.stack <- stack(TreeDBH.temperature.05, TreeDBH.temperature.95, TreeDBH.precipitation.05, TreeDBH.precipitation.95)

# Identify and select the max predicted trait value per grid (to visualize the strongest climate constraints on plant traits)
LeafArea.stack.max <- stackApply(LeafArea.stack, indices = rep(1, nlayers(LeafArea.stack)), fun = max)
LeafArea.stack.max[LeafArea.stack.max < 0] <- NA

LeafCN.stack.max <- stackApply(LeafCN.stack, indices = rep(1, nlayers(LeafCN.stack)), fun = max)
LeafCN.stack.max[LeafCN.stack.max < 0] <- NA

LeafPhenology.stack.max <- stackApply(LeafPhenology.stack, indices = rep(1, nlayers(LeafPhenology.stack)), fun = max)
LeafPhenology.stack.max[LeafPhenology.stack.max < 0] <- NA

TreeDBH.stack.max <- stackApply(TreeDBH.stack, indices = rep(1, nlayers(TreeDBH.stack)), fun = max)
TreeDBH.stack.max[TreeDBH.stack.max < 0] <- NA

# Plots of two leaf traits
LeafArea.pal <- brewer.pal(9,"PuRd")
Phenology.pal <- brewer.pal(9, "YlGnBu")

plot(LeafArea.stack.max, col=LeafArea.pal, xaxt="n", yaxt="n", legend=F, main='Constraints on Leaf Area')
plot(state.sub, add = TRUE, lwd=0.5, xaxt="n", yaxt="n")

plot(LeafPhenology.stack.max, col=Phenology.pal, xaxt="n", yaxt="n", legend=F, main='Constraints on Leaf Phenology')
plot(state.sub, add = TRUE, lwd=0.5, xaxt="n", yaxt="n")
```
<p align="center"><img src="images/TraitMaps.png?" alt="drawing" width="600"/></p>
