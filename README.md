# League of Legends Objective Analysis: What Objective Control Says About Pro League of Legends
### Manning Karahashi

## Introduction

This project analyzes professional League of Legends match data from Oracle’s Elixir. The raw dataset contains both player-level rows and team-level summary rows for each match. Since my analysis focuses on team performance and match outcomes, I restrict the data to only the team-level rows so that each observation represents one team in one game.

The central question of this project is:

**Do teams that secure more neutral objectives tend to win professional League of Legends matches more often?**

This question matters because League of Legends is often described as an objective-based game. Teams fight for dragons, Rift Heralds, Barons, and other objectives to gain map control and convert that control into victories. Understanding how strongly objective control is associated with winning can help explain how professional matches are decided and can also inform predictive modeling.

After filtering to team-level rows, the cleaned dataset contains **20106 rows**.

The columns most relevant to this project are:

- `gameid`: unique identifier for each match
- `teamname`: the team represented by the row
- `position`: identifies whether the row is a player row or team row
- `result`: whether the team won (`1`) or lost (`0`)
- `dragons`: number of dragons secured by the team
- `heralds`: number of Rift Heralds secured
- `barons`: number of Barons secured
- `elders`: number of Elder Dragons secured
- `atakhans`: number of Atakhans secured
- `firstdragon`: whether the team secured the first dragon
- `firstherald`: whether the team secured the first Rift Herald
- `firsttower`: whether the team secured the first tower
- `golddiffat15`: gold difference at 15 minutes
- `xpdiffat15`: experience difference at 15 minutes
- `csdiffat15`: creep score difference at 15 minutes
- `side`: whether the team played on Blue or Red side
- `league`: professional league of the match
- `patch`: game patch version

To summarize overall neutral-objective control, I created a new feature called `total_objectives`, defined as the sum of dragons, heralds, barons, elders, and atakhans.

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The raw dataset includes both player-level rows and team-level rows. Since my question is about whether **team objective control** is associated with winning, I filtered the dataset to include only rows where `position == "team"`. This ensures that every row in the analysis represents the same unit: one team in one match.

Next, I converted objective-related and early-game columns such as `dragons`, `heralds`, `barons`, `elders`, `atakhans`, `result`, `golddiffat15`, `xpdiffat15`, and `csdiffat15` to numeric types. This was necessary because some of these values may be read in as strings from the CSV.

I then created a new feature:

\[
total\_objectives = dragons + heralds + barons + elders + atakhans
\]

This feature summarizes a team’s overall neutral-objective control and is used throughout the exploratory analysis and hypothesis test.

Below is the head of the cleaned DataFrame:

