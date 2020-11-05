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

However, oftentimes data needs to be gathered on a continuous basis. For example, a someone might be interested in gathering tweets about a certain topic on an hourly basis for analysis by stakeholders. The simplest approach would be to simply run their script(s) on a laptop and push the data into a repository somewhere. This would fall under the category of [on-premises approaches](https://en.wikipedia.org/wiki/On-premises_software) - the infrastructure for the project is supplied (and maintained) by the individual/team/company that owns the laptop. While this can be a good starting point for data projects, the dependendence on the availability/reliability of local hardware introduces a large liability into the system.

One way to eliminate this liability is to move the project onto a [cloud-based serverless computing](https://en.wikipedia.org/wiki/Serverless_computing) platform like [AWS Lambda](https://aws.amazon.com/lambda/) or [Google Cloud Functions](https://cloud.google.com/functions/docs/), which can be configured automatically run the script and push the data for a fee. This platforms are very reliable, and are often the best choice for complex data projects.

However, recall that one of the allures of generating your own datasets is the fact that it can much cheaper than simply purchasing a dataset. In this post, I will propose a workflow that leverages [**Github Actions**](https://docs.github.com/en/free-pro-team@latest/actions) for small scale data collection projects, which **adds the benefits of cloud-based serverless computing without (necessarily) adding cost**.

## What are Github Actions?

## An Example: Tweeter Profile Scraper

