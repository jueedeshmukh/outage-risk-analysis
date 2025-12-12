# Outage Risk Analysis

# Introduction
Major power outages disrupt critical infrastructure and daily operations. Predicting outage duration helps hospitals, data centers, and businesses respond appropriately: a 2-hour outage requires minimal intervention, while a 48-hour outage demands full evacuation protocols.

This project analyzes 1,314 major power outages in the continental U.S. from 2000-2016 (outages affecting 50,000+ customers or causing 300+ MW demand loss, as defined by the Department of Energy). Access the dataset here: https://engineering.purdue.edu/LASCI/research-data/outages
The dataset contains information related to major outages, including their weather patterns, climate and economic characteristics of their locations, electricity consumption patterns, and land-cover characteristics. 

The analysis is centered around the question: what factors influence the duration of major power outages?
To answer this question, I:
- Clean the data
- Explore patterns through univariate and bivariate analysis of outage distributions acorss time, climate, and geography
- Analyzed missingness in a key variable: demand loss
- Tested hypotheses about relationships between outage characteristics and duration
- Built predictive models to forecast outage duration using climate, temporal, and regional features
- Evaluated the fairness of my model to ensure the model performs equally well for urban and rural states.

The raw dataset includes 1,534 rows and 55 columns. Here are the relevant columns for my analysis:

| Column                     | Description                                                                                           |
|----------------------------|-------------------------------------------------------------------------------------------------------|
| 'YEAR'                     | Year when outage occurred                                                                             |
| 'MONTH'                    | Month when outage occurred (1–12)                                                                     |
| 'U.S._STATE'               | U.S. state where outage occurred                                                                      |
| 'CLIMATE.REGION'          | U.S. climate regions, defined by the National Centers for Environmental Information                    |
| 'ANOMALY.LEVEL'           | Oceanic ONI index measuring El Niño/La Niña conditions                                                |
| 'OUTAGE.START.DATE'       | Date when the outage started                                                                           |
| 'OUTAGE.START.TIME'       | Time of day when the outage started                                                                   |
| 'OUTAGE.RESTORATION.DATE' | Date when power was fully restored                                                                    |
| 'OUTAGE.RESTORATION.TIME' | Time of day when power was fully restored                                                             |
| 'CAUSE.CATEGORY'          | Category of the event that caused the outage                                                          |
| 'OUTAGE.DURATION'         | Duration of the outage in minutes                                                                     |
| 'DEMAND.LOSS.MW'          | Peak demand lost during the outage (in MW)                                                            |
| 'TOTAL.SALES'             | Total electricity consumption in the state                                                            |
| 'UTIL.REALGSP'            | Economic output from the utility industry, adjusted for inflation                                     |
| 'POPULATION'              | State population for that year                                                                         |
| 'POPPCT_URBAN'            | Percentage of the state's population living in urban areas                                             |



# Data Cleaning and Exploratory Data Analysis

## Cleaning the Data
Cleaning the data is necessary for getting rid of errors, inconsistencies, and values that would mislead analysis.
Here are the steps I took throughout my project: 

1. I dropped columns I won't use in analysis: 'NERC.REGION', 'HURRICANE.NAMES', 'HURRICANE.NAMES', 'RES.PRICE', 'COM.PRICE', 'IND.PRICE', 'TOTAL.PRICE', 'RES.PERCEN', 'COM.PERCEN', 'IND.PERCEN', 'RES.CUST.PCT', 'COM.CUST.PCT', 'IND.CUST.PCT', 'PC.REALGSP.STATE', 'PC.REALGSP.USA', 'PC.REALGSP.REL', 'PC.REALGSP.CHANGE', 'TOTAL.REALGSP', 'UTIL.CONTRI', 'PI.UTIL.OFUSA', 'POPPCT_UC', 'POPDEN_URBAN', 'POPDEN_UC', 'POPDEN_RURAL','AREAPCT_URBAN', 'AREAPCT_UC', 'PCT_LAND', 'PCT_WATER_TOT', 'PCT_WATER_INLAND', 'POSTAL.CODE', 'CAUSE.CATEGORY.DETAIL'

