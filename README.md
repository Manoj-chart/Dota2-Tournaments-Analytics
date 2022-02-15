# Dota2-Tournaments-Analytics
Abstract:-
From 2011, there are lot of professional tournaments organized for Dota 2 game across the world from which some insights can be extracted which gives some significant information about the tournaments, teams and prize pool amount. This report describes about the analytics been done on the Dota 2 tournaments dataset using bigdata architecture and tools. The main objectives of this project are to provide insights about the tournaments, teams with most tournament wins, the teams which are favorites for winning the highest prize pool tournament “The International” which is being held once every year and the prediction of the prize pool of the future tournament. In this project, dominantly spark with the help of pandas, seaborn and spark MLlib were used to do the descriptive analytics and predictive analytics. For this, 5 machine learning models were developed and trained using Ridge regression, Lasso regression, ElasticNet regression, Gradient boosted tree regression and Isotonic regression. The results achieved by the models were compared subsequently, the best model was chosen and used to predict the future prize pool.

Index Term:-
Spark, Big Data Analytics, Isotonic Regression, Dota2 tournament

Introduction:- 
Bigdata analytics is a process of examining vast amount of data to discover some information such as hidden patterns, data correlations,  trends, etc. this often involves using applications like predictive models and tools to extract new information which may be useful for an organization in making some important business decisions. The resulted benefits include more effective marketing strategies and customer perspective operational efficiency. Nowadays bigdata applications are used to provide analytics for many electronic sports such as dota2, Counter strike, etc.
Defense of the Ancients (DOTA) 2 is a multiplayer online battle arena (MOBA) game and also a role-playing game (RPG) developed and published by Valve. This game is played between 2 teams of 5 players who have to control a character called Hero and defend their base against the opponents. The prize pools and the number of players playing Dota 2 makes it the most lucrative esports game ever. Based on the prize pool amount and the participants the tournaments are classified as Tier 1, Tier 2, Tier 3, etc but the analysis for this project will be done from Tier 1 to Tier 3 tournaments. Since there are lot of tournaments held from 2011, large amount of data regarding the tournaments, teams and players are available in the internet which can be used to find some interesting insights. These insights may be useful for Dota 2 teams in making decisions regarding their team composition and participations in the tournaments. Using bigdata technologies, this project answers the following questions.
Which is the team with most tournament wins or highest winrate.
To check whether the team which performs well in the leagues before has high or low chances of winning “The International” tournament.
The prediction of the prize pool of the next “The International” tournament which is going to be conducted on September 2022 as the prize pool has been increasing steeply from USD 1,600,000 to USD 40,018,195 over these years. This prediction may give high motivation to teams to qualify for this tournament, compete and win.   
To achieve the above goals, the data analytic pipeline used has the following stages.
1) Data Collection 
2) Data Transformation 
3) Feature Engineering  
4) Algorithm Selection  
5) Training
6) Testing 
7) Descriptive or Predictive Analysis 
8) Data Visualisation

As there are more than 1 lakh tournament match data available, apache spark has been used to process the data in a multicore cluster in databricks. The required dataset has been retrieved from 2 sources, one is https://liquipedia.net/ and the other is Steam web api. These data were then stored in csv files in databricks filestore. These data was again fetched from the csv files and stored in spark dataframes after the necessary cleaning was done in it. The spark dataframes with the combination of the spark sql were used to do the descriptive and predictive analysis. For the visualizations seaborn package was used to create the charts. The dataset was partitioned and written to the filestore in DBFS, so all the actions that has been done on the spark dataframes and spark sql were ran in parallel using the Cluster producing robust results.

Dataset description:-
The data used for analytics has been fetched from liquipedia website using web scrapping and from Steam web api. 
Steam web api:-
For using the steam web api, first an account needs to be created and an api key needs to be generated. This api key needs to supplied to most of the api endpoints. Using the api “http://api.steampowered.com/IDOTA2Match_205790/GetLeagueListing”, all dota2 tournaments held from 2011 were fetched in the form of Json response. Using pandas “json_normalize” this Json was converted to pandas dataframe which in turn transformed to spark dataframe “leagueListingSparkDf” and written as “LeagueListingFull.csv” to the file store. This has the following  properties in it.
name: name of the tournament
leagueid: Identifier of a tournament
description: small description about the tournament
tournament_url: The link to the tournament website
itemdef:  unique number represents a league

