---
title: "Reproducible Research Assignment 1"
output:
  html_document:
    keep_md: TRUE
---




## Loading and preprocessing the data

```r
fit_data<-read.csv(file="C:/Users/m114403/Documents/Learning/class_data/repdata_data_activity/activity.csv",header=TRUE)
str(fit_data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

## What is mean total number of steps taken per day?

```r
step_by_day<- fit_data %>%
  group_by(date) %>%
  summarise(total_step=sum(steps))

hist(step_by_day$total_step,xlab="Total Steps Every Day",main="Total Steps by Day")
```

![](PA1_template_files/figure-html/fit_data-1.png)<!-- -->

```r
mean(step_by_day$total_step,na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(step_by_day$total_step,na.rm = TRUE)
```

```
## [1] 10765
```
**Mean is 10766, Median is 10765**

## What is the average daily activity pattern?

```r
#Calculate average number of steps within each interval across all days
step_by_interval<-fit_data %>%
  group_by(interval) %>%
  summarise(mean_steps=mean(steps,na.rm=TRUE))
#Time series plot
plot(x=step_by_interval$interval,step_by_interval$mean_steps,type="l",xlab="interval",ylab="average number of steps",main="average number of steps by interval",lwd=2)
abline(v=step_by_interval$interval[which.max(step_by_interval$mean_steps)])
```

![](PA1_template_files/figure-html/daily pattern-1.png)<!-- -->

```r
##Find interval with maximum average steps
step_by_interval$interval[which.max(step_by_interval$mean_steps)]
```

```
## [1] 835
```

```r
##calculate average steps within that interval
max(step_by_interval$mean_steps)
```

```
## [1] 206.1698
```
The interval with maximum average steps is 835, with an average steps of 206.

## Imputing missing values

```r
##Total Number of Missing values
sum(is.na(fit_data))
```

```
## [1] 2304
```

```r
####Fill in missing values with average of that interval
impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))
fit_data_imputed<- fit_data %>%
  group_by(interval) %>%
  mutate(steps_imputed=impute.mean(steps))
```

```
## Warning: package 'bindrcpp' was built under R version 3.5.1
```

```r
sum(is.na(fit_data_imputed$steps_imputed))
```

```
## [1] 0
```

```r
####Create a histogram of imputed steps of each day
step_by_day_imputed<- fit_data_imputed %>%
  group_by(date) %>%
  summarise(total_step=sum(steps_imputed))

hist(step_by_day_imputed$total_step,xlab="Total Steps Every Day",main="Total Steps by Day (with missing value imputed)")
```

![](PA1_template_files/figure-html/missing value-1.png)<!-- -->

```r
mean(step_by_day_imputed$total_step)
```

```
## [1] 10766.19
```

```r
median(step_by_day_imputed$total_step)
```

```
## [1] 10766.19
```
The mean and median of the imputed total steps taken per day are both 10766.19


## Are there differences in activity patterns between weekdays and weekends?

```r
##Create an indicator for Weekdays ###
fit_data_imputed$wkday<-ifelse(weekdays(as.Date(fit_data_imputed$date)) %in% c("Saturday","Sunday"),0,1)
##Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps ##taken, averaged across all weekday days or weekend days (y-axis)

plot_wkday_data<-fit_data_imputed %>%
  group_by(interval,wkday) %>%
  summarise(avg_step=mean(steps_imputed))
plot_wkday_data$wkday<-factor(plot_wkday_data$wkday,levels=c(0,1),labels=c("weekend","weekday"))
#par(mar=c(3,3,3,0))
ggplot(data=plot_wkday_data,mapping=aes(x=interval,y=avg_step))+geom_line()+facet_grid(wkday~.) +ggtitle("average steps within each interval")+ylab("average steps")
```

![](PA1_template_files/figure-html/difference in activity pattern-1.png)<!-- -->
On Weekdays, activity starts earlier in the day and during the day we see lower activity rate compared to weekend.
