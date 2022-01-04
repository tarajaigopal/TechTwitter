Twitter\_01\_Cleaning
================
Tara Jaigopal
12/15/2021

``` r
library(tidyverse)
library(readr)
library(lubridate)
```

# Loading the data

``` r
#tweet data
TweetRaw <- read_csv("G:/My Drive/datares/team hummus/data/TweetRaw.csv")
CompanyTweet <- read_csv("G:/My Drive/datares/team hummus/data/CompanyTweet.csv")

#stock data
stocks <- read_csv("G:/My Drive/datares/team hummus/data/CompanyStockValue.csv")
```

# Cleaning Twitter Data

``` r
# writing company name in full
CompanyName <- data.frame(ticker_symbol = c("AAPL", "AMZN", "GOOG", "GOOGL", "MSFT","TSLA"), 
                          company=c("Apple", "Amazon", "Google", "Google", "Microsoft", "Tesla"))
CompanyName
```

    ##   ticker_symbol   company
    ## 1          AAPL     Apple
    ## 2          AMZN    Amazon
    ## 3          GOOG    Google
    ## 4         GOOGL    Google
    ## 5          MSFT Microsoft
    ## 6          TSLA     Tesla

``` r
# joining ticker symbol to tweets
Tweet <- left_join(TweetRaw, CompanyTweet, by = "tweet_id")
Tweet <- left_join(Tweet, CompanyName, by = "ticker_symbol")

# filtering "GOOG" & adding column for engagement
Tweet <- Tweet %>% filter(Tweet$ticker_symbol != "GOOG") %>% mutate(engagement = comment_num + like_num + retweet_num)

# converting timestamp to date
Tweet <- Tweet %>% mutate(date = as.Date(as_datetime(Tweet$post_date)))

# saving
# write_csv(Tweet, "G:/My Drive/datares/team hummus/data/Tweet.csv")
```

# Cleaning Stocks Data

``` r
# We want stocks from 2015-2019, not 2010-2020! 
# stocks <- stocks %>% mutate(date = as.Date(day_date)) %>% filter(year(date) >= 2015 & year(date) < 2020) 

# Google shares are traded under "GOOGL" (Class A) and "GOOG" (Class C). Since the other stocks are Class A, we'll only consider shares traded under "GOOGL" 

# stocks <- stocks %>% filter(Tweet$ticker_symbol != "GOOG")
```