[| gameid           | datacompleteness   |   url | league   |   year | split   |   playoffs | date                |   game |   patch |   participantid | side   | position   |   playername |   playerid | teamname                | teamid                                  |   firstPick |   champion | ban1    | ban2     | ban3    | ban4     | ban5      | pick1   | pick2   | pick3    | pick4    | pick5    |   gamelength |   result |   kills |   deaths |   assists |   teamkills |   teamdeaths |   doublekills |   triplekills |   quadrakills |   pentakills |   firstblood |   firstbloodkill |   firstbloodassist |   firstbloodvictim |   team kpm |   ckpm |   firstdragon |   dragons |   opp_dragons |   elementaldrakes |   opp_elementaldrakes |   infernals |   mountains |   clouds |   oceans |   chemtechs |   hextechs |   dragons (type unknown) |   elders |   opp_elders |   firstherald |   heralds |   opp_heralds |   void_grubs |   opp_void_grubs |   firstbaron |   barons |   opp_barons |   atakhans |   opp_atakhans |   firsttower |   towers |   opp_towers |   firstmidtower |   firsttothreetowers |   turretplates |   opp_turretplates |   inhibitors |   opp_inhibitors |   damagetochampions |     dpm |   damageshare |   damagetakenperminute |   damagemitigatedperminute |   damagetotowers |   wardsplaced |    wpm |   wardskilled |   wcpm |   controlwardsbought |   visionscore |   vspm |   totalgold |   earnedgold |   earned gpm |   earnedgoldshare |   goldspent |      gspd |   gpr |   total cs |   minionkills |   monsterkills |   monsterkillsownjungle |   monsterkillsenemyjungle |    cspm |   goldat10 |   xpat10 |   csat10 |   opp_goldat10 |   opp_xpat10 |   opp_csat10 |   golddiffat10 |   xpdiffat10 |   csdiffat10 |   killsat10 |   assistsat10 |   deathsat10 |   opp_killsat10 |   opp_assistsat10 |   opp_deathsat10 |   goldat15 |   xpat15 |   csat15 |   opp_goldat15 |   opp_xpat15 |   opp_csat15 |   golddiffat15 |   xpdiffat15 |   csdiffat15 |   killsat15 |   assistsat15 |   deathsat15 |   opp_killsat15 |   opp_assistsat15 |   opp_deathsat15 |   goldat20 |   xpat20 |   csat20 |   opp_goldat20 |   opp_xpat20 |   opp_csat20 |   golddiffat20 |   xpdiffat20 |   csdiffat20 |   killsat20 |   assistsat20 |   deathsat20 |   opp_killsat20 |   opp_assistsat20 |   opp_deathsat20 |   goldat25 |   xpat25 |   csat25 |   opp_goldat25 |   opp_xpat25 |   opp_csat25 |   golddiffat25 |   xpdiffat25 |   csdiffat25 |   killsat25 |   assistsat25 |   deathsat25 |   opp_killsat25 |   opp_assistsat25 |   opp_deathsat25 |   total_objectives | atakhans_missing   |
|:-----------------|:-------------------|------:|:---------|-------:|:--------|-----------:|:--------------------|-------:|--------:|----------------:|:-------|:-----------|-------------:|-----------:|:------------------------|:----------------------------------------|------------:|-----------:|:--------|:---------|:--------|:---------|:----------|:--------|:--------|:---------|:---------|:---------|-------------:|---------:|--------:|---------:|----------:|------------:|-------------:|--------------:|--------------:|--------------:|-------------:|-------------:|-----------------:|-------------------:|-------------------:|-----------:|-------:|--------------:|----------:|--------------:|------------------:|----------------------:|------------:|------------:|---------:|---------:|------------:|-----------:|-------------------------:|---------:|-------------:|--------------:|----------:|--------------:|-------------:|-----------------:|-------------:|---------:|-------------:|-----------:|---------------:|-------------:|---------:|-------------:|----------------:|---------------------:|---------------:|-------------------:|-------------:|-----------------:|--------------------:|--------:|--------------:|-----------------------:|---------------------------:|-----------------:|--------------:|-------:|--------------:|-------:|---------------------:|--------------:|-------:|------------:|-------------:|-------------:|------------------:|------------:|----------:|------:|-----------:|--------------:|---------------:|------------------------:|--------------------------:|--------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-------------------:|:-------------------|
| LOLTMNT03_179647 | complete           |   nan | LFL2     |   2025 | Winter  |          0 | 2025-01-11 11:11:24 |      1 |   15.01 |             100 | Blue   | team       |          nan |        nan | IziDream                | oe:team:84bc703e28859788770611d94cf02ac |           1 |        nan | Vi      | Skarner  | Corki   | K'Sante  | Sylas     | Maokai  | Jinx    | Leona    | Hwei     | Gnar     |         1592 |        0 |       3 |       13 |         5 |           3 |           13 |             0 |             0 |             0 |            0 |            0 |              nan |                nan |                nan |     0.1131 | 0.603  |             0 |         0 |             2 |                 0 |                     2 |           0 |           0 |        0 |        0 |           0 |          0 |                      nan |        0 |            0 |             0 |         0 |             1 |            0 |                6 |            0 |        0 |            1 |          0 |              1 |            0 |        3 |            9 |               0 |                    0 |              1 |                  8 |            0 |                2 |               50143 | 1889.81 |           nan |                2908.79 |                    2642.49 |             7784 |            74 | 2.7889 |            24 | 0.9045 |                   23 |           158 | 5.9548 |       42255 |        24639 |      928.606 |               nan |       38793 | -0.112676 | -3.34 |        nan |           731 |            144 |                     nan |                       nan | 32.9774 |      14266 |    18519 |      313 |          15988 |        19181 |          345 |          -1722 |         -662 |          -32 |           0 |             0 |            1 |               1 |                 1 |                0 |      21498 |    28270 |      496 |          25335 |        28739 |          512 |          -3837 |         -469 |          -16 |           0 |             0 |            3 |               3 |                 9 |                0 |      29475 |    38659 |      656 |          35378 |        42861 |          702 |          -5903 |        -4202 |          -46 |           2 |             3 |            6 |               6 |                14 |                2 |      39226 |    50120 |      845 |          46192 |        55920 |          875 |          -6966 |        -5800 |          -30 |           3 |             5 |           10 |              10 |                26 |                3 |                  0 | False              |
| LOLTMNT03_179647 | complete           |   nan | LFL2     |   2025 | Winter  |          0 | 2025-01-11 11:11:24 |      1 |   15.01 |             200 | Red    | team       |          nan |        nan | Team Valiant            | oe:team:71bd93fd1eab2c2f4ba60305ecabce2 |           0 |        nan | Yone    | Viktor   | Aurora  | Nocturne | Jarvan IV | Varus   | Ivern   | Braum    | Renekton | Orianna  |         1592 |        1 |      13 |        3 |        36 |          13 |            3 |             0 |             0 |             0 |            0 |            1 |              nan |                nan |                nan |     0.4899 | 0.603  |             1 |         2 |             0 |                 2 |                     0 |           1 |           0 |        1 |        0 |           0 |          0 |                      nan |        0 |            0 |             1 |         1 |             0 |            6 |                0 |            1 |        1 |            0 |          1 |              0 |            1 |        9 |            3 |               1 |                    1 |              8 |                  1 |            2 |                0 |               53681 | 2023.15 |           nan |                2527.65 |                    2272.24 |            22322 |            78 | 2.9397 |            35 | 1.3191 |                   33 |           196 | 7.3869 |       53936 |        36320 |     1368.84  |               nan |       43425 |  0.112676 |  3.34 |        nan |           753 |            169 |                     nan |                       nan | 34.7487 |      15988 |    19181 |      345 |          14266 |        18519 |          313 |           1722 |          662 |           32 |           1 |             1 |            0 |               0 |                 0 |                1 |      25335 |    28739 |      512 |          21498 |        28270 |          496 |           3837 |          469 |           16 |           3 |             9 |            0 |               0 |                 0 |                3 |      35378 |    42861 |      702 |          29475 |        38659 |          656 |           5903 |         4202 |           46 |           6 |            14 |            2 |               2 |                 3 |                6 |      46192 |    55920 |      875 |          39226 |        50120 |          845 |           6966 |         5800 |           30 |          10 |            26 |            3 |               3 |                 5 |               10 |                  5 | False              |
| LOLTMNT06_96134  | complete           |   nan | LFL2     |   2025 | Winter  |          0 | 2025-01-11 12:06:37 |      1 |   15.01 |             100 | Blue   | team       |          nan |        nan | Esprit Shōnen           | oe:team:df682d5adfed257780a68a85004ed27 |           1 |        nan | Kalista | Aurora   | Skarner | Akali    | Ziggs     | Varus   | K'Sante | Ivern    | Azir     | Rell     |         1922 |        1 |      21 |       11 |        53 |          21 |           11 |             3 |             0 |             0 |            0 |            1 |              nan |                nan |                nan |     0.6556 | 0.9677 |             0 |         3 |             2 |                 3 |                     2 |           0 |           3 |        0 |        0 |           0 |          0 |                      nan |        0 |            0 |             1 |         1 |             0 |            6 |                0 |            1 |        1 |            0 |          1 |              0 |            1 |       11 |            2 |               1 |                    1 |              8 |                  2 |            3 |                0 |              100112 | 3125.24 |           nan |                3036.15 |                    4077.72 |            33435 |            94 | 2.9344 |            45 | 1.4048 |                   36 |           241 | 7.5234 |       64669 |        43687 |     1363.8   |               nan |       55950 |  0.128226 |  4.34 |        nan |           829 |            199 |                     nan |                       nan | 32.0916 |      18459 |    18423 |      315 |          15355 |        17354 |          282 |           3104 |         1069 |           33 |           8 |            15 |            4 |               3 |                 2 |                8 |      28328 |    28393 |      497 |          23259 |        26379 |          433 |           5069 |         2014 |           64 |          10 |            20 |            6 |               5 |                 8 |               10 |      38360 |    39561 |      657 |          31778 |        38944 |          594 |           6582 |          617 |           63 |          13 |            29 |            8 |               7 |                12 |               13 |      47876 |    52496 |      834 |          39499 |        48190 |          735 |           8377 |         4306 |           99 |          15 |            35 |            8 |               7 |                12 |               15 |                  6 | False              |
| LOLTMNT06_96134  | complete           |   nan | LFL2     |   2025 | Winter  |          0 | 2025-01-11 12:06:37 |      1 |   15.01 |             200 | Red    | team       |          nan |        nan | Skillcamp               | oe:team:b659295189ec2ddd1a30e39fd3a0bd8 |           0 |        nan | Fiora   | Nocturne | Ashe    | Leona    | Yone      | Corki   | Sejuani | Renekton | Orianna  | Nautilus |         1922 |        0 |      10 |       21 |        22 |          10 |           21 |             0 |             0 |             0 |            0 |            0 |              nan |                nan |                nan |     0.3122 | 0.9677 |             1 |         2 |             3 |                 2 |                     3 |           0 |           0 |        0 |        1 |           0 |          1 |                      nan |        0 |            0 |             0 |         0 |             1 |            0 |                6 |            0 |        0 |            1 |          0 |              1 |            0 |        2 |           11 |               0 |                    0 |              2 |                  8 |            0 |                3 |               78010 | 2435.28 |           nan |                4000.89 |                    3408.57 |             5633 |           101 | 3.153  |            41 | 1.2799 |                   32 |           248 | 7.7419 |       50679 |        29697 |      927.066 |               nan |       49208 | -0.128226 | -4.34 |        nan |           790 |            134 |                     nan |                       nan | 28.845  |      15355 |    17354 |      282 |          18459 |        18423 |          315 |          -3104 |        -1069 |          -33 |           3 |             2 |            8 |               8 |                15 |                4 |      23259 |    26379 |      433 |          28328 |        28393 |          497 |          -5069 |        -2014 |          -64 |           5 |             8 |           10 |              10 |                20 |                6 |      31778 |    38944 |      594 |          38360 |        39561 |          657 |          -6582 |         -617 |          -63 |           7 |            12 |           13 |              13 |                29 |                8 |      39499 |    48190 |      735 |          47876 |        52496 |          834 |          -8377 |        -4306 |          -99 |           7 |            12 |           15 |              15 |                35 |                8 |                  2 | False              |
| LOLTMNT06_95160  | complete           |   nan | LFL2     |   2025 | Winter  |          0 | 2025-01-11 13:07:47 |      1 |   15.01 |             100 | Blue   | team       |          nan |        nan | Karmine Corp Blue Stars | oe:team:343268a42863e8fc2767e02e4042de8 |           1 |        nan | Irelia  | Leona    | Ivern   | Sylas    | Olaf      | Corki   | Skarner | Alistar  | Aurora   | Aatrox   |         1782 |        0 |      18 |       22 |        30 |          18 |           22 |             2 |             0 |             0 |            0 |            0 |              nan |                nan |                nan |     0.6061 | 1.3468 |             0 |         0 |             4 |                 0 |                     4 |           0 |           0 |        0 |        0 |           0 |          0 |                      nan |        0 |            0 |             0 |         0 |             1 |            2 |                4 |            0 |        0 |            1 |          0 |              1 |            0 |        3 |           12 |               1 |                    0 |              7 |                 10 |            0 |                3 |               67870 | 2285.19 |           nan |                3924.95 |                    3225.08 |             8908 |            71 | 2.3906 |            23 | 0.7744 |                   22 |           168 | 5.6566 |       51389 |        31835 |     1071.89  |               nan |       49408 | -0.112542 | -1.47 |        nan |           717 |            146 |                     nan |                       nan | 29.0572 |      16335 |    18331 |      298 |          17250 |        18685 |          328 |           -915 |         -354 |          -30 |           5 |             7 |            5 |               5 |                10 |                5 |      26828 |    31544 |      476 |          26710 |        29554 |          519 |            118 |         1990 |          -43 |          10 |            13 |            6 |               6 |                12 |               10 |      35320 |    40785 |      625 |          37247 |        43800 |          729 |          -1927 |        -3015 |         -104 |          13 |            22 |           10 |              10 |                21 |               13 |      42735 |    50030 |      759 |          48356 |        54359 |          866 |          -5621 |        -4329 |         -107 |          14 |            25 |           14 |              14 |                32 |               14 |                  0 | False              |]

