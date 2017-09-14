Loading and preprocessing the data
----------------------------------

#### 1, First of all, the data is loaded.

    url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    temp <- tempfile()
    download.file(url,temp)
    unzip(temp, "activity.csv")
    dataset <- read.csv("activity.csv")
    unlink(temp)

#### 2, Then, processed into a format suitable.

The "data\_tbl" is the same data as the original but tbl format. The
"data\_date" is the data ignoring NA values.

    library(dplyr)

    data_tbl <- tbl_df(dataset)
    data_tbl$date <- as.Date(data_tbl$date)
    data_date <- data_tbl %>%
                    filter(steps != "NA") %>%
                    group_by(date) %>%
                    summarise( steps_sum = sum(steps)) %>%
                    print()

    ## # A tibble: 53 × 2
    ##          date steps_sum
    ##        <date>     <int>
    ## 1  2012-10-02       126
    ## 2  2012-10-03     11352
    ## 3  2012-10-04     12116
    ## 4  2012-10-05     13294
    ## 5  2012-10-06     15420
    ## 6  2012-10-07     11015
    ## 7  2012-10-09     12811
    ## 8  2012-10-10      9900
    ## 9  2012-10-11     10304
    ## 10 2012-10-12     17382
    ## # ... with 43 more rows

What is mean total number of steps taken per day?
-------------------------------------------------

#### 1, Make the histogram of the total number of steps taken each day.

    library(ggplot2)
    g <- qplot(data_date$steps_sum, 
               geom="histogram", 
               xlab="Total Number of Steps per Day")
    print(g)

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

#### 2, Calculate and report the mean and median total number of steps taken per day.

The "data\_sum" is the summrary of data\_date, contains the mean and
median total number of steps taken per day.

    data_sum <- data_tbl %>%
                    filter(steps != "NA") %>%
                    group_by(date) %>%
                    summarise(steps_mean = mean(steps),
                              steps_median = median(steps)) %>%
                    print()

    ## # A tibble: 53 × 3
    ##          date steps_mean steps_median
    ##        <date>      <dbl>        <dbl>
    ## 1  2012-10-02    0.43750            0
    ## 2  2012-10-03   39.41667            0
    ## 3  2012-10-04   42.06944            0
    ## 4  2012-10-05   46.15972            0
    ## 5  2012-10-06   53.54167            0
    ## 6  2012-10-07   38.24653            0
    ## 7  2012-10-09   44.48264            0
    ## 8  2012-10-10   34.37500            0
    ## 9  2012-10-11   35.77778            0
    ## 10 2012-10-12   60.35417            0
    ## # ... with 43 more rows

What is the average daily activity pattern?
-------------------------------------------

#### 1, Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

The "data\_int" is the data grouped by interval, ignoring NA values.

    data_int <- data_tbl %>%
                    filter(steps != "NA") %>%
                    group_by(interval) %>%
                    summarise(steps_mean = mean(steps)) %>%
                    print()

    ## # A tibble: 288 × 2
    ##    interval steps_mean
    ##       <int>      <dbl>
    ## 1         0  1.7169811
    ## 2         5  0.3396226
    ## 3        10  0.1320755
    ## 4        15  0.1509434
    ## 5        20  0.0754717
    ## 6        25  2.0943396
    ## 7        30  0.5283019
    ## 8        35  0.8679245
    ## 9        40  0.0000000
    ## 10       45  1.4716981
    ## # ... with 278 more rows

    plot(data_int$interval,data_int$steps_mean,
         type="l",
         xlab="5-minutes interval",
         ylab="Average Number of Steps Averaged across All Days")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-5-1.png)

#### 2,Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

At first, I need to get the maximum number of steps.

    summary(data_int$steps_mean)

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   0.000   2.486  34.110  37.380  52.830 206.200

Here, I see the maximum number of steps is 206.2.

Then I search the interval number which contains the number of steps
over 206.

    data_max <- filter(data_int,steps_mean > 206)
    data_max

    ## # A tibble: 1 × 2
    ##   interval steps_mean
    ##      <int>      <dbl>
    ## 1      835   206.1698

Here is the answer, the 835th interval contains the maximum number of
steps on average.

Imputing missing values
-----------------------

#### 1,Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

    sum(is.na(data_tbl$steps))

    ## [1] 2304

The total number of rows with NAs is 2304.

#### 2, Devise a strategy for filling in all of the missing values in the dataset.

I will use the mean of steps per day for replacing NAs. Some days have
no steps data (all steps data are NA), so if it is, 0 (zero) is
intoroduced.

First, I make a data table for prepare.

    data_4 <- left_join(data_tbl,data_sum, by="date")

