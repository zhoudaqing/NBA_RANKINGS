Does the World Really Need Another Statistical Prediction Model?
================================================================

When Donald Trump won the 2016 presidential election, both sides of the political spectrum were surprised. The prediction models didn't see it coming and the Nate Silvers of the world took some heat for that (although Nate Silver himself got pretty close). After this, a lot of people would probably agree that the world doesn't need another statistical prediction model.

So, should we turn our backs on forecasting models? No, we just need to revise our expectations. George Box once reminded us that statistical models are, at best, useful *approximations* of the real world. With the recent hype around data science and "money balling" this point is often overlooked.

What *should* we then expect from a statistical model? A statistical model should help us process the many moving parts that often affect a given outcome -- such as an election -- by providing a single prediction for the future. It does this by combining the multitude of inputs, assumptions, trends and correlations with a simplified representation of how the world works. The human mind cannot do this and that is what makes models so valuable. However, if the inputs are bad even a good model is going to be wrong. Moreover, no model predicts the unthinkable unless unthinkable assumptions are made.

Forecasting and Basketball
==========================

So what does this have to do with basketball? In many ways, basketball is a perfect case study for forecasting. The outcome of future games are affected by many different factors; a team's current performance, momentum, the strength of its roster compared to its opponents, as well as the travel schedule. If a team looks great on paper and it's winning games, it'll likely do well in the future. But factors like injuries, coaching changes, and trades can curtail success very quickly. Thus, any model-based prediction is only accurate until something happens that is unaccounted for in the model.

This post discusses a new data-driven approach to predicting the outcome of NBA games. I call this the *Elastic NBA Rankings*. If you love statistics and basketball, then this is the post for you.

The model is based on techniques frequently used across various industries to predict bankruptcy, fraud or customer buying behavior. While it's certain that the model is wrong -- all models are -- I'm hoping that it will also be useful.

Summary -- What You Need to Know About the Elastic NBA Rankings
===============================================================

The Elastic NBA Team Rankings is a dynamic ranking algorithm that is purely based on statistical modeling techniques commonly used across most industries. No qualitative data or judgment is used to decide the ranks or the importance of different variables; the only human judgment applied is the underlying framework behind the algorithm.

At a high level, the model depends on three overall factors:

-   Previous performance.
-   How the team looks on paper. This is measured by the roster composition of "player archetypes."
-   Circumstances -- e.g., traveling, rest days, home-count advantage.

The team rankings produced by this model mainly agree with other prediction models (such as FiveThirtyEight) -- at least when it comes to identifying the strong teams and the weak teams (the "tail teams"). There are some interesting differences, such as different rankings of the Atlanta Hawks, which we will analyze layer in this post using a model-decomposition approach.

Back testing for the 2015-2016 season showed promising results (see more details below), although I have not done any historical benchmark testing against other models. Time will be the judge.

What Does the Model Actually Predict?
-------------------------------------

The model predicts the outcome of future NBA games for the current season. It does not predict the points scored, only the probability of a given team winning.

Where to Find the Model Predictions
-----------------------------------

