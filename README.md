# SQL_LAYOFF-Data-cleaning-and-EDA-Project

 This postgreSQL query was run by https://sqliteonline.com/


## Data Cleaning

**Import layoff dataset from localfile**

```sql
SELECT * FROM layoff;
```

**1. Remove Duplicates**

**2. Standardizing the Data**

**3. Null values or blank Values**

**4. Remove any Columns**


## 1. Remove Duplicates

```sql
CREATE TABLE layoff_staging (like layoff);
INSERT INTO layoff_staging SELECT * from layoff;
SELECT * FROM layoff_staging;
SELECT COUNT (*) FROM layoff_staging;
```
**Total rows 2361**

```sql
SELECT *,
 ROW_NUMBER() OVER(PARTITION by company, 'location', industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions)
 as row_num FROM layoff_staging;

WITH duplicate_cte AS
(
  SELECT *,
  ROW_NUMBER() OVER
  (
    PARTITION by company, 'location', industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions)
    as row_num FROM layoff_staging
 )
  SELECT * FROM duplicate_cte WHERE row_num >1;
  
  
  
  CREATE TABLE layoff_staging2(like layoff_staging);
  ALTER TABLE layoff_staging2 add column row_num INT;
  INSERT INTO layoff_staging2
  SELECT *,
  ROW_NUMBER() OVER(PARTITION by company, 'location', industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions)
  as row_num FROM layoff_staging;

 SELECT * FROM layoff_staging2;
```

 **We can delete rows where row_num is greater than 2**

```sql
 DELETE FROM layoff_staging2 WHERE row_num >1;
 SELECT COUNT (*) FROM layoff_staging2;
 ```
 
 **Total rows 2338**
 
 ## 2. Standardizing the data

 
**Remove unwanted space**
 
 ```sql
  SELECT company, TRIM(company) FROM layoff_staging2;
  UPDATE layoff_staging2 SET company=TRIM(company);
  SELECT company FROM layoff_staging2;
  ```

  **The Crypto has multiple different variations. We need to standardize that - let's say all to Crypto**

  ```sql
  SELECT DISTINCT industry FROM layoff_staging2 ORDER BY 1;
  SELECT * FROM layoff_staging2 WHERE industry LIKE 'Crypto%';
  UPDATE layoff_staging2 SET industry='Crypto' WHERE industry LIKE 'Crypto%';
  SELECT industry FROM layoff_staging2;
```
  
  **We have some "United States" and some "United States." with a period at the end. Let's standardize this.**
  
```sql
  SELECT DISTINCT country FROM layoff_staging2 ORDER BY  country;
  UPDATE layoff_staging2 SET country ='United States' WHERE  country LIKE 'United States%';
  SELECT country FROM layoff_staging2;
```

 **We have company name #Paid.Let's standardize this.**
 
```sql
SELECT DISTINCT company FROM layoff_staging2 ORDER BY  company;
  UPDATE layoff_staging2 SET company ='Paid' WHERE  company LIKE '%#';
  SELECT company FROM layoff_staging2 WHERE company='Paid';
 ```
  
 **Now we can convert the data type properly**

```sql
  ALTER TABLE layoff_staging2 RENAME COLUMN date TO dates;
  ALTER TABLE layoff_staging2 ALTER COLUMN dates TYPE DATE USING TO_DATE(dates,'MM/DD/YYYY');
  SELECT dates FROM layoff_staging2;
```
```sql
ALTER TABLE layoff_staging2
ALTER COLUMN total_laid_off TYPE FLOAT USING total_laid_off::FLOAT,
ALTER COLUMN percentage_laid_off TYPE FLOAT USING percentage_laid_off::FLOAT,
ALTER COLUMN funds_raised_millions TYPE FLOAT USING funds_raised_millions::FLOAT;
```
  
  
 **If we look at industry it looks like we have some null and empty rows, let's take a look at these**

  ```sql  
SELECT DISTINCT industry
FROM layoff_staging2
ORDER BY industry;

SELECT *
FROM layoff_staging2
WHERE industry IS NULL 
OR industry = ''
ORDER BY industry;
```

**Let's take a look at these**

```sql
SELECT *
FROM layoff_staging2
WHERE company LIKE 'Bally%';
```
**Nothing wrong here**

```sql
SELECT * FROM layoff_staging2
WHERE company LIKE 'Airbnb%';
```

**It looks like Airbnb is a travel, but this one just isn't populated**

**I'm sure it's the same for the others. What we can do is**

**Write a query that if there is another row with the same company name, it will update it to the non-null industry values**

