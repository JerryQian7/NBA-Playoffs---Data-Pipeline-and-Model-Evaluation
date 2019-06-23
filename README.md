# NBA-Playoffs---Data-Pipeline-and-Model-Evaluation

# Summary of Findings


## Introduction
---
In this project, I want to predict whether a player will make the playoffs in the 2018 season. This will be a classification problem as there will only be two outcomes to this prediction, either the player makes the playoffs or he does not. I will be using the regular season dataset from 2012-2017 as my primary data, but my target variable will be the players in the 2018 playoff statistics dataset while keeping track of who actually made the playoffs. I  added a column to the dataframe indicating whether the player made the playoffs or not, called 'outcome'.

My objective will be an accuracy score, since there is not a large class imbalance between the two outcomes. Both outcomes are in fact very close to having equal probabilities, due to there being 16 out of 30 teams that make the playoffs each season. This accuracy score will be based on the number of true positives + the number of true negatives divided by the number of positives + number of negatives within every player of 2018. Or in other words, the number of players we predict will make playoffs **over** the total number of players for that season.




### Results: Baseline Model
---

In my baseline model, I first cleaned the data and removed the duplicates of any player who switched teams during the season but kept the last entry to show that they stayed on that final team. Then I used a pipeline with a column transformer and a decision tree classifier. Since there were no ordinal columns, the dataset consisted of only nominal and categorical columns. In the column transformer, I standardized all the nominal columns (26 of them) and transformed the 2 categorical columns (team & position) to one-hot encoded columns.


By using a train-test split on the dataset with a test size of 1/7, I was able to calculate an accuracy score of 0.658. I used a test size of 1/7 because there were 7 seasons worth of data from 2012-2018, but I only wanted to predict on 1 of those 7 years (2018). 

I believe that my model performed fairly well, considering it only used the given features. However, it did receive a a large amount of features to work with, 26+2=28 in total. I want to improve my score through engineering new features and using different classifiers.

### Results: Final Model
---

In my final model, I researched other key basketball statistics and engineered two very powerful features.

The first one is **True Shooting Percentage**, with the formula given below.

**TS**% - True Shooting Percentage; the formula is PTS / (2 * TSA). 

**TSA** - True Shooting Attempts; the formula is FGA + 0.44 * FTA.

True shooting percentage is a measure of shooting efficiency that takes into account field goals, 3-point field goals, and free throws. This feature is important because it shows the true efficiecy of a player on offense. It allows the coach to view that player's current offensive rating, and can also provide insight on how well that player performs in general, which is a big indicator of whether the player will make it into playoffs. 

The second feature I engineered is the **Usage Percentage**.

**Usg%** - Usage Percentage; the formula is 100 * ((FGA + 0.44 * FTA + TOV) * (Tm MP / 5)) / (MP * (Tm FGA + 0.44 * Tm FTA + Tm TOV)). 

Within this formula, there are variable such as 'Tm MP' that indicate that variable is aggregated by team. For example, 'Tm FGA' stands for the field goals attempted by the entire team for that season. I grouped my data by team and year, and transformed a new column that aligned with each player's team and season to indicate his team's minutes played, field goals attempted, free throws attempted, and turnovers. Hence the new columns named ['TM_TOV', 'TM_MP', 'TM_FGA', 'TM_FTA'].

Usage percentage is an estimate of the percentage of team plays used by a player while he was on the floor. This is useful because it shows how often the team uses a certain player in their plays. Likely, the more often a player is used in plays, the more critical they are to the team. This is another huge indicator of whether the player will make the playoffs or not, since his usage percentage is related to how reliable and how useful that player is.

With my new features incorporated into the pipeline, I was able to get a higher accuracy score of 0.681.

Now, I used the grid search cross validation method to search over specified parameter values for the best set of parameters for an estimator. I also incorporated the **random forest classifier** and the **k nearest neighbors** classifier into my grid search. These are the results of using grid search on the 3 classifiers.

**Decision Tree Classifier**: {'classifier__max_depth': 18, 'classifier__min_samples_leaf': 1,
 'classifier__min_samples_split': 2} **Best score: 0.689**

**Random Forest Classifier**: {'classifier__max_depth': 18,
 'classifier__min_samples_leaf': 2,
 'classifier__min_samples_split': 8} **Best score: 0.708**

**K Nearest Neighbors Classifier**: {'classifier__leaf_size': 1, 'classifier__n_neighbors': 7} **Best score: 0.656**

To conclude my final model, I was able to improve the accuracy score from my baseline model that used decision tree classfier and two new engineered features. But after using grid search on two new classifiers, I found that the random forest classifier gave me the highest score of 0.708.

### Results: Fairness Evaluation
---

In my fairness evaluation, I assessed whether my model would be consistent for veterans entering the playoffs. I used the age 32 as a cutoff for being a veteran in the NBA.

After creating my tables of predictions and outcomes, I ran permutation tests on the entire dataset using the accuracy and true positive parity scores. I set my significance value at 0.05.

For the accuracy parity testing, my **test statistic** is the accuracy score and how well it predicts for groups of players that are considered veterans and their entry into the playoffs.

For the accuracy parity testing, my **test statistic** is the recall score and how well it predicts for groups of players that are considered veterans and their entry into the playoffs.

**Null Hypothesis**: Veterans and non veterans can enter the playoffs equally likely.

**Alternative Hypothesis**: Veterans are more likely to enter the playoffs.

For the accuracy parity score testing, I received a p-value of **0.45**.

For the True Positive parity score testing, I received a p-value of **0.37**.

In conclusion, the p-values I got were too high for my significance value and we conclude that we are unable to reject the null hypothesis. Even though the true positive parity score was higher, it was still not significant enough to provide any correlation.
