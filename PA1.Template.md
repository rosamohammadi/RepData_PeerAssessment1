Reproducible Research: Peer Assessment 1
----------------------------------------

#### Set echo = TRUE for all chuncks to show R codes in markdown

    knitr::opts_chunk$set(echo = TRUE)

#### Removing hashtags in final result

#### Setting working directory

    setwd('C:\\Users\\moham\\Documents\\R')

#### Loading and Preprocessing the data

download.file(“<a href="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip" class="uri">https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip</a>”,
“activity.zip”)

#### Unziping the data

    if (!file.exists('activity.csv')) {
      unzip(zipfile = "activity.zip")
    }

#### Reading the data

    activityData <- read.csv(file="activity.csv", header=TRUE)

#### What is the mean total number of steps taken per day?

##### Calculate the total steps taken per day

    totalSteps <- aggregate(steps ~ date, activityData, sum, na.rm = TRUE) 

#### Make a histogram of the total number of steps taken per day

    hist(totalSteps$steps,
         main = "Total Steps per Day",
         xlab = "Number of Steps", breaks = 15)

![](PA1.Template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

#### Calculate and report the mean and median of total steps taken per day

    meanSteps <- mean(totalSteps$steps, na.rm = TRUE)
    medSteps <- median(totalSteps$steps, na.rm = TRUE)
    paste("The mean total steps in a day is", meanSteps, "and the median is", medSteps, ".")

    [1] "The mean total steps in a day is 10766.1886792453 and the median is 10765 ."

#### What is the average daily activity pattern?

#### Make a time-series plot of the 5-minute interval and the average number of steps taken, averaged acoss all days.

    library(ggplot2)
    meanStepsByInt <- aggregate(steps ~ interval, activityData, mean)
    ggplot(data = meanStepsByInt, aes(x = interval, y = steps)) +
      geom_line() +
      ggtitle("Average Daily Activity Pattern") +
      xlab("5-minute Interval") +
      ylab("Average Number of Steps") +
      theme(plot.title = element_text(hjust = 0.5))

![](PA1.Template_files/figure-markdown_strict/unnamed-chunk-9-1.png)

#### Which 5-minute interval across all days contain the maximum number of steps

    maxInt <- meanStepsByInt[which.max(meanStepsByInt$steps),]
    paste("The interval with the largest average number of steps is interval", maxInt$interval, ".")

    [1] "The interval with the largest average number of steps is interval 835 ."

#### Imputing missing values

#### Calculate and report the total number of missing values in the dataset

    missingVals <- is.na(activityData$steps)
    sum(missingVals)

    [1] 2304

#### Devise a strategy for filling in all of the missing values

#### Create a new dataset that is equal to the original dataset but with the missing data filled in.

    imp_activityData <- 
    transform(activityData, steps = ifelse(is.na(activityData$steps),meanStepsByInt$steps[match(activityData$interval, meanStepsByInt$interval)],activityData$steps))

#### Make a histogram of the total number of steps taken each day and

#### and report the mean and median.

    impStepsByInt <- aggregate(steps ~ date, imp_activityData, FUN=sum)
    hist(impStepsByInt$steps,
         main = "Imputed Number of Steps Per Day",
         xlab = "Number of Steps")

![](PA1.Template_files/figure-markdown_strict/unnamed-chunk-13-1.png)

    impMeanSteps <- mean(impStepsByInt$steps, na.rm = TRUE)
    impMedSteps <- median(impStepsByInt$steps, na.rm = TRUE)
    diffMean = impMeanSteps - meanSteps
    diffMed = impMedSteps - medSteps
    diffTotal = sum(impStepsByInt$steps) - sum(totalSteps$steps)
    paste("The new mean total steps in a day is", impMeanSteps, "and the median is", impMedSteps, ", instead of", meanSteps, "and", medSteps, "respectively.")

    [1] "The new mean total steps in a day is 10766.1886792453 and the median is 10766.1886792453 , instead of 10766.1886792453 and 10765 respectively."

#### Are there differences in activity patterns between weekdays and weekends?

#### Create a new factor variable in the dataset with two levels - “weekend” and “weekday”

    DayType <- function(date) {
      day <- weekdays(date)
      if (day %in% c('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'))
        return ("weekeday")
      else if (day %in% c('Saturday', 'Sunday'))
        return ("weekend")
      else
        stop ("Invalid Date Format.")
    }
    imp_activityData$date <- as.Date(imp_activityData$date)
    imp_activityData$day <- sapply(imp_activityData$date, FUN = DayType)

#### Make a panel plot containnig a time-series plot of the 5-minute interval

#### and the average number of steps taken across all weekdays or weekends

    meanStepsByDay <- aggregate(steps ~ interval + day, imp_activityData, mean)
    ggplot(data = meanStepsByDay, aes(x = interval, y = steps)) + 
      geom_line() +
      facet_grid(day ~ .) +
      ggtitle("Average Daily Activity Pattern") +
      xlab("5-minute Interval") +
      ylab("Average Number of Steps") +
      theme(plot.title = element_text(hjust = 0.5))

![](PA1.Template_files/figure-markdown_strict/unnamed-chunk-15-1.png)