# Using Social Media to Map Floods in the United States

General Assembly | Data Science Immersive <br>
Authors: *Andrea Yoss, Grace Powell, Dimitri Kisten*

---
---


## Table of Contents

- [Problem Statement](#introduction) 
- [Executive Summary](#methodology)
- [Data](#Data)
- [Modelling](#model)
- [Mapping](#mapping)
- [Conclusions](#conclusion)
- [Recommendations, & Further Application](#further)
- [Sources](#sources)


# Problem Statement  <a id='introduction'></a>

According to the National Oceanic and Atmospheric Administration (NOAA) the amount of billion dollar (adjusted for inflation) natural disasters per year since since 1980 has been on a steady uptrend with 3 of such events in 1980 and 14 in 2019. Although this number has increased dramatically for the United States as well as the rest of the world, the total number of deaths has fallen dramatically. Part of the reason for this is that people are able to recieve alerts and warnings much faster which allows them more time to prepare and evacuate. Many weather organizations have in place systems to alert the public about natural disasters and which areas are affected. *But what if there are locations that are out of the system's reach? Can these weather systems always give precise mapping of a storm? What if power goes out in a widespread area causing a system to be delayed or innacurate?* **In the face of some of these challenges, social media, specifically Twitter, can be leveraged to accurately detect and map various types of natural disasters**. Twitter allows us to search for tweets pertaining to the event we are looking to map as well as provide geolocation of where that tweet was sent from, and a timestamp of when the tweet was sent. Using these features we can build a model to validate and then map tweets pertaining to an event. This will ultimately allow for a more up to date service first responders can use to reach isolated civilians in the even of a crisis.

# Executive Summary <a id='methodology'></a>

For this project we decided to focus on floods within the United States. To obtain flood data we scraped Tweets using the Twitter Scraper API and the GetOldTweets3 API. These APIs allowed us to scrape tweets with certain keywords pertaining to floods, from a certain location, within a certain timeframe. In order to create a model that validated whether or not a tweet was really about a flood we needed to create data to train on. We used the OPENFema Dataset of Disaster Declaration Summaries to determine which flood tweets from which locations and time ranges would be assigned a target variable of '1' for 'flood' and which tweets would be assigned '0' for no flood or false positive. Using the '.setMaxTweets' parameter we made sure each scrape had an equal amount of data to ensure balanced classes. Our data consisted of 2702 tweets split between tweets pertaining to floods during a flood event and during a period of no floods.  

Each tweet contained the text, geolocation, username, and timestamp. The initial step in cleaning was to drop duplicate tweets and to then clean the actual text.  We removed unecessary characters (https, symbols, etc) and words that did not provide useful information or were not part of the english dictionary. There was also some manual inspection to make sure that tweets actually supported our classification assumption of 'flood' vs 'no flood.' Each dataframe for each state and time period was then combined into one.

To find the best model we applied gridsearch and pipelines to our data. We used Count Vectorizer along with Tifdf Vectorizer on Logistic Regression, Multinomial Naive Bayes, Decision Tree Classifier, Bagged Trees Classifier, and Random Classifier models. For each step of modelling we increased the size of the dataset with additional scraping until we arrived at roughly 2700 rows of data where we obtained our best score on our Count Vectorizer Logistic Regression model. 


# Data <a id='data'></a>

Tweets were gathered using two APIs
* TwitterScrapper
* GetOldTweets

|File name| Description|
|---|---|
|[FEMA Disaster Dataset](./datasets/DisasterDeclarationSummaries.csv)| Summary of all federally declared disasters from 1953. Contains 3 disaster declaration types (major disaster, emergency, and fire management)|
|[FEMA Disaster Dataset - Floods](./datasets/df_fema.csv)| A cleaned dataset of the aforementioned FEMA Disaster Dataset to only include floods. |
|[Scraped Tweets](./datasets/Minnesota_tweets.csv)| Description|
|[State and County Data](./dfmap/.csv)| Description|
|[Dataset5](./datasets/.csv)| Description|
|[Dataset6](./datasets/.csv)| Description|


# Modelling <a id='model'></a>

We decided to optimize our model for sensitivity because in the case of natural disasters we rather say there is a natural disaster when there isn't one than say there isn't one when there is. The reason is that the effect of false negatives has much more detrimental effects than false positives (injuries, death, damage vs over preparedness).

**Baseline Model**: 0.493

#### Model Scores
<img src="./Visualizations/scores.png"  width="1000" height="350">


# Mapping <a id='mapping'></a>

Our mapping of floods was in part based on actual data from our FEMA and in part generated coordinates using that data. Since our Twitter API could not provide county specific or geolocation of a tweet we made a pre-production model that would represent what our map might look like given the access to that data. The dark blue points represent data from our FEMA dataset indicating which counties were affected by floods in 2019. The light blue points represent normally distributed generated data around those counties which might represent mapping if we had access to that data. 

<img src="./Visualizations/map1.png"  width="700" height="400">

<img src="./Visualizations/map2.png"  width="600" height="400">


# Findings & Conclusions <a id='conclusion'></a>
Our primary findings from the model are that it is easier to validate tweets pertaining to floods on a state by state basis rather than on a national level. This is most likely because discussions on Twitter can be state specific so NLP in one state is not the best to apply in another state. If we were to make a production model we would most likely have to make state specific models as incorporating all states into one model caused a significant decrease in our scores. Another thing to consider from our models is that we can't ONLY optimize for specificity or sensitivity given the context of our problem. False negatives and false positives both have adverse effects and highly favoring one over the other has detrimental effects. For example, the goal of this model would be to inform the population of the scope of a natural disaster as well as direct aid to locations affected by natural disasters. If we have too many false negatives then the number of people who will not receive aid increases and many others will not know the full scope of an affected area which could further cause injury / damage. On the flip side if we have too many false positives, we could incur costs on a population as they spend extra time and resources preparing for something that is not imminent. More importantly this might increase the adverse effects of false negatives because now you have aid and resources being directed to an area that is not actually affected which takes away aid from actual affected areas (true positives). Both of these things had to be considered when selecting the 'best' model.

# Further Applications & Limitations <a id='further'></a>
From our map it's evident that most of the floods in the United States in 2019 are from the midwest and south with very few elsewhere which means that the robustness of our model will vary from state to state. We won't have as much data to train and test on in states were floods are rare. Additionally we found that not as many people tweeted about floods; most of our Tweets were from weather accounts and government agencies so the actual 'social mapping' concept is limited as there are very few people live tweeting a flood, or it didn't provide much more information than the tweets generated by weather/gov't accounts. It is a possibility as the popularity of Twitter increases that the intended functionality of the model increases. Addtionally, privacy is increasingly a concern for all social media users and most people opt to not include their city/county or state location in their tweet which also limits the functionality of our model.

One way to get around this would be to scrape the following or followers list of a user and scan those users tweets/bios to pinpoint a probable location of that user by getting data from his/her friends. For example if a user has a large majority of their followers/following who have city X in their bio or as their tweet location, we can predict that this person is likely from city X. We could also scrape the user's past tweets to see if at any point they had their location on when tweeting, or scrape the content of their tweets and liked tweets to see which location is mentioned the most. 

# Sources<a id='sources'></a>


* NOAA: https://www.ncdc.noaa.gov/billions/time-series
* Economist: https://www.economist.com/graphic-detail/2017/08/29/weather-related-disasters-are-increasing
* APIs Used: 
    * Twitter Scraper: https://pypi.org/project/twitterscraper/0.2.7/
    * GetOldTweets: https://pypi.org/project/GetOldTweets3/