All rankings and scores can be found in [this github repo](https://github.com/klarsen1/NBA_RANKINGS). The easiest way to extract the data is to directly read the [raw files](https://raw.githubusercontent.com/klarsen1/NBA_RANKINGS/master/modeldetails).There are two main files of interest:

-   game\_level\_predictions\_2016-MM-DD.csv -- Game-level predictions for each future game.
-   rankings\_2016-MM-DD.csv -- team rankings and predicted win rates.

In addition, the [modeldetails directory](https://github.com/klarsen1/NBA_RANKINGS/tree/master/modeldetails) has detailed information on the underlying mechanics of the model. See more details below on how to use this data.

How the Model Works
===================

The model is based on a three-step procedure:

1.  Create 25 data-driven *archetypes* using [k-means clustering](https://en.wikipedia.org/wiki/K-means_clustering) based on game-level box score statistics from games prior to the 2016-2017 season. The goal of the clustering algorithm is to maximize similarity of players (in terms of offensive and defensive stats) *within* clusters, while minimizing differences *between* clusters. Players are mapped to a given cluster based on their recent performance, which means that players can change archetype if their box score statistics change.
2.  The winner of a given game is predicted based on team archetypes, home team advantage, rest days, miles traveled, previous match-ups between the two teams (during that season), as well as recent win percentages.
3.  Teams are ranked based on the predicted win rate for the season. Hence the ranking is schedule-dependent.

Why is the model called "Elastic NBA Rankings?" There are two reasons for this: first, the model automatically adapts as the season progresses. Second, the regularization technique used to fit the logistic regression model is a special case of the [Elastic Net](https://en.wikipedia.org/wiki/Elastic_net_regularization).

Some Notes on the Model Used in Step 2
--------------------------------------

The model used to predict the winner of a given game is a statistical model that is estimated based on the most recent three seasons. Hence, the relative importance (weights) of the various drivers -- for example, the importance of roster features versus win percentages -- are purely based on the relationship detected from the data. For the stats-minded readers, the model is a logistic regression with an L1 penalty (lasso) to reduce the chance of over-fitting (this worked best in back-testing). V-fold cross-validation was used to choose the penalty parameter.

The model is re-estimated every single day and contains the following variables:

-   Roster composition -- surplus/deficit of minutes allocated to the different archetypes. For example, if a team's lineup has more players on the court of archetype 1 than its opponent, it'll have a surplus of minutes allocated to that archetype, and vice versa.
-   Trailing 90 day winning percentages -- the model assigns less importance to win-streaks early in the season. Moreover, the model assigns higher importance to wins where the opponent has a high [CARM-ELO](http://fivethirtyeight.com/features/how-our-2015-16-nba-predictions-work/) score.
-   Previous match-up outcomes -- for example, let's say Golden State is playing the Clippers and the two teams have played each other earlier in the season. This variable captures the outcomes of those two games. A team is more likely to win if it beat its opponent in the past.
-   Distances traveled prior to the game -- traveling the day before games usually translates into weaker performances, holding everything else constant.
-   Rest days prior to games -- more rest is beneficial during the long NBA season.
-   Home team advantage.

More Details on the Archetype Surplus/Deficit Variables
-------------------------------------------------------

For a given game, the model does the following calculations:

##### Team 1:

X\_1 = % of minutes allocated to cluster 1, X\_2 = % of minutes allocated to cluster 2, etc.

##### Team 2:

Z\_1 = % of minutes allocated to cluster 1, Z\_2 = % of minutes allocated to cluster 2, etc.

From these variables we construct the “delta variables” given by

D\_1 = X\_1 – Z\_1, D\_2 = X\_2 – Z\_2, etc.

These variables are then directly entered into the logistic regression model (labeled as share\_minutes\_cluster\_XX). The regression model then estimates the importance of each archetype. Hence, for team 1's roster to be considered strong, compared to team 2, it must have a surplus of minutes allocated to archetypes with large and positive coefficients, and vice versa for archetypes with negative coefficients.

How Are Players Assigned to Archetypes?
---------------------------------------

The outcome of the [k-means clustering](https://en.wikipedia.org/wiki/K-means_clustering) routine is a set of *centroids* where each centroid represents the box score profile of an archetype. Players are assigned to archetypes by matching their offensive and defensive box score statistics to closest centroids using the Euclidean distance, and hence can switch archetypes at any given time. A decay function was applied such that more recent games receive a larger weight. In addition, games played in the previous season are discounted by a factor of 4 (before the coefficients are estimated).

Predicting Allocation of Minutes for Future Games
-------------------------------------------------

In order to calculate the deficit and surplus variables referenced above, it's necessary to predict how many minutes each player will play. Currently, a 90-day trailing average is used (excluding the off-season). Games played during the prior season are discounted by a factor of 4 (before the coefficients are estimated).

Deciding the Winner of a Game
-----------------------------

The current implementation uses the estimated probabilities from the regularized logistic regression model to pick the winner of a given game. If the estimated probability of a given team winning exceeds 50%, then that team is declared the winner.

I've also been playing around with a simulation approach where each game is "played"" 1000 times, varying the distribution of minutes across archetypes in each iteration. For the season rankings, this did not alter the overall conclusion, although it did provide a better measure of prediction uncertainties.

For simulation playoffs I have been using the simulation approach. This will be covered in another post.

Model Rankings for the 2016-2017 Season
=======================================

All model rankings and results are stored in [this github repo](https://github.com/klarsen1/NBA_RANKINGS). The code below shows how to extract the current rankings and compare to the [FiveThirtyEight win/loss predictions](http://projects.fivethirtyeight.com/2017-nba-predictions/). The folks at FiveThirtyEight do amazing work and so this seems like a good sanity check.

Note that the rankings file stored github repo has three key columns:

-   ytd\_win\_rate -- this is the year-to-date win rate for the season.
-   pred\_win\_rate -- this is the predicted win rate for future games.
-   pred\_season\_win\_rate -- this combines the predicted games with the games that have been played. This is the statistic I'm using to rank teams in the example below.

``` r
library(tidyr)
library(dplyr)
library(knitr)
 
f1 <-
  "https://raw.githubusercontent.com/klarsen1/NBA_RANKINGS/master/rawdata/FiveThirtyEight_2016-11-20.csv"
f2 <-
  "https://raw.githubusercontent.com/klarsen1/NBA_RANKINGS/master/rankings/rankings_2016-11-20.csv"
 
ft8_rankings <- read.csv(f1) %>% rename(team=selected_team)
 
all_rankings <- read.csv(f2) %>%
  inner_join(ft8_rankings, by="team") %>%
  mutate(elastic_ranking=min_rank(-season_win_rate),
         FiveThirtyEight=min_rank(-pred_win_rate_538)) %>%
  select(team, conference, division, elastic_ranking, FiveThirtyEight) %>%
  arrange(elastic_ranking)
 
kable(all_rankings)
```

| team          | conference | division  |  elastic\_ranking|  FiveThirtyEight|
|:--------------|:-----------|:----------|-----------------:|----------------:|
| Cleveland     | East       | Central   |                 1|                2|
| Golden State  | West       | Pacific   |                 2|                1|
| LA Clippers   | West       | Pacific   |                 3|                2|
| Atlanta       | East       | Southeast |                 4|               12|
| Chicago       | East       | Central   |                 5|                6|
| Houston       | West       | Southwest |                 6|                6|
| San Antonio   | West       | Southwest |                 7|                4|
| Portland      | West       | Northwest |                 8|               13|
| Oklahoma City | West       | Northwest |                 9|                9|
| Toronto       | East       | Atlantic  |                10|                5|
| Charlotte     | East       | Southeast |                11|                9|
| Boston        | East       | Atlantic  |                12|               11|
| Utah          | West       | Northwest |                13|                6|
| LA Lakers     | West       | Pacific   |                14|               19|
| Memphis       | West       | Southwest |                15|               13|
| Milwaukee     | East       | Central   |                16|               23|
| Denver        | West       | Northwest |                17|               17|
| Indiana       | East       | Central   |                18|               21|
| Detroit       | East       | Central   |                19|               15|
| Orlando       | East       | Southeast |                20|               20|
| New York      | East       | Atlantic  |                21|               18|
| Minnesota     | West       | Northwest |                22|               16|
| Sacramento    | West       | Pacific   |                23|               25|
| Brooklyn      | East       | Atlantic  |                24|               29|
| Washington    | East       | Southeast |                24|               21|
| Phoenix       | West       | Pacific   |                26|               28|
| New Orleans   | West       | Southwest |                27|               27|
| Miami         | East       | Southeast |                28|               23|
| Dallas        | West       | Southwest |                29|               26|
| Philadelphia  | East       | Atlantic  |                30|               30|

The table shows that the elastic rankings generally agree with FiveThirtyEight -- at least when it comes to the "tail teams." For example, all rankings agree that Golden State, Cleveland and the Clippers will have strong seasons, while Philadelphia and New Orleans will struggle to win games.

But what about Atlanta? The elastic model ranks Atlanta fourth in terms of overall win percentage, while FiveThirtyEight ranks Atlanta at number 12 (as of 2016-11-20). To understand why the elastic model is doing this, we can decompose the predictions into three parts:

-   Roster -- archetype allocation deficits/surpluses. These are the variables labeled "share\_minutes\_cluster\_XX" described above. This group reflects the quality of the roster.
-   Performance -- e.g., win percentages, previous match-ups.
-   Circumstances -- e.g., travel, rest, home-count advantage

For the stats-oriented readers, here's how this works: the underlying predictive variables were multiplied by their respective coefficients, and then aggregated to get the group contributions to the predicted log-odds. The CSV file called score\_decomp\_2016\_MM\_DD contains this information. The code below shows how to use this file:

``` r
library(tidyr)
library(dplyr)
library(knitr)
library(ggplot2)
 
f <-
  "https://raw.githubusercontent.com/klarsen1/NBA_RANKINGS/master/modeldetails/score_decomp_2016-11-20.csv"
 
center <- function(x){return(x-median(x))}
read.csv(f, stringsAsFactors = FALSE) %>%
  select(selected_team, roster, circumstances, performance) %>%
  group_by(selected_team) %>%
  summarise_each(funs(mean)) %>% ## get averages across games by team
  ungroup() %>%
  mutate_each(funs(center), which(sapply(., is.numeric))) %>% ## standardize across teams
  gather(modelpart, value, roster:performance) %>% ## transpose
  rename(team=selected_team) %>%
  ggplot(aes(team, value)) + geom_bar(aes(fill=modelpart), stat="identity") + coord_flip() +
  xlab("") + ylab("") + theme(legend.title = element_blank())
```

![](readme_files/figure-markdown_github/unnamed-chunk-2-1.png)

Here's how to interpret the chart: the bars show the contribution from each part of the model. As expected, circumstances do not affect the overall prediction for the entire season as most teams have similarly taxing schedules. However, contributions from performance (weighted winning percentages) and roster (surplus of important archetypes) vary considerably.

First, let's take a look at Atlanta: the model thinks that Atlanta’s roster is just a hair above average (relative to its opponents). However, Atlanta has been winning a high percentage of their games lately -- they even beat Cleveland -- and thus we get a significant positive contribution from this (recall that the wins are weighted by the opponents [CARM-ELO](http://projects.fivethirtyeight.com/2017-nba-predictions/) rating). This essentially means that if Atlanta stops winning, the elastic model is going to downgrade Atlanta since the favorable view is mainly based on the performance and not its roster.

Next, let's look at Cleveland and Golden State. The model ranks these two teams at the top, both in terms of rosters and performance. In fact, my playoff simulations have these two teams meeting again in the finals and going to seven games (more on that in a later post).

Last, but not least, let's take a look at San Antonio and the Timberwolves -- two teams that are viewed very differently by the model. According to the model, San Antonio has been over-performing. The model does not like how the roster looks on paper, yet performance has been strong so far. This could be due to strong coaching, "corporate knowledge" (as Gregg Popovich calls it) and team chemistry -- factors that the roster component of the model does not capture. Minnesota, on the other hand, is under-performing according to the model; the roster is rated highly compared to its opponents, but the team is not performing well. This could be due to inexperience.

Backtesting
===========

The model was used to predict all games from 2015-11-20 to the end the 2015-2016 season, using only information available as of 2015-11-19 (including model coefficients). This is roughly a five month forecast window (the season ends in April); most teams played around 70 games during this period. I picked this date because the latest model-run for this post was 2016-11-20.

The game level accuracy for the entire period was 64.3% -- i.e., the model predicted the correct winner for 64.3% of games.

The [area under the ROC curve](https://en.wikipedia.org/wiki/Receiver_operating_characteristic) was 0.725.

The code below compares the actual rankings to the predicted rankings (note that this table only covers the games played after 2015-11-19):

``` r
library(dplyr)
library(knitr)
library(ggplot2)
 
f <-
  "https://raw.githubusercontent.com/klarsen1/NBA_RANKINGS/master/rankings/ranking_validation_2015.csv"
 
ranks <- read.csv(f) %>% select(team, rank_actual, rank_pred) %>%
  mutate(diff=abs(rank_pred-rank_actual)) %>%
  arrange(rank_pred) 

ggplot(ranks, aes(x=rank_pred, y=rank_actual)) +
  xlab("Predicted Rank") + ylab("Actual Rank") +
  geom_smooth(method='lm') + 
  geom_text(aes(label=team), hjust=0, vjust=1, size=2) + 
  geom_point(color="green", size=3) +
  geom_point(data=filter(ranks, diff>10), colour="red", size=3) 
```

![](readme_files/figure-markdown_github/unnamed-chunk-3-1.png)

The predicted team rankings were also in line with the actual team rankings (correlation is 66%), except for some significant misses like the Chicago Bulls, Denver and Portland. There might be good reasons why the model misjudged these teams so early in the season (for example, Chicago has been injury plagued). More investigation is needed here.

Next, let's check the game-level predictions for this period. The chart below shows the game-level accuracy for each team. The vertical lines show the overall match rate as well as the match rate we would get from a random draw (50%):

``` r
library(dplyr)
library(knitr)
library(scales)
source("https://raw.githubusercontent.com/klarsen1/NBA_RANKINGS/master/functions/auc.R") 

f <-
  "https://raw.githubusercontent.com/klarsen1/NBA_RANKINGS/master/rankings/game_level_validation_2015.csv"

game_level <- read.csv(f, stringsAsFactors = FALSE)

overall_match_rate=mean(as.numeric(game_level$selected_team_win==game_level$d_pred_selected_team_win))
 
mutate(game_level, match=as.numeric(selected_team_win==d_pred_selected_team_win)) %>%
  mutate(overall_match_percent=mean(match)) %>%
  rename(team=selected_team) %>%
  group_by(team) %>%
  summarise(team_match_percent=mean(match)) %>%
  ggplot(aes(team, team_match_percent)) + geom_bar(stat="identity") + coord_flip() +
  xlab("Accuracy") + ylab("") + theme(legend.title = element_blank()) + 
  geom_hline(yintercept = c(0.5, overall_match_rate), linetype=2) + 
  scale_y_continuous(labels = scales::percent)
```

![](readme_files/figure-markdown_github/unnamed-chunk-4-1.png)

``` r
print(paste0("AUROC = ", AUC(game_level$selected_team_win, game_level$prob_selected_team_win_d)[1]))
```

    ## [1] "AUROC = 0.725354145468453"

Note that the model is most accurate for the "tail teams" such as Golden State and Philadelphia, which is to be expected. There are a some teams where the model completely missed the mark -- e.g., Chicago. In the defense of the model, these predictions were only made 11 games into the season.

Future Development
==================

Dealing With Injuries and Trades
--------------------------------

As mentioned previously, the model depends on rosters *and* previous performance. If a team executes a major mid-season trade, the roster component will react immediately, while the performance component will be slower to react. There are a number of ways around this, but currently no special treatment is being applied at this point.

Player Interaction
------------------

The model currently does not capture any interaction between the archetypes. I have tested basic interaction terms, but that did not help much. More work is needed here.

Schedule-independent Team Rankings
----------------------------------

Currently, the model ranks teams by predicting win rates. This means that, holding everything else constant, the rankings implicitly favor teams that play in weaker divisions. A future development could be to have two rankings: one that predicts the win rate given the current schedule (this is what the model is currently doing), and one that normalizes for schedule differences.
