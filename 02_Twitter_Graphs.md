02\_Twitter\_Graphs
================
Tara Jaigopal
12/15/2021

``` r
library(tidyverse)
library(readr)
library(plotly)
library(packcircles)
library(ggiraph)
# library(htmlwidgets)
library(zoo)
library(reshape2)
library(lubridate)
```

# Loading Data

``` r
Tweet <- read_csv("G:/My Drive/datares/team hummus/data/Tweet.csv")
```

# Graph 01: Circle Packing

``` r
Graph01 <- Tweet %>% group_by(company) %>% summarise(total_tweets = n()) %>% mutate(percent_of_tweets = total_tweets/sum(total_tweets)) %>% slice(2, 1, 3:5)
Graph01
```

    ## # A tibble: 5 x 3
    ##   company   total_tweets percent_of_tweets
    ##   <chr>            <int>             <dbl>
    ## 1 Apple          1426540            0.362 
    ## 2 Amazon          718867            0.182 
    ## 3 Google          327586            0.0830
    ## 4 Microsoft       375938            0.0953
    ## 5 Tesla          1096903            0.278

``` r
0.36153+0.27799
```

    ## [1] 0.63952

``` r
# Source: https://www.r-graph-gallery.com/308-interactive-circle-packing.html

# Generate the layout. This function return a dataframe with one line per bubble. 
# It gives its center (x and y) and its radius, proportional of the value
packing <- circleProgressiveLayout(Graph01$total_tweets, sizetype='area')
 
# We can add these packing information to the initial data frame
Graph01 <- cbind(Graph01, packing)
# Graph01
```

``` r
# The next step is to go from one center + a radius to the coordinates of a circle that
# is drawn by a multitude of straight lines.
Graph01.gg <- circleLayoutVertices(packing, npoints=50)
Graph01.gg$id <- as.factor(Graph01.gg$id)
```

``` r
# Make the plot

# display text
Graph01$display <- paste(Graph01$total_tweets, " tweets \n", round(100*Graph01$percent_of_tweets, 2), "%")

# static plot
p01 <- ggplot() + 
  geom_polygon_interactive(data = Graph01.gg, aes(x, y, group = id, fill=id, tooltip = Graph01$display[id], data_id = id), colour = "#202020", alpha = 0.6, size = 0.7) +
  scale_fill_brewer(palette="Set2")+ 
    geom_text(data = Graph01, aes(x, y, label = company, size=3), color="#202020") +
  theme_void() + 
  theme(legend.position="none", plot.margin=unit(c(0,0,0,0),"cm") ) + 
  coord_equal() +
  ggtitle("Share of Tweets by Company") + 
  theme(plot.title = element_text(hjust=0.5, size=36))

# Turn it interactive
# widg01 <- ggiraph(ggobj = p01, width_svg = 7, height_svg = 7)
# widg01

# save
# saveWidget(widg01, file=("G:/My Drive/datares/team hummus/graphs/Graph01.html"))
```

## Let’s make another circle chart about CEOs\!

Just in case I want to use it…

``` r
# number of followers in millions 
TweetCEO <- data.frame(Company=c("Apple","Amazon","Google","Microsoft","Tesla"),
           CEO=c("Tim Cook", "Jeff Bezos", "Sundar Pichai", "Satya Nadella", "Elon Musk"), 
                 followers_in_millions=c(13.1,3.4,4.1,2.7,68.3))
packing2 <- circleProgressiveLayout(TweetCEO$followers_in_millions, sizetype='area')
TweetCEO <- cbind(TweetCEO,packing2)

TweetCEO.gg <- circleLayoutVertices(packing2, npoints=50)
TweetCEO.gg$id <- as.factor(TweetCEO.gg$id)
```

``` r
# Make the plot

# display text
TweetCEO$display <- paste(TweetCEO$CEO, "\n", TweetCEO$followers_in_millions, "million followers")

# static plot
CEO <- ggplot() + 
  geom_polygon_interactive(data = TweetCEO.gg, aes(x, y, group = id, fill=id, tooltip = Graph01$display[id], data_id = id), colour = "#202020", alpha = 0.6, size = 0.7) +
  scale_fill_brewer(palette="Set2")+ 
    geom_text(data = Graph01, aes(x, y, label = company, size=3), color="#202020") +
  theme_void() + 
  theme(legend.position="none", plot.margin=unit(c(0,0,0,0),"cm") ) + 
  coord_equal() +
  ggtitle("Number of Twitter Followers each CEO has") + 
  theme(plot.title = element_text(hjust=0.5, size=36))

# Turn it interactive
# widg02 <- ggiraph(ggobj = CEO, width_svg = 7, height_svg = 7)
```

