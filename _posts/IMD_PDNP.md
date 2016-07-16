---
layout: post
published: false
title: Untitled
---
# Introduction

The Government's Indices of Deprivation 2015 uses a wide range of indicators to produce nationally consistent measures of deprivation across seven domains or themes. The indices have changed considerably since the first Index of Multiple Deprivation in 2000. Deprivation covers a broad range of issues and refers to unmet needs caused by a lack of resources of all kinds, not just financial. It includes measures such as low income, worklessness, poor qualifications, disability, crime, housing affordability and air quality.

We can now compare between 2004, 2007, 2010 and 2015. These statistics are a measure of deprivation, not affluence, and to recognise that not every person in a highly deprived area will themselves be deprived. Equally, there will be some deprived people living in the least deprived areas.

This analysis:

1. Scrape the data (where available) from [data.gov.uk](https://data.gov.uk/)
2. Subset the data based on Lower Super Output Areas that intersect the Peak District National Park
3. Map the latest data using [Leaflet for R](https://rstudio.github.io/leaflet/)

Super Output Areas are a geographic hierarchy designed to improve the reporting of small area statistics. The 34,753 LSOAs in England (32,844) and Wales (1,909) were built from groups of output areas that were created in 2003 and maintained in 2011. They have a minimum of 1,000 residents.

# Data Munging

## Load the data

2004 Data is not available in the correct format online, so a prepared .csv is loaded in to the working directory

{% highlight r %}
## Import the English Indices of Deprivation 2004 - LSOA Level
IMD04 <- read.csv("/home/rstudio/Dropbox/PDNP/Maps/IMD/IMD DATA 2004/IMD2004.csv")
## Import the English Indices of Deprivation 2007 - LSOA Level
IMD07 <- read.csv(url("http://opendatacommunities.org/data/imd-rank-2007/cube/observations.csv?rows_dimension=http%3A%2F%2Fpurl.org%2Flinked-data%2Fsdmx%2F2009%2Fdimension%23refArea&columns_dimension=http%3A%2F%2Fpurl.org%2Flinked-data%2Fsdmx%2F2009%2Fdimension%23refPeriod&measure_property=http%3A%2F%2Fopendatacommunities.org%2Fdef%2FIMD%23IMD-rank"))
## Import the English Indices of Deprivation 2010 - LSOA Level
IMD10 <- read.csv(url("https://www.gov.uk/government/uploads/system/uploads/attachment_data/file/15240/1871702.csv"))
## Import the English Indices of Deprivation 2015 - LSOA Level
IMD15 <- read.csv(url("https://www.gov.uk/government/uploads/system/uploads/attachment_data/file/467774/File_7_ID_2015_All_ranks__deciles_and_scores_for_the_Indices_of_Deprivation__and_population_denominators.csv"))
{% endhighlight %}

## Tidy the data

{% highlight r %}
## Format the column names 2004
names(IMD04) [2] <- "Rank of Index of Multiple Deprivation Score"
names(IMD04) [3] <- "% Most Deprived"
## Format the column names 2007 
names(IMD07) [1] <- "Reference Area"
names(IMD07) [2] <- "Rank"
## Format the column names 2010
names(IMD10) [1] <- "LSOA CODE"
names(IMD10) [2] <- "PRE 2009 LA CODE"
names(IMD10) [3] <- "PRE 2009 LA NAME"
names(IMD10) [4] <- "POST 2009 LA CODE"
names(IMD10) [5] <- "POST 2009 LA NAME"
names(IMD10) [6] <- "GOR CODE"
names(IMD10) [7] <- "GOR NAME"
names(IMD10) [8] <- "IMD SCORE"
names(IMD10) [9] <- "RANK OF IMD SCORE (where 1 is most deprived)"
## Format the column names 2015
names(IMD15) [1] <- "LSOA code (2011)"
names(IMD15) [2] <- "LSOA name"
names(IMD15) [3] <- "Local Authority District code (2013)"
names(IMD15) [4] <- "Local Authority District name (2013)"
names(IMD15) [5] <- "IMD Score"
names(IMD15) [6] <- "IMD Rank (where 1 is most deprived)"
names(IMD15) [7] <- "IMD Decile"
{% endhighlight %}

Subset the data columns

{% highlight r %}
## Convert column to numeric 2007
IMD07$Rank <- as.numeric(as.character(IMD07$Rank))
## Subset the data columns 2010
IMD10 <- IMD10[ , 1:9]
## Subset the data columns 2015
IMD15 <- IMD15[ , 
{% endhighlight %}

We need to subset the data by geography. For this we use a list of LSOA where the population is greater than 50% in the Peak District National Park Boundary. 2004 has already been processed before it was imported. 

**2007** Full LSOA name is only availble in 2007 data

{% highlight r %}
## Create a character vector of LSOA Codes
LSOAName <- c("High Peak 007C", "High Peak 007A", "High Peak 007B", "Derbyshire Dales 001A", "Derbyshire Dales 001B", 
              "Derbyshire Dales 001C", "Derbyshire Dales 001D", "Derbyshire Dales 002A", "Derbyshire Dales 002B", 
              "Derbyshire Dales 002C", "Derbyshire Dales 002D", "Derbyshire Dales 003A", "Derbyshire Dales 003B", 
              "Derbyshire Dales 003C", "Derbyshire Dales 003D", "Derbyshire Dales 006F", "Derbyshire Dales 008C", 
              "Staffordshire Moorlands 001A", "Staffordshire Moorlands 007A", "Staffordshire Moorlands 007C")
## Subest the data based on that vector
IMD07 <- IMD07[IMD07[['Reference Area']] %in% LSOAName, ]
{% endhighlight %}

Use a pre determined list of LSOA codes to subset the information required. Below is a subset of LSOA codes where they intersect the Peak District National Park Boundary. Further projects could include the weighted polygons to query the data by LSOA = > 50% population. 

{% highlight r %}
## Create a character vector of LSOA Codes
LSOA_PDNP <- c("E01005409", "E01005410", "E01007426", "E01007926", "E01007956", 
               "E01008128", "E01008129", "E01008147", "E01008158", "E01011075", 
               "E01011168", "E01011174", "E01011177", "E01011182", "E01018586", 
               "E01018591", "E01018669", "E01018670", "E01019599", "E01019600", 
               "E01019601", "E01019602", "E01019604", "E01019605", "E01019606", 
               "E01019611", "E01019613", "E01019614", "E01019615", "E01019617", 
               "E01019618", "E01019620", "E01019630", "E01019631", "E01019632", 
               "E01019712", "E01019713", "E01019714", "E01019718", "E01019725", 
               "E01019734", "E01019735", "E01019736", "E01019737", "E01019741", 
               "E01019742", "E01019744", "E01019754", "E01019755", "E01019763", 
               "E01019764", "E01019770", "E01019771", "E01029467", "E01029797", 
               "E01029801", "E01029817")
{% endhighlight %}

**2010 and 2015** Subset the data

{% highlight r %}
## Subest the data based on that vector
IMD10 <- IMD10[IMD10[['LSOA CODE']] %in% LSOA_PDNP, ]
IMD15 <- IMD15[IMD15[['LSOA code (2011)']] %in% LSOA_PDN
{% endhighlight %}

# Results

There are 32,482 LSOA in Engalnd and Wales. Averaging the IMD score and dividing by the total number of areas provided us with an average rank for the area. 

{% highlight r %}
## Average Rank of the data
AverageRank15 <- (mean(IMD15$`IMD Rank (where 1 is most deprived)`)/32482)*100
AverageRank10 <- (mean(IMD10$`RANK OF IMD SCORE (where 1 is most deprived)`)/32482)*100
AverageRank07 <- (mean(IMD07$Rank)/32482)*100
AverageRank04 <- (mean(IMD04$`Rank of Index of Multiple Deprivation Score`)/32482)*100
{% endhighlight %}

{% highlight r %}
Year <- c("Average Rank 2004", "Average Rank 2007", "Average Rank 2010", "Average Rank 2015")
Percent_Most_Deprived <- c(AverageRank04, AverageRank07, AverageRank10, AverageRank15)
AverageRank <- data.frame(Year, Percent_Most_Deprived)
knitr::kable(AverageRank)
{% endhighlight %}

# Map the 2015 Data




{% highlight r %}

{% endhighlight %}





{% highlight r %}

{% endhighlight %}





{% highlight r %}

{% endhighlight %}

