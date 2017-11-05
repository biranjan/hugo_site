---
title: Making Twitter Bot with R
author: ~
date: '2017-10-29'
slug: making-twitter-bot-with-r
categories: [R,Twitter,Automation]
tags: [R,httr,rtweet,twitter,bot]
thumbnailImagePosition: "right"
thumbnailImage: http://res.cloudinary.com/dstcxl6nt/image/upload/v1509295892/bot-icon_kc9ghk.png
draft: yes
---

I was just playing with Twitter API so that I could automatically like Tweets from certain twitter account (for example twitter account of my workplace). And then I realized 
you can do quite more than just like the tweet. You can re-tweet the tweet, follow people, send messages and lot more. In this post I am going to show the basics of how to set-up the twitter automation and you can add your own tricks on top of my basic scripts.
<!--more-->

Before we fire R-studio or R console first you need to set up the twitter app and get the API credential.The R package that we also be using is Rtweet and the package page in git-hub has a detail instruction on how to set up your twitter API [here](http://rtweet.info/articles/auth.html).

Once everything is done let's load the required packages.

```
library(rtweet)
library(httr)

```
*Rtweet* is a wonderful package to do data mining task using data from twitter API, it also has ability to use 
twitter live streaming API which allows access to real-time tweets. And I am using *httr* in addition to rtweet as I couldn't find some certain post request so I am going to post tweet using *httr* post function.

Let's authenticate the app so that we can pull tweets from twitter. Once you run the code below it should redirect you to browser so that you can authenticate the app to use your twitter account.

```
#    Register an application at https://apps.twitter.com/
#    Make sure to set callback url to "http://127.0.0.1:1410/"
#
#    Replace key and secret below

appname <- "appname"

## api key (example below is not a real key)
key <- "Ea...."

## api secret (example below is not a real key)
secret <- "rIaxx....."

myapp <- create_token(app = appname,
                  consumer_key=key, consumer_secret=secret)
```

Once authentication is successful now we can get the search tweet that has hash-tag *Rstudio*, if you don't specify the number then by default search_tweets function from *rtweet* will give 100 tweets and since options helps us to filter the tweets so that we collect the tweet of today.

```
rt2 <- search_tweets("#Rstudio", token = twitter_token,include_rts = TRUE,type = "recent",since_id=Sys.date())
```
Now once you do that **rt2** will return a nice data-frame containing the text of tweet,date, user who tweeted and many other interesting data. Here all I am after is **status_id**, I need status id order to re-tweet the particular tweet. The only problem was that I couldn't find the **Post Retweet** request in the *rtweet* package so I am going to create my own using package *httr*.

We need to authenticate the app again using *httr*. If you have been using R to connect to API then you might not need any introduction but for others who've not used *httr* this is an excellent package by Hadley, see more [here](https://github.com/r-lib/httr/blob/master/demo/oauth1-twitter.r)

```
auth_endpoints("twitter")


myapp2 <- oauth_app("twitter",
  key = key,
  secret = secret)

# 3. Get OAuth credentials
twitter_token <- oauth1.0_token(oauth_endpoints("twitter"), myapp2)

```
Once authentication is successful now we can automate the re-tweet.

```
POST(paste0("https://api.twitter.com/1.1/statuses/retweet/",rt2$status_id[1]),config(token = twitter_token))
```
This should re-tweet the first tweet from the data-frame rt2. If you want to re-tweet all tweet, then we can just write a loop.

```
# Here I am going to retweet the original tweet not the retweet and I will also pause for a whilte before I retweet next tweet so that I wouldn't fire all the retweet in a fraction of seconds(so that my tweet doesn't look suspicious)

for(n in 1:nrow(rt2)){
  if (rt2$is_retweet[n] == FALSE) {
    POST(paste0("https://api.twitter.com/1.1/statuses/retweet/",rt2$status_id[n]),config(token = twitter_token))
    print(paste0("Retweeted: ",rt2$text[n])) 
  } else {
    print(paste0("is retweet: ",rt2$text[n]))
  }
  sleep_time <- runif(1, min=0, max=10)
  print(sleep_time)
  Sys.sleep(sleep_time)
}
```
As I said earlier you can do more than re-tweet. Now using the *rtweet* package you can follow or favorite the tweet without leaving your R console. See all the trick on the git-hub page pf [rtweet](http://rtweet.info/index.html)

```
# If you want to follow the user
post_follow(rt2$user_id[1])
# Post Favourite
post_favorite(rt2$status_id[1])
# or tweet palin text tweet
plain_tweets("txt")
```

Okay there we go, the above script will give you a basic skeleton to automate your re-tweet or tweet. You can schedule the script to run during certain time using cron job or create a loop that keeps finding a new tweet and keeps re-tweeting it. You can automate however you want to, I hope you'd find my script helpful.