# Characterization of social behaviour in Twitter during COVID-19 pandemic in Spain

The goal of this paper is to evaluate the differences of behaviour of users in Twitter before and after the Spanish 2020 confinement, that went from 13th March 2020 to May 2020. The dataset we'll analyse will be tweets from 18.000 Spanish twitter profiles, categorized in 3 types: politicians, journalists and random profiles.

In order to measure the impact in several ways we will create different models. The construction of each model will give us a new perspective and some new insights and indicators for our dataset. We will analyse this derived indicators for each category of users. This work will have three different parts:

1. [Timeseries analysis](./timeseries.html)
2. [Bayesian model](./bayesian.html)
3. [Ego networks analysis](./networks.html)

First we'll view from a global perspective the data as well as explaining how we gathered it. Then we'll create a Bayesian model as simple as we can for each user, in order to measure the impact of confinement in user's activity. Finally we'll try to unveil the differences of the fitted model among the groups of users and study their characteristics.

### Database description

We have a database with the details of each user, from the endpoint [`users/show`](https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-users-show). This details include the number of followers/friends, total number of tweets/likes, and so on. We will define several computed user features along the paper, and add them as new columns to this database.

Also, we have for every user a *light* version of his timeline. The timeline, is the collection of tweets the user has created. We use the endpoint [`statuses/user_timeline`](https://developer.twitter.com/en/docs/tweets/timelines/api-reference/get-statuses-user_timeline) to get the latest 3200 tweets for a given user. This is very problematic in our use case as different users have different tweeting frequencies and as we want to understand overall behaviour through time, the observation start is related to the frequency of the user.

<figure style="text-align:center">
    <iframe frameborder="0" style="width:100%;height:400px;" src="https://app.diagrams.net/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=DataFlowTFM.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1tN6wTtX5RHvi6GIQ_Q8gyoJfQpMo24I3%26export%3Ddownload"></iframe>
<figcaption>Fig.1 - Data flow.</figcaption>
</figure>


There are several features we could extract directly from the timeline that are useful, following [paper here]:

- **`social_ratio`**: the ratio of tweets that involve in some way other users
- **`retweet_social_ratio`**: how many social tweets are a retweet
- **`reply_social_ratio`**: how many social tweets are a reply
- **`quote_social_ratio`**: how many social tweets are a quote
- **`like_tweet_ratio`**: the number of tweets divided by the number of likes (we would also need the like timeline)
- **`frequency`**: how many tweets per day the user makes

Of course all of these features are for what we know of the user timeline. So these two fields are also important:

- **`observed_start`**: date at which the observation starts
- **`observed_end`**: date at which the observation starts

We'll be adding new features as we progress in the paper to this user database. They will all be styled like **`this`**.

For the majority of computations, we'll cut the timelines in October 2019.

### Users

#### Journalists

One common thing in the journalism world is that the Twitter profile of the news media that a journalist writes for follows him in Twitter. And in fact the media pages nearly always only follow very relevant people and their own journalists. One other typical thing among journalists is that they use twitter for professional use. Nearly always a journalist has some variation of *journalist* in their bio. This two factors make getting a few journalist very easy: We search the top social media pages in Spain and we pick the friends of their Twitter accounts, and check those that had some version of *periodista*, *journalist*,...

#### Politicians

We picked the members of the [Congreso XIV Legistlatura](https://twitter.com/i/lists/1193835814754639872) list that contain all the members in Spanish parliament during February-June 2020. We could also had done something similar to the journalists procedure.

#### Random users

Getting random users from Twitter for a given location (we only want Spanish users) is not an easy task. The problem with getting random users with the endpoints of the Twitter API, is that we'll get some kind of bias. In our method we relax several assumptions for getting random users.

Our method works as following:

1. We call the [Standard search API](https://developer.twitter.com/en/docs/tweets/search/api-reference/get-search-tweets) with an empty `query=''` and a   `geocode='39.8952506, -3.4686377505768764, 600000km'` (minimum circumference that includes Spain), and `result_type=recent` and `count=100`
2. We filter those 100 tweets such that the creator of the tweet has a minimum of followers, friends and actions. The user $u_0$ will be the creator of the tweet. If we take directly user $u_0$ as our random user we would have a bias towards users that tweet a lot, high frequency.
3. Instead what we do is get one of the followers or the friends of $u_0$. We can get as much as 5000 in one API call to [`friends/ids`](https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-friends-ids) or [`followers/ids`](https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-followers-ids). Our final user will be a sample of the ids given by the call.

Our method gets two types of random users: ones that we call *random_friends*, are those that we find using ` friends/ids` call, and *random_followers* using `followers/ids`. We get two sets of "random" users, ones biased towards following a lot of people and the others biased towards having a lot of followers. Having this two sets is not ideal, and they are biased as we can see in $\text{Fig.1}$., but being aware that we have those biases 

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/random-types-bias.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.1 - Bias in random profile types (max 5000 followers/following).</figcaption>
</figure>

With the label **`type_profile`** we'll set the user type of each user in the user profile database. The four possible values will be: `politician`, `journalist`, `random-friend` and `random-follower`.



