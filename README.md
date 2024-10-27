# SQL_-Data-cleaning-and-EDA-Project

 This postgreSQL query was run by https://sqliteonline.com/


## Data Cleaning

**Import layoff dataset from localfile**

```sql
select * from layoff;
```

**1. Remove Duplicates**

**2. Standardizing the Data**

**3. Null values or blank Values**

**4. Remove any Column**


## 1. Remove Duplicates

```sql
create table layoff_staging (like layoff);
insert INTO layoff_staging SELECT * from layoff;
SELECT * from layoff_staging;
SELECT count (*) from layoff_staging;


SELECT *,
 ROW_NUMBER() OVER(PARTITION by company, 'location', industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions)
 as row_num FROM layoff_staging;

with duplicate_cte AS
(
  SELECT *,
  ROW_NUMBER() OVER
  (
    PARTITION by company, 'location', industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions)
    as row_num FROM layoff_staging
 )
  SELECT * from duplicate_cte WHERE row_num >1;
  
  
  
  create table layoff_staging2(like layoffs_staging);
  alter table layoff_staging2 add column row_num INT;
  insert INTO layoff_staging2
  SELECT *,
  ROW_NUMBER() OVER(PARTITION by company, 'location', industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions)
  as row_num FROM layoffs_staging;

 SELECT * FROM layoff_staging2;
```
 
 --We can delete rows were row_num is greater than 2

```sql
 DELETE from layoff_staging2 WHERE row_num >1;
 SELECT COUNT (*) FROM layoff_staging2;
 ```
 
 **Total rows 2361. Now 22 duplicate rows are deleted**
 
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
  SELECT * from layoff_staging2 WHERE industry LIKE 'Crypto%';
  UPDATE layoff_staging2 SET industry='Crypto' WHERE industry LIKE 'Crypto%';
  SELECT industry FROM layoff_staging2;
```
  
  **We have some "United States" and some "United States." with a period at the end. Let's standardize this.**
  
```sql
  SELECT DISTINCT country FROM layoff_staging2 ORDER BY  country;
  UPDATE layoff_staging2 SET country ='United States' WHERE  country LIKE 'United States%';
  SELECT country FROM layoff_staging2;
```
  
 **Now we can convert the data type properly**

```sql
  ALTER TABLE layoff_staging2 RENAME COLUMN date TO dates;
  ALTER TABLE layoff_staging2 ALTER COLUMN dates TYPE DATE USING TO_DATE(dates,'MM/DD/YYYY');
  SELECT dates FROM layoff_staging2;
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

**We should set the blanks to nulls since those are typically easier to work with**

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
  SELECT * FROM layoff_staging2 WHERE 
   total_laid_off IS NULL AND 
   percentage_laid_off IS NULL AND
   funds_raised_millions IS NULL;
   
   DELETE FROM layoff_staging2
   WHERE total_laid_off IS NULL
   AND percentage_laid_off IS NULL
   AND funds_raised_millions IS NULL;

    SELECT count (*) from layoff_staging2;
```
    
     
 **After data cleaning proces 41 rows were deleted. Remaining rows are 2297**

## 4. Remove any Column

```sql
ALTER TABLE layoff_staging2
DROP COLUMN row_num;

SELECT * 
FROM layoff_staging2;
```
