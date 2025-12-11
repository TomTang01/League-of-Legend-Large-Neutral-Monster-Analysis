#### Ziheng Tang, December 2025

# Introduction
This is a report about the statistics of large neutral monsters on the final result on the game. The main question to explore here is: Can we predict a winning team just by looking at some combinations of the large neutral monsters data? This report contains a great model that can predict wins given just the large neutral monsters statistics. The accuracy of the model is 92%. The data precessing phase will also showcase the importance of some of the statistics, to see hwich ones are more influencial to the final win. Other related questions that will be explored are: What are some interesting traits of the data for the large neutral monsters (atakhans, barons, dragons, and such)? And is the win rate the same for both blue team and red team?

### About the dataset
The dataset that the model was trained on was the official esports match data from OraclesElixir, from 2025. It contains the end game statistics for random games during 2025, both for a single player and a team. There are a total of 118932 rows, 99110 rows of single player data and 19822 rows of team data. This report will focus on data for a whole team.
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

1. Extracted only the team data.
2. Extracted only the columns listed above.
3. Looked closely at every column and verified their meanings as well as their possible values. The original dataset did not say what the columns represent. Some columns are related to other columns and their meanings can be interpreted by checking other columns. For example, the 'dragons' column is a column that holds numbers that are the sum of 'elementaldrakes' and 'elders', hence it must mean the sum of these two.
4. Made sure there are no unexpected values other than some nan. There is only 1 Atakhan, 1 Herald, at most 6 void grubs, and at most 4 elemental drakes per team, per game. It is verified that there are no unreasonable values for any columns.
5. Observed the reason why nan values are present. Most columns have nan when the data is partial, with the exception of 'dragons (type unknown)', which has nan when data is complete, and 'atakhans' and 'opp_atakhans' having extra nans when both teams did not kill the Atakhan. This step actually made me drop columns for different types of elemental drakes such as 'infernal' or 'ocean'. This is because 'dragons (type unknown)' was found to contain values of just elemental dragons, which means that if I were to fill in values for each elemental drake, I have to consider that there can only be 4 in total and made things wery hard to predict. This means that we lost data that we could have used, but since 'elementaldrakes' contained these information partially, we do not need to worry.