For retrieving the matches held in each league, function “CreateMatchHistoryTable” was written which takes “leagueid” as argument to it. Then the web api endpoint http://api.steampowered.com/IDOTA2Match_570/GetMatchHistory/v1/  was called and the json response was iterated in for loop. Each match data was then added to array “allMatchHistoryDfArray” which was subsequently converted to spark dataframe “allMatchHistorySdf” using the schema as shown below.
 series_id: Identifier for the best of three games
series_type: integer represents type of the series
match_id: unique identifier of each match played in the tournament
match_seq_num: represents the sequence of the match played
lobby_type: the game type which is being played
radiant_team_id: one of the 2 teams played in the match
dire_team_id: one of the 2 teams played in the match
leagueid: Unique identifier of the league in which the match been played

This “allMatchHistorySdf” contains all the matches held in the tournaments. The total rows in this dataframe is approximately 1,16,000. To know the results and player’s scores of each of the matches, an another steam web api call “http://api.steampowered.com/IDOTA2Match_570/GetMatchDetails/v1/” should be made with the match_id of “allMatchHistorySdf” as a parameter. A function “CreateMatchDetailsTable” was defined to make the api call and to store all the rows of the Json response in two arrays, one for match details and another for players however, due to large number of rows in “allMatchHistorySdf”, the estimated time for completing this task is approximately 9 hours. The cluster becomes terminated after 2 hours of time, so this task could not be finished. 

Liquipedia:-
Using the urllib.request, requests were made to the liquipedia urls. BeautifullSoup was used to parse the Html DOM Tree of the response. The parsed data was then stored in spark dataframe “otherTierTournamentsDf” with new column “Year” as the last 4 characters of the “Date” column. The schema of this dataframe is
Tournament: Name of the tournament
Date: Dates in which the tournament took place
Prize: prize pool of the tournament
Teams: number of teams participated in the tournament
Location: city where the tournament held
Winner: the team that won the tournament
Runner-up: The team that came second in the tournament.
Tier: The tier or the division the tournament belongs to.

Data Pre-processing:-
In dataframe “allTierTournamentsSdf”, the column “Year” changed from StringType to IntegerType. In the column “Prize”, “$” and “,” were removed. It was then changed to Doubletype. In the column “Teams”, the string “teams” is removed and the data type was changed to IntegerType. Like wise in the dataframe “leagueListingSparkDf”, there were some unwanted charecters such as “#”, “_” and “DOTAItem” which were removed from the column “Name”. For joining the two dataframes “allTierTournamentsSdf” and “leagueListingSparkDf” which were fetched and constructed from 2 different data sources as described in previous section, Jaccard_join from pystringsimjoin was been tried. The 2 dataframes was profiled to find whether those were suitable for the join. The key attribute were added to both the dataframes based on the index. However, due to some reasons, the expected output was not got from the jaccard_join. So the analytics was proceeded without combining these two dataframes.

Exploratory data analysis:-
From the spark dataframe “allTierTournamentsSdf”, the top 20 tournaments with the highest prize pool were filtered and supplied to Seaborn bar chart. From the resulted plot as shown in the figure 2, we can find that the tournament “TheInternational2021” has the highest ever prize pool till now in the history of Dota2 tournaments.
The leagueListingSparkDf combined with the matchHistorySparkDf using spark innerjoin to form dataframe “leagueMatchesSdf”. From the spark dataframe “leagueMatchesSdf”, the top 20 tournaments with the highest number of matches played were filtered and supplied to Seaborn bar chart. As we can find from the plot that the tournament “TheSummitTicketNoContribution” has the highest number of matches hosted as shown in the figure 3.

Descriptive Analytics:-
A sql view “tournaments_plot” has been created based on the dataframe “allTierTournamentsSdf”. With the usage of sql queries the the data from tournaments_plot was grouped together based on tier and winner. The number of wins of the team was determined by using count() function. The same process was followed for all the 3 tiers and those were combined into one spark dataframe “tierTeamWinData”. This was then converted to pandas dataframe and passed to the seaborn FacetGrid function. As shown in the figure 4, the Navi tops the tier 1 division with 23 tournament wins followed by Team Secret with 13 wins. In tier 2 division, team secret leads with 18 tournament wins. In the Tier 3 division team Neon tops with 24 wins. From the results, we can interpret that the team secret has more number of tournament wins from both tier 1 and tier 2 divisions with 31 tournament wins overall. So the question 1 has been answered with Team Secret.