2. I combined OUTAGE.START.DATE and OUTAGE.START.TIME columns into one Timestamp object, called OUTAGE.START. OUTAGE.RESTORATION.DATE and OUTAGE.RESTORATION.TIME combine to create a Timestamp object OUTAGE.RESTORATION column. I dropped the original columns. 

3. I created a column START.TIME_OF_DAY to indicate the time of day (Dawn, Morning, Daytime, Evening, Night) the outage began at. These are my boundaries:
- 12:00 AM-6:00 AM: Dawn
- 6:00 AM-10:00 AM: Morning
- 10:00 AM-5:00 PM: Daytime
- 5:00 PM-9:00 PM: Evening
- 9:00 PM-12:00 AM: Night

3. I created the column OUTAGE.DURATION_HOURS, which is just OUTAGE.DURATION / 60. I dropped OUTAGE.DURATION. 

4. I dropped any rows where 'CLIMATE.REGION' was missing. There were less than 10 of such rows; they all had states of Alaska or Hawaii, which isn't on mainland United States.

5. Rows where OUTAGE.DURATION_HOURS are missing account for less than 5% of the data. I dropped these rows. There are three rows in my notebook which include outage durations of over 100 hours. While they may be real events, I couldn't find any external research on a 75 day long outage. It looks like they occurred during winter months, in cold-weather states. What I did was remove all outages that are more than the 95th percentile (186 hours). While I might lose extreme but possibly real events, I will remove potential errors and only 5% of the data. 

6. I also made sure all outages were valid, as defined by the Department of Energy: CUSTOMERS.AFFECTED > 50000 or DEMAND.LOSS.MW > 300 MW. 

7. I filtered the dataset to only incude outage durations > 0 hours, since major outages wouldn't have a duration of 0 hours. 

8. I eventually dropped the columns 'DEMAND.LOSS.MW' and 'CUSTOMERS.AFFECTED', since they had many missing values and won't be relevant for my analysis. 

Here's a preview of my cleaned dataset, with a portion of the columns: 
| U.S._STATE   |   YEAR |   ANOMALY.LEVEL |   OUTAGE.DURATION_HOURS | CLIMATE.REGION     |   TOTAL.SALES |
|:-------------|-------:|----------------:|------------------------:|:-------------------|--------------:|
| Minnesota    |   2011 |            -0.3 |              51         | East North Central |   6.56252e+06 |
| Minnesota    |   2014 |            -0.1 |               0.0166667 | East North Central |   5.28423e+06 |
| Minnesota    |   2010 |            -1.5 |              50         | East North Central |   5.22212e+06 |
| Minnesota    |   2012 |            -0.1 |              42.5       | East North Central |   5.78706e+06 |
| Minnesota    |   2015 |             1.2 |              29         | East North Central |   5.97034e+06 |

## Univariate Analysis

Univariate analysis is the examination of single variables- how are they distributed? What are the different values they take on?

Here is the distribution of Power Outage Duration, in Hours. Many outages are under 2.5 hours, and as the hours increase, the frequency of outages with bigger durations decreases. 

<iframe
  src="assets/univariate_1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Here, I want to see how many power outages each climate region has. It looks like the Northeast has the most recorded power outages. This is due to the number of states that make up the Northeast; it adds up.

<iframe
  src="assets/univariate_2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Bivariate Analysis
Here are the most significant bivariate analyses (comparing two variables). 

This chart shows the mean duration of outages that started at each time of day. Notice that power outages started between 12:00 AM-6:00 AM (dawn) are longer, on average, than any other start time. I later explore whether this is a significant difference. 

<iframe
  src="assets/bivariate_1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

I wanted to explore patterns of electricity consumption. This chart shows average electricity consumption across regions. Notice how the South and West have the highest average consumption. This may be due to California and Texas being large states. 

<iframe
  src="assets/bivariate_2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Grouping and Aggregates

This table shows the average outage duration, in hours, across different cause categories and times of day. 
Key findings:
- Fuel supply emergencies have the longest average durations across most times of day, particularly in the dawn/morning hours. 
- Equipment failures show extreme variation by time. Dawn outages average 141 hours while evening outages average only 2 hours. 
- Severe weather outages are consistently long regardless of time of day, reflecting the extensive damage that the climate may cause. 

This analysis reveals that both the cause and timing of an outage significantly influence restoration time, which informed the features used in our predictive model.