# Graph 02: Animated Line Chart

``` r
# Prep for join
CompanyName <- data.frame(
  ticker_symbol = c("AAPL", "AMZN", "GOOG", "GOOGL", "MSFT","TSLA"),
  company=c("Apple", "Amazon", "Google", "Google", "Microsoft", "Tesla"))

# cleaning 
Graph02 <- Tweet %>% mutate(yearmonth = as.yearmon(date), firstofmonth = ymd(format(Tweet$date, "%Y-%m-01"))) %>% group_by(yearmonth, firstofmonth, ticker_symbol) %>% summarize(number_of_tweets = n())
Graph02 <- Graph02 %>% ungroup()
Graph02 <- left_join(Graph02, CompanyName, by = "ticker_symbol")

head(Graph02)
```

    ## # A tibble: 6 x 5
    ##   yearmonth firstofmonth ticker_symbol number_of_tweets company  
    ##   <yearmon> <date>       <chr>                    <int> <chr>    
    ## 1 Jan 2015  2015-01-01   AAPL                     34310 Apple    
    ## 2 Jan 2015  2015-01-01   AMZN                      8686 Amazon   
    ## 3 Jan 2015  2015-01-01   GOOGL                     6755 Google   
    ## 4 Jan 2015  2015-01-01   MSFT                      5425 Microsoft
    ## 5 Jan 2015  2015-01-01   TSLA                      7631 Tesla    
    ## 6 Feb 2015  2015-02-01   AAPL                     34392 Apple

``` r
# subsetting for separate traces
Graph02_AAPL <- Graph02 %>% filter(company == "Apple")
Graph02_AMZN <- Graph02 %>% filter(company == "Amazon")
Graph02_GOOGL <- Graph02 %>% filter(company == "Google")
Graph02_MSFT <- Graph02 %>% filter(company == "Microsoft")
Graph02_TSLA <- Graph02 %>% filter(company == "Tesla")

# new dataframe
Graph02A <- data.frame(decimal_date = decimal_date(Graph02_AAPL$firstofmonth), Month = Graph02_AAPL$yearmonth, 
           Apple = Graph02_AAPL$number_of_tweets, 
           Amazon = Graph02_AMZN$number_of_tweets, 
           Google = Graph02_GOOGL$number_of_tweets, 
           Microsoft = Graph02_MSFT$number_of_tweets, 
           Tesla = Graph02_TSLA$number_of_tweets)
```

``` r
# line chart
p02 <- plot_ly(Graph02A, x = ~Month, y = ~Apple, type = "scatter", 
               mode = "lines", name="Apple", color="#66c2a5")
p02 <- add_trace(p02, y = ~Amazon, type = "scatter", mode = "lines",
                 name="Amazon", color="#8da0cb")
p02 <- add_trace(p02, y = ~Google, type = "scatter", 
                 mode = "lines", name="Google", color="#e78ac3")
p02 <- add_trace(p02, y = ~Microsoft, type = "scatter", mode = "lines", name="Microsoft",color="#fc8d62")
p02 <- add_trace(p02, y = ~Tesla, type = "scatter", mode = "lines", name="Tesla", color="a6d854")

p02 <- p02 %>% 
  layout(title = "Tweet Volume between Jan 2015 - Dec 2019",
         yaxis = list(title="Number of Tweets"), 
         xaxis = list(title="Date"))
# p02
```

``` r
# add animation: https://plotly.com/r/cumulative-animations/

# create accumulation function
accumulate_by <- function(dat, var) {
  var <- lazyeval::f_eval(var, dat)
  lvls <- plotly:::getLevels(var)
  dats <- lapply(seq_along(lvls), function(x) {
    cbind(dat[var %in% lvls[seq(1, x)], ], frame = lvls[[x]])
  })
  dplyr::bind_rows(dats)
}

Graph02B <- Graph02A %>% accumulate_by(~decimal_date)
```

