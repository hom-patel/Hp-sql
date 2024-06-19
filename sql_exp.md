-- Select all records from layoffs_staging2
SELECT *
FROM layoffs_staging2;

-- Select the maximum number of total_laid_off from layoffs_staging2
SELECT MAX(total_laid_off)
FROM world_layoffs.layoffs_staging2;

-- Select the maximum and minimum percentage_laid_off from layoffs_staging2, excluding NULL values
SELECT MAX(percentage_laid_off), MIN(percentage_laid_off)
FROM world_layoffs.layoffs_staging2
WHERE percentage_laid_off IS NOT NULL;

-- Select all records where percentage_laid_off equals 1
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE percentage_laid_off = 1;

-- Select all records where percentage_laid_off equals 1 and order them by total_laid_off in descending order
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY total_laid_off DESC;

-- Select all records where percentage_laid_off equals 1 and order them by funds_raised_millions in descending order
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;

-- Select company names and the sum of total_laid_off for each company, ordered by the sum in descending order
SELECT company, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;

-- Select the minimum and maximum dates from layoffs_staging2
SELECT MIN(`date`), MAX(`date`)
FROM world_layoffs.layoffs_staging2;

-- Select industry names and the sum of total_laid_off for each industry, ordered by the sum in descending order
SELECT industry, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;

-- Select country names and the sum of total_laid_off for each country, ordered by the sum in descending order
SELECT country, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;

-- Select years and the sum of total_laid_off for each year, ordered by year in ascending order
SELECT YEAR(date), SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY YEAR(date)
ORDER BY 1 ASC;

-- Select stages and the sum of total_laid_off for each stage, ordered by the sum in descending order
SELECT stage, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY stage
ORDER BY 2 DESC;

-- Select months (in YYYY-MM format) and the sum of total_laid_off for each month, ordered by month in ascending order
SELECT 
    SUBSTRING(`date`, 1, 7) AS `MONTH`,
    SUM(total_laid_off) AS total_laid_off
FROM 
    world_layoffs.layoffs_staging2
WHERE 
    SUBSTRING(`date`, 1, 7) IS NOT NULL
GROUP BY 
    SUBSTRING(`date`, 1, 7)
ORDER BY 
    1 ASC;

-- Select all records from layoffs_staging2
SELECT * 
FROM layoffs_staging2;

-- Create a Common Table Expression (CTE) to calculate rolling total layoffs by month
WITH DATE_CTE AS (
    SELECT 
        SUBSTRING(`date`, 1, 7) AS `MONTH`,
        SUM(total_laid_off) AS total_laid_off
    FROM 
        world_layoffs.layoffs_staging2
    WHERE 
        SUBSTRING(`date`, 1, 7) IS NOT NULL
    GROUP BY 
        SUBSTRING(`date`, 1, 7)
    ORDER BY 
        1 ASC
)
-- Select months and rolling total layoffs
SELECT 
    `MONTH`,
    SUM(total_laid_off) OVER (ORDER BY `MONTH`) AS rolling_total_layoffs
FROM 
    DATE_CTE
ORDER BY 
    `MONTH` ASC;

-- Select company names and the sum of total_laid_off for each company, ordered by the sum in descending order
SELECT company, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;

-- Select company names, years, and the sum of total_laid_off for each company-year pair, ordered by the sum in descending order
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY company, YEAR(`date`)
ORDER BY 3 DESC;

-- Select company names, years, and the sum of total_laid_off for each company-year pair
SELECT company, YEAR(date) AS years, SUM(total_laid_off) AS total_laid_off
FROM layoffs_staging2
GROUP BY company, YEAR(date);

-- Create a Common Table Expression (CTE) to rank companies by total layoffs per year
WITH Company_Year AS (
  SELECT company, YEAR(date) AS years, SUM(total_laid_off) AS total_laid_off
  FROM layoffs_staging2
  GROUP BY company, YEAR(date)
), COMPANY_RANK_YEAR AS (
  SELECT *, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS RANKING
  FROM Company_Year
  WHERE years IS NOT NULL
)
-- Select the top 5 companies by total layoffs for each year
SELECT * 
FROM COMPANY_RANK_YEAR
WHERE RANKING <= 5;
