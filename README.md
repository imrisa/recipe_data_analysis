# recipe_data_analysis
This is a project for DSC80 at UCSD



## Introduction

This project focuses on analyzing the relationship between recipe preparation time, measured in minutes, and user ratings to uncover trends in recipe popularity. Additionally, it involves developing a predictive model to estimate average recipe ratings using key nutritional factors such as calories, fat, and sugar. By integrating exploratory data analysis with predictive modeling, this project provides valuable insights into how preparation time and nutritional profiles impact user satisfaction and preferences.

### Using recipe data
**1) Import each csv file and take a look at each content for analysis**
- import raw interaction data (rows: 731927, columns: 5)
 
| Column | Description |
| ----------- | ----------- |
| user_id(int)| user id |
| recipe_id(int) | recipe id |
| date(time) | date that the recipe is reviewed (yyyy-mm-dd) |
| rating(float) | rate of the recipe 0-5 |
| review(str) | review about the recipe |


- import raw recipes data (rows: 83782, columns: 12)
 
| Column | Description |
| ----------- | ----------- |
| name(str)| recipe name |
| id(int) | recipe id |
| minutes(int) | time to cook the recipe |
| contributor_id(int) | contributor's id |
| submitted(date) | date the recipe was submitted |
| tags(str list) | Food.com tags for recipe |
| nutrition(str) | list contains each nutrition's amount |
| n_steps(int) | number of steps in recipe |
| steps(str list) | steps to take in the recipe |
| description(str) | description about the recipe |
| ingredients(str list) | ingredients for the recipe |
| n_ingredients(int) | the amount of ingredients |
 
####  Q: How rating is going to change based on the features? 📈
- The columns I need:
    - rating - this is the main
    - minumtes, n_ingredients, steps, description, review, ingredients


## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
- clean data set:
    - merge both data sets with 'recipe_id' and 'id'
    - replace all nan value with 0 in rating column
    - calculate average rating as a Series and add it to the data set
    - put nutrition column's list element into each separated columns
    - drop unnecessary columns
    - deine the range of nutrition so they have right amount  for the recipe
 
      ### After cleaning
      
| Column | Description |
| ----------- | ----------- |
| minutes(int) | time to cook the recipe |
| n_steps(int) | number of steps in recipe |
| description(str) | description about the recipe |
| ingredients(str list) | ingredients for the recipe |
| n_ingredients(int) | the amount of ingredients |
| rating(float) | rate of the recipe 0-5 |
| review(str) | review about the recipe |
| avg_rating(float) | average rating after merging the data |
| calories(float) | amount of calories for the recipe |
| total_fat(float) | amount of total fat for the recipe |
| sugar(float) | amount of sugar for the recipe |
| protein(float)| amount of protein for the recipe |
| carbohydrates(float)| amount of carbohydrates for the recipe |


### Univariate Analysis
For this analysis, shows the two kind of plots to see the distributions for the ingredients and the rates.
- Using plotly to show the distribution
 - Plot1: Histgram for the number of ingredients
     - It shows the frequency of the number of ingredients in the data set. 7-8 ingredients are the most.
 - Plot2: Histgram for the rating of recipes
     - It shows the frequency of the rating. Rate 5 is the most in the data set.
  
<iframe
  src="plot1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
<iframe
  src="plot2.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

### Bivariate Analysis
- Scatter Plot: Rating vs Minutes shows how rating and minits to cook are related each other. Rate 4 and 5 looks like takes more time than others.

<iframe
  src="plot3.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates
 - Aggragate data
     - groupby the rate and shows average length of description and mitures to take for each rate
       - About description length, there is no specific pattern. Almost every rate has the similar length
       - About average minute, the longest minute has 1 rate and the 2nd longest minutes has 5 rate. For other rate, the average minutes are similar.
 - Pivot table
     - index is the minutes_bin and columns are the rate 0-5, and shows each average description length
       - Almost all minutes bins are the similar length of description, but 60-120min tends to have a little longer descriptions.

#### Grouped Table:

|   rating |   avg_description_length |   avg_minutes |
|---------:|-------------------------:|--------------:|
|        1 |                  265.128 |       99.6725 |
|        2 |                  260.945 |       98.0215 |
|        3 |                  237.555 |       87.4976 |
|        4 |                  230.7   |       91.585  |
|        5 |                  248.383 |      106.924  |

#### Pivot Table (columns are the rating and the values are the minute binned):

|     1.0 |     2.0 |     3.0 |     4.0 |     5.0 |
|--------:|--------:|--------:|--------:|--------:|
| 229.889 | 228.259 | 219.929 | 209.638 | 215.936 |
| 252.283 | 227.24  | 219.352 | 218.439 | 232.674 |
| 263.096 | 259.485 | 238.278 | 236.956 | 253.068 |
| 298.739 | 311.365 | 253.216 | 253.18  | 277.199 |

