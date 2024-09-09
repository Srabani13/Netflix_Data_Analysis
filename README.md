# Netflix Shows 

## Project Description
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset
The data for this project is sourced from the Kaggle dataset:

- [Movies Dataset](https://www.kaggle.com/path-to-dataset)



### Netflix Adv.SQL Project 
## Schema
Here is the SQL schema for the `netflix_data` table:

```SQL

CREATE TABLE netflix_data
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);

```
## Checking the data 
```SQL
SELECT * FROM netflix_data;
```

## Checking the total count to verify that the data was successfully imported
```SQL
SELECT COUNT(*) AS total_items
FROM netflix_data;
```

## Verifying the distinct type
```SQL
SELECT DISTINCT type
FROM netflix_data;
```


## Count the number of NULL values for each column in the table
```SQL
SELECT
    -- Count NULL values in the 'show_id' column
    SUM(CASE WHEN show_id IS NULL THEN 1 ELSE 0 END) AS NullCount_show_id,
    -- Count NULL values in the 'type' column
    SUM(CASE WHEN type IS NULL THEN 1 ELSE 0 END) AS NullCount_type,
    -- Count NULL values in the 'title' column
    SUM(CASE WHEN title IS NULL THEN 1 ELSE 0 END) AS NullCount_title,
    -- Count NULL values in the 'director' column
    SUM(CASE WHEN director IS NULL THEN 1 ELSE 0 END) AS NullCount_director,
    -- Count NULL values in the 'cast' column
    SUM(CASE WHEN "casts" IS NULL THEN 1 ELSE 0 END) AS NullCount_cast,
    -- Count NULL values in the 'country' column
    SUM(CASE WHEN country IS NULL THEN 1 ELSE 0 END) AS NullCount_country,
    -- Count NULL values in the 'date_added' column
    SUM(CASE WHEN date_added IS NULL THEN 1 ELSE 0 END) AS NullCount_date_added,
    -- Count NULL values in the 'release_year' column
    SUM(CASE WHEN release_year IS NULL THEN 1 ELSE 0 END) AS NullCount_release_year,
    -- Count NULL values in the 'rating' column
    SUM(CASE WHEN rating IS NULL THEN 1 ELSE 0 END) AS NullCount_rating,
    -- Count NULL values in the 'duration' column
    SUM(CASE WHEN duration IS NULL THEN 1 ELSE 0 END) AS NullCount_duration,
    -- Count NULL values in the 'listed_in' column
    SUM(CASE WHEN listed_in IS NULL THEN 1 ELSE 0 END) AS NullCount_listed_in,
    -- Count NULL values in the 'description' column
    SUM(CASE WHEN description IS NULL THEN 1 ELSE 0 END) AS NullCount_description
FROM netflix_data;  
```
### Now that I have an understanding of the data, I can proceed with a more in-depth analysis.

### 1. Count the Number of Movies vs TV Shows
**Count the number of Movies and TV Shows separately with total count**

```SQL

SELECT
    COUNT(*) AS Total_Records,
    SUM(CASE WHEN type = 'Movie' THEN 1 ELSE 0 END) AS Number_of_Movies,
    SUM(CASE WHEN type = 'TV Show' THEN 1 ELSE 0 END) AS Number_of_TV_Shows

FROM netflix_data;
```


### 2. Find the Most Common Rating for Movies and TV Shows

**Most common rating for Movies and TV Shows combined**
**Creates a `common table expression (CTE)` to count ratings for both movies and TV shows.**

```sql
WITH CommonRatings AS (
    SELECT

    type,

  rating,

COUNT(*) AS count

  FROM netflix_data

 GROUP BY type, rating
)
-- Filters results for movies and TV shows and selects only the rating with the maximum count for each type.

SELECT
    type,
    rating,
    count

FROM CommonRatings

WHERE (type = 'Movie' OR type = 'TV Show')

 AND count = (

      SELECT MAX(count)
        FROM CommonRatings AS CR

  WHERE CR.type = CommonRatings.type
    )
ORDER BY type;
```




















