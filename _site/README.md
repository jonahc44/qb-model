# How Important is the Quarterback?
## Introduction
In football, it well known that the quarterback is the most important position on the team. While every position has an impact on the field, none of them have as much influence over the game as the QB. I have always found it interesting that in a sport with several positions and players that get one the field throughout a game, there is one positions amongst them that is so important. For this reason, I sought to use data as a tool to analyze just *how* important this one position truly is.

To tackle this, I pulled data from [Pro Football Reference](https://www.pro-football-reference.com/), taking quarterback season statistics from the years 2006-2024. I took both passing and rushing data from this period, whose data ended up being 1417 and 1367 rows respectively for a total of 2784 rows. While Pro Football Reference has several statistics to choose from, the ones that I focused on are:

- Team
- Year
- Wins: # of wins the team had in the regular season when the QB was starting
- Losses: # of losses the team had in the regular season when the QB was starting
- Ties: # of ties the team had in the regular season when the QB was starting
- Age as of December 31st of the current season
- Pass Completions
- Pass Attempts
- Passing Yards
- Passing Touchdowns
- Interceptions
- Total QB Rating (QBR): statistic created by ESPN to encapsulate a quarterback's entire performance
- Passer Rating: statistic that only applies to passing
- Sacks
- Sack Yards
- Rush Attempts
- Rushing Yards
- Rushing Touchdowns
- Fumbles

## Data Cleaning and Analysis
While all data from Pro Football Reference is all presented as intended, there were several steps I had to take in order to mold the data to work as I wanted it to. It was easiest for me to get the data I wanted from the [player passing stats pages](https://www.pro-football-reference.com/years/2024/passing.htm) as opposed to pulling from team statistics. This meant that the first and most important thing I had to do was reformat the data so that it was grouped by team and year, not by individual player. For counting stats, I simply summed where rows shared the same team and year. For aggregate stats like QBR and Passer Rating, I took their averages by multiplying each of them by the individual QB's games played, than divided that by the total games played that season. One last thing that I did during this process was get a count of the number of quarterbacks a team started throughout the season, and the percentage of starts that the QB who started the most got. I did this to get a "quarterback stability" metric. I followed a similar process with the rushing data, summing all of those stats since they are all counting data. 

The next step I did was to merge the rushing and passing data together. While not entirely necessary, it would make working with it a little easier. I also changed all team names that were different in the past to their current name throughout the entire dataset for simlar reason. The last thing I did was manually add some data, specifically the division that each team plays in as well as the type of stadium roof (i.e open, closed, or retractable) for each team's home venue.


| **team** |  **pass_yards** |  **pass_tds** | **qbr** | **age** |
|:-------|-------------:|-----------:|--------:|--------:|
| ARI    |         3924 |         17 | 55.7188 | 26.75   |
| ATL    |         2682 |         21 | 49.3    | 26      |
| BAL    |         3535 |         21 | 66      | 33      |
| BUF    |         3051 |         19 | 49.5    | 25      |
| CAR    |         3486 |         19 | 39.1562 | 31.5625 |



Now that the data was cleaned, I could start analyzing it. Below is a graph taking a look at passing yards league wide throughout the years: 
 <iframe
 src="yards_by_year.html"
 width="800"
 height="450"
 frameborder="0"
 ></iframe>
There is a clear upward trend in passing yards until 2015, and then stays fairly stagnant (aside from 2017). This informs me that I should design my future models with this trend in mind. Next I looked at the correlation between QBR and winrate by each team:
<iframe
src="winrate_vs_qbr.html"
width="800"
height="450"
frameborder="0"
></iframe>
There appears to be a linear trend between QBR and winrate, suggesting that it will be a powerful tool for prediction and something I should include in my models.

## The Prediction Problem
After analyzing the data, I had to determine what specifically I wanted to do with it. Ultimately, I decided that I wanted an inference model that takes in a team's QB statistics across a season and outputs a predicted winrate for that team. The overarching goal with this is to create a tool that can help analyze whether a QB is playing up to standard or not. Additionally, it can be used as an indicator that the rest of the team is over or underperforming relative to their QB's play. I chose regular season winrate as the response variable because I think it provides the most comprehensive analysis of how successful a team is in a given year.

## Baseline Model
As a starting point, I chose a few features to point into a linear regression model, two of which are quantitative and one which is nominal. These features are passing touchdowns, interceptions, and division. To scrutinize both my baseline and final models, I used mean absolute error in terms of games. In other words, I took mean abolute error of the winrates predicted by the models, and multiplied it by 17, the number of games in a NFL season. For simplicity, I will simply be referring to this metric as MAE. The MAE produced by the baseline model is 2.25 games. I think this is is a fairly successful result given the model's simplicity.

## Final Model
I made several changes going from the baseline to the final model. The first of these was adding several more features to the model. The quantitative, counting features I added were passing yards, passing touchdowns, pass attempts, pass completions, sacks, sack yards, rushing yards, rushing touchdowns. I added these stats because they all provide unique information to the model that it can choose how to weigh. I also added the two aggregate statistics in the dataset: QBR and passer rating. I added these because they provide the model with advanced stats that, through prior research, I anticipated to be good predictors. Additionally, I added the year that the row of data is from, the percentage of starts the most played QB had over the course of the season, and the stadium roof type. I added year because of the graph of passing yards in the data analysis section. The NFL is continually changing, and the standard for QB play is no different. I wanted to add year to the data to help the model account for historical trends. I added the percentage of starts as a metric of quarterback stability that the model can take into account. Lastly, I added stadium roof type because interestingly there is data that suggests that offenses play slightly worse in open roof stadiums. More information on that can be found [here](https://www.givemesport.com/how-stadium-types-affect-nfl-scoring/).

Next, I changed the counting stats from overall season data to per game data. The reason for this an issue I have yet to address up to this point: NFL seasons have only been 17 games since 2021. This means that for any year prior to 2021, quarterbacks have had one less game to add to their counting stats compared to those today. By taking averages instead of totals for those stats, this becomes less of an issue.

Lastly, I chose the type of model to use, and the hyperparameters to cross-validate. After testing many different models, I landed on two finalists: linear and ridge regression. While sometimes outperformed by linear regression, I went with ridge regression as the final choice. It performs better when working with worse baseline performance, making it the safer choice. The hyperparameters that I chose to tune with cross-validation are polynomial features and alpha. 

The MAE for this model is 1.71 games, making it a 0.54 game improvement over the baseline. While I would have like to have brought this closer to a MAE of 1.5, making a jump in accuracy of 0.54 is a sizeable improvement and makes the model more usable for analyzing quarterbacks and their teams' level of play.


