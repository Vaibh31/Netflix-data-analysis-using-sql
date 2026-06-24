# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Project Overview

This project explores Netflix Movies and TV Shows data using PostgreSQL. The analysis focuses on solving real-world business questions and extracting insights through SQL queries. The project demonstrates data exploration, aggregation, filtering, string manipulation, CTEs, and window functions.

## Project Goals

- Analyze the distribution of Movies and TV Shows.
- Explore content ratings and genres.
- Identify trends based on countries and release years.
- Practice SQL using real-world business scenarios.
- Generate meaningful insights from entertainment data.

## Project Highlights

- Analyzed 8,807 Netflix titles.
- Solved 15 real-world business problems using SQL.
- Used CTEs, Window Functions, Aggregations, and String Manipulation.
- Performed content trend analysis across countries, ratings, genres, and release years.
- Built entirely using PostgreSQL.
- 
## Dataset

- **Dataset Link:** https://www.kaggle.com/datasets/shivamb/netflix-shows

## Database Schema

```sql
DROP TABLE IF EXISTS netflix;

CREATE TABLE netflix
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

# Business Problems and Solutions

## 1. Count the Number of Movies vs TV Shows

**Purpose:** Understand the distribution of content types on Netflix.

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```

---

## 2. Find the Most Common Rating for Movies and TV Shows

**Purpose:** Identify the most frequently assigned rating for each content type.

```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

---

## 3. List All Movies Released in a Specific Year (e.g., 2020)

**Purpose:** Retrieve all movies released in a selected year.

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

---

## 4. Find the Top 5 Countries with the Most Content on Netflix

**Purpose:** Identify the countries contributing the highest number of titles.

```sql
SELECT * 
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

---

## 5. Identify the Longest Movie

**Purpose:** Find the movie with the greatest runtime.

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

---

## 6. Find Content Added in the Last 5 Years

**Purpose:** Analyze recently added content on Netflix.

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

---

## 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

**Purpose:** Explore content created by a specific director.

```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

---

## 8. List All TV Shows with More Than 5 Seasons

**Purpose:** Identify long-running TV series available on Netflix.

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

---

## 9. Count the Number of Content Items in Each Genre

**Purpose:** Analyze content distribution across genres.

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

---

## 10. Find Each Year and the Average Number of Content Releases in India

**Purpose:** Identify the years with the highest percentage of content releases from India.

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

---

## 11. List All Movies that are Documentaries

**Purpose:** Retrieve documentary content from the catalog.

```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

---

## 12. Find All Content Without a Director

**Purpose:** Identify records with missing director information.

```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

---

## 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

**Purpose:** Analyze recent appearances of a specific actor.

```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

---

## 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

**Purpose:** Identify the most frequently featured actors in Indian-produced content.

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

---

## 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

**Purpose:** Classify content based on keywords found in descriptions.

```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```

---


## Key Insights

- Netflix offers a diverse mix of Movies and TV Shows.
- The platform contains content from multiple countries and genres.
- Rating analysis helps understand target audiences.
- Country-wise trends reveal major content-producing regions.
- SQL can effectively uncover business insights from large datasets.

## Conclusion

This project demonstrates practical SQL skills by solving real-world business problems using Netflix data. The analysis showcases data exploration, aggregation, filtering, CTEs, window functions, and string operations in PostgreSQL.

