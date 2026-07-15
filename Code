-- DATA CLEANING

SELECT * 
FROM layoffs;

-- Bcoz we don't alter our original dataset, we'll create another table and copy data there

-- Blueprint of the table
CREATE TABLE layoffs_staging
LIKE layoffs;

-- Data from the table
INSERT layoffs_staging
SELECT * FROM layoffs;

-- Need to assign row numbers to eliminate duplicates and stuff; this doesn't even have any numbering/ID to uniquely identify a row.
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY Company, Location, Industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num -- here, partit. by all columns 
FROM layoffs_staging;

-- Identifying duplicates; i.e., wherever row_num > 1, i.e., 2, 3, ...
SELECT *
FROM (
	SELECT company, industry, total_laid_off,`date`,
		ROW_NUMBER() OVER (
			PARTITION BY company, industry, total_laid_off,`date`
			) AS row_num
	FROM 
		world_layoffs.layoffs_staging
) duplicates
WHERE row_num > 1;

-- Selecting a random company from output to verify if it is indeed a duplicate
select * from layoffs_staging where company = '#Paid';

-- Now, for operational purposes, store these in a CTE
WITH duplicate_cte AS 
(
    SELECT * 
    FROM ( -- this poora is just duplicate selection query copy pasted
        SELECT company, industry, total_laid_off, `date`,
            ROW_NUMBER() OVER (
                PARTITION BY company, industry, total_laid_off, `date`
            ) AS row_num
        FROM world_layoffs.layoffs_staging 
    ) duplicates  -- duplicates means the same as writing 'AS Duplicates' --> it's an alias for the cte name
    WHERE row_num > 1
)
-- Now you can use the CTE, e.g., to view the duplicate rows:
SELECT * FROM duplicate_cte;

-- Unfortunately, becuase CTEs are view only, we can't go like "Delete from duplicate_cte where row_num > 1"
-- So, we create another copy, and delete from there.
-- Create a new table with row numbers, then drop rows where row_num > 1

DROP TABLE IF EXISTS layoffs_staging2;

CREATE TABLE layoffs_staging2 AS
SELECT *,
    ROW_NUMBER() OVER (
        PARTITION BY company, industry, total_laid_off, `date`
    ) AS row_num
FROM layoffs_staging;

-- because safe updates was off, this is how to temporarily turn them on.
-- 1. Disable safe updates (only for this session)
SET SQL_SAFE_UPDATES = 0;
-- 2. Now run your DELETE (this will work!)
DELETE FROM layoffs_staging2 WHERE row_num > 1;
-- 3. Re-enable safe updates to keep your database protected
SET SQL_SAFE_UPDATES = 1;

SELECT * 
FROM world_layoffs.layoffs_staging2 order by company;

-- Standardizing Data Formats (extra Spaces removal at beggining and ending, etc.)

SELECT * 
FROM world_layoffs.layoffs_staging2;

-- View comparison of with and without incorrect spaces
SELECT company, TRIM(company) 
FROM layoffs_staging2;

-- Remove extra spaces
UPDATE layoffs_staging2
SET company = TRIM(company);

-- Also for country
SELECT DISTINCT country
FROM world_layoffs.layoffs_staging2
ORDER BY country;

UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);

-- now if we run this again it is fixed
SELECT DISTINCT country
FROM world_layoffs.layoffs_staging2
ORDER BY country;

-- Clearly, Crypto is one company with multiple different variations for its name. We need to standardize that - let's say all to Crypto
SELECT DISTINCT industry
FROM world_layoffs.layoffs_staging2
ORDER BY industry;

UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency', 'CryptoCurrency');

-- now that's taken care of:
SELECT DISTINCT industry
FROM world_layoffs.layoffs_staging2
ORDER BY industry;

-- Date is of text data type, and we have to change it to 'date' type, so we'll first change it to the date format and then change type in the table
SELECT *
FROM world_layoffs.layoffs_staging2;

-- we can use str to date to update this field
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

-- now we can convert the data type properly
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;

-- Now, we start eliminating the NULL values in the table
SELECT * 
FROM layoffs_staging2 
WHERE total_laid_off IS NULL;

-- Now, it's observed that some of the percentage_laid_off are NULLS too - these rows where both are NULL are pretty much useless
-- Viewing these useless rows
SELECT * 
FROM layoffs_staging2 
WHERE total_laid_off IS NULL 
AND percentage_laid_off IS NULL;

UPDATE layoffs_staging2 
SET industry = NULL 
WHERE industry = '';
-- After running this, 3 chnages observed appears in the output 

-- Since there were some NULLs in the industry column too
SELECT * 
FROM layoffs_staging2 
WHERE industry IS NULL OR industry = '';

-- However, there was an Airbnb with the industry filled before - noticed while looking at the NULLs of total_laid_off. So, we can just fill this using that old value
SELECT *
FROM layoffs_staging2 t1 -- table1
JOIN layoffs_staging2 t2 -- table2
ON t1.company = t2.company 
WHERE (t1.industry IS NULL or t1.industry = '')
AND t2.industry IS NOT NULL; -- this will populate the empty/NULL values where it's the same company but filling out the industry was missed out

-- To see a comparison of populated and non populated ones,
SELECT t1.industry, t2.industry
FROM layoffs_staging2 t1 -- table1
JOIN layoffs_staging2 t2 -- table2
ON t1.company = t2.company 
WHERE (t1.industry IS NULL or t1.industry = '')
AND t2.industry IS NOT NULL;

-- Now, we'll need to update the table itself
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
-- Not working; maybe because of the blanks being kinda unpredictable, so we'll convert them to NULL. For that, go up to line 125 - the one just below identifying the useless rows

-- Since we know Airbnb was one of the changes we'd made,
SELECT * 
FROM layoffs_staging2
WHERE layoffs_staging2.company = 'Airbnb';
-- Clearly, the industry has now been filled, since it reads Travel instead of NULL or a blank now.

-- Now, let's check all by running again
SELECT * 
FROM layoffs_staging2 
WHERE industry IS NULL OR industry = '';

-- Bally's is still empty, so we'll check that specifically
SELECT * 
FROM layoffs_staging2 
WHERE company LIKE 'Bally%';
-- Clearly, there wasn't any other populated row, so can't do anything about this.

-- THAT's ALL FOR POPULATING THE NULL VALUES BECAUSE THE OTHER NULL ENTRIES IN COLUMNS LIKE total_laid_off, PERCENTAGE_LAID_OFF, ETC. CAN'T BE FILLED BY US, HAS TO BE DONE BY USER
-- MIGHT HAVE BEEN POSSIBLE IF WE HAD TOTAL EMPLOYEES PREV OR SOMETHING

-- Now, back to the useless rows - those that have NULLS for total_laid_off and percentage_laid_off - it's really possible if they did not lay off any; we don't know
-- So, we can delete, but deletion is tricky deal. However, these don't seem to serve any purpose, and we can't infer anything, so we'll do it.

DELETE 
FROM layoffs_staging2 
WHERE (total_laid_off IS NULL) AND (percentage_laid_off IS NULL);

SELECT * 
FROM layoffs_staging2;

-- Now, the row_num column has become redundant and is taking unnecessary space. So, we'll drop it from the table.
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;

SELECT * 
FROM layoffs_staging2;

-- Finally, data cleaning has been completed.
-- What was done:
-- Removing duplicates, standardizing the data, dealing with null or empty/blank values and removing redundant columns and rows.
