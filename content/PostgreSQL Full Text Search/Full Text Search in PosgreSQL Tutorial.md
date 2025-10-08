---
title: Full Text Search in PostgreSQL
tags:
  - PostgreSQL
  - FullTextSearch
  - Tutorial
  - Database
  - SQL
---
*A quick tutorial*

In our detailed article on [[Full Text Search in PosgreSQL|Full Text Search (FTS) in PostgreSQL]] we compare search-engines like Elasticsearch or Solr to PostgreSQL FTS, go through configurations, weighting, indexing, briefly explaining the GIN (Generalised Inverted Index), core functions, and ranking results. 

Here we want to give a quick tutorial on FTS with some real world examples. To run yourself checkout this [repo](https://github.com/RiccardoScott1/postgres_text ).
## What You'll Learn
We walk you through building a full-text search system in PostgreSQL. You’ll start by configuring language settings, then create efficient search indexes - first with a simple expression, then with a more powerful generated column.

We’ll add weighted indexing to prioritise certain fields (like giving titles more weight than descriptions), and use `ts_rank` to control how results are ranked. Finally, we’ll explore the different query functions PostgreSQL offers, from simple keyword searches to structured and phrase-based queries.

By the end, you’ll know how to index, rank, and query text effectively using PostgreSQL’s full-text search features.

## The Dataset
We'll be using an extract from the imdb-dataset, found on Kaggle [imdb-dataset-of-top-1000-movies-and-tv-shows](https://www.kaggle.com/datasets/harshitshankhdhar/imdb-dataset-of-top-1000-movies-and-tv-shows) with different columns on movies and series:

- **Poster_Link** - Link of the poster that imdb using
- **Series_Title** - Name of the movie
- **Released_Year** - Year at which that movie released
- **Certificate** - Certificate earned by that movie
- **Runtime** - Total runtime of the movie
- **Genre** - Genre of the movie
- **IMDB_Rating** - Rating of the movie at IMDB site
- **Overview** - mini story/ summary
- **Meta_score** - Score earned by the movie
- **Director** - Name of the Director
- **Star1,Star2,Star3,Star4** - Name of the Stars
- **No_of_votes** - Total number of votes
- **Gross** - Money earned by that movie

The main text columns: `series_title`  and `overview` will be our focus.

| series_title             | overview                                                                                                               |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| The Shawshank Redemption | Two imprisoned men bond over a number of years, finding solace and eventual redemption through acts of common decency. |
| The Godfather            | An organized crime dynasty's aging patriarch transfers control of his clandestine empire to his reluctant son.         |

To get a Postgres instance running and importing the dataset follow this [readme](https://github.com/RiccardoScott1/postgres_text ).

## Language Config
You can set a default for your session or database:
```sql
SET default_text_search_config = 'english';
```
Or always specify the language in your `tsvector()` and `tsquery()` statements.

## Creating and Indexing Text Searchable Columns
###  `tsvector` as a plain Concatenation

PostgreSQL offers multiple ways to implement full-text search; either inline with an expression index or by using a generated column. Here’s a quick comparison of the two approaches:
#### **Option 1: Expression-Based FTS Index**

```sql
CREATE INDEX imdb_movies_idx 
ON imdb_movies 
USING GIN (
  to_tsvector('english', series_title || ' ' || overview)
);
```

This creates a **GIN index directly on a computed tsvector**, built from `series_title` and `overview`. It doesn’t add any new columns, the tsvector is computed **at query time**, then matched against the index.

#### **Option 2: Stored Generated Column with FTS Index**

```sql
ALTER TABLE imdb_movies
ADD COLUMN textsearchable_index_col tsvector
GENERATED ALWAYS AS (
  to_tsvector('english', coalesce(series_title, '') || ' ' || coalesce(overview, ''))
) STORED;

CREATE INDEX textsearch_idx 
ON imdb_movies 
USING GIN (textsearchable_index_col);
```

This adds a **persistent `tsvector` column** that is automatically generated and updated by PostgreSQL whenever the row changes. The index is built on this stored column.

#### Pros of Option 2 (Generated Column)
- **Stored once, indexed once**: The `tsvector` is computed only once per row insert/update - not at query time.    
- **Faster queries**: Queries skip on-the-fly tsvector generation.
- **Reusable**: The column can be used in queries, materialised views, triggers, etc.
- **Debuggable**: Easier to inspect and troubleshoot: you can `SELECT` it directly.
   
#### Cons of Option 2
- **Schema complexity**: Extra column in your table.
- **Increased storage**: The column consumes additional disk space.
- **Less flexible**: Changing the logic requires dropping and recreating the column.

Use **Option 1** for simplicity and flexibility, especially during prototyping. Use **Option 2** for **performance, reusability, and maintainability**. We'll be using **Option 2** in the following.

## `tsvector` with prefixed weights
You can also **assign weights** to different parts of a document to indicate their relative importance in ranking search results. 

PostgreSQL allows you to label tokens in the `tsvector` with weights `A`, `B`, `C`, or `D`, where `A` is considered the most important and `D` the least. The default weights for  `A`, `B`, `C`, and `D` are {1.0, 0.4, 0.2, 0.1} .

This is especially useful when your document has structured fields/columns like a `title`, `body` or `keywords`, and you want matches in more important fields (e.g., the title) to rank higher. In our case we give `series_title` the higher weight `A` and `overview` the lower weight `B`.

```sql
ALTER TABLE imdb_movies
ADD COLUMN textsearchable_index_col_weighted tsvector
GENERATED ALWAYS AS (
	setweight(to_tsvector('english', coalesce(series_title, '')), 'A') ||
	setweight(to_tsvector('english', coalesce(overview, '')), 'B')
) STORED;

CREATE INDEX textsearch_idx_weighted
ON imdb_movies USING GIN (textsearchable_index_col_weighted);
```

Now we have two generated and indexed `tsvector` columns in our table:
- purely concatenated text `textsearchable_index_col` (previous query)
- weighted: `series_title` weighted higher than `overview`: `textsearchable_index_col_weighted` 

## Ranking Full-Text Search Results in PostgreSQL with Weights
Let’s explore how to **rank search results** in PostgreSQL using **`ts_rank`** and custom **weight vectors** to influence relevance. We'll break down this query line-by-line:
```sql
SELECT
  series_title,
  overview,
  ts_rank(textsearchable_index_col_weighted, query) AS rank,
  ts_rank('{0.1, 0.1, 1.0, 0.1}', textsearchable_index_col_weighted, query) AS rank_weights_inverted
-- {0.1, 0.2, 0.4, 1.0} are the default weights for D, C, B, A
FROM
  imdb_movies,
  plainto_tsquery('fish') query
WHERE query @@ textsearchable_index_col_weighted
ORDER BY rank DESC
LIMIT 10;
```

### Line-by-Line Explanation
We're selecting two text columns from the `imdb_movies` table for display - this gives context to each result, `series_title` and `overview`.

**`ts_rank(textsearchable_index_col_weighted, query) AS rank,`**
This computes the **default relevance score** (`rank`) of each document using PostgreSQL’s built-in ranking function.
- `textsearchable_index_col_weighted` is the generated `tsvector` that contains **weighted tokens**, `'A'` for `series_title`  and `'B'` for `overview`
- `query` is a **`tsquery`** (more on that below).
- `ts_rank()` gives higher scores to matches found in higher-weight fields (`A` > `B` > `C` > `D`).

**`ts_rank('{0.1, 0.1, 1.0, 0.1}', textsearchable_index_col_weighted, query) AS rank_weights_inverted`**
- Here’s the cool part: we override the default weights.
- `{0.1, 0.1, 1.0, 0.1}` corresponds to weights `{D, C, B, A}`
- We're saying: **give priority to B-weighted tokens**, downplaying `A`, `C`, and `D`.
This way the **overview** weighs more than the title - the **inverse of the default** that we set [[Full Text Search in PosgreSQL Tutorial#one with prefixed weights|above]].

**`FROM imdb_movies, plainto_tsquery('fish') query`**
- We pull from the `imdb_movies` table and use a **plain text search term** (`'fish'`) to create a `tsquery`: `plainto_tsquery('fish')` tokenises and normalises the input string using the configured language.
- We alias the result as `query`, so it can be reused in the SELECT clause and WHERE condition.

**`WHERE query @@ textsearchable_index_col_weighted`**
This filters the rows to only those where the `tsvector` matches the `tsquery`.
- `@@` is the **match operator**: returns true when the query matches the indexed text.

We finally sort results by the default rank in descending order (highest relevance first) and keep top 10.

### Output

| series_title            | overview                                                                              | rank          | rank_weights_inverted |
| ----------------------- | ------------------------------------------------------------------------------------- | ------------- | --------------------- |
| **Big Fish**            | A frustrated son tries to determine the fact from fiction in his dying father's life. | **0.6079271** | 0.06079271            |
| **The Bourne Identity** | A man is picked up by a fishing boat... suffering from amnesia...                     | 0.24317084    | **0.6079271**         |

 How does **inverting the weights** flip the ranking order?

- With default weights, **title matches** (A) dominate: `Big Fish` ranks higher.
- With inverted weights, the **overview match** (B) dominates: `The Bourne Identity` jumps ahead.

### Key Takeaways
- PostgreSQL's `ts_rank` lets you **tune result relevance** with weighting vectors.
- Default weight order is: `{D, C, B, A} = {0.1, 0.2, 0.4, 1.0}`
- By inverting weights, you can **prioritise body content** over title or metadata.

Want to boost relevance of user-generated descriptions? Or downplay noisy titles? Now you know how. 

## Building FTS Queries in PostgreSQL
PostgreSQL offers several functions to construct `tsquery` objects for full-text search. Each one serves a different purpose, for example handling raw user input, structured queries, or phrase searches. Here's a quick comparison using `series_title` and `overview` from the `imdb_movies` table:

```sql
SELECT
	series_title,
	overview,
	ts_rank(textsearchable_index_col, query) AS rank
FROM
	imdb_movies,
	THE_EXAMPLE_QUERY query
WHERE query @@ textsearchable_index_col_weighted
ORDER BY rank DESC
LIMIT 10;
```

### `to_tsquery`: **Structured Query Logic**
Use this when you want full control over logical operators (`&`, `|`, `!`, `<->`).
```sql
SELECT to_tsquery('english', 'fish & memory');
-- 'fish' & 'memori'
```
Good for advanced filtering or programmatically generated queries.

>Finds: **The Bourne Identity**:*A man is picked up by a fishing boat, bullet-riddled and suffering from amnesia, before racing to elude assassins and attempting to regain his memory.*

### `plainto_tsquery`: **Simple AND Searches**
Great for clean user input or when you just want to match all words.
```sql
SELECT plainto_tsquery('english', 'robot space journey');
-- 'robot' & 'space' & 'journey'
```
Ideal for basic search bars without needing advanced syntax.

>Finds: **WALL·E**: *In the distant future, a small waste-collecting robot inadvertently embarks on a space journey that will ultimately decide the fate of mankind.*

### `phraseto_tsquery`: **Exact Phrase Matching**
Preserves word order with `<->`, perfect for looking up movie taglines or quotes.
```sql
SELECT phraseto_tsquery('english', 'space journey');
-- 'space' <-> 'journey'
```
Also respects stop word positions for accurate phrase structure.

>Finds **WALL·E** again.

### `websearch_to_tsquery`: **Search Engine Style**
Most intuitive for users. Handles quoted phrases, `-NOT`, and `or` logic naturally.
```sql
SELECT websearch_to_tsquery('english', '"space journey" -robot');
-- 'space' <-> 'journey' & !'robot'
```
Perfect for building Google-style search experiences in web apps.

>Finds no results, because the only movie with the phrase "space journey" is **WALL·E** , but it also contains 'robot', which is excluded in the query. 

### Summary

| Function               | Purpose                       | Handles Operators | Preserves Word Order | Best For                          |
| ---------------------- | ----------------------------- | ----------------- | -------------------- | --------------------------------- |
| `to_tsquery`           | Precise logic & filtering     | ✅                 | Optional (`<->`)     | Power users, advanced filters     |
| `plainto_tsquery`      | Quick AND queries             | ❌ (auto `&`)      | ❌                    | Simple inputs, basic search bars  |
| `phraseto_tsquery`     | Phrase and proximity searches | ❌ (auto `<->`)    | ✅                    | Quotes, taglines, search accuracy |
| `websearch_to_tsquery` | Search-engine-like input      | ✅                 | ✅                    | End-user queries, web apps        |

## Wrapping Up
PostgreSQL’s full-text search is powerful, flexible, and surprisingly easy to set up. With just a few lines of SQL, you can build fast, ranked search over rich text data - no external search engine required. FTS in Postgres gives you full control over how text is indexed, weighted, and queried. 

Try it out, tweak the weights, and see how your results change - it’s all right there in your Postgres DB!

For in depth article see [[Full Text Search in PosgreSQL|here]] .
