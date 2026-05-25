## Project Overview

This project focuses on cleaning and preparing a raw layoffs dataset using SQL.
The dataset originally contained duplicate records, NULL values, blank fields, inconsistent formatting, and incorrect data types, which made the data unreliable for analysis.

To ensure data safety, the original raw table was not modified directly. Instead, a copy of the dataset was created in staging tables, and all cleaning operations were performed on those tables.

The main objective of this project was to transform messy raw data into a clean, standardized, and analysis-ready dataset.



# Dataset Information

The original dataset consisted of the following columns:

```sql
CREATE TABLE layoffs (
  company text,
  location text,
  industry text,
  total_laid_off int DEFAULT NULL,
  percentage_laid_off text,
  date text,
  stage text,
  country text,
  funds_raised_millions int DEFAULT NULL
);
```



### Problems Found in Dataset

The dataset contained several data quality issues such as:
* Duplicate rows
* NULL values
* Blank values
* Inconsistent company and country names
* Different formats for the same industry
* Date column stored as TEXT instead of DATE
* Records with missing layoff information



# Data Cleaning Process

## 1. Creating a Staging Table

Before starting the cleaning process, a duplicate copy of the raw dataset was created.

```sql
CREATE TABLE layoffs_staging
LIKE layoffs;

INSERT layoffs_staging
SELECT *
FROM layoffs;
```

### Why This Step Was Important
Creating a staging table helped protect the original dataset from accidental changes.
All transformations and cleaning operations were safely performed on the copied data.



# 2. Removing Duplicate Records

Duplicate records were identified using the `ROW_NUMBER()` window function.

```sql
ROW_NUMBER() OVER(
	PARTITION BY company, location, industry,
	total_laid_off, percentage_laid_off,
	date, stage, country, funds_raised_millions
)
```

A new table called `layoffs_staging2` was created with an additional column named `row_num`.
This column assigned a sequence number to duplicate rows.
Rows where `row_num > 1` were considered duplicates and deleted.


### Outcome
* Duplicate entries were successfully removed
* Dataset became more accurate and reliable



# 3. Standardizing the Data

After removing duplicates, inconsistent values and formatting issues were standardized.


## Cleaning Company Names
Extra spaces in company names were removed using the `TRIM()` function.

```sql
UPDATE layoffs_staging2
SET company = TRIM(company);
```

### Example
Before:
* `" Amazon "`

After:
* `"Amazon"`



## Standardizing Industry Names
Different variations of Crypto-related industries were converted into a single standardized value.

```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

### Example
Before:
* Crypto Currency
* Crypto 

After:
* Crypto


## Cleaning Country Names
Some country names contained unnecessary periods (`.`) at the end.

```sql
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);
```

### Example
Before:
* United States.

After:
* United States


## Converting Date Format
The `date` column was originally stored as TEXT, which was not suitable for date analysis.
The values were converted using:

```sql
STR_TO_DATE(date,'%m/%d/%Y')
```

Then the column datatype was changed to `DATE`.

```sql
ALTER TABLE layoffs_staging2
MODIFY COLUMN date DATE;
```

### Why This Step Was Important
Converting dates into proper DATE format makes filtering, sorting, and time-based analysis much easier.


# 4. Handling NULL and Blank Values

The dataset also contained many missing and blank values.


## Replacing Blank Industry Values with NULL
Blank values in the `industry` column were converted into NULL values.

```sql
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';
```


## Filling Missing Industry Values
Some rows had missing industry names, while other rows from the same company contained valid industry data.
A self join was used to populate missing values.

```sql
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```

### Outcome
* Missing industry values were recovered
* Dataset completeness improved


# 5. Removing Unnecessary Records

Some records contained NULL values in both:
* `total_laid_off`
* `percentage_laid_off`

These rows did not provide any meaningful layoff information.
Before deleting them, a backup table was created.

```sql
CREATE TABLE layoffs_null
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

After backup creation, the records were removed.

```sql
DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

### Outcome
* Irrelevant records were removed
* Final dataset became cleaner and more useful for analysis


# 6. Removing Temporary Columns

The helper column `row_num` was only required during duplicate removal.
After the cleaning process was completed, the column was dropped.

```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```


# SQL Concepts Used
This project involved several important SQL concepts and functions.

| SQL Concept       | Purpose                                            |
| ----------------- | -------------------------------------------------- |
| CREATE TABLE      | Creating staging and backup tables                 |
| INSERT INTO       | Copying data into staging tables                   |
| ROW_NUMBER()      | Identifying duplicate records                      |
| CTE (WITH Clause) | Simplifying duplicate detection queries            |
| WINDOW FUNCTIONS  | Assigning row numbers to partitions                |
| UPDATE            | Standardizing and cleaning data                    |
| DELETE            | Removing duplicate and unnecessary records         |
| SELF JOIN         | Filling missing values using existing company data |
| TRIM()            | Removing unnecessary spaces and characters         |
| LIKE              | Pattern matching for similar values                |
| STR_TO_DATE()     | Converting text into DATE format                   |
| ALTER TABLE       | Modifying table structure                          |
| NULL Handling     | Managing missing data properly                     |


# Final Result

After completing the data cleaning process:
* Duplicate rows were removed
* NULL and blank values were handled
* Data formatting was standardized
* Dates were converted into proper DATE format
* Unnecessary records were removed
* Dataset became clean and analysis-ready

The final cleaned dataset can now be used efficiently for:
* Data Analysis
* Reporting
* Visualization
* Dashboard Creation
* Business Insights