**Makes it easy so if there were thousands we wouldn't have to manually check them all**

**We should set the blanks to nulls since those are typically easier to work with them**

```sql
UPDATE layoff_staging2
SET industry = NULL
WHERE industry = '';
```

**Now if we check those are all null**

```sql
SELECT *
FROM layoff_staging2
WHERE industry IS NULL 
OR industry = ''
ORDER BY industry;
```

**Now we need to populate those nulls if possible**

```sql
UPDATE layoff_staging2 t1
SET industry=t2.industry
FROM layoff_staging2   t2
WHERE t1.company = t2.company
AND t1.industry IS NULL 
AND t2.industry IS NOT NULL;


SELECT *
FROM layoff_staging2
WHERE company LIKE 'Airbnb%';



SELECT *
FROM layoff_staging2
WHERE industry IS NULL 
OR industry = ''
ORDER BY industry;
```

**And if we check it looks like Bally's was the only one without a populated row to populate this null values**


  ## 3. Null values or blank Values
  
```sql
SELECT FROM layoff_staging2
WHERE stage IS NULL
OR dates IS NULL;

DELETE FROM layoff_staging2
WHERE stage IS NULL
OR dates IS NULL;
```

**Total rows 2331**

**The null values in total_laid_off, percentage_laid_off, and funds_raised_millions all look normal**

**I like having them null because it makes it easier for calculations during the EDA phase**

**So there isn't anything I want to change with the null values**


## 4. Remove any Columns

```sql
ALTER TABLE layoff_staging2
DROP COLUMN row_num;

SELECT * 
FROM layoff_staging2;
```

## Exploratory Data Analysis

```sql
SELECT * FROM layoff_staging2;
```

**Looking at Percentage to see how big these layoffs were**

```sql
SELECT MAX(total_laid_off),MAX(percentage_laid_off) FROM layoff_staging2;
```

**Which companies had 1 which is basically 100 percent of they company laid off**

```sql
SELECT * FROM layoff_staging2 where percentage_laid_off=1 ORDER by total_laid_off DESC;
```

**Companies with the most Total Layoffs, Amazon holds the top position.**

```sql
SELECT company, SUM(total_laid_off) FROM layoff_staging2
GROUP BY 1 ORDER BY 2 DESC;

SELECT company, SUM(percentage_laid_off) FROM layoff_staging2
GROUP BY 1 ORDER BY 2 DESC;

SELECT company, AVG(percentage_laid_off) FROM layoff_staging2
GROUP BY 1 ORDER BY 2 DESC;

SELECT company,TO_CHAR(dates,'YYYY'), SUM(total_laid_off) FROM layoff_staging2
GROUP BY 1,2 ORDER BY 3 DESC;
```

**Based on industry type consumer industry on top position.**

```sql
SELECT industry, SUM(total_laid_off) FROM layoff_staging2
GROUP BY 1 ORDER BY 2 DESC;
```


**Based on location US had more number of layoffs**

```sql
 SELECT country, SUM(total_laid_off) FROM layoff_staging2
GROUP BY 1 ORDER BY 2 DESC;
```


**Based on dates**

```sql
SELECT MIN(dates),MAX(dates) FROM layoff_staging2;

SELECT dates, sum(total_laid_off) FROM layoff_staging2 GROUP BY 1 ORDER by 1 DESC;

SELECT TO_CHAR(dates,'YYYY')AS YEAR, sum(total_laid_off) FROM layoff_staging2 GROUP BY 1 ORDER by 1 DESC;

SELECT TO_CHAR(dates,'MM')AS MONTH, sum(total_laid_off) FROM layoff_staging2 GROUP BY 1 ORDER by 1;
 
 

SELECT stage, sum(total_laid_off) FROM layoff_staging2 GROUP BY 1 ORDER by 1 DESC limit 5;
```


**Earlier we looked at Companies with the most Layoffs. Now let's look at that per year. It's a little more difficult.**

```sql
WITH Company_Year AS 
(
  SELECT company, TO_CHAR(dates,'YYYY') AS years, SUM(total_laid_off) AS total_laid_off
  FROM layoff_staging2
  GROUP BY company, TO_CHAR(dates,'YYYY')
)
, Company_Year_Rank AS (
  SELECT company, years, total_laid_off, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
  FROM Company_Year
)
SELECT company, years, total_laid_off, ranking
FROM Company_Year_Rank
WHERE ranking <= 3
AND years IS NOT NULL
ORDER BY years ASC, total_laid_off DESC;
```



