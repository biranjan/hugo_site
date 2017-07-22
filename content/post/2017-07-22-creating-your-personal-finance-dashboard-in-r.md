---
title: How I made my personal finance dashboard
author: ~
date: '2017-07-22'
slug: creating-your-personal-finance-dashboard-in-r
categories: ["R"]
tags: ["Dashboard"]
draft: yes
---
Once in a while I like to know how am I doing with my expenses but my bank doesn't provide any application where I can break down my expenses.
<!--more-->
However, my bank does provide a financial statement albeit in a not so helpful .txt format.


## Preparing Data

Once I read the file in R, I had to build some look-up function that identifies the place and assign proper categories for example if in the financial statement there is name of K-Mart then function assigns it in a category of grocery. All in all I created 6 categories and create it as list as shown below.

```r
categories <- list(food = c("Walmart","M","Tesco","R"),
                  restaurant = c("McDonal","KFC","Cafe"),
                  cloth = c("H","STADIUM","CARLINGS","HENNES"),
                  transport = c("L-line"),
                  household = c(""IKEA","TIGER","AMAZON.UK","AMAZON"),
                  housing = c("bay-housing"),
                  communication = c("T&T"))

```

So one can add as many market or name of the place it probably will depend on how the name is formatted in the financial statement.
And then map function form *plyr* package is used to map values below. It takes the column *place* and assigns them in categories. And if for some reason the item is not in categories you can add another if statement to create *other* category. 

```r

x$Categories <- x$Place

for (i in 1:length(categories)){
                x$Categories = mapvalues(x$Categories, from = categories[[i]],to = sample(names(categories[i]),length(categories[[i]]),replace = T))
        }

```

Once you have categorized the element then next one can just visualize the data. And this time I am using *plotly* to visualize the data. In the following graph I have piloted interactive plotly chart of income and expenses over time.


<iframe width="900" height="400" frameborder="0" scrolling="no" src="//plot.ly/~Biranjan/41.embed"></iframe>


And here is another chart displaying the amount spend in different categories over time.
<br>

<iframe width="900" height="400" frameborder="0" scrolling="no" src="//plot.ly/~Biranjan/3.embed"></iframe>


Lastly, if you'd like you can try to forecast the monthly expenses however with very little historical information forecast can be just for fun.

```r
library(forecast)
exp_ts <- xts(monthly_exp$sumMon,monthly_exp$round_date,name="monthly_expenses")

inc_model1 <- auto.arima(exp_ts) # building time series model
inc_fcast1 <- forecast(inc_model1, h=3,level = c(80, 95))

p <- plot_ly() %>%
  add_lines(x = time(exp_ts), y = monthly_exp$sumMon,
            color = I("black"), name = "observed") %>%
  add_ribbons(x = f_time, ymin = inc_fcast1$lower[, 2], ymax = inc_fcast1$upper[, 2],
              color = I("gray95"), name = "95% confidence") %>%
  add_ribbons(x = f_time, ymin = inc_fcast1$lower[, 1], ymax = inc_fcast1$upper[, 1],
              color = I("gray80"), name = "80% confidence") %>%
  add_lines(x = f_time, y = inc_fcast1$mean, color = I("blue"), name = "prediction")

p
```


<iframe width="900" height="400" frameborder="0" scrolling="no" src="//plot.ly/~Biranjan/43.embed"></iframe>


# Summary 

Now there is all element to create your own database you could either create a **flexdashboard** using r or use plotly web hosted dashboard



