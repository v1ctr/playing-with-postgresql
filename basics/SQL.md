# The SQL Language

## Get all Columns in Table

```SQL
SELECT * FROM titles;
```

## Get specific Columns in Table

```SQL
SELECT title, description
FROM titles;
```

## Filter Rows

```SQL
SELECT title, release_year
FROM titles
WHERE release_year=2019;
```

## Order Rows

```SQL
SELECT title, release_year
FROM titles
ORDER BY release_year;
```

## Ignore Duplicates

```SQl
SELECT DISTINCT country
FROM titles
ORDER BY country;
```

## Show all Titles which were added in the last 12 months

```SQL
SELECT * FROM titles
WHERE date_added >  CURRENT_DATE - INTERVAL '12 months';
```

## Join Tables

```SQL
SELECT titles.title, ratings.description
FROM titles, ratings
WHERE titles.rating = ratings.rating;
```

This can also be written as

```SQL
SELECT titles.title, ratings.description 
FROM titles
INNER JOIN ratings ON (titles.rating = ratings.rating);
```

You'll notice, that the commands above will only show titles with a raiting that is defined in our ratings database.

To show all titles we can use `LEFT OUTER JOIN`.
Titles without a rating description will be `[null]`.

```SQL
SELECT titles.title, ratings.description
FROM titles
LEFT OUTER JOIN ratings ON (titles.rating = ratings.rating);
```

## Aggregate Functions

### Use min() to get the lowest value

```SQL
SELECT min(release_year) FROM titles;
```

If you want to show the title of the movie with the lowest release year,
you need to use a subquery.

```SQL
SELECT title, release_year
FROM titles
WHERE release_year = (SELECT min(release_year) FROM titles);
```

## Update existing rows

```SQL
UPDATE titles
SET release_year = release_year - 2
WHERE release_year = 1925;
```

## Remove rows

```SQL
DELETE FROM titles
WHERE release_year = 1923;
```

## Create View

If you need a SQL command mor often, you can define a view.

```SQL
CREATE VIEW ratings_view AS
    SELECT titles.title, ratings.description
    FROM titles, ratings
    WHERE titles.rating = ratings.rating;
```

Now you can query the view just like a table.

```SQL
SELECT * FROM ratings_view;
```
