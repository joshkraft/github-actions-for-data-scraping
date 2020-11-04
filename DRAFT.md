A common starting point for any data-driven project is the search for good data. There are two traditional approaches to gaining access to data:



| Option                               | Pros/Cons                                                  |
| ------------------------------------ | ---------------------------------------------------------- |
| 1. Buy a dataset from a vendor.      | High quality data, but data is expensive.                  |
| 2. Use a publicly available dataset. | Free or cheap, but must wait for data to become available. |
| 3. Generate your own dataset.        | Free or cheap, abut requires effort.                       |



For many small-scale or exploratory projects, Option 3 makes the most sense. If the data you are trying to collect is available on the web, there are two main ways to collect it: 

- Data can be [*scraped*](https://en.wikipedia.org/wiki/Data_scraping), by using tools to parse through the front end display of a website. This is considered a *brute force* approach, and is often discouraged (or blocked entirely) by major websites.
- Data can be *requested via an [API](https://en.wikipedia.org/wiki/API)*. This is considered the *elegant* approach, as the data is provided from the website in a structured format, though you will have to agree to the limits placed on your requests, such as a limit to the number of requests in a given period of time

Modern Python libaries such as [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) (for scraping) and [Tweepy](https://www.tweepy.org) (for accessing the Twitter API) have lowered the barrier to entry for gathering data through these methods, and serve as a great tool for generating novel datasets. 

However, one use case that is not addressed by these tools is the ability to routinely perform web scraping, either on a schedule or as a result of some other event. As an example, what if you were interested in building a dashboard to visualize a certain persons tweets in near-real time? 

One approach could be to construct a script to pull the persons tweets from the past hour, and then simply run that script each hour to pull new data. Or perhaps, you could use a scheduling program SUCH AS to automate the execution of the script. Of coure, this is not a sustainable approach, as it is dependent on the laptop where the script is living to be always on and available.

There are many ways that you could improve this process: AWS Lambda, Google Cloud Functions, etc. However, all of these tools cost money.

**Introducing Github Actions.**