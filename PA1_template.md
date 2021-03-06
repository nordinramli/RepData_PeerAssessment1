# Reproducible Research: Peer Assessment 1
### *By Nordin Ramli*  
This is the assignment report on Reproducible Research: Peer Assessment 1  

The assignment is to analyze the personal activity monitoring device that collects data at each 5 minute intervals through out the day. The data consists of two months of ata from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

##Data
The data for this assignment can be downloaded from the course web site:

Dataset:[Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
- date: The date on which the measurement was taken in YYYY-MM-DD format
- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset (61 days * 288 time intervals per day).

###Preparation the R environment
In order to analyze the data, the R environment is required to load the necessary libraries. Here, the code chunks by using *echo=TRUE* is used in order to allow others to review the code.


```r
library(knitr)
```

```
## Warning: package 'knitr' was built under R version 3.3.2
```

```r
opts_chunk$set(echo = TRUE, results = 'hold')
```
## Loading and preprocessing the data
Next, unzip the data *activity.zip* that being downloaded from the repository. The summary of the unzipped data is given as the following:


```r
unzip(zipfile="activity.zip")
data <- read.csv("activity.csv")
summary(data)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

## What is mean total number of steps taken per day?
The total number of steps taken per day is calculated and the result is illustrated in the histogram as follows:


```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.3.2
```

```r
total.steps <- tapply(data$steps, data$date, FUN=sum, na.rm=TRUE)
   qplot(total.steps,binwidth=1000,main="Histogram on total number of steps taken per day",xlab="total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

The mean and median of the total number of steps taken per day is given as follows:

```r
mean(total.steps, na.rm=TRUE)
median(total.steps, na.rm=TRUE)
```

```
## [1] 9354.23
## [1] 10395
```

## What is the average daily activity pattern?
The next question to answer is within each recorded interval across all days in each of the two months, what was the average number of steps taken in each interval? 


```r
library(ggplot2)
averages <- aggregate(x=list(steps=data$steps), by=list(interval=data$interval),
                      FUN=mean, na.rm=TRUE)
```

We make the plot with the time series of the average number of steps taken (averaged across all days) versus the 5-minute intervals:

```r
ggplot(data=averages, aes(x=interval, y=steps)) +
        geom_line(color="orange", size=1) +
         ggtitle("Comparison of the average number of steps across all days")+
        xlab("5-minute interval") +
        ylab("average number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

The maximum value of average number of steps taken is given as follows:

```r
averages[which.max(averages$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```

## Imputing missing values

```r
missing <- is.na(data$steps)
# How many missing
table(missing)
```

```
## missing
## FALSE  TRUE 
## 15264  2304
```

The total number of missing values are 2304.

# Replace each missing value with the mean value of its 5-minute interval
In order to reduce the bias in the calculations, we replace each missing value with the mean value of its 5-minute interval as follows:

```r
fill.value <- function(steps, interval) {
        filled <- NA
        if (!is.na(steps))
                filled <- c(steps)
        else
                filled <- (averages[averages$interval==interval, "steps"])
        return(filled)
}
filled.data <- data
filled.data$steps <- mapply(fill.value, filled.data$steps, filled.data$interval)
```
A new dataset is procuded after the above process. We check that are there any missing values remaining or not as follows:

```r
sum(is.na(filled.data$steps))
```

```
## [1] 0
```
It is shown that the missing part is none.

Based on the new dataset, a histogram of the total number of steps taken each day is illustrated as follows:

```r
total.steps <- tapply(filled.data$steps, filled.data$date, FUN=sum)
qplot(total.steps,binwidth=1000,main="Histogram on total number of steps taken per day", xlab="total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

The mean and median value are calculated as follows:

```r
mean(total.steps)
median(total.steps)
```

```
## [1] 10766.19
## [1] 10766.19
```

It shows that both the value of median and mean are identical. However, the new value of mean is differ from the previous estimated value. It is proven that by imputing missing data, we are able to reduce the bias in calculations.

## Are there differences in activity patterns between weekdays and weekends?
Next, in order to separate the data into weekday and weekend subsets for analysis, the date variable is input to the weekdays method and that output is checked against a vector of weekend day names.


```r
weekday.or.weekend <- function(date) {
        day <- weekdays(date)
        if (day %in% c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"))
                return("weekday")
        else if (day %in% c("Saturday", "Sunday"))
                return("weekend")
        else
                stop("invalid date")
}
filled.data$date <- as.Date(filled.data$date)
filled.data$day <- sapply(filled.data$date, FUN=weekday.or.weekend)
```


```r
averages <- aggregate(steps ~ interval + day, data=filled.data, mean)
```

Below is the panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends:


```r
ggplot(averages, aes(interval, steps)) + geom_line(color="orange", size=1) + facet_grid(day ~ .) +
        ggtitle("Comparison of the average number of steps across weekdays and weekends")+
        xlab("5-minute interval") + ylab("Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->


### Observation  
We can see at the graph above that activity on the weekday has the greatest peak from all steps intervals. But, we can see too that weekends activities has more peaks over a hundred than weekday. This could be due to the fact that activities on weekdays mostly follow a work related routine, where we find some more intensity activity in little a free time that the employ can made some sport. In the other hand, at weekend we can see better distribution of effort along the time.
