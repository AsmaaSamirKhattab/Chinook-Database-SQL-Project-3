# Chinook-Database-SQL
## Business Analytics Nanodegree from Udacity with Bertelsmann

### Project 3: Query a Digital Music Store Database using [SQLite local environment](https://sqlitebrowser.org/dl/)
In this project I will be using a new data set. It will be a music store.

## Project Description
In this project, you will query the Chinook Database. The Chinook Database holds information about a music store. 
For this project, you will be assisting the Chinook team with understanding the media in their store, their customers and employees, and their invoice information. 

### The schema for the Chinook Database is provided below:

![Chinook ERD](https://user-images.githubusercontent.com/58610546/191956713-6d0518a2-3712-4387-95b3-65e83bf0b93d.png)

## Instructions
You will need to follow the instructions on the next three concepts to get the Chinook database up and running on your own machine and check that it is set up correctly.

There will be two parts to this project:
- The first part is a series of questions that will assure you have mastered the main concepts taught throughout the SQL lessons.
*Though these questions will not be "graded" by a reviewer, they will help you self-assess.*
- The second part is a presentation. Similar to the first project, there isn't a 'right answer' for the second portion of the project. You have the ability to be creative in the questions you ask. You will then write a SQL query to pull the data needed to answer your question successfully. Use the pulled data to build a visual (bar chart, histogram, or another plot) that answers your question. 
The essentials of your project submission are discussed in the next sections. 
*To review your presentation, you will need to save your slides as a **PDF**.*

## Question 1: Which is the most popular media type?

```{r}
SELECT
  t.MediaTypeId,
  m.Name AS Most_popular_Mediatype,
  COUNT(TrackId) AS tracks
FROM Track t
JOIN MediaType m
  ON m.MediaTypeId = t.MediaTypeId
GROUP BY 1
ORDER BY 3 DESC
LIMIT 3;
```

## Question 2: Which artist has the most albums?

```{r}
SELECT
  l.ArtistId,
  a.Name AS Artist,
  COUNT(AlbumId) AS albums
FROM Artist a
JOIN Album l
  ON l.ArtistId = a.ArtistId
GROUP BY 1
ORDER BY 3 DESC
LIMIT 3;
```

## Question 3: Which Top 3 Artists in number of tracks sang?

```{r}
SELECT
  DISTINCT(a.ArtistId),
  a.Name,
  Count(t.TrackId) AS num_Tracks
FROM Artist a
LEFT JOIN Album l
  ON l.ArtistId = a.ArtistId
LEFT JOIN Track t
  ON t.AlbumId = l.AlbumId
LEFT JOIN InvoiceLine n
  ON n.TrackId = t.TrackId
GROUP BY 1
ORDER BY 3 DESC
Limit 3;
```

## Question 4: How many Artists composed their songs?

```{r}
WITH composer_list AS 
  (SELECT ar.ArtistId,
  ar.Name AS artist_name,
  t.Composer
FROM Artist ar
JOIN Album al
  ON ar.ArtistId = al.ArtistId
JOIN Track t
  ON al.AlbumId = t.AlbumId
ORDER BY 1),

composer_employed
AS (SELECT
  ArtistId,
  artist_name,
  Composer,
  CASE
		WHEN artist_name = Composer
		THEN "self composer"
    WHEN Composer  LIKE (artist_name || "%")
		THEN "self composer"
    WHEN Composer  LIKE ("%" || artist_name)
		THEN "self composer"
    WHEN Composer  LIKE ("&" || artist_name || "%")
		THEN "self composer"
		WHEN artist_name LIKE (Composer || "%")
		THEN "self composer"
    WHEN artist_name LIKE ("%" || Composer)
		THEN "self composer"
    WHEN artist_name LIKE ("&" || Composer || "%")
		THEN "self composer"
		WHEN Composer IS NULL
		THEN "no composer"
    ELSE "outside composer"
	END AS composer_type
FROM composer_list
ORDER BY 4 DESC, 2),

composer_segmentation
AS (SELECT
  CASE
    WHEN composer_type = "self composer"
    THEN 1
    ELSE 0
  END AS self_composer,
  CASE
    WHEN composer_type = "outside composer"
    THEN 1
    ELSE 0
  END AS outside_composer,
  CASE
    WHEN composer_type = "no composer"
    THEN 1
    ELSE 0
  END AS no_composer
FROM composer_employed)

SELECT (SELECT
         COUNT(*)
       FROM composer_list)
       AS total_songs,
       SUM(self_composer) AS self_composed,
       SUM(outside_composer) AS outside_composed,
       SUM(no_composer) AS none_composed
FROM composer_segmentation;
```
