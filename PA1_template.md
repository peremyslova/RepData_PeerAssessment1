# Reproducible Research: Peer Assessment 1

###Step1: Loading and preprocessing the data

Preparing the environment for futher R Markdown file processing:

```r
require(knitr)
```

```
## Loading required package: knitr
```

```r
opts_chunk$set(echo = TRUE, cache = TRUE, cache.path = "cache/", fig.path = "figure/")
```

Unzipping the archive with the data and loading it to create the data frame:

```r
unzip("activity.zip", files = NULL, list = FALSE, overwrite = TRUE,junkpaths = FALSE, exdir = ".", unzip = "internal",setTimes = FALSE)
activity_df <- read.csv("activity.csv", header = TRUE,sep =",",stringsAsFactors = FALSE)
```

Checking what type of variables do we have:

```r
names(activity_df)
```

```
## [1] "steps"    "date"     "interval"
```

Checking how many observations are there and what type of data do we have:

```r
str(activity_df)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

And let's look at the summary of data:

```r
summary(activity_df)
```

```
##      steps            date              interval     
##  Min.   :  0.00   Length:17568       Min.   :   0.0  
##  1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
##  Median :  0.00   Mode  :character   Median :1177.5  
##  Mean   : 37.38                      Mean   :1177.5  
##  3rd Qu.: 12.00                      3rd Qu.:1766.2  
##  Max.   :806.00                      Max.   :2355.0  
##  NA's   :2304
```

It looks like the data is tidy so we don't need to do any additional pre-processing. But some records are missing the steps data. Let's see how many of such records do we have:

```r
sum(is.na(as.numeric(activity_df$steps)))
```

```
## [1] 2304
```

As we can see, there is about one eighth of observations that has missing values for the number of steps but we will decide how to replace missing values later.

For now, we will just remove all incomplete cases:

```r
df_cc<-activity_df[complete.cases(activity_df),]
str(df_cc)
```

```
## 'data.frame':	15264 obs. of  3 variables:
##  $ steps   : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ date    : chr  "2012-10-02" "2012-10-02" "2012-10-02" "2012-10-02" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```
As a result, the dimension of the original dataframe has been reduced.

###Step2: Calculating the mean number of steps taken per day

For this part of the assignment, we will ignore the missing values in the dataset.

```r
df_mean<-aggregate(as.numeric(df_cc$steps),list(df_cc$date),mean)
head(df_mean[,2])
```

```
## [1]  0.43750 39.41667 42.06944 46.15972 53.54167 38.24653
```
We could have done this calculation on the original dataset (witout omitting the NAs) and we would have received the same result (one could try applying the same code to the original data set activity_df). But in this case a warning message saying that NAs have been introduced by coercion would have appeared.


Calculating the median number of steps taken per day. For this part of the assignment, we will also ignore the missing values in the dataset, i.e. apply this operation to the clean dataset.

```r
df_median<-aggregate(as.numeric(df_cc$steps),list(df_cc$date),median)
head(df_median[,2])
```

```
## [1] 0 0 0 0 0 0
```


###Step3: Calculating the total number of steps taken per day

To calculate the total number of steps taken per day, we will need to aggregate the data by date and then summarize the value for each group:

```r
df_total<-aggregate(as.numeric(df_cc$steps),list(df_cc$date),FUN=sum)
#adding appropriate labels to the data frame
colnames(df_total) <- c("date", "total number of steps")
head(df_total[,2])
```

```
## [1]   126 11352 12116 13294 15420 11015
```

The histogram representation of the results gives us the following picture:

```r
hist(df_total[,2],
     xlab="Numer of steps",
     ylab="Days (frequency)", 
     main ="Total number of steps taken per day",
     col = "blue",
     breaks=20,
     density=40,
     label=TRUE,
     ylim=c(0,20),
     xlim=c(0,25000)
     )
```

![](figure/unnamed-chunk-1-1.png) 

Calculation mean and median of total number of steps taken per day:

```r
mean(df_total[,2])
```

```
## [1] 10766.19
```

```r
median(df_total[,2])
```

```
## [1] 10765
```
shows that mean and median values are slightly different.

###Step 4: Defining the average daily activity pattern

In order to define the average daily activity pattern, we wil group the data by time intervals and then we all average it across all intervals, i.e. we will define the number of steps for interval number 5 across all days and then we'll divide it by the number of intervals number 5 etc.:

```r
#aggregating the data by steps, grouping by intervals and applying mean (i.e ariphmetical average)
df_cc_int<-aggregate(df_cc[,1], list(df_cc$interval), mean)
```

And then we will make a time series plot (i.e. type = "l") of the 5-minute intervals (x-axis) and the averaged number of  steps taken (y-axis):

```r
plot(df_cc_int[,1],df_cc_int[,2],
     type="l",
     xlab="Intervals",
     ylab="Number of steps",
     main="Average number of steps taken")
```

![](figure/unnamed-chunk-3-1.png) 

We can see an obvious peak on the plot, which shows the 5-minute interval, on average across all the days in the dataset, containining the maximum number  of steps. Let's define which interval was it:

```r
max_steps<-max(df_cc_int[,2])
max_int<-df_cc[df_cc[,2]==max_steps]
#Adding the labels back to the dataframe
colnames(df_cc_int)<-c("interval","steps")
max_int<-df_cc_int[df_cc_int$steps==max_steps,]
max_int
```

```
##     interval    steps
## 104      835 206.1698
```

So we can see that it was the interval number 835 that resulted in the highest average number of steps across all days.

###Step 5: Imputing missing values

