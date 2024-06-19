-- Select all records from the layoffs table
SELECT *
FROM layoffs;

-- Create a new table layoffs_staging with the same structure as layoffs
CREATE TABLE layoffs_staging
LIKE layoffs;

-- Verify the structure of the newly created table layoffs_staging
SELECT *
FROM layoffs_staging;

-- Insert all records from layoffs into layoffs_staging
INSERT INTO layoffs_staging
SELECT * 
FROM layoffs;

-- Add a row number to each record partitioned by multiple columns to identify duplicates
SELECT *, 
ROW_NUMBER() OVER ( 
  PARTITION BY location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
) AS row_num
FROM layoffs_staging;

-- Create a Common Table Expression (CTE) to find duplicate records
WITH duplicate_cte AS (
  SELECT *, 
  ROW_NUMBER() OVER ( 
    PARTITION BY location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
  ) AS row_num
  FROM layoffs_staging
)
-- Select only the duplicate records
SELECT *
FROM duplicate_cte
WHERE row_num > 1;

-- Check records where the company is 'Viamo'
SELECT *
FROM layoffs_staging
WHERE company = 'Viamo';

-- Create a new table layoffs_staging2 with specified columns
CREATE TABLE `layoffs_staging2` (
  `company` TEXT,
  `location` TEXT,
  `industry` TEXT,
  `total_laid_off` INT DEFAULT NULL,
  `percentage_laid_off` TEXT,
  `date` TEXT,
  `stage` TEXT,
  `country` TEXT,
  `funds_raised_millions` INT DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- Verify the structure of layoffs_staging2
SELECT *
FROM layoffs_staging2
WHERE row_num > 1;

-- Insert data into layoffs_staging2 with row numbers for duplicate identification
INSERT INTO layoffs_staging2
SELECT *, 
ROW_NUMBER() OVER ( 
  PARTITION BY location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
) AS row_num
FROM layoffs_staging;

-- Delete duplicate records from layoffs_staging2
DELETE FROM layoffs_staging2
WHERE row_num > 1;

-- Verify the data after removing duplicates
SELECT *
FROM layoffs_staging2;

-- Select all distinct company names
SELECT DISTINCT(company)
FROM layoffs_staging2;

-- Trim leading and trailing spaces from company names
SELECT TRIM(company)
FROM layoffs_staging2;

-- Update the company names to remove leading and trailing spaces
UPDATE layoffs_staging2
SET company = TRIM(company);

-- Select all distinct industry values
SELECT DISTINCT industry
FROM layoffs_staging2
ORDER BY 1;

-- Update industry values that start with 'Crypto' to 'Crypto'
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

-- Select all distinct industry values to verify the update
SELECT DISTINCT industry
FROM layoffs_staging2
ORDER BY 1;

-- Select all distinct location values
SELECT DISTINCT location
FROM layoffs_staging2
ORDER BY 1;

-- Select distinct country values and trim trailing periods
SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;

-- Update country values to remove trailing periods
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

-- Convert date strings to DATE format
SELECT `date`, STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging2;

-- Update date column to DATE format
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

-- Modify the date column to be of type DATE
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;

-- Verify the data after updating the date column
SELECT *
FROM layoffs_staging2;

-- Select all distinct industry values in world_layoffs.layoffs_staging2
SELECT DISTINCT industry
FROM world_layoffs.layoffs_staging2
ORDER BY industry;

-- Select records where industry is NULL or empty
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE industry IS NULL 
OR industry = ''
ORDER BY industry;

-- Select records where the company name starts with 'Bally'
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE company LIKE 'Bally%';

-- Select records where the company name starts with 'airbnb'
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE company LIKE 'airbnb%';

-- Update industry to NULL where it is an empty string
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

-- Select records where industry is NULL or empty after the update
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE industry IS NULL 
OR industry = ''
ORDER BY industry;

-- Update industry based on matching company names where industry is NULL
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;

-- Verify records where industry is still NULL or empty after the update
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE industry IS NULL 
OR industry = ''
ORDER BY industry;

-- Select all records from layoffs_staging2
SELECT *
FROM layoffs_staging2;

-- Select records where both total_laid_off and percentage_laid_off are NULL
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- Delete records where both total_laid_off and percentage_laid_off are NULL
DELETE FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- Verify the data after deleting the records
SELECT *
FROM layoffs_staging2;

-- Drop the row_num column from layoffs_staging2
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