### Univariate Analysis

The plot below shows the distribution of `total_objectives`.

<iframe
  src="assets/total-objectives-histogram.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of `total_objectives` is right-skewed, meaning that most teams secure only a small number of major objectives, while relatively few teams secure many. This suggests that strong map control is somewhat uncommon and may help distinguish especially dominant team performances.

I also examined the distribution of `golddiffat15`. It is centered around 0, which makes sense because each match contributes one team with a positive gold difference and one with a negative gold difference. Most teams have relatively small gold differences at 15 minutes, suggesting that very large early-game leads are less common.

### Bivariate Analysis

The plot below compares `total_objectives` across winning and losing teams.

<iframe
  src="assets/total-objectives-by-result.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Winning teams tend to secure substantially more total objectives than losing teams. The upward shift in the distribution for winners suggests a strong association between objective control and match outcome.

I also examined win rate by `firstdragon` and found that teams who secure the first dragon tend to win more often than teams who do not. This suggests that early objective control may provide a meaningful advantage.

### Interesting Aggregates

To see whether this relationship holds across different regions, I created a pivot table showing the average number of total objectives for winning and losing teams in each league.

['| league      |   Loss Avg Objectives |   Win Avg Objectives |\n|:------------|----------------------:|---------------------:|\n| AL          |               2.19113 |              5.35495 |\n| ASI         |               2.38095 |              4.78571 |\n| Asia Master |               2.01111 |              4.85    |\n| CD          |               2.23588 |              5.49502 |\n| CT          |               2       |              5.88    |\n| DCup        |               2.28    |              5.41333 |\n| EBL         |               1.7644  |              5.34555 |\n| EM          |               2.20455 |              5.50909 |\n| EWC         |               2.09091 |              5.48485 |\n| FST         |               1.6     |              5.68571 |\n| HC          |               2.21    |              5.42    |\n| HLL         |               2.11312 |              5.46606 |\n| HM          |               2.12632 |              5.26842 |\n| HW          |               2.34211 |              5.20175 |\n| IC          |               2.17857 |              5.53571 |\n| KeSPA       |               1.73214 |              4.94643 |\n| LAS         |               1.92434 |              5.00613 |\n| LCK         |               2.20541 |              5.31532 |\n| LCKC        |               2.0369  |              5.47601 |\n| LCP         |               2.00344 |              5.46735 |\n| LEC         |               2.31699 |              5.81699 |\n| LFL         |               2.4434  |              5.83648 |\n| LFL2        |               2.21888 |              5.70386 |\n| LIT         |               2.17986 |              5.39568 |\n| LJL         |               1.81818 |              5.10963 |\n| LPL         |               1.9677  |              4.52422 |\n| LPLOL       |               1.83125 |              5.54375 |\n| LRN         |               2.25166 |              5.45695 |\n| LRS         |               2.3741  |              5.46043 |\n| LTA         |               1.75    |              5.825   |\n| LTA N       |               2.38498 |              5.57277 |\n| LTA S       |               2.14414 |              5.61261 |\n| LVP SL      |               2.22727 |              5.45833 |\n| MSI         |               2.075   |              5.7125  |\n| NACL        |               2.21377 |              5.77899 |\n| NEXO        |               2.42208 |              5.74675 |\n| NLC         |               1.93976 |              5.51004 |\n| PCS         |               1.86139 |              4.90099 |\n| PRM         |               2.25083 |              5.61716 |\n| PRMP        |               1.65753 |              5.12329 |\n| RL          |               2.43333 |              5.175   |\n| ROL         |               2.0625  |              5.61538 |\n| TCL         |               2.19583 |              5.60833 |\n| VCS         |               2.07801 |              5.56028 |\n| WLDs        |               1.9375  |              5.72917 |']

