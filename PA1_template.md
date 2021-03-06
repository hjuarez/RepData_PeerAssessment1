---
title: "RepData_PeerAssessment1"
author: "Héctor Juárez Lara"
date: "Wednesday, April 15, 2015"
output: html_document
---
# title: "RepData_PeerAssessment1, PA1_Template "
####    author: "Hector Juarez Lara"
####    date: "Wednesday, April 15, 2015"
####    output: html_document


Source file: "actvity.csv"
Downloaded from:
https://class.coursera.org/repdata-013/human_grading/view/courses/973514/assessments/3/submissions
on April 15th, 2015

1.- Load the data, create a copy of the data and convert date field to class "Date"


```r
data <- read.csv("./data/activity.csv")
tdata <- data
tdata$date <- as.Date(data$date)
```

### What is mean total number of steps taken per day?

#####   1.- Calculate the total number of steps taken per day

```r
#stepsperday <- aggregate(steps~date, tdata, sum)
library(plyr)
stepsperday <- ddply(data, .(date), summarize, steps = sum(steps, na.rm=T))
```

#####   2.- Make a histogram of the total number of steps taken each day


```r
# Compute the largest number of steps
max_steps <- max(stepsperday$steps)

# Create a histogram for steps
hist(stepsperday$steps, col=heat.colors(max_steps), main="Histogram of steps", 
     xlab="Steps")
```

<img src="figure/unnamed-chunk-2-1.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" style="display: block; margin: auto;" />

#####   3.- Calculate and report the mean and median of the total number of steps taken per day

```r
meanstepsperday <- mean(stepsperday$steps)
medianstepsperday <- median(stepsperday$steps)
```
#### The mean steps per day is 9354.23 and the median is 10395 .



### What is the average daily activity pattern?

#####   1.- Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
stepsperPeriod <- aggregate(steps~interval, tdata, mean)

stepsperPeriod$timestr <- formatC(stepsperPeriod$interval, width = 4, flag = "0")
stepsperPeriod$MisCinco <- paste(substr(stepsperPeriod$timestr,1,2),substr(stepsperPeriod$timestr,3,4),sep=":")
stepsperPeriod$strDatetime <- paste("2015-04-16", stepsperPeriod$MisCinco, sep=" ")
stepsperPeriod$time <- strptime(stepsperPeriod$strDatetime, "%Y-%m-%d %H:%M")

#timeSeries <- ts(stepsperPeriod$steps, frequency = 1)
#plot(x = timeSeries, y= NULL, plot.type = "multiple", ylab ="")
plot(x=stepsperPeriod$time,y=stepsperPeriod$steps, type = "l", xlab="", ylab="")
title(main = "Mean steps", xlab = "Hours of day",
      ylab = "Mean steps")
maxmeanstepsat = stepsperPeriod[which(stepsperPeriod$steps==max(stepsperPeriod$steps)),]
abline(v=as.numeric(maxmeanstepsat$time), col="blue")
resN2 <- paste("The maximum number of steps were \n during the next 5 minutes after", 
          format(maxmeanstepsat$time,"%H:%M"), sep=" ")
text(as.numeric(maxmeanstepsat$time)+25000, 190, resN2)
```

<img src="figure/unnamed-chunk-4-1.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" style="display: block; margin: auto;" />

#####   2.-Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps? 

R=The maximum number of steps were 
 during the next 5 minutes after 08:35



### Imputing missing values

#####   1.- Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
totalNAsinsteps <- sum(is.na(tdata$steps))
```
R= There are 2304 of na's in the dataset

#####   2.-Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

#####   3.-Create a new dataset that is equal to the original dataset but with the missing data filled in.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)



```r
tdata2 <- tdata
for (i in 1:nrow(tdata2)) {
    if (is.na(tdata2$steps[i])) {
        mireg <- stepsperPeriod[which(stepsperPeriod$interval==tdata2$interval[i]),]
        tdata2$steps[i] <- mireg$steps
    }
}
```

#####   4.-Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
stepsperday2 <- aggregate(steps~date, tdata2, sum)
hist(stepsperday2$steps, col="blue", main="Histogram of steps with replaced NA's", 
     xlab="Steps")
```

<img src="figure/unnamed-chunk-7-1.png" title="plot of chunk unnamed-chunk-7" alt="plot of chunk unnamed-chunk-7" style="display: block; margin: auto;" />

```r
meanstepsperday2 <- mean(stepsperday2$steps)
medianstepsperday2 <- median(stepsperday2$steps)
```

##### The mean steps per day with out NA's is 10766.19 and the median is 10766.19.


### Are there differences in activity patterns between weekdays and weekends?


```r
tdata2$dia <- weekdays(tdata2$date)
tdata2FinSem <- tdata2[which(tdata2$dia == "sábado" | tdata2$dia == "domingo"),]
tdata2EntreSem <- tdata2[which(tdata2$dia != "sábado" & tdata2$dia != "domingo"),]

stepsperFinSem2 <- aggregate(steps~interval, tdata2FinSem, mean)
stepsperEntSem2 <- aggregate(steps~interval, tdata2EntreSem, mean)

stepsperFinSem2$timestr <- formatC(stepsperFinSem2$interval, width = 4, flag = "0")
stepsperFinSem2$MisCinco <- paste(substr(stepsperFinSem2$timestr,1,2),substr(stepsperFinSem2$timestr,3,4),sep=":")
stepsperFinSem2$strDatetime <- paste("2015-04-16", stepsperFinSem2$MisCinco, sep=" ")
stepsperFinSem2$time <- strptime(stepsperFinSem2$strDatetime, "%Y-%m-%d %H:%M")

stepsperEntSem2$timestr <- formatC(stepsperEntSem2$interval, width = 4, flag = "0")
stepsperEntSem2$MisCinco <- paste(substr(stepsperEntSem2$timestr,1,2),substr(stepsperEntSem2$timestr,3,4),sep=":")
stepsperEntSem2$strDatetime <- paste("2015-04-16", stepsperEntSem2$MisCinco, sep=" ")
stepsperEntSem2$time <- strptime(stepsperEntSem2$strDatetime, "%Y-%m-%d %H:%M")

Ylim <- c(0, 225)
par(mfrow=c(2,1), mar=c(0,0,0,0), oma=c(2,2,0.5,0.5))
plot(x=stepsperEntSem2$time,y=stepsperEntSem2$steps, type = "l", ylim = Ylim,
     axes =T, xlab="", ylab="")

maxmeanstepsatEntSem2 = stepsperEntSem2[which(stepsperEntSem2$steps==max(stepsperEntSem2$steps)),]
abline(v=as.numeric(maxmeanstepsatEntSem2$time), col="blue")
titgraph1 <- "Mean steps by hour of day,\n weekdays pattern" 
text(as.numeric(maxmeanstepsatEntSem2$time)-20000, 150, titgraph1)

plot(x=stepsperFinSem2$time,y=stepsperFinSem2$steps, type = "l", ylim = Ylim, 
     axes=T, xlab="", ylab="")

maxmeanstepsatFenSem2 = stepsperFinSem2[which(stepsperFinSem2$steps==max(stepsperFinSem2$steps)),]
abline(v=as.numeric(maxmeanstepsatFenSem2$time), col="red")
titgraph2 <- "Mean steps by hour of day,\n weekends pattern" 
text(as.numeric(maxmeanstepsat$time)-20000, 150, titgraph2)
```

<img src="figure/unnamed-chunk-8-1.png" title="plot of chunk unnamed-chunk-8" alt="plot of chunk unnamed-chunk-8" style="display: block; margin: auto;" />
