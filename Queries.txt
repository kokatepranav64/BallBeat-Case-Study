# Selecting dates and corresponding days of the week to identify weekdays and weekends
SELECT ActivityDate, DAYNAME(ActivityDate) AS DayOfWeek
FROM DailyActivity
;

SELECT ActivityDate, 
	CASE 
		WHEN DayOfWeek = 'Monday' THEN 'Weekday'
		WHEN DayOfWeek = 'Tuesday' THEN 'Weekday'
		WHEN DayOfWeek = 'Wednesday' THEN 'Weekday'
		WHEN DayOfWeek = 'Thursday' THEN 'Weekday'
		WHEN DayOfWeek = 'Friday' THEN 'Weekday'
		ELSE 'Weekend' 
	END AS PartOfWeek
FROM
	(SELECT *, DAYNAME(ActivityDate) AS DayOfWeek
	FROM DailyActivity) as temp
;

# Looking at average steps, distance, and calories on weekdays vs. weekends
SELECT PartOfWeek, AVG(TotalSteps) AS AvgSteps, AVG(TotalDistance) AS AvgDistance, AVG(Calories) AS AvgCalories
FROM 
	(SELECT *, 
		CASE 
			WHEN DayOfWeek = 'Monday' THEN 'Weekday'
			WHEN DayOfWeek = 'Tuesday' THEN 'Weekday'
			WHEN DayOfWeek = 'Wednesday' THEN 'Weekday'
			WHEN DayOfWeek = 'Thursday' THEN 'Weekday'
			WHEN DayOfWeek = 'Friday' THEN 'Weekday'
			ELSE 'Weekend'
		END AS PartOfWeek
	FROM
		(SELECT *, DAYNAME(ActivityDate) AS DayOfWeek
		FROM DailyActivity) as temp
	) as temp2
GROUP BY PartOfWeek
;

# Looking at average steps, distance, and calories per day of the week
SELECT DAYNAME(ActivityDate) AS DayOfWeek, AVG(TotalSteps) AS AvgSteps, AVG(TotalDistance) AS AvgDistance, AVG(Calories) AS AvgCalories
FROM DailyActivity
GROUP BY DayOfWeek
ORDER BY AvgSteps DESC
;

# Looking at average amount of time spent asleep and average time to fall asleep per day of the week
SELECT DAYNAME(SleepDay) AS DayOfWeek, AVG(TotalMinutesAsleep) AS AvgMinutesAsleep, AVG(TotalMinutesAsleep / 60) AS AvgHoursAsleep, AVG(TotalTimeInBed - TotalMinutesAsleep) AS AvgTimeInMinutesToFallAsleep
FROM SleepLog
GROUP BY DayOfWeek
ORDER BY AvgHoursAsleep DESC
;

# Left joining all 3 tables
SELECT *
FROM DailyActivity AS d 
LEFT JOIN SleepLog AS s
ON d.ActivityDate = s.SleepDay AND d.Id = s.Id
LEFT JOIN WeightLog AS w
ON s.SleepDay = w.Date AND s.Id = w.Id
ORDER BY d.Id, Date
;

# Inner joining all 3 tables
SELECT *
FROM DailyActivity AS d 
JOIN SleepLog AS s
ON d.ActivityDate = s.SleepDay AND d.Id = s.Id
JOIN WeightLog AS w
ON s.SleepDay = w.Date AND s.Id = w.Id
ORDER BY d.Id, Date
# Only 35 rows returned
;

# Finding unique participants in the DailyActivity who do not have records in either the SleepLog or WeightLog tables (or both)
SELECT DISTINCT Id
FROM DailyActivity
WHERE Id NOT IN (
	SELECT d.Id
	FROM DailyActivity AS d 
	JOIN SleepLog AS s
	ON d.ActivityDate = s.SleepDay AND d.Id = s.Id
	JOIN WeightLog AS w
	ON s.SleepDay = w.Date AND s.Id = w.Id)
# Out of 33 participants in the DailyActivity table, 28 do not have records in either the SleepLog or WeightLog tables (or both)
;

