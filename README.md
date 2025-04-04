# Overview 
This project focuses on data cleaning and exploratory data analysis(EDA) using SQL. Best practice approach were used to prepare and analyze dataset.

# Key Task 
- [Data Cleaning](#-data-cleaning) : Removed Duplicates, Standardize the Data, Missing values and Remove Any Columns
- [EDA](#-data-exploratory-analysis) : Analyzing the layoffs trends in various companies across different industry sectors

# Tools 
- I used the MySQL program to write in the SQL Language to query from the dataset.

# Datasets 
- The dataset used in this project was provided by Alex The Analyst as part of his SQL course. You can find the original dataset and learning materials here:
  [Alex the Analyst-SQL Course](https://www.youtube.com/watch?v=OT1RErkfLNQ)
  
## 🧹 Data Cleaning 

1. Create a layoff staging table which maintains the intergrity of the original raw data
- layoffs_staging is a duplicate of the origanal table 
```sql
CREATE TABLE LAYOFFS_STAGING   
LIKE LAYOFFS;

Insert LAYOFFS_STAGING
SELECT*
FROM LAYOFFS
;
```
**Result**: 

<kbd><img width="602" alt="Layoff_staging_Table1" src="https://github.com/user-attachments/assets/af406c6a-620c-409d-927f-869cf7fe74fe" />

2. Remove Duplicates

To do this, we will need to create a Common Table Expression (CTE) to store and partition it for every column. This feedbacks it back into the row_num column.  

```sql
(SELECT *, 
 ROW_NUMBER () OVER(
 PARTITION BY COMPANY,LOCATION, INDUSTRY, TOTAL_LAID_OFF, PERCENTAGE_LAID_OFF, `DATE`,STAGE
 ,COUNTRY,FUNDS_RAISED_MILLIONS) AS ROW_NUM
FROM layoffs_staging
)
```

This returns back the rows that has any repetition by filter using row num > 1 .
```sql
SELECT *
FROM DUPLICATE_CTE
WHERE ROW_NUM >1
;
```

**Results**: <br>
This displays the lists of companies that has duplicates.

<kbd><img width="500" alt="Removeduplicate" src="https://github.com/user-attachments/assets/aa7f886c-a4fe-4ce2-b735-0941bd6959e7" />

Now we create second table with the same columns and an additional row_num column into another database layoffs_staging2, to filter out to duplicates as shown above. As CTE cannot be directly deleted.

```sql
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `ROW_NUM` INT   # ADDING A NEW COLUMN = ROW_NUM 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

Insert the data from layofss_staging into layoffs_staging2. Delete company that have the row_num > 1.

```sql
INSERT INTO LAYOFFS_STAGING2
SELECT *, 
 ROW_NUMBER () OVER(
 PARTITION BY COMPANY,LOCATION, INDUSTRY, TOTAL_LAID_OFF, PERCENTAGE_LAID_OFF, `DATE`,STAGE
 ,COUNTRY,FUNDS_RAISED_MILLIONS) AS ROW_NUM
FROM layoffs_staging
;

DELETE 
FROM LAYOFFS_STAGING2
WHERE ROW_NUM >1 ;
```
**Results** :<br>

Removed the duplicates sucessfully! Here’s a snippet showcasing a cleaned dataset for one company, confirming all duplicate rows have been eliminated.

<kbd><img width="684" alt="Final_remove_Dup" src="https://github.com/user-attachments/assets/fd15cf6d-2e31-4c47-9fbd-a8d56c3b0f60" />

3. Standardize Data 
- Trim <br>
  
Trim the white spaces on the front & back end of the company column. Then the updated company name to the trim(company) we have created.

````sql
SELECT Company, TRIM(Company)
FROM layoffs_staging2
;
````
**Results**: <br>
<kbd><img width="123" alt="image" src="https://github.com/user-attachments/assets/c416c2b9-f346-4a44-9d2c-97bb4a475df6" />

- Group the names of the same industry into one, as Crypto is also Crypto Currency.

```sql
UPDATE LAYOFFS_STAGING2 
SET INDUSTRY = 'Crypto'
WHERE INDUSTRY LIKE 'CRYPTO%'
;
```
Before:<br>
<kbd><img width="130" alt="BEFORE_group in one industry" src="https://github.com/user-attachments/assets/447f7acd-7f77-411a-809b-52f9b169239a" />

After:<br>
<kbd><img width="130" alt="image" src="https://github.com/user-attachments/assets/5a336227-b3f6-4f11-bc8e-0d9d5fbb6101" />

- Removing inconsistent data of country for the same name -United States into one.
  
Before:<br>
<kbd><img width="145" alt="image" src="https://github.com/user-attachments/assets/1d4a0056-68dc-4536-ac29-e1d87e7d4fa4" />

````SQL
SELECT DISTINCT COUNTRY, TRIM(TRAILING '.' FROM COUNTRY)
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2 
SET COUNTRY = trim(trailing '.' FROM country)
WHERE COUNTRY LIKE 'UNITED STATES%'
;
````
- Removed the '.' in United States and updated the column to the trimmed column country we made.
  
After: <br>
<KBD><img width="155" alt="image" src="https://github.com/user-attachments/assets/bcb7a316-7508-4e03-8798-89969e35da5c" />

- Changing the Date which is a text into a standard date format and date type.
  
```sql
SELECT `date`
STR_TO_DATE(`date`,'%m/%d/%Y',)
FROM LAYOFSS_STAGING2
;

UPDATE LAYOFSS_STAGING2
SET`date` = STR_TO_DATE('date`,%m,%d,%Y)
;

ALTER TABLE LAYOFFS_STAGING2
MODIFY COLUMN `date` DATE;
```

Before: <br>
<kbd><img width="80" alt="B4String2Date" src="https://github.com/user-attachments/assets/77a5c672-9b60-4180-a053-fa0f7f46ede2" />

After:<br>
<kbd><img width="80" alt="image" src="https://github.com/user-attachments/assets/c7acbc8d-d317-4bf0-ae01-3ef279fe633e" />

4. Remove Nulls and Blanks
- Change Blank Values into Nulls
- Populate the blanks with existing data using JOIN tables

  ```sql
  UPDATE layoffs_staging2
  SET industry = NULL
  WHERE indusry = ''
  ;
  SELECT *
  FROM layoffs_staging2 t1
  JOIN layofss_staging2 t2
      ON t1.company = t2.company
  WHERE (t1.industry IS NULL OR t1.industry = '')
  AND t2.industry IS NOT NULL

  UPDATE layoffs_staging2 t1
  JOIN layofss_staging2 t2
    ON t1.company = t2.company
  SET t1.industry =t2.industry
  WHERE t1.industry IS NULL
  AND t2.industry is NOT NULL
  ;
  ```
**Before**<br>
<kbd><img width="500" alt="image" src="https://github.com/user-attachments/assets/c45f0bf2-7560-4b2f-be6c-51d69228b044" />

  
**Results** : <br>
<kbd><img width="500" alt="image" src="https://github.com/user-attachments/assets/4c7d76ee-9cd3-4538-8397-bcd5dfb7425e" />

5. Remove Columns & Rows
- Delete Rows wherer total_laif_off is blank and percentage_laid_off is NULL
- Absent values on these 2 columns makes the data irrelevant

```sql
DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL
;
```
Previously, we created a row_num column to filter out duplicates in every column.<br>
Now, we are dropping the column as we do not need it anymore.

```sql
ALTER TABLE layoffs_staging2
DROP COLUMN ROW_NUM;
```
**Before**:<br>
<kbd><img width="123" alt="image" src="https://github.com/user-attachments/assets/bc5a0e6f-bdff-4f92-9b9e-ea2b57e02a2b" />

**Results**:<br>

<kbd><img width="123" alt="image" src="https://github.com/user-attachments/assets/c77be6ca-e2ac-43bb-9a01-e22dd2f8c5a6" />

## 🔎 Data Exploratory Analysis

We did some digging into the cleaned data to see insights into the trends on the topic of laid off.

### 1. Max Number of Laid Off

<kbd><img width="200" alt="image" src="https://github.com/user-attachments/assets/4780c221-6173-4980-9fdc-e34ebb08ae2d" />

### 2. Companies with 100% Workforce Laid Off 

- This gives Katera with laying of all of its 2434 employees, leading to company closure.
  
  <kbd><img width="335" alt="image" src="https://github.com/user-attachments/assets/61dd3b04-7c21-4cbf-9841-75c1944febe7" />

### 3. Companies with the Highest Total Layoffs


  <kvd><img width="143" alt="image" src="https://github.com/user-attachments/assets/799794d1-3653-4d76-a099-2785f39d96f7" />

### 4. Industries Most Affected by Layoffs

- The industries with the highest layoffs during the COVID-19 pandemic are likely those heavily reliant on in-person interactions and consumer spending. Lockdowns, social distancing measures, and economic uncertainty significantly reduced foot traffic in retail stores, usage of transportation services, and other service-based sectors. As a result, companies in these industries faced revenue losses, leading to large-scale layoffs.
  
  <kbd><img width="120" alt="image" src="https://github.com/user-attachments/assets/e90b4640-1cab-4a5a-9b57-45cbd5f3f781" />


### 5. Total Layoffs by Year

Looking into individual dates for total laid off.

<kbd><img width="127" alt="image" src="https://github.com/user-attachments/assets/370652ef-c5ba-4f13-8f6f-c1cf703328c8" />

For 2023, data of laid off is collected until March.
  
<kbd><img width="120" alt="image" src="https://github.com/user-attachments/assets/297d181d-1833-49f8-89f4-a6b7348b96be" />

Thus, we conclude that highest year to be 2023 as the numbers is at 120000 only 3 months in.


### 6. Monthly Rolling Sum of Layoffs

- Calculated a rolling sum of total layoffs on a monthly basis for each year to analyze trends over time.
- Created a Commn table expression with the rolling total we did

```sql
WITH ROLLING_TOT AS
(SELECT SUBSTRING(`DATE`,1,7) as `MONTH`, SUM(total_laid_off)AS TOT_OFF
FROM layoffs_staging2
WHERE SUBSTRING(`DATE`,1,7) IS NOT NULL
GROUP BY `MONTH`
ORDER BY 1 ASC
)
SELECT `MONTH`, TOT_OFF,
  SUM(TOT_OFF) OVER(ORDER BY `MONTH`) AS rolling_total
FROM ROlling_total
;
```

<kbd><img width="123" alt="image" src="https://github.com/user-attachments/assets/2f439ada-9575-4b73-be6a-a18c01f9e7ba" />

- This gives is us an insights into the trend of layoffs over time.
- Data shows that an increase over the years of number of laidsofss.
- Around 10,000 people were laid off in **2021** which is good compared to **2022** as the number skyrockets to an estimate of 160,000 people.
- The beginning of **2023** shows the numbers rapidly increases.
- It is assumed that the numbers will rise significantly from there, as effects of COVID pandemic hits worldwide.

## 7. Company Laid off over the Years

- Using CTE to query the top 5 companies for total_laid_off for each year.
- 1st CTE contain the query for total_laid_off over years.
- 2nd CTE queries includes the ranking and ties it to the 1st CTE.
  
```sql
WITH COMPANY_YEAR(COMPANY, YEARS, TOTAL_LAID_OFF) AS  -- 1ST CTE
(
SELECT COMPANY, YEAR(`date`),sum(total_laid_off) 
FROM layoffs_staging2
GROUP BY COMPANY,YEAR(`date`)
)  ,Company_Year_Rank AS -- 2ND CTE USE THE 1ST CTE
(SELECT * ,
DENSE_RANK() OVER( PARTITION BY YEARS ORDER BY TOTAL_LAID_OFF DESC) AS RANKING
FROM COMPANY_YEAR
WHERE YEARS IS NOT NULL 
) 

SELECT *   -- QUERY THE FINAL CTE/ 2ND CTE
FROM COMPANY_YEAR_RANK
WHERE RANKING <= 5
;

```
**1st CTE Results** : <br>

<kbd><img width="161" alt="image" src="https://github.com/user-attachments/assets/3b87ff78-5a48-41f5-b0ca-88b7233b0a58" />

**2nd CTE Results** : <br>

<kbd><img width="154" alt="image" src="https://github.com/user-attachments/assets/d7dde01a-dacd-48a7-b10f-fa0b4d9e94be" />



# Limitations<br>
- Numerical data for total_laid_off and percentage_laid_off which has NULL were not taken into account from this as the data would become inacurate and unreliable.
- The rolling total of employees laid off is an estimate since the data doesn’t fully reflect real-world layoffs. Some small companies and job sectors
  may not report their layoffs.


