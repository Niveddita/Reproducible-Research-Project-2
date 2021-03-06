
```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```



##Synopsis
The data is analysed by firstly reading it into R as a data frame. To enable analysis of the damage to human health, the dataframe is summarised by Event type to calculate the total damage by each event type. Datapoints with comparitively little damage are then filtered out of the dataset to make the final plots more readable. A metric for the total damage to human health (fatalities + injuries) is also calculated with a weighting such that a fatality is worth twice as much as a injury. The total financial damage is calculated by first expanding the data (i.e. 2.5 and k becomes 2500) before summarising it in a similar way to the human health data. In this case the metric for total damage is simply the sum of the property and crop damage. Both datasets are also melted before plotting to make it easier to use plotting libraries (in this case ggplot2). Also, the datasets are arranged in descending order of total damage to make it easier to visualise the data when it is presented in a tabular format.

## Data Processing
Loads the needed libraries
```{r}
library(dplyr)
library(ggplot2)
library(reshape2)
```

Downloads and Reads the dataset into R.
```{r cache=TRUE}
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
download.file(url, "StormData.csv.bz2")
library(R.utils)
bunzip2("StormData.csv.bz2", "StormData.csv")
df <- read.csv("StormData.csv")
```
Data Processing
Health Impact
To evaluate the health impact, the total fatalities and the total injuries for each event type (EVTYPE) are calculated. The codes for this calculation are shown as follows.
```{r}
library(dplyr)
df.fatalities <- df %>% select(EVTYPE, FATALITIES) %>% group_by(EVTYPE) %>% summarise(total.fatalities = sum(FATALITIES)) %>% arrange(-total.fatalities)
head(df.fatalities, 10)
df.injuries <- df %>% select(EVTYPE, INJURIES) %>% group_by(EVTYPE) %>% summarise(total.injuries = sum(INJURIES)) %>% arrange(-total.injuries)
head(df.injuries, 10)
```

## Economic Impact
The data provides two types of economic impact, namely property damage (PROPDMG) and crop damage (CROPDMG). The actual damage in $USD is indicated by PROPDMGEXP and CROPDMGEXP parameters. According to this link, the index in the PROPDMGEXP and CROPDMGEXP can be interpreted as the following:-

H, h -> hundreds = x100
K, K -> kilos = x1,000
M, m -> millions = x1,000,000
B,b -> billions = x1,000,000,000
(+) -> x1
(-) -> x0
(?) -> x0
blank -> x0

The total damage caused by each event type is calculated with the following code.
```{r}
df.damage <- df %>% select(EVTYPE, PROPDMG,PROPDMGEXP,CROPDMG,CROPDMGEXP)

Symbol <- sort(unique(as.character(df.damage$PROPDMGEXP)))
Multiplier <- c(0,0,0,1,10,10,10,10,10,10,10,10,10,10^9,10^2,10^2,10^3,10^6,10^6)
convert.Multiplier <- data.frame(Symbol, Multiplier)

df.damage$Prop.Multiplier <- convert.Multiplier$Multiplier[match(df.damage$PROPDMGEXP, convert.Multiplier$Symbol)]
df.damage$Crop.Multiplier <- convert.Multiplier$Multiplier[match(df.damage$CROPDMGEXP, convert.Multiplier$Symbol)]

df.damage <- df.damage %>% mutate(PROPDMG = PROPDMG*Prop.Multiplier) %>% mutate(CROPDMG = CROPDMG*Crop.Multiplier) %>% mutate(TOTAL.DMG = PROPDMG+CROPDMG)

df.damage.total <- df.damage %>% group_by(EVTYPE) %>% summarize(TOTAL.DMG.EVTYPE = sum(TOTAL.DMG))%>% arrange(-TOTAL.DMG.EVTYPE) 

head(df.damage.total,10)
```
# Results
###Health Impact
The top 10 events with the highest total fatalities and injuries are shown graphically.
```{r}
library(ggplot2)
g <- ggplot(df.fatalities[1:10,], aes(x=reorder(EVTYPE, -total.fatalities), y=total.fatalities))+geom_bar(stat="identity") + theme(axis.text.x = element_text(angle=90, vjust=0.5, hjust=1))+ggtitle("Top 10 Events with Highest Total Fatalities") +labs(x="EVENT TYPE", y="Total Fatalities")
g
```

```{r}
g <- ggplot(df.injuries[1:10,], aes(x=reorder(EVTYPE, -total.injuries), y=total.injuries))+geom_bar(stat="identity") + theme(axis.text.x = element_text(angle=90, vjust=0.5, hjust=1))+ggtitle("Top 10 Events with Highest Total Injuries") +labs(x="EVENT TYPE", y="Total Injuries")
g
```
As shown in the figures, tornado causes the hightest in both the total fatality and injury count.

## Economic Impact
The top 10 events with the highest total economic damages (property and crop combined) are shown graphically.
```{r}
g <- ggplot(df.damage.total[1:10,], aes(x=reorder(EVTYPE, -TOTAL.DMG.EVTYPE), y=TOTAL.DMG.EVTYPE))+geom_bar(stat="identity") + theme(axis.text.x = element_text(angle=90, vjust=0.5, hjust=1))+ggtitle("Top 10 Events with Highest Economic Impact") +labs(x="EVENT TYPE", y="Total Economic Impact ($USD)")

g
```
As shown in the figure, flood has the highest economic impact.

### Conclusion  
From these data, we found that **excessive heat** and **tornado** are most harmful with respect to population health, while **flood**, **drought**, and **hurricane/typhoon** have the greatest economic consequences.
