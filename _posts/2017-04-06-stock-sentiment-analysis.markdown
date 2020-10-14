---
layout: post
title: Stock Sentiment Analysis
date: 2020-10-05 13:32:20 +0300
description: The goal of this analysis was to build a data pipeline using airflow
img: markus-spiske-5gGcn2PRrtc-unsplash.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Airflow, AWS, Docker]
---
By utilizing Airflow, Docker, EC2, S3, and RDS, I built a workflow for ETL processing. One of my interests outside of work is analyzing the stock markets and looking for under-valued or high-upside stocks. Most of the material I read will come from Twitter, blogs, or news sites which really only expresses the views of writers, not the average investor. I figured it would be a fun and interesting challenge to gather data that could be more representative of the public's sentiment on a particular stock. I decided to do this using Reddit's API. While I was not overly optimistic in discovering new stocks to buy, I was curious to see if the markets followed the communities (Reddit) and vice versa. Are certain communities (ex. subreddits) better tracking the markets than others? Are certain communities more difficult to interpret based on their language (sarcasm, etc..)?

## Overall Workflow

## Step 1: Extract Data From Reddit API
~~~python
def get_data():
    import praw
    import pandas as pd
    from praw.models import MoreComments
    from datetime import date,timedelta

    reddit = praw.Reddit(client_id = REDDIT_USER,
                         client_secret = REDDIT_PASS,
                         user_agent = "User-Agent: script:com.example.myredditapp:v1 (by /u/coletrain1)")

    scoring = pd.DataFrame({'source' : [],
                            'subgroup' : [],
                           'topic' : [],
                           'comment' : [],
                           'score' : []})

    subreddits = ['investing','stocks','stockmarket','options','finance','wallstreetbets']

    for subreddit in subreddits:
        print(subreddit + ": ")
        urls = []
        for submission in reddit.subreddit(subreddit).top(time_filter='day', limit=10):
            urls.append(submission.url)
        for topic in urls:
            try:
                submission = reddit.submission(url=topic)
                submission.comments.replace_more(limit=None)
                i=0
                for comment in submission.comments.list():
                    print('comment ' + str(i+1) + " out of " + str(len(submission.comments.list())))
                    i += 1
                    if comment.score > 25:
                        scoring = scoring.append({'source': 'reddit','subgroup' : subreddit,'topic':topic, 
                                        'comment':comment.body,'score':comment.score},
                                        ignore_index=True)
            except:
                pass

    file = '~/scraped_data_' + str(date.today() - timedelta(days = 1)) + ".csv"
    scoring.to_csv(file, index = False)

~~~




![I and My friends]({{site.baseurl}}/assets/img/we-in-rest.jpg)
