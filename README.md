# Identifying Key Health Risk Factors with SQL

## Business Task
After successfully [Designing a Scalable Database for Greenspot Grocer Using MySQL](https://github.com/jef-fortunahamid/GreenspotGrocerDBDesign) in my last project, I'm tackling a more intricate challenge: analyzing health data to identify key risk factors across different age groups for Synapse Health Analytics. The aim is to help healthcare providers pinpoint and implement preventative measures more effectively. The current dataset is comprehensive but requires a thorough analysis to yield actionable insights. My goal is to use advanced SQL queries to uncover which lifestyle factors are most strongly associated with certain health conditions like diabetes, heart disease, and depression.

During this project, I'll be focused on answering the following questions:

- What is the prevalence of different health conditions across various age groups?
- Which lifestyle factors (e.g., exercise, smoking, alcohol consumption, diet) are most correlated with these health conditions?
- How can we segment the population into different risk categories based on these lifestyle factors?
- What actionable recommendations can we provide to healthcare providers for targeted preventative measures?

## Techniques

To achieve these objectives, I'll employ a range of SQL techniques, including but not limited to:
- `UNION ALL`
- `CTE` *(Common Table Expression)*
- `PERCENTILE_CONT`
- `SELECT DISTINCT`
- `MIN`, `MAX`, `AVG`, `STDDEV`
- `ROUND`
- `CAST`
- `CASE`

## Data
*Source of the data used is from [here](https://www.kaggle.com/datasets/alphiree/cardiovascular-diseases-risk-prediction-dataset), a dataset from Kaggle. Uploaded the `.csv` file into PostgreSQL.*

Checking the dataset obtained using the following query:
```sql
SELECT *
FROM cardiovascular_health
LIMIT 5;
```
![image](https://github.com/jef-fortunahamid/HealthcareRiskAnalysis/assets/125134025/7d26426c-adb2-46f2-b20c-e1f852f4ca30)
![image](https://github.com/jef-fortunahamid/HealthcareRiskAnalysis/assets/125134025/5984cf67-3d69-4061-9433-c7fc4b3a447e)

The columns in the dataset are:
1. general_health: Overall health condition (e.g., Poor, Very Good)
2. checkup: Last medical check-up (e.g., Within the past year)
3. exercise: Whether the individual exercises or not
4. heart_disease: Presence of heart disease
5. skin_cancer: Presence of skin cancer
6. other_cancer: Presence of other types of cancer
7. depression: Presence of depression
8. diabetes: Presence of diabetes
9. arthritis: Presence of arthritis
10. sex: Gender of the individual
11. age_category: Age category (e.g., 70-74, 60-64)
12. height_cm: Height in centimeters
13. weight_kg: Weight in kilograms
14. bmi: Body Mass Index
15. smoking_history: Whether the individual has a history of smoking
16. alcohol_consumption: Amount of alcohol consumed
17. fruit_consumption: Amount of fruits consumed
18. green_vegetables_consumption: Amount of green vegetables consumed
19. fried_potato_consumption: Amount of fried potatoes consumed


Now we proceed with our data exploration. Identifying and checking for missing or incoistent data. The following queries were used to check for inconsistency and missing values, if there are any. With the queries, our data is not showing of any inconsistency and doesn't have any missing values.
```sql
-- Cheking for concistency
SELECT DISTINCT general_health
FROM cardiovascular_health;

SELECT DISTINCT checkup
FROM cardiovascular_health;

SELECT DISTINCT exercise
FROM cardiovascular_health;

SELECT DISTINCT heart_disease
FROM cardiovascular_health;

SELECT DISTINCT skin_cancer
FROM cardiovascular_health;

SELECT DISTINCT other_cancer
FROM cardiovascular_health;

SELECT DISTINCT depression
FROM cardiovascular_health;

SELECT DISTINCT diabetes
FROM cardiovascular_health;

SELECT DISTINCT arthritis
FROM cardiovascular_health;

SELECT DISTINCT sex
FROM cardiovascular_health;

SELECT DISTINCT smoking_history
FROM cardiovascular_health;

SELECT DISTINCT alcohol_consumption
FROM cardiovascular_health;
```
With the queries, our data is not showing of any inconsistency and doesn't have any missing values.

We now check for outliers for `height_cm` with the following queries using `PERCENTILE_CONT` function to obtain the top and bottom outliers (if there is any). We get the 99 percentile for our upper end outliers and 1 percentile for our lower end outliers.
```sql
-- Checking for top end outliers
WITH percentile_cte AS (
  SELECT
    PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY height_cm) AS upper_percentile
  FROM
    cardiovascular_health
)
SELECT
	height_cm
FROM
    cardiovascular_health ch
  , percentile_cte pc
WHERE
	height_cm > pc.upper_percentile
ORDER BY
	height_cm DESC
;

-- Checking for bottom end outliers
WITH percentile_cte AS (
  SELECT
    PERCENTILE_CONT(0.01) WITHIN GROUP (ORDER BY height_cm) AS lower_percentile
  FROM
    cardiovascular_health
)
SELECT
	height_cm
FROM
    cardiovascular_health ch
  , percentile_cte pc
WHERE
	height_cm < pc.lower_percentile
ORDER BY
	height_cm ASC;
```
The top end valkues are from 196 to 241 and the bottom end has values ranging from 91 to 149. The lower end is reasonable if there are kids included in our dataset. We'll check the `age_category` column for this.
```sql
-- Checking if there are teenagers and below
SELECT DISTINCT age_category
FROM cardiovascular_health;
```
![image](https://github.com/jef-fortunahamid/HealthcareRiskAnalysis/assets/125134025/7aef2755-0435-422b-8109-f46640286fef)

We can tell kids are not included in the dataset. However, with further research, we will not exclude these values, as this may be extreme, but looking at their BMI and Weight values, they perfectly coincide with their corresponding height.

So let's proceed with our Exploratory Data Analysis (EDA). We will run a summary statistics for categorical and numerical variables. We will use `UNION ALL` to have one query to run on each variable type.
```sql
-- Summary statistics for numerical variables
SELECT
    'height_cm' AS Variable
	, MIN(height_cm) AS minimum
	, MAX(height_cm) AS maximum
	, ROUND(CAST(AVG(height_cm) AS NUMERIC), 2) AS average
	, ROUND(CAST(STDDEV(height_cm) AS NUMERIC), 2) AS std_deviation
FROM
    cardiovascular_health
UNION ALL
SELECT
    'bmi' AS Variable
  , MIN(bmi) AS minimum 
  , MAX(bmi) AS maximum
  , ROUND(CAST(AVG(bmi) AS NUMERIC), 2) AS average 
  , ROUND(CAST(STDDEV(bmi) AS NUMERIC), 2) AS std_deviation
FROM
    cardiovascular_health
UNION ALL
SELECT
    'alcohol_consumption' AS Variable
  , MIN(alcohol_consumption) AS minimum
  , MAX(alcohol_consumption) AS maximum
  , ROUND(CAST(AVG(alcohol_consumption) AS NUMERIC), 2) AS average
  , ROUND(CAST(STDDEV(alcohol_consumption) AS NUMERIC), 2) AS std_deviation
FROM
    cardiovascular_health
UNION ALL
SELECT
    'weight_kg' AS Variable
  , MIN(weight_kg) AS minimum
  , MAX(weight_kg) AS maximum
  , ROUND(CAST(AVG(weight_kg) AS NUMERIC), 2) AS average
  , ROUND(CAST(STDDEV(weight_kg) AS NUMERIC), 2) AS std_deviation
FROM
    cardiovascular_health
UNION ALL
SELECT
    'fruit_consumption' AS Variable
  , MIN(fruit_consumption) AS minimum
  , MAX(fruit_consumption) AS maximum
  , ROUND(CAST(AVG(fruit_consumption) AS NUMERIC), 2) AS average
  , ROUND(CAST(STDDEV(fruit_consumption) AS NUMERIC), 2) AS std_deviation
FROM
    cardiovascular_health
UNION ALL
SELECT
    'green_vegetables_consumption' AS Variable
  , MIN(green_vegetables_consumption) AS minimum
  , MAX(green_vegetables_consumption) AS maximum
  , ROUND(CAST(AVG(green_vegetables_consumption) AS NUMERIC), 2) AS average
  , ROUND(CAST(STDDEV(green_vegetables_consumption) AS NUMERIC), 2) AS std_deviation
FROM
    cardiovascular_health
UNION ALL
SELECT
    'fried_potato_consumption' AS Variable
  , MIN(fried_potato_consumption) AS minimum
  , MAX(fried_potato_consumption) AS maximum
  , ROUND(CAST(AVG(fried_potato_consumption) AS NUMERIC), 2) AS average
  , ROUND(CAST(STDDEV(fried_potato_consumption) AS NUMERIC), 2) AS std_deviation
FROM
    cardiovascular_health
;
```
![image](https://github.com/jef-fortunahamid/HealthcareRiskAnalysis/assets/125134025/46f357b8-a17d-4d77-b07c-32d8a552ab44)

Here's a quick breakdown of the result:
- BMI: Ranges from 12.02 to 99.33, with an average BMI of 28.63 and a standard deviation of 6.52. The average BMI suggests that the majority of the individuals are in the overweight category, according to typical BMI classifications.
- Weight_(kg): The weights range from 24.95 kg to 293.02 kg, with an average weight of 83.59 kg and a standard deviation of 21.34 kg.
- Height_(cm): The heights range from 91 cm to 241 cm, with an average of around 170.62 cm and a standard deviation of 10.66.
- Alcohol_Consumption: Varies from 0 to 30 units, with an average of 5.10 units and a standard deviation of 8.20. This indicates that while some people don't consume alcohol at all, others may consume it more frequently.
- Fruit_Consumption: The consumption ranges from 0 to 120 units, with an average of 29.84 and a standard deviation of 24.88, showing a broad range of fruit consumption habits.
- Green_Vegetables_Consumption: Consumption varies from 0 to 128 units, with an average of 15.11 units and a standard deviation of 14.93, which shows a varied consumption pattern.

```sql
--  Summary statistics for categorical variables
SELECT
    'general_health' AS Variable
  , general_health AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	general_health
UNION ALL
SELECT
    'checkup' AS Variable
  , checkup AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	checkup
UNION ALL
SELECT
    'exercise' AS Variable
  , exercise AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	exercise
UNION ALL
SELECT
    'heart_disease' AS Variable
  , heart_disease AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	heart_disease
UNION ALL
SELECT
    'skin_cancer' AS Variable
  , skin_cancer AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	skin_cancer
UNION ALL
SELECT
    'other_cancer' AS Variable
  , other_cancer AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	other_cancer
UNION ALL
SELECT
    'depression' AS Variable
  , depression AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	depression
UNION ALL
SELECT
    'diabetes' AS Variable
  , diabetes AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	diabetes
UNION ALL
SELECT
    'arthritis' AS Variable
  , arthritis AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	arthritis
UNION ALL
SELECT
    'sex' AS Variable
  , sex AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	sex
UNION ALL
SELECT
    'age_category' AS Variable
  , age_category AS Category
  , COUNT(*) AS Frequency
FROM
    cardiovascular_health
GROUP BY
	age_category
;
```
![image](https://github.com/jef-fortunahamid/HealthcareRiskAnalysis/assets/125134025/509d6951-dcbd-4967-942a-c8893d393318)

Let's summarise some of the key points:
- Sex: The dataset contains information on 160,196 Females and 148,658 Males. This is relatively balanced.
- Arthritis: 101,071 individuals have arthritis, while 207,783 do not.
- Depression: 61,901 individuals have depression, compared to 246,953 who do not.
- Other Cancer: 29,878 individuals have other types of cancer, and 278,976 do not.
- Age Category: The age groups are fairly well-distributed, with the 65-69 age group having the highest number of individuals (33,434).
- General Health: Most people rate their health as either 'Good' or 'Very Good'.
- Exercise: A significant majority (239,381) exercise, while 69,473 do not.
- Heart Disease: A vast majority do not have heart disease (283,883), compared to 24,971 who do.
- Checkup: Most people have had a checkup within the past year (239,371).
- Diabetes: The majority do not have diabetes (259,141), but 40,171 do have it.
- Skin Cancer: 29,994 individuals have skin cancer, compared to 278,860 who do not.

Now we have a good understanding of individual variables, the next step is to explore the relationships between them We'll aim to identify potential risk factors for various health conditions by looking at how they interact with lifestyle variables.
```sql
-- Analyse lifestyle factors vs health conditions
SELECT 
    'diabetes' AS variables
  , diabetes AS category
  , ROUND(CAST(AVG(alcohol_consumption) AS NUMERIC), 2) AS avg_alcohol_consumption
  , ROUND(CAST(AVG(bmi) AS NUMERIC), 2) AS avg_bmi
  , ROUND(CAST(AVG(fruit_consumption) AS NUMERIC), 2) AS avg_fruit_consumption
  , ROUND(CAST(AVG(green_vegetables_consumption) AS NUMERIC), 2) AS avg_green_vegetables_consumption
FROM
    cardiovascular_health
GROUP BY 
    diabetes
UNION ALL
SELECT 
    'heart_disease' AS variables
  , heart_disease AS category
  , ROUND(CAST(AVG(alcohol_consumption) AS NUMERIC), 2) AS avg_alcohol_consumption
  , ROUND(CAST(AVG(bmi) AS NUMERIC), 2) AS avg_bmi
  , ROUND(CAST(AVG(fruit_consumption) AS NUMERIC), 2) AS avg_fruit_consumption
  , ROUND(CAST(AVG(green_vegetables_consumption) AS NUMERIC), 2) AS avg_green_vegetables_consumption
FROM 
    cardiovascular_health
GROUP BY 
    heart_disease
UNION ALL
SELECT
    'skin_cancer' AS variables
  , skin_cancer AS category
  , ROUND(CAST(AVG(alcohol_consumption) AS NUMERIC), 2) AS avg_alcohol_consumption
  , ROUND(CAST(AVG(bmi) AS NUMERIC), 2) AS avg_bmi
  , ROUND(CAST(AVG(fruit_consumption) AS NUMERIC), 2) AS avg_fruit_consumption
  , ROUND(CAST(AVG(green_vegetables_consumption) AS NUMERIC), 2) AS avg_green_vegetables_consumption
FROM 
    cardiovascular_health
GROUP BY 
    skin_cancer
UNION ALL
SELECT
    'other_cancer' AS variables
  , other_cancer AS category
  , ROUND(CAST(AVG(alcohol_consumption) AS NUMERIC), 2) AS avg_alcohol_consumption
  , ROUND(CAST(AVG(bmi) AS NUMERIC), 2) AS avg_bmi
  , ROUND(CAST(AVG(fruit_consumption) AS NUMERIC), 2) AS avg_fruit_consumption
  , ROUND(CAST(AVG(green_vegetables_consumption) AS NUMERIC), 2) AS avg_green_vegetables_consumption
FROM 
    cardiovascular_health
GROUP BY 
    other_cancer
UNION ALL
SELECT
    'depression' AS variables
  , depression AS category
  , ROUND(CAST(AVG(alcohol_consumption) AS NUMERIC), 2) AS avg_alcohol_consumption
  , ROUND(CAST(AVG(bmi) AS NUMERIC), 2) AS avg_bmi
  , ROUND(CAST(AVG(fruit_consumption) AS NUMERIC), 2) AS avg_fruit_consumption
  , ROUND(CAST(AVG(green_vegetables_consumption) AS NUMERIC), 2) AS avg_green_vegetables_consumption
FROM 
    cardiovascular_health
GROUP BY 
    depression
;
```

















