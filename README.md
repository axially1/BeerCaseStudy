# BeerCaseStudy
Case Study 1 - Bilma Sanchez
---
title: "Beer Case Study"
author: "Bilma Sanchez"
date: "March 2, 2019"
output:
  html_document:
  keep_md: true
---

# **Beer Case Study**
## Using Beers.csv and Breweries.csv for analysis
The purpose of this case study is to use R to explore the correlation between the Beer Charactaristics and Brewery Attributes to determine if further marketing and exploration strategies can be identified.

```{r setup, include=TRUE}

install.packages("rmarkdown")
install.packages("summarytools")
library(summarytools)
install.packages("dplyr")
library(dplyr)
library(ggplot2)
install.packages("data.table")
library(data.table)
library(tidyverse)

#Read CSV files for analysis

beers<-read.csv("Beers.csv", header=TRUE)
View(beers)

breweries<-read.csv("Breweries.csv", header=TRUE)
View(breweries)


#Merge Beer and Breweries CSV Files

beers_n_breweries <- merge(beers, breweries, by.x = "Brewery_id", by.y = "Brew_ID")
View(beers_n_breweries)


#Exploratory Summary of Data
data("beers_n_breweries")
dfSummary(beers_n_breweries, 4)
print(dfSummary(beers_n_breweries, valid.col = FALSE, graph.magnif = 0.75), 
      max.tbl.height = 300, method = "render")


```

## Breweries Per State
Budweiser is competing with 2410 breweries in the United States, with Colorado having the most breweries at 265 or nearly 11% of the 2410 United States Breweries. Delaware and West Virginia are tied for the least number of breweries with 2 breweries each.

```{r}
# Question 1 Count of Breweries per State
state_brewery_count <- data.table(beers_n_breweries %>% count(State))
colnames(state_brewery_count)[colnames(state_brewery_count)=="n"] <- "Count"
print(state_brewery_count[order(-Count)])


# Bar Graph
state_count_graph <- ggplot(state_brewery_count, aes(x=reorder(state_brewery_count$State, Count), y=state_brewery_count$Count))+
  geom_bar(stat="identity", fill = "#FF9999", colour = "black", position=position_dodge()) +
  labs(title="Breweries per State", y = "Breweries", x = "State") +
  theme_bw(base_size = 7)+
  theme(plot.title = element_text(hjust = 0.5)) +
  theme(axis.title.y = element_text (size = 12, face = "bold"))
state_count_graph + coord_flip()

```

## First and Last 6 Observations of Merged Brewery and Beer Tables
The first and last 6 observations of the Merged Tables are as Follows:

```{r}
#Question 2 First and Last Six of Merged CSV Files
#Previously merged in exploratory data above so it would be available for question 1
print(head(beers_n_breweries, 6))
print(tail(beers_n_breweries, 6))
```

## Total Missing Values in Each Column
Missing values can have a number of impacts on analysis of data.  If missing values were not or could not be avoided during the data collection process, it is still important to identify the missing values.  Missing data may lead to analysis that may not be representative of the sample, as well as reducing the statistical power of an analysis.

See below for the missing value count per attribute.

```{r}
#Question 3 Count Missing Values

Missing_Values <- sapply(beers_n_breweries, function(y) sum(length(which(is.na(y)))))
Missing_Values <- data.frame(Missing_Values, round( (Missing_Values/2410)*100, digits = 2))
colnames(Missing_Values)[colnames(Missing_Values)=="round..Missing_Values.2410....100..digits...2."] <- "Percent of Total"
print(Missing_Values)
```

## Median Alcohol Content and International Bitterness Unit Per State
With the missing data from the previous table, we are able to identify that although less than 62 of the 2410 of the ABV values are are missing (less than 3%), 1005 of the 2410 values for the IBU (nearly 42%) data are missing.

The median IBU per state is used as a measurement since there are differing sample sizes would affect the mean.

```{r}
# Question4.1 Median 
median_by_state <- aggregate(beers_n_breweries[,c("ABV","IBU")], by = list(beers_n_breweries$State), FUN = median,na.rm=TRUE)
View(median_by_state)
```

## Median Alcohol Content and International Bitterness Unit Per State Bar Chart
```{r}
#Question4.2 ggplot for bar filled graph
median_by_state$Group.1 <- factor(median_by_state$Group.1, levels = median_by_state$Group.1[order(median_by_state$IBU)])
median_bargraph <- median_by_state %>% ggplot(aes(x=Group.1, y=IBU, fill=ABV)) +
  geom_bar(stat="identity", position=position_dodge()) +
  labs(title="Median State IBU", y = "IBU", x="STATE") +
  theme_bw(base_size = 8)+
  theme(plot.title = element_text(hjust = 0.5)) +
  theme(axis.text.y  = element_text(size=5, face="bold"))
median_bargraph + coord_flip()
```


## Maximum Alcohol by Volume Beer (ABV)
The state with the highest ABV beer:
```{r}
# Question5.1 Call row for Highest_ABV
Highest_ABV <- data.frame(beers_n_breweries$State, beers_n_breweries$ABV)
View(Highest_ABV[375,])
```
## Most Bitterness Per Unit Beer (IBU)
The state with the highest IBU beer:
```{r}
# Question5.2 Call row for Highest_IBU
Highest_IBU <- data.frame(beers_n_breweries$State, beers_n_breweries$IBU)
print(Highest_IBU[1857,])
```

## 6. Summary Statistics for Alcohol by Volume Beer (ABV)
The ABV data is normally distributed with a standard deviation of .06,values ranging from 0 to .13, and median of .06.
```{r}
# Question6.1 summary 
data("beers_n_breweries")
dfSummary(beers_n_breweries$ABV)
print(dfSummary(beers_n_breweries$ABV, valid.col = FALSE, graph.magnif = 0.75), 
      max.tbl.height = 300, method = "render")

```
## Summary Table for ABV by State
Each state with it's corresponding ABV summary.
```{r}
#Question6.2 summary for ABV
ABV_by_State_sum <- tapply(beers_n_breweries$ABV, beers_n_breweries$State, summary)
SD_by_state <- tapply(beers_n_breweries$ABV, beers_n_breweries$State,sd)
print(ABV_by_State_sum)
print(SD_by_state)
```

## 7. Relationship Between Beer Bitterness (IBU) and its Alcoholic Content (ABV)
There is evidence there is a positive correlation between ABV and IBU.  However, a paired t-test yields a p-value that indicates the ABV alone does not have a statistically significant causal effect on the IBU.


```{r}
# Question7.2 scatter plot for ABV
scatter_pl <- beers_n_breweries %>% ggplot(aes(x=IBU, y=ABV)) +
  geom_jitter() 
scatter_pl

#t-test
t.test(median_by_state$IBU, median_by_state$ABV, paired = TRUE)
```

##Conclusion
The number of missing values for the IBU are problematic for an analysis of that particular variable's impact. Although the correlation between IBU and ABV positive, as a company, Budweiser should concentrate on ABV data and how this data is correlated to the States' alcohol content preference.  In the future, gathering IBU data without so many missing values could allow for more in depth analysis between the correlation of these two variables.

https://www.screencast.com/t/4wnKGNEmck
