R Markdown
----------

<br> This assignment makes use of data from a personal activity
monitoring device. This device collects data at 5 minute intervals
through out the day.

The data consists of two months of data from an anonymous individual
collected during the months of October and November, 2012 and include
the number of steps taken in 5 minute intervals each day.

The variables included in this dataset are:

-   steps: Number of steps taking in a 5-minute interval (missing values
    are coded as NA)
-   date: The date on which the measurement was taken in YYYY-MM-DD
    format
-   interval: Identifier for the 5-minute interval in which measurement
    was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset. <br> \#\# Loading
and preprocessing the data

### 1. Load the data

    D <- read.csv(file="repdata_data_activity.csv",header=T)
    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.5.3

### 2. Process/transform the data (if necessary) into a format suitable for your analysis

    str(D)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

    summary(D)

    ##      steps                date          interval     
    ##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
    ##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
    ##  Median :  0.00   2012-10-03:  288   Median :1177.5  
    ##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
    ##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
    ##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
    ##  NA's   :2304     (Other)   :15840

    head(D)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

What is mean total number of steps taken per day?
-------------------------------------------------

### 1. Calculate the total number of steps taken per day

    StepsPerDay <- aggregate(D$steps, list(D$date), FUN=sum)
    head(StepsPerDay,3)

    ##      Group.1     x
    ## 1 2012-10-01    NA
    ## 2 2012-10-02   126
    ## 3 2012-10-03 11352

    colnames(StepsPerDay)<-c("Date","StepsDuringThatDay")
    head(StepsPerDay,3)

    ##         Date StepsDuringThatDay
    ## 1 2012-10-01                 NA
    ## 2 2012-10-02                126
    ## 3 2012-10-03              11352

    G <- ggplot(StepsPerDay, aes(x=StepsDuringThatDay)) + geom_histogram(boundary=0, binwidth=2500, col="darkblue", fill="steelblue")+ggtitle("Steps per day")+xlab("Steps")+ylab("Frequency") + theme(plot.title = element_text(face="bold", size=12)) + scale_x_continuous(breaks=seq(0,25000,2500)) + scale_y_continuous(breaks=seq(0,18,2))
    print(G)

    ## Warning: Removed 8 rows containing non-finite values (stat_bin).

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

### 2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

N/A

### 3. Calculate and report the mean and median of the total number of steps taken per day

The mean:

    mean(StepsPerDay$StepsDuringThatDay,na.rm=T)

    ## [1] 10766.19

The median:

    median(StepsPerDay$StepsDuringThatDay,na.rm=T)

    ## [1] 10765

What is the average daily activity pattern?
-------------------------------------------

### 1. Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

    head(D$steps)

    ## [1] NA NA NA NA NA NA

    head(D$days)

    ## NULL

    head(D)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

    five_min_mean <- aggregate(D$steps, by=list(D$interval), FUN=mean, na.rm=TRUE)
    head(five_min_mean)

    ##   Group.1         x
    ## 1       0 1.7169811
    ## 2       5 0.3396226
    ## 3      10 0.1320755
    ## 4      15 0.1509434
    ## 5      20 0.0754717
    ## 6      25 2.0943396

    plot(five_min_mean, type = "l", xlab="min", ylab="num of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png) The
interval with the maximum steps over all days is calculated.

    max_mean <- five_min_mean[five_min_mean$x == max(five_min_mean$x),]
    max_mean

    ##     Group.1        x
    ## 104     835 206.1698

Imputing missing values
-----------------------

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

The numbers of NA rows for steps are:

    sum(is.na(D$steps))

    ## [1] 2304

(All of these are missing data!)

Those rows with NA could be replaced by the mean of the coresponding
interval (which has been calculated in the above step).

    merged <- merge(x=D, y=five_min_mean, by.x="interval", by.y="Group.1")
    my.na <- is.na(merged$steps)
    merged[my.na, 2] <- merged[my.na, 4]
    D2 <- data.frame(steps=merged$steps, date=merged$date, interval=merged$interval)
    five_min_mean <- aggregate(D2$steps, by=list(D2$interval), FUN=mean)
    plot(five_min_mean$x, type = "l", xlab = "minutes", ylab = "number of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-9-1.png) The
mean and median for the new data set.

    daily_steps <- aggregate(D2$steps, by=list(D2$date), FUN=sum);
    mean(daily_steps$x,na.rm=T);

    ## [1] 10766.19

    median(daily_steps$x,na.rm=T);

    ## [1] 10766.19

The median has changed.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

For this part the weekdays() function may be of some help here. Use the
dataset with the filled-in missing values for this part.

### 1.Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

### 2.Make a panel plot containing a time series plot (i.e. type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

New column weekday and weekend.

    library(timeDate)

    ## Warning: package 'timeDate' was built under R version 3.5.2

    D2 = within(D2, {
        daytype = as.factor(ifelse(isWeekday(D2$date), "weekday", "weekend"))
     })
    str(D2)

    ## 'data.frame':    17568 obs. of  4 variables:
    ##  $ steps   : num  1.72 0 0 0 0 ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 54 28 37 55 46 20 47 38 56 ...
    ##  $ interval: int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ daytype : Factor w/ 2 levels "weekday","weekend": 1 1 2 1 2 1 2 1 1 2 ...

2 plots: weekdays AVG and weekends AVG.

    par(mfrow = c(2,1))
    X <- split(D2, D2$daytype)
    weekday_interval_means <- aggregate(X[[1]]$steps, by=list(X[[1]]$interval), FUN=mean)
    weekend_interval_means <- aggregate(X[[2]]$steps, by=list(X[[2]]$interval), FUN=mean)
    plot(weekday_interval_means$x, type = "l", xlab = "minutes", ylab = "#_steps", main = "Weekdays")
    plot(weekend_interval_means$x, type = "l", xlab = "minutes", ylab = "#_steps", main = "Weekends")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-12-1.png)

On weekends one seems to be definitely more active than on the weekdays.
Who would've guessed?
