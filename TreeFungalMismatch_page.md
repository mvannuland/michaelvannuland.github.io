## Measuring spatial changes in tree-fungal partnerships
 
### Project 1 --- Michael Van Nuland 
**Description:** This code comes from an ongoing project where I am forecasting how climate change may cause geographic mismatches in North American ectomycorrhizal symbioses. Because this work is unpublished and in collaboration with researchers at Stanford University and UC Santa Cruz, I am unable to share the full datasets and methods right now. However, I can describe the general approach and provide a few data objects to recreate some recent analyses and figures below.
<br>
<br>
Climate disruptions may result in spatial mismatches between plants and microbial distributions, and the loss of historical interactions or gain of novel associations can have important consequences for biodiversity and ecosystem functioning. Biogeographic mismatches caused by climate change might arise if plants and microbes respond to different climate variables or if they differ in their sensitivity to the same variable. Previous work has shown how climate warming is predicted to reduce ectomycorrhizal fungal diversity in pine-dominated forest systems <a href="https://doi.org/10.1111/jbi.13802">(Steidinger et al. 2020)</a>, but this assumes that tree and fungal species ranges are static. Realistically, both plant and fungal distributions are likely to be reshuffled (to various extents) as their environments change. Forecasting where species interactions may be gained/lost on the landscape is a key first step towards linking changes in plant-mycorrhizal symbioses to ecosystem processes and forest resiliency.
<br>
<br>
The general approach is to build <a href="https://en.wikipedia.org/wiki/Species_distribution_modelling">species distribution models (SDMs)</a> for trees and their mycorrhizal fungi partners, calculate the extent of overlapping habitat suitability under current climate conditions, and compare this to projected overlap levels under future climates. To do this, I am using tree species occurrence records gathered from the <a href="https://doi.org/10.1111/2041-210X.12861">‘bien’ R package</a> (which uses databases such as <a href="https://www.fia.fs.fed.us/tools-data/">FIA</a> and <a href="https://www.gbif.org/">GBIF</a>, among others), and ectomycorrhizal occurrence records using fungal sequence data from samples across North America that capture the majority of ecoregions and climate space. Additionally, each fungal sample included details on aboveground woody vegetation, which I use to cross-reference and filter tree species IDs with ectomycorrhizal occurrences to create host-specific models of fungal species and community distributions. While I cannot share all the data-wrangling and SDM steps just yet, the data and code below shows some of my thoughts on how to combine plant and fungal observations in unique ways to model their potential climate change responses.
<br>
<br>
### Contents:
**1   -** Setup
<br>
**2   -** Estimating geographic overlap between tree species and fungal species ranges
<br>
**3   -** Mapping fungal diversity "left behind" as tree species distributions respond to climate change
<br>
**4   -** Summarizing fungal species range shift extent and direction
<br>

### 1. Setup
Load R libraries and data objects. 

```javascript

# Anlaysis


# Spatial tools and mapping

# Plotting

```

### 2. Estimating geographic overlap between tree species and fungal species ranges
DESCRIPTION
```javascript

```
<p align="center"><img src="images/?" alt="drawing" width="900"/></p>


### 3. Mapping fungal diversity "left behind" as tree species distributions respond to climate change
DESCRIPTION
```javascript

```
<p align="center"><img src="images/?" alt="drawing" width="900"/></p>


### 4. Summarizing fungal species range shift extent and direction
DESCRIPTION
```javascript

```
<p align="center"><img src="images/?" alt="drawing" width="600"/></p>
