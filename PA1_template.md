---
title: "Reproducible Research: Peer Assessment 1"
output: 
html_document:
keep_md: true
---
# Reproducible Research: Peer Assessment 1
This project demonstrates the use of the knitr package in aid of reproducible research. It is a required project for the [Reproducible Research](https://www.coursera.org/learn/reproducible-research/home) course hosted on Coursera.

## Loading and preprocessing the data
The "activity.zip" that contains the data is included in the GIT repository. Unzip the file if needed.

```r
if (!file.exists("activity.csv")) {
    unzip("activity.zip")
}
```
Read the CSV file. Convert the data frame object to a dplyr data frame. Set the data types of the columns appropriately.

```r
d = read.csv("./activity.csv",
             header = TRUE,
             stringsAsFactors = FALSE)
library(dplyr)
d <- tbl_df(d)
d$steps <- as.numeric(d$steps)
d$date <- as.Date(d$date)
d$interval <- as.numeric(d$interval)
```

## What is mean total number of steps taken per day?
Summarise the data to average the total number of steps per day. Ignore missing values.

```r
by_date <- d %>% 
    na.omit() %>% 
    group_by(date) %>% 
    summarise(total=sum(steps))
```
Plot a histogram of the total number of steps taken each day. We are using the ggplot2 package for plotting in this report.

```r
library(ggplot2)
qplot(by_date$total,binwidth=5000)
```

![plot of chunk hist_total_steps_per_day](figure/hist_total_steps_per_day-1.png)

Calculate the mean and median total number of steps per day.

```r
mean_total_steps_per_day <- mean(by_date$total)
median_total_steps_per_day <- median(by_date$total)
```
The mean total number of steps per day is: 1.0766189 &times; 10<sup>4</sup>. The median total number of steps per day is: 1.0765 &times; 10<sup>4</sup>.
## What is the average daily activity pattern?
Calculate the average number of steps taken, per interval, averaged across all days.

```r
by_interval <- d %>% 
            na.omit() %>% 
            group_by(interval) %>% 
            summarise(mean_steps=mean(steps))
```
Plot the 5-minute intervals vs. the average number of steps.

```r
qplot(by_interval$interval,by_interval$mean_steps,data=by_interval, geom="line")
```

![plot of chunk plot_steps_by_interval](figure/plot_steps_by_interval-1.png)

Calculate the interval, on average across all days, that contains the maximum number of steps.

```r
interval_with_max_steps <- by_interval$interval[which.max(by_interval$mean_steps)]
```
The ``835``<sup>th</sup> interval contains the maximum number of steps, on average, across all days.
## Imputing missing values
Calculate the total number of rows with missing values. 

```r
total_num_of_na_rows <- sum(is.na(d$steps)|is.na(d$date)|is.na(d$interval))
```
The total number of rows with missing values is ``2304``. Investigating the individual columns of the data reveals that only the steps column contains missing data.

Given that the average number of steps varies greatly across the intervals, use the mean of the corresponding 5-minute interval to impute the value of missing steps entries.

Create a new dataset that substitutes missing values with their imputed values. 

```r
d_no_nas <- d
# TODO: Find a better method to update the missing values. 
# Tried the dplyr mutate with a case_when, but ran into an NA issue that I could not resolve.
for (i in 1:nrow(d)){
    if (is.na(d_no_nas$steps[i])){
        int_val <- d_no_nas$interval[i]
        avg_steps <- by_interval$mean_steps[ by_interval$interval ==int_val]
        d_no_nas$steps[i] <- avg_steps
    }
}
```
Calculate the total number of steps per day using the imputed dataset. Also calculate the mean and median total number of steps per day.

```r
by_date_no_nas <- d_no_nas %>% 
    group_by(date) %>% 
    summarise(total=sum(steps))
mean_total_steps_per_day_no_nas <- mean(by_date_no_nas$total)
median_total_steps_per_day_no_nas <- median(by_date_no_nas$total)
```
The new mean total number of steps per day is ``1.0766189 &times; 10<sup>4</sup>`` as compared to the previous value of ``1.0766189 &times; 10<sup>4</sup>``. The new median total number of per day is ``1.0766189 &times; 10<sup>4</sup>`` as compared to the previous value of ``1.0765 &times; 10<sup>4</sup>``.

Plot a histogram of the total number of steps taken each day using the new dataset.

```r
qplot(by_date_no_nas$total,binwidth=5000)
```

![plot of chunk plot_imputed_total_steps](figure/plot_imputed_total_steps-1.png)

The imputed data has resulted in the number of days where the number of steps falls in the [7500, 17500] interval having increased significantly.

## Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
d_day_type <- d_no_nas %>%
    mutate(day_type = case_when(
        weekdays(.$date,abbreviate=TRUE) %in% c("Mon","Tue","Wed","Thu","Fri") ~ "weekday",
        weekdays(.$date,abbreviate=TRUE) %in% c("Sat","Sun") ~ "weekend")) 
```
Calculate the average number of steps taken, averaged across all weekdays or weekend days, grouped by intervals.

```r
by_day_type <- d_day_type %>%
    group_by(day_type,interval) %>%
    summarise(avg_steps=mean(steps)) %>%
    select (day_type,interval,avg_steps)
```
Create a panel plot to depict the average number of steps per interval, comparing the weekend and weekday activity patterns.

```r
qplot(x=interval,y=avg_steps,data=by_day_type,facets = day_type~.,geom="line") 
```

![plot of chunk plot_day_type](figure/plot_day_type-1.png)

Clearly, there was more activity during the weekdays than the weekends. Also, while the activity peaks early in the day during weekdays, during weekends, the activity appears to be evenly spread between the [750, 2000] interval range.
