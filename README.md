# Overview 
This project focuses on data cleaning and exploratory data analysis(EDA) using SQL. Best practice approach were used to prepare and analyze dataset.

# Key Task 
- **Data Cleaning** : Removed Duplicates, Standardize the Data, Missing values and Remove Any Columns
- **EDA** : Analyzing the layoffs trends in various companies across different industry sectors

# Tools 
- I used the MySQL program to write in the SQL Language to query from the dataset.

# Datasets 
- The dataset used in this project was provided by Alex The Analyst as part of his SQL course. You can find the original dataset and learning materials here:
  [Alex the Analyst-SQL Course](https://www.youtube.com/watch?v=OT1RErkfLNQ)
  
# Next Steps 
I plan to document my queries and findings in detail soon. Stay tuned for updates!

## 🧹 Data Cleaning 

1. Create a layoff staging table which maintains the intergrity of the original raw data
  - layoffs staging is a duplicate of the origanal table 
```
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

To do this, we will need to create a Common Table Expression (CTE) to store and partition it for every column. 

```
(SELECT *, 
 ROW_NUMBER () OVER(
 PARTITION BY COMPANY,LOCATION, INDUSTRY, TOTAL_LAID_OFF, PERCENTAGE_LAID_OFF, `DATE`,STAGE
 ,COUNTRY,FUNDS_RAISED_MILLIONS) AS ROW_NUM
FROM layoffs_staging
)
```

 This returns back the rows that has any repetition by filter using row num > 1 .
```
SELECT *
FROM DUPLICATE_CTE
WHERE ROW_NUM >1
;
```

**Results**: 
This displays the lists of companies that has duplicates.

<kbd><img width="452" alt="Removeduplicate" src="https://github.com/user-attachments/assets/aa7f886c-a4fe-4ce2-b735-0941bd6959e7" />

