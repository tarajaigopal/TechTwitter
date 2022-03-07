# A Deep-Dive into Tech Twitter: Which companies are #trending?

In the last two decades, technology has completely changed how we lead our lives. From communication to cars, it has reimagined and revolutionized several industries. Hence, tech companies such as Apple, Amazon, Google, Microsoft, and Tesla rank among the largest and most successful companies in recent years, known for their quality products and high stock values. However, despite their success, these companies are often in hot water when they make decisions that impact customers — who frequently vent their concerns on social media platforms like Twitter, a microblogging platform in which users can post “tweets” of up to 280 characters and various multimedia.

Team Hummus at UCLA DataRes explored and compared public sentiment surrounding these companies by analyzing a [Kaggle dataset](https://www.kaggle.com/omermetinn/tweets-about-the-top-companies-from-2015-to-2020) which contains every tweet about these five companies from 2015 to 2020.  We published our findings in the Medium article ["A Deep-Dive into Tech Twitter"](https://ucladatares.medium.com/a-deep-dive-into-tech-twitter-which-companies-are-trending-76a7db50df19). This repository contains all the code, analyses, and explanations for the visualizations and insights found in the above article.

Contributors: [Tara Jaigopal](https://github.com/tarajaigopal), Prateik Sinha, Taylor Kim, Austin Gigi

# The Dataset

The dataset contains the details of nearly 4 million tweets, each with a unique identification number, classified based on which of five tech companies (Apple, Amazon, Google, Microsoft, or Tesla) they referenced. It also contains several other attributes that could help answer our research questions, including:

* `post_date`: the date on which the tweet was posted
* `body`: the ‘body’ or text of the tweet
* `comment_num`: the number of comments or replies the tweet received
* `like_num`: the number of times the tweet was liked (i.e., how many different users liked the tweet by clicking on the ♡ button)
* `retweet_num`: the number of times the tweet was ‘retweeted’ or reposted

These tweets were identified or scraped based on whether they contained each company’s “ticker symbol.” A ticker symbol is a combination of letters, often drawn from the company name itself, used to identify its shares on the stock market. For example, **AAPL** represents Apple, **AMZN** represents Amazon, and so on. Thus, the dataset is more likely to contain tweets from shareholders and stock market enthusiasts. However, since the popularity and public sentiment surrounding a company has a close relationship with its share price, it is possible to draw significant conclusions from this subset of tweets.

# Data Analysis and Code

While we were able to choose any language or platform for our analysis, we mainly utilized R to clean our data and create our visualizations For R, some of the specific packages we used include `tidyverse`, `ggplot2`, `plotly`, `ggiraph` and more to create interactive visualizations. In addition, in order to calculate sentiment scores for each tweet, we used the VADER library available in Python before switching back to R for visualization development. 

## Research Questions 


### Which companies are the most tweeted about? 

We developed an [interactive circle chart](https://tarajaigopal.github.io/TechTwitter/) to compare the the proportion of tweets about each company, which revealed that it is clear that both Apple and Tesla dominate Twitter, with almost 64% of all tweets in the dataset pertaining to one of the two companies. We also wanted to examine the frequency with which each company was being tweeted about, so we plotted the volume of tweets about each company against each month in an [animated time series chart](https://tarajaigopal.github.io/TechTwitter/Graph02.html). Again, most of the tweets were about Apple and Tesla. However, some other trends emerged: prior to mid-2017, tweets about Apple far surpassed those about the other companies and hit a peak in September 2016. However, after early 2018, tweets about Apple were no longer the majority. Instead, the number of tweets about Tesla skyrocketed, peaking in August 2018. 

Curious about why? Check out our [article](https://ucladatares.medium.com/a-deep-dive-into-tech-twitter-which-companies-are-trending-76a7db50df19) where we offer a few hypotheses. 

### How do Twitter users engage with these tweets?

A key metric to examine is engagement, a term used to describe a user’s interaction with a tweet. Attributes such as the number of likes, comments, and retweets (all provided in the dataset) are key indicators of this metric. The higher the engagement of a tweet, the greater its reach, so companies with high-engagement tweets tend to be more prevalent on Twitter than those with low-engagement tweets. We calculated an “engagement score” for each tweet by summing up the number of likes, comments, and retweets, and compared the mean engagement score of tweets by company in this [stacked bar graph](https://miro.medium.com/max/1400/0*tUs9uDmYFUFYhx2S). It emerged that Tesla stands above the rest with a mean engagement score of nearly 7, while other companies have engagement scores hovering around 1.5 to 2. 

Note that this metric is highly skewed due to outliers: while the mean engagement score for all companies is above 1, the median engagement score for Apple, Amazon, Google, and Microsoft is 0 — which means most tweets do not receive any engagement at all! Once again, Tesla proves to be an exception — its median engagement score is precisely 1.

### What do these tweets have in common? What topics do they cover?

To better understand the entire dataset, we created this [word cloud](https://miro.medium.com/max/1366/0*g9VAoNzIEBkOjCYV) of the most common words in the bodies of all tweets. We excluded the ticker symbols of each company as they were used to identify the tweets and skewed the graphic. Much of the word cloud contains words related to the stock market. We also used Latent Dirichlet Allocation to classify tweets into topics. You can browse [the article](https://ucladatares.medium.com/a-deep-dive-into-tech-twitter-which-companies-are-trending-76a7db50df19) to view our results. 

### Which companies receive the most overall positive sentiment? Which receive the most negative?

We used the **VADER** (Valence Aware Dictionary and sEntiment Reasoner) package on Python to measure both the polarity (i.e., negative or positive bent) of a word as well as the valence or intensity of that sentiment. It gave each tweet a compound score that is standardized between -1 and 1, with negative values representing negative tweets and positive values representing positive tweets. Scores close to zero are neutral in sentiment. 

On average, the compound scores of tweets skewed positive due to promotional tweets and advertisements. Thus, we filtered out tweets with positive sentiment scores as the negative tweets revealed far more interesting insights into public sentiment. We then created [this boxplot](https://miro.medium.com/max/1400/0*E44ZDNoqX1G-c6Xg) which revealed that tweets about Tesla are visibly more negative than those about other companies, with a lower median compound VADER score. Furthermore, Tesla also seems to have the greatest number of highly negative tweets (VADER score < -0.9). 