This table shows that, across many leagues, winning teams consistently secure more objectives on average than losing teams. This suggests that the relationship between objective control and winning is not limited to a single region, but appears broadly across professional play.

---

## Assessment of Missingness

A plausible **MNAR** column in this dataset is `atakhans`. One reason is that missing values in this column may reflect more than a random omission; they may depend on patch-specific rules or match conditions that are not fully captured elsewhere in the data. If the missingness depends on such unobserved factors, then the data may be Missing Not At Random.

At the same time, if we had more detailed metadata explaining whether the Atakhan objective was available or relevant in a given patch, then the missingness might instead be explainable as Missing At Random.

To study the missingness more concretely, I created an indicator column for whether `atakhans` is missing and performed permutation tests to determine whether its missingness depends on other observed columns.

First, I tested whether `atakhans` missingness depends on `patch`.

- **Observed TVD:** 0.301
- **p-value:** 0.001

Because the p-value is small, I reject the null hypothesis that missingness is independent of patch. This suggests that missingness in `atakhans` does depend on game version.

Next, I tested whether `atakhans` missingness depends on `side`.

- **Observed TVD:** 0.000
- **p-value:** 1.000

In this case, I fail to reject the null hypothesis. This suggests that `atakhans` missingness does **not** depend on whether the team is on Blue or Red side.

