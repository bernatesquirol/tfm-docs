# Introduction and context

The goal of this paper is to evaluate the differences of behaviour between users in Twitter before and after the Spanish 2020 confinement, that went from 13th March 2020 to May 2020. The dataset we will analyse will be tweets from 18.000 Spanish twitter profiles, categorized in 3 types: politicians, journalists and random profiles.

In order to measure the impact in several ways we will create different models. The construction of each model will give us a new perspective, new insights and indicators for our dataset. We will investigate this derived indicators for each category of users. This work will have three different parts:

1. [Timeseries analysis](./timeseries.html)
2. [Bayesian model](./bayesian.html)
3. [Ego networks analysis](./networks.html)

First we will take a look at the data from a global perspective as well as explain how we gathered it. In order to measure the impact of confinement in user's activity. Then we will create two models: one based on timeseries intervention analysis and another based on a Bayesian model as simple as we can for each user. Finally we will introduce the concepts of ego-networks and ego-networks evolution in time and analyse it during the period of confinement. All of this will be done while trying to unveil the differences of the fitted models among the groups of users and study their characteristics.

## Database

We have a database with the details of each user, from the endpoint [`users/show`](https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-users-show) in Twitter API. This details include the number of followers/friends, total number of tweets/likes, and so on. We will define several computed user features in the paper, and add them as new columns to this database. We will be adding new features as we progress in the paper to this user database. They will all be styled like **`this`**.

We also have a *light* version of every user's timeline. A timeline is the collection of tweets a user has created. We use the endpoint [`statuses/user_timeline`](https://developer.twitter.com/en/docs/tweets/timelines/api-reference/get-statuses-user_timeline) from Twitter API to get the latest 3200 tweets for a given user. This is very problematic in our use case as different users have different tweeting frequencies and as we want to understand overall behaviour through time, the observation start is correlated to the frequency of the user.

<figure style="text-align:center">
    <iframe frameborder="0" style="width:100%;height:400px;" src="https://app.diagrams.net/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=DataFlowTFM.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1tN6wTtX5RHvi6GIQ_Q8gyoJfQpMo24I3%26export%3Ddownload"></iframe>
<figcaption>Fig.1 - Data flow.</figcaption>
</figure>


There are several features we could extract directly from the timeline that are useful, following **[paper here]**:

- **`social_ratio`**: the ratio of tweets that involve in some way other users
- **`retweet_social_ratio`**: how many social tweets are a retweet
- **`reply_social_ratio`**: how many social tweets are a reply
- **`quote_social_ratio`**: how many social tweets are a quote
- **`like_tweet_ratio`**: the number of tweets divided by the number of likes (we would also need the like timeline)
- **`frequency`**: how many tweets per day the user posts

Of course all of these features are for what we know of the user timeline. So these two fields are also important:

- **`observed_start`**: date at which the observation of the timeline starts
- **`observed_end`**: date at which the observation of the timeline ends



## Users

### Journalists

There is one common feature in the journalism world: the Twitter profile of the news media that a journalist writes for follows him on Twitter. And in fact, media pages nearly always only follow very relevant people and their own journalists. One other typical thing among journalists is that they use twitter for professional use. It is also typical for a journalist to have some variation of the word *journalist* in their bio. This two factors allow us to find journalists very easily: We search the top social media pages in Spain and we pick their friends (users being followed by), and check those that had some version of *periodista*, *journalist*,.... 

### Politicians

We picked the members of the [Congreso XIV Legistlatura](https://twitter.com/i/lists/1193835814754639872) list that contain all the members in Spanish **parliament** during the February-June 2020 period. Obviously there are only 350 congressmen in Spanish congress.

### Random users

Getting random users from Twitter for a given location (we only want Spanish users) is not an easy task. The problem with getting random users with the endpoints of the Twitter API, is that we will most likely introduce some kind of bias. In our method we relax several assumptions in order to get random users.

Our method works as following:

1. We call the [Standard search API](https://developer.twitter.com/en/docs/tweets/search/api-reference/get-search-tweets) with an empty `query=''` and a   `geocode='39.8952506, -3.4686377505768764, 600000km'` (minimum circumference that includes Spain), and `result_type=recent` and `count=100`
2. We filter those 100 tweets such that the creator of the tweet has a minimum of followers, friends and actions. The user $u_0$ will be the creator of the tweet. If we take directly user $u_0$ as our random user we would have a bias towards users that tweet a lot, high frequency.
3. Instead what we do is get one of the followers or the friends of $u_0$. We can get as much as 5000 in one API call to [`friends/ids`](https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-friends-ids) or [`followers/ids`](https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-followers-ids). Our final user will be a sample of the ids given by the call.

Our method gets two types of random users: *random_friends* are those that we find using ` friends/ids` call, and *random_followers* using `followers/ids`. The first group biased towards following a lot of people and the other biased towards having a lot of followers. Having these two sets is not ideal, and they are also biased, as we can see in $\text{Fig.1}$, but we avoid having biases in the frequency of tweet, that we will be analysing deeply.

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/random-types-bias.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.2 - Bias in random profile types (max 5000 followers/following).</figcaption>
</figure>


With the label **`type_profile`** we will set the user type for each user in the user profile database. The four possible values will be: `politician`, `journalist`, `random-friend` and `random-follower`.

## Temporality

There is one issue that affects the data that we gathered, specifically the user timelines, and can introduce some biases. Twitter API, can only give us the last 3200 tweets from a user, that means that we will have different starting dates from different users, and the starting date will be correlated with the frequency of tweet of the user, as we won't get as much back in time with the profiles that tweet a lot. It is important to know this bias and take it into account, as we will plot a lot of global timelines, for all users, and for user types. We should divide the timeseries value for the user starting date bias, every time that we plot a timeline.

<figure style="text-align:center">
    <iframe height='420' scrolling='no' src='../tfm-plots/intro-bias.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.3 - Bias in the starting date of our observations.</figcaption>
</figure>

## Overall behaviour

To start to gain intuition about our data, we will plot the number of tweets per week for all our timelines. We will compute the how many tweets per week a user created and we will normalize between weeks (so the more active week gets a 1 and the least active week gets a 0). The sum all of these levels of activity per week (divided by the starting date bias) together is plotted in $\text{Fig.3}$. We can already see a bump in the plot while the confinement.

<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='../tfm-plots/intro-overall-activity-2.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.4 - Activity per week for all our users.</figcaption>
</figure>



Also we can look at how the type of twitter activity changes. For example we can look at the mean ratio of retweets per week between all users.

<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='../tfm-plots/intro-overall-ratio-rt.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.4 - Retweet ratio mean for all users.</figcaption>
</figure>

In the following sections, we will try to define ways to analyse one user behaviour by creating simplistic models that gather the important information, and then we will gather all models to visualize the impact of COVID19 in Twitter.