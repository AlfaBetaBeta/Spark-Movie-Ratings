# Movie rating database analysis

## Introduction and background description

This repository aims at concisely summarising the analysis process of a movie rating dataset elaborated in `movieRatingsEDA.ipynb` and presenting the conclusions arising from such analysis. The dataset considered is well known in the technical literature, it stems from a [movie recommendation service](http://movielens.org/) and constitutes an established research asset, publicly available provided appropriate referencing is made.

In essence, the dataset comprises a sample of the movie & user grid of a given online streaming platform, alongside some basic attributes of these. Users assign movies a rating, spanning from 0.5 to 5.0 in a 0.5 increment scale. They are completely anonymised to preserve their privacy and are represented by unique integer indices. Aside from the rating, users may also assign an arbitrary number of tags to any movie. All the movies included in the dataset have at least one tag and/or one rating, whereas all users considered have a rating history of at least 20 movies. The timestamp at which a rating or a tag is assigned is also included, as well as some basic movie metadata (title and genre). Further details about the data can be found in `movieRatingsEDA.ipynb`.

The tool used to perform the analysis is the `pyspark.sql` module of Apache Spark. The notebook displays all the commands required to generate the outputs succinctly presented here.


## Scope and purpose of the analysis

The outburst of online streaming platforms for visual entertainment contents belongs to a wider phenomenon, namely the full digital transformation of the supply means of goods and services. Such phenomenon poses new challenges but also offers new ways of gaining insights to adjust to rapidly changing environments. This trade-off is typically encountered in online retail businesses in general, and also in the specific case of online streaming in particular.

The contents catalogue of one such platform is not bounded by physical space any longer but rather by other factors related to technology, business and law. This entails that users face a range of contents beyond what they can measure, which in an initially counterintuitive way makes the decision making harder than before, potentially leading users to cancel their membership. It is therefore relevant from the business perspective to understand the behaviour of users, adjust contents accordingly and ultimately assist users to better exploit the platform resources. This is the framework defining the current study, analysing movie catalogue and user data over 22 years.


## Analysis

The dataset is clean and has no missing values, which is a consistent with a well-established benchmark set. Although timestamps are an accurate metric, it is decided to cast them to a standard date type for ease of inspection. It also allows for time (period) related queries in a way that is easier to interpret, as is showcased in subsequent sections. A detailed inspection of the datasets in terms of distinct values is shown in the notebook, though it suffices to state here that each movie is unique in the `movie` metadata set and each combination (user, movie) is unique in the `ratings` dataset. That is, there are no conflicting ratings from the same user for the same movie.

The number of ratings per year displays some oscillations, as shown below, though since 2015 it has been consistently above its historical average (roughly equal to the rating count of year 2008).

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/ratings-per-year.png" width=100% height=100%>

If the rating frequency is inspected by month, May and November seem to be on average the months with historically more rating activity, a somewhat surprising outcome considering they precede the typical holiday months.

In terms of rating level distribution, 4.0 and 3.0 are significantly more frequent than all other levels. The distribution below also reveals that users are more prone to giving ratings above 3.0. This is not irrelevant, as it points towards an essential setback of considering rating as a continuous variable, or even a multi-class discrete variable: the metric may be consistent across movies but it is not perceived consistently across users. The same level of appreciation does not translate into the same rating for different users. A potential remedy is to set a representative rating threshold and transform the rating into a binary feature.

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/rating-histogram.png" width=100% height=100%>

All movies listed in the `movies` dataset have been reviewed or tagged (or both) at least once. The majority of movies (8,170) have a rating but not a tag, some (1,554) have both a rating and a tag, and a small minority (18) have a tag but not a rating. The predominant movie group (rated untagged movies), filtered by the condition of movies having more than 30 ratings, can be summarised by the range of average rating and the range of rating count below.

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/rating-boundaries.png" width=65% height=65%>

The small group of tagged unrated movies, alongside the corresponding users and tags is also shown below for illustration.

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/tagged-unrated-movies.png" width=90% height=90%>

Unfortunately, most of these tags are not useful to infer how the users might rate the movies, since the tags relate to movie availability or simply replicate the movie genre.

Following the preliminary data exploration presented so far, a basic investigation on service-oriented business queries is carried out and concisely summarised hereafter. Further details regarding the query handling can be found in the notebook.

### What is the predominant (frequency based) genre per rating level?

The classic 'pure' genres `Drama` and `Comedy` predominantly appear on all rating levels, even though there are roughly 1,000 genre combinations (separated by pipes in the `movies` dataset). To some extent this could be expected since these two 'pure' genres account approximately for 20% of all movies in the dataset.

<p align="justify">
  <img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/rating-per-genre.png" width=30% height=30%>
  <img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/count-per-genre.png" width=40% height=40%>
</p>

The top 10 most/least rated genres are also shown below for further illustration.

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/most-least-rated-genres.png" width=70% height=70%>

*Unconventional* genre combinations (e.g. `Horror|Romance|Sci-Fi`) barely have any rating (solely one user) since such movies are uncommon and typically not compelling to a wide audience. Users may be reluctant to genre mixings that they are not used to and it may therefore be hard to expose these titles for user exploration.

### What is the predominant tag per genre and the most tagged genres?

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/tags-per-genre.png" width=65% height=65%>

About half of the most frequent tags are actually platform related, as they refer to the corresponding movie queue status instead of the movie contents. This may be convenient for the users, since it facilitates their movie list handling but it is not always relevant beyond this context. Tag insights can only be generated if they actually refer to the movie contents filtered by the tagging user experience of the movie. This also invalidates tags that simply repeat the movie genre.

As expected, `Drama` and `Comedy` appear in a notable majority of the most tagged genres due to their prevalence in the dataset.

### What are the most predominant (popularity based) movies?

Consider the top 10 movies in terms of the number of ratings (top) and the average rating (bottom, based on movies with more than 30 ratings) shown below.

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/top10-rating-count.png" width=70% height=70%>
<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/top10-rating-average.png" width=100% height=100%>

It is interesting to remark that only one movie appears in both lists (`Shawshank Redemption`), which clearly suggests that a distinction must be made between movies that are greatly appreciated and movies that are widely watched. In essence, and leaving aside many other possible considerations, these two groups would correspond to classics (high average rating by active users) versus blockbusters (high rating count).

### Do popular movies belong to a particular time frame?

Taking as reference the tables of popularity-based predominant movies from the previous query, 90% of the top 10 movies by number of ratings are from the 1990s decade. In the table of the top 10 movies by average rating, the distribution is:

* 30% from the 1990s
* 30% from the 1960s
* 20% from the 1970s
* 10% from the 1950s
* 10% from the 2000s

As comparison, this is the distribution of release dates over the whole dataset:

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/release-date-distribution.png" width=100% height=100%>

Most movies were released in the 2000s or the 1990s, though in fairness the 2010s decade is not completely represented in this dataset. It is noteworthy that movies from the 1950s- 1970s are amongst the most popular (by average rating) despite comprising significantly fewer titles than more recent times. In contrast, for instance, there is no movie whatsoever from the 1980s in either of the top 10 lists despite this decade representing 12% of the movies.

Put into perspective, though, 50 or 60 years ago the cinematographic industry could afford to produce less movies per year than nowadays, and the ones persisting in time (hence being incorporated into online platform catalogues) are largely classics prone to attract the audience (at least the audience subset of *active* users).

### What would a basic recommendation engine predict for selected users?

A collaborative-filtering based engine is trained with the aim of predicting ratings. The details of the recommender instantiation can be found in the notebook. Although it would be of use to predict the rating of unrated movies based on the tags, this is not possible at this stage with the current dataset, as such cases are very limited and the tag contents is mainly uninformative.

Focus is hence shifted to retrieving the top recommendation for the most active users (i.e. with the largest number of ratings). That is, retrieving the movie with highest predicted rating for that selected set of users. Such set is summarised below for reference, together with the recommendations.

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/ratings-active-users.png" width=60% height=60%>
<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/recommendations-active-users.png" width=100% height=100%>

The rating is considered a continuous variable, as can be seen by the predictions going beyond the theoretical upper bound (5.0). Consistently with the recommendation being the top one per user, all predicted ratings are considerably above their historical average.

If focus is placed on the release year, with the exception of users 474 and 288 (whereby interestingly these two have the lowest average movie year), all other users are suggested to watch older movies than the average they rate. This is consistent with the previous finding that there are notably less movies from past decades but they are more likely to be appreciated by active users. This is also reinforced by the fact that all users have been exposed to older movies (from the 30s or earlier) before.

<img src="https://github.com/AlfaBetaBeta/Spark-Movie-Ratings/blob/master/img/movie-year-active-users.png" width=75% height=75%>


## Conclusions

The movie online streaming business is indeed consolidated, and poses challenges unseen by traditional business models. Insights via user experience to adjust the platform contents and improve the user assistance are essential for business sustainability. In this regard, the following can be concluded from the present analysis:

* Over the last years, the trend has consolidated for users to actively rate more than the historical average, and they do so largely with positive ratings. Measures to ensure rating metric consistency across users might be advisable, as not all users perceive equally the same rating level.

* Tags are functional and convenient for users to manage their lists, but they seem to constitute an underexploited resource. Merely reflecting a queue status or replicating a movie genre is not sufficient for prediction/recommendation purposes, and initiatives to incentivise users to tag more comprehensively could be rewarding.

* Unconventional genre combinations are hard to expose for user exploration, as such movies may be perceived as picturesque if outside of the users' comfort zone. These movies do have a viewer's niche, though, and could greatly benefit from a well-trained recommendation engine.

* When designing the movie repository available to users, attention must be paid to differentiating between classics (high average rating) and blockbusters (high rating count) whilst including both. Greatly appreciated movies and extensively consumed ones do typically attract different kinds of user that need accommodating. Classics constitute a small amount of items compared with the large number of movies released per year in recent decades, and will most likely become highly favoured movies for active users.