We've already discovered the number of days/intervals where there are missing values (see above). Previously, we've excluded such objects from the analysis. Such method of analysis is called 'complete cases'. Although, we have a relatively small number of incomplete cases, the strategy of excluding them altogether might bias our analysis because the units with missing values potentially differ systematically from the completely observed cases (Source: http://www.stat.columbia.edu/~gelman/arm/missing.pdf).

There is a number of strategies around imputing missing values in datasets. Usually, the strategy chosen depends on the patterns around the missing data. If we could confirm that missing data is completely random, i.e. it doesn't correlate with other factors such as a days of the week or time of the day, then in this case we would simply used mean or median for the day or interval for which the data is missing. In order to do a quick analysis of randomness of missing values in the data frame, special vizualization packages can be be used.

We will leave it out of scope for the current analysis and we'll proceed with a simple replacement of missing values of steps with a mean for numbers of steps for this day:

```r
library(plyr)
#creating a function replacing a value with its mean
impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))

#imputing missing values by groups (intervals)
imp_df <- ddply(activity_df, ~ interval, transform, steps = impute.mean(steps))
```


The new dataset that is equal to the original dataset but with the missing data filled in now looks like:

```r
summary(imp_df)
```

```
##      steps            date              interval     
##  Min.   :  0.00   Length:17568       Min.   :   0.0  
##  1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
##  Median :  0.00   Mode  :character   Median :1177.5  
##  Mean   : 37.38                      Mean   :1177.5  
##  3rd Qu.: 27.00                      3rd Qu.:1766.2  
##  Max.   :806.00                      Max.   :2355.0
```

We can now compare it with the summary of the original data frames and see if anything has changed: 

```r
summary(activity_df)
```

```
##      steps            date              interval     
##  Min.   :  0.00   Length:17568       Min.   :   0.0  
##  1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
##  Median :  0.00   Mode  :character   Median :1177.5  
##  Mean   : 37.38                      Mean   :1177.5  
##  3rd Qu.: 12.00                      3rd Qu.:1766.2  
##  Max.   :806.00                      Max.   :2355.0  
##  NA's   :2304
```

As we can see, the mean and median values of steps stayed the same, however, the 3rd quartile has changed.

###Step 6: Making a histogram of the total number of steps taken each day after missing values were imputed

We will now build a histogram of the total number of steps taken each day to see how it differs from the original one we build before imputing missing values:

```r
library(dplyr)
imp_df_daily<-summarize(group_by(imp_df,date),sum(steps))
imp_df_daily$date<-as.Date(imp_df_daily$date)
#recovering proper labels
colnames(imp_df_daily)<-c("Date","Steps")
hist(imp_df_daily$Steps,
     breaks=20,
     density=40,
     col="blue",
     label=TRUE,
     ylim=c(0,20),
     xlim=c(0,25000),
     xlab="Number of steps",
     ylab="Days (frequency)",
     main="Total number of steps taken per day (after imputting missing data)"
     )
```

![](figure/unnamed-chunk-4-1.png) 

We will also compare mean and median daily values of the new data set with the original vectors of means and median values. Calculating means:

```r
df_imp_mean<-aggregate(as.numeric(imp_df$steps),list(imp_df$date),mean)
head(df_imp_mean[,2])
```

```
## [1] 37.38260  0.43750 39.41667 42.06944 46.15972 53.54167
```

Calculating medians:

```r
df_imp_median<-aggregate(as.numeric(imp_df$steps),list(imp_df$date),median)
head(df_imp_median[,2])
```

```
## [1] 34.11321  0.00000  0.00000  0.00000  0.00000  0.00000
```

Calculating mean and median of total number of steps taken per day after imputing missing values:

```r
mean(imp_df_daily$Steps)
```

```
## [1] 10766.19
```

```r
median(imp_df_daily$Steps)
```

```
## [1] 10766.19
```

As we can see, the mean and median values are now the same, which is expected since we've used means to replace the missing values.

###Step 7: Activity patterns on weekdays vs. weekends

In order to research the differences in activity patterns between weekdays and weekends, we will create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day. We will use the dataset with the filled-in missing values for this part.

```r
imp_df$day<-weekdays(imp_df$date)
imp_df$isweekday<-(imp_df$day=="Saturday" | imp_df$day=="Sunday")

#separating the dataset into two
#for weekends
imp_df_weekend<-subset(imp_df,isweekday==FALSE)
#for weekdays
imp_df_weekday<-subset(imp_df,isweekday==TRUE)
#grouping the data by dates and calculating the mean for each date
df_weekend<-aggregate(imp_df_weekend[,1], list(imp_df_weekend$interval), mean)
df_weekday<-aggregate(imp_df_weekday[,1], list(imp_df_weekday$interval), mean)
#restoring the knowledge about weekends vs. weekdays
df_weekend$day<-"weekend"
df_weekday$day<-"weekday"
#merging the datasets back into one
df<-rbind(df_weekend,df_weekday)
#restoring proper column names
colnames(df)<-c("interval","steps","day")

#adding a factor variable in order to separate the data for weekends and weekdays to use it later for plotting
f <- factor(df$day, labels = c("weekend", "weekday"))
```

To visualize it, we will build a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis):

```r
library(lattice)
xyplot(df$steps ~ df$interval | f, panel = function(x, y, ...){
    # call the default panel function for xyplot
    panel.xyplot(x, y, ...)},
    type="l",
    xlab="Intervals",
    ylab="Number of steps",
    layout=c(1,2))
```

![](figure/unnamed-chunk-6-1.png) 

As expected, weekdays have shown higher activity in the beginning of the day, which could be explained by morning commute to work. The activity during the day on weekdays stayed moderate, while the weekend activity stayed roughtly same throught the day except for the night degradation.

Further analysis of patterns aroung the missing values should be conducted in order to develop better missing values imputation strategy that could be used for the future modeling and predictions.
