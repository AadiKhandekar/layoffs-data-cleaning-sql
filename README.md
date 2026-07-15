# Layoffs Dataset Preparation and Data Quality Enhancement

A SQL-based data cleaning project focused on preparing a real-world layoffs dataset for downstream analysis. The project demonstrates a structured data-cleaning workflow using MySQL, including duplicate removal, data standardization, handling missing values, and improving overall data quality.

## Dataset

**Source:** Kaggle - Layoffs 2022 Dataset

The dataset contains information on company layoffs across industries and countries, including:
- Company
- Location
- Industry
- Total Employees Laid Off
- Percentage Laid Off
- Date
- Company Stage
- Country
- Funds Raised (Millions)

> Note: The dataset was obtained from Kaggle and is used only for educational and portfolio purposes.

---

## Project Objectives

- Preserve the original dataset using staging tables.
- Identify and remove duplicate records.
- Standardize inconsistent categorical values.
- Convert improperly formatted data types.
- Handle missing values wherever possible.
- Remove records that contain insufficient information for analysis.
- Produce a clean, analysis-ready dataset.

---

## Technologies Used

- MySQL
- SQL
- Window Functions
- Common Table Expressions (CTEs)
- Joins
- Data Cleaning Techniques

---

## Data Cleaning Workflow

### 1. Create a Staging Table

A staging table was created to ensure that the original dataset remained unchanged throughout the cleaning process.

- Created an exact copy of the original table
- Imported all records into the staging table

---

### 2. Identify and Remove Duplicate Records

Since the dataset did not contain a unique identifier, duplicate detection was performed using SQL window functions.

Techniques used:

- `ROW_NUMBER()`
- `PARTITION BY`

Duplicate rows were assigned row numbers, after which only the first occurrence of each record was retained.

---

### 3. Standardize Data

Several inconsistencies were corrected to improve data quality.

Examples include:

- Removing leading and trailing spaces
- Standardizing country names
- Standardizing industry labels
- Converting text-based dates into SQL DATE format

---

### 4. Handle Missing Values

Missing values were addressed wherever reliable information was available.

This included:

- Converting blank values into NULLs
- Populating missing industry values using self joins
- Leaving values NULL when no reliable replacement existed

---

### 5. Remove Unusable Records

Rows containing insufficient information (for example, missing both total layoffs and percentage laid off) were removed because they could not contribute meaningful insights during analysis.

---

## SQL Concepts Demonstrated

- Window Functions (`ROW_NUMBER()`)
- Common Table Expressions (CTEs)
- Self Joins
- UPDATE and DELTE statements
- Data Type Conversion
- String Functions (`TRIM`)
- Date Functions (`STR_TO_DATE`
- Staging Tables

---

## Key Learnings

Through this project, I gained hands-on experience with:

- Building reproducible SQL data-cleaning workflows
- Preparing raw datasets for analysis
- Using window functions for duplicate detection
- Handling real-world inconsistencies and missing values
- Writing clean and maintainable SQL scripts

---

## Future Improvements

- Perform Exploratory Data Analysis (EDA) on the cleaned dataset.
- Build interactive dashboards using Power BI or Tableau.
- Automate the cleaning workflow using stored procedures.
- Add validation queries to compare raw and cleaned datasets.

---

## Author

**Aadi Khandekar**

Electronics and Communication Engineering  
Thapar Institute of Engineering and Technology

