# LoL Roles Predictor
Random Forest Classification of LoL Roles

## Introduction and Problem Identification

The goal of this project is to create a model that can use stats from a League of Legends game for any given player to answer the question "What role/position did they play?"

The dataset used to train this model contains professional-level League of Legends matches from the year 2022. Since there are 5 possible positions that a player can be assigned to in a LoL team, my model will be multiclass classifier with possible response values of `'top'`, `'jng'`, `'mid'`, `'bot'`, and `'sup'`. The metric I am using to evaluate my model is **accuracy score** because the distribution of values for my response variable is almost completely balanced since each match has exactly 2 top laners, 2 junglers, 2 mid laners, 2 bottom laners, and 2 supports. This means that I don't have to deal with one of the major drawbacks of the accuracy metric, which is that some models get deceptively high accuracy scores when the dataset has far more data points in one class and far fewer in the other classes.

Since the goal of this model is to predict what role someone played based on various performance stats in the game, I will assume the time at which I am making the prediction is after the game has been going on for a while or after the game has been completed, allowing me to use data from during each match, such as kills, deaths, assists, gold earned, or monster kills.

## Baseline Model

The baseline model consists of a Decision Tree Classifier with a max depth of 3 that uses 3 features coming from the columns `'result'`, `'kdratio'`, and `'monsterkills'` in the cleaned DataFrame.

**Nominal Features:**
- Result (Win/Loss) was one-hot encoded (1/0)

**Quantitative Features:**
- K/D Ratio was standardized; (value - mean) / std. dev.
- Monster Kills; missing values were imputed using mean (only ~8 missing values)

**Accuracy Score: 0.59**; I don't believe this baseline model is very good because its accuracy could easily be improved by engineering and making use of some more features. For example, the Result feature is not utilized well because every match in League of Legends has a player that wins and one that loses for each position, so the Result column alone doesn't say anything about what position someone might be playing. Instead, the result info needs to be combined with some other column to create a better feature.

## Final Model

**Feature Selection:**
- K/D Ratio Standardized by Result - In my previous project at https://github.com/jeffmeredith/LoL-roles-analysis.git, I found that different positions' performances affect the result of the game to varying degrees, as can be seen by the visualization immediately below. Because performance impacts the result of the game to different extents based on position played, it makes sense to engineer a feature by first grouping data by result (Win/Loss) and then standardizing the K/D ratios of each group.
<iframe src="assets/kdbywinner.html" width=800 height=600 frameBorder=0></iframe>
- Gold Earned - I engineered a feature out of the `'earnedgold'` column by transforming the values into percentiles using the QuantileTransformer() from sklearn. I chose this feature because it was clear from the below visualization that different positions have very different gold income distributions, with bot lane generally having the highest and support having the lowest.
<iframe src="assets/gold.html" width=800 height=600 frameBorder=0></iframe>
- Assists, CS Per Minute - Both of these features are engineered by taking their respective columns and standardizing the values, similar to the K/D Ratio, but no grouping is done before standardization. I chose these features because it is clear their distributions vary for different positions from the visualizations below.
<iframe src="assets/assists.html" width=800 height=600 frameBorder=0></iframe>
<iframe src="assets/cspm.html" width=800 height=600 frameBorder=0></iframe>
- Damage Dealt, Monster Kills, Vision Score - All of these features are included in the model because, once again, each of their distributions vary greatly for different positions played according to the below visualizations. Damage dealt is generally highest for bot lane, vision score tends to be highest for support, and monster kills is almost always much higher for the jungle role. Each of these features had their missing values imputed by the mean, which was acceptable in this case because there were so few missing values in each of these columns.
<iframe src="assets/damagedealt.html" width=800 height=600 frameBorder=0></iframe>
<iframe src="assets/monsterkills.html" width=800 height=600 frameBorder=0></iframe>
<iframe src="assets/visionscore.html" width=800 height=600 frameBorder=0></iframe>

**Model and Hyperparameter Selection:**
The model I chose is the **Random Forest Classifier** because it may decrease the variance of my model by training multiple decision trees with smaller subsets of the training set and having the various trees vote on classifications. I specify two hyperparameters for the Random Forest: **max depth** of each tree and **minimum number of samples to split**. I used a GridSearchCV to test all combinations of 10 options for max depth and 10 options for minimum number of samples to split. The optimal hyperparameters were found to be max depth of **22** and minimum samples to split of **30**. The final improved accuracy was **0.71**, a significant improvement over the baseline accuracy of **0.59**.

Below is the final model's confusion matrix:
<iframe src="assets/cmfig2.html" width=800 height=600 frameBorder=0></iframe>

## Fairness Analysis

After completing the final model, I assessed whether the model was fair in its accuracy to both players who won and players who lost.

**Groups Chosen**
- Group 1 - Players who won the game
- Group 2 - Players who lost the game

**Null Hypothesis**
This model is fair. The accuracy score for the final model on players who won and players who lost are roughly the same and differences are due to random chance.

**Alternative Hypothesis**
This model is unfair. The accuracy score for the final model on players who won is higher than its score for players who lost in a significant way.

**Test Statistic**
Difference in accuracy score between winners and losers

**Significance Level:** 0.05

**P-Value:** 0.0

**Conclusion**
We have evidence to reject the null hypothesis because the p-value is lower than our significance level. There is evidence supporting the idea that this final model has a higher accuracy for predicting the positions of players who won the game than for players who lost.

Below is the distribution of test statistics simulated with permutations under the null hypothesis and how it compares to the actual observed test statistic.
<iframe src="assets/permtest2.html" width=800 height=600 frameBorder=0></iframe>