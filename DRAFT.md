# Github Actions for Data Scraping

## Introduction

A common starting point for any data-driven project is the search for good data. There are a few ways to go about gaining access to data:

| Option                            | Pros/Cons                                                  |
| --------------------------------- | ---------------------------------------------------------- |
| Buy a dataset from a vendor.      | High quality data, but is often very expensive.            |
| Use a publicly available dataset. | Free or cheap, but must wait for data to become available. |
| Generate your own dataset.        | Free or cheap, but requires effort.                        |

For many small-scale or exploratory projects, the first two options are often too expensive or too slow. If the data you are trying to collect is available on the web, there are a few ways you can go about collecting it: 

- **Data can be [scraped](https://en.wikipedia.org/wiki/Data_scraping)**, by using tools to parse through the front end of a website. This is considered a *brute force* approach, and is often discouraged (or blocked entirely) by major websites.
- **Data can be requested via an [API](https://en.wikipedia.org/wiki/API)**. This is considered the *elegant* approach, as the data is provided from the website in a structured format, though you will typically have to deal with limits placed on your requests.



Modern Python libraries such as [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) and [Tweepy](https://www.tweepy.org) have lowered the barrier to entry for generating datasets through scraping or APIs. After a brief stroll through the documentation, you can pretty reliably start gathering your own data for future use. 

However, oftentimes data needs to be gathered on a continuous basis. For example, a someone might be interested in gathering tweets about a certain topic on an hourly basis for analysis by stakeholders. The simplest approach would be to simply run their script(s) on a laptop and push the data into a repository somewhere. This would fall under the category of [on-premises approaches](https://en.wikipedia.org/wiki/On-premises_software) — the infrastructure for the project is supplied (and maintained) by the individual/team/company that owns the laptop. While this can be a good starting point for data projects, the dependence on the availability/reliability of local hardware introduces a large liability into the system.

One way to eliminate this liability is to move the project onto a [cloud-based serverless computing](https://en.wikipedia.org/wiki/Serverless_computing) platform like [AWS Lambda](https://aws.amazon.com/lambda/) or [Google Cloud Functions](https://cloud.google.com/functions/docs/), which can be configured automatically run the script and push the data for a fee. This platforms are very reliable, and are often the best choice for complex data projects.

However, recall that one of the allures of generating your own datasets is the fact that it can much cheaper than simply purchasing a dataset. In this post, I will propose a workflow that leverages [**Github Actions**](https://docs.github.com/en/free-pro-team@latest/actions) for small scale data collection projects, which **adds the benefits of cloud-based serverless computing without (necessarily) adding cost**.

## What are Github Actions?

Github Actions are integrated into Github repositories, and can be used to run small workloads on cloud servers owned, operated, and maintained by Github. Here are a few of the most common use cases for Github Actions:

- Automatically run tests when new code is checked into a repository, before merging it into the overall codebase. This is known as [**Continuous Integration**](https://en.wikipedia.org/wiki/Continuous_integration).
- Automatically release new code to end users, typically after passing the suite of tests. This is known as [**Continuous Delivery**](https://en.wikipedia.org/wiki/Continuous_delivery). 
- Scheduled jobs. For example, you might want to aggregate some metrics on changes in a repository and send it to relevant stakeholders.

For more information about the capabilities (and limitations) of Github Actions, I would reccomend checking out the official [Quickstart for GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions/quickstart). 

In this case, we can use the scheduling capabilities of Github Actions to run our data collection script on a defined schedule, and let the Github servers handle the collection and pushing of data. 

> **Note**: AWS Lambda, Google Cloud Functions, and Github Actions all have both free and paid tiers, and could fill this use case. However, my experience is that Github Actions are a better choice for small scale projects due to the generosity of the free tier, ease of use, and close integration with Github, where a project's code is typically already living.

## An Example: Twitter Profile Scraper

As an example, I will walk through a simple data project I am currently working on. The overall goal of this project is to have an interesting visual way to see the tweets of the two leading candidates in the 2020 US Presidential Election, Donald Trump and Joe Biden. The crucial first step of this project is to start collecting this data. Here are some constraints I decided upon for this project:

- Data must be grabbed from Twitter on a near-real time basis. Tweets from last month are much less relevant than tweets from the last hour.
- The data should be easily accessible to anyone. Due to the small amount of data in this project, we will just store it on Github for the sake of simplicity.
- The data collection must be completely automated. 
- Credentials should be securely stored in the cloud.
- The project should be completely free.

> It should be noted that there are limitations to what Github Actions can be used for, especially while staying in the free tier. You should consult the [official documentation](https://docs.github.com/en/free-pro-team@latest/actions/reference/usage-limits-billing-and-administration) when deciding if this platform is right for your project. 