``` r
p03 <- Graph02B %>% 
  plot_ly(x = ~decimal_date, y = ~Apple, frame = ~frame, type = 'scatter',
    mode = 'lines', line = list(simplyfy = F), name="Apple", color="#66c2a5", 
    text=~as.character(Month), 
    hovertemplate = paste('<b>%{text}</b>','<br>Tweets: %{y}')) %>%
  
  add_trace(y = ~Amazon, frame = ~frame, type = 'scatter',
    mode = 'lines', line = list(simplyfy = F), name="Amazon", color="#8da0cb",
    text=~as.character(Month), 
    hovertemplate = paste('<b>%{text}</b>','<br>Tweets: %{y}')) %>%
  
  add_trace(y = ~Google, frame = ~frame, type = 'scatter',
    mode = 'lines', line = list(simplyfy = F), name="Google", color="#a6d854",
    text=~as.character(Month), 
    hovertemplate = paste('<b>%{text}</b>','<br>Tweets: %{y}')) %>% 
  
  add_trace(y = ~Microsoft, frame = ~frame, type = 'scatter',
    mode = 'lines', line = list(simplyfy = F), name="Microsoft", color="#e78ac3",
    text=~as.character(Month), 
    hovertemplate = paste('<b>%{text}</b>','<br>Tweets: %{y}')) %>%
  
  add_trace(y = ~Tesla, frame = ~frame, type = 'scatter',
    mode = 'lines', line = list(simplyfy = F), name="Tesla", color="#fc8d62",
    text=~as.character(Month), 
    hovertemplate = paste('<b>%{text}</b>','<br>Tweets: %{y}'))

p03 <- p03 %>% layout(xaxis = list(title = "Date",zeroline = F),
                      yaxis = list(title = "Number of Tweets",zeroline = F),
                      title = "Tweet Volume between 2015 - 2020")
p03 <- p03 %>% animation_opts(frame = 100, transition = 0, redraw = FALSE)
p03 <- p03 %>% animation_slider(hide = T)
p03 <- p03 %>% animation_button(x = 1, xanchor = "right", y = 0, yanchor = "bottom")

# p03

Graph02 <- p03
```

# Focusing on the engagement metric

## Some EDA

``` r
max(Tweet$engagement)
```

    ## [1] 1703

``` r
Tweet %>% arrange(desc(engagement)) %>% slice(1:10) 
```

    ## # A tibble: 10 x 11
    ##    tweet_id writer          post_date body      comment_num retweet_num like_num
    ##       <dbl> <chr>               <dbl> <chr>           <dbl>       <dbl>    <dbl>
    ##  1  6.92e17 ValaAfshar     1453861082 Apple ha~          42         984      677
    ##  2  7.70e17 cnntech        1472491321 Apple's ~          11         729      918
    ##  3  5.75e17 RANsquawk      1425929198 Loving m~          66         882      654
    ##  4  8.16e17 DavidSchawel   1483470318 Sometime~          14         646      900
    ##  5  8.55e17 philstockworld 1492608950 Will We ~           0         969      520
    ##  6  8.55e17 philstockworld 1492608950 Will We ~           0         969      520
    ##  7  1.02e18 QTRResearch    1532375225 Guys - I~         207         317      899
    ##  8  8.76e17 SJosephBurns   1497574819 $AMZN ha~          40         509      837
    ##  9  8.62e17 philstockworld 1494424155 Watergat~           1         971      400
    ## 10  6.14e17 Carl_C_Icahn   1435156866 Sold las~         153         671      533
    ## # ... with 4 more variables: ticker_symbol <chr>, company <chr>,
    ## #   engagement <dbl>, date <date>

``` r
Tweet %>% filter(engagement >= 1000) %>% group_by(company) %>% summarize(number_of_tweets = n())
```

    ## # A tibble: 5 x 2
    ##   company   number_of_tweets
    ##   <chr>                <int>
    ## 1 Amazon                  17
    ## 2 Apple                   35
    ## 3 Google                   6
    ## 4 Microsoft                8
    ## 5 Tesla                   82

``` r
# more Apple & Tesla
Tweet %>% filter(engagement >= 100) %>% dim()
```

    ## [1] 17249    11

