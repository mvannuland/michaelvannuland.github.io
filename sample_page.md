## Project 1: Quantifying the benefits of fungal symbiont partners to plant hosts

**Project description:** This project shows an example of customized scripts for linear mixed effects modeling, matrix bootstrapping, and convex hull analysis of response surface volumes in R.

Data are from Van Nuland et al. (2020) "Symbiotic niche mapping reveals functional specialization by two ectomycorrhizal fungi that expands the host plant niche" in <em>Fungal Ecology</em>. Briefly, this experiment tested how different species of symbiotic fungi (mycorrhizal fungi) influence plant responses to soil nutrient limitation. I hypothesized that the fungi would provide greater benefits to the plants under increasing nutrient stress, but that the extent of these positive growth effects would differ depending on the specific fungal species and type of nutrient stress (e.g., nitrogen vs. phosphorus limitation). 

See the full project on Github here: https://github.com/mvannuland/pinus_myc_project,  
and more thorough descriptions and interpretations in the paper here: https://doi.org/10.1016/j.funeco.2020.100960.

### 1. Getting started
Load the relevant R libraries and project dataset.

```javascript
# Libraries for data wrangling and analysis 
library(lme4)
library(lmerTest)
library(geometry)
library(fields)
library(reshape2)

# Libraries for plotting
library(ggplot2)
library(ggpubr)
library(manipulateWidget)
library(plotly)
library(viridis)
library(webshot)
theme_set(theme_bw())

# Load dataset
pinus.myc.dat <- readRDS(file="pinus.myc.rds")

# Subset data by fungal treatments (for response surface analysis later)
Control <- subset(pinus.myc.dat, pinus.myc.dat$MYC=="Control")
Fungi1 <- subset(pinus.myc.dat, pinus.myc.dat$MYC=="Fungi1")
Fungi2 <- subset(pinus.myc.dat, pinus.myc.dat$MYC=="Fungi2")
Mixed <- subset(pinus.myc.dat, pinus.myc.dat$MYC=="Mixed (F1+F2)")
```

### 2.1 Hypothesis testing with linear mixed effects model
This model tests whether the mycorrhizal fungi treatments (MYC), nutrient fertilization treatments (Nitrogen, Phosphorus), and/or their interactions affect total plant biomass. The experiment was created with randomized blocks (REP) which are included as random effects to account for any potential variation in the room where plants were growing that might have unintentionally altered their growth.

```javascript
TotalBiomass.mod <- lmer(log(TotalBiomass) ~ MYC * log(Nitrogen) * log(Phosphorus) + (1|REP), data = na.exclude(pinus.myc.dat))

anova(TotalBiomass.mod)
```
| Factor      | F value    | p value    |
| :---        |   :----:   |       ---: |
| MYC         | 0.8        | 0.5        |
| Nitrogen    | 448.5      | <0.001 *** |
| Phosphorus  | 0.2        | 0.7        |
| MYC x N     | 1.7        | 0.2        |
| MYC x P     | 2.9        | 0.04   *   |
| N x P       | 65.4       | <0.001 *** |
| MYC x N x P | 0.9        | 0.5        |

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
# Create & smooth 7x7 matrices of total biomass responses across N and P gradients
Control.mat <- acast(Control, Nitrogen~Phosphorus, value.var = "TotalBiomass", mean, drop=TRUE)
Fungi1.mat <- acast(Fungi1, Nitrogen~Phosphorus, value.var = "TotalBiomass", mean, drop=TRUE)
Fungi2.mat <- acast(Fungi2, Nitrogen~Phosphorus, value.var = "TotalBiomass", mean, drop=TRUE)
Mixed.mat <- acast(Mixed, Nitrogen~Phosphorus, value.var = "TotalBiomass", mean, drop=TRUE)

# Smooths over missing matrix values
Control.mat.smooth <- image.smooth(Control.mat, theta=1)
Fungi1.mat.smooth <- image.smooth(Fungi1.mat, theta=1)
Fungi2.mat.smooth <- image.smooth(Fungi2.mat, theta=1)
Mixed.mat.smooth <- image.smooth(Mixed.mat, theta=1)

# Set same Z-axes limits across plots for easier comparison
axz <- list(range = c(180, 1180), title=list(text="Plant biomass")) 

# Control
Control.surface <-
  plot_ly( z = ~Control.mat.smooth$z) %>% 
  add_surface(opacity = 0.95) %>% 
  hide_colorbar() %>%
  layout(
  title=list(text="Control", y = 0.90, font=list(color="black", size=20)),
  scene=list(
    aspectmode = 'cube',
    xaxis=list(title="Phosphorus", autorange = "reversed"),
    yaxis=list(title="Nitrogen", autorange = "reversed"), 
    zaxis=axz))

# Fungi1
Fungi1.surface <- 
  plot_ly( z = ~Fungi1.mat.smooth$z) %>% 
  add_surface(opacity = 0.95) %>% 
  hide_colorbar() %>%
  layout(
  title=list(text="Fungi 1", y = 0.90, font=list(color="black", size=20)),
  scene=list(
    aspectmode = 'cube',
    xaxis=list(title="Phosphorus", autorange = "reversed"),
    yaxis=list(title="Nitrogen", autorange = "reversed"), 
    zaxis=axz))

# Fungi2
Fungi2.surface <- 
  plot_ly( z = ~Fungi2.mat.smooth$z) %>% 
  add_surface(opacity = 0.95) %>% 
  hide_colorbar() %>%
  layout(
  title=list(text="Fungi 2", y = 0.90, font=list(color="black", size=20)),
  scene=list(
    aspectmode = 'cube',
    xaxis=list(title="Phosphorus", autorange = "reversed"),
    yaxis=list(title="Nitrogen", autorange = "reversed"), 
    zaxis=axz))

# Mixed (F1 + F2)
Mixed.surface <-
  plot_ly( z = ~Mixed.mat.smooth$z) %>% 
  add_surface(opacity = 0.95) %>% 
  hide_colorbar() %>%
  layout(
  title=list(text="Mixed (F1 + F2)", y = 0.90, font=list(color="black", size=20)),
  scene=list(
    aspectmode = 'cube',
    xaxis=list(title="Phosphorus", autorange = "reversed"),
    yaxis=list(title="Nitrogen", autorange = "reversed"), 
    zaxis=axz))

combineWidgets(Control.surface, Fungi1.surface, Fungi2.surface, Mixed.surface, ncol = 2, nrow=2)
```


### 4. Provide a basis for further data collection through surveys or experiments

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. 

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).
