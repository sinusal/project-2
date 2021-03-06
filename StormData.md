
---
title: "StormData"
author: "Sinusal"
date: "28/04/2021"
output: html_document
---

## Characteristics of major storms and weather events in the United States

### Synonpsis  
This report aims to analyze Characteristics of different weather events and their impact on public health and economic in the United State. This analyzes was based on the storm database collected from the U.S. National Oceanic and Atmospheric Administration's (NOAA) from 1950 - 2011. 
The present analysis shows that tornados have the most harmful impact on people's health. From an economic view, floods are most expensive.


### Data Processing
 Load the required libraries
```{r}
library(R.utils)
library(ggplot2)
library(plyr)
library(dplyr)
library(data.table)
require(gridExtra)
```

Downloading and unzipping file from web
The source data file is downloaded from https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2.
It covers wheather events between 1950 and 2011.
```{r}
if (!"stormData.csv.bz2" %in% dir("./data/")) {
  print("hhhh")
  download.file("http://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", destfile = "data/stormData.csv.bz2")
  bunzip2("data/stormData.csv.bz2", overwrite=T, remove=F)
}
```
Reading StormData csv file
```{r}
StormData<-read.csv("data/stormData.csv.bz2", sep = ",")
# Show the structure of the dataset
str(StormData)
```

