---
title: Data Analysis of Maryland Counties Using R Markdown
---

This project was done my junior year of college, and the goal was to implement some R Markdown skills and create models comparing per capita income and walking rate to work in Anne Arundel County, Baltimore County, and Baltimore City. The code below is only for Baltimore County since the code is similar for all three places. There are no visuals for this page but, a link to a PDF of this file can be found by clicking [this link](488github.pdf). This page is only for the purpose of showcasing my entry level experience with using R. 

## Baltimore County

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r, warning=FALSE}
#install.packages("tidycensus")
#install.packages("tidyverse")
#install.packages("tigris")
#install.packages("sf")
library(tidycensus)
library(tidyverse)
library(tigris)
library(sf)
library(RColorBrewer)
options(tigris_class = "sf")
options(tigris_use_cache = TRUE)
census_api_key("02fd95ffa4b152f4183ec87b9bc382caf029e468")
```

#Before knitting document, install all packages!

This is where I will get the data for Baltimore County. In the get.acs function I specificed Baltimore County as the county so it would only retrieve the data for that area. 
```{r, warning=FALSE}
fd <- get_acs(geography = "tract", 
                 variables = c("B08301_019","B08301_001","B19301_001"),
                 state = c("MD"), county = "Baltimore County", geometry = TRUE, output = "wide")
metros <- core_based_statistical_areas(cb = TRUE) %>%
  filter(GEOID %in% c("12580")) %>%
  select(metro_name = NAME)

balcit <- st_join(fd, metros, join = st_within, left = FALSE) 
head(balcit)
```

This section divides the amount of people that walk to work by the total number of people using all types of transportation to get the walking rate.  
```{r}
balcit$walking_rate <- balcit$B08301_019E / balcit$B08301_001E
```


```{r}
balcit.lm <- lm(balcit$B19301_001E ~ balcit$walking_rate)
```


```{r}
balcit_comp <- balcit %>% filter(walking_rate >= 0 & B19301_001E >= 0)
```

This code runs a moran's test for per capita income which calculates if the variable is spatially autocorrelated with itslef. 
```{r bmore_analysis, warning=FALSE}
#install.packages("spdep")
library(spdep)
library(sf)
c <- poly2nb(as(balcit_comp, "Spatial"), row.names=balcit_comp$GEOID)
ww <- nb2listw(c)
balcit_comp.moran <- moran.test(balcit_comp$B19301_001E, listw=ww, randomisation=FALSE)
balcit_comp.moran
```

This code runs a moran's test for walking rate which calculates if the variable is spatially autocorrelated with itslef. 
```{r, warning=FALSE}
#install.packages("spdep")
library(spdep)
library(sf)
c <- poly2nb(as(balcit_comp, "Spatial"), row.names=balcit_comp$GEOID)
ww <- nb2listw(c)
balcit_comp.moran <- moran.test(balcit_comp$walking_rate, listw=ww, randomisation=FALSE)
balcit_comp.moran
```

```{r}
balcit_comp.z <- (balcit_comp.moran$estimate[1] - balcit_comp.moran$estimate[2]) /
                           balcit_comp.moran$estimate[3]
summary(balcit_comp.z)
```

From this code we find that per capita income is spatially autocorrelated with itself in Baltimore County due to the value being closer to 1. Walking rate however is not spatially autocorrelated with itself as much due to the number being closer to 0. The Z score tells us that there is some clustering between the values due to it being a postive number. 

```{r}
balcit.lm <- lm(balcit$B19301_001E ~ balcit$walking_rate)
ggplot(balcit, aes(x = balcit$walking_rate , y = balcit$B19301_001E)) +
  geom_point(shape = 1) +
  geom_smooth(method = "lm", se = FALSE, color = "purple")
```

The graph shown above shows that theses two datasets are in fact correlated. It shows that as per capita income increases, the walking rate decreases. This means that in the Baltimore County area, people who make more money are less likely to walk to work. This is the opposite than what is found in Baltimore City. 

This code turns the dataset into a spatial data base so it can be used in ggplot. 
```{r}
library(broom)
balcit_comp.sp <- as(balcit_comp, 'Spatial')
balcit_df <- tidy(balcit_comp.sp)
head(balcit_df)
```

```{r}
balcit_comp.sp$polyID <- sapply(slot(balcit_comp.sp, "polygons"), function(x) slot(x, "ID"))
balcit_df <- merge(balcit_df, balcit_comp.sp, by.x = "id", by.y="polyID")
head(balcit_df)
```

Below is the map for Walking Rate in Baltimore County.
```{r}
library(ggplot2)

ggplot() +                                               
  geom_polygon(                                         
    data = balcit_df,                                   
    aes(x = long, y = lat, group = group,                
        fill = cut_number(walking_rate, 4))) +                
  scale_fill_brewer("Walking Rate", palette = "Blues") + 
  ggtitle("Walking Rate in Baltimore County") +    
  theme(line = element_blank(),                          
        axis.text=element_blank(),                       
        axis.title=element_blank(),                      
        panel.background = element_blank()) +            
  coord_fixed(1.3)                                         