The plot below shows the permutation distribution of the TVD statistic for the missingness-vs-patch test.

<iframe
  src="assets/atakhans-missingness-permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed statistic lies far to the right of the null distribution, which supports the conclusion that missingness in `atakhans` depends on `patch`.

---

## Hypothesis Testing

To answer the main question of the project, I performed a one-sided permutation test comparing the average number of total objectives secured by winning and losing teams.

- **Null hypothesis:** The number of total objectives secured is independent of match outcome. Any observed difference is due to random chance.
- **Alternative hypothesis:** Teams that win secure more total objectives on average than teams that lose.

I used the following test statistic:

\[
\text{mean(total objectives | win)} - \text{mean(total objectives | loss)}
\]

This statistic is appropriate because `total_objectives` is a quantitative variable, and the question is specifically about whether winning teams secure more of them on average.

Using a significance level of 0.05, the observed test statistic was **3.247**, meaning that winning teams secured about 3.25 more objectives on average than losing teams.

After 10,000 permutations, the resulting p-value was **0.0001**. Since this is far below 0.05, I reject the null hypothesis.

This provides strong evidence that teams that win tend to secure more neutral objectives than teams that lose. While this does not prove causation, it does show a strong association between objective control and match success in professional League of Legends.

---

## Framing a Prediction Problem

