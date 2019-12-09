---
title: "README"
author: "Ki Oh"
date: "12/9/2019"
output: html_document
---
## iHealth - R Analysis of Apple Health Exports
## Load Libraries
```{r}
#=====================================
library(dplyr)
library(ggplot2)
library(lubridate)
library(XML)
```

## Data Wrangling
```{r}
#=====================================
#load apple health export.xml file
xml <- xmlParse("data/export.xml")
#=====================================
#transform xml file to data frame - select the Record rows from the xml file
#=====================================
df <- XML:::xmlAttrsToDataFrame(xml["//Record"])
str(df)
#make value variable numeric
#=====================================
df$value <- as.numeric(as.character(df$value))
str(df)
#make endDate in a date time variable POSIXct using lubridate with eastern time zone
#=====================================
df$endDate <-ymd_hms(df$endDate,tz="America/New_York")
str(df)

##add in year month date dayofweek hour columns
df$month<-format(df$endDate,"%m")
df$year<-format(df$endDate,"%Y")
df$date<-format(df$endDate,"%Y-%m-%d")
df$dayofweek <-wday(df$endDate, label=TRUE, abbr=FALSE)
df$hour <-format(df$endDate,"%H")
str(df)
```

#show steps by month by year using dplyr then graph using ggplot2
```{r}
df %>%
  filter(type == 'HKQuantityTypeIdentifierStepCount') %>%
  filter(year==2018) %>%
  group_by(year,month) %>%
  summarize(steps=sum(value)) %>%
  #print table steps by month by year
  print (n=100) %>%
  #graph data by month by year
  ggplot(aes(x=month, y=steps, fill=year)) + 
  geom_bar(position='dodge', stat='identity') +
  scale_y_continuous(labels = scales::comma) +
  scale_fill_brewer() +
  theme_bw() +  
  theme(panel.grid.major = element_blank())+geom_title("Steps by Month")
#================================================================
```
<img src="md_img/stepsbymonths2018.png"/>