---
title: "Species Distribution Data"
author: "Juan Carlos Rubio-Polania, PhD"
date: "2023-01-07"
---

# 🌍🦀 Script Species Distribution Data Video 📊🐍

## Overview

This workflow demonstrates multiple approaches for downloading, cleaning, analyzing, and visualizing species distribution data using both **R** and **Python**. The document integrates biodiversity databases, spatial visualization tools, occurrence retrieval packages, and mapping approaches commonly used in ecology, biogeography, marine sciences, and species distribution modeling (SDM).

The script includes examples using several biodiversity databases and APIs such as:

- GBIF
- iNaturalist
- OBIS
- BISON
- eBird
- VertNet
- EcoEngine

The workflow also demonstrates:

- Data retrieval
- Data cleaning
- Spatial filtering
- Geographic visualization
- Shapefile integration
- Mapping using base R
- Mapping using ggplot2
- Python spatial visualization
- Web scraping approaches
- Cross-language workflows using R and Python

---

# Required Packages

## R Packages

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r, include=TRUE}
M = c('dismo','intSDM','rgbif','ntbox',
      'spocc','maps','ggplot2','reticulate')

lapply(M, require, character.only = TRUE)
```

---

# dismo Package

The `dismo` package provides tools for downloading biodiversity records directly from GBIF databases.

## Downloading Occurrence Data

```{r, echo=TRUE}
g1 = gbif('Callinectes','similis',
          geo = TRUE,
          ntries = 1,
          nrecs = 300)

dim(g1)
```

## Exploring the Dataset

```{r, echo = TRUE}
head(g1[,1:5])
```

Checking the dimensions of the data frame, downloading it later, and sorting it while removing missing values in the essential categories reveal differences between the total number of records and the number of reliable records available for analyses.

```{r, echo = TRUE}
dim(g1)
```

## Removing Missing Values

```{r, echo=TRUE}
g11 = g1[!is.na(g1$individualCount) &
         !is.na(g1$lon) &
         !is.na(g1$lat) &
         !is.na(g1$datasetName),]

dim(g11)
```

---

# Basic Plot of *Callinectes similis*

```{r, fig.cap = "Figure 1. Distribution data for C. similis available from GBIF database", include = TRUE}
plot(g11$lon,
     g11$lat,
     col = "red",
     pch = 19,
     xlab = "Longitude",
     ylab = "Latitude",
     xlim = c(-100, -65),
     ylim = c(17, 40),
     main = expression(paste("Distribution data for ",
                             italic("C. similis"))))

legend("topright",
       legend = expression(italic("C. similis")),
       col = c("red"),
       pch = 19,
       box.lwd = 1)

map(add = TRUE)

GISTools::north.arrow(
  xb = -100,
  yb = 38.5,
  len = 0.33,
  lab = "N",
  col = "black",
  cex.lab = 2
)
```

---

# Visualization Using ggplot2

```{r, fig.cap = "Figure 2. Distribution data for C. similis available from the GBIF database", include = TRUE}

map_data <- map_data("world")

MapG1 = ggplot(g11, aes(x = lon, y = lat)) +
  geom_point(aes(color = individualCount), size = 2) +
  coord_cartesian(
    ylim = c(17,45),
    xlim = c(-100,-65)
  ) +
  labs(
    x = "Longitude",
    y = "Latitude",
    color = "Count"
  ) +
  theme_classic() +
  theme(
    panel.border = element_rect(
      color = "black",
      fill = NA,
      size = 1
    )
  ) +
  theme(
    text = element_text(
      size = 12,
      family = "Arial",
      colour = "black"
    ),
    axis.text.x = element_text(size = 17),
    axis.text.y = element_text(size = 17),
    axis.title.x = element_text(size = 17),
    axis.title.y = element_text(size = 17)
  ) +
  borders("world",
          colour = "black",
          size = 1) +
  ggtitle(
    expression(
      paste("Distribution data for ",
            italic("Callinectes similis"))
    )
  ) +
  theme(
    plot.title = element_text(hjust = 0.5)
  )

MapG1 +
  geom_polygon(
    data = map_data,
    aes(x = long,
        y = lat,
        group = group),
    fill = "grey",
    color = "black",
    alpha = 0.5
  )
```

---

# rgbif Package

This package provides options to retrieve data based on:
- Depth
- Elevation
- Geographic area
- Taxonomic information

The examples evaluate the depth distribution of:

- *Achelous spinicarpus*
- *Achelous spinimanus*

---

## Downloading Occurrence Data

```{r, echo=TRUE}
g21 = occ_data(
  scientificName = "Achelous spinicarpus",
  hasCoordinate = TRUE,
  limit = 30000
)