For answering the question 2 of the hypothesis, a view “allTierTournaments” was created. The last 3 years tier 1 tournament data was taken into consideration for this. For the years 2021, 2019, 2018, the winners and runner-up teams of the tournaments held before the “TheInternational” tournament has been got using SqlContext.sql function and groupBy function. 
For year 2021, as shown in the figure 5, the team which won "The International 2021" has never won any tournaments major or minor before in year 2021. Team "Team Spirit" never won any major tournaments before but won the International tournament which has the highest prize pool. This team stunned the strongest teams in the world.

For year 2019, as shown in the figure 6, the team which won "The International 2019" has never won any tournaments major or minor before in year 2019. Team "OG" never won any major tournaments before but won the International tournament which has the highest prize pool in 2019. This team also stunned the strongest teams in the world.

For year 2018, as shown in the figure 7, the team which won "The International 2018" has never won any tournaments major or minor before in year 2018. Team "OG" never won any major tournaments before but won the International tournament which has the highest prize pool in year 2018.

From the results above, the trend of winning the “TheInternational” tournament is the team which was not won any tournaments before the international tournament has higher chances of winning it.


Predictive Analytics:-
The allTierTournamentsSdf was sorted based on the column year and a view “tournaments” was created. Using spark sql, the tier 1 International tournaments that held as of today were filtered and stored in the spark dataframe “tournament_international”. Statistics.corr to find the correlations of the features “Teams” and “Year” with “Prize”. As a result these two features has the highest correlation values of 0.57 and 0.99. So these two features were taken into consideration for the prediction analytics. A bar chart showing the raise of the prize pool of the international tournament which constructed using the “tournament_international”. As shown in the figure 8 we can find that the prize stays almost same for the years 2011, 2012 and 2013. After that it increases steeply with the years finally reaching USD 40,018,195 in 2021. 
A column “log_prize” was added into the dataframe “tournament_international” by taking the log value of the prize column. The two features “Teams” and “Year” were supplied to the VectorAssembler for transforming the features into feature vector. A label column was added to tournament_international by rounding the log_prize values. The data then splitted into training and testing data.

Isotonic Regression:-
It is a process of fitting a free form line to a sequence of observations which are increasing everywhere. It is a monotonic function and is suitable for predicting the sequences of certain detail as it either preserves or reverses the order[3]. IsotonicRegression function of MLlib was used to initialize with the feature and label column. The model was created by fitting it with the training data and it was used to transform the test data to the prediction data which was stored in the predictionData dataframe. As the result the predictons were approximately same as that of the testing data with minimal deviations. 
A function called “getR2AndRmse” was created to calculate the r-squared error and Root Mean Squared Error.

Ridge Regression:-
This is one of the model tuning methods which does L2 regularization to the prediction. A function “getBestModel” was defined which uses CrossValidator from MLlib to test with the different parameter combinations and returns a model which has the best prediction[4]. For applying this to the training data the regularization parameter used are “0.1, 0.01, 0.001” and the elastic net parameter is “0.0” CrossValidator was fitted with the training data and the resulting best model was applied to the test data. The R-squared and Root Mean Squared error were computed and added to a dataframe “model_perf”.

Lasso Regression:-
This preforms L1 regularization that shrinks the data values towards a central point as a mean[5]. The regularization parameter for this are 0.1, 0.01 and the elastic net parameters are 0.2 and 0.5. CrossValidator applies this model to the training data and the best model was chosen. This was then applied to the test data and the R-squared and RMSE were added to the model_perf.

Elasticnet Regression:-
This type of regression combines both the L1 and L2 regularization to the loss function of the Linear regression[6]. The parameters used were 0.1, 0.01 and elasticnet parameters 0.2 and 0.5. CrossValidator applies this model to the training data and the best model was chosen. This was then applied to the test data and the R-squared and RMSE were added to the model_perf.

Gradient Boosted Tree Regression:-
GBTRegressor function of MLlib has been initialized with the features and label columns of the input dataframes. The estimator is fitted with the training data and the transformer transforms the test data to the prediction data. The getR2AndRmse was used to compute the R-squared error and RMSE which was written to model_perf dataframe.

Result:-
As shown in the figure 9, Isotonic regression has the highest R-squared value and low RMSE. So this is the best model to predict the prize pool of the International Tournament. The predictions for the future prize pool of the International tournament also done by constructing a new dataframe which has the International tournament data going to hold on  September 2022. All the 5 models were applied to this new dataframe “tournament2022ModelDf”. As a result, we can find that IsotonicModel predict the prize as same as the previous international tournament held on 2021. Isotonic model was good in predicting the values within the boundary but Ridge regression performs well in predicting the future values.