|   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
|           141.173   |                482.725  |              8.97222 |    3.70952  |         78      |          76.396  |                        15.1937  |
|             7.13333 |                264.904  |              3.99215 |    0.593333 |         14.3181 |          72.9585 |                         9.81458 |
|             4.64103 |                154.211  |              7.17593 |    2.98083  |         25.0682 |          61.7188 |                         8.8174  |
|             1.95667 |                 27.9333 |             13.4657  |    4.53519  |         44      |          58.6657 |                         5.44778 |
|            12.9958  |                 14.7083 |             14.513   |    5.9      |         22.2833 |          49.4547 |                        58.4     |



# Assessment of Missingness

## NMAR Analysis
A column I investigated is CAUSE.CATEGORY.DETAIL. In the raw dataset, there were 1059 non-null values out of 1533 values. Two NMAR Mechanisms for this column would be: the detail of the cause of the outage is genuinely unknown, or very unclear. You can't record what you don't know. Also, sensitive causes are deliberately not detailed. The nature of the cause determines whether it gets recorded. For example, an intentional attack might be psychologically traumatic and hard to recall.

I can collect data about the investigation of the power outage cause. For example, I can create an ordinal column that indicates the level at which the outage was investigated (1 being not at all, 5 being extensively). If lower numbers of this investigation level column tend to have more missing CAUSE.CATEGORY.DETAIL values, I am one step closer to determining if the column is MAR.

## Missingness Dependency

I assess the missingness of the column DEMAND.LOSS.MW. This column represents the amount of peak electricity demand lost during an outage event, in Megawatts. Assessing the missingness of demand loss will help inform if certain factors affect reporting, which can affect how long an outage is reported for.

Dependent on YEAR: Through a permutation test with 10,000 iterations, I found that the missingness of DEMAND.LOSS.MW does depend on the year of the outage (p-value = 0.0). This suggests that reporting practices may have changed over time for demand loss. 

<iframe
  src="assets/p_value_1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Not dependent on MONTH: Using the same permutation test approach, I found that the missingness of DEMAND.LOSS.MW does not depend on the month of the outage (p-value = 0.31). The distribution of months is similar whether demand loss data is missing or present, indicating no seasonal pattern in reporting. 

<iframe
  src="assets/p_value_2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Hypothesis Testing
Previously, we see that outages that started at dawn had a higher duration, on average, than outages started at other times of the day. I will be using a permutation test to see whether this difference is significant. This test can inform whether or not the time of day an outage starts will affect its duration.

Relevant columns for this test include: START.TIME_OF_DAY and OUTAGE.DURATION_HOURS. 

Null hypothesis: Dawn outages have the same mean duration as non-dawn outages
Alternative hypothesis: Dawn outages have a longer mean duration than non-dawn outages
Test statistic: Difference in means, dawn_duration-other_duration. 

I first made a new column, is_dawn, which is a binary column indicating whether the start time was 'Dawn' or not. True if Dawn, False if otherwise. 

Here's the mean duration of outages, dawn vs. not dawn:

<iframe
  src="assets/dawn_means.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Here's the observed statistic, the difference in duration for dawn outages and non-dawn outages: **25.332048049900173**

After a permutation test with 1000 iterations, I received a p-value of 0.0. This signifies that outages that **start** at dawn have a longer mean duration than non-dawn outages.

<iframe
  src="assets/hypothesis_test_pval.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


# Framing a Prediction Problem
My model will attempt to predict the duration of a power outage. This is a regression problem because the target variable, outage duration (hours) is a continuous numeric variable.

To evaluate the model performance, I will use the Root Mean Squared Error. In this context, RMSE informs how far off my model's predictions are from true outage durations, in hours. 

In the scenario that an outage occurs and officials are trying to see how long it'll last, the features they'd know include: 
- Month
- Climate Region (East, Northeast, etc)
- Anomaly Level of the region (how warm or cold the Pacific Ocean is, calculated by averaging sea surface temperature anomalies over 3 months)
- Time of day the outage started
- The percentage of the state's total population that is urban
- Total electricity consumption of the region, in megawatt-hours
- Population of the state where the outage occurs
- The GSP contributed by the utility industry of the state for that year

 Throughout building the models, I used these features to help predict the duration of an outage. 