## Assessment of Missingness
### NMAR Analysis
- column [review] - not all people review for the recipe, regardless of how good it is (there are 57 missing)
- column [minutes_binned] - if the recipe takes more than 120min, it will be Nan (there are 24294 mmissing)

### Missingness Dependency
Because of the amount of missingness, for [minutes binned], choose two columns for each to see the dependency if the reason of missingness is NMAR or not.
- column depends on other variable: minutes (p-value is 0.0)
    - reason: If the recipe takes long hours over the set bins, it will be missing values
- column doesn't depend on other variable: rating (p-value is 1.0)
    - reason: Rating and time to take for preparation has no correlation. Rating might be strongly related to other factor.
 
### Check the dependency if the missingness column will be affected by other values
- For minute column (if minutes_binned DOES depend on minute)
  - make a new column ['missing_minutes_binned'] and check the dependency
Calcurate the observed test statistic and the difference in mean is 675. The test statistic (difference in means) represents the difference in numerical measure between two groups. One is in the column has missing value and another in the column doesn't have missing values.

#### For minute column 
(if missing_minutes_binned depends on minute column)
#### Hypothesis 
- Null Hypothesis (H₀): The missingness of the column is independent of the variable used to calculate the difference in means. The observed difference (675) is due to random chance.
- Alternative Hypothesis (H₁): The missingness of the column is dependent on the variable, meaning the difference in means (675) is significant and not due to chance.

### Permutation test
This part performs a permutation test to evaluate whether there is a dependency between the column missing_minutes_binned (a categorical variable) and minutes (a numerical variable) in the dataset merged_data.
1. Running 1000 permutations
2. Shuffling the missing_minutes_binned column
3. Computing the test statistic for the shuffled data - This breaks any real relationship between missing_minutes_binned and minutes, simulating the null hypothesis.
4. Computing the test statistic for the shuffled data
   - If missing_minutes_binned has no relationship with minutes (null hypothesis), we expect the test statistic (difference in group means) to be close to zero.
   - If there is a relationship between missing_minutes_binned and minutes (alternative hypothesis), we expect the test statistic to deviate significantly from zero.

<iframe
  src="plot4.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Result with p-value: 
- p-value 0.0 Because of the p-value < 0.05, reject H0 and missing_minute_binned IS depending on minute


#### For rating column
(if missing_minutes_binned depends on rating column)
#### Hypothesis 
- For rating column (if missing_minutes_binned depends on rating column)
 - Null Hypothesis (H₀): The missingness of the column is independent of the variable used to calculate the difference in means. The observed difference (-0.057) is due to random chance.
 - Alternative Hypothesis (H₁): The missingness of the column is dependent on the variable, meaning the difference in means (-0.057) is significant and not due to chance

### Permutation test
This part performs same as the previous permutation test to evaluate whether there is a dependency between the column missing_minutes_binned (a categorical variable) and rating (a numerical variable) in the dataset merged_data.
1. Running 1000 permutations
2. Shuffling the missing_minutes_binned column
3. Computing the test statistic for the shuffled data - This breaks any real relationship between missing_minutes_binned and rating, simulating the null hypothesis.
4. Computing the test statistic for the shuffled data
   - If missing_minutes_binned has no relationship with rating (null hypothesis), we expect the test statistic (difference in group means) to be close to zero.
   - If there is a relationship between missing_minutes_binned and rating (alternative hypothesis), we expect the test statistic to deviate significantly from zero.
<iframe
  src="plot5.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
#### Result with p-value: 
- p-value 1.0 Because of the p-value > 0.05, fail to reject H0 and missing_minute_binned IS NOT depending on rating


## Hypothesis Testing
### Q: Does the average rating differ significantly between recipes with short cooking times and long cooking times?
- H0: The average rating for both short and long preparation time for the recipes is the same. Any observed differences in average ratings are due to random variation.
- H1: The average rating for short and long preparation time for the recipe is different.　Any observed differences in average ratings are not due to random variation.

- Test statistic: Difference in rating means between the two groups (short time cooking and long time cooking)
- Significance level: 0.05 (5%)

#### Justification:
1. It directly addresses the null hypothesis by generating a null distribution through label shuffling.
2. It avoids the need for assumptions about data normality or variance homogeneity.
3. It uses the actual observed data to create a distribution, making it robust and reliable for this specific comparison.

#### Result:
- Observed Difference: 0.03412279818560471
- P-Value: 0.0
- Because pvalue < 0.05 - regect H0. It means that the average rating for short and long preparation time for the recipe is different. Any observed differences in average ratings are not due to random variation.

<iframe
  src="plot6.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Framing a Prediction Problem
#### Q: Predict average ratings of recipes based on nutritions (calories, total fat, sugar)
Which columns are the best for the features?
- Prediction type: regression (Linear Regression model)
- y (prediction value): average rating
- X (features):
    - minutes - time to cook 
    - n_steps - the number of steps to cook
    - n_ingredients - the number of ingredients
    - nutritions - any specific nutrition (calories, sugar, sodium...etc)
