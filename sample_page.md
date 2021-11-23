## Project 1: Quantifying and mapping plant trait limits at climate extremes
 
**Project description:** This project shows how to combine quantile regressions with plant trait data and climate variables to better understand species geographic ranges. Data are from Van Nuland et al. (2020) "Intraspecific trait variation across elevation predicts a widespread tree species' climate niche and range limits" in <em>Ecology and Evolution</em>.
 
Briefly, this study leveraged the variation across thousands of plant trait measuresments (e.g., leaf area, tree diameter) to predict where climates may be too difficult for a tree species to grow and thrive. The ecological idea behind this project is relatively straightforward: if climate stress leaves a consistent signature on plant trait variation, then trait distributions should be informative for predicting the temperature and precipitation extremes that define species range limits. Here, using quantile regression is helpful because you might expect that a given trait-climate relationship could differ between the upper 95th percentile, median 50th percentile, and lower 5th percentile of the climate gradient. For example, leaf traits might respond differently to temperature extremes at the upper warm edge vs. the lower cold edge of the species climate range, and quantile regressions can be useful for teasing apart these differences. 

Below is an overview of the approach I used to sample plant traits across elevation gradients (which act as natural climate gradients) to capture the necessary variation in trait-climate relationships in order to test this idea. See the full project and results from the paper here for more information: https://doi.org/10.1002/ece3.5969.
<p align="center"><img src="images/TraitClimateOverview.png?" alt="drawing" width="500"/></p>


### 1. Getting started
Load the relevant R libraries and project dataset. 

```javascript

```

### 2.1 Hypothesis testing with linear mixed effects model
This model tests whether the mycorrhizal fungi treatments (MYC), nutrient fertilization treatments (Nitrogen, Phosphorus), and/or their interactions affect total plant biomass. The experiment was created with randomized blocks (REP) which are included as random effects to account for any potential variation in the room where plants were growing that might have unintentionally altered their growth.

```javascript

```

One big take-away from these results is in the MYC x P interaction term, which shows that plant growth responses to phosphorus fertilization depend on the mycorrhizal treatments (evidence that supports one of the main hypotheses in this project).


### 2.2 Partial regression plots
We can explore these patterns further with partial regressions, which is one way to isolate single variable effects while accounting for the variation attributed to other variables using residuals.

```javascript
# Calculate residuals for Nitrogen and Phosphorus effects (isolate variation explained by Nitrogen vs. Phosphorus nutrient treatments)
TotalBiomass.mod.N.resids <- lm(log(TotalBiomass) ~ log(Nitrogen), data = pinus.myc.dat)
TotalBiomass.mod.P.resids <- lm(log(TotalBiomass) ~ log(Phosphorus), data = pinus.myc.dat)
N.resids <- resid(TotalBiomass.mod.N.resids)  
P.resids <- resid(TotalBiomass.mod.P.resids)
pinus.myc.dat <- cbind.data.frame(pinus.myc.dat, N.resids, P.resids)

# Plot partial regressions using residuals
ggplot(aes(x=log(Nitrogen), y=P.resids), data = pinus.myc.dat) +
  geom_point(position = position_jitterdodge(dodge.width = 0.3, jitter.width = 0.1), size = 1.5, aes(color = MYC), show.legend = T) +
  geom_smooth(method = "loess", se=F, lwd=1, show.legend = F, aes(color = MYC), formula = 'y~x') +
  scale_colour_viridis_d("Fungal treatment", direction = -1) +
  xlab("Log Nitrogen (mg/kg soil)") +
  ylab("Total Plant Biomass (residuals)") +
  ylim(-1.5, 1.2) +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank())

ggplot(aes(x=log(Phosphorus), y=N.resids), data = pinus.myc.dat) +
  geom_point(position = position_jitterdodge(dodge.width = 0.3, jitter.width = 0.1), size = 1.5, aes(color = MYC), show.legend = T) +
  geom_smooth(method = "loess", se=F, lwd=1, show.legend = F, aes(color = MYC), formula = 'y~x') +
  scale_colour_viridis_d("Fungal treatment", direction = -1) +
  xlab("Log Phosphorus (mg/kg soil)") +
  ylab("Total Plant Biomass (residuals)") +
  ylim(-1.5, 1.2) +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank())
```
<p align="center"><img src="images/growthplots.jpeg?" alt="drawing" width="700"/></p>


### 3. 3-D response surface plots
Visualizing plant growth simultaneously across the two-dimensional axes of critical soil resources (nitrogen and phosphorus).

```javascript

```


### 4. Provide a basis for further data collection through surveys or experiments