g21 = data.frame(g21$data)
```

```{r, echo=TRUE}
head(g21[,1:5])
```

```{r, echo=TRUE}
dim(g21)
```

## Removing Missing Values

```{r, echo=TRUE}
g211 = g21[
  !is.na(g21$individualCount) &
  !is.na(g21$decimalLongitude) &
  !is.na(g21$decimalLatitude) &
  !is.na(g21$datasetName),
]

dim(g211)
```

---

# Basic Plot of *A. spinicarpus* and *A. spinimanus*

```{r, fig.cap = "Figure 3. Distribution data for A. spinicarpus and A. spinimanus available from GBIF database", include = TRUE}

plot(
  g211$decimalLongitude,
  g211$decimalLatitude,
  col = "red",
  pch = 16,
  cex = 0.5,
  xlab = "Longitude",
  ylab = "Latitude",
  xlim = c(-100,0),
  ylim = c(-32,45),
  main = expression(
    paste(
      "Distribution data for ",
      italic("A. spinicarpus"),
      " (red) and ",
      italic("A. spinimanus"),
      " (orange)"
    )
  )
)

points(
  g221$decimalLongitude,
  g221$decimalLatitude,
  col = "orange",
  pch = 17,
  cex = 0.5
)

legend(
  "bottomright",
  legend = expression(
    italic("A. spinicarpus"),
    italic("A. spinimanus")
  ),
  col = c("red", "orange"),
  pch = c(16,17),
  box.lwd = 1
)

map(add = TRUE,
    col = "grey",
    fill = TRUE)

box()
```

---

# ntbox Package

The `ntbox` package provides tools for retrieving and visualizing biodiversity data from GBIF.

```{r, echo=TRUE, fig.cap = "Figure 5. Distribution data of Phyllospongia foliascens available from GBIF database", include = TRUE}

g3 = search_gbif_data(
  "Phyllospongia",
  "foliascens",
  occlim = 10000,
  writeFile = FALSE,
  leafletplot = TRUE,
  showClusters = FALSE
)
```

---

# intSDM Package

This package integrates shapefiles with occurrence data.

```{r, echo=TRUE}
shp = sf::st_read(
  dsn ="C:/shapefile/CaribbeanManagementAreas_po.shp"
)
```

```{r, echo=TRUE, fig.cap = "Figure 6. Shapefile of Puerto Rico, St. Thomas, St. John, and St. Croix islands", include = TRUE}
plot(shp)
```

---

# spocc Package

The `spocc` package allows simultaneous queries from multiple biodiversity databases.

```{r, echo=TRUE}
g4 = occ(
  query = "Dendrogyra cylindrus",
  from = c(
    'gbif',
    'bison',
    'inat',
    'ebird',
    'ecoengine',
    'vertnet',
    'obis'
  ),
  limit = 10000,
  geometry = shp
)

g4
```

---

# Using Python

The workflow also demonstrates biodiversity analyses using Python.

## Importing Libraries

```{python}
import requests
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from pygbif import occurrences
import geopandas as gpd
```

---

# Web Scraping Function Approach

```{python}
import requests

def get_data(taxon_name):

    url = f'https://api.inaturalist.org/v1/observations?taxon_name={taxon_name}&quality_grade=research&per_page=100'

    response = requests.get(url)

    if response.status_code == 200:
        return response.json()

    else:
        print(f'Error fetching data: {response.status_code}')
```

---

# Example Using *Panulirus argus*

```{python}
taxon_name = 'Panulirus argus'

sp_data = get_data(taxon_name)
```

---

# pygbif Package in Python

```{python}
occ_data = occurrences.search(
    scientificName = 'Periclimenes iridescens'
)

type(occ_data)
```

```{python}
occ_df = pd.json_normalize(
    occ_data['results']
)

type(occ_df)
```

---

# Python Spatial Visualization

```{python}
naturalearth_lowres = gpd.read_file(
    gpd.datasets.get_path('naturalearth_lowres')
)

fig, ax = plt.subplots(figsize=(10, 6))

naturalearth_lowres.plot(
    ax=ax,
    color='lightgray',
    edgecolor='black'
)

ax.set_xlim([-120, 50])
ax.set_ylim([-50,60])

ax.set_xlabel(
    'Longitude',
    color='black',
    size=11
)

ax.set_ylabel(
    'Latitude',
    color='black',
    size=11
)

plt.show()
```

---

# Applications

This workflow can be adapted for:

- Species Distribution Modeling (SDM)
- Marine biodiversity studies
- Ecological niche modeling
- Conservation planning
- Spatial ecology
- Biogeographic analyses
- Environmental monitoring
- Biodiversity mapping
- Educational tutorials

---

# Author

**Juan Carlos Rubio-Polania**  
Doctor in Marine Sciences

---