# Baseline Model

My baseline model is a Linear Regression model. Specifically, a multiple linear regression model. 
Features used:
- Start time of day of outage (START.TIME_OF_DAY). This is a nominal variable. I used a OneHotEncoder transformer to feed it into the linear regression model (which only takes numeric inputs).
- Anomaly level (ANOMALY.LEVEL). This is a numeric variable, and I didn't perform any transformations to it. 

To train a model, you first split the dataset into **training** and **testing** datasets. My train/test split was 80/20. 

## Performance
My baseline RMSE is 37.01 hours. This means that this model's predictage outage durations are usually within +/- 37 hours of reality. That's over a day. In addition, the mean duration is around 29 hours, so the predictions are off by more than the average duration itself. 
These initial results suggests that duration is really hard to predict with these features alone. There's high variance in the data. 

# Final Model
For my final model, I made several changes from the baseline:
1. I added new features. Adding new features can help improve model accuracy IF the features are relevant. More informative features = more patterns the model can learn on. Features added include:
- MONTH: Seasonal weather patterns can inform outage durations because of how severe weather affects how long an outage lasts.
- CLIMATE.REGION: The model can capture geographic patterns with respect to outage duration. 
- POPPCT_URBAN: Urban vs. Rural areas can affect repair speed and resource availability, contributing to how long an outage lasts.

2. In addition to features originally in the dataset, I engineered two new features. Feature engineering can give the  model better signals to learn from, and can uncover hidden patterns. Here are the two features I created:

- SALES_PER_CAPITA: This is the energy intensity per person: how much electricity each person in the state uses on average. It's calculated by TOTAL.SALES / POPULATION. A high value indicates energy-dependent population (people may use more AC, heating, etc). A low value indicates a less energy intensive lifestyle. It might help predict duration because a higher per capita use may indicate that there's more critical infrastructure, or a more complex grid that could be better maintained, leading to faster outage repairs (less duration). 

- UTILITY_CAPACITY_PER_CAPITA: This feature indicates a state's economic capacity for repair. It's calculated by UTIL.REALGSP/POPULATION, which is the state's utility's sector output per person. A higher value indicates a stronger utility sector relative to the population, meaning more resources available for repairs. It captures resource availability, which can affect how long there's an outage. 

3. Instead of a linear regression model, I used a RandomForestRegressor. A RandomForestRegressor builds many decision trees (many small if-then rules) and makes each tree slightly different by using random data and random features. Then, it averages all the trees' predictions to get one final, stable prediction. It handles nonlinear data really well and captures interactions **between** the features. Not all features I'm using are linearly associated with outage duration, which made this model a better choice. 

4. RandomForestRegressor has hyperparameters to tune. After performing a GridSearchCV to find the best combination of max_depth and min_samples_split. The best parameters were **None** and **20**, respectively. 

The final model's RMSE is ~33.5 hours. This is a 9.5% reduction in error. For a utility company, 3.5 fewer hours of error per outage translates into better repair-time estimates, which matters when people are without power. Like mentioned before, there's a lot of variance in data and durations take on a wide range of values. 


# Fairness Analysis
To see if my model is fair, I split the testing (unseen) data into two groups: Urban vs. Rural areas. These two areas can differ in their outage durations because of how many resources are available. Seeing if my model predicts each group with similar RMSE helps reveal whether it performs consistently or if it systematically disadvantages one group. 

My evaluation metric will be the RMSE, similar to that of model training. I use permutation tests to calculate the RMSE for urban vs. rural areas, that are randomly shuffled.

**Null Hypothesis**: The model is fair. RMSE for Urban areas and Rural areas is roughly the same. 
**Alternative Hypothesis**: The model is unfair. RMSE differs between urban and rural areas.

To determine if an area is urban vs. rural, I use the column POPPCT_URBAN. The median percentage is 84%. I create a new column in my testing dataset, IS_URBAN_STATE, which is **True** if the percentage is above this median, and **False** otherwise. 

For a permutation test with 1000 trials, I achieved a p-value of 0.177. This means that ~18% of shuffled differences were as large or larger than my observed RMSE difference of 5.14 hours. The difference is likely due to random chance, and my model is fair with respect to urbanization

<iframe
  src="assets/fairness_analysis.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>