![Netfli](NetflixLogo.jfif)
# Netflix Shows 

## Project Overview: 
 In this project, I will comprehensively analyze Netflix’s movie and TV show datasets using SQL. I aim to extract valuable insights and address key business questions based on the data. This README provides a detailed account of the project’s objectives, the business challenges I aim to address, the solutions I implemented, and the findings and conclusions I derived.


## Purposes
- Analyze the Distribution of Content Types: Examine the distribution between movies and TV shows to understand their prevalence in the dataset.
- Identify Common Ratings: Determine the most frequent ratings assigned to both movies and TV shows, providing insights into content quality and audience preferences.
- Examine Content by Release Year, Country, and Duration: Analyze and categorize content based on their release years, countries of origin, and durations to identify trends and patterns.
- Explore and Categorize Content by Keywords: Investigate and classify content using specific criteria and keywords to uncover thematic trends and content characteristics.
  
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

### 3. Oldest movie in the dataset
``` sql
SELECT
    show_id,
    title,
    country,
    release_year,
    rating,
    description
FROM
    netflix_data
WHERE
    type = 'Movie'
ORDER BY
    release_year ASC
LIMIT 1;
```

### 4. Find the Top 5 Countries with the Most Content on Netflix
``` sql

SELECT new_country, COUNT(show_id) AS total_content
FROM (
    SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS new_country, show_id
    FROM netflix_data
    WHERE country IS NOT NULL
) AS t1
WHERE new_country IS NOT NULL
GROUP BY new_country
ORDER BY total_content DESC
LIMIT 5;
```

### 5. Rank directors based on the number of shows they have directed
``` sql
SELECT 
    director, 
    COUNT(*) AS show_count,
    RANK() OVER (ORDER BY COUNT(*) DESC) AS rank
FROM netflix_data
WHERE director IS NOT NULL
GROUP BY director;
``` 

### 6. Calculate the difference in the number of shows added between consecutive months.
``` sql
SELECT 
    TO_CHAR(CAST(date_added AS DATE), 'YYYY-MM') AS month, 
    COUNT(*) AS show_count,
    LAG(COUNT(*)) OVER (ORDER BY TO_CHAR(CAST(date_added AS DATE), 'YYYY-MM')) AS previous_month_count,
    (COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY TO_CHAR(CAST(date_added AS DATE), 'YYYY-MM'))) / LAG(COUNT(*)) OVER (ORDER BY TO_CHAR(CAST(date_added AS DATE), 'YYYY-MM')) * 100 AS growth_rate
FROM netflix_data
GROUP BY month;
``` 

### 7. count the number of shows in custom duration categories (e.g., short, medium, long)
``` sql
SELECT 
    CASE 
        WHEN CAST(REGEXP_REPLACE(duration, '[^0-9]', '', 'g') AS INTEGER) < 30 THEN 'Short'  --[REGEXP_REPLACE: to strip out any non-numeric characters from the duration column, leaving only the numeric part]
        WHEN CAST(REGEXP_REPLACE(duration, '[^0-9]', '', 'g') AS INTEGER) BETWEEN 30 AND 60 THEN 'Medium'
        ELSE 'Long'
    END AS duration_category,
    COUNT(*) AS show_count
FROM netflix_data
WHERE type = 'Movie'
GROUP BY duration_category;
``` 

### 8. Find the directors who have directed shows in multiple countries.
``` sql
SELECT director
FROM netflix_data
GROUP BY director
HAVING COUNT(DISTINCT country) > 1
LIMIT 5;
``` 

### 9. Calculate the percentage of total shows each director has directed.
``` sql
SELECT 
    director, 
    COUNT(*) AS show_count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS percentage_of_total
FROM netflix_data
WHERE director IS NOT NULL
GROUP BY director;
``` 
### 10. Find shows that have the same director but different countries
``` sql
SELECT a.title AS show1, b.title AS show2, a.director
FROM netflix_data a
JOIN netflix_data b ON a.director = b.director AND a.country <> b.country;
```

### 11. Find the most common words in the descriptions of shows.
``` sql
SELECT word, COUNT(*) AS word_count
FROM (
    SELECT UNNEST(STRING_TO_ARRAY(description, ' ')) AS word
    FROM netflix_data
) AS words
GROUP BY word
ORDER BY word_count DESC
LIMIT 10;
``` 
### 12. Identify and remove duplicate shows based on title and release year
``` sql
DELETE FROM netflix_data
WHERE show_id NOT IN (
    SELECT MIN(show_id)
    FROM netflix_data
    GROUP BY title, release_year
);
SELECT * FROM netflix_data;
``` 













