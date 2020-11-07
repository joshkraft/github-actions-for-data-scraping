# How to Use Github Actions for Data Collection

> **Summary**
> In this blogpost, I will outline why **Github Actions can be useful for small-scale data collection projects**, by offering a free way to repeatedly run small data collection scripts in the cloud and push the data into a repository. Then, I will walk through how this tool can be used for **scraping Twitter data**, including how to **securely store API credentials** in Github.

[TOC]



## Motivation

A common starting point for any data-driven project is the search for good data. There are a few ways to go about gaining access to data:

| Option                            | Pros/Cons                                                  |
| --------------------------------- | ---------------------------------------------------------- |
| Buy a dataset from a vendor.      | High quality data, but is often very expensive.            |
| Use a publicly available dataset. | Free or cheap, but must wait for data to become available. |
| Generate your own dataset.        | Free or cheap, but requires effort.                        |

For many small-scale or exploratory projects, the first two options are often too expensive or too slow. If the data you are trying to collect is available on the web, there are a few free or cheap ways to go about collecting it: 

- **Data can be [scraped](https://en.wikipedia.org/wiki/Data_scraping)**, by using tools to parse through the front end of a website. This is considered a *brute force* approach, and is often discouraged (or blocked entirely) by major websites.
- **Data can be requested via an [API](https://en.wikipedia.org/wiki/API)**. This is considered the *elegant* approach, as the data is provided from the website in a structured format, though you will typically have to deal with limits placed on your requests.

Modern Python libraries such as [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) and [Tweepy](https://www.tweepy.org) have lowered the barrier to entry for generating datasets through scraping or accessing APIs. After a brief dive into the documentation, you can pretty reliably start gathering your own data for future use. 

However, oftentimes data needs to be gathered on a continuous basis. For example, a someone might be interested in gathering tweets about a certain topic on an hourly basis for analysis by stakeholders. The simplest approach would be to simply run the program on a laptop, hourly, and push the data into a repository somewhere. This is considered an [on-premises approach](https://en.wikipedia.org/wiki/On-premises_software) — the infrastructure for the project is supplied and maintained by the person that owns the laptop. This can be a great starting point for data projects, but the dependence on the availability of your laptop introduces an unnecessary liability into the system.

One way to eliminate this liability is to use a [cloud-based serverless computing](https://en.wikipedia.org/wiki/Serverless_computing) platform like [AWS Lambda](https://aws.amazon.com/lambda/) or [Google Cloud Functions](https://cloud.google.com/functions/docs/), which can be configured automatically run scripts and move the data for a fee. This platforms are very reliable, and are often the best choice for complex data projects.

However, recall that one of the allures of generating your own datasets is the fact that it can much cheaper than simply purchasing a dataset. AWS Lambda and Google Cloud functions are great platforms, but are often overkill for small scale projects. In this post, I will propose a workflow that leverages [**Github Actions**](https://docs.github.com/en/free-pro-team@latest/actions) for small scale data collection projects, which **adds the benefits of cloud-based serverless computing without (necessarily) adding cost**.

> **Note**: AWS Lambda, Google Cloud Functions, and Github Actions all have both free and paid tiers, and could fill this use case. However, my experience is that Github Actions are a better choice for small scale projects due to the generosity of the free tier, ease of use, and close integration with Github, where a project's code is typically already living.

## What are Github Actions?

Github Actions are a tool integrated into Github repositories, used to run small workloads on servers owned, operated, and maintained by Github. Here are a few of the most common use cases for Github Actions:

- Automatically run tests when new code is checked into a repository, before merging it into the overall codebase. ([**Continuous Integration**](https://en.wikipedia.org/wiki/Continuous_integration))
- Automatically release new code to end users, typically after passing the suite of tests. ([**Continuous Delivery**](https://en.wikipedia.org/wiki/Continuous_delivery)) 
- Scheduled jobs. For example, you might want to aggregate some metrics on changes in a repository and send it to relevant parties.

For more information about the capabilities (and limitations) of Github Actions, I would reccomend checking out the official [Quickstart for GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions/quickstart). 

In this case, we can use the scheduling capabilities of Github Actions to run our data collection script on a defined schedule, and let the Github servers handle the collection and moving of data. 

> **Note**: there are limitations to what Github Actions can be used for, especially while staying in the free tier. You should consult the [official documentation](https://docs.github.com/en/free-pro-team@latest/actions/reference/usage-limits-billing-and-administration) when deciding if this platform is right for your project. 

## An Example: Twitter Profile Scraper

As an example, I will walk through a simple data project I am currently working on. The overall goal of this project is to have an interesting visual way to see the tweets of the two main candidates in the 2020 US Presidential Election, Donald Trump and Joe Biden. 

While Twitter does have a freely accessible API, there are limitations that you must adhere to. Due to the volume of data on the platform, you cannot make a request like this:

> Get all tweets from *User X*.

Instead, the following approach must be taken:

> Get a small number of tweets from *User X*, that meet certain conditions, that have occured *after Date Y* or *since the last tweet we retrieved from User X.*

Additionally, here are some constraints I decided upon for this project:

- Data must be grabbed from Twitter on a near-real time basis. Tweets from last month are much less relevant than tweets from the last hour.
- The data should be easily accessible to anyone. Due to the small amount of data in this project, we will just store it on Github for the sake of simplicity.
- The data collection must be completely automated. 
- Credentials should be securely stored in the cloud.
- The project should be completely free.
  - It should be noted that there are limitations to what Github Actions can be used for, especially while staying in the free tier. You should consult the [official documentation](https://docs.github.com/en/free-pro-team@latest/actions/reference/usage-limits-billing-and-administration) when deciding if this platform is right for your project. 

Before beginning, you must gain access to the Twitter API by signing up for the [Twitter Developer Platform](https://developer.twitter.com/en/docs/twitter-api). Once you have been accepted, create an Application with Read/Write access, and make sure to securely store your Consumer Key, Consumer Secret, Access Token, and Access Token Secret. 

There are three main files that make this project work:

1. **scrape.py**
   1. This Python script performs the actual data collection.
2. **workflow.yml**
   1. This file defines our main Github action, and orchestrates the running of scrape.py
3. **decrypt_secret.sh**
   1. This file is referenced by workflow.yml, and is used to securely decrypt the API credentials used to access Twitter.

## scrape.py

The main function of this script can be seen here:

```python
def main():
    api = authenticate_with_secrets('/home/runner/secrets/secrets.json')
    usernames = ["realDonaldTrump", "JoeBiden"]

    last_tweet_ids = get_last_tweet_ids()

    for user in usernames:
        file_path = "data/" + user + "/data.csv"
        processed_tweets = []
        tweets = get_tweets_from_user(api, user, last_tweet_ids)
        if tweets:
            for tweet in tweets:
                processed_tweet = process_raw_tweet(tweet)
                processed_tweets.append(processed_tweet)
            upload_tweets(processed_tweets, file_path)
    update_last_tweet_ids(last_tweet_ids)
```

This program can be broken down into the following steps:

1. Authenticate with Twitter API.

2. Specify which accounts to collect tweets from.

3. See if we have a Tweet ID to use as a starting point.

4. Grab tweets from each account, proccess them, and then append them to a CSV file.

5. Update the Tweet IDs from Step 3.

   

### Authenticate with Twitter API

The following  function can be used to authenticate via Tweepy and access the Twitter API:

```python
import tweepy 

def authenticate_with_secrets(secret_filepath):
    secret_file = open(secret_filepath)
    secret_data = json.load(secret_file)
    CONSUMER_KEY = secret_data["API_KEY"]
    CONSUMER_SECRET = secret_data["API_SECRET"]
    ACCESS_TOKEN = secret_data["ACCESS_TOKEN"]
    ACCESS_TOKEN_SECRET = secret_data["ACCESS_SECRET"]
    secret_file.close()

    auth = tweepy.OAuthHandler(CONSUMER_KEY, CONSUMER_SECRET)
    auth.set_access_token(ACCESS_TOKEN, ACCESS_TOKEN_SECRET)
    api = tweepy.API(auth)
    
    return api
```



### Specify which accounts to collect tweets from

### See if we have a Tweet ID to use as a starting point

### Grab tweets from each account, proccess them, and then append them to a CSV file

### Update the Tweet IDs from Step 3



## workflow.yml

## decrypt_secret.sh











For now, you can set the `secret_filepath` to whatever you would like. However, it is important to note that this code will eventually be running on a Github server so you will need to update it accordingly once local developement is done.

One important limitation of the Twitter API is the inability to grab a users tweet from more than 7 days ago. As such, I have decided to store the 'most recently retrieved tweet ID' for each user in a separate JSON file. Each hour, we will poll the Twitter API and see if there are any tweets with a more recent ID for that user. If there are, we will grab those tweets and then update the JSON file to reflect the newest ID. In this case, here is what our JSON file would look like:

```json
{
  "realDonaldTrump": "1324401527663058944", 
  "JoeBiden": "1324445128434438145"
}
```

To set these IDs, you can visit any tweet from a given user in the past 7 days, and extract it from the end of the URL. For example, a tweet from the @realDonaldTrump account would have the following URL:

https://twitter.com/realDonaldTrump/status/**1324401527663058944**

To open this file, we will define a simple function:

```python
def get_last_tweet_ids():
    with open("most_recent_tweet_id.json", "r") as file:
        return json.load(file)
```

