---
title: "R Notebook for Data Incubator (1st graph)"
output:
  html_document:
    df_print: paged
---

```{r = FALSE}
knitr::opts_chunk$set(
  fig.path = "README_figs/README-"
)
library(readr)
library(ggplot2)
library(dplyr)
library(maps)
library(mapproj)
library(ggthemes)
library(maptools)

latlong2state <- function(pointsDF) {
  # Prepare SpatialPolygons object with one SpatialPolygon
  # per state (plus DC, minus HI & AK)
  states <- map('state', fill=TRUE, col="transparent", plot=FALSE)
  IDs <- sapply(strsplit(states$names, ":"), function(x) x[1])
  states_sp <- map2SpatialPolygons(states, IDs=IDs,
                                   proj4string=CRS("+proj=longlat +datum=WGS84"))
  
  # Convert pointsDF to a SpatialPoints object 
  pointsSP <- SpatialPoints(pointsDF, 
                            proj4string=CRS("+proj=longlat +datum=WGS84"))
  
  # Use 'over' to get _indices_ of the Polygons object containing each point 
  indices <- over(pointsSP, states_sp)
  
  # Return the state names of the Polygons object containing each point
  stateNames <- sapply(states_sp@polygons, function(x) x@ID)
  stateNames[indices]
}

FEMA <- read_csv("C:/Users/yeung/Documents/FEMA.csv")
FEMA <- FEMA[!is.na(FEMA$latitude), ]
FEMA <- FEMA[!is.na(FEMA$longitude), ]

FEMA$state <- latlong2state(data.frame(FEMA$longitude, FEMA$latitude))

FEMA <- filter(FEMA, FEMA$amountpaidonbuildingclaim > 0)
FEMA <- FEMA[!is.na(FEMA$state),]

FEMA_average <- FEMA %>% group_by(state) %>% 
  summarise(total.amount = sum(amountpaidonbuildingclaim)/n())

us_states <- map_data("state")

FEMA_average$region <- tolower(FEMA_average$state)

FEMA_state_average <- left_join(us_states, FEMA_average)

FEMA_map <- ggplot(data = FEMA_state_average,
            aes(x = long, y = lat, group = group, fill = total.amount)) +
            geom_polygon(color = "gray90", size = 0.1) +
            coord_map(projection = "albers", lat0 = 39, lat1 = 45) +
            theme_map() +
            scale_fill_gradient2(low = "blue", high = "red",
            breaks = c(0, 10000, 20000, 30000, 40000)) +
            labs(title = "Amount paid per flood insurance building claim (1973-2019)", fill = "USD$")
FEMA_map
```