There are 902297 observation and 37 variables in the dataset
1.Variables that will be used: EVTYPE: Event Type (Tornados, Flood, ..), FATALITIES: Number of Fatalities,INJURIES: Number of Injuries, PROGDMG: Property Damage, PROPDMGEXP: Units for Property Damage (magnitudes - K,B,M), CROPDMG: Crop Damage, CROPDMGEXP: Units for Crop Damage (magnitudes - K,BM,B)
2. According to the NOAA (https://www.ncdc.noaa.gov/stormevents/details.jsp) the full set of wheather events (48 event types) is available since 1996. Between 1950 and 1995 only a subset (Tornado, Thunderstorm Wind and Hail) of these events is available in the storm database. In order to have o comparable basis for the analysis the dataset is limited to the observations between 1996 and 2011.
3.We excluded observations without any information about health and/or economic damage from analysis.
```{r}
stormDataSelected <- select(StormData, BGN_DATE, EVTYPE, PROPDMG, PROPDMGEXP, CROPDMG, CROPDMGEXP, FATALITIES, INJURIES)


# Format the BGN_DATE variable as a date
stormDataSelected$BGN_DATE <- as.Date(stormDataSelected$BGN_DATE, "%m/%d/%Y")
stormDataSelected$YEAR <- year(stormDataSelected$BGN_DATE)

# Tornado 1950 - 1954
# Tornado, Thunderstorm Wind, Hail 1955 - 1995
# 48 Events since 1996
# Only use events since 1996
stormDataSelected <- filter(stormDataSelected, YEAR >= 1996)

# We will use Only events with either health impact or economic damage
stormDataSelected <- filter(stormDataSelected, PROPDMG > 0 | CROPDMG > 0 | FATALITIES > 0 | INJURIES > 0)
```

Adjustment are needed in for economic damage variables
```{r}
table(stormDataSelected$PROPDMGEXP)
table(stormDataSelected$CROPDMGEXP)
```
convert the exponents into corresponding factors:
"", "?", "+", "-": 1
"0": 1
"1": 10
"2": 100
"3": 1.000
"4": 10.000
"5": 100.000
"6": 1.000.000
"7": 10.000.000
"8": 100.000.000
"9": 1.000.000.000
"H": 100
"K": 1.000
"M": 1.000.000
*"B": 1.000.000.000
```{r}
stormDataSelected$PROPDMGEXP <- toupper(stormDataSelected$PROPDMGEXP)
stormDataSelected$CROPDMGEXP <- toupper(stormDataSelected$CROPDMGEXP)

stormDataSelected$CROPDMGFACTOR[(stormDataSelected$CROPDMGEXP == "")] <- 10^0
stormDataSelected$CROPDMGFACTOR[(stormDataSelected$CROPDMGEXP == "?")] <- 10^0
stormDataSelected$CROPDMGFACTOR[(stormDataSelected$CROPDMGEXP == "0")] <- 10^0
stormDataSelected$CROPDMGFACTOR[(stormDataSelected$CROPDMGEXP == "2")] <- 10^2
stormDataSelected$CROPDMGFACTOR[(stormDataSelected$CROPDMGEXP == "K")] <- 10^3
stormDataSelected$CROPDMGFACTOR[(stormDataSelected$CROPDMGEXP == "M")] <- 10^6
stormDataSelected$CROPDMGFACTOR[(stormDataSelected$CROPDMGEXP == "B")] <- 10^9

stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "")] <- 10^0
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "-")] <- 10^0
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "?")] <- 10^0
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "+")] <- 10^0
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "0")] <- 10^0
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "1")] <- 10^1
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "2")] <- 10^2
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "3")] <- 10^3
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "4")] <- 10^4
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "5")] <- 10^5
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "6")] <- 10^6
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "7")] <- 10^7
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "8")] <- 10^8
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "H")] <- 10^2
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "K")] <- 10^3
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "M")] <- 10^6
stormDataSelected$PROPDMGFACTOR[(stormDataSelected$PROPDMGEXP == "B")] <- 10^9
```

Creating a new variable where fatalities and injuries are added to form HEALTHIMP.
crop and property damages (in USD) variables are multiplied by their corresponding factor and added to form a new variable ECONOMICCOST.
```{r}
stormDataSelected <- mutate(stormDataSelected, HEALTHIMP = FATALITIES + INJURIES)
stormDataSelected <- mutate(stormDataSelected, ECONOMICCOST = PROPDMG * PROPDMGFACTOR + CROPDMG * CROPDMGFACTOR)
```

```{r}
stormDataSelected$EVTYPE <- toupper(stormDataSelected$EVTYPE)
dim(data.frame(table(stormDataSelected$EVTYPE)))
evtypeUnique <- unique(stormDataSelected$EVTYPE)
evtypeUnique[grep("THUND", evtypeUnique)]
```

health impact (HEALTHIMP) is summed up per event type
```{r}
healthImpact <- with(stormDataSelected, aggregate(HEALTHIMP ~ EVTYPE, FUN = sum))
subset(healthImpact, HEALTHIMP > quantile(HEALTHIMP, prob = 0.95))

stormDataSelected$EVTYPE[(stormDataSelected$EVTYPE == "HURRICANE")] <- "HURRICANE (TYPHOON)"
stormDataSelected$EVTYPE[(stormDataSelected$EVTYPE == "STORM SURGE")] <- "STORM SURGE/TIDE"
```

###Result
The cleaned data frame stormDataSelected is been aggregated per EVTYPE and provided in a descending order in the new data frame healthImpact.
```{r}
healthImpact <- stormDataSelected %>% 
                group_by(EVTYPE) %>% 
                summarise(HEALTHIMP = sum(HEALTHIMP)) %>% 
                arrange(desc(HEALTHIMP))
#healthImpact[1:10,]
g1 <- ggplot(healthImpact[1:10,], aes(x=reorder(EVTYPE, -HEALTHIMP),y=HEALTHIMP,color=EVTYPE)) + 
      geom_bar(stat="identity", fill="white") + 
      theme(axis.text.x = element_text(angle = 90, hjust = 1)) + 
      xlab("Event") + ylab("Number of fatalities and injuries") +
      theme(legend.position="none") +
      ggtitle("Fatalities and injuries in the US caused by severe weather events")
g1
```
The figure shows that Tornados are the most harmful weather events for people's health.

The cleaned data frame stormDataSelected is been aggregated per EVTYPE and provided in a descending order in the new data frame economicCost.
```{r}
economicCost <- stormDataSelected %>% 
                group_by(EVTYPE) %>% 
                summarise(ECONOMICCOST = sum(ECONOMICCOST)) %>% 
                arrange(desc(ECONOMICCOST))
#economicCost[1:10,]
g1 <- ggplot(economicCost[1:10,], aes(x=reorder(EVTYPE, -ECONOMICCOST),y=ECONOMICCOST,color=EVTYPE)) + 
      geom_bar(stat="identity", fill="white") + 
      theme(axis.text.x = element_text(angle = 90, hjust = 1)) + 
      xlab("Event") + ylab("Economic cost in USD") +
      theme(legend.position="none") +
      ggtitle("Economic cost in the US caused by severe weather events")
g1
```
The figure shows that Floods cause the biggest economical damages.

### Conclusion  
From these data, we found that **excessive heat** and **tornado** are most harmful with respect to population health, while **flood**, **drought**, and **hurricane/typhoon** have the greatest economic consequences.