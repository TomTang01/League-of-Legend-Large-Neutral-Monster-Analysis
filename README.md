# League of Legend Large Neutral Monster Analysis
#### Ziheng Tang, December 2025

## Introduction
This is a report about the statistics of large neutral monsters on the final result on the game. The main question to explore here is: Can we predict a winning team just by looking at some combinations of the large neutral monsters data? This report contains a great model that can predict wins given just the large neutral monsters statistics. The accuracy of the model is 92%. The data precessing phase will also showcase the importance of some of the statistics, to see hwich ones are more influencial to the final win. Other related questions that will be explored are: What are some interesting traits of the data for the large neutral monsters (atakhans, barons, dragons, and such)? And is the win rate the same for both blue team and red team?

#### About the dataset
The dataset that the model was trained on was the official esports match data from OraclesElixir, from 2025. It contains the end game statistics for random games during 2025, both for a single player and a team. This report will focus on data for a whole team. There are a total of 118932 rows, 99110 rows of single player data and 19822 rows of team data.
This dataset contains the information similar to the Results Screen after each game. The containing columns are of the following catagories:

1. **global identification data**: gameid, url, league, year, split, playoffs, date, game, patch, participantid, playername, playerid, teamname, teamid	
2. **pre-game data**: side, position, champion, ban1, ban2, ban3, ban4, ban5, pick1, pick2, pick3, pick4, pick5
3. **end results**: gamelength, result, kills, deaths, assists, teamkills, teamdeaths, doublekills, triplekills, quadrakills, pentakills, firstblood, firstbloodkill, firstbloodassist, firstbloodvictim, team kpm, ckpm, firstdragon, dragons, opp_dragons, elementaldrakes, opp_elementaldrakes, infernals, mountains, clouds, oceans, chemtechs, hextechs, dragons (type unknown), elders, opp_elders, firstherald, heralds, opp_heralds, void_grubs, opp_void_grubs, firstbaron, barons, opp_barons, atakhans, opp_atakhans, firsttower, towers, opp_towers, firstmidtower, firsttothreetowers, turretplates, opp_turretplates, inhibitors, opp_inhibitors, damagetochampions, dpm, damageshare, damagetakenperminute, damagemitigatedperminute, damagetotowers, wardsplaced, wpm, wardskilled, wcpm, controlwardsbought, visionscore, vspm, totalgold, earnedgold, earned gpm, earnedgoldshare, goldspent, gspd, gpr, total cs, minionkills, monsterkills, monsterkillsownjungle, monsterkillsenemyjungle, cspm, goldat10, xpat10, csat10, opp_goldat10, opp_xpat10, opp_csat10, golddiffat10, xpdiffat10, csdiffat10, killsat10, assistsat10, deathsat10, opp_killsat10, opp_assistsat10, opp_deathsat10, goldat15, xpat15, csat15, opp_goldat15, opp_xpat15, opp_csat15, golddiffat15, xpdiffat15, csdiffat15, killsat15, assistsat15, deathsat15, opp_killsat15, opp_assistsat15, opp_deathsat15, goldat20, xpat20, csat20, opp_goldat20, opp_xpat20, opp_csat20, golddiffat20, xpdiffat20, csdiffat20, killsat20, assistsat20, deathsat20, opp_killsat20, opp_assistsat20, opp_deathsat20, goldat25, xpat25, csat25, opp_goldat25, opp_xpat25, opp_csat25, golddiffat25, xpdiffat25, csdiffat25, killsat25, assistsat25, deathsat25, opp_killsat25, opp_assistsat25, opp_deathsat25
4. datacompleteness

We will only use the rows that are team data, and only the columns:
 - for easier to identify each game: `gameid`
 - win result : `result`
 - neutral monsters :
   1. `firstdragon`,
   2. `dragons`,
   3. `opp_dragons`,
   4. `elementaldrakes`,
   5. `opp_elementaldrakes`,
   6. `dragons (type unknown)`,
   7. `elders`,
   8. `opp_elders`,
   9. `firstherald`,
   10. `heralds`,
   11. `opp_heralds`,
   12. `void_grubs`,
   13. `opp_void_grubs`,
   14. `firstbaron`,
   15. `barons`,
   16. `opp_barons`,
   17. `atakhans`,
   18. `opp_atakhans`
 - to make missingness easier: `datacompleteness`, containing values: ['complete' 'partial']
 - to know if it is team data or single player data: `position`
 - to identify which side is the team on: `side`
