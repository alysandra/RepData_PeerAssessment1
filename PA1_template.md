  
```{r}
title: "PA1_template.Rmd"
output: html_document
echo = TRUE
options(scipen = 1)
```
## Part 1: Loading and preprocessing the data
```{r}
data <- read.csv("~/mm/activity.csv",  colClasses = c("numeric", "character", 
                                                          "numeric"))
data$month <- as.numeric(format(data$date, "%m"))
noNA <- na.omit(data)
rownames(noNA) <- 1:nrow(noNA)
head(noNA)
dim(noNA)
library(ggplot2)
activity$date <- as.Date(activity$date, "%Y-%m-%d")
```

## Part 2: What is mean total number of steps taken per day?
```{r}
StepsTotal <- aggregate(steps ~ date, data = activity, sum, na.rm = TRUE)

hist(StepsTotal$steps, main = "Total Steps By Day", xlab = "Days", col = "yellow")

mean(StepsTotal$steps)
##Mean: [1] 10766.19
median(StepsTotal$steps)
##Median: [1] 10765
```

##Part 3: What is the average daily activity pattern?

###Use time series plot to check 5 minute intervals
```{r}
time_series <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)

##And plot: 
plot(row.names(time_series), time_series, type = "l", xlab = "5-min interval", 
     ylab = "Average across all Days", main = "Average number of steps taken", 
     col = "red")
## to find max number of steps on average in a 5 min interval
max_interval <- which.max(time_series)
names(max_interval)
```
##Part 4: Imputing missing values

###Find the number of missing values:
```{r}
activity_NA <- sum(is.na(activity))
activity_NA

##Replace the NA value with avg 5 min intervals
StepsAverage <- aggregate(steps ~ interval, data = activity, FUN = mean)
fillNA <- numeric()
for (i in 1:nrow(activity)) {
  obs <- activity[i, ]
  if (is.na(obs$steps)) {
    steps <- subset(StepsAverage, interval == obs$interval)$steps
  } else {
    steps <- obs$steps
  }
  fillNA <- c(fillNA, steps)
}
##Create a new datasset, then aggregate the steps and make a new histogram.
new_activity <- activity
new_activity$steps <- fillNA

StepsTotal2 <- aggregate(steps ~ date, data = new_activity, sum, na.rm = TRUE)
hist(StepsTotal2$steps, main = "Total steps by day", xlab = "day", col = "blue")

##New Mean and median

mean(StepsTotal2$steps)
##[1] 10766.19
median(StepsTotal2$steps)
##[1] 10766.19
```

## Part 5: Are there differences in activity patterns between weekdays and weekends?
###Weekday versus weekend walking: differentiate between the two first:
```{r}
day <- weekdays(activity$date)
daylevel <- vector()
for (i in 1:nrow(activity)) {
  if (day[i] == "Saturday") {
    daylevel[i] <- "Weekend"
  } else if (day[i] == "Sunday") {
    daylevel[i] <- "Weekend"
  } else {
    daylevel[i] <- "Weekday"
  }
}
##Compare the two types
activity$daylevel <- daylevel
activity$daylevel <- factor(activity$daylevel)

stepsByDay <- aggregate(steps ~ interval + daylevel, data = activity, mean)
names(stepsByDay) <- c("interval", "daylevel", "steps")
```
##Create the plot:
```{r}
xyplot(steps ~ interval | daylevel, stepsByDay, type = "l", layout = c(1, 2), 
       xlab = "Interval", ylab = "Number of steps")
```