After exploring the relationship between objectives and winning, I shifted to a predictive task: predicting whether a team will win or lose a match.

The response variable is `result`, where `1` indicates a win and `0` indicates a loss. This makes the problem a **binary classification** problem.

I frame the prediction task as making a prediction **around 15 minutes into the match**. Because of this, I only use features that would reasonably be known by that point, such as:

- `golddiffat15`
- `xpdiffat15`
- `csdiffat15`
- `firstdragon`
- `firstherald`
- `firsttower`
- `side`
- `league`
- `patch`

I intentionally avoid full-game features like total Barons, total dragons, or `total_objectives`, since those would leak information about the final outcome.

I use **accuracy** as the evaluation metric because the dataset is balanced: each game contributes one winning team and one losing team, so the classes are approximately equal in size. Accuracy is therefore interpretable and appropriate.

---

## Baseline Model

My baseline model is a **logistic regression classifier** implemented in a single sklearn Pipeline.

The model uses the following features:

- `golddiffat15` — quantitative
- `xpdiffat15` — quantitative
- `side` — nominal
- `league` — nominal

For preprocessing:

- quantitative features were median-imputed and standardized
- nominal features were imputed with the most frequent value and one-hot encoded

This baseline model achieved:

- **Training accuracy:** 0.732
- **Test accuracy:** 0.735

