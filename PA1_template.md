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


## Plot time series of steps per interval
In the following figure I show the average of steps per 5 minutes
interval across all dates.
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

```
## [1] 2304
```
## Fill in NAs with generated values
To fill the NAs with other values I use the following strategy:

If an NA value is found the interval it occurs in is replaced by the mean over the steps per this respective interval over all days.



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


We can see that activity is more distributed in a homogenous way on the weekend case. Here, the person is more active over the whole day. During the week, a peak occurs in the morning followed by a long period of low activity and a small one at the end of the day.

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png) 

