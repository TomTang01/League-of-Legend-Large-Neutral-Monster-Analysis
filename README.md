#### Ziheng Tang, December 2025

# Introduction
This is a report about the statistics of large neutral monsters on the final result on the game. The main question to explore here is: Can we predict a winning team just by looking at some combinations of the large neutral monsters data? This report contains a great model that can predict wins given just the large neutral monsters statistics. The accuracy of the model is 92%. The data precessing phase will also showcase the importance of some of the statistics, to see hwich ones are more influencial to the final win. Other related questions that will be explored are: What are some interesting traits of the data for the large neutral monsters (atakhans, barons, dragons, and such)? And is the win rate the same for both blue team and red team?

### About the dataset
The dataset that the model was trained on was the official esports match data from OraclesElixir, from 2025, [here](https://oracleselixir.com/tools/downloads). It contains the end game statistics for random games during 2025, both for a single player and a team. There are a total of 118932 rows, 99110 rows of single player data and 19822 rows of team data. This report will focus on data for a whole team.
This dataset contains the information similar to the Results Screen after each game. Most columns in this dataset are useless to us. Only these columns are going to be considered:
 - `gameid`: the id for which game this data is collected from.
 - `result`: the win result.
 - neutral monsters :
   1. `firstdragon`: did the team get the first elemental dragon, containing values: [0., 1.]
   2. `dragons`: the total number of elemental drakes and elder the team killed, containing values: [0., 1., 2., 3., 4., 5., 6., 7.]
   3. `opp_dragons`: the total number of elemental drakes and elder the opponent team killed, containing values: [0., 1., 2., 3., 4., 5., 6., 7.]
   4. `elementaldrakes`: the total number of elemental drakes the team killed. Data is nan only when the data is considered 'partial'. Containing values: [0.,  1.,  2.,  3.,  4., nan]
   5. `opp_elementaldrakes`: the total number of elemental drakes the opponent team killed. Data is nan only when the data is considered 'partial'. Containing values: [0.,  1.,  2.,  3.,  4., nan]
   6. `dragons (type unknown)`: This column is special. It is not nan only when `datacompleteness` is 'partial'. This column represents the total number of elemental drakes the team killed, when the data is incomplete. We can also conclude that this column is probabily a main reason why `datacompleteness` is 'partial'. This column contains values: [0.,  1.,  2.,  3.,  4., nan]
   7. `elders`: the total number of elders the team killed. Data is nan only when the data is considered 'partial'. Containing values: [0.,  1.,  2.,  3., nan]
   8. `opp_elders`: the total number of elders the opponent team killed. Data is nan only when the data is considered 'partial'. Containing values: [0.,  1.,  2.,  3., nan]
   9. `firstherald`: Whether the team takes the first herald or not. Data is nan only when `datacompleteness` is 'partial'. Containing values: [0., 1., nan]
   10. `heralds`: The total number of heralds the team killed, containing values: [0., 1.]
   11. `opp_heralds`: The total number of heralds the opponent team killed, containing values: [0., 1.]
   12. `void_grubs`: The total number of void grubs killed by the team, containing values: [0., 1., 2., 3., 4., 5., 6.]
   13. `opp_void_grubs`: The total number of void grubs killed by the opponent team, containing values: [0., 1., 2., 3., 4., 5., 6.]
   14. `firstbaron`: Whether the team get the first Baron. Data is nan only when the data is considered 'partial'. Containing values: [0.,  1., nan]
   15. `barons`: The total number of Barons killed by the team, containing values: [0., 1., 2., 3., 4.]
   16. `opp_barons`: The total number of Barons killed by the opponent team, containing values: [0., 1., 2., 3., 4.]
   17. `atakhans`: Did the team kill the Atakhan. Data is nan when the data is considered 'partial'. But it is also nan when both teams do not kill the atakhan for the whole game. Containing values: [0., 1., nan]
   18. `opp_atakhans`: Did the opponent team kill the Atakhan. Data is nan when the data is considered 'partial'. But it is also nan when both teams do not kill the atakhan for the whole game. Containing values: [0., 1., nan]
 - `datacompleteness`: is the data complete or not, containing values: ['complete', 'partial']
 - `position`: 'team' for team data.
 - `side`: which side this data was obtained from. Containing values: ['blue', 'red']

# Data Cleaning

1. Extracted only the team data. After sampling data frome the full dataset, I am able to confirm that team data are data with the `'position'` column with the value 'team'. All other single player data have values such as 'top' or 'mid'. In addition, the other columns such as `'gold'` or `'experience'` all have significantly lower values for 'position' other than 'team'
2. Extracted only the columns listed above. These are all the columns related to large neutral monsters. There are 7 more columns for all the elemental drakes but I have decided to drop them by the 5th step below.
3. Looked closely at every column and verified their meanings as well as their possible values. The original dataset did not say what the columns represent. Some columns are related to other columns and their meanings can be interpreted by checking other columns. For example, the `'dragons'` column is a column that holds numbers that are the sum of `'elementaldrakes'` and `'elders'`, hence it must mean the sum of these two.
4. Made sure there are no unexpected values other than some nan. There is only 1 Atakhan, 1 Herald, at most 6 void grubs, and at most 4 elemental drakes per team, per game. It is verified that there are no unreasonable values for any columns.
5. Observed the reason why nan values are present. Most columns have nan when the data is partial, with the exception of `'dragons (type unknown)'`, which has nan when data is complete, and `'atakhans'` and `'opp_atakhans'` having extra nans when both teams did not kill the Atakhan. This step actually made me drop columns for different types of elemental drakes such as `'infernals'` or `'oceans'`. This is because `'dragons (type unknown)'` was found to contain values of just elemental dragons, which means that if I were to fill in values for each elemental drake, I have to consider that there can only be 4 in total and made things very hard to predict. This means that we lost data that we could have used, but since `'elementaldrakes'` contained this information partially, we do not need to worry too much.
6. Set `'datacompleteness'` into binary variables, True if complete, False if partial. This would make later data analysis easier.
7. Columns such as `'gameid'` and `'datacompleteness'` are needed for data analysis and filling in missing values. These columns will be deleted after these steps.

The head of the final dataset I will use looks like this:
```markdown
| gameid           | side   | datacompleteness   |   firstdragon |   dragons |   opp_dragons |   elementaldrakes |   opp_elementaldrakes |   elders |   opp_elders |   heralds |   opp_heralds |   void_grubs |   opp_void_grubs |   firstbaron |   barons |   opp_barons |   atakhans |   opp_atakhans |   result |
|:-----------------|:-------|:-------------------|--------------:|----------:|--------------:|------------------:|----------------------:|---------:|-------------:|----------:|--------------:|-------------:|-----------------:|-------------:|---------:|-------------:|-----------:|---------------:|---------:|
| LOLTMNT03_179647 | Blue   | True               |             0 |         0 |             2 |                 0 |                     2 |        0 |            0 |         0 |             1 |            0 |                6 |            0 |        0 |            1 |          0 |              1 |        0 |
| LOLTMNT03_179647 | Red    | True               |             1 |         2 |             0 |                 2 |                     0 |        0 |            0 |         1 |             0 |            6 |                0 |            1 |        1 |            0 |          1 |              0 |        1 |
| LOLTMNT06_96134  | Blue   | True               |             0 |         3 |             2 |                 3 |                     2 |        0 |            0 |         1 |             0 |            6 |                0 |            1 |        1 |            0 |          1 |              0 |        1 |
| LOLTMNT06_96134  | Red    | True               |             1 |         2 |             3 |                 2 |                     3 |        0 |            0 |         0 |             1 |            0 |                6 |            0 |        0 |            1 |          0 |              1 |        0 |
| LOLTMNT06_95160  | Blue   | True               |             0 |         0 |             4 |                 0 |                     4 |        0 |            0 |         0 |             1 |            2 |                4 |            0 |        0 |            1 |          0 |              1 |        0 |
```

# Exploratory Data Analysis

## Univariate Analysis

By graphing out all the columns into histograms, it is confirmed that most columns contain reasonable values with reasonable distributions. For example, the graph of the `'dragons'` column does seem to agree with the fact that the column is the sum of `'elementaldrakes'` and `'elders'`, as seen below:

<iframe
  src="assets/dragons.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Most of the values fall under 0-4 and very few data have values exceeding 5. This means that this column cannot be just for elemental drakes due to the fact that there cannot be more than 4 elemental dragon kills for each team. But we still cannot conclude whether it is the sum of `'elementaldrakes'` and `'elders'` or some other columns. Let's take a look at the histogram for `'elementaldrakes'`:

<iframe
  src="assets/elementaldrakes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This histogram's distribution looks very very similar to the distribution of the values 1-4 in the histogram of `'dragons'`. One thing to mention is that elders spawn only when one of the teams killed 4 elemental drakes. This means that elder kills are not very likely for teams with less than 4 elemental dragon kills since they will be needing the opponent team killing 4 elemental drakes. Due to the fact that the `'elementaldrakes'` histogram looks like the `'dragons'` histogram so much, we can confirm their relationship if there are a lot of 0s for the `'elders'` column:

<iframe
  src="assets/elders.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This histogram for `'elders'` strongly proves my assumption, having basically all 0s and the percentage of 1s are around 3.97%, which is close to the 2.56% for 5s in the `'dragons'` histogram. All other columns have been ploted as well but their distributions are no where close to `'dragons'`. Hence we are safe to conclude that `'dragons'` is the sum of `'elders'` and `'elementaldraks'`. 

## Bivariate Analysis

The `'elders'` histogram raise another question for me: according to my understanding of the game, elder kills are extremely significant to the outcome of the game, and even with its very complicated spawn requirements, the graph should not be leaning so heavily towards 0. Another reason why this elders data is so malformed may be because a lot of the jungle players in this dataset are unskilled. Let's make a scatter plot of the `'gained_gold'` at the end of the match of each lane compared to the total `'gained_gold'` of the whole team. The original full dataset will be needed for this. We will randomly sample 1000 game ids because there are 19822 games in this dataset, the graph may look very messy if we graph all games.

<iframe
  src="assets/jungles.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This graph proves that the jungle players in this dataset is fine and not below average. It is a little bit hard to tell but jungle players are bunched up in the mid left of the graph, which is normal because usually the econ goes more to the mid, top, and bot. As long as jungles do not fall behind too much, they are doing their job.

Another columns that I want to focus on is the `'Void_Grubs'` column. According to my understanding of the game, killing 0 or 1 Void Grub will not impact you team's chance of winning. But killing 2 or more will. Let's take a look at the graph:

<iframe
  src="assets/void_grubs.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

It can be seen here that most games are lost for 0 or 1 kill, and most game are won for 2 or above. This suggests that my assumption is correct. Later on, I will binarize this column at a threshold of 1.

Let's take a step back and look at the win rate for all the large neutral monster statitics to see which ones are worth killing:

<iframe
  src="assets/monsters.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

'Killed less' means that the team killed less of this monster in this game than the opponent team. 'Killed more' means that the team killed more of this monster in this game than the opponent team. And 'killed the same' means that the two teams killed the same amount. Most columns seem reasonable, with 'killed more' having the highest win rates. The exception is still the `'elders'` column. However, as we have explored before, this is just due to the harsh spawn requirements for elders.
Here is a table of the win rate version of the same data:
```markdown
|                 |   dragons win rate |   elementaldrakes win rate |   elders win rate |   heralds win rate |   void_grubs win rate |   atakhans win rate |   barons win rate |
|:----------------|-------------------:|---------------------------:|------------------:|-------------------:|----------------------:|--------------------:|------------------:|
| killed less     |           0.145192 |                   0.218041 |         0.0914136 |         0.336697   |              0.358793 |           0.266976  |         0.0728484 |
| killed more     |           0.708506 |                   0.649379 |         0.060337  |         0.661185   |              0.536777 |           0.706488  |         0.694582  |
| killed the same |           0.146302 |                   0.13258  |         0.848249  |         0.00211886 |              0.104429 |           0.0265362 |         0.23257   |
```

As shown here and the graph above, the biggest difference between the win rate of killing more and killing less is the barons column. This means that barons are very essential and impacts the outcome of the game a lot. This is a column that must be considered as a feature for my models.

## Interesting Aggregates

Looking at all the 'first kills', which one or which combination would give the highest win rate?

```markdown
|            |       0.0 |      1.0 |
|:-----------|----------:|---------:|
| (0.0, 0.0) | 0.0919696 | 0.733429 |
| (0.0, 1.0) | 0.335839  | 0.866149 |
| (1.0, 0.0) | 0.169713  | 0.82553  |
| (1.0, 1.0) | 0.522269  | 0.920855 |
```
The top index is for `'firstbaron'`, and the side index is the tuple (`'firstdragon'`, `'heralds'`). Since there can only be one Herald, I will count it as the first kill as well. This table shows that getting more kills definately guarentees more wins significantly. It also shows that getting the first kill on the Baron is more essential than getting the Herald, which is more essential than getting the first elemental drake.

### Conclusion: 
After analysis all the columns and some relationships, I am certain that the rankings for the important of the monsters to the final win is:
1. The Elder Drake and the Baron (hard to say which is more important)
2. Elemental Drakes
3. Atakhan
4. Herald
5. Void Grubs

Note that Elemental Drakes are ranked higher than Herald, which may seem contradictory to previous analysis showing first dragon kills are not more important than first Herald kills. The short answer is that they are not contradictory, the first kill is different from the total number of kills.

# Assessment of Missingness

## NMAR

No columns has missingness of NMAR because all plots of the columns show reasonable distributions with appropriate values. This means that none of the missing values are missing because of their actual values. Another reason why none show be NMAR is because they are all MAR, as proved below。

## MAR and MCAR

As mentioned in the data preprocessing phase, data is missing if `'datacompletenss'` is 'partial', or after cleaning, is 'False'. We can further prove this by looking at histograms and permutation tests done on `'datacompletenss'` and other columns such as `'elementaldrakes'`:

<iframe
  src="assets/datacompletenes_elementaldrakes_missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

These 2 histograms show that when data is complete, there are no nan values for `'elementaldrakes'`, and when data is not complete, `'elementaldrakes'` are all nan.
Permutation tests on all all other columns with nan values other than `'atakhans'` and `'dragons (type unknown)'` all have p-values of 0, which very likely means that nan appear if and only if `'datacompletenss'` is True. `'Dragons (type unknown)'` is the exact opposite, with nan if and only if `'datacompletenss'` is False. After filling in the 0 for `'atakhans'` when both teams decide not to kill the Atakhan, this column also follows the rule of having nan if and only if `'datacompletenss'` is True. The plots and permutation test all show that all columns with missing values have missing values depending on `'datacompletenss'`. This means that all missingness are MAR, depending on `'datacompletenss'`.

```
Permutation Test Results ('datacompleteness' vs. elementaldrakes):
  Observed Mean Diff (Missing Group - Observed Group): 1.0000
  P-value: 0.0000 (Alpha=0.05)
  Conclusion: The relationship is STATISTICALLY significant

Permutation Test Results ('datacompleteness' vs. opp_elementaldrakes):
  Observed Mean Diff (Missing Group - Observed Group): 1.0000
  P-value: 0.0000 (Alpha=0.05)
  Conclusion: The relationship is STATISTICALLY significant

Permutation Test Results ('datacompleteness' vs. dragons (type unknown)):
  Observed Mean Diff (Missing Group - Observed Group): 1.0000
  P-value: 0.0000 (Alpha=0.05)
  Conclusion: The relationship is STATISTICALLY significant

Permutation Test Results ('datacompleteness' vs. elders):
  Observed Mean Diff (Missing Group - Observed Group): 1.0000
  P-value: 0.0000 (Alpha=0.05)
  Conclusion: The relationship is STATISTICALLY significant

Permutation Test Results ('datacompleteness' vs. opp_elders):
  Observed Mean Diff (Missing Group - Observed Group): 1.0000
  P-value: 0.0000 (Alpha=0.05)
  Conclusion: The relationship is STATISTICALLY significant

Permutation Test Results ('datacompleteness' vs. firstbaron):
  Observed Mean Diff (Missing Group - Observed Group): 1.0000
  P-value: 0.0000 (Alpha=0.05)
  Conclusion: The relationship is STATISTICALLY significant

Permutation Test Results ('datacompleteness' vs. atakhans):
  Observed Mean Diff (Missing Group - Observed Group): 0.9692
  P-value: 0.0000 (Alpha=0.05)
  Conclusion: The relationship is STATISTICALLY significant

Permutation Test Results ('datacompleteness' vs. opp_atakhans):
  Observed Mean Diff (Missing Group - Observed Group): 0.9692
  P-value: 0.0000 (Alpha=0.05)
  Conclusion: The relationship is STATISTICALLY significant
```

I have done a second round of permutation test for all columns with missing values with the column `'result'`, and all tests gave a p-value of 1, which means that the missingness id very likely not related to the result of the game. This means the missing values can be filled in without considering the `'result'` column.

```
Permutation Test Results ('result' vs. elementaldrakes):
  Observed Mean Diff (Missing Group - Observed Group): 0.0000
  P-value: 1.0000 (Alpha=0.05)
  Conclusion: The relationship is NOT statistically significant

Permutation Test Results ('result' vs. opp_elementaldrakes):
  Observed Mean Diff (Missing Group - Observed Group): 0.0000
  P-value: 1.0000 (Alpha=0.05)
  Conclusion: The relationship is NOT statistically significant

Permutation Test Results ('result' vs. dragons (type unknown)):
  Observed Mean Diff (Missing Group - Observed Group): 0.0000
  P-value: 1.0000 (Alpha=0.05)
  Conclusion: The relationship is NOT statistically significant

Permutation Test Results ('result' vs. elders):
  Observed Mean Diff (Missing Group - Observed Group): 0.0000
  P-value: 1.0000 (Alpha=0.05)
  Conclusion: The relationship is NOT statistically significant

Permutation Test Results ('result' vs. opp_elders):
  Observed Mean Diff (Missing Group - Observed Group): 0.0000
  P-value: 1.0000 (Alpha=0.05)
  Conclusion: The relationship is NOT statistically significant

Permutation Test Results ('result' vs. firstbaron):
  Observed Mean Diff (Missing Group - Observed Group): 0.0000
  P-value: 1.0000 (Alpha=0.05)
  Conclusion: The relationship is NOT statistically significant

Permutation Test Results ('result' vs. atakhans):
  Observed Mean Diff (Missing Group - Observed Group): 0.0000
  P-value: 1.0000 (Alpha=0.05)
  Conclusion: The relationship is NOT statistically significant

Permutation Test Results ('result' vs. opp_atakhans):
  Observed Mean Diff (Missing Group - Observed Group): 0.0000
  P-value: 1.0000 (Alpha=0.05)
  Conclusion: The relationship is NOT statistically significant
```

Conclusion: all missingness are MAR, none are MCAR or NMAR

## Filling in Nan Values

Since some Nan values can be computed by other columns or some other information, we will do that first.

1. First, we will deal with the easy one: `atakhans` and `opp_atakhans`. Fill in the Nan for `atakhans` and `opp_atakhans` with 0 if `'datacompletenss'` is True, since previous analysis above proves that this column has Nan values when both teams do not kill the atakhans but the data is complete.

2. Next, we will look at the `'elementaldrakes'` column. Again, just to clearify, I am not considering the columns for individual lemental dragons. These columns also have missing values, but they will be extremely hard to fill in due to the fact that there is an additional rule for having only a maximum of 4 elemental dragons killed per team. This means that if I want to fill in the missing values for any of these elemental dragons with statistics like the mean or the median or even random numbers extracted from a distribution, we may find games having more than 4 elemental dragons killed for a single team, making this invalid. Hence, I have decided to drop all the types of elemental dragons and just keep `elementaldrakes` and `opp_elementaldrakes`. I can fill in the missing values for these 2 columns because I know the missing data from `dragons (type unknown)`. Recall that `dragons (type unknown)` holds data for `elementaldrakes` when `'datacompletenss'` is False.

3. The remaining columns with missing values are `'elders'`, `'opp_elders'`, `'atakhans'`, `'opp_atakhans'` and `'firstbaron'`. But since these missingness are related to `'datacompleteness'`, yet all are Nan when `'datacompleteness'` is False, we are not able to fill in the values based on `'datacompleteness'`. Instead, we will use the distribution of `'elders'`, `'atakhans'`, and `'firstbaron'` from existing columns, grouped by the value in `'result'`, to fill in the missing values, since we know killing more would result in a more likely win, which means that winning should mean a more likely kill of these monsters. Then, we we will fill in the `'opp_atakhans'` and `'opp_elders'`columns, since we know that they must match the corresponding columns for the opponent. Note that this method may result in both teams in a game having the first Baron or Atakhan, but since we extracted the values out of the distribution, it is very unlikely and it will not affect future predictions too much. It just would not make sense when looking at this data.

All missing values are now filled in.

# Hypothesis Testing

### 1. Hypothesis Test for `'elders'`

When analyzing the dataset, we saw that barely any team killed any elder dragon. As I have pointed out in previous steps, elder dragon kills are much rarer than any other large neutral monster kills due to it having very strict spawn rules. According to my rankings of importance of the monsters back in the analysis step, killing a elder dragon is very important. ChatGPT agrees with me and says that any elder dragon kills for a team will increase their win rate up to 70%. But I think the win rate is even higher. Let's conduct a hypothesis test to reject ChatGPT.

 - `Null Hypothesis`: Given that any elder dragons were killed by a team, the team's win rate is 70%
 - `Alternative Hypothesis`: Given that any elder dragons were killed by a team, the team's win rate is more than 70%

 - `Statistic`: Win rate given more than one elder dragon was killed by the team
 - `alpha`: 0.05

Results: The win rate for at least 1 elder dragon kill is 0.8217592592592593, which is the observed test statistics. The p-value is 0.0, hence we are safe to reject the Null at a significance level of 0.05. ChatGPT is very likely wrong. But it does confirm that my assumption for elder kills are very important for the final win. The `'elders'` column would be an excellent choice for the feature vector for my baseline model.

### 2. Hypothesis Test for `'side'`

A question that has bugged me for some time is whether the side (Blue or Red) influences the win rate. This is supposed to be a 5v5 fair game, but it would be interesting if being on one side would increase the chance of winning. Let's do a hypothesis test to check.

 - `Null Hypothesis`: Being on the Blue side has 50% chance of winning.
 - `Alternative Hypothesis`: Being on the Blue side does not have a 50% chance of winning.

 - `Statistic`: abs((percentage of games where Blue won) - 0.5)
 - `alpha`: 0.05

 Results: The chance of Blue winning is 0.5338512763596004, which is the observed test statistic. The p-value is 0.0, which means that we are safe to reject the Null at a significance level of 0.05. The chance of Blue winning is not 50%. I have confirmed this with the gaming community and it seems that the Blue has a slightly higher win rate of 52%. It seems that the game is not balanced by sides, and I may want to test if my model, later on, is fair for both the Blue team and the Red team.

 # Framing a Prediction Problem

The final Prediction problem for this project is simple:

Given statistics of the large neutral monster kills of a team, predict whether the team will eventually win the game or not. This is a binary classification problem.

 - `Inputs`: combinations of the game metadata: `'firstdragon'`, `'dragons'`, `'opp_dragons'`, `'elementaldrakes'`, `'opp_elementaldrakes'`, `'elders'`, `'opp_elders'`, `'heralds'`, `'opp_heralds'`, `'void_grubs'`, `'opp_void_grubs'`, `'firstbaron'`, `'barons'`, `'opp_barons'`, `'atakhans'`, and `'opp_atakhans'`
 - `Output`: prediction of `'result'`

I am predicting the `'result'` because it is the column that tells whether the team won at the end or not very straight forward. The evaluation metric I am going to use is accuracy. This is because since all data came from both sides of a game, and there must be a win and a lose for each game, the amount of wins are the exact same as the amount of loses. This means that the data is perfectly balanced and accuracy is a valid metric. I will not be prioritizing precision because I have no reason to ignore data with 0 for `'result'`, as precision looks only at `'result'` = 1. I will not be prioritizing recall because I have no reason to ignore my predictions that are 0, as recall looks only at my predictions that are 1. Accuracy is a very balanced evaluation metric that takes all data into account and suits this binary classification problem very much.

The data is split into training data and testing data. All models will have the same training data and testing data for better comparison.

# Baseline Model

The baseline model would be a decision tree that only considers how many more `'barons'` and `'elders'` a team has killed than the opponent team. That is, create 2 new features: 
  1. `'barons_diff'` = `'barons'` - `'opp_barons'`; 
  2. `'elders_diff'` = `'elders'` -`'opp_elders'`. 
Use these 2 column values to determine whether the team wins or not. There is no need for any transformers because both these columns are linear and their scales would not matter for a decision tree. Both the features are discrete columns.

Use GridSearchCV to test out the best hyperparameters for this tree. The hyperparameters considered are:
```
hyperparams = {
    'max_depth' : [2, 4, 8, 16],
    'min_samples_split' : [100, 1000, 2000],
    'criterion' : ['gini', 'entropy']
}
```
The best hyperparameter combination is `{'criterion': 'entropy', 'max_depth': 4, 'min_samples_split': 1000} ` and the Best Cross-Validation Accuracy is 0.8183. Using this hyperparameter combination to train the full training data and test on the testing data, the baseline model accuracy on test set is 0.8164. Meaning that the best Decision Tree model with these 2 features correctly predicts the game result 81.64% of the time.
This is a decent model given its high accuracy. We know from the Aggregates section above, that the win rate for getting more barons than the opponent team is around 70%. And from the hypothesis test done on the `'elders'` column, we know that killing at least one elder has a win rate of over 70%. By combining these feature, we are getting a higher accuracy of an additional 10%, which is decent. But there are so many more features we did not use. By using all these features, we may get an even better accuracy.

# Final Model

The final model would be a Random Forest using all the features of the large neutral monsters statistics. I choose Random Forest because it is more stable and hard to overfit. And since the Decision Tree Baseline Model gives a decent result, making it a forest would be a reasonable step.
```
Features: `'firstdragon'`, 
`'elementaldrakes'`, `'opp_elementaldrakes'`, 
`'elders'`, `'opp_elders'`, 
`'heralds'`, `'opp_heralds'`, 
`'void_grubs'`, `'opp_void_grubs'`, 
`'firstbaron'`, 
`'barons'`, `'opp_barons'`, 
`'atakhans'`, and `'opp_atakhans'`
```

`'void_grubs'` and `'opp_void_grubs'` will be binarized because, as stated in the data analysis phase, killing 1 or 0 Void Grub will not impact the final win in any way. And killing 2 or above is considered useful but there is no significant difference after 2.

All other columns are quatitative columns and will be standardized just to make the final parameters more interpretable. Some quantitative columns will be skipped because they are more like binary columns, having only 0s and 1s. These includes: `'firstdragon'`, `'heralds'`, `'opp_heralds'`, `'firstbaron'`, `'atakhans'`, and `'opp_atakhans'`.

Since all of these features are in are independent from each other for a team, and all these feature show that killing more would guarentee a more likely win, it is a logical descision to include all these features. Note that I excluded `'dragons'` and `'opp_dragons'` because these 2 columns are dependent on `'elementaldrakes'`, `'opp_elementaldrakes'`, `'elders'`, and `'opp_elders'`. Hence including all six would mean that there are dependent features, which would not help our prediction very well.

1. Transform the columns using the `ColumnTransformer` to binarize and standardize the columns.
2. Set the hyperparameter choices for GridSearchCV to try:
  ```
  hyperparams = {
    'rf__n_estimators': [10, 50, 100],
    'rf__max_depth' : [4, 8, 16, 32],
    'rf__min_samples_split' : [5, 10, 100, 1000],
    'rf__criterion' : ['gini', 'entropy']
  }
  ```
3. Use GridSearchCV to find the best parameters.
  - Best Parameters Found: {'rf__criterion': 'entropy', 'rf__max_depth': 16, 'rf__min_samples_split': 10, 'rf__n_estimators': 50}
  - Best Cross-Validation Score: 0.9277
4. Use the best hyperparameter combination to train on the full training data and then test on the test set.
  - Model Accuracy on Test Set: 0.9314
  - Interpretation: The best All Features Random Forest model correctly predicts the game result 93.14% of the time.

### Conclusion:
The final model has a test accuracy of 0.9314, which is very high. It is better than the base model because it contains more features, all correlated to the final win result and independent from each other. Also because Random Forests are more robust than a single Decision Tree.

# Fairness Analysis

As we proved in the hypothesis test, the data hints that blue and red teams do not have a even 50/50 chance of winning. Let's see if my All Features Random Forest model is fair enough to have equally accurate predictions for both teams. We will check this by using a permutation test to see if they are from the same distribution.

 - **Null Hypothesis**: Our model is fair. The accuracy for predicting the right result for blue team and red team are roughly the same, and any differences are due to random chance.
 - **Alternative Hypothesis**: Our model is unfair. The accuracy for predicting the right result for blue team and red team are not the same

 - **Statistic**: abs((percentage of games where Blue was predicted correctly) - (percentage of games where Red was predicted correctly))
 - **alpha**: 0.05
 - **Evaluation Metric**: accuracy

I chose accuracy again for the same reason above: I want to get all of my predictions right, not just the predicted wins or just the predicted loses. The total amount of red team data in the test set is 962, and out of these 962 data, there are 438 with actual wins. The total number of blue team data in the test set is 1021, and out of these 1021 data, there are 541 with actual wins. Hence the data is stll balanced and accuracy is a valid metric.

Result: The accuracy of Blue predictions from the model is 0.9324191968658179. The accuracy of Red predictions from the model is 0.9106029106029107. Therefore, we failed to reject the Null, the p-value is 0.0774. The accuracy of blue and red win predictions are very similar. The model is very likely to be fair.

### Conclusion:
The final All Features Random Forest model acheives accuracy parity.