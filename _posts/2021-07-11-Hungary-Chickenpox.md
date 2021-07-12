---
layout: post
title: "Dataset Analysis: Forecasting chickenpox cases in Hungary"
date: 2021-07-11 16:30:00 +0200
categories: # Learning | Data Science | Security | Meta | Stories
    - data-science
tags:
    - r
    - time series
    - decomposition
    - forecasting
    - medical
description: The analysis and ARIMA forecasting of a time series about chickenpox cases in Hungary.
readtime: 5
# last_modified_at: 2021-07-07 09:40:00 +0200
author: Daniel Szogyenyi
---

## Introducing the dataset

The dataset I work with is from [UCI Machine Learning Repository](https://archive-beta.ics.uci.edu/ml/datasets/580). It's spatio-temporal dataset of weekly chickenpox cases from Hungary, which consists of a county-level adjacency matrix and time series of the county-level reported cases between 2005 and 2015.  
In this analysis I will not use the county adjacency-matrix, as I will work with an aggregated, country-level data.


{% highlight r %}
library(readr) # for reading CSV
library(lubridate) # for managing times better
library(forecast) # for forecasting
{% endhighlight %}

## Loading and preprocessing

Right after loading into a dataframe, I convert the Data to POSIXct to be able to use as a `ts` object.


{% highlight r %}
df <- read_csv('src/hungary_chickenpox/hungary_chickenpox.csv')
df$Date <- as.POSIXct(df$Date, format="%d/%m/%Y", tz="CET")
head(df)
{% endhighlight %}



{% highlight text %}
## # A tibble: 6 x 21
##   Date                BUDAPEST BARANYA  BACS BEKES BORSOD CSONGRAD FEJER  GYOR HAJDU HEVES  JASZ KOMAROM NOGRAD  PEST SOMOGY SZABOLCS TOLNA   VAS VESZPREM  ZALA
##   <dttm>                 <dbl>   <dbl> <dbl> <dbl>  <dbl>    <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>   <dbl>  <dbl> <dbl>  <dbl>    <dbl> <dbl> <dbl>    <dbl> <dbl>
## 1 2005-01-03 00:00:00      168      79    30   173    169       42   136   120   162    36   130      57      2   178     66       64    11    29       87    68
## 2 2005-01-10 00:00:00      157      60    30    92    200       53    51    70    84    28    80      50     29   141     48       29    58    53       68    26
## 3 2005-01-17 00:00:00       96      44    31    86     93       30    93    84   191    51    64      46      4   157     33       33    24    18       62    44
## 4 2005-01-24 00:00:00      163      49    43   126     46       39    52   114   107    42    63      54     14   107     66       50    25    21       43    31
## 5 2005-01-31 00:00:00      122      78    53    87    103       34    95   131   172    40    61      49     11   124     63       56     7    47       85    60
## 6 2005-02-07 00:00:00      174      76    77   152    189       26    74   181   157    44    95      97     26   146     59       54    27    54       48    60
{% endhighlight %}
### Aggregating

I created an aggregated column using an `apply` function, which simply sums the chickenpox cases for every date.


{% highlight r %}
df$AGGREGATED <- apply(df, 1, function(x){
  sum(as.numeric(x[2:21]))
})
{% endhighlight %}

## Creating and analysing a time series object

The `freq` is calculated by `365.25/7`. The 0.25 is added because of leap years, and divided by seven, because the dataset is a weekly time series.


{% highlight r %}
ts <- ts(df$AGGREGATED, freq=365.25/7, start=decimal_date(ymd(min(df$Date))))
plot(
  ts,
  main = "Chickenpox cases in Hungary from 2005 to 2015",
  ylab = "Cases",
  xlab = "Year"
)
grid()
{% endhighlight %}

![plot of chunk plot-time-series](/assets/Rfig/plot-time-series-1.svg)

The plot already hints some insights:

- We can see a clear *seasonality*. The peak of the cases is always in the winter, and the minimum of each year is somewhere near summer-fall.
- A little *decreasing trend* is also visible, it seems like the average case number is decreased by about 500 (in the winters) in 10 years.

### Pseudo-trend with smoothing

A pseudo-trend line can be created using the `scatter.smooth` function, just to get a sight of the change in the number of cases.


{% highlight r %}
scatter.smooth(
  ts,
  main = "Smoothed line of chickenpox cases",
  ylab = "Cases",
  xlab = "Year",
  ylim = c(0,1750),
  pch=19,
  col="pink",
  cex=0.25,
  lpars = list(
    col="black",
    lwd=2.5,
    type="l"
  ),
)
grid()
{% endhighlight %}

![plot of chunk pseudo-trendline](/assets/Rfig/pseudo-trendline-1.svg)
The decreasing attribute of the cases which was suggested by the above plot is clearly visible now. From 2005 to 2015 the "average" number of chickenpox cases dropped by more than 500.

### Decomposition

I chose a multiplicative decomposition, because it seems like the seasonality of the cases is lowering over time instead of being a constant.


{% highlight r %}
dc <- decompose(ts, "multiplicative")
plot(dc)
grid()
{% endhighlight %}

![plot of chunk decomposition](/assets/Rfig/decomposition-1.svg)

#### Trend


{% highlight r %}
plot(
  dc$trend,
  main = "Trend of chickenpox cases",
  xlab = "Time",
  ylab = "Cases"
)
grid()
{% endhighlight %}

![plot of chunk trendline](/assets/Rfig/trendline-1.svg)

We can definitely see a decreasing trend-line, however, there seems to be some *bi-yearly seasonality* too (look at the waves from 2007 to 2014).


{% highlight r %}
scatter.smooth(
  dc$trend,
  main = "Trend of chickenpox cases - smoothed",
  xlab = "Time",
  ylab = "Cases",
  pch=19,
  col="gray",
  cex=0.25,
  lpars = list(
    col="red",
    lwd=2.5,
    type="l"
  ),
)
grid()
{% endhighlight %}

![plot of chunk trendline-smoothed](/assets/Rfig/trendline-smoothed-1.svg)

If the trend is smoothed, the decreasing attribute is even more visible.

#### Seasonality


{% highlight r %}
# xaxt for the custom X axis
plot(
  dc$figure, xlim=c(2,50),
  xaxt="n", type="o", pch=19, cex=0.75,
  main = "One season of chickenpox", xlab = "Month", ylab = "Seasonality"
)
abline(1,0,lty="dashed",col="red", lwd=0.75)

# 4.35 = 52.25/12, it's for a monthly representation
# (0:11)+2 indicates one tick at the second week (middle) of each month
axis(1, 4.35*(0:11)+2, c("Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"))
grid(nx=12, ny=0, col="gray", lty="solid")
{% endhighlight %}

![plot of chunk seasonality](/assets/Rfig/seasonality-1.svg)

Looking at one season (year), we can see a strong split in seasonality. From *middle of May to middle of September* the cases are getting constantly *lower*, and they are *increasing from September to December*, and staying near a constant 1.5 from January to May.  
As known, the majority of chickenpox cases happen between the ages of 3 and 10, meaning the *patients are attending pre-school or school*, where the spread of the virus is helped by the personal contact. This shows a really strong correlation with this seasonality, as May-September - when there are fewer cases - is the time of summer vacation. 

## Forecasting

One of the most popular and best working forecasting method for evenly-spaced time series is ARIMA. One implementation is `forecast::auto.arima()` which return the best ARIMA model according to either AIC, AICc or BIC value.


{% highlight r %}
library(forecast)
m <- auto.arima(dc$x)
fc <- forecast(m, h = 8*52)
plot(fc, main="Forecast of chickenpox cases from 2014 to 2022", xlab="Year", ylab="Cases", ylim=c(0,2000))
grid()
{% endhighlight %}

![plot of chunk forecast-auto-arima](/assets/Rfig/forecast-auto-arima-1.svg)

It's hard to interpret this plot, the seasonality is still present, and we can see a decreasing trend.


{% highlight r %}
forecast.decomp <- decompose(fc$mean, "multiplicative")
plot(forecast.decomp$trend, ylim=c(200,600), main="Trend of forecasted chickenpox cases", ylab="Cases", xlab="Year", lwd=3, col="#0088fa")
grid()
{% endhighlight %}

![plot of chunk unnamed-chunk-7](/assets/Rfig/unnamed-chunk-7-1.svg)
The trend helps understanding the forecast model: it predicts a constant decrease of the chickenpox cases.

### Comparison with real data

The true numbers of chickenpox cases in Hungary are available at the website of the [Hungarian Central Statistical Office](https://www.ksh.hu/stadat_files/ege/en/ege0060.html). In this section, I will aggregate the cases by year, and compare the real numbers with the predicted ones.


{% highlight r %}
df_new <- read_delim('src/hungary_chickenpox/chickenpox_2015-2020.csv', ";", col_names=T)

df_dummy <- data.frame(list(year = df_new$Period, cases = df_new$Varicella))
df_new <- df_dummy
rm(list = c("df_dummy"))
{% endhighlight %}

The case numbers have spaces inside, so I have to remove them:

{% highlight r %}
df_new$cases <-unlist(lapply(df_new$cases, function(x){
  as.numeric(gsub(" ","", x))
}))
{% endhighlight %}

Create a new dataframe with the columns `year`, `real` and `forecast`, and store the aggregated numbers by year.


{% highlight r %}
fcYears <- coredata(floor(time(fc$mean)))
l <- lapply(unique(df_new$year), function(x){
  fcy <- round(sum(fc$mean[x == fcYears]))
  rey <- sum(df_new[df_new["year"] == x,]$cases)
  c(year=x, forecast=fcy, real=rey)
})

comparisonDf <- data.frame(matrix(unlist(l), nrow=length(l), byrow=TRUE))
names(comparisonDf) <- c("year","forecast","real")
{% endhighlight %}

Plot the real and forecasted values.


{% highlight r %}
plot(
  comparisonDf$year, comparisonDf$real, ylim=c(00000,50000),
  type="b", col="black", pch=19, cex=1.25,
  main="Forecasted and real chickenpox cases from 2015 to 2020", xlab="Year", ylab="Cases"
)
lines(comparisonDf$year, comparisonDf$forecast, type="b", pch=17, col="red", cex=1.25)
legend("topright", legend = c("Real","Forecasted"), pch = c(19,17), col=c("black", "red"))
grid()
{% endhighlight %}

![plot of chunk unnamed-chunk-11](/assets/Rfig/unnamed-chunk-11-1.svg)

- As seen on this plot, the forecast was under the real value in the first years, and got really close by 2018.  
- In 2019 the case numbers raised comparing to the previous year, while the model predicts the constant lowering.  
- In 2020 the chickenpox cases in Hungary drastically dropped, probably due to the home office caused by COVID-19. It was the only year, when the ARIMA forecast a higher value than the real, but I suppose a high leverage of the unexpected pandemic.
