# Full Text Search

## Searching a Table without an Index

```SQL
SELECT description
FROM titles
WHERE to_tsvector('english', description) @@ to_tsquery('english', 'friend');
```

In my setup this takes approximately 400ms to find 429 rows.

To find the 10 latest titles added with the description containing  `friend`, we can use following command.

```SQL
SELECT description, date_added
FROM titles
WHERE to_tsvector('english', description) @@ to_tsquery('english', 'friend')
ORDER BY date_added DESC
LIMIT 10;
```

## Creating an Index

To speed up the search we create an Index for the `description` column.

```SQL
CREATE INDEX description_idx ON titles USING gin(to_tsvector('english', description));
```

The above command now only takes 100ms to find 429 rows.

## Create a seperate `tsvector` column

```SQL
ALTER TABLE titles ADD COLUMN textsearchable_description tsvector;
UPDATE titles SET textsearchable_description =
     to_tsvector('english', coalesce(description,''));
```

And create an index for the new column

```SQL
CREATE INDEX textsearch_idx ON titles USING gin(textsearchable_description);
```

To see what `tsvector`does behind the scenes we can execute the following command:

```SQL
SELECT to_tsvector('english', 'a fat  cat sat on a mat - it ate a fat rats');
```

This will output `'ate':9 'cat':3 'fat':2,11 'mat':7 'rat':12 'sat':4`.

## Create a concatenated `tsvector` column

```SQL
ALTER TABLE titles ADD COLUMN ts_title_description tsvector;
UPDATE titles SET ts_title_description =
    to_tsvector('english', coalesce(title,'') || ' ' || coalesce(description,''));
```

And create an index for the column.

```SQL
CREATE INDEX ts_title_description_idx ON titles USING gin(ts_title_description);
```

## Defining Weights

```SQL
UPDATE titles SET ts_title_description =
    setweight(to_tsvector(coalesce(title,'')), 'A')    ||
    setweight(to_tsvector(coalesce(description,'')), 'B');
```

## Ranking

```SQL
SELECT description, ts_rank_cd(ts_title_description, query, 32 /* rank/(rank+1) */ ) AS rank
FROM titles, to_tsquery('friend | enemy') query
WHERE  query @@ ts_title_description
ORDER BY rank DESC
LIMIT 10;
```

## Highlighting results with `ts_headline`

```SQL
SELECT show_id, ts_headline(description, q), rank
FROM (SELECT show_id, description, q, ts_rank_cd(textsearchable_description, q) AS rank
      FROM titles, to_tsquery('friend') q
      WHERE textsearchable_description @@ q
      ORDER BY rank DESC
      LIMIT 10) AS foo;
```

## Use `querytree` to find out which tokens are used for searching

```SQL
SELECT querytree(to_tsquery('english', 'Hello & who & is & this'));
```

In this case only `hello`is used.

## Triggers for Automatic Updates

When storing the tokens in a seperate column, you need to create a trigger to update the tokens every time the original column is updated.

Because the built-in triggers `tsvector_update_trigger()`and `tsvector_update_trigger_column()` treat all the input columns alike, we need to create a custom trigger to weight `title` differently from `description`.

First we define the weighting function:

```SQL
CREATE FUNCTION title_description_trigger() RETURNS trigger AS $$
begin
  new.ts_title_description :=
     setweight(to_tsvector('pg_catalog.english', coalesce(new.title,'')), 'A') ||
     setweight(to_tsvector('pg_catalog.english', coalesce(new.description,'')), 'B');
  return new;
end
$$ LANGUAGE plpgsql;
```

and then the custom trigger:

```SQL
CREATE TRIGGER ts_title_description_update BEFORE INSERT OR UPDATE
    ON titles FOR EACH ROW EXECUTE PROCEDURE title_description_trigger();
```

Now when you update a row for example,

```SQL
UPDATE titles
SET title = title || ' cars'
WHERE show_id = 80173271;
```

the `car` token gets added to `ts_title_description` automatically.

## Use `ts_stat()` to get document statistics

- **ndoc**: number of documents (tsvectors) the word occurred in
- **nentry**: total number of occurrences of the word

```SQL
SELECT * FROM ts_stat('SELECT ts_title_description FROM titles')
ORDER BY nentry DESC, ndoc DESC, word
LIMIT 10;
```

## Use `pg_trgm` to find similiar words (Did you mean...?)

Use the `pg_trgm` module for determining the similarity of alphanumeric text based on trigram matching

```SQL
CREATE EXTENSION pg_trgm;
```

Create a new table with all the words of `title` and `description`.

```SQL
CREATE TABLE words AS SELECT word FROM
        ts_stat('SELECT to_tsvector(''simple'', title) || to_tsvector(''simple'', description) FROM titles');
```

Create a trigram index on the word column

```SQL
CREATE INDEX words_idx ON words USING GIN (word gin_trgm_ops);
```

To return all words that are sufficiently similar to `friend` execute

```SQL
SELECT word, similarity(word, 'friend') AS sml
FROM words
WHERE word % 'friend'
ORDER BY sml DESC, word;
```

To return all words for which there is a continuous extent in the corresponding ordered trigram set that is sufficiently similar to the trigram set of `friend` execute

```SQL
SELECT word, word_similarity('friend', word) AS sml
FROM words
WHERE 'friend' <% word
ORDER BY sml DESC, word;
```
