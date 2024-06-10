# Trigram in Postgresql

The PostgreSQL similarity function is part of the `pg_trgm` extension, which provides functions and operators for determining the similarity between text strings based on trigram matching. 

### Basic of Trigram:

A trigram is a group of three consecutive characters from a string. For example, the word "hello" can be broken down into the following trigrams:

- '  h'

- ' he'

- 'hel'

- 'ell'

- 'llo'

- 'lo '

  

For the word 'hi', trigram would be

- '  h'
- ' hi'
- 'hi '

### Similarity Function

The `similarity` function calculates the similarity between two text strings by comparing their trigrams. Here's how it works step-by-step:

1. Total number of trigrams from both the string. In the above: 6 + 3 = 9
2. Total number of string matches or intersects. Here it is: '  h' only. So count is 1.
3. Total number of uniq trigrams from both the string. In the above: 8

So the calculation will be: 1/8 = .125

### How it works in Query tool:

To use the `similarity` function, you first need to enable the `pg_trgm` extension:

```sql
CREATE EXTENSION pg_trgm;
```

Then, you can use the `similarity` function in a query:

```sql
SELECT similarity('hello', 'hi');
```

This query would return a similarity score between "hello" and "hi".

You can expand and compare your calculation by running:

```sql
SELECT show_trgm('hello'), show_trgm('hi'), similarity('hello', 'hi');
```

which will results in:

| show_trgm text[]                | show_trgm text[]    | similarity |
| ------------------------------- | ------------------- | ---------- |
| {"  h"," he",ell,hel,llo,"lo "} | {"  h"," hi","hi "} | 0.125      |

I think by default similarity threashold is set to 0.3. So if your searched string's similarity function is >= 0.3, only then you will get the result.

Another way is to adjust the threshold in the query itself.



```sql
SELECT name
FROM my_table
WHERE similarity(name, 'search_string') > 0.125
ORDER BY similarity(name, 'search_string') DESC;
```

**NB**:  For better performance, especially with large datasets, you can use a GIN (Generalized Inverted Index) or GIST (Generalized Search Tree) index on text columns to accelerate similarity searches.