```

Below is the map for Per Capita Income in Baltimore County.
```{r}
library(ggplot2)

ggplot() +                                               
  geom_polygon(                                         
    data = balcit_df,                                   
    aes(x = long, y = lat, group = group,                
        fill = cut_number(B19301_001M, 4))) +                
  scale_fill_brewer("Per Capita Income", palette = "Blues") + 
  ggtitle("Per Capita in Baltimore County") +    
  theme(line = element_blank(),                          
        axis.text=element_blank(),                       
        axis.title=element_blank(),                      
        panel.background = element_blank()) +            
  coord_fixed(1.3)                                         
```


```{r}
#install.packages("ggmap")
library(ggmap)
myLocation <- myLocation <- c(lon = -76.61, lat = 39.29)
geocode("Maryland")
```

Below is a map of Per Capita Income in Baltimore County using ggmap. 

```{r}
BaltimoreMap <- get_map(location=myLocation, zoom = 9,
source="google", maptype="terrain", crop=FALSE)
ggmap(BaltimoreMap) + 
  geom_polygon(                                         
    data = balcit_df,                                   
    aes(x = long, y = lat, group = group,                
        fill = cut_number(B19301_001M, 4))) +                
  scale_fill_brewer("Per Capita Income", palette = "Blues") + 
  ggtitle("Per Capita Income in Baltimore County") +    
  theme(line = element_blank(),                          
        axis.text=element_blank(),                       
        axis.title=element_blank(),                      
        panel.background = element_blank()) +            
  coord_fixed(1.3) 
```

Below is a map of Walking Rate in Baltimore County using ggmap. 

```{r}
BaltimoreMap <- get_map(location=myLocation, zoom =9,
source="google", maptype="terrain", crop=FALSE)
balcit_comp.sp <- as(balcit_comp, 'Spatial')
ggmap(BaltimoreMap) + 
  geom_polygon(                                         
    data = balcit_df,                                   
    aes(x = long, y = lat, group = group,                
        fill = cut_number(walking_rate, 4))) +                
  scale_fill_brewer("Walking Rate", palette = "Purples") + 
  ggtitle("Walking Rate in Baltimore County") +    
  theme(line = element_blank(),                          
        axis.text=element_blank(),                       
        axis.title=element_blank(),                      
        panel.background = element_blank()) +            
  coord_fixed(1.3) 
```

The code below takes values in the walking rate column and reassigns them new values high, medium, and low. 
```{r}
walk_hml <- function(walk_rate) {
  sapply(walk_rate, function(walk_rate){
  if(is.na(walk_rate) | is.null(walk_rate)) {
    return("N/A")
  } else if(walk_rate >= 0  & walk_rate <=0.01) {
    return("lo")
  } else if(walk_rate > 0.015 & walk_rate <=0.02) {
    return("md")
  } else if(walk_rate > 0.022 & walk_rate <= 0.4) {
    return("hi")
  } else {
    return("ERR")
  }
})}

# Testing our four outputs. This should all evaluate to TRUE
walk_hml(NA) == "N/A" # case 1
walk_hml(0.0) == "lo" # case 2
walk_hml(0.001) == "lo" # case 2
walk_hml(0.02) == "md" # case 3
walk_hml(0.025) == "hi" # case 4
walk_hml(0.4) == "hi" # case 4
walk_hml("meme") == "ERR" # else

# You should modify this range because the max is about 0.5, and the 75% cutoff is about .1
summary(balcit$walking_rate)
```

This code makes a new columb and brings in the outputs from the previous code. 
```{r}
balcit <- balcit %>% mutate(walking_rate_hml = walk_hml(walking_rate))

unique(balcit$walking_rate_hml)

plot(balcit["walking_rate_hml"])

```

This code takes the values for per capita income and reassigns them new values high, medium, and low.
```{r}
pci_hml <- function(B19301_001E) {
  sapply(B19301_001E, function(B19301_001E){
  if(is.na(B19301_001E) | is.null(B19301_001E)) {
    return("N/A")
  } else if(B19301_001E >= 0  & B19301_001E <=25000) {
    return("lo")
  } else if(B19301_001E > 25100 & B19301_001E <=50000) {
    return("md")
  } else if(B19301_001E > 50100 & B19301_001E <= 110000) {
    return("hi")
  } else {
    return("ERR")
  }
})}

# Testing our four outputs. This should all evaluate to TRUE
pci_hml(NA) == "N/A" # case 1
pci_hml(0.0) == "lo" # case 2
pci_hml(25000) == "lo" # case 2
pci_hml(30000) == "md" # case 3
pci_hml(51000) == "hi" # case 4
pci_hml(101000) == "hi" # case 4
pci_hml("meme") == "ERR" # else

# You should modify this range because the max is about 0.5, and the 75% cutoff is about .1
summary(balcit$B19301_001E)
```

This code creates a new column from the new values created in the previous code. 
```{r}
# Create the new column
balcit <- balcit %>% mutate(pci_rate_hml = pci_hml(B19301_001E))

# Now spit out the outputs (notice there's only low and medium)
unique(balcit$pci_rate_hml)