-> choose nutrition, specifically total fat, and sugar.

 
#### Justification
- About the time
    - All features need to be known before the rate is stated by the user.
    - User interaction data should be avoided for the feature such as review because this is stated after recipe is published.


### Baseline Model
#### Model Description
- Type:
    - Linear Regression model
- Pipleline:
  - Scaler: StandardScaler was used to normalize the quantitative features, ensuring they are on the same scale
  - Regressor: LinearRegression to predict avg_rating
- Peformance:
    - RMSE: to see the accuracy of the prediction line with the error size

#### Features
- Features (All Quantitative because of numerical): total fat and sugar
- Models: Linear Regression
    - 20% for the test for unseen data, 80% for the training
    - Test data set: To see the unbiased estimate of how well the model gereralizes to new data
    - Training data set: To learn the relationship between the features and the target prediction value
 
#### Making a prediction and evaluate
 - Root Mean Squared Error (RMSE): 0.4968026952854217
  - The square root of MSE, indicating the model's average error in the same units as the target variable (avg_rating)
    if RMSE is 0.2 - 0.5, it shows accurately predict the data, however my scatter plot doesn't look like each dot spreads away to predict the rate accurately. This is because this recipe data has more rate 5 than rate 1 and it led to have the nutrition more on the higher rate and the linear line is holizontal.

<iframe
  src="scatterplot_total_fat.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="scatterplot_fat_sugar_regression.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>



## Final Model
#### Immprovement
- Having different models for the pipeline:
    - StandardScaler: Standardizes numerical features by removing the mean and scaling to unit variance
        - Ensures all features are on the same scale
    - PolynomialFeatures: Generates additional features by computing interactions and powers of existing features (calories * sugar)
        - Capture the non-linear relation and it enhances the model flexibility and the prediciton accuracy.
    - QuantileTransformer: Transforms the data into a normal or uniform distribution.
        - Helps models handle skewed data better by normalizing feature distributions.
    - RandomForestRegressor: Splitting data recursively and building a series of decision trees.
        - Working with a dataset containing complex relationships between features and the target variable. 

- Features:
  - Adding two new features (protein, carbohydrates)
   - The reason why this feature is good: nutritions highly affect the recipe's taste and how people feel. Also it is depending on the individual preference, such as healthy conscious or not. Protein and carbohydrates are more prominent.
   - Nutritions are highly related to the quality of the recipe and the rate
   - Protein and carbohydrates are especially related to individual preferences and it affects the rate

   
- Type:
    - RandomForestRegressor: To handle non-linear relation to have more accuracy.

- RMSE is a little bit changed from the previous 0.49 to 0.39, but does not necessarily guarantee that the prediction is "more accurate" because the model might be overfitting.
  Although, 
    - standardizes variance makes the prediction easier because all features are closer
    - captured the non-linear relation is crucial because linear regression is robust to the outliner

 By observing each scatter plot, the patterns are like the same as the previous, such as most of data gather around the same rate, so it is hard to tell if the prediction is accurate or not. 
For example, carbohydrates vs Actual average rating is below and the rest of the features have similar patterns.
 <iframe
  src="scatterplot_carbohydrates_regression.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
 

## Fairness Analysis
#### Check the fairness about each nutritions(high and low value) in the final model if the samples are fair to get the result of relation between the predict value. 

For fairness analysis, I split each features into two groups to see the RMSE. A difference in RMSE across groups suggests that the model performs better for one group than the other. (However, it is sensetive to outliers). Also perfortm permutation to asses whether the observed difference in RMSE between the groups is statistically significant under the null hypothesis that the two groups have the same performance distribution. 

#### Hypothesis
- H0: Our model is fair. Its precision for two groups (high and low), and any differences are due to random chance.
- H1: Our model is unfair. Its precision for two groups (high and low) and any differences are not random chance.

#### Result
  - fat:
   - Observed RMSE Difference: 0.05196006747246118
   - P-value: 0.0
   - Reject the null hypothesis: The model's RMSE differs significantly between the two groups -> samples are unfair between the two groups
     
  - sugar: samples are unfair between the two groups. Prediction result might be affected depending on the sample.
   - Observed RMSE Difference: 0.008516514407459153
   - P-value: 0.52
   - Fail to reject the null hypothesis: No significant difference in RMSE between the two groups. -> samples are fair between the two groups
     
  - protain:
   - Observed RMSE Difference: 0.049162329606651456
   - P-value: 0.0
   - Reject the null hypothesis: The model's RMSE differs significantly between the two groups. -> samples are unfair between the two groups
     
  - carbohydrates:
   - Observed RMSE Difference: 0.023228975210453806
   - P-value: 0.08
   - Fail to reject the null hypothesis: No significant difference in RMSE between the two groups. -> samples are unfair between the two groups
 


So three of four feartures are unfair. This observed unfairness in my model's performance between groups could stem from several potential causes, including dataset issues. Another potential issue is model issue, such as overfitting.The model may overfit to the majority group at the expense of the minority group.


Still working on .....
