---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.4
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

---
title: "WSIM-GLDAS: Acquisition, Exploration, and Integration"
author: 
  - "Josh Brinks"
  - "Elaine Famutimi"
date: "June 3, 2024"
bibliography: "references/wsim-gldas-references.bib"
---

## Overview

In this lesson, you will acquire the dataset called **Water Security Indicator Model - Global Land Data Assimilation System (WSIM-GLDAS)** from the [NASA Socioeconomic Data and Applications Center (SEDAC)](https://sedac.ciesin.columbia.edu/) website, perform various pre-processing tasks, and explore the dataset with advanced visualizations. We will also integrate WSIM-GLDAS with a global administrative boundary dataset called [geoBoundaries](https://www.geoboundaries.org/api.html) and a population dataset called the [Gridded Population of the World](https://sedac.ciesin.columbia.edu/data/collection/gpw-v4). You will learn how to perform numerous data manipulations and visualizations throughout this lesson.

## Learning Objectives

After completing this lesson, you should be able to:

-   Retrieve WSIM-GLDAS data from SEDAC.
-   Retrieve Administrative Boundaries using the geoBoundaries API.
-   Subset WSIM-GLDAS data for a region and time period of interest.
-   Visualize geospatial data to highlight precipitation deficit patterns.
-   Write a pre-processed NetCDF-formatted file to disk.
-   Perform visual exploration with histograms, choropleths, and time series maps.
-   Integrate gridded population data with WSIM-GLDAS data to perform analyses and construct visualizations to understand how people are impacted.
-   Summarize WSIM-GLDAS and population raster data using zonal statistics.

## Introduction

The water cycle is the process of circulation of water on, above, and under the Earth's surface [@NOAA2019]. Human activities produce greenhouse gas emissions, land use changes, dam and reservoir development, and groundwater extraction have affected the natural water cycle in recent decades [@intergovernmentalpanelonclimatechange2023]. The influence of these human activities on the water cycle has consequential impacts on oceanic, groundwater, and land processes, influencing phenomena such as droughts and floods [@Zhou2016].

Precipitation deficits, or periods of below-average rainfall, can lead to drought, characterized by prolonged periods of little to no rainfall and resulting water shortages. Droughts often trigger environmental stresses and can create cycles of reinforcement impacting ecosystems and people [@Rodgers2023]. For example, California frequently experiences drought but the combination of prolonged dry spells and sustained high temperatures prevented the replenishment of cool fresh water to the Klamath River, which led to severe water shortages in 2003 and again from 2012 to 2014. These shortages affect agricultural areas like the Central Valley, which grows almonds, one of California's most important crops, with the state producing 80% of the world's almonds. These severe droughts, coupled with competition for limited freshwater resources, resulted in declining populations of [Chinook salmon](https://www.fisheries.noaa.gov/species/chinook-salmon) due to heat stress and gill rot disease disrupting the food supply for Klamath basin tribal groups [@guillen2002; @Bland2014].

![](docs/images/watercycle_rc.jpg)[^1]

[^1]: Photo Credit, NASA/JPL.

::: column-margin
::: {.callout-tip style="color: #5a7a2b;"}
## Data Science Review

A [raster](https://docs.qgis.org/2.18/en/docs/gentle_gis_introduction/raster_data.html) dataset is a type of geographic data in digital image format with numerical information stored in each pixel. (Rasters are often called grids because of their regularly-shaped matrix data structure.) Rasters can store many types of information and can have dimensions that include latitude, longitude, and time. NetCDF is one format for raster data; others include Geotiff, ASCII, and many more. Several raster formats like NetCDF can store multiple raster layers, or a "raster stack," which can be useful for storing and analyzing a series of rasters.
:::
:::

The **Water Security (WSIM-GLDAS) Monthly Grids, v1 (1948 - 2014)** The Water Security (WSIM-GLDAS) Monthly Grids, v1 (1948 - 2014) dataset "identifies and characterizes surpluses and deficits of freshwater, and the parameters determining these anomalies, at monthly intervals over the period January 1948 to December 2014" [@isciences2022]. The dataset can be downloaded from the [NASA SEDAC](https://sedac.ciesin.columbia.edu/data/set/water-wsim-gldas-v1) website. Downloads of the WSIM-GLDAS data are organized by a combination of thematic variables (composite surplus/deficit, temperature, PETmE, runoff, soil moisture, precipitation) and integration periods (a temporal aggregation) (1, 3, 6, 12 months). Each variable-integration combination consists of a NetCDF raster (.nc) file ( with a time dimension that contains a raster layer for each of the 804 months between January, 1948 and December, 2014. Some variables also contain multiple attributes each with their own time series. Hence, this is a large file that can take a lot of time to download and may cause computer memory issues on certain systems. This is considered BIG data.

::: callout-note
## Knowledge Check

1.  How would you best describe the water cycle?
    a.  A prolonged period of little to no rainfall.
    b.  Low precipitation combined with record temperatures.
    c.  The circulation of water on and above Earth's surface.
    d.  A cycle that happens due to drought.
2.  What human interventions affect the water cycle (select all that apply)?
    a.  Greenhouse gas emissions.
    b.  Land use changes.
    c.  Dam and reservoir development.
    d.  Groundwater over-exploitation.
3.  What is a precipitation deficit?
    a.  A period of rainfall below the average.
    b.  A prolonged period of little to no rainfall.
    c.  A period of chain reactions.
    d.  A period of rainfall above the average.
:::

## Acquiring the Data

::: {.callout-tip style="color: #5a7a2b;"}
## Data Science Review

The **Water Security (WSIM-GLDAS) Monthly Grids dataset** used in this lesson is hosted by [NASA's Socioeconomic Data and Applications Center (SEDAC](https://sedac.ciesin.columbia.edu/)), one of several [Distributed Active Archive Centers (DAACs)](https://www.earthdata.nasa.gov/eosdis/daacs). SEDAC hosts a variety of data products including geospatial population data, human settlements and infrastructure, exposure and vulnerability to climate change, and satellite-based land use, air, and water quality data. In order to download data hosted by SEDAC, you are required to have a free NASA EarthData account. You can create an account here: [NASA EarthData](https://urs.earthdata.nasa.gov/users/new).
:::

For this lesson, we will work with the **WSIM-GLDAS dataset Composite Anomaly Twelve-Month Return Period NetCDF** file. This contains water deficit, surplus, and combined "Composite Anomaly" variables with an integration period of 12 months. Integration period represents the time period at which the anomaly values are averaged over. The 12-month integration averages water deficits (droughts), surpluses (floods), and combined (presence of droughts and floods) over a 12-month period ending with the month specified. We begin with the 12-month aggregation because this is a snapshot of anomalies for the entire year making it useful to get an understanding of a whole year; once we identify time periods of interest in the data, we can take a closer look using the 3-month or 1-month integration periods.

When working on your local system you can directly download WSIM-GLDAS from the SEDAC website, however, we are working in the cloud so we will connect to the TOPS-SCHOOL S3 data bucket. The [dataset documentation](https://sedac.ciesin.columbia.edu/downloads/docs/water/water-wsim-gldas-v1-documentation.pdf) describes the composite variables as key features of WSIM-GLDAS that combine "the return periods of multiple water parameters into composite indices of overall water surpluses and deficits [@isciences2022a]". The composite anomaly files present the data in terms of how often they occur; or a "return period." For example, a deficit return period of 25 signifies a drought so severe that we would only expect it to happen once every 25 years. Please go ahead and download the file.

## Reading the Data

::: column-margin
::: {.callout-tip style="color: #5a7a2b;"}
## Data Science Review

This lesson uses the [`stars`](https://r-spatial.github.io/stars/), [`terra`](https://rspatial.org/pkg/), [`sf`](https://r-spatial.github.io/sf/), [`lubridate`](https://lubridate.tidyverse.org/), [`httr`](https://httr.r-lib.org/), [cubelyr](https://cran.r-project.org/web/packages/cubelyr/index.html), [`exactextractr`](https://isciences.gitlab.io/exactextractr/), [`ggplot2`](https://ggplot2.tidyverse.org/), [`data.table`](https://rdatatable.gitlab.io/data.table/), and [kableExtra](https://cran.r-project.org/web/packages/kableExtra/index.html) packages. If you'd like to learn more about the functions used in this lesson you can use the help guides on their package websites.

The `stars` and `terra` packages are designed to work with large and complex raster spatial data while the `sf` package lets you handle and analyze vector spatial data (more on that later). The `cubelyr` package helps you create and analyze multidimensional data cubes, making it easier to explore complex datasets and discover patterns. The `lubridate` package makes it easy to work with dates and times in R, and `httr` provides methods for users to interact with online databases and websites to directly download data. `exactextractr` performs fast and efficient zonal statistics that summarizes raster and vector data. `data.table` makes standard data processing much easier, and lastly, `ggplot2` and `kableExtra` are used for creating plots, maps, and tables.

Make sure they are installed before you begin working with the code in this document. If you'd like to learn more about the functions used in this lesson you can use the help guides on their package websites.
:::
:::

Next install and load the R packages required for this exercise. The packages we're using today are listed below, but we are using the TOPS-SCHOOL Docker image for our analysis so the cloud environment we're using today is already set up with everything we need.

```{r eval=FALSE}
install.packages('stars')
install.packages('terra')
install.packages('sf')
install.packages('cubelayer')
install.packages('lubridate')
install.packages('httr')
install.packages('data.table')
install.packages('exactextractr')
install.packages('ggplot2')
install.packages('kableExtra')
```

## Load Data & Attribute Selection

First we will connect to our TOPS-SCHOOL cloud storage "bucket" to download the WSIM-GLDAS file. If you were working on a local system you would use the SEDAC website to download the files, but that is not feasible in a live cloud workshop. Reading the file with the *argument* `proxy = TRUE` allows you to inspect the basic elements of the file without reading the whole file into memory. Multidimensional raster datasets can be extremely large and bring your computing environment to a halt if you have memory limitations.

```{r}
# read in the 12 month integration WSIM-GLDAS file in our s3 bucket
wsim_gldas <- stars::read_stars("/vsicurl/https://s3-us-west-2.amazonaws.com/tops-school/water-module/composite_12mo.nc", proxy = TRUE)

print(wsim_gldas)
```

Now we can use the *print* command to view basic information. The output tells us we have 5 attributes (deficit, deficit_cause, surplus, surplus_cause, both) and 3 dimensions. The first 2 dimensions are the spatial extents (x/y--longitude/latitude) and time is the 3rd dimension.

This means that the total number of individual raster layers in this NetCDF is 4020 (5 attributes x 804 time steps/months). Again, BIG data.

We can manage this large file by selecting a single variable; in this case "deficit" (drought). We'll read the data back in; this time with `proxy = FALSE` to read the data into actual memory, but only selecting the deficit layer with the `sub = 'deficit'` to make it more manageable. This takes approximately 45-60 seconds on 2i2c's platform.

```{r}
# read in just the deficit layer using proxy = false
wsim_gldas <- stars::read_stars("/vsicurl/https://s3-us-west-2.amazonaws.com/tops-school/water-module/composite_1mo.nc", proxy = FALSE, sub = 'deficit')
```

## Time Selection

Specifying a temporal range of interest will make the file size smaller and more manageable. We'll select every year for the range 2000-2014. This can be accomplished by generating a sequence for every year between December 2000 and December 2014, and then passing that list of dates, in the form of a vector, using the function `filter`. Remember, we're using the 12-month integration of WSIM-GLDAS. This means that each time step listed averages the deficit over the 12 prior months. Therefore, if we only select a sequence of December months spanning 2000-2014, each resulting layer will represent the average deficit for that year.

```{r}
# generate a vector of dates for subsetting
keeps<-seq(lubridate::ymd("2000-12-01"),
           lubridate::ymd("2014-12-01"), 
           by = "year")
#change data type to POSIXct
keeps <- as.POSIXct(keeps)
# filter the dataset using the vector of dates
wsim_gldas <- dplyr::filter(wsim_gldas, time %in% keeps)
# re-check the basic info
print(wsim_gldas)
```

Now we're down to a single attribute ("deficit") with 15 time steps. We can take a look at the first 6 years by passing the object through `slice` and then into `plot`.

```{r warning=FALSE}
wsim_gldas |>
  # slice out the first 6 time steps
  dplyr::slice(index = 1:6, along = "time") |>
  # plot them with some test breaks for the legend
  plot(key.pos = 1, breaks = c(0, -5, -10, -20, -30, -50), key.lab = "Deficit")
```

Although we have now reduced the data to a single attribute with a restricted time of interest, we can take it a step further and limit the spatial extent to a country or state of interest.

::: callout-note
## Knowledge Check

4.  Which of these best describe a raster dataset?
    a.  A type of geographic data made up of square cells.
    b.  A table or list of geographic numbers.
    c.  A geographic region of interest.
    d.  A type of geographic data made up of lines and points.
5.  Which of the following is true about the information that rasters can store? (select all that apply)
    a.  Attributes (thematic content)
    b.  Dimensions (information expressing spatial or temporal extent information)
    c.  Geographic coordinates
    d.  A list of numbers
:::

## Spatial Selection

::: column-margin
::: {.callout-tip style="color: #5a7a2b;"}
## Data Science Review

[GeoJSON](https://geojson.org/) is a format for encoding, storing and sharing geographic data as vector data, i.e., points, lines and polygons. It's commonly used in web mapping applications to represent geographic features and their associated attributes. GeoJSON files are easily readable by both humans and computers, making them a popular choice for sharing geographic data over the internet.
:::
:::

We can spatially crop the raster stack with a few different methods. Options include using a bounding box in which the outer geographic coordinates are specified (xmin, ymin, xmax, ymax), using another raster object, or using a vector boundary like a shapefile or GeoJSON to crop the extent of the original raster data.

In this example we use a vector boundary to accomplish the geoprocessing task of cropping the data to an administrative or political unit. First, we acquire the data in GeoJSON format for the United States from the geoBoundaries API. (Note it is also possible to download the vectorized boundaries directly from <https://www.geoboundaries.org/> in lieu of using the API).

To use the geoBoundaries' API, the root URL below is modified to include a 3 letter code from the International Standards Organization used to identify countries (ISO3), and an administrative level for the data request. Administrative levels correspond to geographic units such as the Country (administrative level 0), the State/Province (administrative level 1), the County/District (administrative level 2), and so on:

*https://www.geoboundaries.org/api/current/gbOpen/**ISO3**/**LEVEL**/*

::: column-margin
::: {.callout-tip style="color: #5a7a2b;"}
## Data Science Review

Built by the community and [William & Mary geoLab](https://github.com/wmgeolab), the geoBoundaries Global Database of Political Administrative Boundaries is an online, open license (CC BY 4.0 / ODbL) resource for administrative boundaries (i.e., state, county, province) for every country in the world. Since 2016, this project has tracked approximately 1 million spatial units within more than 200 entities; including all UN member states.
:::
:::

For this example we adjust the bolded components of the sample URL address below to specify the country we want using the ISO3 Character Country Code for the United States (**USA**) and the desired Administrative Level of State (**ADM1**).

```{r}
# make the request to geoboundarie's website for the USA boundaries
usa <- httr::GET("https://www.geoboundaries.org/api/current/gbOpen/USA/ADM1/")
```

In the line of code above, we used `httr:GET` to obtain metadata from the URL. We assign the result to a new variable called "usa". Next we will examine the `content`.

```{r}
# parse the content into a readable format
usa <- httr::content(usa)
# look at the labels for available information
names(usa)
```

The parsed content contains 32 components. Item 29 is a direct link to the GeoJSON file (gjDownloadURL) where the vector boundary data is located. Next we will obtain the GeoJSon and check the results.

```{r}
# directly read in the geojson with sf from the geoboundaries server
usa <- sf::st_read(usa$gjDownloadURL)
# check the visuals
plot(sf::st_geometry(usa))
```

Upon examination, we see that this GeoJSon includes all US states and overseas territories. For this demonstration, we can simplify it to the contiguous United States. (Of course, it could also be simplified to other areas of interest simply by adapting the code below.)

We first create a list of the geographies we wish to remove and assign them to a variable called "drops". Next, we reassign our "usa" variable to include only the entries in the continental US and finally we plot the results.

```{r}
# create a vector of territories we don't want in our CONUSA boundary
drops<-
  c("Alaska", "Hawaii", 
    "American Samoa",
    "Puerto Rico",
    "Commonwealth of the Northern Mariana Islands", 
    "Guam", 
    "United States Virgin Islands")

# select all the states and territories not in the above list
# in R the "!" signifies NOT or the OPPOSITE
usa<-usa[!(usa$shapeName %in% drops),]
# check the visuals
plot(sf::st_geometry(usa))
```

We can take this a step further and select a single state for analysis. Here we use a slightly different method by creating a new object called "texas" by subsetting the state out by name.

```{r}
# extract just texas from the CONUSA boundary
texas <- usa[usa$shapeName == "Texas",]
# check the visuals
plot(sf::st_geometry(texas))
```

From here we can clip the WSIM-GLDAS raster stack by using the stored boundary of Texas. You can call the `sf::st_crop()` function to crop the WSIM-GLDAS layer, but as you see below, more simply, you can just use bracket indexing to crop a **stars** object with a **sf** object.

::: {.callout-tip style="color: #5a7a2b;"}
## Drought in the News

Texas experienced a severe drought in 2011 that caused rivers to dry up and lakes to reach historic low levels [@StateImpact]. The drought was further exacerbated by high temperatures related to climate change in February of 2013. Climate experts discovered that the drought was produced by "La Niña", a weather pattern that causes the surface temperature of the Pacific Ocean to be cooler than normal. This, in turn, creates drier and warmer weather in the southern United States. La Niña can occur for a year or more, and returns once every few years [@NOAA2023].

It is estimated that the drought cost farmers and ranchers about \$8 billion in losses.[@Roeseler2011] Furthermore, the dry conditions fueled a series of wildfires across the state in early September of 2011, the most devastating of which occurred in Bastrop County, where 34,000 acres and 1,300 homes were destroyed [@Roeseler2011].
:::

```{r warning = FALSE}
# crop the wsim-gldas file to the extent of te texas boundary
wsim_gldas_tex <- wsim_gldas[texas]
```

Finally, we visualize the last time-step in the WSIM-GLDAS dataset (15/December, 2014) and render it with an overlay of the Texas boundary to perform a visual check of our processing.

```{r warning = FALSE}
# slive out the first 15 timesteps of wsim-gldas and plot them
wsim_gldas_tex |>
  dplyr::slice(index = 15, along = "time") |>
  # plot with 
  plot(reset = FALSE, breaks = c(0, -5, -10, -20, -30, -50))
# add the texas boundary on top
plot(sf::st_geometry(texas),
     add = TRUE,
     lwd = 3,
     fill = NA,
     border = 'purple')
```

Voila! We successfully cropped the WSIM-GLDAS to the extent of Texas and created an overlay map with both dat sets to check the results. If you were carrying out further analysis or wanted to share your work with colleagues, you may want to save the processed WSIM-GLDAS to disk.

Multidimensional (deficit, time, latitude, longitude) raster files can be saved with the `stars::write_mdim` function and vector data can be saved using `sf::st_write`.

```{r eval = FALSE}
# write the processed wsim-gldas file to disk as a netcdf
stars::write_mdim(wsim_gldas_tex, "output/wsim_gldas_tex.nc")
# write the Texas boundary to disk
sf::st_write(texas, "output/texas.geojson")
```

The size of the pre-processed dataset is 1.6 MB compared to the original dataset of 1.7 GB. This is much more manageable in cloud environments, workshops, and git platforms. T

## Advanced Visualizations and Data Integrations

Now that we've introduced the basics of manipulating and visualizing WSIM-GLDAS, we can explore more advanced visualizations and data integrations. Let's clear the workspace and start over again with the same **WSIM-GLDAS Composite Anomaly Twelve-Month Return Period** we used earlier. We will spatially subset the data to cover only the Continental United States (CONUSA) which will help to minimize our memory footprint. We can further reduce our memory overhead by reading in just the variable we want to analyze. In this instance we can read in just the `deficit` attribute from the WSIM-GLDAS Composite Anomaly Twelve-Month Return Period file, rather than reading the entire NetCDF with all of its attributes.

::: column-margin
::: {.callout-tip style="color: #5a7a2b;"}
## Coding Review

Random Access Memory (RAM) is where data and programs that are currently in use are stored temporarily. It allows the computer to quickly access data, making everything you do on the computer faster and more efficient. Unlike the hard drive, which stores data permanently, RAM loses all its data when the computer is turned off. RAM is like a computer's short-term memory, helping it to handle multiple tasks at once.
:::
:::

For this exercise, we can quickly walk through similar pre-processing steps we performed earlier in this lesson and then move on to more advanced visualizations and integrations. Read the original 12-month integration data back in, filter with a list of dates for each December spanning 2000-2014, and then crop the raster data with the boundary of the contiguous United States using our geoBoundaries object.

```{r warning=FALSE}
# read in the wsim-gldas layer from SCHOOL s3 bucket
wsim_gldas <- stars::read_stars("/vsicurl/https://s3-us-west-2.amazonaws.com/tops-school/water-module/composite_12mo.nc", proxy = FALSE, sub = 'deficit')

# generate a vector of dates for subsetting
keeps<-seq(lubridate::ymd("2000-12-01"),
           lubridate::ymd("2014-12-01"), 
           by = "year")

#change data type to POSIXct
keeps <- as.POSIXct(keeps)
# filter using that vector
wsim_gldas <- dplyr::filter(wsim_gldas, time %in% keeps)
# crop to conusa
wsim_gldas<-wsim_gldas[usa]
#rename the time dimension to just the years each time-step represents
wsim_gldas <-
  wsim_gldas |> stars::st_set_dimensions("time", values = as.character(seq(2000,2014)))
```

Double check the object information.

```{r}
# check the basic information again
print(wsim_gldas)
```

You will want to review the printout to make sure it looks okay.

-   Does it contain the variables you were expecting?

-   Do the values for the variables seem plausible?

Other basic descriptive analyses are useful to verify and understand your data. One of these is to produce a frequency distribution (also known as a histogram), which is reviewed below.

::: callout-note
## Knowledge Check

6.  What are the two ways we reduced our memory footprint when we loaded the WSIM-GLDAS dataset for this lesson? (select all that apply)
    a.  Spatially subsetting the data to the CONUSA
    b.  Reading in only one year of data
    c.  Reading in only the attribute of interest (i.e., 'deficit')
    d.  All of the above.
:::

## Annual CONUSA Time Series

The basic data properties reviewed in the previous step are useful for exploratory data analysis, but we should perform further inspection. We can start our visual exploration of annual drought in the CONUSA by creating a map illustrating the deficit return period for each of the years in the WSIM-GLDAS object.

```{r warning = FALSE, message = FALSE}
# load the base data of the usa boundary
ggplot2::ggplot(usa)+
  # plot the stars wsim object
  stars::geom_stars(data = wsim_gldas)+
  # set equal coordinates for axes
  ggplot2::coord_equal()+
  # create multiple panels for each time step
  ggplot2::facet_wrap(~time)+
  # plot the usa boundary with just the outline
  ggplot2::geom_sf(fill = NA)+
  # add the palette for wsim-gldas 
  ggplot2::scale_fill_stepsn(
    colors = c(
    '#9B0039',
    # -50 to -40
    '#D44135',
    # -40 to -20
    '#FF8D43',
    # -20 to -10
    '#FFC754',
    # -10 to -5
    '#FFEDA3',
    # -5 to -3
    '#FFF4C7',
    # -3 to 0
    '#FFFFFF'), 
    # set the breaks on the palette
    breaks = c(-60, -50, -40, -20, -10,-5,-3, 0))+
  # add plot labels
  ggplot2::labs(
    title="Annual Mean Deficit Anomalies for the CONUSA",
    subtitle = "Using Observed 12 Month Integrated WSIM-GLDAS Data for 2000-2014",
    fill = "Deficit Return Period"
  )+
  # set the minimal theme
  ggplot2::theme_minimal()+
  # turn off some extra graphical elements
  ggplot2::theme(
    axis.title.x=ggplot2::element_blank(),
    axis.text.x=ggplot2::element_blank(),
    axis.ticks.x=ggplot2::element_blank(),
    axis.title.y=ggplot2::element_blank(),
    axis.text.y=ggplot2::element_blank(),
    axis.ticks.y=ggplot2::element_blank())
```

This visualization shows that there were several significant drought events (as indicated by dark red deficit return-period values) throughout 2000-2014. Significant drought events included the southeast in 2000, the southwest in 2002, the majority of the western 3rd in 2007, Texas-Oklahoma in 2011, Montana-Wyoming-Colorado in 2012, and the entirety of the California coast in 2014. The droughts of 2012 and 2011 are particularly severe and widespread with return periods greater than 50 years covering multiple states. Based on historical norms, we should only expect droughts this strong every 50-60 years!

::: callout-note
## Knowledge Check

7.  The output maps of Annual Mean Deficit Anomalies for the CONUSA indicate that...
    a.  The most significant deficit in 2004 is located in the western United States.
    b.  The least significant deficit 2003 is located in the Midwest.
    c.  The most significant deficit in 2011 is located around Texas and neighboring states.
    d.  The least significant deficit in 2000 is located in the southeast.
:::

## Monthly Time Series

We can get a more detailed look at these drought events by using the 1-month composite WSIM-GLDAS dataset and cropping the data to a smaller spatial extent matching one of the events we've noted in the previous plot. Let's take a closer look at the 2014 California drought.

::: {.callout-tip style="color: #5a7a2b;"}
## Drought in the News

The California drought of 2012-2014 was the worst in 1,200 years [@WHOI2014]. This drought caused problems for homeowners, and even conflicts between farmers and wild salmon! Governor Jerry Brown declared a drought emergency and called on residents to reduce water intake by 20%. Water use went up by 8% in May of 2014 compared to 2013, in places like coastal California and Los Angeles. Due to the water shortages, the state voted to fine water-wasters up to \$500 dollars. The drought also affected residents differently based on economic status. For example, in El Dorado County, located in a rural area east of Sacramento, residents were taking bucket showers and rural residents reported wells, which they rely on for fresh water, were drying up. The federal government eventually announced a \$9.7 million emergency drought aid for those areas [@Sanders2014].

Additionally, there were thousands of adult salmon struggling to survive in the Klamath River in Northern California, where water was running low and warm due to the diversion of river flow into the Central Valley, an agricultural area that grows almond trees. Almonds are one of California's most important crops, with the state producing 80% of the world's almonds. However, salmon, which migrate upstream, could get a disease called gill rot, which flourishes in warm water and already killed tens of thousands of Chinook in 2002. This disease was spreading through the salmon population again due to this water allocation, affecting local Native American tribes that rely on the fish [@Bland2014].
:::

In order to limit the amount of computing memory required for the operation, we will first clear items from the in-memory workspace and then reload a smaller composite file, we'll start by removing the 12-month composite object.

```{r}
# remove the large wsim object from the environment
rm(wsim_gldas)
rm(wsim_gldas_tex)

# clear the memory; R has an automated garbage collector to clear unused memory
# but you can invoke it by using the gc() command
gc()
```

Again we'll connect to the SCHOOL cloud bucket and read in the 1 month integration file. The structure is the same as the 12-month integration so we will go ahead and load in just the deficit layer and check the object with `print`.

```{r}
# connect to bucket and download 1-month integration
# read in the wsim-gldas layer from SCHOOL s3 bucket
wsim_gldas_1mo <- stars::read_stars("/vsicurl/https://s3-us-west-2.amazonaws.com/tops-school/water-module/composite_1mo.nc", proxy = FALSE, sub = 'deficit')

print(wsim_gldas_1mo)
```

Once again, we'll subset the time dimension for our period of interest. However, this time we want every month for 2014 so we can take a closer look at the California drought.

```{r}
# generate a vector of dates for subsetting
keeps<-seq(lubridate::ymd("2014-01-01"),
           lubridate::ymd("2014-12-01"), 
           by = "month")
#change data type to POSIXct
keeps <- as.POSIXct(keeps)
# filter using that vector
wsim_gldas_1mo <- dplyr::filter(wsim_gldas_1mo, time %in% keeps)
# check the info
print(wsim_gldas_1mo)
```

Now we have 12 rasters with monthly data for 2014. Let's zoom in on California and see how this drought progressed over the course of the year.

```{r warning = FALSE, message = FALSE}
# isolate only the california border
california<-usa[usa$shapeName=="California",]
# subset wsim-gldas to the extent of the california boundary
wsim_gldas_california <- wsim_gldas_1mo[california]
# give the time dimension pretty labels
wsim_gldas_california <-
  wsim_gldas_california |>
    stars::st_set_dimensions("time", values = month.name)

# monthly plots of california
# load the base california data
ggplot2::ggplot(california)+
  # load the wsim stars object
  stars::geom_stars(data = wsim_gldas_california)+
  # equal aspect ratio for lat/long axe
  ggplot2::coord_equal()+
  # create a subplot for each time step
  ggplot2::facet_wrap(~time)+
  # only plot the outline of california
  ggplot2::geom_sf(fill = NA)+
  # set the wsim palette
  ggplot2::scale_fill_stepsn(
    colors = c(
    '#9B0039',
    # -50 to -40
    '#D44135',
    # -40 to -20
    '#FF8D43',
    # -20 to -10
    '#FFC754',
    # -10 to -5
    '#FFEDA3',
    # -5 to -3
    '#FFF4C7',
    # -3 to 0
    '#FFFFFF'), 
    # set the wsim breaks
    breaks = c(-60, -50, -40, -20, -10,-5,-3, 0))+
  # add labels
  ggplot2::labs(
    title="Deficit Anomalies for California",
    subtitle = "Using Observed Monthly WSIM-GLDAS Data for 2014",
    fill = "Deficit Return Period"
  )+
  # start with a base minimal theme
  ggplot2::theme_minimal()+
  # remove additional elements to make a cleaner map
  ggplot2::theme(
    axis.title.x=ggplot2::element_blank(),
    axis.text.x=ggplot2::element_blank(),
    axis.ticks.x=ggplot2::element_blank(),
    axis.title.y=ggplot2::element_blank(),
    axis.text.y=ggplot2::element_blank(),
    axis.ticks.y=ggplot2::element_blank())
```

This series of maps shows a startling picture. California faced massive water deficits throughout the state in January and February. This was followed by water deficits in the western half of the state in May-August. Although northern and eastern California saw some relief by September, southwest California continued to see deficits through December.

## Monthly Histograms

::: column-margin
::: {.callout-tip style="color: #5a7a2b;"}
## Data Science Review

A [data frame](https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/data.frame) is a data structure used for storing tabular data. It organizes data in rows and columns. Each column can have a different type of data (numeric, character, factor, etc.), and rows represent individual observations or cases. Data frames provide a convenient way to work with structured data, making them essential for data analysis and statistics projects.
:::
:::

We can explore the data further by creating a frequency distribution (also called a histogram) of the deficit anomalies for any given spatial extent; here we are still looking at the deficit anomalies in California. We start by extracting the data from the raster time series and then created a data frame of values that are easier to manipulate into a histogram. [R data frames](https://www.w3schools.com/r/r_data_frames.asp) are data displayed in table format, which can be plotted on graphs or charts.

```{r}
# extract the raster values into a dataframe
deficit_hist <-  
  wsim_gldas_california |>
  as.data.frame(wsim_gldas_california$deficit)

# remove the NA values
deficit_hist<-stats::na.omit(deficit_hist)

# create the histogram
ggplot2::ggplot(deficit_hist, ggplot2::aes(deficit))+
  # style the bars
  ggplot2::geom_histogram(binwidth = 6, fill = "#325d88")+
  # subplot for each timestep
  ggplot2::facet_wrap(~time)+
  # use the minimal theme
  ggplot2::theme_minimal()
```

The histograms start to quantify what we saw in the time series maps. Whereas the map shows where the deficits occur, the frequency distribution indicates the number of raster cells for each return period of the deficit range. The number of raster cells under a 60-year deficit (return period) is very high in most months, far exceeding any other value in the range.

::: callout-note
## Knowledge Check

8.  What's another term for 'histogram'?
    a.  Choropleth
    b.  Frequency Distribution
    c.  Data array
    d.  Box plot
:::

## Zonal Summaries

To this point we've described the 2014 California drought by examining the state as a whole. Although we have a sense of what's happening in different cities or counties by looking at the maps, they do not provide quantitative summaries of local areas.

Zonal statistics are one way to summarize the cells of a raster layer that lie within the boundary of another data layer (which may be in either raster or vector format). For example, aggregating deficit return periods with another raster depicting land cover type or a vector boundary (shapefile) of countries, states, or counties, will produce descriptive statistics by that new layer. These statistics could include the sum, mean, median, standard deviation, and range.

For this section, we begin by calculating the mean deficit return period within California counties. First, we retrieve a vector dataset of California counties from the geoBoundaries API. Since geoBoundaries does not attribute which counties belong to which states, we utilize a spatial operation called intersect to select only those counties in California.

```{r}
# get the adm2 (county) level data for the USA
# request the adm2 usa information
cali_counties <- httr::GET("https://www.geoboundaries.org/api/current/gbOpen/USA/ADM2/")
# parse the content to find the direct download link
cali_counties <- httr::content(cali_counties)
# read in the geojson directly from geoboundaries
cali_counties <- sf::st_read(cali_counties$gjDownloadURL)

# geoBoundaries does not list which counties belong to which state so you need to run an intersection
# intersect the usa adm2 data with just the california boundary
cali_counties<-sf::st_intersection(cali_counties, california)
# plot the result
plot(sf::st_geometry(cali_counties))
```

The output of that intersection looks as expected. As noted above, in general a visual and/or tabular check on your data layers is always a good idea. If you expect 50 counties in a given state, you should see 50 counties resulting from your intersection of your two layers, etc. You may want to be on the look out for too few (such as an island area that may be in one layer but not the other) or too many counties (such as those that intersect with a neighboring state).

::: column-margin
::: {.callout-tip style="color: #5a7a2b;"}
## Coding Review

The [exactextractr](https://github.com/isciences/exactextractr) [@Baston2023] R package summarizes raster values over groupings, or zones, also known as zonal statistics. Zonal statistics help in assessing the statistical characteristics of a certain region.

The [terra](https://cran.r-project.org/web/packages/terra/index.html) R package processes raster geospatial data, offering functionalities such as data manipulation, spatial analysis, modeling, and visualization, with a focus on efficiency and scalability.
:::
:::

We will perform our zonal statistics using the `exactextractr` package [@Baston2023]. It is the fastest, most accurate, and most flexible zonal statistics tool for the R programming language, but it currently has no default methods for the **stars** package and stars objects, so we'll switch to `terra` for this portion of the lesson. You'll notice that there are slight differences in syntax to perform the same operations in **terra** as **stars.**

Read the data back in with **terra**.

```{r}
# remove the previous 1 month object
rm(wsim_gldas_1mo)

# create a terra collection with the wsim-gldas 1 month file from SEDAC
wsim_gldas_1mo<-terra::sds("/vsicurl/https://s3-us-west-2.amazonaws.com/tops-school/water-module/composite_1mo.nc")
```

We'll check the basic info with `print`.

```{r}
print(wsim_gldas_1mo)
```

You can see that the information from **terra** is arranged differently but it's all still there. The attributes are numbered in the "subdatasets" (5) and listed by name in the "names" column. The time dimension is not listed by name, but "nlyr" tells us there are 804 layers for each attribute/subdataset. You can access more detailed information for the time dimensions using `terra::time`.

Subset to just deficit and then again along the time dimension.

```{r}
# pull out just the deficit layer
 wsim_gldas_1mo<-wsim_gldas_1mo["deficit"]
# create a sequence of dates to keep
keeps<-seq(lubridate::ymd("2014-01-01"), lubridate::ymd("2014-12-01"), by = "month")
# subset the terra object for only those time steps
wsim_gldas_1mo<-wsim_gldas_1mo[[terra::time(wsim_gldas_1mo) %in% keeps]]
# label the time steps
names(wsim_gldas_1mo) <- keeps
# check the info
print(wsim_gldas_1mo)
```

Now that we've selected just the "deficit" variable **terra** lists the "varname" as deficit and shows t he time-steps in the "names" column. A little confusing but **terra** changes the display once the object goes from multiple variables to just one.

Time to perform the zonal summary.

```{r warning=FALSE, message=FALSE}
# run the extraction
cali_county_summaries<-
  exactextractr::exact_extract(
    # the raster data
    wsim_gldas_1mo, 
    # the boundary data
    cali_counties, 
    # the calculation
    'mean', 
    # don't show progress
    progress = FALSE)
# give the timestep prettier labels
names(cali_county_summaries)<-lubridate::month(keeps, label = TRUE, abbr = FALSE)
```

`exactextractr` will return summary statistics in the same order of the input boundary file, therefore we can join the California county names to the **exactextract** summary statistics output for visualization. We will also make a version to view as a table to inspect the raw data. We can take a quick look at the first 10 counties to see their mean deficit return period for January-June.

```{r}
# bind the extracted means with the california boundary
cali_counties<-cbind(cali_counties, cali_county_summaries)
# prepare a version to create a table
cali_county_table<-cbind(County=cali_counties$shapeName,
                         round(cali_county_summaries))
# create the table with only the first 10 rows and 7 columns
kableExtra::kbl(cali_county_table[c(1:10),c(1:7)]) |>
    kableExtra::kable_styling(
      bootstrap_options = c("striped", "hover", "condensed"))
```

This confirms the widespread distribution of high deficit values (all the bright red) in our exploratory maps. The data is currently in wide format, which makes for easy viewing of a time series, but more advanced programmatic visualization typically requires data to be in a long format (more on that later).

::: callout-note
## Knowledge Check

9.  What can zonal statistics be used for?
    a.  Subsetting data to a time period of interest.
    b.  Creating a time series.
    c.  Summarizing the values of cells that lie within a boundary.
    d.  All of the above.
:::

## County Choropleths

Now that we've inspected the raw data we can make a choropleth out of the mean deficit return period data. We previously demonstrated more complex maps using **ggplot2**, **sf**, and **stars**, but you can also make quick plots of **sf** objects with the base plotting function. By default **sf** will make a map for every column listed in the dataset. In this case we only want to look at the monthly means so we will just plot columns 11 through 23. You can make simple alterations to the color palette and position of the legend, but custom map titles and legend titles are not easily accomplished with multi-panel maps (multiple maps in one) like the one pictured below.

```{r warning=FALSE, message=FALSE}
# plot the data for a check using only the 12 monthly summaries in columns 11 through 23
plot(cali_counties[c(11:23)],
     # the palette we've been using
     pal = c(
    '#9B0039',
    # -50 to -40
    '#D44135',
    # -40 to -20
    '#FF8D43',
    # -20 to -10
    '#FFC754',
    # -10 to -5
    '#FFEDA3',
    # -5 to -3
    '#FFF4C7',
    # -3 to 0
    '#FFFFFF'), 
    # the breaks we want
    breaks = c(-61, -50, -40, -20, -10,-5,-3, 5), 
    # where we're placing the legend
    key.pos = 1, 
    # how thick we want the legend to be
    key.width = 0.3)
```

Due to the widespread water deficits in the raw data, the mean values do not appear much different from the raw deficit raster layer, however, choropleth maps, also called thematic maps, can make it easier for users to survey the landscape by visualizing familiar places (like counties) that place themselves and their lived experiences alongside the data.

While this paints a striking picture of widespread water deficits, how many people are affected by this drought? Although the land area appears rather large, if one is not familiar with the distribution of population and urban centers in California it can be difficult to get a sense of the direct human impact. (This is partly because more populous locations are usually represented by smaller land areas and the less populous locations are usually represented by large administrative boundaries containing much more land area). Normalizing a given theme by land area may be something an analyst wants to do but we cover another approach below.

::: callout-note
## Knowledge Check

10. Choropleth maps, aka thematic maps, are useful for
    a.  Visualizing data in familiar geographies like counties or states
    b.  Finding directions from one place to another
    c.  Visualizing data in uniform geographic units like raster grid cells.
    d.  Calculating return periods.
:::

## Integrating Population Data

**Gridded Population of the World (GPW)** is a data collection from SEDAC that models the distribution of the global human population as counts and densities in a raster format [@CIESIN2018].We will take full advantage of **exactextractr** to integrate across WSIM-GLDAS, geoBoundaries, and GPW. To begin, we need to download the 15-minute resolution (roughly 30 square kilometer at the equator) population density data for the year 2015 from GPW. This version of GPW most closely matches our time period (2014) and the resolution of WSIM-GLDAS (0.25 degrees). Although in many applications one might choose to use GPW's population count data layers, because we are using **exactextractr** we can achieve more accurate results (especially along coastlines) by using population density in conjunction with land area estimates from the **exactextractr** package.

::: {.callout-tip style="color: #5a7a2b;"}
## Data Review

The Gridded Population of the World Version 4 is available in multiple target metrics (e.g. counts, density), periods (2000, 2005, 2010, 2015, 2020), and spatial resolutions (30 sec, 2.5 min, 15 min, 30 min, 60 min). Read more about GPW at the [collection home page on SEDAC](https://sedac.ciesin.columbia.edu/data/collection/gpw-v4). GPW is one of four global gridded population datasets available in raster format. These datasets vary in the degree to which they use additional information as ancillary variables to model the spatial distribution of the population from the administrative units (vector polygons) in which they originate. A review of these datasets and their underlying models is found in a paper by Leyk and colleagues [@leyk2019]. You can learn more about gridded population datasets at POPGRID.org. Fitness-for-use is an important principle in determining the best dataset to use for a specific analysis. The question we ask here is --- what is the population exposure to different levels of water deficit in California? --- uses spatially coarse inputs and is for a place with high-quality data inputs, GPW is a good choice for this analysis. Users with vector-format census data (at the county or sub-county level) could also adopt this approach for those data. In the case of California, the US Census data and GPW will produce nearly identical estimates because GPW is based on the census inputs.
:::

Load in the GPW tif from the SCHOOL s3 cloud storage.

```{r}
# read the file with terra from our s3 bucket
pop_dens<- 
terra::rast("/vsicurl/https://s3-us-west-2.amazonaws.com/tops-school/water-module/gpw_v4_population_density_rev11_2015_15_min.tif")

# check the basic information
print(pop_dens)
```

For this example we'll classify the WSIM-GLDAS deficit return period raster layer into eight categories. Binning the data will make it easier to manage the output and interpret the results.

```{r}
# set the class breaks row-wise (from, to, new label)
# e.g. first row states: "all return periods from 0 to 5 will now be labeled 0
m <- c(0, 5, 0,
       -3, 0, -3,
       -5, -3, -5,
       -10, -5, -10,
       -20, -10, -20,
       -40, -20, -40,
       -50, -40, -50,
       -65, -50, -60)
# convert the vector into a matrix
rclmat <- matrix(m, ncol=3, byrow=TRUE)
# classify the data
wsim_gldas_1mo_class <-
  terra::classify(wsim_gldas_1mo, rclmat, include.lowest = TRUE)
```

In our previous example, we used **exactextractr**'s built-in `'mean'` function, but we can also pass custom functions to **exactextractr** that will carry out several operations at once as well. The following code could be combined into a single function passed to **exactextractr**, but it is presented here as multiple functions in order to follow along more easily. You can read more about **exactextractr** arguments in the package [help guide](https://cran.r-project.org/web/packages/exactextractr/exactextractr.pdf). The key arguments to be aware of are the calls to:

1.  `weights = pop_dens`: summarizes each WSIM-GLDAS cell's deficit return period with the corresponding population density value.
2.  `coverage_area = TRUE`: calculates the corresponding area of the WSIM-GLDAS raster cell that is covered by the California boundary.

```{r}
# run the extraction
pop_by_rp <-
  exactextractr::exact_extract(wsim_gldas_1mo_class, california, function(df) {
    df <- data.table::setDT(df)}, 
  # convert output to data frame
  summarize_df = TRUE, 
  # specify the weights we're using
  weights = pop_dens, 
  # return the coverage area (m^2)
  coverage_area = TRUE,
  # return the county name with the output data
  include_cols = 'shapeISO', 
  # don't show progress
  progress = FALSE)
```

This returns a `data.frame` with a row for every raster cell in the WSIM-GLDAS layer that is overlapped by the California boundary. Let's take a look at the first 6 rows.

```{r}
head(pop_by_rp)
```

-   `shapeISO`: The label of the polygon boundary where the cell is located. This was passed on from the California geojson boundary as specified in the `include_cols = 'shapeISO'` argument. In this instance, it's not very helpful because we used the state-level California boundary, but if we passed the ADM2 boundary with counties it would provide the name of the county where the cell is located.
-   `2014-01-01` to `2014-12-01`: The next 12 columns list the deficit return period classification value for the cell in each of the 12 months corresponding to the time dimension of the `wsim_gldas_1mo` raster layer.
-   `weight`: The `weight` column lists the corresponding population density value (persons per km\^2) for that WSIM-GLDAS cell. The WSIM-GLDAS and GPW raster layers have the same projection and resolution. Therefore, because they are perfectly aligned, each WSIM return period cell has a corresponding GPW population weight that can be layered exactly on it.
-   `coverage_area`: The total area (m\^2) of the WSIM-GLDAS cell that is covered by the California boundary layer. Given the total area of the WSIM cell that is covered, and the GPW persons per unit area weight, we can calculate the number of people estimated to be living within this cell under this WSIM deficit return period.

We will need to perform a few more processing steps to prepare this `data.frame` for a time series visualization integrating all of the data. We will use the `melt` function to transform the data from wide format to long format in order to produce a visualization in `ggplot2`. Specifically, we need to use `melt` to make the 12 month columns (`2014-01-01` to `2014-12-01`) into 2 new columns: 1) specifying the WSIM-GLDAS deficit return period value and 2) the month it came from.

::: column-margin
::: {.callout-tip style="color: #5a7a2b;"}
## Coding Review

Converting data from wide to long or long to wide formats is a key component to data processing, however, it can be confusing. To read more about melting/pivoting longer (wide to long) and casting/pivoting wider (long to wide) check out the *data.table* vignette [Efficient reshaping using data.tables](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-reshape.html) and the *dplyr* [`pivot_longer`](https://tidyr.tidyverse.org/reference/pivot_longer.html) and [`pivot_wider`](https://tidyr.tidyverse.org/reference/pivot_wider.html) reference pages.
:::
:::

```{r}
# convert the dataset from wide to long (melt)
pop_by_rp <-
  data.table::melt(
    # data we're melting
    pop_by_rp,
    # the columns that need to stay columns/ids
    id.vars = c("shapeISO", "coverage_area", "weight"),
    # name for the month columns we're melting into a single column
    variable.name = "month",
    # name for the values being converted into a single column
    value.name = "return_period")

# check the first 5 rows of melted data
head(pop_by_rp)
```

Each row lists the land area (coverage_area) covered by the zone, the population density value (weight) for the zone, the month the deficit return period corresponds to, and the actual deficit return period class for the zone.

Next, we'll summarize the data by return-period class.

```{r}
# create new column with population totals for each month and return period combination
# divide by 1000000
pop_by_rp <-
  pop_by_rp[, .(pop_rp = round(sum(coverage_area * weight) / 1e6)), by = .(month, return_period)]
# some cells do not have deficit return period values and result in NaN--just remove
pop_by_rp <- na.omit(pop_by_rp)
# create a new column with the total population for that month (will be the same for each month but needed to create wsim class fraction)
pop_by_rp[, total_pop := sum(pop_rp), by = month]
# check the first rows
head(pop_by_rp)
```

Now we have a row for every unique combination of month, return period class, and the total population. We can calculate the percent of the total population represented by this return period class with 2 more lines.

```{r}
# calculate the fraction of that month-wsim class combination relative to the total population
pop_by_rp[, pop_frac := pop_rp / total_pop][, total_pop := NULL]
# check the first rows
head(pop_by_rp)
```

Before plotting we'll make the month labels more legible for plotting, convert the WSIM-GLDAS return period class into a factor, and set the WSIM-GLDAS class palette.

::: column-margin
::: {.callout-tip style="color: #5a7a2b;"}
## Coding Review

Factors are the most common way to handle categorical data in R. Although converting your categorical variables into factors is not not always the best choice, in many instances (especially plotting with *ggplot2*) the benefits will out way any annoyances. To learn more about factors and R check out Hadley Wickham's chapter on factors in [**R for Data Science 2nd Edition**.](https://r4ds.hadley.nz/factors.html)
:::
:::

```{r warning=FALSE}
# ggplot is easier with factors
pop_by_rp$return_period<-
  factor(pop_by_rp$return_period, 
         levels = c("0", "-3", "-5", "-10", "-20", "-40", "-50", "-60"))
# create the palette to pass to ggplot
leg_colors<-c(
    '#9B0039',
    # -50 to -40
    '#D44135',
    # -40 to -20
    '#FF8D43',
    # -20 to -10
    '#FFC754',
    # -10 to -5
    '#FFEDA3',
    # -5 to -3
    '#fffdc7',
    # -3 to 0
    '#FFF4C7',
    # 0-3
    "#FFFFFF")

# pretty month labels
pop_by_rp[,month:=lubridate::month(pop_by_rp$month, label = TRUE)]
```

Now we can put it all together into a visualization.

```{r}
# add the base data
ggplot2::ggplot(pop_by_rp, 
                # set plot wide aesthetics
                ggplot2::aes(x = month, 
                             y = pop_frac,
                             group = return_period, 
                             fill = return_period))+
  # determine the overlay and positioning of each wsim-class' bar
  ggplot2::geom_bar(stat = "identity", 
                    position = "stack", 
                    color = "darkgrey")+
  # set palettes on the wsim class groups
  # rev() the order of the order of the colors so we can have -60 on the bottom
  ggplot2::scale_fill_manual(values = rev(leg_colors))+
  # limit y axis to 0/1
  ggplot2::ylim(0,1)+
  # labels
  ggplot2::labs(title = "Monthly Fraction of Population Under Water Deficits in California During 2014",
                subtitle = "Categorized by Intensity of Deficit Return Period",
                x = "",
                y = "Fraction of Population*",
                caption = "*Population derived from Gridded Population of the World (2015)",
                color = "Return Period", fill = "Return Period", group = "Return Period", alpha = "Return Period")+
  # use basic theme
  ggplot2::theme_minimal()
```

This figure really illustrates the human impact of the 2014 drought. Nearly 100% of the population was under a 60+ year deficit in January followed by 66% in May and approximately 40% for the remainder of the summer. That is a devastating drought!

::: callout-note
## Knowledge Review

1.  There are several options for spatially subsetting (or clipping) a raster object to a region of interest. What method was used in this lesson?
    a.  Using a vector of dates.
    b.  Using another raster object.
    c.  Specifying a bounding box.
    d.  Using a vector boundary dataset.
2.  When running into memory issues, what is something you can do to reduce the computational load?
    a.  Work with one time frame or region at a time.
    b.  Save it as a new file.
    c.  Subset the data to a region of interest/time frame.
    d.  Find other data to work with.
3.  What is the importance of subsetting data?
    a.  Freeing up space.
    b.  Analyzing a certain time or area of interest.
    c.  Making code run faster.
    d.  All of the above.
4.  Gridded population datasets…(select all that apply)
    a.  Show where river tributaries are.
    b.  Model the distribution of the global human population in raster grid cells.
    c.  Allow analyses of the number of persons impacted by a hazard such as drought.
    d.  Vary in the extent to which ancillary data are used in their production.
:::

## In this Lesson, You Learned...

Congratulations! Now you should be able to:

-   Navigate the SEDAC website to find and download datasets.\
-   Access administrative boundaries from geoBoundaries data using API.
-   Temporally subset a NetCDF raster stack using R packages such as dplyr and lubridate.
-   Crop a NetCDF raster stack with a spatial boundary.
-   Write a subsetted dataset to disk and create an image to share results.
-   Identify areas of severe drought and select these areas for further analysis.
-   Summarize data by county using the exactextractr tool.
-   Integrate WSIM-GLDAS deficit, GPW population, and geoBoundaries administrative boundary data to create complex time series visualizations.

## Lesson 2

In this lesson we explored the California drought of 2014. In our next lesson, we will examine near real-time flood data in California using the MODIS data product.

[Lesson 2: Moderate Resolution Imaging Spectroradiometer (MODIS) Near-Real Time (NRT) flood data](https://ciesin-geospatial.github.io/TOPSTSCHOOL-water/m102-lance-modis-nrt-global-flood.html){.btn .btn-primary .btn role="button"}
