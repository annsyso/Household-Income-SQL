# SQL Data Cleaning and Analysis Project

## Overview
This SQL project focuses on cleaning and analyzing U.S. household income data. The process includes:
1. Importing data and ensuring proper formatting.
2. Identifying and removing duplicates.
3. Standardizing naming conventions.
4. Performing exploratory analysis on land and water area distribution.
5. Joining datasets to examine income levels across locations.

Each step is designed to ensure data integrity and facilitate meaningful analysis.

---

## Step-by-Step Process

### **1. Importing Data and Ensuring Readability**
**Objective:** Ensure column names are clear and readable.

```sql
ALTER TABLE ushouseholdincome RENAME COLUMN `ALand` TO `Area_Land`;
ALTER TABLE ushouseholdincome RENAME COLUMN `AWater` TO `Area_Water`;
```

### **2. Checking for Duplicates**
**Objective:** Identify and remove duplicate rows to maintain data accuracy.

#### **Identify Duplicates**
```sql
SELECT id,
COUNT(id) AS count
FROM ushouseholdincome
GROUP BY id
HAVING count > 1;
```

#### **Assign Row Numbers to Duplicates**
```sql
SELECT *
FROM (
	SELECT row_id, id,
	ROW_NUMBER() OVER(PARTITION BY id ORDER BY id) as row_num
	FROM ushouseholdincome
	GROUP BY row_id, id
) AS dups
WHERE row_num > 1;
```

#### **Delete Duplicates**
```sql
DELETE FROM ushouseholdincome
WHERE row_id IN (
	SELECT row_id FROM (
		SELECT row_id, id,
		ROW_NUMBER() OVER(PARTITION BY id ORDER BY id) as row_num
		FROM ushouseholdincome
		GROUP BY row_id, id
	) AS dups
	WHERE row_num > 1
);
```

### **3. Standardizing Naming Conventions**
**Objective:** Correct inconsistencies in state names, empty fields, and data types.

#### **Fix Misspelled State Names**
```sql
UPDATE ushouseholdincome
SET State_Name = 'Georgia' WHERE State_Name = 'georia';
```

#### **Replace Blank Place Names**
```sql
UPDATE ushouseholdincome
SET Place = 'Autaugaville'
WHERE Place = '' AND County = 'Autauga County';
```

#### **Correct Lowercase Entries**
```sql
UPDATE ushouseholdincome
SET `Primary` = 'Place'
WHERE `Primary` = 'place';
```

#### **Standardize 'Type' Column**
```sql
UPDATE ushouseholdincome
SET `Type` = 'Borough'
WHERE `Type` = 'Boroughs';

UPDATE ushouseholdincome
SET `Type` = 'CDP'
WHERE `Type` = 'CPD';
```

### **4. Exploratory Analysis**
**Objective:** Analyze geographic land and water distribution.

#### **State-Level Land vs. Water Analysis**
```sql
SELECT State_Name,
ROUND(AVG(Area_Land), 2) AS avg_land,
ROUND(AVG(Area_Water), 2) AS avg_water
FROM ushouseholdincome
WHERE Area_Land <> 0 AND Area_Water <> 0
GROUP BY State_Name;
```

#### **County-Level Land vs. Water Analysis**
```sql
SELECT State_Name, County,
ROUND(AVG(Area_Land), 2) AS avg_land,
ROUND(AVG(Area_Water), 2) AS avg_water
FROM ushouseholdincome
WHERE Area_Land <> 0 AND Area_Water <> 0
GROUP BY State_Name, County;
```

#### **Counties with Largest Land Area**
```sql
SELECT State_Name, County,
ROUND(AVG(Area_Land), 2) AS avg_land,
ROUND(AVG(Area_Water), 2) AS avg_water
FROM ushouseholdincome
WHERE Area_Land <> 0 AND Area_Water <> 0
GROUP BY State_Name, County
ORDER BY avg_land DESC;
```

### **5. Income Analysis**
**Objective:** Join tables and analyze income distribution by location.

#### **Join Household Income Data with Income Statistics**
```sql
SELECT *
FROM ushouseholdincome u
JOIN ushouseholdincome_statistics us
ON u.id = us.id;
```

#### **Analyze Income by Location**
```sql
SELECT u.State_Name, u.County, Mean, Median
FROM ushouseholdincome u
JOIN ushouseholdincome_statistics us
ON u.id = us.id
WHERE Mean <> 0;
```

#### **Counties with Highest Median Income**
```sql
SELECT u.State_Name, u.County,
ROUND(AVG(Median), 2) AS avg_median
FROM ushouseholdincome u
JOIN ushouseholdincome_statistics us
ON u.id = us.id
WHERE Mean <> 0
GROUP BY u.State_Name, u.County
ORDER BY avg_median DESC;
```

#### **Top 10 States by Median Income**
```sql
SELECT u.State_Name,
ROUND(AVG(Median), 2) AS avg_median
FROM ushouseholdincome u
JOIN ushouseholdincome_statistics us
ON u.id = us.id
WHERE Mean <> 0
GROUP BY u.State_Name
ORDER BY avg_median DESC
LIMIT 10;
```

---

## Reasoning Behind Each Query
- **Renaming Columns**: Enhances readability and avoids confusion.
- **Checking for Duplicates**: Ensures data integrity by removing redundant records.
- **Standardizing Names**: Fixes inconsistencies that could lead to incorrect analysis.
- **Land and Water Analysis**: Provides insights into geographic distributions.
- **Income Analysis**: Helps identify economic patterns across different regions.

This structured approach ensures clean, reliable data, making analysis more effective and insightful.

---

## Conclusion
By following this process, we create a cleaned dataset suitable for advanced data analysis. This project sets the stage for further exploration into socioeconomic trends in the U.S.

