---
title: "CIS670 Project: Analyze COVID-19 in South Korea"
author: "Grayson Guan, Haoran Wang"
date: "5/1/2021"
output: 
  html_document:
    keep_md: true
---

<!-- README.md is generated from README.Rmd. Please edit that file -->



## Introduction

In this project, we perform in-depth EDA(Exploratory Data Analysis) on a collection of data sets about COVID-19 in South Korea. Our goal is to use these data to better understand COVID and learn how to prevent it effectively to save lives.

## Library Dependencies
Our project utilizes the following libraries, please make sure they are installed by using the **install.packages()** command.

* tidyverse
* ggplot2
* ggmap
* gganimate
* gifski
* tibbletime
* data.table
* htmltools
* leaflet
* htmlwidgets

## Datasets
The data is collected from KCDC (Korea Centers for Disease Control & Prevention).
The data set is available on [Kaggle](https://www.kaggle.com/kimjihoo/coronavirusdataset).


```r
# Read Datasets, treat empty string value as NA
case <- read.csv('./Data/Case.csv', na.strings = c("", "NA"))
patient <- read.csv('./Data/PatientInfo.csv', na.strings = c("", "NA"))
policy <- read.csv('./Data/Policy.csv', na.strings = c("", "NA"))
region <- read.csv('./Data/Region.csv', na.strings = c("", "NA"))
searchtrend <- read.csv('./Data/SearchTrend.csv', na.strings = c("", "NA"))
seoulfloating <- read.csv('./Data/SeoulFloating.csv', na.strings = c("", "NA"))
time <- read.csv('./Data/Time.csv', na.strings = c("", "NA"))
timeage <- read.csv('./Data/TimeAge.csv', na.strings = c("", "NA"))
timegender <- read.csv('./Data/TimeGender.csv', na.strings = c("", "NA"))
timeprovince <- read.csv('./Data/TimeProvince.csv', na.strings = c("", "NA"))
weather <- read.csv('./Data/Weather.csv', na.strings = c("", "NA"))
route <- read.csv('./Data/routes.csv', na.strings = c("", "NA"))
patient_1 <- read.csv('./Data/patients.csv', na.strings = c("", "NA"))
```

## Explotary Data Analysis (EDA)
This section contains our analysis of the data in many directions by using visualization.

### Load Libraries

```r
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──
```

```
## ✓ ggplot2 3.3.3     ✓ purrr   0.3.4
## ✓ tibble  3.1.2     ✓ dplyr   1.0.6
## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
## ✓ readr   1.4.0     ✓ forcats 0.5.1
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(ggplot2)
library(ggmap)
```

```
## Google's Terms of Service: https://cloud.google.com/maps-platform/terms/.
```

```
## Please cite ggmap if you use it! See citation("ggmap") for details.
```

```r
library(gganimate)

library(tibbletime)
```

```
## 
## Attaching package: 'tibbletime'
```

```
## The following object is masked from 'package:stats':
## 
##     filter
```

```r
library(data.table)
```

```
## 
## Attaching package: 'data.table'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     between, first, last
```

```
## The following object is masked from 'package:purrr':
## 
##     transpose
```

```r
library(htmltools)
library(leaflet)
library(htmlwidgets)
```
### Total Confirmed Cases by Dates

```r
# Filter out empty value for confirmed date
patient_dates <- patient_1 %>% 
  filter(!is.na(confirmed_date))

# Plot confirmed cases for each day
patient_dates %>%
  group_by(confirmed_date) %>%
  summarise(n = n()) %>%
  ggplot(aes(as.Date(confirmed_date, format = "%m/%d/%Y"), n))+
  geom_line(size = 2, alpha = 0.8, col="red")+
  geom_point(size = 5, alpha = 1, col="purple")+
  labs(title = "Confirmed Cases of COVID-19 in South Korea by Date",
       subtitle = paste0("Total Confirmed Cases: ", nrow(patient)),
       caption = "Green Line Indicates the Trend") + 
  scale_x_date(date_labels = "%b %d", date_breaks = "2 days") +
  theme_light() +
  theme(plot.title = element_text(face = "bold", 
                                  hjust = 0.5, size = 18, color = "black"),
        plot.subtitle = element_text(face = "bold", 
                                     size = 12, color = "blue")) +
  ggrepel::geom_label_repel(aes(label=n), hjust=1.5, vjust=0.9, 
                            fill ="orange") +
  geom_smooth(method = "gam", se = TRUE, alpha = 0.5, col = "green")
```

![](README_figs/README-unnamed-chunk-4-1.png)<!-- -->

### Confirmed Cases Among Different Age Groups

```r
# Calculate Age for each Confirmed Case
patient_age <- patient_1
patient_age$age = 2020 - patient_1$birth_year

# Calculate Age Groups for Confirmed Cases by 10
patient_age$age_group = cut(patient_age$age,
                        breaks = seq(0,90, by=10),
                        right = FALSE,
                        labels = c("0-10","10-20","20-30",
                                   "30-40","40-50","50-60",
                                   "60-70","70-80","80-90"))

patient_age <- patient_age %>%
    select(age_group) %>%
    na.omit() %>%
    group_by(age_group) %>%
    summarise(confirmed_cases = n())
  
ggplot(patient_age, aes(y = age_group, x = confirmed_cases, fill = age_group)) +
  geom_bar(stat="identity") +
  xlab("Confirmed Cases") +
  ylab("Age Group") +
  ggtitle(label = "Confirmed Cases by Age Group")
```

![](README_figs/README-unnamed-chunk-5-1.png)<!-- -->

### Deceased Cases Among Different Age Groups

```r
# Calculate Age for each Confirmed Case
patient_age <- patient_1
patient_age$age = 2020 - patient_1$birth_year

# Calculate Age Groups for Confirmed Cases by 10
patient_age$age_group = cut(patient_age$age,
                            breaks = seq(0,90, by=10),
                            right = FALSE,
                            labels = c("0-10","10-20","20-30",
                                       "30-40","40-50","50-60",
                                       "60-70","70-80","80-90"))
patient_age = patient_age[!is.na(patient_age$deceased_date),]

patient_age <- patient_age %>%
  select(age_group) %>%
  na.omit() %>%
  group_by(age_group) %>%
  summarise(deceased_cases = n())

ggplot(patient_age, aes(y = age_group, x = deceased_cases, fill = age_group)) +
  geom_bar(stat="identity") +
  xlab("deceased Cases") +
  ylab("Age Group") +
  ggtitle(label = "Deceased Cases by Age Group")
```

![](README_figs/README-unnamed-chunk-6-1.png)<!-- -->

### Province with the Highest Number of Confirmed Cases


```r
# Group number of patients by province
patient_province <- patient
patient_province <- patient_province %>%
  select(province) %>%
  na.omit()%>%
  group_by(province)%>%
  summarise(n=n())

ggplot(patient_province, aes(y=n, x=reorder(province,n), fill=province)) +
    geom_bar(stat="identity") +
    coord_flip() +
    xlab("Province Name") +
    ylab("Confirmed Cases") +
    ggtitle(label = "Confirmed Cases in Each Province")
```

![](README_figs/README-unnamed-chunk-7-1.png)<!-- -->

### Confirmed Cases among Genders

```r
patient_gender <- patient
patient_gender <- patient_gender %>%
  select(sex) %>%
  na.omit() %>%
  group_by(sex) %>%
  summarise(patient_num = n())

ggplot(patient_gender,
       aes(x="",y=patient_num,fill=sex)) + 
  geom_bar(width=1, stat="identity",color="white") + 
  ggtitle(label = "Confirmed Cases Gender Ratio in South Korea") + 
  coord_polar("y",start=0) + 
  geom_text(aes(label=paste0(round(patient_num/ sum(patient_num)*100,1),"%")),
            position=position_stack(vjust=0.5)) + 
  theme_void() + 
  theme(plot.title = element_text(size = 30, face = "bold")) 
```

![](README_figs/README-unnamed-chunk-8-1.png)<!-- -->

### Deceased Cases among Genders

```r
patient_gender <- patient
patient_gender = patient_gender[!is.na(patient_gender$deceased_date),]
patient_gender <- patient_gender %>%
  select(sex) %>%
  na.omit() %>%
  group_by(sex) %>%
  summarise(patient_num = n())
ggplot(patient_gender,
       aes(x="",y=patient_num,fill=sex)) + 
  geom_bar(width=1, stat="identity",color="white") + 
  ggtitle(label = "Deceased Gender Ratio in South Korea") + 
  coord_polar("y",start=0) + 
  geom_text(aes(label=paste0(round(patient_num/ sum(patient_num)*100,1),"%")),
            position=position_stack(vjust=0.5)) + 
  theme_void() + 
  theme(plot.title = element_text(size = 30, face = "bold")) 
```

![](README_figs/README-unnamed-chunk-9-1.png)<!-- -->

### Mortality Rate among Genders

```r
patient_gender <- patient
patient_gender_d = patient_gender[!is.na(patient_gender$deceased_date),]

patient_gender <- patient_gender %>%
  select(sex) %>%
  na.omit() %>%
  group_by(sex) %>%
  summarise(patient_num = n())

patient_gender_d <- patient_gender_d %>%
  select(sex) %>%
  na.omit() %>%
  group_by(sex) %>%
  summarise(patient_num = n())

patient_gender_d[2] = patient_gender_d[2]/ patient_gender[2] * 100

ggplot(patient_gender_d, aes(x = sex, y = patient_num, fill = sex)) +
  geom_bar(stat="identity") +
  xlab("mortality rate") +
  ylab("Gender") +
  ggtitle(label = "Mortality Rate by Gender")
```

![](README_figs/README-unnamed-chunk-10-1.png)<!-- -->

```r
# Filter out empty value for confirmed date
time_data <- time
# Plot confirmed cases for each day
ggplot () + 
  geom_point(data = time_data, aes(x = as.Date(date, format = "%Y-%m-%d"), y = confirmed), color = "blue") +
  geom_point(data = time_data, aes(x = as.Date(date, format = "%Y-%m-%d"), y = deceased), color = "red") + 
  geom_point(data = time_data, aes(x = as.Date(date, format = "%Y-%m-%d"), y = released), color = "green") +
  scale_x_date(date_labels = "%b %d", date_breaks = "30 days")
```

![](README_figs/README-unnamed-chunk-11-1.png)<!-- -->

### Confirmed Cases: Age Group vs. Province

```r
patient_age_province <- patient_1
patient_age_province$age = 2020 - patient_1$birth_year
patient_age_province$age_group = cut(patient_age_province$age,
                        breaks = seq(0,90, by=10),
                        right = FALSE,
                        labels = c("0-10","10-20","20-30",
                                   "30-40","40-50","50-60",
                                   "60-70","70-80","80-90"))

patient_age_province %>%
    select(age_group, province) %>%
    na.omit() %>%
    group_by(age_group, province) %>%
    summarise(Count = n()) %>%
    ggplot() +
    geom_tile(aes(x = age_group, y = province, fill = Count)) +
    geom_vline(xintercept = seq(0.5,9.5,by = 1), linetype = 'dashed') +
    geom_hline(yintercept = seq(0.5,14.5,by = 1), linetype = 'dashed') +
    scale_fill_gradientn(colours = c("green","red"),
                         values = c(0,0.1,1)) +
    scale_x_discrete(expand = c(0,0)) +
    scale_y_discrete(expand = c(0,0)) +
    theme_classic() +
    theme(text = element_text(size = 15, face = "bold"),
          axis.text.x = element_text(angle = 90),
          legend.position = "top",
          legend.key.width = unit(3,"cm"))
```

![](README_figs/README-unnamed-chunk-12-1.png)<!-- -->

### Confirmed Cases: Age Group vs. Province vs. Gender

```r
patient_age_province_gender <- patient_age_province
patient_age_province_gender %>%
    select(sex, age_group, province) %>%
    na.omit() %>%
    group_by(sex, age_group, province) %>%
    summarise(Count = n()) %>%
    ggplot() +
    geom_tile(aes(x = age_group, y = province, fill = Count)) +
    geom_vline(xintercept = seq(0.5,9.5,by = 1), linetype = 'dashed') +
    geom_hline(yintercept = seq(0.5,14.5,by = 1), linetype = 'dashed') +
    scale_fill_gradientn(colours = c("green","red"),
                         values = c(0,0.1,1)) +
    scale_x_discrete(expand = c(0,0)) +
    scale_y_discrete(expand = c(0,0)) +
    theme_classic() +
    theme(text = element_text(size = 15, face = "bold"),
          axis.text.x = element_text(angle = 90),
          legend.position = "top",
          legend.key.width = unit(3,"cm"))   +
    facet_wrap(~sex)
```

![](README_figs/README-unnamed-chunk-13-1.png)<!-- -->


### Visualize Confirmed Cases on a Map

```r
case$longitude <- as.numeric(as.character(case$longitude))
```

```
## Warning: NAs introduced by coercion
```

```r
case$latitude <- as.numeric(as.character(case$latitude))
```

```
## Warning: NAs introduced by coercion
```

```r
# Set Google API key
register_google(key = "AIzaSyCiHpBw6DsGxXs6pgfgQTd2tP4wAvzrmJI")
# Get Map from Google Maps API
map = get_googlemap(center = c(lon = 127.7669, lat = 35.9087),
                    zoom = 7, scale = 4,
                    maptype = 'roadmap',
                    color = 'color')

figure <- ggmap(map) +
    theme_void() +
    ggtitle("COVID-19 Cases in South Korea") +
    scale_color_gradientn(colors = rainbow(5)) +
    theme(plot.title = element_text(color = "blue"),
          panel.border = element_rect(color = "grey", fill = NA, size = 2))

figure + geom_point(data = case, 
                    aes(longitude, latitude, size= confirmed, 
                        color=confirmed, group=confirmed))
```

```
## Warning: Removed 109 rows containing missing values (geom_point).
```

![](README_figs/README-unnamed-chunk-14-1.png)<!-- -->

### Confirmed Cases Over Time Animation

```r
# Get a data frame that contains both time and coordinates
case_bak <- subset(case, select = c(province, longitude, latitude))

geo_time <- merge(case_bak, timeprovince, by="province")
geo_time$date <- as.Date(geo_time$date, format = "%Y-%m-%d")


# Set Google API key
register_google(key = "AIzaSyCiHpBw6DsGxXs6pgfgQTd2tP4wAvzrmJI")
# Get Map from Google Maps API
map = get_googlemap(center = c(lon = 127.7669, lat = 35.9087),
                    zoom = 7, scale = 4,
                    maptype = 'roadmap',
                    color = 'color')

figure <- ggmap(map) +
    theme_void() +
    ggtitle("COVID-19 Cases in South Korea") +
    scale_color_gradientn(colors = rainbow(5)) +
    theme(plot.title = element_text(color = "blue"),
          panel.border = element_rect(color = "grey", fill = NA, size = 2))

figure + geom_point(data = geo_time, 
                    aes(longitude, latitude, size= confirmed, 
                        color=confirmed, group=confirmed)) +
  transition_time(date)
```
![](./README_figs/README-unnamed-chunk-15-1.gif)
### Confirmed Case Trajectory Visualization

```r
# Set Google API key
register_google(key = "AIzaSyCiHpBw6DsGxXs6pgfgQTd2tP4wAvzrmJI")
# Get Map from Google Maps API
map = get_googlemap(center = c(lon = 127.7669, lat = 35.9087),
                    zoom = 7, scale = 4,
                    maptype = 'roadmap',
                    color = 'color')

ggmap(map) +
  geom_point(data = route,
             aes(x = longitude, y = latitude,col =  as.factor(global_id)), size = 5) +
  geom_line(data = route,
            aes(x = longitude, y = latitude,col =  as.factor(global_id), group = as.factor(global_id)), 
            size = 2,alpha = 0.7) +
  guides(col = FALSE) +
  theme(text = element_text(size = 15, face = "bold"))
```

```
## Warning: Removed 1954 rows containing missing values (geom_point).
```

```
## Warning: Removed 1954 row(s) containing missing values (geom_path).
```

![](README_figs/README-unnamed-chunk-16-1.png)<!-- -->


### Massive Infection Visualization


```r
# Known Group Infection Events
group_infec <- tibble(lon = c(127.0518049,126.9165592,127.1217472,128.7368285,129.0771404,128.4941047,128.5707814,128.5665743,128.9099632),
                          lat = c(37.2367708,37.6338843,37.3881528,35.6486172,35.2159947,36.0579173,35.8507155,35.8397168,36.9273528),
                          address = c('650-103 mangpo-dong, yeongtong-gu, suwon, gyeonggi-do, south korea','1021 tongil-ro, jingwan-dong, eunpyeong-gu, seoul, south korea','20 seohyeon-ro 180(baekpalsip)beo, seohyeon-dong, bundang-gu, seongnam-si, gyeonggi-do, south korea','100-1 beomgok-ri, hwayang-eup, cheongdo, gyeongsangbuk-do, south korea','436-8 oncheon 1(il)-dong, dongnae-gu, busan, south korea','918 yuhak-ro, gasan-myeon, chilgok-gun, gyeongsangbuk-do, south korea','72-10 seongdang-dong, dalseo-gu, daegu, south korea','81 daemyeong-ro, daemyeong 10(sip)-dong, nam-gu, daegu, south korea','916 soro-ri, chunyang-myeon, bonghwa-gun, gyeongsangbuk-do, south korea'),
                          korean_name = c('수원 생명샘교회','은평 성모병원','분당 제생병원','청도 대남병원','부산 온천교회','칠곡 밀알 사랑의집','대구 한마음아파트','대구 신천지 교회','봉화 푸른 요양원'),
                          english_name = c('Soowon Saengmyungsame Church','Eunpyung Sungmo Hospital','Boondang Jesaeng Hospital','Chungdo Daenam Hospital','Busan Onchun Church','Chilgok Milal Sarang House','Daegu Hanmaeum Apartment','Daegu Sinchunji Church','Bonghwa Pureun Clinic'),
                          n_of_infections= c(10,14,11,118,34,22,46,4482,51))

# Pre-processing
group_infec_2 <- group_infec %>% select(lon,lat,korean_name,english_name,n_of_infections) %>% 
  rename(latitude2 = lat, longitude2 = lon, id = english_name) %>% 
  as.data.table() 

group_infec_3 <- group_infec_2[, list(label=HTML(
  paste(sep = "<br>",
        paste("G.I(eng): ", id),
        paste("G.I(kor): ", korean_name),
        paste("Num of infects:", n_of_infections)))),
  list(id, longitude2, latitude2)]

#Combining route and patient and pre-processing

agg_1 <- route %>% left_join(patient_1, by="global_id") %>% 
  mutate(infected_by = ifelse(is.na(infected_by),"Unk",infected_by)) %>% 
  as.data.table

agg_2 <- agg_1[, list(label=HTML(
  paste(sep="<br>",
        paste("Patient Id :", global_id),
        paste("Sex :", sex),
        paste("Country :", country),
        paste("Disease :", disease),
        paste("Group:", group),
        paste("Infection_reason:", infection_reason),
        paste("Infection_order:",  infection_order),
        paste("Infected_by:", infected_by),
        paste("Contact_number:", contact_number),
        paste("Confirmed_date:", confirmed_date),
        paste("State:", state)))), 
  list(global_id, longitude, latitude)]

listed <- list(agg_2, group_infec_3)
agg_3 <- rbindlist(listed, use.names = TRUE, fill = TRUE)

m = agg_3 %>% 
  leaflet() %>%
  setView(128.5, 36.5, zoom = 7) %>%
  addTiles() %>%
  addProviderTiles("CartoDB.Positron") %>%
  #addProviderTiles : Stamen.Toner, CartoDB.Positron, Esri.NatGeoWorldMap, MtbMap
  addCircleMarkers(
    lat =  ~ latitude,
    lng =  ~ longitude,
    label = ~label,
    clusterOptions = markerClusterOptions(),
    labelOptions = labelOptions(noHide = F, 
                                direction = "left",
                                style = list(
                                  "color" = "black",
                                  "box-shadow" = "4px 4px rgba(0,0,0,0.25)",
                                  "font-size" = "13px",
                                  "border-color" = "rgba(0.5,3,0.5,0.5)"))) %>% 
  
  #Addding group infections
  addMarkers(
    lat =  ~ latitude2,
    lng =  ~ longitude2,
    label = ~label,
    labelOptions = labelOptions(noHide = F, 
                                direction = "right",
                                style = list(
                                  "color" = "red",
                                  "font-size" = "13px")))
```

```
## Warning in validateCoords(lng, lat, funcName): Data contains 338 rows with
## either missing or invalid lat/lon values and will be ignored
```

```
## Warning in validateCoords(lng, lat, funcName): Data contains 2050 rows with
## either missing or invalid lat/lon values and will be ignored
```

```r
saveWidget(m, file="mass.html")
```
![](./mass.html)



## Regression Analysis
