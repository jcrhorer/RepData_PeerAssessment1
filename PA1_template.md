# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
  
The data file can be obtained through the course github account.  Use the following code to load the data file:


```r
activity <- read.csv("~/Coursera_RepData_PA1/activity.csv")
```



## What is mean total number of steps taken per day?
The first step to address this question is to calculate the sum of steps by day using the *aggregate()* function


```r
steps.bydate <- aggregate(steps ~ date, data = activity, FUN = sum)
```

Next, we build a historgram of the steps by day data.  For this exercise, we will use the *hist()* function.

```r
hist(steps.bydate$steps)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

  
Lastly, to find the mean and the median of the data, we use the *mean()* and *median()* functions, respectively.

```r
mean(steps.bydate$steps)
```

```
## [1] 10766
```

```r
median(steps.bydate$steps)
```

```
## [1] 10765
```

  
  
## What is the average daily activity pattern?
First, to address this question we need to calculate the average number of steps by interval.  We can do this by using the *aggregate()* function.

```r
interval.avgsteps <- aggregate(steps ~ interval, data = activity, FUN = mean)
```

Once we have the data summarized, we can build a time series plot using the *plot()* function.  In addition, we can draw a line at the highpoint of the chart by using the *abline()* function coupled with the *max()* function.  Lastly, we can add a label that contains the interval with the highest average number of steps by using the *text()* function.

```r
plot(interval.avgsteps, type = "l")
abline(h = max(interval.avgsteps$steps), col = "red", lwd = 1)
text(1500, 190, paste("busiest interval:", interval.avgsteps$interval[interval.avgsteps$steps == 
    max(interval.avgsteps$steps)], sep = "  "), cex = 0.8, col = "red")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

  
  
## Imputing missing values
This element of the assignment is a multi-step process. 
  
First, we need to separate the rows with NAs from those without NAs.  We do this using the *is.na()* function.

```r
activity.NAs <- activity[is.na(activity$steps), ]
activity.noNAs <- activity[!is.na(activity$steps), ]
```


Second, we need to get suitable values to replace the NAs with.  In order to do that, we will take the average number of steps per interval.


```r
interval.avgsteps <- aggregate(steps ~ interval, data = activity, FUN = mean)
interval.avgsteps$steps <- as.integer(interval.avgsteps$steps)
```

Then, we will replace the NAs with the averages from the same interval. We will use the *merge()* function for this part of the process.  We also cleanup the column title of the new step data and finally get rid of the NAs using the *subset()* function. 

```r
activity.replNAs <- merge(activity.NAs, interval.avgsteps, by = "interval")
activity.replNAs$steps <- activity.replNAs$steps.y
activity.replNAs <- subset(activity.replNAs, select = c("steps", "date", "interval"))
```

Lastly, we will stitch together the newly imputed data set with the subset without NAs using the *rbind()* function.

```r
activity.clean <- rbind(activity.noNAs, activity.replNAs)
```

  
To plot the data, we summarize the data using *aggregate()* then use the *hist()* function.

```r
dailysteps.activity <- aggregate(steps ~ date, data = activity.clean, FUN = sum)
hist(dailysteps.activity$steps)
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 

```r
mean(dailysteps.activity$steps)
```

```
## [1] 10750
```

```r
median(dailysteps.activity$steps)
```

```
## [1] 10641
```

Imputing data has changed the values a bit; the mean has shifted a bit lower and the median has shifted lower by a larger margin.
  
  
## Are there differences in activity patterns between weekdays and weekends?
First, we need to cleanup the data by setting the "date" column to a date format.

```r
activity.clean$date <- as.Date(as.character(activity.clean$date), format = "%Y-%m-%d")
```

  
Then, we need to distinguish between weekdays and the weekend.  We do this by using the *weekdays()*,*factor()*, and *level()* functions.

```r
activity.clean$weekday <- weekdays(activity.clean$date, abbreviate = TRUE)
activity.clean$daytype <- factor(activity.clean$weekday)
levels(activity.clean$daytype) <- list(weekend = c("Sat", "Sun"), weekday = c("Mon", 
    "Tue", "Wed", "Thu", "Fri"))
```

  
Next, we subset the data by whether or not the day is a weekday or weekend day.

```r
steps.weekday <- activity.clean[activity.clean$daytype == "weekday", ]
steps.weekend <- activity.clean[activity.clean$daytype == "weekend", ]
```

  
In order to get the right results, we summarize each of these data sets using the *aggregate()* function.

```r
avgsteps.weekday <- aggregate(steps ~ interval, data = steps.weekday, FUN = mean)
avgsteps.weekend <- aggregate(steps ~ interval, data = steps.weekend, FUN = mean)
```

  
Lastly, we plot the 2 data sets in the same window using rh *par()* function in conjunction with *plot()*.

```r
par(mfrow = c(2, 1), mar = c(4, 4, 2, 1))

plot(avgsteps.weekday$interval, avgsteps.weekday$steps, main = "Weekday average steps by interval", 
    type = "l", xlab = "interval", ylab = "avg steps")

plot(avgsteps.weekend$interval, avgsteps.weekend$steps, main = "Weekend average steps by interval", 
    type = "l", xlab = "interval", ylab = "avg steps")
```

![plot of chunk unnamed-chunk-16](figure/unnamed-chunk-16.png) 