# Looking at instances where users don't have records in SleepLog based on day of the week
SELECT DAYNAME(ActivityDate) AS DayOfWeek, COUNT(*) AS num
FROM DailyActivity AS d 
	LEFT JOIN SleepLog AS s
	ON d.ActivityDate = s.SleepDay AND d.Id = s.Id
WHERE s.TotalMinutesAsleep IS NULL
GROUP BY DayOfWeek
ORDER BY num DESC
;

# Looking at calories and active minutes
SELECT Id, ActivityDate, Calories, SedentaryMinutes, LightlyActiveMinutes, FairlyActiveMinutes, VeryActiveMinutes
FROM DailyActivity
;

# Looking at calories and active distances
SELECT Id, ActivityDate, Calories, SedentaryActiveDistance, LightActiveDistance, ModeratelyActiveDistance, VeryActiveDistance, TotalDistance
FROM DailyActivity
;

# Looking at calories and total steps
SELECT Id, ActivityDate, Calories, TotalSteps
FROM DailyActivity
;

# Looking at calories and total minutes asleep
SELECT d.Id, d.ActivityDate, Calories, TotalMinutesAsleep
FROM DailyActivity AS d 
INNER JOIN SleepLog AS s 
ON d.Id = s.Id AND d.ActivityDate = s.SleepDay
;

# Looking at calories and total minutes & hours asleep from day before
SELECT d.Id, d.ActivityDate, Calories, TotalMinutesAsleep,
			LAG(TotalMinutesAsleep, 1) OVER (ORDER BY d.Id, d.ActivityDate) AS MinutesSleptDayBefore,
			LAG(TotalMinutesAsleep, 1) OVER (ORDER BY d.Id, d.ActivityDate) / 60 AS HoursSleptDayBefore
FROM DailyActivity AS d 
INNER JOIN SleepLog AS s 
ON d.Id = s.Id AND d.ActivityDate = s.SleepDay
;

# Looking at manual reports vs. automated reports in WeightLog table; also looking at average weight of participants whose reports were generated manually vs. automatically
SELECT IsManualReport, COUNT(DISTINCT Id)
FROM WeightLog
GROUP BY IsManualReport
;

SELECT IsManualReport, COUNT(*) AS num_reports, AVG(WeightPounds) AS avg_weight
FROM WeightLog
GROUP BY IsManualReport
;

# Looking at all Minutes (inc. new column of total minutes) in DailyActivity table
SELECT Id, ActivityDate, (SedentaryMinutes + LightlyActiveMinutes + FairlyActiveMinutes + VeryActiveMinutes) AS TotalMinutes, SedentaryMinutes, LightlyActiveMinutes, FairlyActiveMinutes, VeryActiveMinutes
FROM DailyActivity
;

# Looking at non-sedentary minutes and total sleep
SELECT d.Id, ActivityDate, LightlyActiveMinutes, FairlyActiveMinutes, VeryActiveMinutes, (LightlyActiveMinutes + FairlyActiveMinutes + VeryActiveMinutes) AS TotalMinutes, TotalMinutesAsleep, (TotalTimeInBed - TotalMinutesAsleep) AS MinutesToFallAsleep
FROM DailyActivity AS d
JOIN SleepLog AS s
ON d.Id = s.Id AND ActivityDate = SleepDay
;

# Looking at number of days where total steps is equal to or greater than the CDC-recommended amount of 10,000
SELECT DAYNAME(ActivityDate) AS DayOfWeek, COUNT(*)
FROM DailyActivity
WHERE TotalSteps >= 10000
GROUP BY DayOfWeek
;

# Looking at number of days where users got the CDC-recommended amount of sleep (7-9 hours a night)
SELECT DAYNAME(ActivityDate) AS DayOfWeek, COUNT(*) AS NumDays
FROM DailyActivity AS d 
JOIN SleepLog AS s
ON d.Id = s.Id AND d.ActivityDate = s.SleepDay
WHERE TotalMinutesAsleep >= 420 AND TotalMinutesAsleep <= 540
GROUP BY DayOfWeek
ORDER BY NumDays DESC
;