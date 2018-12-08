---
title: Final Project Proposal
---
by Kristian Nelson

### Part 1 

For my final project proposal I plan to create a mock-up insurance job that will look at expensive
houses in Hawaii and the level of risk they face from lava flow or fires, which will help insurance companies determine
what houses they should insure and how much they should make the home insurance. 

My question is: Where are there houses worth over $765,000 (above average housing price) that are within areas of high fire 
risk, and/or are in the potential path of lava?

My lava flow data will come from [here.](http://geoportal.hawaii.gov/datasets/volcano-lava-flow-hazard-zones)

My fire risk data will come from [here.](http://geoportal.hawaii.gov/datasets/fire-risk-areas)

My housing data will come from [here.](http://geoportal.hawaii.gov/datasets/1eb5fa03038d49cba930096ea67194e0_5)

My DEM data will come from [here.](http://www.soest.hawaii.edu/coasts/data/hawaii/dem.html)

The first three non-superficial tools I am going to use SQL, Python, and 3D mapping for this project. Using python, 
I will load in the fire risk and lava flow data and use a selection tool to load in an already cleaned up shape file 
that only has the High/Medium/Low values I need, as well as get rid of any N/A values. I am going to use the QGIS 3D 
mapping tool to show a unique view of Hawaii where youcan see the elevation and why the lava flows around the 
island the way that it does.

I will know when I have answered my question when I have 3 maps of risk, lava risk, and expensive homes and do an analysis on 
where there are houses at risk and where there are houses that are not at risk. 

### Part 2 

In addition to the 3 tools previously described, I am also going to do a heat map analysis where I will create centroids for 
each of the housing poligons. This will then be used to show where there are high concentrations of high priced houses. I chose
this because it will help with the analysis since the housing polygons are somewhat small compared to the entire island. 
This is more invloved because it combines the skills from multiple labs together into one project. 

If I am able to I want to create a model that looks at a possible correlation between fire risk areas and lava flow areas. So
far I have not been able to do this but I will be in the final project if I can do it successfully.

### Part 3

To go above and beyond, I also want to try and incorporate some remote sensing into this project. I have never used a Thermal
band from Landsat 8 and would like to download some imagery from around May 3, 2018 when Kilauea erupted, and use the Thermal
band to show where the lava flowed across the island. 
