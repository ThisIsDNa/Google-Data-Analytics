# Google Data Analytics Capstone: Bellebeat Fitness

![Bellebeat Logo](https://user-images.githubusercontent.com/42982734/174476404-0f292d86-b46e-4239-a9d3-289abeaf9ebc.png)

## Introduction

Bellebeat is a high-tech manufacturer of health-focused products for women. Recently, they have been investing extensively on digital marketing platforms such as Google Search and being active on various social media platforms. However, their CEO, Urska Srsen, believes that by analyzing their smart device fitness data they can unlock new opportunities of growth for their company. In this case study, I will primarily focus on their product the Bellabeat Leaf - a wellness tracker that connects to their app to track activity, sleep, and stress - in order to identify trends and gain insight on how consumers are using their Leaf. With this information I will then provide high-level  recommendations for Bellebeat's marketing strategy.

## The Details
Bellebeat is a successful small company, but they have the potential to hold their own in the global smart device market. While they have several products, their claim to fame is their Belleabeat Leaf - a water resistant device that tracks activity, stress, sleep, mediation, and reproductive health. This device then interprets this data to  their consumer to help adjust diet, excercise, and wellness to help their clients become the best version of themselves.

In this company there are two key stakeholders:

Urška Sršen, Bellabeat’s co-founder and Chief Creative Officer;
Sandro Mur, Bellabeat’s co-founder, and key member of the Bellabeat executive team. 

## The Data
This project utilizes the FitBit Fitness Tracker Data, a Public Domain dataset made available through Mobius. These datasets were generated by people who responded to a distributed survey bia Amazon Mechanical Turke from April 12, 2016 - May 12, 2016. Thirty FitBit users consented to share their personal tracker data, including minute-level output for physical activity, heart rate, sleep monitoring, and individual behaviors/preferences. Individual reports are anonymized and utilize ID as an identifier so the data is private and secure. However, there are several issues with this data straight from the get-go:

1. Small sample size (30 participants)
2. Short observation period (1 month)
3. Data was recorded over 5 years ago (2016)

## Processing
I decided to utilize SQL and Tableau for this project. 
The reason I chose SQL was because I would be able to quickly and efficiently retrieve specific information from a large database.
I chose Tableau as my data visualization tool since it was user-friendly and able to handle large amounts of data.

### Quick overview of data, then looking to see how many users there are
~~~
SELECT *
FROM fitbit_fitness.dailyActivity_merged
LIMIT 10;

SELECT DISTINCT ID
FROM fitbit_fitness.dailyActivity_merged;
~~~
Distinct ID's by Table
- dailyActivity: 33
- hourlyCalories: 33
- hourlyIntensities: 33
- hourlySteps: 33
- sleepDay: 24
- weightLogInfo: 8

### Get a quick statistical summary to find interesting data
~~~
SELECT MAX(TotalSteps) AS Maximum,
       MIN(TotalSteps) AS Minimum,
       AVG(TotalSteps) AS Average,
       STDDEV(TotalSteps) AS StdDev,
       VARIANCE(TotalSteps) AS Variance,
FROM fitbit_fitness.dailyActivity_merged;
~~~
![image](https://user-images.githubusercontent.com/42982734/174471414-37012bed-29db-41c7-96ec-ecb166ba97d6.png)

### Checking for duplicates
~~~
SELECT a.*
FROM `master-scanner-338804.fitbit_fitness.dailyActivity_merged` a
JOIN (SELECT Id, ActivityDate, COUNT(*)
FROM `master-scanner-338804.fitbit_fitness.dailyActivity_merged`
GROUP BY Id, ActivityDate
HAVING COUNT(*) > 1) b
ON a.Id = b.Id
AND a.ActivityDate = b.ActivityDate
ORDER BY a.Id;
~~~

There were duplicates found in sleepDay. So, further cleaning was necessary

### Removing Duplicates
~~~
WITH CTE AS (
    SELECT Id, SleepDay, TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed,
        RN = ROW_NUMBER()OVER(PARTITION BY Id ORDER BY Id)
    from `master-scanner-338804.fitbit_fitness.sleepDay_merged`
)
DELETE FROM CTE WHERE RN > 1;
~~~

### Checking for NA
~~~
SELECT *
FROM `master-scanner-338804.fitbit_fitness.dailyActivity_merged`
WHERE Id IS NULL;
~~~


## Analyzing
### Looking at Activity Level (Time) vs Calories
~~~
SELECT Id, 
       ActivityDate,
       Calories,
       (VeryActiveMinutes+FairlyActiveMinutes+LightlyActiveMinutes) AS TotalActiveMinutes, 
       SedentaryMinutes
FROM fitbit_fitness.dailyActivity_merged
LIMIT 10;
~~~
![image](https://user-images.githubusercontent.com/42982734/174474102-8b4bad4d-23fe-4460-9848-acada9729d16.png)

It may sound obvious, but the data shows that the more time you spend active the more calories you burn.

### Looking at Activity Level (Time) vs Sleep
~~~
SELECT activity.Id, 
       ActivityDate, 
       Calories, 
       TotalSleepRecords, 
       TotalMinutesAsleep, 
       TotalTimeInBed, 
       (TotalTimeInBed - TotalMinutesAsleep) AS TimeToSleep,
       (VeryActiveMinutes+FairlyActiveMinutes+LightlyActiveMinutes) AS ActiveMinutes, 
       SedentaryMinutes
FROM fitbit_fitness.dailyActivity_merged AS activity
INNER JOIN fitbit_fitness.sleepDay_merged AS sleep
ON activity.Id = sleep.Id AND activity.ActivityDate = sleep.SleepDay
LIMIT 10;
~~~
![image](https://user-images.githubusercontent.com/42982734/174474310-abc7a87a-60d7-455d-ad69-598f365443fa.png)

In short, the more time you spend active the less time it takes for people to go to sleep. 
Additionally, people who spend too much sedentary time may find that it takes longer for them to go to sleep.

### Looking at Activity Level (Time) vs BMI
~~~
SELECT activity.Id,
       ActivityDate,
       Calories, 
       BMI, 
       TotalSteps, 
       TotalDistance, 
       (VeryActiveDistance + ModeratelyActiveDistance + LightActiveDistance) AS TotalActiveDistance,
       SedentaryActiveDistance, 
       (VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes) AS TotalActiveMinutes, 
       SedentaryMinutes
FROM fitbit_fitness.dailyActivity_merged AS activity
INNER JOIN fitbit_fitness.weightLogInfo_merged AS weight
ON activity.Id = weight.Id;
~~~~
![image](https://user-images.githubusercontent.com/42982734/174474389-0ffcc7f3-7f4e-4665-aac7-d2092af5208e.png)

It shows a relationship that a person who has a higher average non-active minutes tends to have a higher average BMI. 
However, there are only 8 individuals who submitted their BMI information.
As such, this information can only provide a general idea of the relationship b/w Activity Level vs BMI

### Looking at how users spend their time
![image](https://user-images.githubusercontent.com/42982734/174474538-eaf9d89d-8f77-42ba-b864-e34699a50ae5.png)
![image](https://user-images.githubusercontent.com/42982734/174474744-ba90225f-a85c-4268-b3fa-c076fc237b2a.png)

In short, users spend most of their time at rest.
They tend to be more active in the middle of the week (Tuesday, Wednesday, Thursday) with their level of activity slowly decreasing and on Saturdays.
Sunday is generally considered as a rest day.

## Conclusion
From the results of the analysis, it is evident that there are clear benefits to having an active lifestyle. 
However, it is also clear that users spend most of their time at rest. 
As such, I recommend the following changes to get users more active and help them stay active.

- Advertisements can show the benefits of an active lifestyle (reduction in body fat, better sleep schedules, and better mental health) and how the Bellebeat Leaf can help them achieve their goals.
- Create weekly challenges that awards users points upon completion.
- Create a community/leaderboard so that users can share their progress with friends and compete with one another.
- Include a function on the app to remind users to be move active if they have been sedentary for too long.
- Encourage users to provide more information (BMI, Gender, Age, etc.) do that the Leaf can more accurately help them.

## Afterword
Additional data that would be relevant to enhancing my work.
1. Bellebeat Leaf Sales Data
2. Expanded Personal Information of Bellebeat Users for Segmentation Analysis