# Plot it
plot(balcit["pci_rate_hml"])

```

This code joins both of the new values into a new column to compare the two variables. 
```{r}
balcit2 <- balcit

balcit2 <- balcit2 %>% mutate(walkpci = paste(walking_rate_hml, pci_rate_hml, sep="&"))

plot(balcit2["walkpci"])
```

This turns the new dataset into a spatial dataframe so it can be used in ggplot. 
```{r}
library(broom)
balcit2.sp <- as(balcit2, 'Spatial')
balcit2_df <- tidy(balcit2.sp)
head(balcit2_df)
```

```{r}
balcit2.sp$polyID <- sapply(slot(balcit2.sp, "polygons"), function(x) slot(x, "ID"))
balcit2_df <- merge(balcit2_df, balcit2.sp, by.x = "id", by.y="polyID")
head(balcit2_df)
```

This code plots the most significant values from the new column that has both walking rate and per capita income in it. 
```{r}
library(ggplot2)

balsout <- ggplot() +                                               
  geom_polygon(data = balcit2_df, 
               aes(x = long, y = lat, group = group), 
               #color="#999999", 
               fill = "#cfcfcf") +
  geom_polygon(data = balcit2_df[balcit2_df$walkpci == "hi&hi", ], 
               aes(x = long, y = lat, group = group), 
               fill = "#1f78b4") +
  geom_polygon(data = balcit2_df[balcit2_df$walkpci == "lo&lo", ], 
               aes(x = long, y = lat, group = group), 
               fill = "#33a02c") +
  geom_polygon(data = balcit2_df[balcit2_df$walkpci == "hi&lo", ], 
               aes(x = long, y = lat, group = group), 
               fill = "#a6cee3") +
  geom_polygon(data = balcit2_df[balcit2_df$walkpci == "lo&hi", ], 
               aes(x = long, y = lat, group = group), 
               fill = "#b2df8a") +
  ggtitle("Walking Rate & Per Capita Income - Baltimore County") +
  theme(line = element_blank(),                          
        axis.text=element_blank(),                       
        axis.title=element_blank(),                      
        panel.background = element_blank()) +            
  coord_fixed(1.3)  
balsout
```

The next two blocks of codes create leaflet maps of both walking rate and per capita income. 
```{r}
#install.packages("leaflet")
#install.packages("tmap")
library(leaflet)
library(tmap)
pal_fun <- colorQuantile("Purples", NULL, n = 4)

p_popup <- paste0("<strong>Walking Rate: </strong>", balcit_comp.sp$walking_rate)
leaflet(balcit_comp.sp) %>%
  addPolygons(
    stroke = FALSE, 
    fillColor = ~pal_fun(walking_rate),
    fillOpacity = 0.8, smoothFactor = 0.5,
    popup = p_popup,
    group = "Baltimore County") %>%
    addTiles(group = "OSM") %>%
  addProviderTiles("CartoDB.DarkMatter", group = "Carto") %>%
  addLegend("bottomright",  
            pal=pal_fun,   
            values=~walking_rate, 
            title = 'Walking Rate in Baltimore County') %>% 
  addLayersControl(baseGroups = c("OSM", "Carto"), 
                   overlayGroups = c("Baltimore County")) 
```

```{r}

library(leaflet)
library(tmap)
library(classInt)
pal <- brewer.pal(4, "OrRd")
breaks_qt <- classIntervals(balcit_comp.sp$B19301_001E, 
                            n = 4, 
                            style = "quantile")
pal_fun <- colorQuantile("Reds", NULL, n = 4)

p_popup <- paste0("<strong>B19301_001E: </strong>", balcit_comp.sp$B19301_001E)

leaflet(balcit_comp.sp) %>%
  addPolygons(
    stroke = FALSE, 
    fillColor = ~pal_fun(B19301_001E),
    fillOpacity = 0.8, smoothFactor = 0.5,
    popup = p_popup,
    group = "Baltimore County") %>%
    addTiles(group = "OSM") %>%
  addProviderTiles("CartoDB.DarkMatter", group = "Carto") %>%
  addLegend("bottomright",  
           colors = brewer.pal(4, "YlOrRd"), 
            labels = paste0("$", as.character(round(breaks_qt$brks[-1]))),
            title = 'Per Capita Income ') %>%
  addLayersControl(baseGroups = c("OSM", "Carto"), 
                   overlayGroups = c("Baltimore County")) 
```

From the maps made from this project we can see that there is a trend in all three counties. In Baltimore City it can be seen that with high per capita income comes high rate of walking to work, which can be explained by wealthier people living closer to the center of the city and using walking as their main form of commuting. In Baltimore County, we see an opposite trend where with higher per capita income there is a lower rate of walking. This is most likely due to wealthier people living in the suburbs outside of Baltimore and using a car or public transportation to get to work. Anne Arundel County shows the same trend but not as significant, and this could be due to people not living as close to a city center like people in Baltimore County. People in Anne Arundel County could drive to work if they work in DC or Baltimore, but if they work more locally they might walk. This is why the trend is not as strong in Anne Arundel County. 