This is a reasonable starting point. It performs better than random guessing and suggests that early-game gold and experience advantages already contain useful predictive information. However, it likely misses more detailed patterns involving objective control and feature interactions.

---

## Final Model

To improve on the baseline model, I built a **Random Forest classifier** that uses additional early-game features and engineered variables.

In addition to the baseline features, I added:

- `csdiffat15`
- `patch`
- `firstdragon`
- `firstherald`
- `firsttower`

I also engineered two new features:

1. **`objective_score15`**  
   Defined as:
   \[
   firstdragon + firstherald + firsttower
   \]
   This summarizes how many key early objectives a team secured.

2. **`econ_advantage15`**  
   Defined as:
   \[
   golddiffat15 + xpdiffat15
   \]
   This summarizes a team’s overall early-game economic lead.

These features were chosen based on the data generating process. In League of Legends, teams that gain early gold and experience advantages often convert those advantages into objective control, and both together help describe the state of the game around 15 minutes.

I used **GridSearchCV** with 5-fold cross-validation to tune the following hyperparameters:

- `n_estimators`
- `max_depth`
- `min_samples_leaf`

The best-performing hyperparameters were:

- `n_estimators = 200`
- `max_depth = 5`
- `min_samples_leaf = 1`

The final model achieved:

- **Training accuracy:** 0.755
- **Test accuracy:** 0.751

Compared to the baseline test accuracy of 0.735, this is an improvement of about **1.6 percentage points**. The improvement is modest but meaningful, and it suggests that early objective control and nonlinear feature interactions help improve match-outcome prediction.

---

## Fairness Analysis

Finally, I evaluated whether the final model performs differently for teams on Blue side versus Red side.

I defined the groups as:

- **Group X:** Blue side teams
- **Group Y:** Red side teams

I used **accuracy** as the fairness metric and computed the following test statistic:

\[
accuracy_{Blue} - accuracy_{Red}
\]

The observed difference in accuracy was **0.0026**, which is very small.

I then performed a permutation test by randomly shuffling the side labels 10,000 times and recomputing the accuracy gap each time.

- **Null hypothesis:** The model is fair with respect to side; any difference in accuracy is due to random chance.
- **Alternative hypothesis:** The model performs differently for Blue and Red side teams.

The resulting p-value was **0.431**.

Since this p-value is much larger than 0.05, I fail to reject the null hypothesis. This suggests that there is no statistically significant evidence that the model performs differently for Blue-side and Red-side teams.

In other words, the final model appears reasonably fair with respect to side.

---