Then, fill in the values. The "steps\_rep" is the new steps data filled
in all the missing values.

    data_rep <- data_4 %>%
            mutate( steps_rep = ifelse(is.na(steps),
                                  ifelse(is.na(steps_mean),0,steps_mean),
                                  steps)) %>%
            print()

    ## # A tibble: 17,568 × 6
    ##    steps       date interval steps_mean steps_median steps_rep
    ##    <int>     <date>    <int>      <dbl>        <dbl>     <dbl>
    ## 1     NA 2012-10-01        0         NA           NA         0
    ## 2     NA 2012-10-01        5         NA           NA         0
    ## 3     NA 2012-10-01       10         NA           NA         0
    ## 4     NA 2012-10-01       15         NA           NA         0
    ## 5     NA 2012-10-01       20         NA           NA         0
    ## 6     NA 2012-10-01       25         NA           NA         0
    ## 7     NA 2012-10-01       30         NA           NA         0
    ## 8     NA 2012-10-01       35         NA           NA         0
    ## 9     NA 2012-10-01       40         NA           NA         0
    ## 10    NA 2012-10-01       45         NA           NA         0
    ## # ... with 17,558 more rows

#### 3, Create a new dataset that is equal to the original dataset but with the missing data filled in.

The "data\_rep" contains the new steps data but some variables are
unnecessary now, so I make new dataset, selecting necessary variables
from "data\_rep".

    data_new <- data_rep %>%
                    select(steps_rep,date,interval) %>%
                    print()

    ## # A tibble: 17,568 × 3
    ##    steps_rep       date interval
    ##        <dbl>     <date>    <int>
    ## 1          0 2012-10-01        0
    ## 2          0 2012-10-01        5
    ## 3          0 2012-10-01       10
    ## 4          0 2012-10-01       15
    ## 5          0 2012-10-01       20
    ## 6          0 2012-10-01       25
    ## 7          0 2012-10-01       30
    ## 8          0 2012-10-01       35
    ## 9          0 2012-10-01       40
    ## 10         0 2012-10-01       45
    ## # ... with 17,558 more rows

#### 4, Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.

The same way as the first part.

    data_date_new <- data_new %>%
            group_by(date) %>%
            summarise( steps_sum = sum(steps_rep)) %>%
            print()

    ## # A tibble: 61 × 2
    ##          date steps_sum
    ##        <date>     <dbl>
    ## 1  2012-10-01         0
    ## 2  2012-10-02       126
    ## 3  2012-10-03     11352
    ## 4  2012-10-04     12116
    ## 5  2012-10-05     13294
    ## 6  2012-10-06     15420
    ## 7  2012-10-07     11015
    ## 8  2012-10-08         0
    ## 9  2012-10-09     12811
    ## 10 2012-10-10      9900
    ## # ... with 51 more rows

    g <- qplot(data_date_new$steps_sum, 
               geom="histogram", 
               xlab="Total Number of Steps per Day")
    print(g)

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-12-1.png)

#### Do these values differ from the estimates from the first part of the assignment?

-&gt; Yes, they do.

#### What is the impact of imputing missing data on the estimates of the total daily number of steps?

-&gt; There is one peak in the histogram from the first part. This
suggests the data have taken from one population.

On the other hand, there are two peaks in the histogram from this part.

The second histogram suggests that the data contains at least 2 groups
which have different features from each other.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

#### 1,Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

    Sys.setlocale("LC_TIME","English")

    ## [1] "English_United States.1252"

    weekends <- c("Sunday","Saturday")
    data_new <- data_new %>%
            mutate(weekday=weekdays(data_new$date),
                   week = ifelse(weekday %in% weekends,"weekend","weekday")) %>%
            print()

    ## # A tibble: 17,568 × 5
    ##    steps_rep       date interval weekday    week
    ##        <dbl>     <date>    <int>   <chr>   <chr>
    ## 1          0 2012-10-01        0  Monday weekday
    ## 2          0 2012-10-01        5  Monday weekday
    ## 3          0 2012-10-01       10  Monday weekday
    ## 4          0 2012-10-01       15  Monday weekday
    ## 5          0 2012-10-01       20  Monday weekday
    ## 6          0 2012-10-01       25  Monday weekday
    ## 7          0 2012-10-01       30  Monday weekday
    ## 8          0 2012-10-01       35  Monday weekday
    ## 9          0 2012-10-01       40  Monday weekday
    ## 10         0 2012-10-01       45  Monday weekday
    ## # ... with 17,558 more rows

#### 2, Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

    data_week <- data_new %>%
                    group_by(week,interval) %>%
                    summarise(steps_mean = mean(steps_rep)) %>%
                    print()

    ## Source: local data frame [576 x 3]
    ## Groups: week [?]
    ## 
    ##       week interval steps_mean
    ##      <chr>    <int>      <dbl>
    ## 1  weekday        0 2.02222222
    ## 2  weekday        5 0.40000000
    ## 3  weekday       10 0.15555556
    ## 4  weekday       15 0.17777778
    ## 5  weekday       20 0.08888889
    ## 6  weekday       25 1.31111111
    ## 7  weekday       30 0.62222222
    ## 8  weekday       35 1.02222222
    ## 9  weekday       40 0.00000000
    ## 10 weekday       45 1.60000000
    ## # ... with 566 more rows

    library(lattice)

    xyplot(steps_mean ~ interval | week,
           data = data_week, layout=c(1,2),type="l",
           xlab = "Interval",
           ylab = "Number of Steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-14-1.png)
