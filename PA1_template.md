# Activity Monitoring Data



## Switching to working directory and loading the csv
We set the working directory.
We read in the activity.csv.
The date column has to be transformed to the Date type.

```r
setwd(wd)
df <- read.csv("activity.csv", sep = ",", header = TRUE, na.strings = "NA")
df$date <- as.Date(df$date, "%Y-%m-%d")
```
Calculate the sum of the steps per day. All log entries from the same day
are summed up. 

```r
steps_per_day <- tapply(df$steps, format(df$date, '%Y-%m-%d'), sum) 
```
## Histogram of the steps per day
In this section we show the histogram of the steps logged per day. 

```r
qplot(steps_per_day, geom="histogram") 
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 


```r
# Calculate mean
steps_mean <- mean(steps_per_day, na.rm = TRUE)
print(steps_mean)
```

```
## [1] 10766.19
```

```r
# Calculate median
steps_median <- median(steps_per_day, na.rm = TRUE)
print(steps_median)
```

```
## [1] 10765
```


```r
# Calculate steps per interval
df_cmp <- df[complete.cases(df),]
steps_per_interval <- tapply(df_cmp$steps, df_cmp$interval, mean)
steps_per_interval <- aggregate(df_cmp$steps, by=list(df_cmp$interval), mean)
```
## Plot time series of steps per interval
In the following figure I show the average of steps per 5 minutes
interval across all dates.

```r
names(steps_per_interval)[names(steps_per_interval) == 'Group.1'] <- 'Interval'
g <- ggplot(steps_per_interval, aes(x = Interval, y = x))
g2 <- g + geom_line(color = "steelblue", size = 1)
print(g2)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 

## Maximum number of steps in interval

In the following I determine the interval with the maximum number of steps:


```r
stp_int_ord <- steps_per_interval[with(steps_per_interval, order(-x)), ]
print(stp_int_ord[1,1])
```

```
## [1] 835
```

## Count the numbers of rows containing NA

```r
sum(is.na(df))
```

```
## [1] 2304
```
## Fill in NAs with generated values
To fill the NAs with other values I use the following strategy:

If an NA value is found the interval it occurs in is replaced by the mean over the steps per this respective interval over all days.


```r
# Fill NAs with means from steps_per_interval: 
# 
df_filled <- df
df_filled$steps <- ifelse(is.na(df$steps), steps_per_interval$x[match(df$interval, steps_per_interval$Interval)], df$steps)
```

## Recalculate the steps per day
Here I recalculate the sum of the steps per day with the dataframe in which the NAs have been replaced. Then I show the corresponding histogram of that data and calculate again the mean and the median.

```r
steps_per_day_filled <- tapply(df_filled$steps, format(df_filled$date, '%Y-%m-%d'), sum) 
qplot(steps_per_day_filled, geom="histogram") 
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 

```r
steps_mean_filled <- mean(steps_per_day_filled, na.rm = TRUE)
print(steps_mean_filled)
```

```
## [1] 10766.19
```

```r
steps_median_filled <- median(steps_per_day_filled)
print(steps_median_filled)
```

```
## [1] 10766.19
```
The values for mean and median do not deviate from the above values very much. The impact of imputing missing values with this strategy is rather low in this case.

# Compare weekdays with weekend
Here I want to compare the profile of steps per interval at weekdays with that during the week. 

```r
df_filled$we <- ifelse(wday(df_filled$date) > 5, 1, 0)


steps_per_interval_f <- tapply(df_filled$steps, df_filled$interval, mean)
steps_per_interval_f <- aggregate(df_filled$steps, by=c(list(df_filled$interval), list(df_filled$we)), mean)
names(steps_per_interval_f)[names(steps_per_interval_f) == 'Group.1'] <- 'Interval'
names(steps_per_interval_f)[names(steps_per_interval_f) == 'Group.2'] <- 'Weekend'
names(steps_per_interval_f)[names(steps_per_interval_f) == 'x'] <- 'Steps'

steps_per_interval_f$Weekend[steps_per_interval_f$Weekend == 0] <- "No weekend"
steps_per_interval_f$Weekend[steps_per_interval_f$Weekend == 1] <- "Weekend"

pg <- ggplot(steps_per_interval_f, aes(Interval, Steps)) + geom_line(color = "steelblue", size = 1) + geom_point() + 
  facet_wrap(~Weekend, ncol = 1)
```

We can see that activity is more distributed in a homogenous way on the weekend case. Here, the person is more active over the whole day. During the week, a peak occurs in the morning followed by a long period of low activity and a small one at the end of the day.


```r
print(pg)
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png) 