``` r
# engagement is honestly low across the board
```

``` r
# types of engagement

# total engagement
sum(Tweet$engagement)
```

    ## [1] 12729249

``` r
# percent likes
sum(Tweet$like_num)/sum(Tweet$engagement)
```

    ## [1] 0.6981095

``` r
# percent comments
sum(Tweet$comment_num)/sum(Tweet$engagement)
```

    ## [1] 0.09719489

``` r
# percent retweet
sum(Tweet$retweet_num)/sum(Tweet$engagement)
```

    ## [1] 0.2046956

## Creating Bar Charts

``` r
# Data processing
Graph03 <- Tweet %>% group_by(ticker_symbol) %>% summarize(
  mean_engagement = (mean(engagement)),
  mean_like_num = mean(like_num),
  mean_comment_num = mean(comment_num), 
  mean_retweet_num = mean(retweet_num)
  )

Graph03 <- cbind(company=c("Apple", "Amazon", "Google", "Microsoft", "Tesla"), Graph03)
Graph03$company <- factor(Graph03$company, levels=c("Apple", "Amazon", "Google", "Microsoft", "Tesla"))
# Graph03
```

# 

``` r
# engagement bar chart
p04A <- plot_ly(Graph03, x = ~company, y = ~mean_engagement, type = 'bar', color = ~ticker_symbol)
p04A <- p04A %>% layout(title = "Mean Tweet Engagement by Company",
                        yaxis = list(title = 'Engagement Score'), 
                        xaxis = list(title = 'Company'), 
                        barmode = 'group', showlegend = FALSE)

# likes bar chart
p04B <- plot_ly(Graph03, x = ~company, y = ~mean_like_num, type = 'bar', color = ~ticker_symbol)
p04B <- p04B %>% layout(title = "Mean Tweet Engagement by Company",
                        yaxis = list(title = 'Number of Likes'), 
                        xaxis = list(title = 'Company'), 
                        barmode = 'group', showlegend = FALSE)

# comments bar chart
p04C <- plot_ly(Graph03, x = ~company, y = ~mean_comment_num, type = 'bar', color = ~ticker_symbol)
p04C <- p04C %>% layout(title = "Mean Tweet Engagement by Company",
                        yaxis = list(title = 'Number of Comments'), 
                        xaxis = list(title = 'Company'), 
                        barmode = 'group', showlegend = FALSE)

# retweets bar chart
p04D <- plot_ly(Graph03, x = ~company, y = ~mean_retweet_num, type = 'bar', color = ~ticker_symbol)
p04D <- p04D %>% layout(title = "Mean Tweet Engagement by Company",
                        yaxis = list(title = 'Number of Retweets'), 
                        xaxis = list(title = 'Company'), 
                        barmode = 'group', showlegend = FALSE)

#p04A
#p04D
```

``` r
# some calculations regarding Tesla
Tesla <- Tweet %>% filter(company=="Tesla")
sum(Tesla$like_num)/sum(Tweet$like_num) #majority
```

    ## [1] 0.6492001

``` r
sum(Tesla$comment_num)/sum(Tweet$comment_num) #majority
```

    ## [1] 0.6165559

``` r
sum(Tesla$retweet_num)/sum(Tweet$retweet_num) #plurality
```

    ## [1] 0.4138718

## Stacked bar

``` r
p05 <- plot_ly(Graph03, x = ~company, y = ~mean_like_num, type = 'bar', name = 'Likes')
p05 <- p05 %>% add_trace(y = ~mean_comment_num, name = 'Comments')
p05 <- p05 %>% add_trace(y = ~mean_retweet_num, name = 'Retweets')
p05 <- p05 %>% layout(yaxis = list(title = 'Mean Engagement Score'), 
                      xaxis = list(title = 'Company'),
                      barmode = 'stack')

#p05
```

``` r
Tweet %>% group_by(company) %>% summarise(median_engagement = median(engagement))
```

    ## # A tibble: 5 x 2
    ##   company   median_engagement
    ##   <chr>                 <dbl>
    ## 1 Amazon                    0
    ## 2 Apple                     0
    ## 3 Google                    0
    ## 4 Microsoft                 0
    ## 5 Tesla                     1
