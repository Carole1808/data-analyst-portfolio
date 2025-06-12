# Bellabeat Case Study 2

**Project Name:** Bellabeat Smart Wellness Optimization  
**Role:** Junior Data Analyst  
**Author:** Itsevy Carole Dominguez  
**Date:** June 2025  

## Executive Summary

This report analyzes Fitbit smart device data to uncover user behavior trends and provides actionable insights to inform Bellabeat's marketing strategy. Using SQL for analysis and Tableau for visualization, the study reveals patterns in physical activity, calorie burn, sedentary behavior, and engagement. Recommendations are aligned with Bellabeat’s product offerings and target user segments.

---

## 1. Business Task

Analyze smart device usage data (from Fitbit) to understand user behavior patterns in physical activity, intensity, calories burned, and engagement. These insights will guide marketing strategies for Bellabeat products and identify how users could benefit from Bellabeat's wellness-focused features.

---

## 2. Data Preparation

- Dataset: Publicly available from Kaggle ('FitBit Fitness Tracker Data' by Mobius)
- 18 Excel files tracking minute-level physical activity, steps, calories, heart rate, and sleep data from 30 users.
- Datasets selected for analysis:
  - `dailyActivity_merged`
  - `dailyCalories_merged`
  - `dailyIntensities_merged`
  - `dailySteps_merged`
  - `heartrate_seconds_merged`
  - `sleepDay_merged`
  - `weightLogInfo_merged`

---

## 3. Data Cleaning

- Duplicates removed: 1 in `dailySteps_merged`, 14,786 in `heartrate_seconds_merged`
- Dates standardized (MM/DD/YYYY), time split using 'Text to Column'
- Invalid and blank rows removed
- Data uploaded to Google BigQuery for analysis; Tableau used for visualization.

---

## 4. SQL Analysis

Unique users identified with:

```sql
SELECT 
  COUNT(DISTINCT(Id)) AS user_number
FROM `cases-studies-459821.Bellabeat.daily_activity`
```

- 5 tables shared the same 33 user IDs; others (like `weight`) had fewer.

---

## 5. Data Merging and User Segmentation

Five tables with identical user IDs and activity dates were merged using LEFT JOINs:

```sql
SELECT 
  activity.Id AS id,
  activity.ActivityDate AS activity_date,
  activity.TotalSteps AS total_steps,
  activity.TotalDistance AS total_distance,
  calories.Calories AS calories,
  intensities.SedentaryMinutes AS sedentary_minutes,
  intensities.LightlyActiveMinutes AS lightly_active_minutes,
  intensities.FairlyActiveMinutes AS fairly_active_minutes,
  intensities.VeryActiveMinutes AS very_active_minutes
FROM `cases-studies-459821.Bellabeat.daily_activity` AS activity
LEFT JOIN `cases-studies-459821.Bellabeat.daily_calories` AS calories
  ON activity.Id = calories.Id AND activity.ActivityDate = calories.ActivityDay
LEFT JOIN `cases-studies-459821.Bellabeat.daily_intensities` AS intensities
  ON activity.Id = intensities.Id AND activity.ActivityDate = intensities.ActivityDay
LEFT JOIN `cases-studies-459821.Bellabeat.daily_steps` AS steps
  ON activity.Id = steps.Id AND activity.ActivityDate = steps.ActivityDay
```

- Weight table excluded (only 8 users).

---

## 6. Classifying Users by Frequency of Use

Users categorized as light, moderate, and active by days logged:

```sql
SELECT
  id,
  COUNT(id) AS total_uses,
  CASE
    WHEN COUNT(id) BETWEEN 4 AND 13 THEN 'light_user'
    WHEN COUNT(id) BETWEEN 14 AND 23 THEN 'moderate_user'
    WHEN COUNT(id) BETWEEN 24 AND 31 THEN 'active_user'
  END AS user_classification
FROM `cases-studies-459821.Bellabeat.daily_activity_full`
GROUP BY id
```

- 87.88% of users are active, logging data most days in the 31-day period.

---

## 7. Descriptive Statistics

Minimum, maximum, and average values for steps, distance, calories, and activity intensities:

```sql
SELECT 
  id,
  ROUND(MIN(total_steps),2) AS min_total_steps,
  ROUND(MAX(total_steps),2) AS max_total_steps,
  ROUND(AVG(total_steps),2) AS avg_total_steps,
  ROUND(MIN(calories),2) AS min_calories,
  ROUND(MAX(calories),2) AS max_calories,
  ROUND(AVG(calories),2) AS avg_calories,
  ROUND(AVG(very_active_minutes),2) AS avg_very_actives_minutes,
  ROUND(AVG(fairly_active_minutes),2) AS avg_fairly_active_minutes,
  ROUND(AVG(lightly_active_minutes),2) AS avg_lightly_active_minutes,
  ROUND(AVG(sedentary_minutes),2) AS avg_sedentary_minutes
FROM `cases-studies-459821.Bellabeat.daily_activity_full`
GROUP BY id
```

- Sedentary minutes had the highest average, indicating prolonged inactivity.

---

## 8. Lifestyle Classification by Steps

NIH guidelines used to classify users by average daily steps:

```sql
SELECT 
  id,
  avg_total_steps,
  CASE 
    WHEN avg_total_steps < 5000 THEN 'sedentary_life'
    WHEN avg_total_steps BETWEEN 5000 AND 7499 THEN 'low_active_life'
    WHEN avg_total_steps BETWEEN 7500 AND 9999 THEN 'moderately_active_life'
    WHEN avg_total_steps BETWEEN 10000 AND 12499 THEN 'active_life'
    WHEN avg_total_steps > 12500 THEN 'highly_active_life'
  END AS life_classification
FROM `cases-studies-459821.Bellabeat.min_max_avg_from_daily_activity`
```

---

## 9. Lifestyle Classification by Calories Burned

Based on Total Daily Energy Expenditure standards:

```sql
SELECT 
  id,
  avg_calories,
  CASE
    WHEN avg_calories < 1600 THEN 'sedentary_life'
    WHEN avg_calories BETWEEN 1600 AND 1999 THEN 'lightly_active_life'
    WHEN avg_calories BETWEEN 2000 AND 2199 THEN 'moderately_active_life'
    WHEN avg_calories BETWEEN 2200 AND 2600 THEN 'active_life'
    WHEN avg_calories > 2600 THEN 'highly_active_life'
  END AS life_calories_classification
FROM `cases-studies-459821.Bellabeat.min_max_avg_from_daily_activity`
```

---

## 10. Correlation Analysis

- Steps vs Intensity: R = 0.611 (Strong positive)
- Intensity vs Calories: R = 0.354 (Moderate positive)
- Steps vs Calories: R = 0.326 (Moderate positive)

> Steps and intensity are strongly associated; calorie burn is influenced by several factors (weight, pace, terrain).

---

## 11. WHO Guidelines Evaluation

WHO recommends 150–300 minutes of moderate-intensity activity weekly. Analysis (excluding lightly active minutes):

```sql
SELECT 
  id,
  avg_activity_minutes_per_week_per_user,
  CASE
    WHEN avg_activity_minutes_per_week_per_user < 150 THEN "is not performing as recommended"
    WHEN avg_activity_minutes_per_week_per_user BETWEEN 150 AND 300 THEN "is performing as recommended"
    WHEN avg_activity_minutes_per_week_per_user > 300 THEN "is exceeding recommendations"
  END AS performance_category
FROM `cases-studies-459821.Bellabeat.avg_minutes_activity_intensity_no_ligthly`
```

- 9 users: not performing as recommended
- 15 users: met recommendations
- 9 users: exceeded recommendations

---

## 12. Key Insights

- 87.88% of users were highly engaged (active category)
- Sedentary minutes had the highest daily averages
- Step-based and calorie-based lifestyle classifications differ significantly
- Steps and intensity have the strongest correlation (R = 0.611)
- WHO guideline adherence: 45% met, 27% exceeded, 27% did not meet

---

## 13. Marketing Recommendations

- Engage sedentary/low-active users with micro-goals and movement prompts
- Promote hydration synergy with the Spring water bottle and fitness tracking
- Educate on step intensity benefits with visual content and progress tracking
- Highlight Bellabeat Time and Leaf for activity/stress monitoring
- Use segmentation (moderate vs high performers) for personalized campaigns

---

## Appendix: Data Visualizations (Descriptions)

- **User Classification by App Usage:**
  Bar chart of percentage of user types by login frequency.-
   ![User Classification 1](https://github.com/Carole1808/data-analyst-portfolio/blob/main/Bellabeat-Case-Study/Visualizations/User%20Classification%201.png)
- **Activity Classification per Minute:**
  Chart of activity (sedentary, fairly, lightly, very active) by time per login.
  ![Activity Classification](https://github.com/Carole1808/data-analyst-portfolio/blob/main/Bellabeat-Case-Study/Visualizations/Activity%20classification%202.png)
- **Activity Classification per Day:**
  Chart of activity classification per day of week.
  ![Activity classification per day of week](https://github.com/Carole1808/data-analyst-portfolio/blob/main/Bellabeat-Case-Study/Visualizations/Activity%20classification%20per%20day%20of%20week.png)
- **Life Users Classification:**
  Treemap of user lifestyle categories by steps per day.
  ![Life Style User Classification](https://github.com/Carole1808/data-analyst-portfolio/blob/main/Bellabeat-Case-Study/Visualizations/life%20Style%20User%20Classification.png)
- **Performing of the Users:**
  Pie chart on WHO recommendation adherence.
  ![Performing Recomended](https://github.com/Carole1808/data-analyst-portfolio/blob/main/Bellabeat-Case-Study/Visualizations/Performing%20Recomended.png)
- **Steps vs Calories Burned:**
  Scatter plot of average daily steps vs calories burned, with trendline.
  ![Correlation Steps Vs Calories](https://github.com/Carole1808/data-analyst-portfolio/blob/main/Bellabeat-Case-Study/Visualizations/Correlation%20Steps%20Vs%20Calories.png)
- **Correlation among Steps, Calories, Intensity:**
  Visualization comparing their relationships.
  ![Performing Recomended](https://github.com/Carole1808/data-analyst-portfolio/blob/main/Bellabeat-Case-Study/Visualizations/Correlation%20among%20Steps%2C%20Calories%2C%20Intensity.png)
