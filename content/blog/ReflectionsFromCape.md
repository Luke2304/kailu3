---
title: Getting started with Riot's API
date: 2018-07-05

categories:
- Python
- API

tags:
- Python
- League of Legends
- API
---
If you saw my post on Doublelift, you might remember that I took my data from Tim “Magic” Sevenhuysen's [Oracle Elixir](https://oracleselixir.com/match-data/). He provides data on league tournament data free of charge, which is awesome. However, I wanted to try to scrape my own League of Legends data.

Initially, I had the idea of scraping my own match data in R using `Selenium` and `selector gadget` on OP.GG. However, I soon came to realize that the data was quite messy and would take quite an effort to format cleanly into a dataframe.  

Then I found out that Riot had its own [Developer API](https://developer.riotgames.com/), which made my life so much easier. To sign up for a Developer API key, you simply need a League Account.

After playing around with the API's `GET` requests a bit, I found the API endpoint I needed to get data from a specific match: `/lol/match/v3/matches/{matchId}`

![API](/img/API_001.jpg)

To get the match ID for a game, simple go to your [match history](https://matchhistory.euw.leagueoflegends.com/en/#match-history/NA1/202611910) and copy the first 10 digits in your match URL. For instance in

![URL](/img/URL.jpg)

`2787975451` would be that match's Identifier.

Since this match happened on North American Servers, we will have to find the right regional endpoint for it.

![ENDPOINT](/img/endpoints.jpg)

Sending a `GET` request should be pretty easy now that we have all our information. In terminal, we can simply type (you can also copy paste the link into a browser):

```{python}
curl https://na1.api.riotgames.com/lol/match/v3/matches/{match_id}?api_key={KEY}

# replace {match_id} with your specific match id and {KEY} with your generated API key
```

And we get:

![BLEEDINGEYES](/img/bleedingeyes.jpg)

YIKES!

Well, at least we know that it's working. To avoid bleeding your eyes out, I suggest downloading [RestClient](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo), which is an extension that handles HTTP requests pretty well.

![Restclient](/img/Restclient.png)

MUCH BETTER! 

The type of the data from Riot's API is a Dictionary, which makes Python a perfect candidate to practice data wrangling. I picked up Python two weeks ago and I've found it to be an amazing language to work with data.

I won't be going in detail of how I dealt with reformatting the data. This project will be in my [Github repo](https://github.com/Luke2304/Riot-API-scraper).

One problem I ran into when reformatting the data into CSV format is that the column orders are alphabetical, which in my opinion makes working with the data more frustrating in the future. I learned that I could print out all the columns and reformat them as I liked:

```{python}
cols = list(dataDF.columns.values)
print(cols)

dataDF = dataDF[Consts.IDEAL_LIST_ORDER]
```
Here is the column order I ended up with

```
IDEAL_LIST_ORDER = ['participantId', 'summonerId','accountId', 'summonerName', 'profileIcon', 'teamId',
                    'championId', 'matchHistoryUri','currentAccountId','currentPlatformId',
                    'highestAchievedSeasonTier', 'win', 'champLevel', 'spell1Id', 'spell2Id',
                    'combatPlayerScore', 'kills', 'deaths', 'assists', 'doubleKills', 'tripleKills',
                    'quadraKills', 'pentaKills', 'killingSprees', 'largestCriticalStrike', 'largestKillingSpree',
                    'largestMultiKill', 'longestTimeSpentLiving','damageDealtToObjectives', 'damageDealtToTurrets',
                    'damageSelfMitigated', 'totalDamageDealt', 'totalDamageDealtToChampions','totalDamageTaken',
                    'trueDamageDealt', 'trueDamageDealtToChampions', 'trueDamageTaken', 'physicalDamageDealt',
                    'physicalDamageDealtToChampions', 'physicalDamageTaken', 'magicDamageDealt', 'magicDamageDealtToChampions',
                    'magicalDamageTaken', 'visionScore', 'sightWardsBoughtInGame', 'visionWardsBoughtInGame',
                    'wardsKilled', 'wardsPlaced','firstBloodAssist', 'firstBloodKill', 'firstTowerAssist',
                    'firstTowerKill', 'timeCCingOthers', 'totalHeal', 'totalTimeCrowdControlDealt',
                    'totalUnitsHealed', 'totalMinionsKilled', 'goldEarned', 'goldSpent', 'totalPlayerScore',
                    'totalScoreRank', 'objectivePlayerScore', 'turretKills', 'unrealKills', 'inhibitorKills',
                    'neutralMinionsKilled', 'neutralMinionsKilledEnemyJungle', 'neutralMinionsKilledTeamJungle',
                    'item0', 'item1', 'item2', 'item3', 'item4', 'item5', 'item6', 'perk0', 'perk0Var1',
                    'perk0Var2', 'perk0Var3', 'perk1', 'perk1Var1', 'perk1Var2', 'perk1Var3', 'perk2',
                    'perk2Var1', 'perk2Var2', 'perk2Var3', 'perk3', 'perk3Var1', 'perk3Var2', 'perk3Var3',
                    'perk4', 'perk4Var1', 'perk4Var2', 'perk4Var3', 'perk5', 'perk5Var1', 'perk5Var2',
                    'perk5Var3', 'perkPrimaryStyle', 'perkSubStyle',  'platformId'
                    ]


```

Finally, I saved my dataframe as a CSV.

```{python}
dataDF.to_csv('match.csv', index=False)
```

Opening the file, it looks pretty good!

![match](/img/match.jpg)


## Moving Forward

So this only does data for one match currently. For multiple matches, we would simply have to loop it through multiple match ids and modify the program to append the new rows each time. Something cool I could do in the future is to run this program on an AWS instance, looping through hundreds of thousands of matches, and then uploading to an S3 bucket. I also want to look into scraping tournament data (which don't have match ids), but it seems that Riot's current API does not support this.


