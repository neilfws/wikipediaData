# Outcomes of Missions to Mars
Neil Saunders  
`r Sys.Date()`  

## Introduction
By _outcomes_ we mean either success or failure. This information is available on the Wikipedia page [List of missions to Mars](https://en.wikipedia.org/wiki/List_of_missions_to_Mars).

## Methods
The following code:

* reads the table from Wikipedia
* parses and reformats launch date as Date object
* assigns the outcome (0 = success, 1 = failure)
* calculates a running total of failures and percent failure at each launch date

Note that some outcomes are recorded as "partial failure"; the code assigns value 1 to those.


```r
library(ggplot2)
library(rvest)

mars <- read_html("https://en.wikipedia.org/wiki/List_of_missions_to_Mars") %>% html_nodes("table") %>% .[[1]] %>% html_table()
mars$failed <- 0
mars$failed[grep("failure", mars$`Outcome[1]`)] <- 1
mars$date <- sapply(strsplit(mars$`Launch date[1]`, "0000"), function(x) x[4])
mars$Date <- as.Date(mars$date, "%e %B %Y")
mars$cs <- cumsum(mars$failed)
mars$pc <- 100 * (mars$cs / 1:nrow(mars))
```

## Results
Now we can plot percentage of missions failed by launch date.


```r
ggplot(mars, aes(Date, pc)) + geom_jitter(aes(color = factor(failed))) + theme_bw() + 
    geom_smooth() + labs(x = "Date of launch", y = "Percent of missions failed at date of launch", 
    title = "Outcomes Of Missions To Mars") + scale_x_date(date_breaks = "10 years", 
    date_labels = "%Y") + scale_color_manual(values = c("skyblue3", "darkorange"), 
    name = "outcome", labels = c("success", "failure"))
```

![](marsMissions_files/figure-html/plot-data-1.png)<!-- -->

It looks as though we're getting better at getting to Mars. Around half of missions up to and including the most recent have failed. Thirty years ago, the figure was closer to two-thirds. The chart also suggests a slight upward bump in failure rate when missions resumed in the late 1980s. Perhaps this corresponds to a learning curve, brought about by new experimental technologies and resumption after the long hiatus.
