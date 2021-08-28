---
title: "How Does a Bike-Share Navigate Speedy Success?"
author: "Bleron Ireland-Fejzullahu"
output: html_notebook
---

## How do annual members and casual  riders use Cyclistic bikes differently?

### Set up environment 

```{r}
install.packages("tidyverse")
install.packages ("rmarkdown")
library(tidyverse)
library(readr)
```
### Import Cyclistic trip data

```{r}
trip_data <- read_csv("divvy-tripdata.csv")
colnames(trip_data)
View(trip_data)
```

## Clean the data
### Load "tidyr"

Separate started_at column into start_date and start time.
Separate ended_at column into end_date and end_time

```{r}
library(tidyr)
cleaned_trip_data <- trip_data %>%
  separate(started_at, into = c('start_date', 'start_time'), sep = (' ')) %>%
  separate(ended_at, into = c('end_date','end_time'), sep = (' '))
View(cleaned_trip_data)
```

### Create two new columns: "day_of_the_week", "ride_length" 
Note that 1 = Sunday and 7 = Saturday

1) We download ("lubricate") packages and load(dplyr)
2) Format dates into yyyymd Hms
3) Calculate ride_length as the difference between started_at and ended_at
4) Use wday function to arrange a number to the day of the week, 1 = Sunday in a new column named "days_of_the_week"

```{r}
install.packages("lubridate")

library(dplyr)

library(lubridate)

trip_data <- trip_data %>% 
  mutate(form_start_date = parse_date_time(started_at, orders = ("%y/%m/%d  %H:%M:%S")))

trip_data <- trip_data %>% 
  mutate(form_end_date = parse_date_time(ended_at, orders = ("%y/%m/%d %H:%M:%S")))

trip_data <- trip_data %>%
  mutate(ride_length=as.duration(form_start_date%--%form_end_date))

trip_data <- trip_data %>%
  mutate(day_of_the_week= lubridate::wday(trip_data$form_start_date))
```

Finally we created a new column named "day_of_the_week". Note that 1 = Sunday and 7 = Saturday 

## Run the following calculations:
### For both members and casual riders together and seperately

* Calculate the mean of ride_length
* Calculate the max ride_length
* Calculate the mode of day of week 



```{r}
trip_data %>% 
  summarise(average_ride_length = mean(ride_length)/60)

trip_data %>% 
  summarise(max_ride_length = max(ride_length)/60)

trip_data %>% 
  summarise(mode_day_of_the_week = table(day_of_the_week))

```

From these results we can see gain the following statistics for both members and casual riders.

average ride length across the dataset = 35.85 minutes
max ride length = 58720 minutes and the mode day of the week is Sunday with 17915 occurances 

## Filtering for differences between members and casual riders
### Casual riders

```{r}
trip_data %>% 
  filter(member_casual == "casual") %>%
  summarise(average_ride_length = mean(ride_length)/60)

trip_data %>% 
  filter(member_casual=="casual") %>%
  summarise(max_ride_length = max(ride_length)/60)

trip_data %>% 
  filter(member_casual=="casual") %>%
  summarise(mode_day_of_the_week = table(day_of_the_week))
```

Here we calculate the following statistics for casual riders

* average ride length = 73.1 minutes

* max ride length = 55684 minutes

* mode day of the week is Sunday with 6475 out of 23628 of casual riders choosing to use cyclistic on this day

### Member riders 
```{r}
trip_data %>% 
  filter(member_casual == "member") %>%
  summarise(average_ride_length = mean(ride_length)/60)

trip_data %>% 
  filter(member_casual=="member") %>%
  summarise(max_ride_length = max(ride_length)/60)

trip_data %>% 
  filter(member_casual=="member") %>%
  summarise(mode_day_of_the_week = table(day_of_the_week))
```

Here we calculate the following statistics for member riders

* average ride length = 21.46 minutes
* max ride length = 58720 minutes

* mode day of the week is Sunday with 11440 out of 61148 of member riders choosing to use cyclistic on this day. 

## Create new data frame 
### Containing variables for start date and time, ride length, type of rider and day of the week

```{r}
trip_data_trends <- trip_data %>%
  subset(select=c(member_casual, form_start_date, ride_length, day_of_the_week))
```
Add start_time column from cleaned_trip data dataset
```{r}
trip_data_trends$start_time <- cleaned_trip_data$start_time
```

# Export dataset to excel

```{r}
install.packages("rio")
library(rio)
export(trip_data_trends,"trip_data.xlsx")
```

Load excel data set. Select ride_length column and then go to Data --> text to columns and apply split with delimiter "s". Use this column to create a new column "ride_length (minutes)" by dividing previous column by 60.

#### Convert number dates to character string 

Select day_of_the_week column, CNTRL + H and replace numbers with days of the week. 1 = Sunday and 7 = Saturday.

Then import excel file into public tableau for the following visualizations.

## Proportion of different riders and their average ride length across the week

[Click here for data visualisation](https://public.tableau.com/app/profile/bleron.ireland/viz/CyclisticDataVizs_16300804436190/Dashboard1)

```{r}
avg_ride_length_casual <- c(72.7, 110.5, 74.9, 63.2, 84.4, 62.3, 58.9)
avg_ride_length_member <- c(25.9, 18.4, 20.0, 16.3, 21.7, 17.8, 26.7)

avg_ride_length <- data.frame(avg_ride_length_casual, avg_ride_length_member)

avg_ride_length %>%
  summarise(sd(avg_ride_length_casual),sd(avg_ride_length_member))
```
Measure standard deviations to show member riders tend to use the services on a more consistent basis through out the week spending similar with lower variability time riding using Cyclistic. 

Key findings:

* Member riders make up just over 70 % of all riders using Cyclistic bike sharing services.

* The average length of a casual ride length for a casual rider is more than 3x that of a member rider. 21.46 and 73.1 minutes respectively.

* Both types of riders spend the lowest amount of time using Cyclistic service on a Wednesday and Monday whilst the longest amount of time on average spent on bikes differ between riders. For annual members the longest times on average is spent on a Sunday and for a casual member it is Saturday 

* Member riders are more consistent with the amount of time they spend using Cyclistic services throughout the week than casual rider. 


## How the usage of Cyclistic service changes during the week and how this differs between members. 

[Click here for Visualisation](https://public.tableau.com/app/profile/bleron.ireland/viz/CyclisticDaysoftheweek/Dashboard2)

Key findings: 

* Both types of riders use Cyclistic bike services the most on a Sunday. 

* Member riders tend to use Cyclistic services greater on weekdays than casual riders do on weekdays.

* Wednesday has the lowest amounts of rides taking place for both types of riders. 

### Three recommendation to Cyclistic Bike-share

* Increase the amount of time riders can use your bike-share service for in their annual membership since data shows that casual members tend to use your service for on average over 3x as long per journey. 

* Introduce promotional offers exclusive annual members aimed at Sundays since it is the most popular day for casual riders encouraging them to switch for these exclusive benefits with over 27.4% of all rides taken by casual rider occurring on this day. 
 
* Increase prices for casual members using bikes longer than 25 minutes. Since data has shown that on average ride length is 73.1 minutes compared to 21.46 minutes for member riders. Encouraging them to get a better deal by switching to an annual membership. 
